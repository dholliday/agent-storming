# 8. Implementing Agent Storming in your business

## 8.1 Where it fits in delivery

Agent Storming sits in the **discovery → design** seam, just before build:

1. **Discovery** — a Big-Picture session maps the candidate automated business line and its biggest unknowns.
2. **Design** — Process and then Design-level sessions on the specific flow you've chosen to build first.
3. **Build** — engineering implements from the Design-level board, which stays pinned as the living spec.

## 8.2 Picking your first process

Choose a first candidate that is **valuable, bounded, and tolerant of iteration**:

- High enough volume that automation matters.
- A clear trigger and a clear "done".
- Existing human judgement you can point at (so you know where the human checkpoints belong).
- Low blast radius if the agents get it wrong early — *avoid* irreversible, high-stakes actions for your very first build.

Support triage, lead qualification, document intake, and internal-IT request routing are common, friendly starting points.

## 8.3 Making it a practice, not a one-off

- Run a session **before** each significant agentic build, and **re-storm** when the process changes materially.
- Keep the board as **living documentation**. A board that drifts from reality is worse than none — assign an owner.
- Build a small **library of patterns** your org reuses: a standard "human approves anything that spends money" checkpoint, a standard PII guardrail, a standard "route to human on low confidence" policy.

## 8.4 Handing off to engineering

A Design-level board is the engineering brief: every agent's role/model/tools/memory, every policy, every guardrail, every human checkpoint is on it. Engineers should be *in* the session, so the handoff is a continuation, not a translation.

The direction of travel — and the reason the grammar is kept precise — is that a Design-level board becomes **machine-readable** enough to (a) be **critiqued by AI** for missing checkpoints, unguarded tool calls, single points of failure, and orphaned policies, and (b) eventually be **compiled into runnable agent scaffolding**. Treat that as where the method is heading, not a feature to depend on today; the design value stands on its own regardless.

## 8.5 Common pitfalls

- **Over-detailing too early.** Jumping to Design-level before the Big Picture is agreed. Storm broad first.
- **Skipping guardrails and human checkpoints.** The most dangerous board is the one that looks complete but has an agent spending money or contacting customers with nobody watching. Always do the "what could go wrong, who's watching?" pass (§6.3 step 7).
- **Enforcing the grammar too rigidly.** The grammar is a thinking aid, not a compliance exercise. Surface problems as Hotspots and warnings; don't let notation policing kill the conversation.
- **No room mix.** Business-only sessions miss what's buildable; engineering-only sessions miss what the process actually is. Mix the room.
- **Resolving every Hotspot live.** Capture them and move on; a session that stalls on one argument loses the wall.

---
