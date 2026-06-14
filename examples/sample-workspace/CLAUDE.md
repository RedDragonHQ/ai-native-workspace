# CLAUDE.md — my-workspace

> Agent runtime config. Loaded automatically when the agent starts in this directory.

## Directory structure

```
my-workspace/
├── CLAUDE.md              ← You are here (workspace-level cognitive config)
├── inbox/                 ← Draft buffer: session summaries, TODO, quick notes
│   ├── README.md          ← Inbox file format
│   └── TODO.md            ← Cross-session running list
├── projects/              ← Concrete projects
│   └── example-project/
│       └── CLAUDE.md      ← Project-level cognitive config
└── wiki/                  ← Matured knowledge base (optional, build later)
    └── MANIFEST.md        ← Progressive-loading index
```

## Workflow

### Session end (do this every time)

1. Write an inbox file: `inbox/YYYY-MM-DD-short-description.md`
   - Format: tags + summary + achievement list + output file paths
2. Update `inbox/TODO.md` (add new items / move done items to ✅)

### Session resume

1. Read the "Next time start here" section of `inbox/TODO.md`
2. Read recent `inbox/` files for context (only if needed)
3. Check recent `inbox/` files for context from prior sessions

### Knowledge sedimentation

```
Information → inbox/ (draft, low friction)
            → mature into:
               - project docs (project-specific technical details)
               - wiki/ (cross-project general knowledge)
            → deprecate when stale:
               - mark freshness: sleeping/deprecated
               - or move to archive/
```

## Conventions

- **CLAUDE.md is the agent's runtime config.** Update it when workflow changes.
- **Read the index first, details on demand.** Don't load the whole workspace at boot.
- **Technical correctness > social comfort.** When given feedback, verify against facts before implementing or pushing back.
- **TODO.md is a draft, not a command.** Always validate paths and classifications before acting on a TODO item.
- **Inbox is sacred.** Don't auto-write inbox files without prompting the user.

## When to use which file

| Content type | Where it lives |
|--------------|----------------|
| Session summary / achievements | `inbox/YYYY-MM-DD-xxx.md` |
| Cross-session running TODOs | `inbox/TODO.md` |
| Quick technical note | `inbox/` (upgrade later) |
| Stable project knowledge | `projects/<name>/` |
| Stable cross-project knowledge | `wiki/` (after inbox sediment) |
| Workflow conventions | This file (root CLAUDE.md) |
| Project-specific conventions | `projects/<name>/CLAUDE.md` |
