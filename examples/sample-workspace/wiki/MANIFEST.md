# Wiki MANIFEST

> Progressive-loading index. **Read this file first (cheap), then pick 2-5 entries to drill into (expensive).**
>
> Format: `- path — one-line summary`

## How to use this file

1. Agent reads MANIFEST.md at session start (or when wiki is needed).
2. Scans the one-line summaries to find 2-5 relevant entries.
3. Reads only those entries in full.
4. Never loads the entire wiki into context.

## Adding entries

When a wiki file is created or updated:
1. Add a one-line summary here.
2. Keep summaries ≤ 20 words.
3. Group by directory or topic.

---

## Concepts

*(cross-project ideas, design principles)*

- `concepts/inbox-buffer.md` — Why capture-first beats classify-first
- `concepts/progressive-loading.md` — Context window management via index-first access

## Notes

*(loose observations, half-formed ideas)*

- `notes/pending-xyz.md` — Question about X that needs investigation

## References

*(links to external resources worth remembering)*

- `references/anthropic-claude-md-docs.md` — Official Anthropic CLAUDE.md best practices
