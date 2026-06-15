# Agent Commerce Hub

> Built for the AI x Web3 Agentic Builders Hackathon by Casual Hackathon, Cobo Agentic Wallet track.

Agent Commerce Hub is an agent-native service marketplace. Sellers publish AI services on-chain. A Buyer Agent discovers services, ranks candidates against a user request and budget, pays through an x402-style HTTP payment flow using Cobo Agentic Wallet, and records a delivery proof on Sepolia.

## Demo

- Live demo: https://gradually-clicker-tacking.ngrok-free.dev
- Demo video: https://youtu.be/EiuFYwxbaDU?is=JfCJIHOXJhDgpMjg

The live demo is hosted through a temporary tunnel for hackathon judging. If it is offline, run the project locally with the quick start steps below.

This repository contains the standalone hackathon version: Solidity contracts, Python/FastAPI backend, CLI tools, a browser demo, and tests.

## Why this exists

Most agent demos stop at API calls. Commerce needs a stricter loop:

1. discover an available service,
2. check price and budget,
3. get user-controlled payment authorization,
4. execute payment,
5. return a result,
6. leave an audit trail.

Agent Commerce Hub implements that loop with real testnet transactions instead of mocked payments.

## What it does

- On-chain service registry on Sepolia for seller service metadata, pricing, endpoints, and delivery proofs.
- Buyer Agent API that accepts a natural-language task, filters services by budget, ranks candidates, and can auto-pay for the selected service.
- Seller Agent API that turns a service brief into registry metadata and publishes the service behind an admin-gated demo endpoint.
- x402-style seller endpoint: unpaid requests receive `402 payment_required`; paid requests return service delivery and record proof.
- Cobo Agentic Wallet integration for Pact creation/reuse and token transfer execution.
- Agent Console UI for judges and users to inspect candidate ranking, payment state, transaction hashes, wallet status, and proof records.
- CLI for service listing, procurement, seller registration, direct x402 purchase, and proof inspection.

## Demo flow

```text
Seller Agent / Seller UI
        |
        | publish service metadata, price, endpoint, payment address
        v
Sepolia ServiceRegistryV2
        |
        | discover services
        v
Buyer Agent
        |
        | rank by request + budget, enforce max price
        v
x402 service endpoint
        |
        | 402 payment_required
        v
Cobo Agentic Wallet Pact + transfer
        |
        | approved payment tx hash
        v
service delivery + on-chain proof
```

## Key design choices

### Budget safety

The Buyer Agent does not only filter candidates before payment. It also passes a live `max_price_wei` guard into the actual x402 purchase call. If a seller changes the payment amount between selection and payment, the purchase is rejected before the CAW transfer.

### User-authorized agent payments

The demo uses a server-paired CAW wallet for automated payment execution. CAW Pact policy constrains payment scope by chain, token, destination, amount, and operation type. The browser UI shows the buyer CAW session state, but private CAW credentials are never stored in this repository.

### Auditable delivery

After payment, the seller endpoint returns the delivery result and records proof data on the registry contract. The UI and CLI expose transaction hashes and proof records so the flow can be audited after the agent acts.

### Demo safety

Seller Agent registration is protected by `DEMO_ADMIN_TOKEN` when configured. The public buyer/seller UI does not expose arbitrary deletion of proof history. Service removal is implemented as deactivation so historical proof remains visible.

## Tech stack

| Layer | Technology |
| --- | --- |
| Smart contracts | Solidity 0.8.x, Sepolia |
| Backend | Python 3.11+, FastAPI, web3.py |
| Payments | x402-style HTTP flow, Cobo Agentic Wallet, CAW Pact/Transfer |
| Frontend | HTML, CSS, vanilla JavaScript |
| Testing | pytest, FastAPI TestClient, Python compile checks |

## Repository structure

```text
agent_commerce_sandbox/
  caw_client.py          # Wrapper around the Cobo Agentic Wallet CLI
  chain_client_v2.py     # ServiceRegistryV2 client
  x402_client.py         # Buyer-side x402/CAW purchase client
  x402_server.py         # Seller-side x402 service and management APIs
  procurement_agent.py   # Natural-language service matching/procurement logic
  engine.py              # Earlier CAW orchestration flow kept for reference
contracts/
  ServiceRegistryV2.sol  # Main registry used by the demo
  deployed_v2.json       # Public Sepolia deployment metadata
scripts/
  deploy_v2.py           # Contract deployment helper
  register_demo_services_v2.py
web/
  app.py                 # FastAPI app and API routes
  index.html             # Buyer/Seller/Agent Console demo UI
tests/
  test_buyer_agent_api.py
  test_seller_agent_api.py
  test_agent_console_ui.py
run.py                   # CLI entry point
```

## On-chain deployment

The current demo deployment is on Sepolia.

| Item | Value |
| --- | --- |
| Contract | `ServiceRegistryV2.sol` |
| Network | Sepolia (`chain_id: 11155111`) |
| Address | [`0x3f945ba7BFE2181B506390c0C5e9d2328495Cc40`](https://sepolia.etherscan.io/address/0x3f945ba7BFE2181B506390c0C5e9d2328495Cc40) |
| Deploy tx | [`0x8e6ca555c927a5c9416a0db037ed70847c2592e650ce56ac60d4c281b6e1e18c`](https://sepolia.etherscan.io/tx/0x8e6ca555c927a5c9416a0db037ed70847c2592e650ce56ac60d4c281b6e1e18c) |

These addresses and transaction hashes are public testnet data. No private keys or CAW credentials are included.

## Quick start

### Prerequisites

- Python 3.11+
- Cobo Agentic Wallet CLI (`caw`) with a paired wallet for payment execution
- Sepolia ETH / SETH test assets
- A Sepolia RPC endpoint
- Optional: a browser wallet such as MetaMask or Rabby for seller-side service management

### Install

```bash
git clone https://github.com/adureychloe/agent-commerce-hub-cobo-hackathon.git
cd agent-commerce-hub-cobo-hackathon
python3 -m venv .venv
source .venv/bin/activate
pip install -r web/requirements.txt web3 pytest
```

### Configure

Copy the sample environment file and fill in your local values:

```bash
cp .env.example .env
```

Required for live chain operations:

```bash
RPC_URL=https://your-sepolia-rpc.example
CHAIN_ID=11155111
# Add a testnet-only EOA private key in .env when you run live writes.
# Do not paste a real key into README or commit it.
TEST_PRIVATE_KEY=
```

Optional:

```bash
DEMO_ADMIN_TOKEN=change-me
```

CAW credentials are read by the local `caw` CLI. Do not commit `.env`, private keys, API keys, or CAW credential files.

### Run the web demo

```bash
python3 -m uvicorn web.app:app --host 0.0.0.0 --port 8080 --timeout-keep-alive 600
```

Open:

```text
http://localhost:8080
```

### CLI examples

```bash
# List services from the registry
python run.py list

# Match a service for a natural-language task
python run.py procure "write a short ETH market analysis"

# Buyer Agent API call through the local FastAPI server
python run.py agent-buyer "write a short ETH market analysis" --budget 0.00003

# Seller Agent registration through the local FastAPI server
python run.py agent-seller "a research agent that writes concise ETH market summaries" \
  --seller 0xYourSellerAddress \
  --price 0.00002 \
  --endpoint http://localhost:8080/api/x402/request

# Direct x402 purchase for a service ID
python run.py request 1 "generate a research summary"

# Inspect recorded proofs
python run.py proofs

# Check system status
python run.py status
```

## Main API routes

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/services` | Discover active services |
| `POST` | `/api/procure/match` | Match services for a request without paying |
| `POST` | `/api/procure` | Match and purchase a service |
| `POST` | `/api/agent/buyer/procure` | Buyer Agent service discovery, ranking, budget checks, optional x402 payment |
| `POST` | `/api/agent/seller/register` | Seller Agent service registration, admin-gated when `DEMO_ADMIN_TOKEN` is set |
| `POST` | `/api/x402-buy` | Direct x402/CAW purchase for a selected service |
| `GET` | `/api/proofs` | Delivery proof records |
| `GET` | `/api/status` | Wallet, contract, service, and demo status |

Seller service management:

| Method | Path | Purpose |
| --- | --- | --- |
| `POST` | `/api/x402/register_v2` | Register a service |
| `GET` | `/api/x402/seller/services_v2` | List services owned by a seller |
| `POST` | `/api/x402/seller/update_v2` | Update seller-owned service metadata |
| `POST` | `/api/x402/seller/remove_v2` | Deactivate a seller-owned service |
| `GET` | `/api/x402/health` | Seller x402 health check |

## Testing

```bash
python3 -m py_compile web/app.py agent_commerce_sandbox/*.py
PYTHONPATH=. pytest -q
```

For the inline JavaScript in `web/index.html`:

```bash
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
class Parser(HTMLParser):
    def __init__(self):
        super().__init__(); self.scripts=[]; self.in_script=False; self.buf=[]
    def handle_starttag(self, tag, attrs):
        if tag == 'script': self.in_script=True; self.buf=[]
    def handle_data(self, data):
        if self.in_script: self.buf.append(data)
    def handle_endtag(self, tag):
        if tag == 'script' and self.in_script:
            self.scripts.append(''.join(self.buf)); self.in_script=False
p = Parser(); p.feed(Path('web/index.html').read_text())
Path('/tmp/agent-commerce-hub-inline.js').write_text('\n'.join(p.scripts))
print(f'extracted {len(p.scripts)} script block(s)')
PY
node --check /tmp/agent-commerce-hub-inline.js
```

## Security notes

- This is a hackathon demo running on Sepolia test assets.
- Do not use production private keys.
- Do not commit `.env`, CAW credential files, API keys, or RPC credentials.
- CAW Pact policy should be scoped to the minimum chain, token, destination, amount, and operation needed for the demo.
- Seller registration and demo admin operations should be gated with `DEMO_ADMIN_TOKEN` in any shared deployment.

## Hackathon context

- Event: AI x Web3 Agentic Builders Hackathon by Casual Hackathon
- Track: Cobo Agentic Wallet
- Direction: agent-native payments, resource procurement, and agent-to-agent commerce

## License

MIT
