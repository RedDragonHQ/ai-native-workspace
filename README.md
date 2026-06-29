# AI-Native Workspace

> **A layered framework for turning any AI coding agent into a personal operating system.**
>
> Works with Claude Code, Codex, Cursor, or any agent that can read a markdown file.

**🌏 Non-English readers**: Chinese versions of every document live in [`zh/`](zh/) — [`zh/README.md`](zh/README.md) · [`zh/ARCHITECTURE.md`](zh/ARCHITECTURE.md) · [`zh/BOOTSTRAP.md`](zh/BOOTSTRAP.md) · [`zh/examples/sample-workspace/`](zh/examples/sample-workspace/).

**This is not another `.cursorrules` template.** It's an opinionated architecture for running a real research/engineering workspace where the AI agent is the primary operator — and humans are the fallback when agents aren't available.

---

## Why this framework

Most "Claude Code workflow" repos fall into one of two camps:

- **Dotfiles dumps** — my `.claude/` config, my rules, my prompts. Hard to generalize.
- **Single-ecosystem lock-in** — designed for one agent + one model + one provider. Breaks the day you want to switch.

This framework is built on a different premise:

> **The workspace is the OS. The agent is the process. The model is the kernel.**
>
> You can swap any layer without rewriting the others.

### Multi-layer decoupling — the actual differentiator

| Layer | Single-ecosystem approach | This framework |
|-------|---------------------------|----------------|
| **Model** | Claude-only (or GPT-only) | DeepSeek, Claude, Qwen, OpenAI, Gemini — same `.env` swap |
| **Agent** | Claude Code only | Claude Code, Codex CLI, Cursor, custom LangGraph agent |
| **Protocol** | Provider-specific (Anthropic MCP, OpenAI function calling) | File-based CLAUDE.md + CLI + MCP (optional) |
| **Application** | Vendor templates | Your projects, your PKM, your workflow |

**Why this matters**: if Anthropic ships a better agent next year, you don't rewrite your workspace. If DeepSeek undercuts Claude on price, you don't rewrite your workflow. If you outgrow MCP, you don't lose your knowledge base.

The framework is **protocol-based**, not **product-based**.

### Real-world proof, not theory

AI-Native Workspace was extracted from a live multi-project workspace, not invented as a clean-room template. The production version has been used for 6+ months across ML research, multi-agent systems, infrastructure, security research, content sites, and PKM.

The public repo keeps only the reusable architecture, but the original workspace stress-tested the core ideas daily:

- layered `CLAUDE.md` files for global → workspace → project cognition
- `inbox/TODO.md` as cross-session working memory
- `wiki/MANIFEST.md` for progressive context loading instead of context dumps
- local CLI/MCP services as protocol extensions, not hard dependencies
- model/provider swaps without rewriting the application layer

In short: **if the agent disappears, the workspace still works** — because the durable state lives in plain files and explicit conventions.

### AI-first, human-readable

CLAUDE.md is **optimized for agent consumption** — structured so an AI agent can load it and start working immediately. But it's not agent-exclusive: the same file is fully human-readable, and when agents aren't available, a human can step into the agent layer and operate the workspace using the same protocols.

- Layered cognitive loading: global → workspace → project
- Progressive context: read the index first, dive into details on demand
- Structured continuity: cross-session state lives in CLAUDE.md and project docs, not in chat history
- Workflow contracts: session-end protocols, recovery protocols, audit trails

Humans don't just get side benefits — they're the **fallback operator**. When the agent layer fails (API down, model degraded, edge case the agent can't handle), the human reads the same CLAUDE.md and picks up where the agent left off.

---

## The 4-layer architecture

```
┌─────────────────────────────────────────────────────────┐
│  Application                                            │
│  Projects · Inbox (draft buffer) · Wiki (matured KB)    │
├─────────────────────────────────────────────────────────┤
│  Protocol                                               │
│  CLAUDE.md conventions · CLI tools · MCP Servers        │
├─────────────────────────────────────────────────────────┤
│  Agent                                                  │
│  Claude Code · Codex · Cursor · any markdown-aware agent│
├─────────────────────────────────────────────────────────┤
│  Model                                                  │
│  DeepSeek · Claude · Qwen · OpenAI · local              │
└─────────────────────────────────────────────────────────┘
```

Each layer has a single responsibility. The boundaries are enforced by **file conventions**, not code — which is why any agent that can read markdown can run this framework.

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full design.

---

## What's in this repo

```
ai-native-workspace/
├── README.md                  ← You are here (English)
├── zh/README.md               ← 中文版
├── ARCHITECTURE.md            ← 4-layer design, trade-offs, anti-patterns (English)
├── zh/ARCHITECTURE.md         ← 中文版
├── BOOTSTRAP.md               ← Agent init instructions (English)
├── zh/BOOTSTRAP.md            ← 中文版 (agent 初始化指令)
├── examples/
│   └── sample-workspace/      ← Minimal runnable workspace (English)
├── zh/examples/
│   └── sample-workspace/      ← 中文版
└── LICENSE
```

**`BOOTSTRAP.md` is the interesting file.** It's the document an AI agent reads to set up a fresh workspace from scratch. Paste it into Claude Code / Codex / Cursor in an empty directory and the agent will build the structure for you. Chinese users: use [`zh/BOOTSTRAP.md`](zh/BOOTSTRAP.md) instead.

---

## Quick start

### Option A: let an agent bootstrap it

1. Create a new directory for your workspace.
2. Open Claude Code / Codex / Cursor in that directory.
3. Paste the contents of [`BOOTSTRAP.md`](BOOTSTRAP.md) as your first prompt.
4. The agent will scan your directory and build the structure.

### Option B: copy the example

```bash
cp -r examples/sample-workspace ~/my-workspace
cd ~/my-workspace
$EDITOR CLAUDE.md          # adjust to your context
```

Chinese version:

```bash
cp -r zh/examples/sample-workspace ~/my-workspace
cd ~/my-workspace
$EDITOR CLAUDE.md
```

### Then what?

Start using it. The framework reveals itself through use:

- **Day 1**: you write `inbox/YYYY-MM-DD-thing.md` at end of session
- **Week 2**: `inbox/TODO.md` becomes your cross-session state
- **Month 2**: `wiki/` emerges when inbox gets crowded
- **Month 3+**: CLAUDE.md and project docs accumulate persistent facts about your preferences

No setup ceremony. The architecture is the daily practice, not the installation.

---

## Key ideas (TL;DR)

1. **Inbox is the buffer.** Low-friction capture. Anything can land in `inbox/` — you sort later.
2. **Wiki is the sediment.** When inbox items get referenced 3+ times, they graduate to `wiki/`.
3. **MANIFEST is the index.** Don't read the whole wiki. Read MANIFEST.md (one-line summaries), then dive.
4. **Anti-patterns are the guardrails.** The framework is defined as much by what it forbids as what it enables.

---

## Anti-patterns (a taste)

| ❌ Don't | ✅ Do |
|---------|------|
| Start with multi-agent | Single agent; split only when context is genuinely exhausted |
| Load all workspace files at boot | Read the index, drill into what you need |
| Skip inbox and write straight to wiki | Inbox is the buffer; let sediment form naturally |
| Write a 500-line CLAUDE.md | ≤ 100 lines; details go in project CLAUDE.md |
| Treat TODO.md as a command queue | It's a draft — validate before acting |
| Auto-write inbox files without confirmation | Prompt the user; don't surprise them |

See [ARCHITECTURE.md](ARCHITECTURE.md#anti-patterns) for the full list with rationale.

---

## When to use this

**Use it if:**

- You work on multiple projects with an AI coding agent and context loss is a real problem
- You've tried `.cursorrules` / `.claude/CLAUDE.md` but they don't scale past one project
- You want a framework that survives switching models/agents/providers
- You're doing research or infrastructure work where knowledge accumulates across months

**Don't use it if:**

- You're working on a single short-term project (a simple CLAUDE.md is enough)
- You don't need cross-session memory (just use chat history)
- You're not using an AI agent daily (the framework rewards consistent practice)

---

## Origins

This framework was extracted from a real research workspace running 8+ projects (ML research, multi-agent systems, infrastructure, security research, PKM) over 6+ months of daily use. It was refined through repeated failures — the anti-patterns list is longer than the features list because each one represents a real mistake.

The workspace is called **RedDragon Workspace** internally. This release strips the personal content and keeps the architecture.

---

## Contributing

Issues and PRs welcome. Especially:

- Adapters for other agents (Codex, Aider, Continue, etc.)
- Example workspaces for specific domains (research, SWE, content)
- Translations of BOOTSTRAP.md and ARCHITECTURE.md

---

## License

MIT
