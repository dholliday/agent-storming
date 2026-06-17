# 2. Why it exists

Multi-agent systems are deceptively hard to reason about. The failure modes are different from ordinary software:

- **Non-deterministic actors.** An agent might classify the same ticket two different ways. The design has to tolerate that.
- **Invisible handoffs.** Agent A delegates to Agent B, which calls a tool, which hands back to a third agent. Drawn out, this is obvious; in a codebase or a Slack thread it is not.
- **Missing human checkpoints.** It is easy to let an agent issue a refund, send an email, or change a record with no one watching — until it does something expensive or wrong.
- **Unguarded tool calls and runaway cost.** Agents that call APIs in loops, retry forever, or burn budget without a cap.
- **Single points of failure.** One orchestrator everything depends on; one classification step the whole flow trusts blindly.
- **Orphaned logic.** Policies that never fire, outcomes nothing reacts to, tasks dispatched to no clearly-owned agent.

Agent Storming exists to make all of this **visible and discussable before you write code.** When the whole process is on one wall, a room of people can see the gaps: "Wait — what happens if the classifier is wrong?" "Who approves a refund over £100?" "This API call has no retry." These conversations are nearly free at the wall and very expensive in production.

The deeper bet is that **the board is a specification, not a sketch.** Because the grammar is precise (every sticky has a type and a phrasing convention) the board can be read by a person *or* a machine. That opens the door to AI-assisted critique ("you have an unguarded tool call here") and, eventually, to compiling a finished design-level board into runnable agent scaffolding. The method works perfectly well as a pure design tool today; that machine-readability is the upside that makes the discipline worth keeping.

---
