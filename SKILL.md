---
name: freelance-ops
description: Automated freelance job hunting — scan Upwork, validate jobs, send proposals
arguments: mode
user-invocable: true
argument-hint: "[{upwork-url} | scan | validate | propose | pipeline | tracker | patterns]"
license: MIT
---

# freelance-ops — Router

## Mode Routing

Determine mode from `$mode`:

| Input | Mode |
|-------|------|
| (empty / no args) | `discovery` |
| Upwork job URL | `auto-pipeline` |
| `scan` | `scan` |
| `validate` | `validate` |
| `propose` | `propose` |
| `pipeline` | `pipeline` |
| `tracker` | `tracker` |
| `patterns` | `patterns` |

**Auto-pipeline detection:** If `$mode` is not a known sub-command AND contains `upwork.com/jobs/` or `upwork.com/nx/jobs/` (with or without query params), execute `propose` mode with the full URL as target. Known sub-commands take priority: if `$mode` is exactly `scan`, `validate`, `propose`, `pipeline`, `tracker`, or `patterns`, route to that mode regardless of URL-like content.

## Discovery Mode

Show this menu:

```
freelance-ops — Command Center

  /freelance-ops {URL}      → AUTO: validate + propose
  /freelance-ops scan       → Browse Upwork, collect qualifying jobs
  /freelance-ops pipeline   → Process all URLs in data/pipeline.md
  /freelance-ops validate   → Validate single job URL (no proposal sent)
  /freelance-ops tracker    → Proposal history + reply rates
  /freelance-ops patterns   → Conversion analysis + filter recommendations
```

## Context Loading

All modes: Read `modes/_shared.md` then `modes/{mode}.md`, then execute.

For `scan`, `propose`, `pipeline`: launch as Agent with `_shared.md` + mode file content injected into subagent prompt.
