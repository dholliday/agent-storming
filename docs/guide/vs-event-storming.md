# 3. Agent Storming vs Event Storming

If you know Event Storming, you already know 70% of Agent Storming. The format is identical: a long timeline, read left to right, covered in coloured stickies that everyone places together. What changes is **what each colour means** — each Event Storming element is reinterpreted for agentic systems — plus a handful of **agent-native additions** that have no equivalent in the original.

## 3.1 The mapping (same shape, new meaning)

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

## 3.2 The agent-native additions (no Event Storming equivalent)

These exist because agentic systems have concerns classical domain modelling didn't:

- **Guardrail** — a validation, safety, budget, or content gate placed *before or after* a step: a schema check, a PII filter, a cost cap, a content policy. Agents are non-deterministic and can be expensive; guardrails are how you constrain them.
- **Human-in-the-loop** — an explicit human checkpoint: review, approve, correct, escalate. In Event Storming humans are *actors who trigger*; here, a human is also a deliberate *control point inside* an automated flow.
- **Swarm** (container) — a region grouping *different* agents collaborating toward one goal (typically an orchestrator plus workers with dynamic handoffs).
- **Fleet** (container) — a region denoting *N parallel instances of the same agent* (homogeneous horizontal scale, e.g. 50 workers draining a queue).

## 3.3 The deeper differences

- **You are modelling non-deterministic actors.** An Agent is not a deterministic aggregate; it can be wrong, vary, or hallucinate. That is *why* guardrails and human checkpoints are first-class.
- **Safety and cost are part of the grammar**, not an afterthought. A board with no guardrails and no human checkpoints on risky actions is, visibly, an incomplete design.
- **Suggest, don't enforce.** Event Storming is already loose and exploratory; Agent Storming keeps that. The grammar is a shared language that helps you *notice* problems (a tool call that doesn't start from an agent, a policy with no triggering outcome), not a rigid schema that blocks you. Surface issues as warnings or Hotspots — don't hard-stop the conversation.
- **Same muscle memory, new subject.** The timeline still reads left→right as causality and sequence; pivotal events still divide it into phases. If your team has stormed a domain before, the only new thing to learn is the vocabulary.

---
