# 9. Quick reference / cheat sheet

*Print this and pin it next to the wall.*

**Elements**

| Colour | Element | Phrasing |
|---|---|---|
| 🟡 Yellow `#FFE066` (small) | Trigger | noun/event — "Support ticket arrives" |
| 🔵 Blue `#4DABF7` | Task | imperative — "Classify ticket" |
| 🟠 Amber `#FFC078` (large) | Agent | role name — "Triage Agent" |
| 🟠 Orange `#FF922B` | Outcome / Event | past tense — "Ticket classified" |
| 🟣 Lilac `#B197FC` | Policy / Routing | "Whenever \<outcome\>, dispatch \<task\> to \<agent\>" |
| 🩷 Pink `#F783AC` | Tool / External system | "Stripe API" |
| 🟢 Green `#69DB7C` | Context / Read model | "Customer history" |
| 🟦 Teal `#3BC9DB` | Guardrail | "PII filter", "Max $0.50/run" |
| 🤍 Cream `#F8F9FA` (person) | Human-in-the-loop | "Lead reviews refund" |
| 🔴 Red `#FF6B6B` | Hotspot / Risk | free text + `?` |

**Connectors:** Flow (solid) · Handoff (dashed) · Tool call (dotted) · Reads (thin) · Milestone (vertical divider)
**Containers:** Swarm (violet frame, different agents) · Fleet (teal frame, N copies of one agent)

**Say it out loud:**
- When **{trigger}**, start **{task}**.
- **{task}** is dispatched to **{agent}**.
- **{agent}** produces **{outcome}**.
- Whenever **{outcome}**, dispatch **{task}** to **{agent}**.
- Before/after **{step}**, check **{guardrail}**; on failure → **{route}**.
- **{human}** reviews **{outcome}** and approves/rejects.
- **{agent}** calls **{tool}** to **{action}**.

If you can say the sentence, the sticky is good. If you can't, you've found a gap.
