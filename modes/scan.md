# freelance-ops — Scan Mode

Browse Upwork job listings, do a quick filter pass, add qualifying URLs to pipeline.

## Steps

### 1. Read config

Read `~/.claude/skills/freelance-ops/data/filters.json`. Note `negative_keywords`, `budget.fixed_min`, `budget.hourly_min`.

### 2. Navigate to Upwork search

Run two separate searches (to cover both specializations):

**Search A — Flutter:**
```
browser_navigate("https://www.upwork.com/nx/find-work/?q=flutter+developer&sort=recency")
```

**Search B — React Native:**
```
browser_navigate("https://www.upwork.com/nx/find-work/?q=react+native+developer&sort=recency")
```

### 3. For each search result page

Use `browser_get_content()` to extract visible job listings.

For each listing, do a quick pre-filter (no need to open the job page):
- Does title/snippet contain any `negative_keywords`? → skip
- Does stated budget look below minimum? → skip
- Does client show "Payment not verified"? → skip

Collect URLs of listings that pass quick pre-filter.

### 4. Scroll and collect

Scroll down twice to collect at least 10-20 candidate URLs per search:

```
browser_evaluate("window.scrollTo(0, document.body.scrollHeight)")
browser_wait(500)
browser_get_content()
```

### 5. Write to pipeline

Read current `~/.claude/skills/freelance-ops/data/pipeline.md`.

Append qualifying URLs below the `<!-- JOBS -->` marker, one per line. Skip URLs already present in the file.

### 6. Report

```
Scan complete.
Flutter search: {N} candidates found
React Native search: {N} candidates found
Total added to pipeline: {N} new URLs

Run /freelance-ops pipeline to process.
```
