# 6. How to run a session

## 6.1 Who to invite

The whole point is mixing perspectives. Aim for:

- **A facilitator** (keeps the timeline honest, asks the dumb questions, enforces the grammar gently).
- **Business / domain experts** — the people who actually understand the process being automated.
- **Process owners / operations** — who know the edge cases and where humans currently intervene.
- **Engineers / AI architects** — who know what agents and tools can realistically do.
- Keep it under ~8 active modellers per wall; split into parallel walls for larger groups.

## 6.2 Space and materials

- **Physical:** a long, unbroken wall (or several whiteboards in a row), a roll of paper, and sticky notes in the grammar colours. Print the §9 cheat sheet and pin it up.
- **Digital / remote:** an infinite canvas. The companion Agent Storming app is purpose-built for this (the grammar colours, the typed connectors, swarm/fleet frames, and the sentence-template validation are baked in), but any Miro-style tool with the legend pinned works for a first session.
- A visible, agreed **legend** is non-negotiable — the colours only help if everyone reads them the same way.

## 6.3 The flow of a session

1. **Frame the scope and pick a level (§5).** Write the trigger(s) for the process on the far left, the desired end state on the far right. Everything happens in between.
2. **Chaotic exploration — Outcomes first.** Just like Event Storming, start by throwing **Outcomes** (orange, past tense) onto the wall: everything the system *produces*. Don't worry about order yet. Embrace duplication and mess.
3. **Enforce the timeline.** Sort the Outcomes left→right into causal/temporal order. Collapse duplicates. Argue about ordering — the arguments are the value.
4. **Add the actors and work.** For each Outcome, place the **Agent** that produced it and the **Task** that was dispatched to it, and the **Trigger** that kicked the chain off. Speak the sentence templates as you go.
5. **Wire the orchestration.** Add **Policies** ("whenever this outcome, dispatch that task to that agent") to connect outcomes back into new tasks. This is where the system's logic becomes visible.
6. **Bring in tools and context.** Add **Tools** the agents call and **Context/Read models** they read. Draw the tool-call (dotted) and reads (thin) connectors.
7. **Add safety: guardrails and humans.** Walk the board and ask, at every risky step: *what could go wrong, what would it cost, who should be watching?* Add **Guardrails** and **Human-in-the-loop** checkpoints. A board with none on its risky actions is unfinished.
8. **Mark Hotspots ruthlessly.** Any disagreement, unknown, hallucination risk, or single point of failure gets a **red Hotspot**. Don't resolve them in the moment — capture them.
9. **Group into swarms and fleets.** Where different agents collaborate on a goal, draw a **Swarm** frame. Where you need N parallel copies of one agent, draw a **Fleet** with an instance count.
10. **Add milestones.** Drop vertical **phase dividers** at the pivotal moments (usually a human approval or major state change).
11. **Walk the flow to validate.** Trace the timeline left→right out loud using the sentence templates. Every sticky should connect; every risky action should have a guard or a human; every policy should have a triggering outcome. Dead ends and orphans become new Hotspots.

## 6.4 Timeboxing

A first Big-Picture session is typically **half a day**. A single Process-level pass is **2–4 hours**. Design-level work on one process can take a **full day** or recur over several sessions. Resist the urge to finish everything in one sitting — Hotspots are a perfectly good place to stop.

---
