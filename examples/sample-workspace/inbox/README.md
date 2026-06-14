# inbox/ — the draft buffer

Anything can land here. Sort later.

## File format

Filename: `YYYY-MM-DD-short-description.md`

Content:

```markdown
---
tags: [relevant, tags]
date: YYYY-MM-DD
---

# Title

## Summary
One paragraph of what happened / what was learned.

## Achievements
- Did thing 1
- Did thing 2
- Shipped thing 3

## Output files
- `path/to/file1` — description
- `path/to/file2` — description

## Discussion points
(optional) Key decisions, open questions, things to revisit.
```

## Why this format?

- **Frontmatter tags** let you grep / filter by topic later.
- **Summary** forces a one-paragraph distillation — more useful than raw notes.
- **Achievements** are the resume-building material; list them explicitly.
- **Output files** with full paths let future-you (or future-agent) find the artifacts.
- **Discussion points** capture open questions that might otherwise get lost.

## What NOT to put in inbox

- **Stable project knowledge** → goes in `projects/<name>/` directly
- **Cross-project reference material** → goes in `wiki/` (after it proves worth)
- **User preferences / persistent facts** → goes in CLAUDE.md or project docs

Inbox is the **buffer**, not the archive.
