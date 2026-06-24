# Agent Commerce Hub vs AEP (1st Place) — Post-Mortem & Hackathon Playbook

> Our project: **Agent Commerce Hub** (Cobo track, did not place)
> Winning project: **AEP — Agent Escrow Protocol** (1st place, Cobo track)
> Repo: https://github.com/zane199109/AEP-Hackathon

---

## TL;DR — What We Got Right vs What We Missed

We built a working x402 + CAW payment flow with Buyer/Seller Agents. Solid engineering for 3 days. But judges don't judge engineering — they judge **narrative, depth, and presentation**. AEP won because they told a better story, went deeper on CAW, and treated the demo like a theater production, not a code review.

---

## 1. Head-to-Head Comparison

| Dimension | Agent Commerce Hub (Us) | AEP (Winner) | Gap |
| --- | --- | --- | --- |
| **Core problem** | "AI agents need to buy services" | "AI agents need escrow + quality verification so they don't get scammed" | They found a sharper, more urgent problem |
| **CAW usage depth** | Single server-paired wallet, Pact for transfer | **3 CAW wallets** (Buyer MPC paired, Provider auto-approve, Sub-Provider auto-approve), Pact lock + Recipe release, dual wallet modes | We used CAW as a payment tool; they used it as the protocol foundation |
| **Payment model** | x402: pay first, get result | **Escrow**: lock funds → verify quality → release or refund | Escrow is a fundamentally stronger narrative for "agent safety" |
| **Evaluation** | None — payment = delivery | **Dual-track**: Rule engine (hard veto) + LLM quality scoring | They added a whole verification layer we didn't attempt |
| **Multi-agent** | Buyer Agent + Seller Agent (2 parties) | **Buyer → Provider → Sub-Provider** (3 parties, subcontracting) | Their demo showed agent-to-agent delegation |
| **Reputation** | Not implemented | On-chain `AEPReputation.sol`, score 0-100, cross-task persistence | They closed the loop: good work → higher score → more jobs |
| **Smart contract** | ServiceRegistryV2 (registry + proofs) | `AEPBounty.sol` (state machine) + `AEPReputation.sol` (scores) | Two contracts, more complex state machine |
| **Contract verified** | ❌ Not on Etherscan | ✅ Verified on Sepolia Etherscan | Huge credibility gap for judges |
| **Backend language** | Python FastAPI | **Go** (Chi, Zap, PGX, go-redis) | Go signals production intent to judges |
| **Database** | In-memory / JSON files | **PostgreSQL + Redis** (Docker Compose) | Real persistence, distributed locks |
| **Frontend** | Vanilla HTML + CSS + JS | **React + Vite + MUI + ReactFlow** | Topology visualization is memorable |
| **Real-time** | Polling (fetch/refresh) | **SSE push** from Go backend | Live topology pulses look fantastic to judges |
| **Docker** | ❌ No | ✅ `docker-compose.yml` with health checks | Judges can spin it up themselves |
| **Offline fallback** | ❌ No | ✅ Replay mode from real `cast`-captured events | If WiFi dies, demo still works |
| **Documentation** | README (originally Chinese) | README + PRD + design.md + demo_presentation.md + VERIFICATION_REPORT.md + screenshots | 5× more docs, all judge-facing |
| **Demo script** | "Click Buy and see what happens" | 8-10 minute rehearsed script with timing, exact words, what to show, what to say | Their demo was a show; ours was a walkthrough |
| **Demo video** | Yes (we added this late) | Yes (built into the flow) | - |
| **Code structure** | Flat-ish Python modules | Clean Go packages: `internal/config`, `internal/engine`, `internal/listener`, `internal/relayer`, `internal/provider`, `internal/store` | More professional repo structure |
| **C4 architecture diagrams** | Text flowchart in README | **4 Mermaid C4 diagrams** (Context, Container, Component, Code) | Judges love diagrams |

---

## 2. The Three Things That Lost Us the Competition

### 2.1 We built a marketplace. They built a protocol.

A marketplace ("list services, pick one, pay") is a known pattern. Judges see 10 marketplace demos per hackathon.

A protocol ("lock funds in escrow, verify quality with dual-track AI, release or refund, update reputation on-chain") is a **novel contribution**. It says "I invented something," not "I connected existing things."

**The difference:** x402 payment is a feature. Escrow + evaluation + reputation is a system.

### 2.2 We used CAW as a payment tool. They used CAW as the protocol foundation.

In our project, CAW does one thing: transfer SETH from server wallet to seller. It's a payment plugin.

In AEP, CAW is the backbone:
- **Pact = escrow vault** (funds locked but still in buyer's control)
- **Recipe = conditional settlement** (release on verification pass, refund on fail)
- **Dual wallet modes** (MPC paired for human-in-the-loop Buyer, custodial auto-approve for autonomous Provider/Sub-Provider agents)
- **3 separate wallets** interacting in a single demo flow

This directly aligns with Cobo's own positioning of CAW. The judges (likely from Cobo) saw their own product used the way it was designed to be used.

**Lesson:** When you compete on a platform sponsor's track, use their product the way *they* envision it, not the simplest way that works.

### 2.3 We treated the demo as a live test. They treated it as a theater production.

Their demo:
- **8-10 minute rehearsed script** with exact timing per step
- Pre-written narration: what to say, what the audience sees, what to point at
- **Visual pulses** (🔵 blue = funds locked, 🟢 green = settled, 🔴 red = refunded)
- **Topology visualization** with ReactFlow showing agent nodes lighting up
- **SSE real-time push** — no "refresh the page to see the update"
- **Both success AND failure paths** demonstrated ("here's what happens when the agent delivers garbage")
- **Offline fallback** — if RPC dies, replay mode from real events

Our demo:
- "Here's the page. Click Find & Pay. Wait for CAW approval. It works."
- No failure path demonstrated
- No visual storytelling
- Relied on the user refreshing or checking the console

---

## 3. What They Did Better — By Dimension

### 3.1 Problem Framing (Narrative)

| Us | Them |
| --- | --- |
| "Agents need to discover and pay for services" | "Agents need **trust** — escrow, verification, and reputation — to do real commerce" |
| Problem feels like a convenience | Problem feels like an existential requirement |

**Lesson:** Frame your problem as "without this, agents can't function." Not "this makes agents more convenient."

### 3.2 CAW Integration Depth

| Us | Them |
| --- | --- |
| Single wallet, single Pact, single transfer | 3 wallets, Pact lock/unlock lifecycle, Recipe-based settlement, dual MPC modes |
| CAW is called from a Python `subprocess` wrapper | CAW is called from a Go SDK (`cobo-go-api`) |

**Lesson:** Know the platform sponsor's full API surface and use the non-obvious parts. Everyone does "create Pact + transfer." Almost nobody does "Recipe-based conditional settlement" or "dual wallet modes."

### 3.3 Documentation (Judge-Facing)

Their docs folder:
```
docs/
  prd.md                    — Full product requirements (27 pages of Chinese text)
  design.md                 — Architecture with C4 diagrams
  demo_presentation.md      — Word-for-word demo script with timings
  cobo_setup.md             — CAW-specific setup guide
  setup_checklist.md        — Pre-demo environment checklist
  start.md                  — Getting started
  project_description.md    — Project overview
```

Our docs:
```
README.md                   — (was originally in Chinese, now translated)
AGENT_COMMERCE_HUB_PLAN.md  — Internal planning notes, not judge-facing
```

**Lesson:** Judges read your docs before they see your demo. If your docs look like internal notes, they assume the project is unfinished. If your docs look like a product launch, they assume it's real.

### 3.4 Production Polish

| Area | Us | Them |
| --- | --- | --- |
| Deployment | Manual `uvicorn` start | `docker compose up -d` |
| Database | None (in-memory) | PostgreSQL + Redis with health checks |
| Logging | `print()` statements | Structured Zap logging with TraceID propagation |
| Configuration | `.env` file parsing | YAML config + Viper |
| Contract | Not verified | Verified on Etherscan ✅ |
| Build system | No Makefile | Makefile with `build-contract`, `test-contract`, `build-backend`, `verify-day0` |

**Lesson:** Hackathon judges look for "would I believe this could go to production?" signals. Docker, verified contracts, structured logging, and Makefiles are those signals.

### 3.5 The "Wow" Moment

Every winning hackathon project has a moment where judges think "I didn't expect *that*."

AEP had at least three:

1. **Sub-Provider delegation**: "Provider realized the task needed a sub-task and automatically hired a Sub-Provider" — this is agent-to-agent commerce, not just agent-to-human
2. **Failure path demonstrated**: "Watch what happens when the agent delivers garbage — 🔴 red pulse, funds refunded, reputation slashed"
3. **Dual wallet mode contrast**: "Buyer needs human approval on CAW App. Provider auto-approves because it's an unpaired MPC wallet. Two modes, one protocol."

Our project's "wow" was: "It works end-to-end with real CAW payments." That's table stakes, not a wow.

---

## 4. Hackathon Playbook — How To Win Next Time

### Rule 1: Solve a Scarier Problem

Don't solve "agents need to pay for things." Solve "agents will get scammed without trust infrastructure." The scarier the problem, the more your solution feels necessary, not optional.

### Rule 2: Know the Sponsor's Product Better Than They Do

Cobo's CAW has Pact, Recipe, dual wallet modes (MPC paired vs custodial auto-approve), TSS nodes, and an official Go SDK. Most teams used 20% of this. AEP used 80%. The judges work at Cobo. They noticed.

**Action:** Before the next hackathon, read the sponsor's entire API documentation. Find the features 90% of teams won't touch. Build your demo around those.

### Rule 3: Docs Are Part of the Product

Allocate 30% of hackathon time to documentation:
- PRD (what problem, why it matters, how it works)
- Architecture diagrams (C4 style, Mermaid)
- Demo script (word-for-word, timed)
- Verification report (contract addresses, tx hashes, screenshots)
- Setup guide (so judges can run it themselves)

If a judge can understand your project without you in the room, you're winning before the demo starts.

### Rule 4: The Demo Is a Show, Not a Test

- Write a script. Rehearse it. Time each section.
- Demonstrate **both success AND failure**. Failure paths are more memorable.
- Use visual storytelling: color-coded pulses, topology diagrams, real-time updates.
- Have a **pre-recorded backup video** and an **offline fallback mode**.
- Start with: "Here is the problem. Here is why existing solutions fail. Here is how we solve it."

### Rule 5: Polish Signals Production Intent

Small things that signal "this could be real":
- ✅ Docker Compose (one-command startup)
- ✅ Contract verified on Etherscan
- ✅ Structured logging (not `print()`)
- ✅ Makefile or Taskfile
- ✅ Database with migrations (not in-memory JSON)
- ✅ Configuration via YAML/env (not hardcoded)
- ✅ Health check endpoints
- ✅ Clean package structure (not one flat folder)

### Rule 6: Have One "They Didn't Expect That" Moment

Your project needs one thing that makes a judge lean forward. For AEP it was:
- Sub-contracting: an agent autonomously hires another agent
- Dual-track evaluation with rule engine veto power
- The contrast between "human-approved" and "auto-approved" wallet modes

For your next project, ask: "What will make the judge say 'wait, show me that again'?"

### Rule 7: Align With Judging Criteria Explicitly

AEP's PRD has a section titled **"场景贴合度" (Scenario Fit)** that explicitly maps every feature to the judging rubric. They didn't hope judges would notice — they spelled it out.

**Action:** Add a section to your README/PRD: "How we align with the judging criteria." Put the rubric items in a table. Fill in your evidence.

---

## 5. What We Did Right (Don't Throw This Away)

Not everything was a miss. We did several things well:

- **Real CAW payments on Sepolia** with actual tx hashes and proof records. Many teams only mocked this.
- **Buyer Agent budget enforcement** — the `max_price_wei` guard against price changes between selection and payment. This is a real safety consideration that AEP would appreciate.
- **CLI + Web UI dual interface** — agencies can use the CLI, humans can use the UI.
- **Python + FastAPI** is the right stack for rapid prototyping. Don't switch to Go just because AEP used Go. But do add Docker + proper persistence.
- **Buyer/Seller/Agent Console three-role UI** — this was the right idea. We just didn't give it the visual "wow" AEP did with ReactFlow topology.

---

## 6. If We Had 3 More Days — What Would Have Changed

Here's what would have closed the gap, ranked by impact:

1. **Demo script + pre-recorded video** (2 hours) — biggest ROI. A rehearsed 5-minute demo with narration beats a 15-minute live walkthrough.
2. **Contract verified on Etherscan** (30 min) — instant credibility boost.
3. **React frontend with topology visualization** (1 day) — one Mermaid diagram on a static page is not a "dashboard."
4. **Dual-track evaluation** (1 day) — add a rule engine that checks delivery quality, not just "did payment succeed." This turns "payment marketplace" into "escrow protocol."
5. **Offline replay mode** (2 hours) — if WiFi dies, replay from recorded events.

---

## 7. TL;DR — The One Sentence Summary

**AEP won because they solved a scarier problem (agent trust, not agent payments), went deeper on CAW (Pact escrow + Recipe settlement, not just transfer), and treated the demo like a product launch (diagrams, script, screenshots, verified contracts, Docker), not a code walkthrough.**
