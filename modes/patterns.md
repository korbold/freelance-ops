# freelance-ops — Patterns Mode

Analyze tracker data to find what converts. Output actionable filter recommendations.

## Steps

### 1. Read data

Read `~/.claude/skills/freelance-ops/data/tracker.json`.

If fewer than 5 entries:
```
Not enough data yet. Need at least 5 proposals to detect patterns.
Current count: {N}
```
Stop.

### 2. Analyze by dimension

For each dimension, group entries and calculate reply rate and win rate:

**A. CV type (flutter vs react_native vs default)**
- Which CV gets more replies? More hires?

**B. Budget range**
- Group: <$500, $500-$1000, $1000-$3000, $3000+
- Which range converts best?

**C. Client score**
- Group by score: 1-2, 3, 4
- Does higher score = better outcome?

**D. Boosted vs non-boosted**
- Did Boost proposals outperform regular ones?

**E. Job title keywords**
- Extract most common words from `job_title` of hired/replied entries
- vs most common words in ghosted/rejected entries

### 3. Output analysis

```
Patterns Analysis — {N} proposals analyzed
═══════════════════════════════════════════

CV PERFORMANCE:
  Flutter CV:       {sent} sent | {reply_rate}% reply | {win_rate}% win
  RN CV:            {sent} sent | {reply_rate}% reply | {win_rate}% win

BUDGET SWEET SPOT:
  Best range:       ${min}-${max} ({win_rate}% win rate)
  Avoid:            ${range} ({win_rate}% win rate)

BOOST IMPACT:
  Boosted:          {reply_rate}% reply | {win_rate}% win
  Not boosted:      {reply_rate}% reply | {win_rate}% win

TOP CONVERTING KEYWORDS:
  {keyword1}, {keyword2}, {keyword3}

RECOMMENDATIONS:
  {list of specific filter adjustments to make in filters.json}
```

### 4. Suggest filter updates

Based on the analysis, suggest concrete edits to `filters.json`:

Example outputs:
- "Raise `fixed_min` from $300 to $500 — jobs under $500 have 0% win rate"
- "Add 'no experience needed' to negative_keywords — all rejections contain this"
- "Lower `boost_threshold` to 3 — boosted jobs reply 2× more"

Ask user: "Apply these filter updates? [Y/N]"

If Y: read filters.json, apply changes, write back, confirm.
