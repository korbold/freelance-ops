# freelance-ops — Validate Mode

Validate a single Upwork job URL against all hard filters and calculate priority score.

## Input

`$mode` contains the job URL, or the user provided it in the invocation.

## Steps

### 1. Read config

Read `~/.claude/skills/freelance-ops/data/filters.json`. Extract:
- `budget.fixed_min`, `budget.hourly_min`
- `rating_min`, `hires_min`
- `negative_keywords` list

### 2. Load job page

```
browser_navigate(job_url)
browser_get_content()  # get full page HTML/text
```

### 3. Extract job fields

From page content, extract:
- `job_title` — title of the posting
- `client_rating` — star rating (e.g. 4.93)
- `payment_verified` — boolean (look for "Payment verified" badge)
- `client_hires` — number of hires (e.g. "47 hires")
- `client_total_spent` — total spend (e.g. "$12K+ spent")
- `client_reviews` — number of reviews
- `budget` — fixed amount or hourly range
- `job_description` — full description text

### 4. Run hard filters (stop at first failure)

Check in this order:

**Filter 1 — Payment verified:**
If `payment_verified == false` → return:
```
SKIP: Payment not verified
Job: {job_title}
```

**Filter 2 — Client rating:**
If `client_rating < rating_min` OR rating missing → return:
```
SKIP: Client rating {client_rating} below minimum {rating_min}
Job: {job_title}
```

**Filter 3 — Prior hires:**
If `client_hires < hires_min` → return:
```
SKIP: Client has {client_hires} hires (minimum: {hires_min})
Job: {job_title}
```

**Filter 4 — Budget:**
Parse budget type (fixed vs hourly):
- Fixed: if amount < `fixed_min` → SKIP
- Hourly: if rate < `hourly_min` → SKIP
Return:
```
SKIP: Budget {budget} below minimum (fixed: ${fixed_min} / hourly: ${hourly_min}/hr)
Job: {job_title}
```

**Filter 5 — Negative keywords:**
Check `job_description.toLowerCase()` for each keyword in `negative_keywords`.
If any match → return:
```
SKIP: Negative keyword detected — "{matched_keyword}"
Job: {job_title}
```

### 5. Calculate priority score (only if all filters pass)

| Condition | Points |
|-----------|--------|
| client_total_spent > $10,000 | +2 |
| client_reviews > 10 | +1 |
| job_description word count > 200 | +1 |

### 6. Output validation result

**If PASS:**
```
✅ PASS — {job_title}
─────────────────────────────────────
Client: ⭐ {client_rating} | {client_hires} hires | {client_total_spent} spent
Budget: {budget}
Score: {score}/4 {boosted_marker}
CV: {cv_selected}

{job_description first 100 words...}
```

Where `{boosted_marker}` = "⚡ BOOST" if score ≥ boost_threshold, else "".
Where `{cv_selected}` = determined by CV Selection Logic from _shared.md.

**If SKIP:**
```
❌ SKIP — {job_title}
Reason: {reason}
```
