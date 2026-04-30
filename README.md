<div align="center">

<img src="https://static.wixstatic.com/media/e2da02_2be72852d6ae4857a5b78c5c4cd53e0d~mv2.png" alt="Hermes Loop" width="420" />

### The control room for the Hermes Agent.
**Persistent agent crews. Hashed receipts. Approval gates. Honest providers.**

[![Verified by Hermes Agent](https://img.shields.io/badge/%E2%9C%93%20Verified%20by-Hermes%20Agent-A78BFA?style=for-the-badge&labelColor=22c55e)](https://github.com/NousResearch/hermes-agent)

[![Website](https://img.shields.io/badge/Website-hermesloop.cc-22c55e?style=flat-square&logo=globe&logoColor=white)](https://hermesloop.cc)
[![Follow on X](https://img.shields.io/badge/Follow-%40HermesLoop-000?style=flat-square&logo=x&logoColor=white)](https://x.com/HermesLoop)
[![License: MIT](https://img.shields.io/badge/License-MIT-A78BFA.svg?style=flat-square)](LICENSE)
[![Next.js 14](https://img.shields.io/badge/Next.js-14.2-000?style=flat-square&logo=next.js&logoColor=white)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Prisma](https://img.shields.io/badge/Prisma-Postgres-2D3748?style=flat-square&logo=prisma&logoColor=white)](https://www.prisma.io/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.4-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)
[![Zod](https://img.shields.io/badge/Zod-validated-3068B7?style=flat-square)](https://zod.dev)
[![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-MIT-A78BFA?style=flat-square)](https://github.com/NousResearch/hermes-agent)
[![MCP](https://img.shields.io/badge/MCP-native-22d3ee?style=flat-square)](https://modelcontextprotocol.io)
[![Deploy on Vercel](https://img.shields.io/badge/Deploy-Vercel-000?style=flat-square&logo=vercel&logoColor=white)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fctrlshifthash%2FHermesAgent)

[Website](https://hermesloop.cc) · [X / @HermesLoop](https://x.com/HermesLoop)

</div>

---

## What it is

Hermes Loop is the **operator surface** for the [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Nous Research's autonomous AI engine. Hermes does the reasoning. **Hermes Loop wires the runtime around it**: ordered crews of subagents, sandboxed tools, an inbound triage queue, governed memory, approval gates, and a hashed Workflow Receipt at the end of every run that proves what happened.

> **One design decision drives everything else: agent runs are treated like database transactions, not chat sessions.** Every prompt, every raw response, every tool call, every approval, every memory injection is a row you can query, replay, and prove. If it's worth doing, it has a row. If it has a row, it's in the receipt.

---

## Why it matters

| Without Hermes Loop | With Hermes Loop |
|---|---|
| "The AI said it would…" | A SHA-256 hashed `WorkflowReceipt` you hand to compliance |
| Implicit memory, hidden context | Operator-approved memory with diffs + cross-session recall |
| Vibes-based safety | First-class `ApprovalItem` rows for drafts, trades, exports |
| Estimated cost | Live per-model rates × actual tokens, recorded per receipt |
| Schema parsing roulette | Zod self-correction loop with structured failures |
| Brittle one-shot scripts | Ordered crews with retries + resumable failures |

---

## Feature matrix

All status flags are read **live** from `process.env` at request time — no fake green checks.

### Reasoning & memory
- ✅ **Hermes chat completions** with retry / backoff / `Retry-After` honoring
- ✅ **Multi-model routing** — `HERMES_MODEL_FAST` / `STRONG` / `JUDGE` / `VISION` per agent role
- ✅ **Cross-session memory recall** — every mission queries operator-approved memory across all prior runs
- ✅ **Learning loop** — after every settle, the JUDGE model distils up to 3 reusable lessons into `Skill` rows scoped to the crew
- ✅ **MCP integration** — native Model Context Protocol client (JSON-RPC, no SDK); tools surface alongside built-ins

### Tools
- ✅ Browser QA (`browser_qa_audit` — Playwright + Chromium)
- ✅ Terminal (`terminal_exec`) and Python RPC (`python_rpc`) — local **+** Docker backends with `--network=none --read-only --cap-drop=ALL`
- ✅ Web search (Tavily / Brave / SerpAPI fallback chain)
- ✅ Vision (Gemini 2.5 Flash via Hermes — multimodal)
- ✅ Image generation (Hermes image-capable models, Fal / Replicate fallback, auto cheapest-model selection)
- ✅ Text-to-speech (ElevenLabs `eleven_multilingual_v2`)

### Operations
- ✅ Triage inbox + signed inbound webhooks (Discord / Slack / Email / generic)
- ✅ Persistent job queue with retries, exponential backoff, optimistic-lock pickup
- ✅ Scheduled missions — `DAILY` / `WEEKDAYS` / `WEEKLY` / `MONTHLY` / `ONCE` cadences with NL parsing
- ✅ Long-lived worker (`scripts/worker.ts`) with structured tick logs

### Governance & proof
- ✅ Hashed `WorkflowReceipt` — agent timeline, tool I/O, approvals, memory injections, real cost, integrity hash
- ✅ Approval gates as first-class objects (drafts, trades, exports, gated tools)
- ✅ Trust ledger — aggregate reliability roll-up across every run
- ✅ Schema self-correction loop — Zod-validated outputs with re-prompt on parse failure
- ✅ Real evals harness — `npm run evals` runs full missions against live Hermes, exit code = failed cases

→ Live source-of-truth: [`/hermes/parity`](https://hermes-agent-mocha-five.vercel.app/hermes/parity)

---

## Quick start

### 1. Clone and install

```bash
git clone https://github.com/ctrlshifthash/HermesAgent.git
cd HermesAgent
npm install
```

### 2. Configure

```bash
cp .env.example .env.local
# then edit .env.local — at minimum:
#   DATABASE_URL=postgresql://...   (Postgres, e.g. neon.tech free tier)
#   HERMES_API_KEY=sk-or-v1-...     (OpenRouter key)
```

### 3. Database + run

```bash
npx prisma db push          # apply schema to your Postgres
npm run db:seed             # optional — seed demo workspace
npm run dev                 # http://localhost:3001
```

### 4. Verify

```bash
curl 'http://localhost:3001/api/hermes/health?force=1'
# → { "ok": true, "mode": "hermes", "latencyMs": <small> }

npm run evals:tools         # 7 cheap gating + metadata checks
npm run evals               # real mission suite (~$0.005, ~70s)
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Hermes Loop (Next.js 14)                 │
│                                                                 │
│   /missions/new ──────► Mission ────► Orchestrator              │
│                                          │                      │
│                          ┌───────────────┼─────────────────┐    │
│                          ▼               ▼                 ▼    │
│                     AgentRunStep     ToolCall        ApprovalItem│
│                          │               │                 │    │
│                          └───────────────┴─────────────────┘    │
│                                          │                      │
│                                          ▼                      │
│                                  WorkflowReceipt                │
│                                  (SHA-256 hashed)               │
│                                          │                      │
│                                          ▼                      │
│                                    Trust ledger                 │
└─────────────────────────────────────────────────────────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │ Hermes Agent (engine) │  ←  multi-model
                              │ via /chat/completions │      routing
                              └───────────────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │ Tools                 │
                              │ Playwright, terminal, │  ←  Zod-validated
                              │ Python RPC, search,   │      sandboxed
                              │ vision, image, TTS,   │      approval-gated
                              │ MCP servers           │
                              └───────────────────────┘
```

Every box maps to a Prisma table. Open `prisma studio` to inspect any past run.

---

## The desk — every surface mapped

Hermes Loop ships ~45 operator pages under the `(desk)` route group. Each one reads live from the database; nothing is mock data.

### Mission control
| Surface | What it shows |
|---|---|
| `/dashboard` | At-a-glance: live missions, queued jobs, last 10 receipts, trust roll-up, provider readiness |
| `/missions/new` | Pick a crew template + objective, attach documents, optionally schedule, run sync or queued |
| `/missions/[id]` | Live timeline: every `AgentRunStep`, raw + parsed output, every `ToolCall` with I/O, approvals, memory injections, settle status |
| `/jobs` | Persistent job queue — RUN_MISSION / TRIAGE_INBOX / SCHEDULE_TICK with attempts, locks, exponential backoff |
| `/schedules` | NL-parsed cadences (`DAILY`, `WEEKDAYS`, `WEEKLY`, `MONTHLY`, `ONCE`); per-row "run now" |

### Crews, templates, skills
| Surface | What it shows |
|---|---|
| `/templates` | The 4 shipped crew templates with full agent ordering and tool grants |
| `/templates/[key]` | Per-template detail: agent specs, schemas, tools allowed, model role per step |
| `/crews/new` · `/crews/[key]/edit` | Author custom crews — agent ordering, Zod schemas, tool grants, model roles |
| `/skills` | `Skill` rows distilled by the JUDGE model after every mission settle (up to 3 per run) |

### Approvals, receipts, trust
| Surface | What it shows |
|---|---|
| `/approvals` | Open `ApprovalItem` rows — drafts, paper trades, exports, gated tools — with approve/reject + reason |
| `/receipts` | All `WorkflowReceipt` rows with their integrity hash, cost, model breakdown, success state |
| `/receipts/[id]` | Full receipt: the timeline, every tool call I/O, approvals, memory injections, real cost per agent |
| `/trust` | Aggregate reliability roll-up — success rate, avg cost, parse-failure rate per crew |

### Memory
| Surface | What it shows |
|---|---|
| `/memory` | Operator-approved `MemoryItem` rows scoped by workspace |
| `/memory/diff` | Pending `MemorySuggestion` rows with the proposed diff vs current memory |
| `/memory/hygiene` | Stale / unused / contradictory memory flags + bulk archive |
| `/hermes/memory` | What Hermes already has loaded for the current session |
| `/hermes/skills` | Skills Hermes has ingested (`HermesSkill` + install events) |

### Inbox & integrations
| Surface | What it shows |
|---|---|
| `/inbox` | `InboxItem` triage queue — inbound webhooks, signed and verified |
| `/integrations/discord` · `slack` · `email` | Per-integration status, last 20 events, signing-secret check |

### Tools, runtime, media
| Surface | What it shows |
|---|---|
| `/tools` | All 17 registered tools with their Zod input/output schemas + sandbox flags |
| `/runtime` | Active runtime backend — `local` vs `docker` — with isolation guarantees |
| `/runtime/terminal` · `/runtime/python` | Sandbox surfaces for `terminal_exec` + `python_rpc` (require approval to run) |
| `/runtime/policy` · `/runtime/isolation` | The active policy / isolation matrix per tool |
| `/media/web-search` · `vision` · `tts` · `images` | Per-tool playgrounds — wire your providers + run queries |

### Hermes-specific
| Surface | What it shows |
|---|---|
| `/hermes/parity` | Live source-of-truth — every advertised feature with a flag read live from `process.env` |
| `/hermes/commands` · `tools` · `memory` · `skills` | What Hermes is actually configured with right now |
| `/settings/providers` | Per-provider readiness — `READY` / `NEEDS PROVIDER` / `MISCONFIGURED` |

### Operator onboarding
`/onboarding`, `/about`, `/how-it-works`, `/demo`, `/demo/checklist` — plain-English explainers, the four pillars, mission lifecycle, vocabulary, 13-step demo path.

---

## Crews & agents

The repo ships **4 crew templates**, each composed of ordered agent specs from `lib/agents/templates.ts`. Every agent has a Zod schema, a model role (`fast` / `strong` / `judge` / `vision`), and an explicit allow-list of tools.

### Paper Trading Desk (`paper-trading`)
6 agents — researches a thesis, sizes risk, paper-executes, reports.
`market-scout` → `news` → `strategy` → `risk` → `backtest` → `paper-execution`

> **Safety:** every fill writes a `PaperTrade` row with `simulatedOnly = true`. No broker, no API keys, no real money.

### Bug Hunter Crew (`bug-hunter`)
4 agents — picks a flow, audits accessibility + visual + behavioral, files structured bugs.
`explorer` → `flow-tester` → `accessibility` → `bug-reporter`

> **Tools:** `browser_qa_audit` (Playwright + Chromium), SSRF-guarded against `localhost` / private IPs / metadata endpoints in prod.

### Life Admin Crew (`life-admin`)
6 agents — intakes a personal task, gathers evidence, drafts replies, queues approval.
`intake` → `evidence` → `policy` → `draft` → `critic` → `follow-up`

> **Safety:** drafts land as `EMAIL_DRAFT` approvals — nothing sent unless the operator clicks approve.

### Codebase Debugger Crew (`codebase-debugger`)
5 agents — locates the broken file, runs the build, parses the error, plans the fix, reports.
`repo-scout` → `build-runner` → `error-analyst` → `fix-planner` → `report`

> **Tools:** sandboxed `terminal_exec` + `python_rpc` with `--network=none --read-only --cap-drop=ALL` when `RUNTIME_BACKEND=docker`.

### Custom crews
`/crews/new` lets operators compose any sequence of agents from the catalog and ship a Zod schema for the final deliverable. Custom crews land in the `CrewTemplate` table and immediately appear on `/missions/new`.

---

## Tools registry

All 17 tools register through [`lib/tools/registry.ts`](lib/tools/registry.ts) with Zod input/output schemas, an explicit `gated` flag (= requires `ApprovalItem`), and a `provider` requirement (= surfaces `NEEDS PROVIDER` if missing).

| Tool | Purpose | Provider | Gated |
|---|---|---|---|
| `browser_qa_audit` | Playwright Chromium walkthrough — screenshots, axe-core a11y, console errors | bundled | no |
| `terminal_exec` | Shell command in sandboxed runtime | local / docker | yes |
| `python_rpc` | Python 3 RPC against the sandbox | local / docker | yes |
| `web_search` | Tavily → Brave → SerpAPI fallback chain | one of 3 | no |
| `web_snapshot` | Fetch + extract main content of a URL | bundled | no |
| `document_extract` | Pull text from PDF / DOCX / HTML | bundled | no |
| `vision_analyze` | Multimodal image understanding | Gemini / Hermes-vision | no |
| `image_generate` | Image gen — auto-picks cheapest capable model | Hermes / Fal / Replicate | yes |
| `text_to_speech` | ElevenLabs `eleven_multilingual_v2` | ElevenLabs | yes |
| `price_series` | Historical OHLCV for backtests | bundled | no |
| `report_export` | Markdown / PDF deliverable | bundled | yes |
| `deadline_create` | Operator-visible deadline tracker | bundled | no |
| Plus 5 more in [`lib/tools/`](lib/tools/) — see `/tools` for the live registry. | | | |

The runtime backend is settled at request time by [`lib/tools/runtime-backend.ts`](lib/tools/runtime-backend.ts): if `RUNTIME_BACKEND=docker` and `docker version` succeeds, all `terminal_exec` / `python_rpc` calls run inside ephemeral containers; otherwise they fall back to host-local execution and the receipt's `runtimeBackends` block records that fact. Honest fallback, never silent.

---

## Mission lifecycle

```
1. POST /missions/new
   ├─ create Mission (queued | running)
   ├─ create N MissionAgent rows (one per spec in the crew template)
   └─ if MISSION_RUN_MODE=queued → enqueue AgentJob[type=RUN_MISSION]

2. Orchestrator (lib/agents/orchestrator.ts)
   for each MissionAgent in order:
     ├─ build prompt (objective + prior outputs + memory injections)
     ├─ call Hermes with the agent's modelRole
     ├─ persist AgentRunStep (prompt, raw, parsed, tokens, cost, model)
     ├─ Zod-validate output
     │   └─ on parse failure → re-prompt with structured error (max 2 retries)
     └─ for each declared tool call:
         ├─ build ToolCall row
         ├─ check gated → create ApprovalItem if needed (mission pauses)
         ├─ execute via runtime backend
         └─ persist ToolCall result + cost

3. Settle
   ├─ JUDGE model reviews timeline
   ├─ writes up to 3 Skill rows scoped to the crew
   └─ produces WorkflowReceipt
       ├─ SHA-256 hash over (steps, tools, approvals, memory)
       ├─ real cost = Σ(tokens × per-model rates from /settings/providers)
       └─ trust ledger update

4. Operator
   /receipts/[id] — replay the run
   /approvals    — clear gated items
   /memory/diff  — review proposed memory updates
```

Every row above is a Prisma model. Open `npm run db:studio` to walk any past run.

---

## Data model

The schema lives in [`prisma/schema.prisma`](prisma/schema.prisma). Grouped by concern:

**Mission graph**
`Mission` → `MissionAgent[]` → `AgentRunStep[]` → `ToolCall[]` · `ApprovalItem[]` · `Deliverable[]` · `DocumentItem[]`

**Receipts & trust**
`WorkflowReceipt` (one per settled mission) → `ReceiptEvent[]` (the immutable timeline)

**Memory system**
`MemoryItem` (operator-approved) ← `MemorySuggestion` (pending diffs) · `MemoryUsage` (which step recalled which memory) · `MemoryChange` (audit log)

**Skills (learning loop)**
`Skill` (crew-scoped, written by the JUDGE) · `SkillRun` (each invocation) · `HermesSkill` + `HermesSkillInstallEvent` (skills shared with the Hermes engine)

**Operations**
`AgentJob` (the queue) · `ScheduledMission` (cron-like cadences) · `InboxItem` (inbound webhooks) · `AuditEvent` (operator actions)

**Sandbox & safety**
`SandboxToolRun` (every gated tool invocation) · `PaperTrade` (`simulatedOnly = true` enforced at the row level)

**Workspace**
`UserProfile` · `CrewTemplate` (custom crews live here)

---

## API surface

REST routes under [`app/api/`](app/api/):

| Route | Method | Purpose |
|---|---|---|
| `/api/hermes/health` | GET | Provider probe — `{ ok, mode, latencyMs }`; `?force=1` bypasses cache |
| `/api/hermes/status` | GET | Per-tool readiness — feeds `/settings/providers` |
| `/api/jobs/run-due` | POST | Process every due `AgentJob` once (idempotent) |
| `/api/jobs/[id]` | GET / DELETE | Inspect / cancel a queued job |
| `/api/schedules` | GET / POST | List + create scheduled missions |
| `/api/schedules/[id]` | GET / PATCH / DELETE | Manage a schedule |
| `/api/schedules/[id]/run-now` | POST | Force-fire a schedule outside its cadence |
| `/api/schedules/run-due` | POST | Fire every schedule whose `nextRunAt` ≤ now |
| `/api/automation/tick` | POST | Single worker tick (used by external cron / Railway) |
| `/api/integrations/webhook` | POST | Generic HMAC-signed inbound — creates `InboxItem` |
| `/api/integrations/discord` | POST | Discord-flavored signed webhook |
| `/api/integrations/slack` | POST | Slack signing-secret verified |
| `/api/integrations/email` | POST | Email-as-webhook (e.g. SendGrid Inbound Parse) |
| `/api/receipts/[id]/export` | GET | Signed export of a `WorkflowReceipt` (JSON + integrity hash) |
| `/api/demo/seed` · `/api/demo/reset` | POST | Demo workspace lifecycle |

Every webhook endpoint enforces HMAC if its secret env var is set; missing secrets surface as `MISCONFIGURED` on `/settings/providers` rather than silently accepting traffic.

---

## Memory & skills lifecycle

**Memory** is operator-governed, never implicit:

```
agent run                       operator review                next run
   │                                  │                            │
   ├─► proposes change ────────► MemorySuggestion ──► approve ──► MemoryItem
   │       (with diff)                │                            │
   │                                  └──► reject (audit logged)   │
   │                                                               │
   └────────────────────── recall on next mission ◄────────────────┘
                              (writes MemoryUsage)
```

Every recall writes a `MemoryUsage` row tying the memory back to the agent step that consumed it — so the receipt shows exactly which prior knowledge influenced this run.

**Skills** are the learning loop. After every settle, the JUDGE model reviews the timeline and distils up to 3 reusable lessons into `Skill` rows scoped to the crew. On the next run for that crew, those skills are injected into the orchestrator prompt — the crew literally gets better at its own job.

Skills can be promoted to `HermesSkill` and shared with the Hermes engine itself via `HermesSkillInstallEvent`, persisting learned behavior across the whole agent ecosystem.

---

## Worker & scheduling

`scripts/worker.ts` is a long-lived poller that:

1. Claims due `AgentJob` rows with `SELECT ... FOR UPDATE SKIP LOCKED`-style optimistic locking — concurrent workers won't double-execute.
2. Runs the job (`RUN_MISSION`, `TRIAGE_INBOX`, `SCHEDULE_TICK`).
3. On success: marks complete + records cost.
4. On failure: increments `attempts`, writes error, schedules retry with exponential backoff (capped).
5. Emits a structured tick log every poll (visible on `/jobs`).

Schedules:

| Cadence | Behavior |
|---|---|
| `DAILY` / `WEEKDAYS` | Fires at the configured hour every day (or Mon–Fri only) |
| `WEEKLY` | Fires on the configured weekday |
| `MONTHLY` | Fires on the configured day-of-month (clamps to month length) |
| `ONCE` | Fires once at `nextRunAt` then disables itself |

Schedules accept natural-language input on `/schedules` ("every weekday at 9am", "first of every month") — parsed locally, no LLM call.

---

## Tech stack

<table>
<tr>
  <td><strong>App</strong></td>
  <td>Next.js 14 (App Router) · React 18 · TypeScript 5.6 · Tailwind CSS 3.4</td>
</tr>
<tr>
  <td><strong>Data</strong></td>
  <td>Prisma 5 · Postgres (Neon / Supabase / Railway) · Zod 3</td>
</tr>
<tr>
  <td><strong>AI runtime</strong></td>
  <td>Hermes Agent · OpenRouter (chat completions) · Gemini 2.5 Flash (vision + image) · Tavily (search) · ElevenLabs (TTS) · MCP (Model Context Protocol)</td>
</tr>
<tr>
  <td><strong>Tooling</strong></td>
  <td>Playwright (Chromium) · Docker runtime backend · tsx CLI</td>
</tr>
<tr>
  <td><strong>Deploy</strong></td>
  <td>Vercel (web) · Railway (Postgres + worker)</td>
</tr>
</table>

---

## Configuration

Full template in [`.env.example`](.env.example). The 4 you cannot skip:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Postgres connection string |
| `HERMES_API_BASE` | `https://openrouter.ai/api/v1` (or your Hermes endpoint) |
| `HERMES_API_KEY` | OpenRouter key |
| `HERMES_MODEL` | e.g. `nousresearch/hermes-4-70b` |

Provider keys (each missing → the matching tool surfaces `NEEDS PROVIDER`, never fakes a result):
`TAVILY_API_KEY` · `BRAVE_SEARCH_API_KEY` · `SERPAPI_API_KEY` · `VISION_MODEL` · `FAL_KEY` · `REPLICATE_API_TOKEN` · `IMAGE_MODEL` · `ELEVENLABS_API_KEY` · `ELEVENLABS_VOICE_ID` · `DISCORD_WEBHOOK_URL` · `DISCORD_BOT_TOKEN` + `DISCORD_CHANNEL_ID` · `MCP_SERVERS` (JSON array) · `INTEGRATIONS_WEBHOOK_SECRET` · `DISCORD_WEBHOOK_SECRET` · `SLACK_SIGNING_SECRET` · `EMAIL_WEBHOOK_SECRET`

Operational:
`MISSION_RUN_MODE` (`sync` | `queued`) · `RUNTIME_BACKEND` (`local` | `docker`) · `WORKER_POLL_INTERVAL_MS` · `OPENROUTER_SITE_URL` · `OPENROUTER_APP_NAME`

Live status of each provider in your environment: [`/settings/providers`](https://hermes-agent-mocha-five.vercel.app/settings/providers).

---

## Deploy

### Vercel (web tier)
```bash
# 1. Import the repo on Vercel
# 2. Set the env vars above (Postgres URL must NOT start with file:)
# 3. Build command stays the default: npm run build
# 4. Run npx prisma db push once against the prod DB
```
Full walkthrough: [`docs/deployment/vercel.md`](docs/deployment/vercel.md)

### Railway (recommended — web + worker + Postgres)
```bash
# 1. + New → Database → Postgres
# 2. + New → Empty Service → connect this repo
# 3. railway.json (already in the repo) wires:
#    - preDeployCommand: npx prisma db push --accept-data-loss
#    - startCommand:     npm run start
#    - healthcheck:      /api/hermes/health
```
Full walkthrough: [`docs/deployment/railway.md`](docs/deployment/railway.md)

---

## CLI

```bash
npm run foundry -- health                 # probe Hermes — exit 0 iff healthy
npm run foundry -- jobs run-due           # process every due job once
npm run foundry -- jobs list --limit 10   # recent agent jobs as JSON
npm run foundry -- receipts list          # recent workflow receipts as JSON
npm run foundry -- mission create --crew bug-hunter --title "Audit demo" --objective "..."
npm run foundry -- evals tools            # cheap tool gating checks
npm run foundry -- evals                  # real Hermes mission suite
```

---

## Verification gates

Every push to `main` must pass:
```bash
npm run typecheck      # tsc --noEmit
npm run lint           # next lint
npm run build          # prisma generate && next build
npm run evals:tools    # 7/7 gating + metadata checks
```
CI runs the same in [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

---

## Documentation

- [`docs/deployment/vercel.md`](docs/deployment/vercel.md) — Vercel recipe + env-var matrix
- [`docs/deployment/railway.md`](docs/deployment/railway.md) — Railway recipe with worker
- [`/about`](https://hermes-agent-mocha-five.vercel.app/about) — plain-English explanation, the four pillars, mission lifecycle, vocabulary
- [`/how-it-works`](https://hermes-agent-mocha-five.vercel.app/how-it-works) — what each surface does, what to click to test
- [`/hermes/parity`](https://hermes-agent-mocha-five.vercel.app/hermes/parity) — live source-of-truth on shipped vs roadmap
- [`/demo/checklist`](https://hermes-agent-mocha-five.vercel.app/demo/checklist) — 13-step operator demo path

---

## Safety guarantees

- ❌ **No real trading** — Paper Trading Desk creates `PaperTrade` rows with `simulatedOnly = true`. No broker integration.
- ❌ **No automatic emails** — Life Admin produces `EMAIL_DRAFT` approvals only. Nothing sent.
- ❌ **No silent fallbacks** — when Hermes env vars are configured, provider failures are real errors, never a fake-data cover.
- 🛡 **SSRF guard** on `browser_qa_audit` blocks localhost / private IPs / metadata endpoints by default.
- 🛡 **Approval-gated tools** refuse to run from the sandbox surface.
- 🛡 **Every prompt + raw response + parsed output** is persisted as a database row for audit.

---

## Contributing

Pull requests welcome. The bar:
1. `npm run typecheck && npm run lint && npm run build` must pass
2. New tools register through `lib/tools/registry.ts` and pass Zod validation
3. New surfaces respect the [`/settings/providers`](https://hermes-agent-mocha-five.vercel.app/settings/providers) source-of-truth pattern — never fake green
4. Receipt-relevant changes update `lib/receipts/` and pass `npm run evals`

---

## License

MIT — see [`LICENSE`](LICENSE).

Hermes Agent is independently MIT-licensed by Nous Research at [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent).

---

<div align="center">

**Hermes does the autonomy. Hermes Loop adds the proof + control.**

[Website](https://hermesloop.cc) · [X / @HermesLoop](https://x.com/HermesLoop)

</div>
