# 4. The grammar (notation reference)

The grammar is the heart of the method. Keep it visible in the room.

## 4.1 Elements (sticky notes)

| Element | Colour | Meaning | How to phrase it |
|---|---|---|---|
| **Trigger** | Yellow `#FFE066` (small) | What starts a flow: human request, schedule/cron, inbound webhook/event | noun/event — "Support ticket arrives", "Nightly 02:00" |
| **Task** | Blue `#4DABF7` | A unit of work dispatched to an agent | imperative — "Classify ticket", "Draft reply" |
| **Agent** | Amber `#FFC078` (large, bold border) | The actor that receives tasks and produces outcomes; carries role, model, tools, memory | role name — "Triage Agent" |
| **Outcome / Event** | Orange `#FF922B` | What an agent produced | past tense — "Ticket classified as billing" |
| **Policy / Routing rule** | Lilac `#B197FC` | Reactive orchestration: routing, fan-out, retries, escalation | "Whenever \<outcome\>, dispatch \<task\> to \<agent\>" |
| **Tool / External system** | Pink `#F783AC` | A deterministic capability an agent calls | "Stripe API", "Vector store" |
| **Context / Read model** | Green `#69DB7C` | Information an agent reads to decide/act | "Customer history" |
| **Guardrail** | Teal `#3BC9DB` | Validation/safety/budget gate before or after a step | "PII filter", "Max $0.50/run" |
| **Human-in-the-loop** | Cream `#F8F9FA` (dark border, person glyph) | Explicit human checkpoint: review/approve/correct/escalate | "Agent reviews refund" |
| **Hotspot / Risk** | Red `#FF6B6B` | Open question, hallucination risk, single point of failure, "we need to decide" | free text + `?` |

## 4.2 Containers (regions, not stickies)

- **Individual agent** — a single Agent sticky.
- **Swarm** — a translucent **violet** frame grouping *different* agents collaborating toward one goal (orchestrator + workers, dynamic handoffs).
- **Fleet** — a translucent **teal** frame denoting *N parallel instances of the same agent*. Drawn as a stacked card with an instance-count badge.

## 4.3 Connectors

- **Flow** (solid line) — sequence/causality along the timeline: Trigger → Task → Agent → Outcome → Policy → Task…
- **Handoff** (dashed) — agent-to-agent delegation inside a swarm.
- **Tool call** (dotted) — Agent → Tool (and back).
- **Reads** (thin) — Context/Read model → Agent (informs a decision).

## 4.4 Pivotal markers

- **Milestone / phase divider** — a vertical line splitting the timeline into stages (Event Storming's "pivotal event"). Usually coincides with a human approval or a major state change.

## 4.5 Sentence templates (the grammar that checks itself)

If you can fill in the blank, the sticky is well-formed and connected. If you can't, you have a gap. Use these out loud while placing notes:

- **Trigger** → "When **{trigger}**, start **{task}**."
- **Task → Agent** → "**{task}** is dispatched to **{agent}**."
- **Agent → Outcome** → "**{agent}** produces **{outcome}**."
- **Policy** → "Whenever **{outcome}**, dispatch **{task}** to **{agent}**."
- **Guardrail** → "Before/after **{step}**, check **{guardrail}**; on failure → **{route}**."
- **Human** → "**{human}** reviews **{outcome}** and approves/rejects."
- **Tool** → "**{agent}** calls **{tool}** to **{action}**."

A board reads correctly when you can walk it left to right, speaking these sentences in order, with no dead ends.

---
