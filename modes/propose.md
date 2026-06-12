# freelance-ops — Propose Mode

Full pipeline: validate job → draft proposal → show Y/N confirmation → send → log.

## Input

Job URL from `$mode` (auto-pipeline) or user-provided.

## Phase 1: VALIDATE

Run the full validation logic from `validate.md`. 

If result is SKIP: display the skip reason and stop. Do NOT proceed to draft.

If result is PASS: continue to Phase 2.

## Phase 2: DRAFT

Using the validated job data:

### 2a. Select CV

Apply CV Selection Logic from `_shared.md`:
- Flutter keywords in title/description → Flutter CV
- RN keywords → RN CV
- Otherwise → default (Flutter CV)

### 2b. Calculate bid

- Fixed price: use midpoint of stated range. If single number shown, use 95% of it.
- Hourly: $25/hr for straightforward tasks, $30/hr for standard mobile dev, $35/hr for architecture/tech lead work.

### 2c. Write proposal

Follow Proposal Writing Rules from `_shared.md`:
1. Hook: 1 sentence referencing their specific problem
2. Experience: 2-3 sentences with a named project from Danny's portfolio
3. CTA: 1 sentence asking to connect

**Portfolio references to draw from (pick most relevant):**
- **Turnly** — appointment management app (Flutter), self-funded, 500+ active users
- **LegalTech Ecuador** — legal document automation (Flutter), self-funded
- **AkíClub** — loyalty app for Corporación Favorita (Flutter, Android + iOS), 100k+ users
- **Kruger Corp projects** — enterprise mobile apps for Ecuador's largest retailer
- React Native projects for BairesDev clients (if RN job)

### 2d. Determine boost

If `score >= boost_threshold` from filters.json → set `boosted = true`

## Phase 3: CONFIRM

Display confirmation screen and wait for Y/N:

```
┌─────────────────────────────────────────────────────────┐
│  freelance-ops — PROPOSAL READY                         │
├─────────────────────────────────────────────────────────┤
│  Job:    {job_title}                                     │
│  Client: ⭐{rating} | {hires} hires | {spent} spent     │
│  Budget: {budget}   Bid: {bid}                          │
│  Score:  {score}/4  {BOOST if applicable}               │
│  CV:     {cv_name}                                      │
├─────────────────────────────────────────────────────────┤
│  PROPOSAL:                                              │
│                                                         │
│  {proposal_text}                                        │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  Send this proposal? [Y/N]                              │
└─────────────────────────────────────────────────────────┘
```

If user types N: log as SKIPPED_BY_USER in tracker and stop.
If user types Y: proceed to Phase 4.

## Phase 4: SEND

```
browser_navigate(job_url)
# Wait for page load
browser_wait("Apply Now" or "Submit a Proposal" button)
browser_click("Submit a Proposal" button)
# Wait for proposal form
browser_wait(cover letter textarea)
browser_fill(cover letter textarea, proposal_text)
browser_fill(bid amount field, bid_value)
# If boosted: browser_click("Boost your proposal" option)
browser_screenshot()  # capture form filled state
browser_click("Send Proposal" button)
browser_wait(confirmation message)
browser_screenshot()  # capture success confirmation
```

If any browser step fails: stop, report error, do NOT log as sent.

## Phase 5: LOG

Read current `~/.claude/skills/freelance-ops/data/tracker.json`.
Append new entry:

```json
{
  "date": "{today YYYY-MM-DD}",
  "job_title": "{job_title}",
  "job_url": "{job_url}",
  "client_rating": {client_rating},
  "client_hires": {client_hires},
  "client_spent": "{client_total_spent}",
  "budget": "{budget}",
  "bid": "{bid}",
  "cv_used": "{flutter|react_native|default}",
  "score": {score},
  "boosted": {true|false},
  "status": "sent"
}
```

Write updated array back to `tracker.json`.

Output:
```
✅ Proposal sent!
Logged to tracker. Use /freelance-ops tracker to view history.
```
