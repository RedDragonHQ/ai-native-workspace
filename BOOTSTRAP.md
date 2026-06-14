# AI-Native Workspace — Agent Bootstrap Instructions

> **This file is designed for AI agents.**
>
> If you're a human reading this: this is the bootstrap document an AI agent reads to set up a fresh workspace from scratch. Paste it into Claude Code / Codex / Cursor in an empty directory and the agent will build the structure for you. You don't need to read it yourself.
>
> **Non-English readers**: translations are in `zh/` (Chinese: [`zh/BOOTSTRAP.md`](zh/BOOTSTRAP.md)).
>
> For the framework design, see [`ARCHITECTURE.md`](ARCHITECTURE.md).
> For a runnable example, see [`examples/sample-workspace/`](examples/sample-workspace/).

---

## 1. The Four-Layer Architecture

```
┌─────────────────────────────────────────────────────┐
│  Application Layer                                  │
│  Projects · Knowledge base · Inbox · Workflows      │
├─────────────────────────────────────────────────────┤
│  Protocol Layer                                     │
│  CLAUDE.md conventions · CLI tools · MCP servers    │
├─────────────────────────────────────────────────────┤
│  Agent Layer                                        │
│  Claude Code · Codex · Cursor · any md-aware agent  │
├─────────────────────────────────────────────────────┤
│  Model Layer                                        │
│  LLM APIs · multi-provider · Anthropic-compatible   │
└─────────────────────────────────────────────────────┘
```

### 1.1 Model Layer

The LLM reasoning capability behind the agent. **Key design: use a unified protocol across providers, swappable at any time.**

| Concept | Description |
|---------|-------------|
| **Unified protocol** | Prefer the Anthropic Messages API compatible protocol as the standard interface. DeepSeek, Qwen, and third-party relays all support `base_url` override, so the agent can swap models without code changes |
| **Multi-provider** | Have at least 2: a low-cost daily driver (e.g. DeepSeek), and a strong-reasoning fallback (e.g. Claude via relay). A single provider is a risk |
| **Environment isolation** | Use env-var scripts per project/scenario (e.g. `source daily.sh` / `source strong.sh`); switching only needs a re-source |
| **Proxy chain** | Some APIs (e.g. OpenRouter) have region restrictions requiring SOCKS5/HTTP proxies. The proxy chain is infrastructure, not something the agent manages |

**You don't need to configure the model layer at init time** — the user has already given you access to this document, which means they have their own API key / access path. Just know the model layer exists; the agent gets its reasoning through it.

### 1.2 Agent Layer

**One agent is enough.** Don't reach for multi-agent from the start.

| Concept | Description |
|---------|-------------|
| **Layered CLAUDE.md cognition** | The agent's runtime config, not a regular README. Auto-loaded in layers: global → workspace root → subproject |
| **Progressive context** | Agent context windows are finite. Don't read every file; scan the index/summary first, dive on demand |
| **Tool calling** | Agent gets execution power (files, search, git, docker, etc.) through MCP servers and CLIs |
| **Persistent state** | Cross-session critical info goes in CLAUDE.md or project docs, auto-loaded at next launch |

**CLAUDE.md layered loading protocol**:

```
~/.claude/CLAUDE.md              ← Global (agent identity, universal conventions)
<workspace>/CLAUDE.md            ← Workspace root (directory tree, project list, cross-project workflows)
<workspace>/<project>/CLAUDE.md  ← Subproject level (auto-appended when you cd in)
```

**Key principles**:
- CLAUDE.md is **the agent's operating system config**. Optimized for agent consumption, but fully human-readable — when agents aren't available, a human can step in and operate using the same protocols.
- Each CLAUDE.md has a different job: root manages global cognition, subproject manages technical detail.
- Cross-session facts (user preferences, decisions, lessons) go in CLAUDE.md or project docs — not work logs.

### 1.3 Protocol Layer

The channels the agent uses to interact with the outside world. **Three protocol types, increasing capability**:

```
File protocol    CLI tools           MCP Servers
(CLAUDE.md)      (git/curl/docker)   (search/DB/API)
     ↓                ↓                  ↓
  Agent's manual   Agent's hands     Agent's specialist tools
```

| Protocol | Description | Required at init |
|----------|-------------|:-----------:|
| **File protocol** | CLAUDE.md conventions (dir structure, naming, workflows) | ✅ yes |
| **CLI tools** | git, curl, docker, node, python, etc. | ✅ user's env has them |
| **MCP Server** | Search engines, databases, external API bridges | ⬜ add on demand |

**MCP servers are extended capability, not a requirement.** Don't configure any MCP at init time. The user adds them as needed later (search, wiki semantic search, external API bridges).

**File protocol is the core** — CLAUDE.md itself is the most important protocol. It defines how the agent understands the workspace, operates files, and manages knowledge.

### 1.4 Application Layer

The actual work. **Three core components**:

```
workspace/
├── CLAUDE.md              ← Workspace cognitive entry point
├── inbox/                 ← Information buffer (session wrap-ups, TODO, quick notes)
│   ├── README.md          ← Inbox format conventions
│   └── TODO.md            ← Cross-session running list
├── projects/              ← Concrete projects (or directly in root)
│   ├── project-a/
│   │   └── CLAUDE.md      ← Project-level cognition
│   └── project-b/
│       └── CLAUDE.md
└── wiki/                  ← Knowledge base (optional, build when mature)
    └── MANIFEST.md        ← Progressive-loading index
```

| Component | Role | Required at init |
|-----------|------|:---:|
| **inbox/** | Information buffer: session summaries, quick notes, running TODO | ✅ yes |
| **Project folders** | One folder per project + own CLAUDE.md | ✅ scan existing |
| **wiki/** | Structured knowledge base (tiered docs + progressive index) | ⬜ optional |

**inbox/ is the core innovation**:
- It's a **draft zone**, not the final output. Information lands in inbox first, matures into wiki or project docs later.
- At each session end, write an inbox file (summary + outputs + key points) — the agent's "work log".
- `TODO.md` is the single cross-session todo list. The agent reads it first at launch to know "where we left off".
- inbox lowers cognitive load: no need to decide where info belongs on capture; toss it in inbox, sort later.

**wiki/ is the advanced capability**:
- Build it when inbox accumulates enough to need structure.
- Core mechanism: **progressive loading**. `MANIFEST.md` is the index (one-line summary per entry). Agent reads index first, then dives into selected entries on demand.
- Docs can carry maturity tiers (`raw` → `distilled` → `canonical`) and freshness tiers (`active` → `sleeping` → `deprecated`). These come from PKM practice — if there's enough interest, a deeper PKM guide will follow.

---

## 2. Initialization Steps

**You are the AI agent. Now initialize the workspace following these steps.**

### Pre-step: Confirm the workspace

```
1. Confirm the current directory is the user-specified workspace root
2. Check if CLAUDE.md already exists (yes → incremental update; no → fresh create)
3. Check for .claude/ or .git/ (to understand current config state)
```

### Step 1: Scan the workspace

```
Scan every subdirectory under the root, build a directory tree. For each:
- Identify if it's a "project" (has code files / README / package.json / .git etc.)
- Identify if it already has a CLAUDE.md
- Generate a one-line summary (inferred from folder name and contents)

Scan root-level files:
- Identify script files (.sh / .py etc.)
- Identify config files (.json / .yml / .env etc.)
- Ignore hidden directories (.git / .venv / node_modules etc.)
```

**Output**: a structured workspace map, used to generate CLAUDE.md.

### Step 2: Generate the root CLAUDE.md

```markdown
# CLAUDE.md — <workspace name>

> Agent runtime guidance for this workspace. Auto-loaded at every launch.

## Directory structure

<Step 1 scan results, tree format, one-line annotation per dir/file>

## Workflow

### Session end (must-do)
1. Write an inbox file: `inbox/YYYY-MM-DD-short-description.md`
   - Format: tags + summary + achievement list + output file paths
2. Update `inbox/TODO.md` (add new / mark done items)

### Session resume
1. Read the "Next time start here" section of `inbox/TODO.md`
2. Read recent inbox/ files on demand for context

### Knowledge sedimentation path
inbox/ (draft) → project docs or wiki/ (mature knowledge)

## Collaboration principles

- CLAUDE.md is the agent's runtime config; changes take effect at next launch
- Don't bulk-read large files; scan the index first, dive on demand
- Technical correctness > social comfort: when given feedback, verify facts before implementing or pushing back
- inbox/TODO.md is a draft; always validate paths and classifications before acting on it
```

**Notes**:
- Keep it concise, under 100 lines. Detailed tech specifics go in subproject CLAUDE.md files.
- Don't pre-write specific tech stacks / tool lists — those come at subproject level or from the user later.
- Workflow conventions are the core — they define the agent's daily behavioral patterns.

### Step 3: Create the inbox structure

**3a. Create `inbox/` dir + `inbox/README.md`**:

```markdown
# inbox/ — Information buffer

## File format

Filename: `YYYY-MM-DD-short-description.md`

File content:
---
tags: [relevant, tags]
date: YYYY-MM-DD
---

# Title

## Summary
One paragraph summarizing the work / discovery.

## Achievements
- Did thing 1
- Did thing 2

## Output files
- `path/to/file1` — description
- `path/to/file2` — description

## Discussion points
(optional) Key discussions and decisions.
```

**3b. Create `inbox/TODO.md`**:

```markdown
# TODO — Cross-session running list

> Agent: read the "Next time start here" section at every session start.
> Move done items to the "✅ Done" area at the bottom.

## 🔄 Next time start here

(Current top-priority TODOs, 1-3 items)

## 📋 In progress

## 📋 Pending

## ✅ Done
```

### Step 4: (optional) Generate subproject CLAUDE.md files

For each project folder identified in Step 1 that doesn't already have a CLAUDE.md:

```markdown
# CLAUDE.md — <project name>

> Agent runtime guidance for this project. Auto-appended when you cd in.

## Project overview
<One-line summary from scan>

## Directory structure
<This project's tree>

## Key files
<List the 3-5 most important files and their roles>

## How to run
<Inferable from package.json / Makefile / README>
```

**Note**: Subproject CLAUDE.md files should be skeletal only. Let the user and agent flesh them out through daily use. Don't try to write them perfectly up front.

### Step 5: Verify

```
1. Check CLAUDE.md is syntactically correct and structurally clear
2. Check inbox/ directory and its two files were created
3. If subproject CLAUDE.md files exist, check each is reasonable
4. Output an init summary telling the user what to do next
```

---

## 3. Daily Workflow Conventions

### 3.1 Session lifecycle

```
Launch → read CLAUDE.md + TODO.md
  → user makes request
  → agent works (read/write files, call tools)
  → session ends:
     1. write inbox/YYYY-MM-DD-xxx.md (work summary)
     2. update inbox/TODO.md (add/complete/adjust items)
```

### 3.2 Knowledge sedimentation path

```
Info generated → inbox/ (draft, low friction, toss in anytime)
              → upgrade to:
                 - project docs (project-specific technical detail)
                 - wiki/ (cross-project general knowledge)
              → downgrade when stale:
                 - change freshness to sleeping/deprecated
                 - or move to archive/
```

**The value of inbox**: lower the psychological bar to writing docs. Not everything is worth a formal doc, but everything is worth a note. inbox is that "casual capture" place.

### 3.3 CLAUDE.md maintenance principles

| When to update | Who updates | What to update |
|----------------|-------------|----------------|
| New subproject added | Agent (at init) | Root CLAUDE.md tree + subproject CLAUDE.md |
| Workflow changes | User decides, agent executes | Root CLAUDE.md workflow section |
| Pitfall / architectural change | Agent (mid-session) | Relevant project's CLAUDE.md |
| User preference / feedback | Agent (when feedback received) | CLAUDE.md or project docs |

**Iron rule**: Keep CLAUDE.md concise. Over 150 lines means it should be split into subproject or wiki.

### 3.4 Anti-patterns (don'ts)

| ❌ Don't | ✅ Do |
|---------|------|
| Reach for multi-agent from the start | Start with single agent; split only when genuinely complex |
| Bulk-read every workspace file | Scan CLAUDE.md index first, dive on demand |
| Skip inbox and write directly to wiki | inbox is the buffer, wiki is the sediment |
| Write a 500-line CLAUDE.md | Keep under 100 lines, detail goes in subproject files |
| Treat TODO as a mindless command queue | TODO is a draft, validate before acting |
| Auto-trigger inbox file writes | Prompt the user, don't write without confirmation |
| Prescribe best practices per project up front | Observe the user's working style first, then suggest |

---

## 4. Extension Paths (upgrade post-init on demand)

Optional upgrades for a mature workspace. **Don't do these at init time**:

| Upgrade | Trigger condition | Description |
|---------|-------------------|-------------|
| **wiki/ knowledge base** | inbox hits 20+ files, needs structure | Build wiki/ + MANIFEST.md index + progressive loading |
| **MCP Server** | Need search / database / external API | Wire up SearXNG / semantic search / specialized tools |
| **Multi-model routing** | Single model insufficient or too costly | Multi-LLM providers + env-var swap scripts |
| **Homepage dashboard** | Want to visualize work log and project progress | Static HTML + worklog.js data |
| **Self-hosted services** | Need RSS / automation / search infra | Docker-deploy SearXNG / Miniflux / n8n |
| **Automation scripts** | Manual operations too frequent | build_manifest.py / rss_digest.py etc. |
| **Multi-Agent** | Single agent context window genuinely insufficient | Split into specialist agents, orchestrate via workflow |

---

## 5. Final Reminders for the Agent

1. **You're the tool, not the protagonist.** User decides, you execute. When uncertain, ask — don't take initiative.
2. **CLAUDE.md is your operating system.** Read it at every launch; it is your behavioral spec.
3. **inbox is your work log.** Write an entry at every session end; it bridges your cross-session continuity.
4. **Progressive loading is your survival strategy.** Context is finite; index first, details later, never bulk-read.
5. **Simple beats perfect.** A usable inbox is a hundred times more valuable than a perfect wiki.

---

*This document was extracted from the RedDragon Workspace Project (a running AI-native personal operating system). All project-specific content was removed; only the reusable architectural skeleton remains.*
