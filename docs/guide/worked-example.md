# 7. Worked example: AI support-ticket triage

Let's storm a realistic process end-to-end: **an AI system that triages inbound support tickets, resolves what it can, and escalates the rest** — at **Process** level, graduating toward Design-level for one branch.

## 7.1 Building it up, sticky by sticky

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

## 7.2 A swarm and a fleet

Two structural patterns show up in this process:

- **Swarm** (violet frame): the **Triage Agent** (orchestrator) hands off to specialist workers — **Billing Agent**, **Tech-Support Agent**, **Account Agent** — with dynamic handoffs depending on classification. They are *different* agents collaborating toward one goal: a resolved ticket.
- **Fleet** (teal frame): on Monday mornings there's a backlog. Draw a **Fleet of Triage Agents ×25** — 25 parallel instances of the *same* agent draining the ticket queue. Same role, horizontal scale.

## 7.3 The resulting board (sketch)

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

## 7.4 What the board just told us

Reading the finished board surfaces exactly the things §2 worried about:

- The **misclassification Hotspot** is unresolved — maybe add a confidence threshold and a "route to human if unsure" policy.
- The **cost guardrail** and **PII guardrail** are present — good.
- The **human checkpoint** exists for large refunds but not for technical replies sent to customers — *should it?* Another Hotspot.
- The **Triage Agent** is a single point of failure in the swarm — the Fleet mitigates throughput but not logic. Worth a note.

None of these cost anything to find at the wall. All of them are expensive to find in production.

---
