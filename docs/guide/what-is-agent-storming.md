# 1. What is Agent Storming?

**Agent Storming is a collaborative modelling workshop for designing a business process that will be executed — wholly or partly — by AI agents.** You and your team map the process on a long left-to-right timeline using a grammar of coloured sticky notes: the *triggers* that start work, the *tasks* that get dispatched, the *agents* that do them, the *outcomes* they produce, the *tools* they call, the *guardrails* that keep them safe, and the *humans* who stay in the loop.

It is a direct descendant of **Event Storming**, Alberto Brandolini's technique for exploring a business domain with domain experts and developers around a wall of stickies. Agent Storming keeps the same muscle memory — same wall, same timeline, same colour-coded grammar, same "everyone models together" energy — but changes the subject. Where Event Storming models *what happens in a domain*, Agent Storming models *who (which agent) does what, when, and with what safeguards* in an automated, agentic process.

**For business readers:** think of it as a structured whiteboard session that turns a vague ambition ("let's automate support with AI") into a concrete, shared picture of how the work actually flows — including where it can go wrong and where a human must still sign off.

**For practitioners:** the finished board is not just a diagram, it is a **specification**. It names every agent's role, the tasks routed to it, the tools it calls, the context it reads, and the policies that wire outcomes back into new tasks. That specification can be reviewed for missing checkpoints and failure modes, and is intended to become the source of truth that a multi-agent implementation is built from.

## When to use it

- You are about to build (or have started building) a system where **multiple AI agents collaborate**, or a single agent orchestrates tools and humans.
- You need **business and engineering in the same room**, speaking the same language, before code is written.
- You want to surface **risks, handoffs, and human checkpoints early**, while they are cheap to change.

## When *not* to use it

- A single, simple prompt-in/answer-out feature with no orchestration, tools, or routing — that doesn't need a workshop.
- A pure software domain with no AI agents — use classic Event Storming instead.

---
