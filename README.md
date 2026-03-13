# Open Brain

A self-hosted, MCP-connected persistent memory layer for AI. One database. One open protocol. Every AI you use shares the same context.

---

## Getting Started

**What you need:** ~45 minutes, free accounts on three services, no coding experience required.

**Step 1 — Create accounts (free tier on all three):**
- [Supabase](https://supabase.com) — your database
- [OpenRouter](https://openrouter.ai) — your AI gateway (add $5 in credits, lasts months)
- [Slack](https://slack.com) — your capture interface

**Step 2 — Clone this repo and open it in Claude Code:**
```bash
git clone https://github.com/davidthingsbot/openbrain.git
cd openbrain
claude
```

**Step 3 — Point Claude Code at the build guide:**
```
Build the Open Brain system described in the README and at
https://promptkit.natebjones.com/20260224_uq1_guide_main

Start with Phase 1: Supabase setup, then the ingest-thought Edge Function,
then the MCP server. I have accounts on Supabase, OpenRouter, and Slack ready to go.
```

Claude Code will walk through the entire build. The guide has every SQL command, every line of Edge Function code, and every config step — it's designed to be handed directly to an AI.

**Step 4 — Connect your AI clients** (Phase 2 in the plan below) and start capturing thoughts in Slack.

> ⚠️ Keep your credentials (Supabase secret key, OpenRouter key, Slack bot token, MCP access key) in a local file outside the repo. Never commit them.

---

## The Problem

Every AI tool starts from zero. Claude doesn't know what ChatGPT knows. Your coding agent doesn't know your constraints. You burn your best thinking on re-explaining context instead of real work — and the corporate "memory" features are designed as lock-in, not as infrastructure you own.

## The Architecture

```
Capture (Slack / any MCP client)
        │
        ▼
  Edge Function
  ┌─────────────────────────────┐
  │  embed → pgvector           │
  │  extract metadata (LLM)     │
  │  store → Supabase/Postgres  │
  └─────────────────────────────┘
        │
        ▼
  MCP Server (hosted Edge Function)
        │
        ├── search_thoughts  (semantic search by meaning)
        ├── browse_recent    (last N captures)
        ├── stats            (patterns, counts)
        └── capture_thought  (write from any AI)
        │
        ▼
  Any MCP client: Claude Desktop, ChatGPT, Cursor,
                  Claude Code, VS Code Copilot, etc.
```

**Stack:**
- **Supabase** — Postgres + pgvector + Edge Functions (free tier)
- **OpenRouter** — embeddings (text-embedding-3-small) + metadata LLM (gpt-4o-mini)
- **Slack** — capture interface (free tier)

**Cost:** ~$0.10–0.30/month at 20 captures/day

---

## Implementation Plan

### Phase 1 — Core Infrastructure (Week 1)

#### 1.1 Supabase Setup
- [ ] Create Supabase project
- [ ] Enable pgvector extension
- [ ] Create `thoughts` table (id, content, embedding 1536-dim, metadata JSONB, created_at, updated_at)
- [ ] Create `match_thoughts` semantic search function (cosine similarity)
- [ ] Lock down with Row Level Security (service role only)

#### 1.2 OpenRouter
- [ ] Create account + API key
- [ ] Add $5 credits (lasts months at this volume)
- [ ] Confirm `text-embedding-3-small` and `gpt-4o-mini` access

#### 1.3 Capture — Slack → Supabase Edge Function
- [ ] Create Slack workspace + private `#capture` channel
- [ ] Create Slack app with `channels:history`, `groups:history`, `chat:write` scopes
- [ ] Write `ingest-thought` Edge Function:
  - Receive Slack event webhook
  - Generate embedding via OpenRouter (parallel)
  - Extract metadata via LLM: type, topics, people, action_items
  - Store both in Supabase
  - Reply in-thread with capture confirmation
- [ ] Deploy to Supabase Edge Functions
- [ ] Wire Slack Event Subscriptions → Edge Function URL
- [ ] Test end-to-end capture flow

#### 1.4 Retrieval — MCP Server Edge Function
- [ ] Write `open-brain-mcp` Edge Function using `@hono/mcp` + MCP SDK:
  - `search_thoughts` — semantic similarity search with threshold
  - `browse_recent` — last N thoughts with optional filter
  - `stats` — counts, top people, top topics
  - `capture_thought` — write directly from any AI
- [ ] Deploy to Supabase
- [ ] Secure with access key (`?key=` URL param + `x-brain-key` header)
- [ ] Test MCP connection from Claude Desktop

---

### Phase 2 — Client Connections (Week 1–2)

- [ ] **Claude Desktop** — Settings → Connectors → Add custom connector (remote MCP URL)
- [ ] **ChatGPT** — Developer Mode + Apps & Connectors
- [ ] **Claude Code** — `claude mcp add --transport http`
- [ ] **Cursor / VS Code** — `mcp-remote` bridge via JSON config
- [ ] Verify cross-tool reads: capture in Slack, retrieve in Claude, retrieve in ChatGPT

---

### Phase 3 — Capture Habits & Prompts (Week 2)

Build the four companion workflows:

- [ ] **Memory Migration** — prompt to extract existing AI memories (Claude memory, ChatGPT memory) and bulk-import into Open Brain
- [ ] **Open Brain Spark** — interview prompt to generate a personalised "first 20 captures" list based on actual workflow
- [ ] **Quick Capture Templates** — 5 starters for clean metadata extraction: decision, person note, insight, meeting debrief, idea
- [ ] **Weekly Review** — Friday synthesis prompt: cluster topics, surface forgotten action items, find cross-week connections

---

### Phase 4 — Extensions (Ongoing)

Potential additions once the core is solid:

- **Alternative capture sources** — beyond Slack: Signal bot, web clipper, voice-to-thought pipeline
- **Agent integration** — give autonomous agents read/write access to Open Brain as persistent working memory
- **Visualisation** — thinking pattern dashboard built from the MCP data
- **Daily digest** — surface forgotten ideas based on what you're currently working on
- **Multi-user** — shared brain for a team (separate RLS policies per user)
- **Local-first option** — swap Supabase for a self-hosted Postgres for full offline control

---

## References

- [Nate Jones — "You Don't Need SaaS" video](https://www.youtube.com/watch?v=2JiMmye2ezg)
- [Full setup guide (promptkit.natebjones.com)](https://promptkit.natebjones.com/20260224_uq1_guide_main)
- [Supabase MCP docs](https://supabase.com/docs/guides/getting-started/byo-mcp)
- [OpenRouter](https://openrouter.ai)
- [MCP SDK](https://github.com/modelcontextprotocol/typescript-sdk)
