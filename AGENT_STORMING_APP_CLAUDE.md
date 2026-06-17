# CLAUDE.md — Agent Storming

> Project memory for Claude Code. Keep this file concise and current. The **grammar** in §2 is the single source of truth for the product; UI colours, labels, and validation all derive from it.

---

## 1. Vision

**Agent Storming** is a collaborative, Miro-style web app for designing multi-agent AI systems using the techniques of **Event Storming** (Alberto Brandolini). Instead of modelling a domain with sticky notes for events/commands/aggregates, users model a *business process executed by AI agents*: individual agents, **swarms** (heterogeneous agents collaborating on a goal), and **fleets** (parallel instances of the same agent), plus the humans, tools, policies, and guardrails that wire them together.

The canvas is more than a diagram — it's a **specification**. The long-term differentiator is that a finished board can be (a) validated by AI for missing checkpoints/failure modes and (b) compiled into runnable agent scaffolding. See §10.

Design bar: **slick, fast, tactile.** Infinite canvas, smooth pan/zoom, springy drag, dark workspace so the sticky colours pop. It should feel like Miro/tldraw, not like a form builder.

---

## 2. The Agent Storming method (DOMAIN GRAMMAR — read this first)

Event Storming uses a coloured-sticky grammar on a long timeline read left→right. Agent Storming keeps the same muscle memory but reinterprets each element for agentic systems.

### 2.1 Elements (sticky notes)

| Element | Colour (hex) | Event-Storming analogue | Meaning | Phrasing |
|---|---|---|---|---|
| **Trigger** | Yellow `#FFE066` (small) | Actor | What starts a flow: a human request, a schedule/cron, an inbound webhook/event | noun/event — "Support ticket arrives", "Nightly 02:00" |
| **Task** | Blue `#4DABF7` | Command | A unit of work dispatched to an agent | imperative — "Classify ticket", "Draft reply" |
| **Agent** | Amber `#FFC078` (large, bold border) | Aggregate | The actor that receives tasks and produces outcomes. Carries `role`, `model`, `tools`, `memory`, `subtype` | role name — "Triage Agent" |
| **Outcome / Event** | Orange `#FF922B` | Domain Event | What an agent produced | past tense — "Ticket classified as billing" |
| **Policy / Routing rule** | Lilac `#B197FC` | Policy | Reactive orchestration glue: conditional routing, fan-out, retries, escalation | "Whenever <outcome>, dispatch <task> to <agent>" |
| **Tool / External system** | Pink `#F783AC` | External system | A deterministic capability an agent calls: API, DB, MCP server, search, code-exec | "Stripe API", "Vector store" |
| **Context / Read model** | Green `#69DB7C` | Read model | Information an agent reads to decide/act: shared memory, retrieved docs, prior messages, KB | "Customer history" |
| **Guardrail** | Teal `#3BC9DB` | — (agent-native) | Validation/safety/budget gate before or after a step: schema check, PII filter, cost cap, content policy | "PII filter", "Max $0.50/run" |
| **Human-in-the-loop** | Cream `#F8F9FA` (dark border, person glyph) | — (agent-native) | Explicit human checkpoint: review, approve, correct, escalate | "Agent reviews refund" |
| **Hotspot / Risk** | Red `#FF6B6B` | Hotspot | Open question, hallucination risk, single point of failure, "we need to decide" | free text + `?` |

> Faithful to Event Storming: Trigger and Agent share the yellow family (actor vs aggregate) but differ by size and border, exactly as in the original.

### 2.2 Containers (frames/regions, not stickies)

- **Individual agent** — a single Agent sticky.
- **Swarm** — a translucent **violet** frame grouping *different* agents collaborating toward one goal (usually an orchestrator + workers with dynamic handoffs).
- **Fleet** — a translucent **teal** frame denoting *N parallel instances of the same agent* (homogeneous horizontal scale, e.g. 50 workers draining a queue). Render as a stacked card with an instance-count badge.

### 2.3 Connectors

- **Flow** (solid) — sequence/causality along the timeline: Trigger → Task → Agent → Outcome → Policy → Task…
- **Handoff** (dashed) — agent-to-agent delegation (A2A) inside a swarm.
- **Tool call** (dotted) — Agent → Tool (and back).
- **Reads** (thin) — Context/Read model → Agent (informs a decision).

### 2.4 Pivotal markers

- **Milestone / phase divider** — a vertical line splitting the timeline into stages (Event Storming's "pivotal event"). Usually coincides with a human approval or a major state change.

### 2.5 Sentence templates (also used by the AI generator/validator)

- Trigger → "When **{trigger}**, start **{task}**."
- Task → Agent → "**{task}** is dispatched to **{agent}**."
- Agent → Outcome → "**{agent}** produces **{outcome}**."
- Policy → "Whenever **{outcome}**, dispatch **{task}** to **{agent}**."
- Guardrail → "Before/after **{step}**, check **{guardrail}**; on failure → **{route}**."
- Human → "**{human}** reviews **{outcome}** and approves/rejects."
- Tool → "**{agent}** calls **{tool}** to **{action}**."

### 2.6 Levels (workshop modes — a board-level setting)

1. **Big Picture** — sketch the whole automated business line; where humans hand off to agents and agents to each other. Broad, chaotic, exploratory.
2. **Process** — one process end-to-end with policies, tools, guardrails.
3. **Design-level** — detailed enough to implement: each agent's role/model/tools/guardrails specified. This is the level that compiles to scaffolding (§10).

---

## 3. Tech stack

TypeScript everywhere. Be decisive; ask before adding a dependency not listed here.

- **Framework:** Next.js (App Router) + React.
- **Styling/chrome:** Tailwind CSS + **shadcn/ui** for toolbars, dialogs, panels.
- **Canvas:** **tldraw** — ✅ **decided.** Freeform, Miro-style infinite canvas; selection, arrows, multiplayer story, custom shapes. Each grammar element = a custom tldraw shape carrying its props. We chose freeform over a strict node-graph (React Flow / `@xyflow/react`) deliberately: the brainstorming feel is the priority, and grammar correctness is handled by *suggest/warn* validation rather than hard structural enforcement (see §9).
  - ⚠️ tldraw shows a "made with tldraw" watermark unless you hold a business licence — fine for prototype, factor in before commercial launch.
- **App/UI state:** tldraw store for the canvas; Zustand for app state if needed.
- **Backend + realtime (pick ONE, see §9):**
  - **Convex** — reactive DB + serverless functions, realtime out of the box. Recommended for a collab-first app.
  - **Supabase** — Postgres + Auth + Realtime, more familiar.
  - **Liveblocks** for the collab layer + Postgres/Prisma for storage.
- **Auth:** Clerk, Supabase Auth, or Auth.js.
- **AI:** Anthropic API (Claude) for generate / critique / compile features (§10).
- **Hosting:** Vercel + chosen backend.
- **Testing:** Vitest + React Testing Library; Playwright for e2e on the canvas.
- **Tooling:** pnpm, TypeScript `strict`, ESLint, Prettier.

---

## 4. Architecture & project structure

```
/app
  /(marketing)        landing
  /dashboard          board list (auth)
  /board/[id]         the editor
/components
  /canvas             tldraw setup, custom shape utils, tools
  /palette            element palette + colour legend
  /panels             inspector (edit agent props, policy condition, etc.)
  /ui                 shadcn components
/lib
  /grammar            SINGLE SOURCE OF TRUTH: element defs, colours, sentence templates
  /schema             zod schemas + TS types (board/node/edge/frame)
  /ai                 prompt builders for generate/critique/compile
  /validation         grammar rules (e.g. "tool call must originate from an agent")
/convex | /server     backend functions
/public
```

---

## 5. Data model (sketch — formalise in `/lib/schema` with zod)

```ts
type NodeType =
  | 'trigger' | 'task' | 'agent' | 'outcome' | 'policy'
  | 'tool' | 'context' | 'guardrail' | 'human' | 'hotspot';

interface BaseNode { id: string; type: NodeType; x: number; y: number; w: number; h: number; text: string; }
interface AgentNode extends BaseNode {
  type: 'agent';
  role: string;
  model?: string;
  tools: string[];
  memory: 'none' | 'short' | 'long';
  subtype: 'orchestrator' | 'worker' | 'specialist' | 'evaluator';
}
interface PolicyNode   extends BaseNode { type: 'policy'; condition: string; action: string; }
interface GuardrailNode extends BaseNode { type: 'guardrail'; kind: string; onFailure: string; }

type EdgeKind = 'flow' | 'handoff' | 'tool' | 'reads';
interface Edge { id: string; source: string; target: string; kind: EdgeKind; label?: string; }

interface Frame { id: string; kind: 'swarm' | 'fleet'; childIds: string[]; instanceCount?: number /* fleet */; }

interface Board { id: string; title: string; level: 'big-picture' | 'process' | 'design'; nodes: BaseNode[]; edges: Edge[]; frames: Frame[]; }
```

> With tldraw, geometry/selection live in its store and type-specific fields live in `shape.props`. Persist via tldraw snapshots **or** map to the normalised `Board` schema above for export/AI — decide and keep one canonical form.

---

## 6. Implementation roadmap

Estimates assume one capable dev using Claude Code heavily. Calendar time is longer if part-time. Cut scope aggressively early.

| Phase | Goal | Key deliverables | Est. (focused days) | Milestone |
|---|---|---|---|---|
| **0 — Setup** | Skeleton deploys | Next.js + TS + Tailwind + shadcn, tldraw hello-world, repo, Vercel | 0.5–1 | Blank canvas live on Vercel |
| **1 — Grammar canvas** | Place the notes | Custom shapes for all 10 elements, palette + legend, create/edit/drag/recolour | 3–5 | Can build a board by hand |
| **2 — Flow semantics** | Wire them up | Typed connectors (flow/handoff/tool/reads), swarm & fleet frames, basic grammar validation | 3–5 | A complete process reads end-to-end |
| **3 — Persistence + auth** | Save & own boards | Backend, board CRUD, dashboard, accounts | 2–4 | Reload a saved board |
| **4 — Realtime collab** | Multiplayer | Live cursors, presence, sync (tldraw sync / Liveblocks) | 3–6 | Two browsers edit together |
| **5 — Polish** | Make it sexy | Design pass, animations, onboarding, templates, PNG/JSON export, empty states | 3–5 | Demo-ready v1 |
| **6 — AI features** | The differentiator | Prompt→board, board critique, compile→scaffolding | 4–8 | Generate + critique a board |

**Headline numbers**
- **Fastest demo** (phases 0–2, single-player, localStorage): **~1 week**.
- **MVP** (0–3, persistent, lightly polished): **~2 weeks**.
- **Collaborative polished v1** (0–5): **~4–6 weeks**.
- **+ AI compile** (0–6): **~6–8 weeks**.

**Suggested MVP cut:** phases 0–3 single-player. Get the *concept* tangible first; collaboration (4) and AI (6) are the risky/expensive parts — layer them once the grammar feels right.

---

## 7. Conventions & quality bar

- `/lib/grammar` is canonical — never hard-code a colour or label in a component; import it.
- TypeScript `strict`; no `any`. Validate external/AI data with zod at the boundary.
- Small, focused components. Keep canvas re-renders cheap (don't re-render the whole board on every change).
- Accessibility: keyboard for palette/selection, sensible ARIA on chrome. (Canvas is desktop-first; mobile is not a v1 target.)
- Before declaring a task done: run typecheck + lint + relevant tests.
- Small commits with clear messages. Ask before adding dependencies.

### 7.1 Red → Green → Refactor (default workflow)

Test-driven development is the default loop for all non-trivial logic:

1. **Red** — write the smallest failing test that specifies the next behaviour. Run it; confirm it fails for the right reason.
2. **Green** — write the minimum code to make it pass. No more.
3. **Refactor** — clean up names/structure with the suite green. Commit on green.

Work in small steps — one behaviour per cycle. Never finish a task with a red test, and never delete or weaken a test just to go green.

**TDD is strict here** (pure, deterministic — write the test first):
- `/lib/grammar` — element defs, colour/label lookups, sentence-template rendering
- `/lib/schema` — zod parsing and board (de)serialisation round-trips
- `/lib/validation` — grammar rules (e.g. "a tool-call edge must originate from an Agent")
- `/lib/ai` — parsing/validating model output into a `Board`

**TDD is pragmatic here** (UI/canvas — don't chase coverage on tldraw internals or pixels):
- Component behaviour (palette, inspector) → React Testing Library; test-after is fine
- Canvas interactions (drag, connect, frame) → a few Playwright e2e happy-paths, not exhaustive unit tests

Keep the suite fast so the loop stays tight. `pnpm test` is part of "done".

---

## 8. Commands

> Confirm/adjust once tooling is in place.

```bash
pnpm dev          # run app locally
pnpm build        # production build
pnpm lint         # eslint
pnpm typecheck    # tsc --noEmit
pnpm test         # vitest
pnpm e2e          # playwright
# npx convex dev  # if using Convex
```

---

## 9. Decisions & open questions

**✅ Decided**
- **Canvas engine:** freeform canvas via **tldraw** — the Miro-style brainstorming feel is prioritised over strict node-graph enforcement (React Flow considered and set aside).

**Open questions**
1. **Backend/realtime:** Convex vs Supabase vs Liveblocks+Postgres. Pick before phase 3.
2. **Canonical board form:** tldraw snapshot vs normalised `Board` schema (§5) for export/AI.
3. **Grammar strictness:** how much to *enforce* vs *suggest*. Given the freeform-canvas decision, lean toward **suggest/warn** (surface a Hotspot or inline warning) rather than hard-blocking invalid structures.
4. **Compile targets** *(later phase — see §10):* which agent framework(s) to generate — **LangGraph**, **Foundry (Azure AI Foundry)**, CrewAI, OpenAI Agents SDK, Anthropic agent patterns, or a neutral JSON/YAML orchestration spec.
5. **Auth provider** + monetisation (tldraw watermark/licence).

---

## 10. Stretch / differentiators (the reason this isn't "just a diagram")

1. **Prompt → board.** User describes a process in plain English; Claude returns a structured `Board` (nodes/edges/frames) that renders on the canvas. Uses the §2.5 templates as the contract.
2. **Board critique.** Claude audits a board for: missing human checkpoints on high-risk actions, unguarded tool calls, single points of failure, policies with no triggering outcome, runaway-cost paths, missing retry/escalation. Surfaces results as Hotspot stickies.
3. **Compile → scaffolding.** ⏳ *Later-phase TODO — explicitly deferred.* Export a Design-level board to **deployable** multi-agent code — auto-generating the agents themselves into **LangGraph**, **Foundry (Azure AI Foundry)**, or other agent frameworks — or a neutral orchestration spec. This is the long-term payoff that turns the board into a true source of truth, but it is parked until the canvas + grammar (phases 0–5) are solid. Target framework(s) TBD (§9.4).
4. **Simulate a run.** Step through the flow, highlighting the active path, to sanity-check routing before writing code.

---

*Keep this file lean — it loads every session. Update §2 and §9 as decisions land.*
