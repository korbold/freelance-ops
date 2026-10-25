# freelance-ops — Pipeline Mode

Process all job URLs in `data/pipeline.md` through the propose flow.

## Steps

### 1. Read pipeline

Read `~/.claude/skills/freelance-ops/data/pipeline.md`.

Extract all lines below `<!-- JOBS -->` that:
- Start with `https://`
- Are not commented out (no leading `#`)

If no URLs found:
```
Pipeline is empty. Run /freelance-ops scan to find jobs, or add URLs manually to data/pipeline.md
```
Stop.

### 2. Report queue

```
Pipeline: {N} jobs queued
────────────────────────
{list of job URLs}

Processing now...
```

### 3. Process each URL

For each URL in order:

Run the full propose flow (Phase 1 through Phase 5 from propose.md).

After each proposal (sent or skipped): remove the URL from `data/pipeline.md` so it won't be processed again on next run.

### 4. Session summary

After all URLs processed:

```
Pipeline complete.
────────────────────────────────
Sent:    {N}
Skipped: {N} (hard filter failures)
Passed:  {N} (user declined)

Run /freelance-ops tracker to see full history.
```
