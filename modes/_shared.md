# freelance-ops — Shared Context

## User Profile

**Name:** Your Name  
**Role:** Mobile Developer (Flutter & React Native)  
**Location:** Your City, Country  
**Timezone:** Your Timezone  
**Upwork profile:** authenticated in browser session  
**Languages:** Your languages

## CV Paths

- Flutter: `/path/to/your/CV_Flutter.pdf`  ← update in data/filters.json
- React Native: `/path/to/your/CV_ReactNative.pdf`  ← update in data/filters.json
- Default: Flutter CV

## Data Files

- Filters config: `~/.claude/skills/freelance-ops/data/filters.json`
- Proposal tracker: `~/.claude/skills/freelance-ops/data/tracker.json`
- Pipeline inbox: `~/.claude/skills/freelance-ops/data/pipeline.md`

## Platform: Upwork

- Base URL: `https://www.upwork.com`
- Job search URL: `https://www.upwork.com/nx/find-work/`
- Job URL pattern: `https://www.upwork.com/jobs/~*` or `/nx/jobs/~*`

## Hard Filter Rules (read from filters.json)

Before drafting any proposal, validate ALL of these. Any failure = SKIP immediately.

1. **Payment verified** — client must have payment method verified (look for verified badge)
2. **Rating** — client rating must be ≥ `rating_min` (default 4.8). If no rating, skip.
3. **Hires** — client must have ≥ `hires_min` prior hires (default 1)
4. **Budget** — for fixed jobs: budget ≥ `fixed_min`. For hourly: rate ≥ `hourly_min`
5. **Negative keywords** — if JD contains any keyword from `negative_keywords` list → SKIP

## Priority Score

After passing hard filters, calculate:
- Client total spent > $10,000 → +2 pts
- Client total reviews > 10 → +1 pt
- Job description word count > 200 → +1 pt
- Max possible: 4 pts

If score ≥ `boost_threshold` (default 4): mark job for Boost proposal.

## CV Selection Logic

Check JD (title + description) for keywords in priority order:
1. Any `flutter` keyword → use Flutter CV
2. Any `react_native` keyword → use RN CV
3. Neither → use default (Flutter CV)

## Proposal Writing Rules

Structure every proposal:
1. **Hook** (1 sentence): Reference their specific problem or goal from the JD. Do NOT start with "Hi, I'm Danny" or generic openers.
2. **Relevant experience** (2-3 sentences): Name a specific project from portfolio that matches their need. Be concrete — app name, platform, result.
3. **CTA** (1 sentence): "Can we connect this week to discuss?" or similar low-friction ask.

Total length: 80-120 words. No bullet points. Conversational tone.

## Bid Calculation

- Fixed price job: bid = midpoint of client's stated range, or 90% of max if range not shown
- Hourly job: bid $25-35/hr depending on complexity (junior task → $25, senior arch → $35)

## Portfolio References

Use these when writing proposals (pick most relevant to the JD):
- **Turnly** — appointment management app (Flutter), self-funded, 500+ active users
- **LegalTech Ecuador** — legal document automation (Flutter), self-funded
- **AkíClub** — loyalty app for Corporación Favorita (Flutter, Android + iOS), 100k+ users
- **Kruger Corp projects** — enterprise mobile apps for Ecuador's largest retailer
- **BairesDev React Native projects** — RN apps for international clients

## Tracker Entry Schema

When logging to tracker.json, append this object:
```json
{
  "date": "YYYY-MM-DD",
  "job_title": "...",
  "job_url": "...",
  "client_rating": 0.0,
  "client_hires": 0,
  "client_spent": "$0",
  "budget": "...",
  "bid": "...",
  "cv_used": "flutter | react_native | default",
  "score": 0,
  "boosted": false,
  "status": "sent"
}
```
