# CLAUDE.md — example-project

> Agent runtime config for this project. Loaded automatically when the agent `cd`s into this directory.
> Inherits from the workspace root `CLAUDE.md`.

## Project overview

A one-paragraph description of what this project is.

## Directory structure

```
example-project/
├── CLAUDE.md              ← You are here
├── README.md              ← Human-facing docs
├── src/                   ← Source code
└── tests/                 ← Tests
```

## Key files

| File | Purpose |
|------|---------|
| `src/main.py` | Entry point |
| `src/core.py` | Core logic |
| `tests/test_core.py` | Main test file |

## How to run

```bash
# Development
python -m src.main

# Tests
pytest tests/
```

## Project-specific conventions

*(things specific to this project, not applicable workspace-wide)*

- Prefer X over Y for reason Z
- When doing A, always check B first
- This project uses library C because D

## Known pitfalls

*(things that have burned you in this project)*

- Pitfall 1 → how to avoid
- Pitfall 2 → how to avoid
