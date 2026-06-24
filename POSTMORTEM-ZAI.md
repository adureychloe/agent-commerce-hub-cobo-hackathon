# ZAI Track Winners — Post-Mortem Comparison & Hackathon Patterns

> Our project: **Agent Commerce Hub** (Cobo track)
> ZAI track winners we studied:
> - 🥇 **Digital Pompeii** — [repo](https://github.com/10yu7ian/digital-pompeii) · 链上灾难博物馆 / AI forensic agent
> - 🥇🥈 **Daedalus** — [repo](https://github.com/XuetaoZhang/Daedalus) · Agentic 3D world-building platform
> - 🥇🥈 **Agent Rush** — [repo](https://github.com/chxii/agent-rush) · MEV roguelike strategy card game
>
> All three were from the **ZAI track** (Long-Horizon Task). Our project was Cobo track. Different judging criteria,
> but the same hackathon — so the patterns of what wins are transferable.

---

## 1. What The Winners Built — In One Sentence

| Project | Track | One-sentence summary |
| --- | --- | --- |
| **Digital Pompeii** | ZAI · Long-Horizon | An autonomous AI forensic agent that investigates real on-chain disasters — 6-step self-correcting investigation loop, 9 read-only tools, 5 cases totaling $1.357B in losses. |
| **Daedalus** | ZAI · Long-Horizon | A platform that turns natural-language briefs into browser-rendered 3D worlds — plan → validate → repair → export workflow, 19 controllable landmark models, hex-tile terrain engine. |
| **Agent Rush** | ZAI · Long-Horizon | A roguelike MEV strategy game where an LLM-driven Executor agent plans, calls on-chain tools, and re-plans in real time when rival bots front-run you — 20-layer progression, 13 test files, batch-simulated balance. |

---

## 2. Head-to-Head Comparison — Six Dimensions

### Dimension 1: Problem Framing

| Project | Problem | Why it's compelling |
| --- | --- | --- |
| **Digital Pompeii** | $1.357B in crypto disasters nobody can easily explain. Chain data is public but unreadable. Same failure patterns repeat. | Solves a real, expensive, recurring problem. The "public good" framing makes it feel noble, not just clever. |
| **Daedalus** | Web3 projects need immersive spaces, but 3D world-building is slow and requires specialists. | Turns a 2-week design task into a 30-second prompt. Clear "before → after" value proposition. |
| **Agent Rush** | Long-horizon agents are black boxes — users can't see *how* they think. MEV is the perfect adversarial testbed. | Uses a game to solve an AI explainability problem. "Play to understand" is more fun than "read a whitepaper." |
| **Agent Commerce Hub (us)** | Agents need to discover and pay for services. | Honest, but not urgent. "Convenience" problems rarely win hackathons. |

**Pattern:** Winners solve problems where the alternative is *real pain* (money lost, time wasted, knowledge hidden). We solved a convenience problem.

### Dimension 2: AI Usage Depth

| Project | AI Model | How AI is used | Why it's impressive |
| --- | --- | --- | --- |
| **Digital Pompeii** | GLM-5.1 | 6-step self-correcting investigation loop. Agent autonomously chooses which tool to call next, forms hypotheses, seeks counter-evidence, and only concludes when evidence is sufficient. 6-12 tool calls per case. | Not a wrapper. Not a chatbot. A genuinely autonomous agent that *investigates* — judges can see the intermediate reasoning. |
| **Daedalus** | GLM-5.1 (configurable) | Converts natural language into structured scene specs (Zod-validated JSON), not arbitrary code. Plan → validate → repair loop. | Controlled output schema prevents the LLM from hallucinating broken 3D scenes. The agent *plans*, the renderer *executes*. |
| **Agent Rush** | GLM-5.1 | Executor agent: plan → tool call → re-plan loop. 7 tools, JSON Schema constrained. SSE streaming of reasoning token-by-token. Falls back to RuleDecider on API failure. | LLM is used in exactly ONE place — the execution loop — and the game explicitly labels everything else as rule-engine. This discipline impresses judges. |
| **Agent Commerce Hub (us)** | DeepSeek (optional) | Keyword or LLM-based service matching. Single API call, not an agent loop. | We used LLM as a matching engine. They used LLM as an autonomous agent. The difference is "one call" vs "a 6-to-12-step investigation." |

**Pattern:** Winners used AI for *long-horizon autonomous loops* — the AI decides what to do next, calls tools, observes results, and iterates. We used AI for single-step classification.

### Dimension 3: The "Wow" Demonstrated

| Project | The "wow" moment |
| --- | --- |
| **Digital Pompeii** | "Watch the AI autopsy The DAO hack — it found the reentrancy bug, traced the fatal transaction, and wrote a literary epitaph. It did this *autonomously*, without a human telling it where to look. And it cross-verified its own conclusions." |
| **Daedalus** | "Type 'create a coastal world with a castle, windmill, and dock' — and watch the agent plan the terrain, place landmarks, validate the scene, and render it in 3D. The agent's thought process is visible in the workflow panel." |
| **Agent Rush** | "An enemy bot front-ran your arbitrage. Watch the Executor agent realize this in real time, abandon the stolen card, reallocate Gas, and try a different strategy — all while its reasoning streams live on screen." |
| **Agent Commerce Hub (us)** | "Click Buy, approve in CAW app, get a result." |

**Pattern:** Every winner's "wow" involves the agent *thinking visibly*. The user watches the AI struggle, adapt, and succeed — or fail gracefully. Our "wow" was the *result*, not the *process*.

### Dimension 4: Documentation & Polish

| Element | Digital Pompeii | Daedalus | Agent Rush | Us |
| --- | --- | --- | --- | --- |
| **README** | Bilingual, 10 sections, problem → architecture → cases → roadmap | Bilingual, product-surface framing, 19 landmark models with previews | Bilingual, problem → why AI → why Web3 → how it works → validation | English (translated late) |
| **PRD / Design doc** | FACT_CHECK.md (external verification of all 5 cases) | docs/PRD.md, docs/TerrainSystem.md | docs/GDD (Game Design Document), docs/slides.html | AGENT_COMMERCE_HUB_PLAN.md (internal notes) |
| **Demo video** | ✅ Google Drive | ✅ Live demo URL | ✅ Google Drive | ✅ YouTube (added late) |
| **Demo script** | ✅ DEMO_SCRIPT.md | Implicit in README workflow | Implicit in GDD | ❌ |
| **Live demo URL** | ✅ Vercel | ✅ Vercel | ✅ Vercel | ✅ ngrok tunnel |
| **On-chain proof** | OpenTimestamps (Bitcoin-anchored) | N/A (not a chain project) | N/A (pure simulation) | Sepolia tx hashes |
| **Contract verified** | N/A (read-only chain data) | N/A | N/A | ❌ |
| **Tests** | Not visible in repo | Not visible in repo | 13 test files, batch sim with target clear rates | 13 pytest tests |

**Pattern:** Winners document like they're launching a product, not finishing a homework assignment. Bilingual READMEs, external fact-checking, game design documents, demo scripts — these signal "we thought about the audience, not just the code."

### Dimension 5: Technical Architecture Discipline

| Project | Architecture choice | Why it matters |
| --- | --- | --- |
| **Digital Pompeii** | Agent loop with 9 read-only tools, no wallet, no chain writes. OpenTimestamps for proof anchoring. | Clear security boundary: "we only read, never write." Reduces judge anxiety about rug pulls. |
| **Daedalus** | LLM outputs Zod-validated JSON → renderer turns it into 3D. LLM never generates code, only data. | Prevents LLM hallucination from producing broken scenes. Schema as guardrail. |
| **Agent Rush** | LLM used in exactly 1 place (Executor loop). Everything else is deterministic rule engine. Auto-fallback to RuleDecider on API failure. JSON Schema + ajv validation. | "We know exactly where AI stops and rules begin." Judges respect this honesty more than "AI everywhere." |
| **Agent Commerce Hub (us)** | FastAPI monolith, CAW CLI subprocess wrapper, in-memory state. | Functional but not architecturally intentional. No clear separation between "what the agent decides" and "what the system enforces." |

**Pattern:** Winners draw a bright line between "where AI makes decisions" and "where deterministic rules enforce safety." This is a trust signal. Judges at Web3 hackathons are especially sensitive to "what if the AI goes rogue?"

### Dimension 6: The "Beyond Hackathon" Vision

| Project | How they signal this is more than a demo |
| --- | --- |
| **Digital Pompeii** | Full commercial roadmap: Anti-Sudden-Death Index → Failure Archive (data flywheel) → risk API → wallet pre-sign warnings → Launchpad screening. Cites specific business models and target customers. |
| **Daedalus** | Platform framing (not single demo page): Home page for brand, Studio page for creation. Mentions product showcases, community spaces, virtual demo environments as use cases. |
| **Agent Rush** | "This can become a semi-real sandbox with real-time chain data, or the Executor loop can be extracted as a standalone long-horizon agent visualization shell." Cites specific future directions. |
| **Agent Commerce Hub (us)** | Agent Console UI with three roles. But no roadmap, no business model, no "what this becomes after the hackathon." |

**Pattern:** Winners make judges believe the project will continue after the hackathon. They have a concrete "next step" that isn't just "add more features."

---

## 3. The Three Meta-Patterns Across All Winners

### Pattern A: **Show the agent thinking, not just the result.**

Every winning project spent significant engineering effort on *visualizing the agent's internal process*:

- Digital Pompeii: 6-step investigation loop visible, tool call logs in `runs/`, cross-validation step explicit
- Daedalus: Workflow panel showing plan → validate → repair states
- Agent Rush: SSE streaming of reasoning token-by-token, thought chain panel

**Our project:** The Buyer Agent does match, filter, and buy — but the UI shows "matched service X, paying..." with no visibility into *why* it chose X over Y, or what the budget constraint looked like.

**Lesson:** Build a "thinking panel." Show the agent's intermediate steps, not just the final output. This is especially important for the ZAI "Long-Horizon Task" track — if judges can't see the long horizon, it doesn't count.

### Pattern B: **Treat failure as a feature, not a bug to hide.**

- Digital Pompeii: "Evidence insufficient → labeled '存疑' (uncertain), NOT fabricated." External FACT_CHECK.md documents every correction.
- Agent Rush: "GLM timed out → auto-fallback to RuleDecider, UI shows [自动保底]. Game continues." Three fallback paths tested and verified.
- Daedalus: Validation step in the workflow — if the scene spec is invalid, the agent repairs it.

**Our project:** We only demonstrated the success path. The failure path ("CAW approval rejected") was handled in code but never shown in the demo.

**Lesson:** Demonstrating graceful failure is more impressive than demonstrating perfect success. It shows you thought about edge cases.

### Pattern C: **The project is a product, not a codebase.**

- Agent Rush: 20-layer progression, 4 player strategies, batch-simulated clear rates (8%-82%), role buffs, boss layers. This is a *game*, not a demo of a game.
- Daedalus: Home page + Studio page. Brand, metrics, entry points. This is a *platform*, not a demo of a platform.
- Digital Pompeii: 5 cases, external fact-checking, OpenTimestamps anchoring, commercial roadmap with specific business models. This is a *product*, not a demo of a product.

**Our project:** A single-page app with three tabs. Functional, but not product-shaped.

**Lesson:** A hackathon project should feel complete *as a product*, not as a technical proof-of-concept. Have a landing page. Have content. Have a roadmap. Make the judge think "I would use this."

---

## 4. What We Did Right (Relative to ZAI Winners)

The ZAI track winners are a different category (Long-Horizon Task AI with GLM, no on-chain payments), but some of our strengths are worth noting:

- **Real on-chain integration**: We had actual Sepolia transactions, CAW Pact approvals, and on-chain proof records. Digital Pompeii reads chain data but never writes. Agent Rush and Daedalus don't touch chain at all. In a Web3 hackathon, actually transacting on-chain is a differentiator — if you *show* it.
- **Multi-interface**: CLI + Web UI is more than most teams build. Agent Rush is pure web. Daedalus is web-only. Digital Pompeii has a Python agent but no CLI.
- **Budget safety**: Our `max_price_wei` guard against price changes between selection and payment is a genuine security consideration. Most projects don't think about TOCTOU in agent payments.

---

## 5. The Combined Hackathon Playbook

Merging lessons from both Cobo track (AEP) and ZAI track (Digital Pompeii, Daedalus, Agent Rush):

### 7 Rules That Predict Winners

1. **Solve a problem where the alternative is pain, not inconvenience.**
   - ✅ "Agents will get scammed without trust infrastructure" (AEP)
   - ✅ "$1.357B in crypto disasters nobody can explain" (Digital Pompeii)
   - ✅ "3D world-building takes weeks without specialists" (Daedalus)
   - ❌ "Agents need a more convenient way to pay" (us)

2. **Show the agent thinking. The process IS the product.**
   - Agent loops, tool-call traces, thought chains, streaming reasoning.
   - If the judge can't see *how* your agent works, it doesn't count as agentic.

3. **Use AI in exactly the right place, not everywhere.**
   - Agent Rush: one LLM touchpoint. Everything else is rules. Explicitly labeled.
   - AEP: rule engine has veto power over LLM evaluation.
   - Digital Pompeii: LLM investigates, but every conclusion must cite on-chain evidence.

4. **The demo is a show. Rehearse it.**
   - Digital Pompeii has DEMO_SCRIPT.md. AEP has an 8-minute timed script with exact narration. Agent Rush has a 20-layer guided progression with teaching layers 1-3.

5. **Document like you're launching a product.**
   - Bilingual README. Architecture diagrams. PRD. Verification report. Fact-check document. Game design document. Demo script. Screenshots.
   - Judges read docs before demos. If docs look like internal notes, the project looks unfinished.

6. **Have a "beyond hackathon" story.**
   - Digital Pompeii: Failure Archive → risk API → wallet warnings. Specific business models.
   - AEP: "This protocol can underpin all agent-to-agent commerce."
   - Daedalus: "Platform for immersive spaces, not a single demo page."

7. **Make failure look intentional.**
   - Demonstrate what happens when the agent fails. Show fallback paths. Prove you thought about edge cases.
   - "It works when everything goes right" is table stakes. "It handles when things go wrong" wins.

### The Meta-Rule

**Judges don't judge your code. They judge their experience of your project.**

Every winning project optimized for the judge's experience:
- Can I understand the problem in 30 seconds? (clear README, strong problem framing)
- Can I see the agent working? (visible agent loops, streaming reasoning, thought chains)
- Do I believe this could be real? (product shape, roadmap, verified contracts, Docker, tests)
- Did anything surprise me? (failure paths, self-correction, agent-to-agent delegation, literary epitaphs for dead contracts)
- Would I remember this tomorrow? (visual storytelling, memorable demos, unique angles)

---

## 6. If We Had 3 More Days — What Would Have Closed The Gap

Ranked by impact:

1. **Agent thinking panel** (1 day) — Show the Buyer Agent's matching process, budget filtering, and candidate ranking step-by-step in the UI. SSE streaming of agent reasoning. This alone would transform the demo from "it works" to "look how it thinks."

2. **Demonstrate graceful failure** (2 hours) — Show what happens when a CAW Pact is rejected. Show what happens when budget is insufficient. Show what happens when no services match. Failure paths are more memorable than success paths.

3. **Demo script with narration** (1 hour) — Write a 5-minute script: "I'm going to show you three things. First, how a Seller Agent publishes a service. Second, how a Buyer Agent discovers and evaluates it. Third, what happens when the agent faces a budget constraint." Timed, rehearsed, practiced.

4. **Product-shaped UI** (1 day) — Landing page with problem statement, metrics, and entry points. Not just a dashboard.

5. **Beyond-hackathon roadmap** (1 hour) — "This becomes a multi-buyer CAW marketplace. Here's the architecture for that. Here are the three integrations that make it real."
