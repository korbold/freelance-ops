# freelance-ops — Shared Context

## User Profile

Read `~/.claude/skills/freelance-ops/data/profile.md`: name, hourly bid tiers, portfolio references. Upwork profile: authenticated in browser session.

## CV Paths

Set in `data/filters.json` → `cv_paths` (flutter, react_native, default).

## Data Files

All personal and gitignored. If one is missing, copy it from its `*.example.*` sibling first.

- Profile: `~/.claude/skills/freelance-ops/data/profile.md`
- Filters config: `~/.claude/skills/freelance-ops/data/filters.json`
- Proposal tracker: `~/.claude/skills/freelance-ops/data/tracker.json`
- Pipeline inbox: `~/.claude/skills/freelance-ops/data/pipeline.md`

## Platform: Upwork

- Base URL: `https://www.upwork.com`
- Job search URL: `https://www.upwork.com/nx/find-work/`
- Job URL pattern: `https://www.upwork.com/jobs/~*` or `/nx/jobs/~*`

## Browser Tools

Primary: **Playwright MCP in extension mode** (`@playwright/mcp --extension`, tools prefixed `mcp__playwright-extension__`). It drives the user's real, logged-in Chrome over CDP, so clicks are trusted events — Upwork's air3 dropdowns respond to them. browser-pilot is the fallback only.

| Action | Playwright tool |
|--------|-----------------|
| Open URL | `browser_navigate(url)` |
| Read page data | `browser_evaluate(() => ...)` — runs via CDP, not blocked by Upwork's CSP |
| Find an element ref | `browser_find(text \| regex)` — cheaper than a full `browser_snapshot` |
| Click | `browser_click(target)` — `target` = snapshot ref (`f3e299`) or unique CSS selector |
| Type into input | `browser_type(target, text)` or `browser_fill_form` |
| Upload file | click the attach control, then `browser_file_upload([absolute paths])` |
| Keyboard | `browser_press_key("End")` |
| Wait | `browser_wait_for(text \| time)` |
| Evidence | `browser_take_screenshot(scale: "css")` |

Connection notes:
- First call of a session blocks until the user approves the connection in Chrome (Playwright Extension tab/icon) and picks the tab. If a call hangs, tell the user to approve it — don't retry.
- Refs (`f3e555`) change after every navigation or re-render. Re-run `browser_find` before clicking.
- The tool writes snapshots/logs to `.playwright-mcp/` in the current directory. Keep it out of git.

## Upwork Selectors

Job page (`/jobs/~{cipher}`):
- Description: `[data-test=Description]`
- Job type, rate/amount, duration, level: `.features`
- Client box: `[data-test=about-client-container]`
- Client activity (proposals, interviewing, invites): `.client-activity-items`

Proposal form (`/nx/proposals/job/~{cipher}/apply/`):
- Hourly rate: `#step-rate`
- Rate increase: combobox "How often do you want a rate increase?" → option "Never" (hides the % field)
- Cover letter: first `textarea`; screening questions are the following `textarea`s
- Fixed price "By project": `input[name=milestoneMode][value=default]`, amount: `input[data-test=currency-input]`
- Fixed price duration: combobox → click the option by name
- Send: `button.air3-btn-primary`
- Fixed-price confirm modal: `[role=dialog] input[type=checkbox]`, then `[role=dialog] button.air3-btn-primary`

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
1. **Hook** (1 sentence): Reference their specific problem or goal from the JD. Do NOT start with "Hi, I'm {name}" or generic openers.
2. **Relevant experience** (2-3 sentences): Name a specific project from portfolio that matches their need. Be concrete — app name, platform, result.
3. **CTA** (1 sentence): "Can we connect this week to discuss?" or similar low-friction ask.

Total length: 80-120 words. No bullet points. Conversational tone.

## Bid Calculation

- Fixed price job: bid = midpoint of client's stated range, or 90% of max if range not shown
- Hourly job: pick the tier from "Hourly Bid" in `data/profile.md` by complexity

## Portfolio References

Use "Portfolio References" in `data/profile.md` — pick the one most relevant to the JD.

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
