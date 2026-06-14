# Architecture

> **How the 4 layers fit together, why the boundaries matter, and what breaks when you ignore them.**

This document is for humans who want to understand the design. If you're an agent being asked to set up a workspace, read [`BOOTSTRAP.md`](BOOTSTRAP.md) instead.

---

## The core claim

> An AI agent's workspace is an operating system. The agent is a process. The model is the kernel. The protocols are syscalls. The applications are user programs.

This metaphor isn't poetic — it's the design constraint. Every layer has a single responsibility and a well-defined interface to the layers above and below. Violate the boundary and you get lock-in, fragility, or both.

---

## The 4 layers, in detail

### Layer 4: Application

**What lives here**: your projects, your inbox, your wiki.

```
workspace/
├── CLAUDE.md              ← Workspace-level cognitive config
├── inbox/                 ← Draft buffer
│   ├── README.md          ← Inbox file format
│   └── TODO.md            ← Cross-session running list
├── projects/              ← Concrete work
│   ├── project-a/
│   │   └── CLAUDE.md      ← Project-level cognitive config
│   └── project-b/
│       └── CLAUDE.md
└── wiki/                  ← Matured knowledge base
    └── MANIFEST.md        ← Progressive-loading index
```

**Single responsibility**: hold the work product and its accumulated knowledge.

**Boundary with layer below (Protocol)**: the application layer only interacts with the outside world through protocols defined in layer 3. It never calls model APIs directly.

**Core design: the inbox buffer**

Most knowledge management frameworks assume you know where information belongs when you capture it. That's false. You almost never know — is this a bug report, a design decision, a meeting note, or just noise?

The inbox says: **capture first, classify later**. An inbox file has low production cost (a few lines) and low commitment (you can delete it tomorrow). Only when an inbox item gets referenced 3+ times does it graduate to `wiki/`.

This is the same principle as David Allen's GTD capture step, applied to AI-agent workflow: **reduce friction at the point of capture**.

**Core design: progressive loading via MANIFEST**

Context windows are finite. A workspace with 200 wiki entries can't all be in context. The solution: `MANIFEST.md` is an index with one-line summaries per file. The agent reads MANIFEST first (200 lines), picks 2-5 relevant files based on the summaries, and loads only those.

This is the same principle as a database index or a library card catalog: **scan cheap, fetch expensive**.

### Layer 3: Protocol

**What lives here**: the conventions, CLI tools, and MCP servers that let the agent interact with the world.

```
Protocol types:
┌─────────────────────────────────────────────┐
│  File protocol (CLAUDE.md conventions)       │ ← always present
│  CLI tools (git, curl, docker, node, python) │ ← usually present
│  MCP servers (SearXNG, custom, etc.)         │ ← optional, added as needed
└─────────────────────────────────────────────┘
```

**Single responsibility**: define how the agent interacts with the outside world.

**Boundary with layer below (Agent)**: the agent uses protocols but doesn't implement them. Swap the agent, keep the protocols.

**Boundary with layer above (Application)**: the application layer only uses protocols to interact with external systems. It doesn't know whether those protocols are served by MCP, a CLI tool, or a curl wrapper.

**Core design: file protocol as the lowest common denominator**

MCP is powerful but optional. CLI tools are powerful but not always available. **A markdown file is always readable.** The framework's core configuration (CLAUDE.md) is a markdown file — which means any agent that can read text can at least partially run the framework, even with zero integrations.

This is the Postel's law of AI agent frameworks: **be liberal in what you accept**.

### Layer 2: Agent

**What lives here**: the software that reads CLAUDE.md and decides what to do next.

Currently tested with:
- Claude Code (Anthropic) — most mature integration
- Codex CLI (OpenAI) — works with minor CLAUDE.md adjustments
- Cursor — works via `.cursorrules` adaptation
- Custom LangGraph agents — works with custom CLAUDE.md parser

**Single responsibility**: reasoning, planning, tool orchestration.

**Boundary with layer below (Model)**: the agent calls model APIs but doesn't care which model responds. Same prompt, different backend.

**Boundary with layer above (Protocol)**: the agent uses protocols but doesn't own them. Swap the agent, keep the protocols.

**Core design: agent-agnostic configuration**

Most workflow frameworks are designed for one agent. Anthropic's official docs assume Claude Code. Cursor rules assume Cursor. This framework's CLAUDE.md is designed to be readable by any agent that can parse markdown — which is every current agent and most future ones.

The name "CLAUDE.md" is a convention from Anthropic but the content is agent-agnostic. If your agent uses a different config filename (`.cursorrules`, `AGENTS.md`, etc.), just copy the content.

### Layer 1: Model

**What lives here**: the LLM that powers the agent's reasoning.

```
Model providers (tested):
├── DeepSeek (Anthropic-compatible API) — daily driver, low cost
├── Anthropic Claude (direct or via Bedrock/Vertex)
├── Qwen (Alibaba, Anthropic-compatible via DashScope)
├── OpenAI GPT family
├── Local models (ollama, vLLM)
└── Third-party relays (OpenRouter, etc.)
```

**Single responsibility**: reasoning capability.

**Boundary with layer above (Agent)**: the model receives prompts and returns completions. It doesn't know about projects, inboxes, or wikis.

**Core design: model swap as an environment variable**

Because the framework uses Anthropic-compatible API conventions across providers, swapping models is:

```bash
# Daily: cheap and fast
source daily.sh     # sets ANTHROPIC_BASE_URL=deepseek

# Heavy task: strong reasoning
source strong.sh    # sets ANTHROPIC_BASE_URL=claude-relay
```

Same agent, same workspace, same workflow. Different model. The model is **the most disposable layer** — which is the point, because model capabilities and pricing change every quarter.

---

## Why multi-layer decoupling matters

The single-ecosystem approach (one agent + one model + one provider) works fine when:

- You're happy with the current provider forever
- You're building for one use case
- You don't care about portability

It breaks when:

- **Your provider raises prices** (happened to OpenAI in 2024, will happen again)
- **Your provider's quality drops** (happened to several in 2025)
- **A better model ships** (happens every 6 months)
- **Your use case expands** (you start with SWE, then add research, then add content)
- **Your agent of choice discontinues a feature** (Cursor has pivoted multiple times)

The 4-layer framework costs a little more setup. It pays back in:

| Year | Single-ecosystem | Multi-layer |
|------|------------------|-------------|
| Year 1 | Easy start, easy progress | Slightly more setup, easy progress |
| Year 2 | Easy progress | Easy progress |
| Year 3 | Stuck on old provider; rewrite to switch | Swap a layer; keep the rest |
| Year 4 | Locked in | Portable |

This is the standard build-vs-buy calculus, applied to AI workflows.

---

## Anti-patterns (full list)

Anti-patterns define the framework as much as features do. Each one below represents a real failure in the workspace this framework was extracted from.

### Agent layer

| ❌ Don't | ✅ Do | Why |
|---------|------|-----|
| Start with multi-agent | Single agent; split only when context is genuinely exhausted | Multi-agent adds coordination overhead that 90% of workloads don't need |
| Load all workspace files at boot | Read the index, drill into what you need | Context windows are finite; loading everything wastes them |
| Write a 500-line CLAUDE.md | ≤ 100 lines; details go in project CLAUDE.md | The root CLAUDE.md is the table of contents, not the book |
| Auto-write inbox files without confirmation | Prompt the user; don't surprise them | Writing is a commitment; surprise writes create debt |
| Treat TODO.md as a command queue | It's a draft — validate before acting | TODOs accumulate cruft; unvalidated execution amplifies mistakes |

### Protocol layer

| ❌ Don't | ✅ Do | Why |
|---------|------|-----|
| Require MCP for core workflow | File protocol is the floor; MCP is the ceiling | MCP is great but optional; your core workflow shouldn't depend on a running server |
| Build a custom CLI for everything | Use git/curl/docker first | Reinventing standard tools wastes time and confuses collaborators |
| Expose raw API keys in scripts | Use env files + env-var swaps | Secrets belong in env, not in source |

### Application layer

| ❌ Don't | ✅ Do | Why |
|---------|------|-----|
| Skip inbox and write straight to wiki | Inbox is the buffer; let sediment form naturally | Premature wiki entries create orphaned, half-baked docs |
| Scatter facts across random files | Put stable facts in dedicated project docs or wiki | CLAUDE.md is for workflow; facts belong in stable docs |
| Keep TODOs forever | Archive or delete stale TODOs weekly | A TODO list that never shrinks is a guilt list |
| Let one project dominate the root CLAUDE.md | Move project detail to project CLAUDE.md | Root CLAUDE.md is cross-project; project CLAUDE.md is intra-project |

### Model layer

| ❌ Don't | ✅ Do | Why |
|---------|------|-----|
| Hard-code one model | Use env-var swaps | Models change quarterly; env swaps are free |
| Assume model quality is stable | Test on real tasks regularly | Regressions happen; silent degradation is the worst kind |
| Treat cost as a fixed | Track cost per task type | Cheap models are 80% as good for 5% of the cost on many tasks |

---

## When the framework doesn't apply

This framework is overkill for:

- **Single short-term projects** — a simple CLAUDE.md is enough
- **One-off tasks** — no cross-session state means no need for inbox/wiki
- **Non-agent workflows** — if you're not using an AI agent daily, the framework's daily practices don't pay off
- **Teams with strong existing conventions** — if your team has a working workflow, don't replace it for theory

The framework pays off when:

- **You're using an AI agent daily** — the practices compound
- **You work across multiple projects** — cross-project coordination is where inbox/MANIFEST shine
- **Your work accumulates knowledge** — research, infrastructure, PKM
- **You care about portability** — you don't want to be locked into one provider

---

## Origins and lineage

This framework was extracted from **RedDragon Workspace**, a real research workspace running since 2025 with:

- 8+ active projects across ML research, multi-agent systems, infrastructure, security research, PKM
- 6+ months of daily agent-driven use
- Multiple model swaps (OpenAI → Claude → DeepSeek) without rewriting the workspace
- 150+ inbox files, 60+ wiki entries

The framework is the **sediment**, not the design. Every convention in this repo was a response to a real failure. The anti-patterns list is longer than the features list because each one is a scar.

Lineage:
- **PKM principles** from Zettelkasten, PARA, Johnny Decimal
- **GTD** from David Allen (the inbox buffer is pure GTD capture)
- **Progressive disclosure** from information architecture
- **Agent-first design** from observing how Claude Code / Codex actually read files (index first, detail later)
- **Multi-provider API design** from the Anthropic-compatible API conventions adopted by DeepSeek, Qwen, and others

---

## Reading list

If this framework resonates, you might also like:

- **Anthropic's official CLAUDE.md docs** — the baseline, Claude-specific but solid
- **`awesome-claude-code`** — community-curated list of CLAUDE.md examples
- **MindStudio "Personal AI OS" series** — similar ideas from a SaaS angle (closed-source but good writing)
- **Johnny Decimal** — the original "folder structure is cognitive infrastructure" framework
- **Building a Second Brain (Tiago Forte)** — PKM principles this framework inherits from
