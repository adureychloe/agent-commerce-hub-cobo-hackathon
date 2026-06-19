# Agent Commerce Hub — Crash Course for Beginners

> If you can read this project end-to-end, you can build Web3 + AI Agent projects on your own.

This guide assumes you know basic Python. Everything else is explained from scratch.

---

## 0. The Big Picture — What Does This Project Actually Do?

Every line of code in this repo serves one goal: **let an AI agent buy a paid service on a blockchain, without a human clicking "pay."**

Here is the full flow, in plain English:

```
1. Alice (seller) publishes a service to an on-chain registry
   → "I will do ETH market analysis for 0.00005 SETH"

2. Bob (buyer) tells his agent: "find me a cheap market analyst"
   → Agent reads the registry, finds Alice's service

3. Agent tells Bob: "Alice's service matches, costs 0.00005 SETH"

4. Bob's wallet creates a Pact (a permission slip)
   → "Allow up to 0.00005 SETH to be sent to Alice"

5. Bob approves the Pact in the Cobo wallet app

6. The money moves on-chain; Alice's endpoint returns the analysis

7. A proof (receipt) is stored on the contract
```

The whole repo is the machinery that makes steps 1-7 happen.

---

## 1. Five Concepts You Must Understand First

### 1.1 Blockchain and Smart Contracts

**What is a blockchain?** A public database that nobody can delete from. Every write is permanent and visible.

**What is a smart contract?** A program that lives on the blockchain. Once deployed, it runs the same way forever.

**In this project:** `contracts/ServiceRegistryV2.sol` is our smart contract. It stores:
- Which services are available (name, price, seller address)
- Which payments have been made (proof records)

**Key mental model:**
- **Reading** from a contract is free and instant (like a Google search)
- **Writing** to a contract costs gas (a small fee in ETH) and takes ~15-30 seconds on Sepolia

```solidity
// This is Solidity — looks like JavaScript/TypeScript
struct Service {
    uint256 id;
    string name;
    string description;
    address paymentAddress;   // where money goes
    uint256 priceWei;          // how much
    bool active;
}
```

**What you need to learn:** Read [Solidity by Example](https://solidity-by-example.org/) chapters 1-8. You don't need to write contracts, just understand `struct`, `mapping`, `modifier`, and `event`.

### 1.2 web3.py — Talking to the Blockchain from Python

`web3.py` is the Python library that lets you read and write to Ethereum/Sepolia.

```python
from web3 import Web3

# Connect to Sepolia
w3 = Web3(Web3.HTTPProvider("https://ethereum-sepolia-rpc.publicnode.com"))

# Read a contract
contract = w3.eth.contract(address="0x3f94...", abi=abi_json)
services = contract.functions.getActiveServices(0, 50).call()
# ↑ This is a READ — free, instant

# Write to a contract
tx = contract.functions.register(...).build_transaction({...})
signed = w3.eth.account.sign_transaction(tx, private_key)
tx_hash = w3.eth.send_raw_transaction(signed.raw_transaction)
# ↑ This is a WRITE — costs gas, takes time
```

**Where to find this in the code:** `agent_commerce_sandbox/chain_client_v2.py`

**What to learn:** The difference between `.call()` (read) and `.build_transaction()/.send_raw_transaction()` (write). This is the most important distinction in Web3 development.

### 1.3 FastAPI — Making Python Code Accessible via HTTP

FastAPI turns Python functions into web API endpoints.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/status")           # GET request → http://localhost:8080/api/status
def get_status():
    return {"status": "ok", "services": 7}

@app.post("/api/pay")             # POST request → send JSON, get JSON back
async def pay_service(payload: dict):
    result = do_payment(payload)
    return result
```

**Where to find this:** `web/app.py` — this is the main server file. Every `@app.get(...)` or `@app.post(...)` is an API endpoint.

**Debug tip:** When something breaks, start by curling the endpoint:

```bash
curl http://localhost:8080/api/status | python3 -m json.tool
```

### 1.4 x402 — The Payment Protocol

x402 is a pattern, not a library. It works like this:

```
Step 1: Buyer sends POST /request (no payment proof)
        → Seller replies: "402 Payment Required"
        → Response header: X-Payment-Info: {chain:"SETH", token:"SETH", amount:"0.00005", address:"0x..."}

Step 2: Buyer pays (via CAW transfer), gets a transaction hash

Step 3: Buyer sends POST /request?tx_hash=0x...
        → Seller checks the blockchain: "yes, this tx paid me"
        → Seller returns the actual service result
```

The "402" bit is a real HTTP status code. HTTP 402 means "Payment Required" — like 404 means "Not Found."

**Where to find this:**
- Seller side: `agent_commerce_sandbox/x402_server.py` — handles the `/request` endpoint
- Buyer side: `agent_commerce_sandbox/x402_client.py` — sends requests, parses 402, pays, retries

### 1.5 Cobo Agentic Wallet (CAW) — AI-Friendly Payments

Normal crypto wallets (MetaMask, Rabby) are browser extensions. A human must click "Approve" for every transaction.

CAW is different. It lets you:
1. Create a **Pact** — a permission slip that says "allow up to X SETH to be sent to address Y"
2. The human approves the Pact **once** (in the CAW mobile app)
3. After approval, an agent (or server) can execute transfers **without** the human clicking again

**The Pact is the key.** It constrains what the agent can do:
- Which chain? (Sepolia only)
- Which token? (SETH only)
- Which destination? (the seller's address)
- How much? (capped at the service price)

The agent can't drain the wallet because the Pact is scoped.

**Where to find this:**
- `agent_commerce_sandbox/caw_client.py` — Python wrapper around the `caw` CLI
- The `caw` CLI is a separate binary installed on the server

**Commands you should know:**
```bash
caw pact list --status active       # Show active Pacts
caw pact create ...                 # Create a new Pact
caw transfer --pact-id <id> ...     # Execute a transfer under a Pact
```

---

## 2. How the Code Fits Together — Module by Module

### 2.1 Entry Points (how you start the project)

| File | Role |
| --- | --- |
| `web/app.py` | Main FastAPI server — serves the web UI + all API endpoints |
| `run.py` | CLI tool — `python run.py discover`, `python run.py procure`, etc. |
| `web/index.html` | Browser UI — single HTML file with inline JavaScript |

### 2.2 Core Business Logic (the "engine room")

| File | Role |
| --- | --- |
| `agent_commerce_sandbox/chain_client_v2.py` | Talks to the Solidity contract on Sepolia. Every read (list services) and write (register service, record proof) goes through this file. |
| `agent_commerce_sandbox/caw_client.py` | Wraps the `caw` CLI. Handles Pact creation, reuse, transfer execution, and transaction tracking. |
| `agent_commerce_sandbox/x402_client.py` | The buyer-side x402 client. Sends a request, receives 402, parses the payment info, pays via CAW, retries with `tx_hash`. |
| `agent_commerce_sandbox/x402_server.py` | The seller-side x402 server. Receives a request, returns 402 if unpaid, verifies payment on-chain, returns the service result. |
| `agent_commerce_sandbox/procurement_agent.py` | Matches a user's natural-language request ("find me a cheap market analyst") to available services. Uses DeepSeek LLM (or falls back to keyword matching). |

### 2.3 Supporting Files

| File | Role |
| --- | --- |
| `agent_commerce_sandbox/engine.py` | Older payment orchestration flow. Still referenced by some routes. |
| `agent_commerce_sandbox/chain_client.py` | Older V1 contract client. Only used by legacy/debug routes. |
| `contracts/ServiceRegistryV2.sol` | The smart contract (Solidity). |
| `contracts/ServiceRegistryV2.abi.json` | The ABI — a JSON file that tells Python what functions the contract has. |
| `contracts/deployed_v2.json` | Where the contract lives on Sepolia (address + deploy tx hash). |
| `tests/` | Three test files using pytest + FastAPI TestClient. |

---

## 3. Reading the Code — Follow One Request End-to-End

The best way to understand this project is to trace **one complete purchase**.

### Step 1: User clicks "Buy" in the browser

File: `web/index.html`

```javascript
// The browser sends this:
fetch("/api/agent/buyer/procure", {
    method: "POST",
    body: JSON.stringify({
        request: "find me a cheap market analyst",
        budget_seth: 0.00005,
        wallet_address: "0x9e01...",
        auto_pay: true
    })
})
```

### Step 2: FastAPI receives the request

File: `web/app.py` → function `api_agent_buyer_procure()`

```python
@app.post("/api/agent/buyer/procure")
async def api_agent_buyer_procure(payload: BuyerProcureRequest):
    # 2a. Fetch services from the blockchain
    services = await _list_v2_services()

    # 2b. Match services to the user's request
    ranked, source = match_and_rank(payload.request, services)

    # 2c. Filter by budget
    affordable = [s for s in ranked if s["priceWei"] <= budget_wei]

    # 2d. Auto-pay (optional)
    result = await _auto_pay_x402(best_service, payload)
```

### Step 3: Reading from the blockchain

File: `agent_commerce_sandbox/chain_client_v2.py` → `list_active_services()`

```python
def list_active_services(self, offset=0, limit=50):
    # This calls the Solidity contract on Sepolia
    raw = self.contract.functions.getActiveServices(offset, limit).call()
    return [_service_from_tuple(s) for s in raw]
```

This is a **read** (free, instant). No gas, no waiting.

### Step 4: Matching services to the request

File: `agent_commerce_sandbox/procurement_agent.py` → `_ai_match()`

```python
# Sends the user's request + service list to DeepSeek
# Gets back: [{"service_id": 5, "score": 9, "reason": "perfect match"}]
```

### Step 5: Auto-paying via x402 + CAW

File: `web/app.py` → function around line 1279

```python
# 5a. Send request to seller's endpoint
resp = urlopen(Request(url, data=..., headers=...), timeout=30)
# Gets back: 402 Payment Required + X-Payment-Info header

# 5b. Create or reuse a CAW Pact
caw.submit_pact(policies, intent=...)

# 5c. Wait for CAW approval (human approves in app)
# 5d. Execute the transfer
tx_id = caw.execute_transfer(pact_id, amount, token, dest)

# 5e. Retry the request with tx_hash as proof
resp = urlopen(Request(f"{url}?tx_hash={tx_id}"), timeout=180)
# Gets back: the actual service result
```

### Step 6: Recording the delivery proof

File: `agent_commerce_sandbox/chain_client_v2.py` → `record_delivery()`

```python
def record_delivery(self, service_id, tx_hash, summary):
    tx = self.contract.functions.recordDelivery(
        service_id, tx_hash, summary
    ).build_transaction({...})
    signed = self.w3.eth.account.sign_transaction(tx, self._pk)
    tx_hash = self.w3.eth.send_raw_transaction(signed.raw_transaction)
```

This is a **write** (costs gas, takes time). The proof is now permanently on Sepolia.

---

## 4. What You Should Learn To Truly Master This

Here's the learning path, ordered by priority:

### Level 1: Read the project (1-2 days)

- [ ] Understand the flow diagram in section 0
- [ ] Open `web/app.py` and trace one API endpoint from `@app.post(...)` to `return`
- [ ] Read `contracts/ServiceRegistryV2.sol` and map each Solidity function to its Python caller in `chain_client_v2.py`
- [ ] Run the project locally with `uvicorn web.app:app --port 8080`

### Level 2: Understand the core concepts (3-5 days)

- [ ] **Solidity basics**: struct, mapping, modifier, event, require. Watch [Solidity by Example](https://solidity-by-example.org/) chapters 1-8.
- [ ] **web3.py**: understand `.call()` vs `.build_transaction()` vs `.send_raw_transaction()`. Practice by writing a small script that reads a public contract.
- [ ] **FastAPI**: understand route decorators (`@app.get`, `@app.post`), Pydantic models (`BaseModel`), dependency injection, async vs sync.
- [ ] **HTTP/API design**: understand HTTP methods (GET vs POST), status codes (200, 402, 403, 404), headers, query parameters.
- [ ] **The x402 pattern**: re-read sections 1.4 and 1.5 until you can explain the flow to someone else.

### Level 3: Build something (5-10 days)

- [ ] Clone the repo and get it running locally
- [ ] Add a new API endpoint (e.g., `/api/services/count` that returns just the number of services)
- [ ] Add a new field to the `Service` struct in the Solidity contract
- [ ] Re-deploy the contract to Sepolia and update `deployed_v2.json`
- [ ] **The ultimate test**: Add a "service rating" feature — let buyers rate services 1-5 stars and store the average rating on-chain.

### Level 4: Go deeper (ongoing)

- [ ] Learn ethers.js if you want to build a React/Next.js frontend instead of vanilla HTML
- [ ] Learn about gas optimization, reentrancy guards, and Solidity security
- [ ] Study the Cobo Agentic Wallet documentation to understand Pact policies in depth
- [ ] Build your own x402-compatible service from scratch

---

## 5. How to Debug This Project

### 5.1 The Most Common Debugging Pattern

When something breaks, start with the **narrowest possible test**:

```bash
# 1. Is the server even running?
curl -sS --max-time 10 http://localhost:8080/api/status | python3 -m json.tool

# 2. Can we read from the blockchain?
curl -sS http://localhost:8080/api/services | python3 -m json.tool | head -30

# 3. Does the Solidity contract exist?
# Check on Etherscan:
# https://sepolia.etherscan.io/address/0x3f945ba7BFE2181B506390c0C5e9d2328495Cc40

# 4. Is the CAW wallet healthy?
caw status
caw pact list --status active
```

### 5.2 Python Debugging

**Add print statements to the running server:**

The easiest way to see what's happening is to watch the uvicorn logs:

```bash
# Start the server in a visible terminal
python3 -m uvicorn web.app:app --host 0.0.0.0 --port 8080 --timeout-keep-alive 600

# Every incoming request is logged:
# INFO:     127.0.0.1:54321 - "GET /api/status HTTP/1.1" 200 OK
```

You can also add temporary `print()` calls to any Python file. They will appear in the uvicorn output.

**Run individual test files:**

```bash
# Run all tests
PYTHONPATH=. pytest -q

# Run one test file
PYTHONPATH=. pytest tests/test_buyer_agent_api.py -v

# Run a specific test function
PYTHONPATH=. pytest tests/test_buyer_agent_api.py::test_auto_pay_enforces_budget -v
```

### 5.3 Frontend/JavaScript Debugging

Open the browser's DevTools (F12), then:

1. **Console tab** — check for JavaScript errors
2. **Network tab** — see every API call, its payload, and response
3. **Application tab → Local Storage** — check if `agentCommerceBuyerCAWAddress` is set

**Check JavaScript syntax without a browser:**

```bash
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
class P(HTMLParser):
    def __init__(self): super().__init__(); self.scripts=[]; self.in=False; self.buf=[]
    def handle_starttag(self, t, a):
        if t=='script': self.in=True; self.buf=[]
    def handle_data(self, d):
        if self.in: self.buf.append(d)
    def handle_endtag(self, t):
        if t=='script' and self.in: self.scripts.append(''.join(self.buf)); self.in=False
p=P(); p.feed(Path('web/index.html').read_text())
Path('/tmp/inline.js').write_text('\n'.join(p.scripts))
PY
node --check /tmp/inline.js
```

### 5.4 CAW / Blockchain Debugging

**Check if a Pact exists and is active:**

```bash
caw pact list --status active
caw pact list --status pending_approval
```

**Check if a transaction succeeded:**

```bash
caw tx list --limit 5
```

**Check the contract on Sepolia Etherscan:**

https://sepolia.etherscan.io/address/0x3f945ba7BFE2181B506390c0C5e9d2328495Cc40

Click the "Read Contract" tab to see all services and proofs.

### 5.5 Common Errors and Their Fixes

| Error | Meaning | Fix |
| --- | --- | --- |
| `ContractLogicError` or `execution reverted` | The Solidity `require()` check failed (e.g., price must be > 0) | Check the contract code to see which `require` failed |
| `No TEST_PRIVATE_KEY in .env` | You need to set up the `.env` file for write operations | `cp .env.example .env` and fill in the values |
| `x402 buy failed after CAW step` | Payment was made but the seller didn't respond in time | Check if the seller endpoint is reachable; the payment tx is still valid on-chain |
| `Pact approval timed out` | You didn't approve the Pact in the CAW app within the timeout window | Open the CAW app, approve the Pact, and try again |
| `uvicorn: address already in use` | An old server process is still running on port 8080 | `ss -tlnp \| grep 8080` to find the PID, then `kill -9 <PID>` |
| `ModuleNotFoundError: No module named 'web3'` | Missing Python dependency | `pip install -r web/requirements.txt web3 pytest` |

---

## 6. Key Files to Read in Order

If you have 2 hours to understand this project, read these files in this order:

1. **`README.md`** (5 min) — What the project does
2. **`contracts/ServiceRegistryV2.sol`** (25 min) — The smart contract
3. **`agent_commerce_sandbox/chain_client_v2.py`** (25 min) — How Python talks to the contract
4. **`agent_commerce_sandbox/caw_client.py`** (25 min) — How payments work
5. **`web/app.py`** (first 200 lines, 15 min) — The main API server
6. **`agent_commerce_sandbox/x402_server.py`** (first 100 lines, 15 min) — How seller endpoints work
7. **`web/index.html`** (search for `fetch(` calls, 10 min) — How the browser calls the API
8. **`tests/test_buyer_agent_api.py`** (10 min) — How to test without touching the blockchain

---

## 7. Quick Reference — Commands You'll Use Often

```bash
# Start the server
cd /path/to/agent-commerce-hub-cobo-hackathon
python3 -m uvicorn web.app:app --host 0.0.0.0 --port 8080 --timeout-keep-alive 600

# Run tests
PYTHONPATH=. pytest -q

# Check JavaScript syntax
python3 -m py_compile web/app.py agent_commerce_sandbox/*.py

# CLI usage
python run.py discover              # List services
python run.py status                # Wallet + contract health
python run.py procure "request"     # Find and buy a service
python run.py proofs                # Show proof records

# CAW wallet
caw status                          # Wallet status
caw pact list --status active       # Active Pacts
caw pact list --status pending      # Pacts awaiting approval
caw tx list --limit 10             # Recent transactions

# Process management
ss -tlnp | grep 8080               # Who's using port 8080?
ps aux | grep uvicorn              # Find running server processes
kill -9 <PID>                      # Force-kill a process
```
