# freelance-ops — Scan Mode

Browse Upwork job listings, do a quick filter pass, add qualifying URLs to pipeline.

## Steps

### 1. Read config

Read `~/.claude/skills/freelance-ops/data/filters.json`. Note `negative_keywords`, `budget.fixed_min`, `budget.hourly_min`.

### 2. Navigate to Upwork search

Run two separate searches (to cover both specializations):

Tools: Playwright MCP (see Browser Tools in `_shared.md`).

**Search A — Flutter:**
```
browser_navigate("https://www.upwork.com/nx/search/jobs/?q=flutter+developer&sort=recency")
```

**Search B — React Native:**
```
browser_navigate("https://www.upwork.com/nx/search/jobs/?q=react+native+developer&sort=recency")
```

### 3. For each search result page

Extract the visible listings with one `browser_evaluate`:

```js
() => {
  const seen = new Map();
  for (const c of document.querySelectorAll('article')) {
    const a = c.querySelector('a[href*="/jobs/"][href*="~0"]');
    const cipher = a?.href.match(/~0\d+/)?.[0];
    if (!cipher || seen.has(cipher)) continue;
    seen.set(cipher, {
      title: a.textContent.trim(),
      url: `https://www.upwork.com/jobs/${cipher}`,
      text: c.innerText.slice(0, 600),
    });
  }
  return [...seen.values()];
}
```

If it returns nothing, Upwork changed the markup: run `browser_snapshot` once, find the job tile structure, and adjust the selector.

For each listing, do a quick pre-filter on `title` + `text` (no need to open the job page):
- Does title/snippet contain any `negative_keywords`? → skip
- Does stated budget look below minimum? → skip
- Does client show "Payment unverified"? → skip

Collect URLs of listings that pass quick pre-filter.

### 4. Scroll and collect

Scroll down twice to collect at least 10-20 candidate URLs per search:

```
browser_press_key("End")
browser_wait_for(time: 1)
browser_evaluate(<same extractor>)
```

Dedupe by URL — the list virtualizes, so earlier tiles may disappear from the DOM.

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
