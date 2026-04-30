<div align="center">

<img src="https://static.wixstatic.com/media/e2da02_2be72852d6ae4857a5b78c5c4cd53e0d~mv2.png" alt="Hermes Loop" width="420" />

### The control room for the Hermes Agent.
**Persistent agent crews. Hashed receipts. Approval gates. Honest providers.**

[![CI](https://img.shields.io/github/actions/workflow/status/ctrlshifthash/HermesAgent/ci.yml?branch=main&label=CI&style=flat-square&logo=github&logoColor=white&color=22c55e)](https://github.com/ctrlshifthash/HermesAgent/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-A78BFA.svg?style=flat-square)](LICENSE)
[![Next.js 14](https://img.shields.io/badge/Next.js-14.2-000?style=flat-square&logo=next.js&logoColor=white)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Prisma](https://img.shields.io/badge/Prisma-Postgres-2D3748?style=flat-square&logo=prisma&logoColor=white)](https://www.prisma.io/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3.4-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)
[![Zod](https://img.shields.io/badge/Zod-validated-3068B7?style=flat-square)](https://zod.dev)
[![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-MIT-A78BFA?style=flat-square)](https://github.com/NousResearch/hermes-agent)
[![MCP](https://img.shields.io/badge/MCP-native-22d3ee?style=flat-square)](https://modelcontextprotocol.io)
[![Deploy on Vercel](https://img.shields.io/badge/Deploy-Vercel-000?style=flat-square&logo=vercel&logoColor=white)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fctrlshifthash%2FHermesAgent)
[![Run on Railway](https://img.shields.io/badge/Run-Railway-0B0D0E?style=flat-square&logo=railway&logoColor=white)](https://railway.app/template/new?template=https%3A%2F%2Fgithub.com%2Fctrlshifthash%2FHermesAgent)

[Live demo](https://hermes-agent-mocha-five.vercel.app) · [Documentation](docs/) · [Architecture](#architecture) · [Deploy](#deploy)

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

[Live demo](https://hermes-agent-mocha-five.vercel.app) · [Open the desk](https://hermes-agent-mocha-five.vercel.app/dashboard) · [Parity board](https://hermes-agent-mocha-five.vercel.app/hermes/parity)

</div>
