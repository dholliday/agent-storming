# Agent Storming

*A workshop method for designing multi-agent AI systems — by reinterpreting Event Storming for a world where the actors are AI agents.*

> This document describes the **method**. It is the canonical written reference for facilitators, architects, and the business stakeholders who join an Agent Storming session. A companion web app (a Miro-style collaborative canvas) is being built to support it; its specification lives in [`AGENT_STORMING_APP_CLAUDE.md`](./AGENT_STORMING_APP_CLAUDE.md). You do not need the app to run a session — a wall and sticky notes work fine.

---

## Table of contents

1. [What is Agent Storming?](#1-what-is-agent-storming)
2. [Why it exists](#2-why-it-exists)
3. [Agent Storming vs Event Storming](#3-agent-storming-vs-event-storming)
4. [The grammar (notation reference)](#4-the-grammar-notation-reference)
5. [The three levels](#5-the-three-levels)
6. [How to run a session](#6-how-to-run-a-session)
7. [Worked example: AI support-ticket triage](#7-worked-example-ai-support-ticket-triage)
8. [Implementing Agent Storming in your business](#8-implementing-agent-storming-in-your-business)
9. [Quick reference / cheat sheet](#9-quick-reference--cheat-sheet)

---

## 1. What is Agent Storming?

**Agent Storming is a collaborative modelling workshop for designing a business process that will be executed — wholly or partly — by AI agents.** You and your team map the process on a long left-to-right timeline using a grammar of coloured sticky notes: the *triggers* that start work, the *tasks* that get dispatched, the *agents* that do them, the *outcomes* they produce, the *tools* they call, the *guardrails* that keep them safe, and the *humans* who stay in the loop.

It is a direct descendant of **Event Storming**, Alberto Brandolini's technique for exploring a business domain with domain experts and developers around a wall of stickies. Agent Storming keeps the same muscle memory — same wall, same timeline, same colour-coded grammar, same "everyone models together" energy — but changes the subject. Where Event Storming models *what happens in a domain*, Agent Storming models *who (which agent) does what, when, and with what safeguards* in an automated, agentic process.

**For business readers:** think of it as a structured whiteboard session that turns a vague ambition ("let's automate support with AI") into a concrete, shared picture of how the work actually flows — including where it can go wrong and where a human must still sign off.

**For practitioners:** the finished board is not just a diagram, it is a **specification**. It names every agent's role, the tasks routed to it, the tools it calls, the context it reads, and the policies that wire outcomes back into new tasks. That specification can be reviewed for missing checkpoints and failure modes, and is intended to become the source of truth that a multi-agent implementation is built from.

### When to use it

- You are about to build (or have started building) a system where **multiple AI agents collaborate**, or a single agent orchestrates tools and humans.
- You need **business and engineering in the same room**, speaking the same language, before code is written.
- You want to surface **risks, handoffs, and human checkpoints early**, while they are cheap to change.

### When *not* to use it

- A single, simple prompt-in/answer-out feature with no orchestration, tools, or routing — that doesn't need a workshop.
- A pure software domain with no AI agents — use classic Event Storming instead.

---

## 2. Why it exists

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

## 3. Agent Storming vs Event Storming

If you know Event Storming, you already know 70% of Agent Storming. The format is identical: a long timeline, read left to right, covered in coloured stickies that everyone places together. What changes is **what each colour means** — each Event Storming element is reinterpreted for agentic systems — plus a handful of **agent-native additions** that have no equivalent in the original.

### 3.1 The mapping (same shape, new meaning)

| Event Storming element | Agent Storming element | What it now means |
|---|---|---|
| **Actor** (who triggers) | **Trigger** | What *starts* a flow: a human request, a schedule/cron, an inbound webhook or event. |
| **Command** | **Task** | A unit of work dispatched *to an agent*. |
| **Aggregate** | **Agent** | The actor that receives tasks and produces outcomes — carrying a role, a model, tools, and memory. |
| **Domain Event** | **Outcome / Event** | What an agent *produced* (past tense), e.g. "Ticket classified as billing". |
| **Policy** | **Policy / Routing rule** | The reactive glue: conditional routing, fan-out, retries, escalation. |
| **External system** | **Tool / External system** | A deterministic capability an agent calls: API, DB, MCP server, search, code-exec. |
| **Read model** | **Context / Read model** | Information an agent reads to decide or act: shared memory, retrieved docs, KB, prior messages. |
| **Hotspot** | **Hotspot / Risk** | An open question, a risk, a "we need to decide" — including AI-specific risks like hallucination. |

The kinship runs deep enough that, faithfully to the original, **Trigger and Agent share the yellow/amber family** (the way Actor and Aggregate are visually related in Event Storming) and are told apart by size and border — small Trigger, large bold-bordered Agent.

### 3.2 The agent-native additions (no Event Storming equivalent)

These exist because agentic systems have concerns classical domain modelling didn't:

- **Guardrail** — a validation, safety, budget, or content gate placed *before or after* a step: a schema check, a PII filter, a cost cap, a content policy. Agents are non-deterministic and can be expensive; guardrails are how you constrain them.
- **Human-in-the-loop** — an explicit human checkpoint: review, approve, correct, escalate. In Event Storming humans are *actors who trigger*; here, a human is also a deliberate *control point inside* an automated flow.
- **Swarm** (container) — a region grouping *different* agents collaborating toward one goal (typically an orchestrator plus workers with dynamic handoffs).
- **Fleet** (container) — a region denoting *N parallel instances of the same agent* (homogeneous horizontal scale, e.g. 50 workers draining a queue).

### 3.3 The deeper differences

- **You are modelling non-deterministic actors.** An Agent is not a deterministic aggregate; it can be wrong, vary, or hallucinate. That is *why* guardrails and human checkpoints are first-class.
- **Safety and cost are part of the grammar**, not an afterthought. A board with no guardrails and no human checkpoints on risky actions is, visibly, an incomplete design.
- **Suggest, don't enforce.** Event Storming is already loose and exploratory; Agent Storming keeps that. The grammar is a shared language that helps you *notice* problems (a tool call that doesn't start from an agent, a policy with no triggering outcome), not a rigid schema that blocks you. Surface issues as warnings or Hotspots — don't hard-stop the conversation.
- **Same muscle memory, new subject.** The timeline still reads left→right as causality and sequence; pivotal events still divide it into phases. If your team has stormed a domain before, the only new thing to learn is the vocabulary.

---

## 4. The grammar (notation reference)

The grammar is the heart of the method. Keep it visible in the room.

### 4.1 Elements (sticky notes)

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

### 4.2 Containers (regions, not stickies)

- **Individual agent** — a single Agent sticky.
- **Swarm** — a translucent **violet** frame grouping *different* agents collaborating toward one goal (orchestrator + workers, dynamic handoffs).
- **Fleet** — a translucent **teal** frame denoting *N parallel instances of the same agent*. Drawn as a stacked card with an instance-count badge.

### 4.3 Connectors

- **Flow** (solid line) — sequence/causality along the timeline: Trigger → Task → Agent → Outcome → Policy → Task…
- **Handoff** (dashed) — agent-to-agent delegation inside a swarm.
- **Tool call** (dotted) — Agent → Tool (and back).
- **Reads** (thin) — Context/Read model → Agent (informs a decision).

### 4.4 Pivotal markers

- **Milestone / phase divider** — a vertical line splitting the timeline into stages (Event Storming's "pivotal event"). Usually coincides with a human approval or a major state change.

### 4.5 Sentence templates (the grammar that checks itself)

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

## 5. The three levels

Agent Storming runs at three zoom levels. Pick one per session (it's a board-level setting) and graduate upward as understanding solidifies.

1. **Big Picture** — sketch the *whole* automated business line: where humans hand off to agents and agents to each other. Broad, chaotic, exploratory. Goal: a shared map and a pile of Hotspots. *Done when:* the room agrees on the major phases and the biggest unknowns are marked.
2. **Process** — take *one* process end-to-end and add the policies, tools, guardrails, and human checkpoints. *Done when:* you can walk the timeline with the sentence templates and it reads with no dead ends.
3. **Design-level** — detailed enough to implement: each agent's role, model, tools, memory, and guardrails specified. This is the level a board can be handed to engineering and (eventually) compiled to scaffolding. *Done when:* every Agent sticky carries its props and every risky action has a guardrail or human checkpoint.

Don't start at Design-level. Over-specifying before the shape is agreed is the most common way to waste a session (see §8 pitfalls).

---

## 6. How to run a session

### 6.1 Who to invite

The whole point is mixing perspectives. Aim for:

- **A facilitator** (keeps the timeline honest, asks the dumb questions, enforces the grammar gently).
- **Business / domain experts** — the people who actually understand the process being automated.
- **Process owners / operations** — who know the edge cases and where humans currently intervene.
- **Engineers / AI architects** — who know what agents and tools can realistically do.
- Keep it under ~8 active modellers per wall; split into parallel walls for larger groups.

### 6.2 Space and materials

- **Physical:** a long, unbroken wall (or several whiteboards in a row), a roll of paper, and sticky notes in the grammar colours. Print the §9 cheat sheet and pin it up.
- **Digital / remote:** an infinite canvas. The companion Agent Storming app is purpose-built for this (the grammar colours, the typed connectors, swarm/fleet frames, and the sentence-template validation are baked in), but any Miro-style tool with the legend pinned works for a first session.
- A visible, agreed **legend** is non-negotiable — the colours only help if everyone reads them the same way.

### 6.3 The flow of a session

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

### 6.4 Timeboxing

A first Big-Picture session is typically **half a day**. A single Process-level pass is **2–4 hours**. Design-level work on one process can take a **full day** or recur over several sessions. Resist the urge to finish everything in one sitting — Hotspots are a perfectly good place to stop.

---

## 7. Worked example: AI support-ticket triage

Let's storm a realistic process end-to-end: **an AI system that triages inbound support tickets, resolves what it can, and escalates the rest** — at **Process** level, graduating toward Design-level for one branch.

### 7.1 Building it up, sticky by sticky

**Start with the trigger and the first task.**

> *When* **a support ticket arrives** (Trigger 🟡), *start* **Classify ticket** (Task 🔵).

**Add the agent and its outcome.**

> **Classify ticket** is dispatched to the **Triage Agent** (Agent 🟠-amber).
> The **Triage Agent** *produces* **Ticket classified as billing** (Outcome 🟠).

Already a question surfaces: what if the classifier is wrong? Drop a **Hotspot** 🔴: *"Misclassification → wrong queue. Hallucination risk?"*

**Give the agent context to decide well.**

> **Customer history** (Context 🟢) and the **Knowledge base / vector store** (Context 🟢, via a *reads* connector) inform the **Triage Agent**.

**Guard the inputs.**

> *Before* classification, check the **PII filter** (Guardrail 🟦-teal); on failure → redact then continue.
> *Across the run*, enforce **Max $0.20 / ticket** (Guardrail 🟦); on failure → stop and flag.

**Route on the outcome (the orchestration glue).**

> *Whenever* **Ticket classified as billing**, *dispatch* **Resolve billing query** *to the* **Billing Agent** (Policy 🟣 → Task 🔵 → Agent 🟠).
> *Whenever* **Ticket classified as technical**, *dispatch* **Draft technical reply** *to the* **Tech-Support Agent**.

**Let an agent use a real-world tool.**

> The **Billing Agent** *calls* the **Stripe API** (Tool 🩷, dotted connector) *to* look up the invoice and, if warranted, issue a refund.

**Insert the human checkpoint where money moves.**

> *Whenever* a refund **> £100** is proposed, a **Support Lead reviews the refund** (Human-in-the-loop 🤍) *and approves/rejects.*

This is also a natural **milestone** — drop a vertical phase divider here ("Human approval"). Small refunds flow straight through; large ones cross the line into the reviewed phase.

**Close the loop.**

> The **Billing Agent** *produces* **Refund issued** (Outcome 🟠).
> *Whenever* **Refund issued** *or* **Reply sent**, *dispatch* **Close ticket & request CSAT** (Task 🔵).

### 7.2 A swarm and a fleet

Two structural patterns show up in this process:

- **Swarm** (violet frame): the **Triage Agent** (orchestrator) hands off to specialist workers — **Billing Agent**, **Tech-Support Agent**, **Account Agent** — with dynamic handoffs depending on classification. They are *different* agents collaborating toward one goal: a resolved ticket.
- **Fleet** (teal frame): on Monday mornings there's a backlog. Draw a **Fleet of Triage Agents ×25** — 25 parallel instances of the *same* agent draining the ticket queue. Same role, horizontal scale.

### 7.3 The resulting board (sketch)

```
  TRIGGER        TASK            AGENT (swarm)         OUTCOME           POLICY ──► TASK ──► AGENT        OUTCOME        HUMAN          OUTCOME
  ───────        ────            ─────────────         ───────           ──────────────────────────       ───────        ─────          ───────
┌─────────┐   ┌──────────┐   ╔═══════════════════╗  ┌──────────────┐  ┌────────────────┐ ┌──────────┐  ┌──────────┐ │ ┌──────────┐  ┌──────────┐
│ Ticket  │──►│ Classify │──►║   Triage Agent    ║─►│  Classified  │─►│ Whenever       │►│ Resolve  │─►│ Refund   │ │ │ Support  │─►│ Refund   │
│ arrives │   │  ticket  │   ║   (orchestrator)  ║  │  as billing  │  │ billing →      │ │ billing  │  │ proposed │ │ │ Lead     │  │ issued   │
└─────────┘   └──────────┘   ║                   ║  └──────────────┘  │ Billing Agent  │ │  query   │  │  >£100   │ │ │ reviews  │  └──────────┘
                  ▲          ║  ┌─────────────┐  ║                    └────────────────┘ └──────────┘  └──────────┘ │ └──────────┘
            ┌─────┴─────┐    ║  │Billing Agent│◄═╝                                            │ (dotted)    │       │      MILESTONE: human approval
            │ PII filter│    ║  │Tech Agent   │  ║                                       ┌──────────┐        │       │
            │ (guardrail)│   ║  │Account Agent│  ║                                       │Stripe API│◄───────┘       │
            └───────────┘    ║  └─────────────┘  ║                                       │  (tool)  │                │
                             ╚═══════════════════╝                                       └──────────┘                │
   reads ▲  ▲                  Fleet: Triage Agent ×25  (parallel, drains backlog)
  ┌───────┐ ┌──────────────┐                                                          🔴 Hotspot: misclassification →
  │Customer│ │ Knowledge    │                                                            wrong queue / hallucination risk?
  │history │ │ base (vector)│
  └───────┘ └──────────────┘
```

(The sketch is deliberately rough — a real board is messier and that's healthy. The point is that you can *walk it left to right and read it as sentences*.)

### 7.4 What the board just told us

Reading the finished board surfaces exactly the things §2 worried about:

- The **misclassification Hotspot** is unresolved — maybe add a confidence threshold and a "route to human if unsure" policy.
- The **cost guardrail** and **PII guardrail** are present — good.
- The **human checkpoint** exists for large refunds but not for technical replies sent to customers — *should it?* Another Hotspot.
- The **Triage Agent** is a single point of failure in the swarm — the Fleet mitigates throughput but not logic. Worth a note.

None of these cost anything to find at the wall. All of them are expensive to find in production.

---

## 8. Implementing Agent Storming in your business

### 8.1 Where it fits in delivery

Agent Storming sits in the **discovery → design** seam, just before build:

1. **Discovery** — a Big-Picture session maps the candidate automated business line and its biggest unknowns.
2. **Design** — Process and then Design-level sessions on the specific flow you've chosen to build first.
3. **Build** — engineering implements from the Design-level board, which stays pinned as the living spec.

### 8.2 Picking your first process

Choose a first candidate that is **valuable, bounded, and tolerant of iteration**:

- High enough volume that automation matters.
- A clear trigger and a clear "done".
- Existing human judgement you can point at (so you know where the human checkpoints belong).
- Low blast radius if the agents get it wrong early — *avoid* irreversible, high-stakes actions for your very first build.

Support triage, lead qualification, document intake, and internal-IT request routing are common, friendly starting points.

### 8.3 Making it a practice, not a one-off

- Run a session **before** each significant agentic build, and **re-storm** when the process changes materially.
- Keep the board as **living documentation**. A board that drifts from reality is worse than none — assign an owner.
- Build a small **library of patterns** your org reuses: a standard "human approves anything that spends money" checkpoint, a standard PII guardrail, a standard "route to human on low confidence" policy.

### 8.4 Handing off to engineering

A Design-level board is the engineering brief: every agent's role/model/tools/memory, every policy, every guardrail, every human checkpoint is on it. Engineers should be *in* the session, so the handoff is a continuation, not a translation.

The direction of travel — and the reason the grammar is kept precise — is that a Design-level board becomes **machine-readable** enough to (a) be **critiqued by AI** for missing checkpoints, unguarded tool calls, single points of failure, and orphaned policies, and (b) eventually be **compiled into runnable agent scaffolding**. Treat that as where the method is heading, not a feature to depend on today; the design value stands on its own regardless.

### 8.5 Common pitfalls

- **Over-detailing too early.** Jumping to Design-level before the Big Picture is agreed. Storm broad first.
- **Skipping guardrails and human checkpoints.** The most dangerous board is the one that looks complete but has an agent spending money or contacting customers with nobody watching. Always do the "what could go wrong, who's watching?" pass (§6.3 step 7).
- **Enforcing the grammar too rigidly.** The grammar is a thinking aid, not a compliance exercise. Surface problems as Hotspots and warnings; don't let notation policing kill the conversation.
- **No room mix.** Business-only sessions miss what's buildable; engineering-only sessions miss what the process actually is. Mix the room.
- **Resolving every Hotspot live.** Capture them and move on; a session that stalls on one argument loses the wall.

---

## 9. Quick reference / cheat sheet

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
