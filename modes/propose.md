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
- Hourly: tier from "Hourly Bid" in `data/profile.md` (straightforward / standard / architecture).

### 2c. Write proposal

Follow Proposal Writing Rules from `_shared.md`:
1. Hook: 1 sentence referencing their specific problem
2. Experience: 2-3 sentences with a named project from "Portfolio References" in `data/profile.md`
3. CTA: 1 sentence asking to connect

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

Tools: Playwright MCP. Selectors: see Upwork Selectors in `_shared.md`.

```
# Open the form directly — {cipher} is the ~0... id from the job URL
browser_navigate("https://www.upwork.com/nx/proposals/job/~{cipher}/apply/")
browser_wait_for(text: "Cover Letter")

# Hourly job
browser_type("#step-rate", bid_value)          # replaces the profile rate
browser_find(text: "How often do you want a rate increase?")
browser_click(<combobox ref>)
browser_find(text: "Never")
browser_click(<option "Never" ref>)

# Fixed-price job (instead of the hourly block)
browser_click("input[name=milestoneMode][value=default]")
browser_type("input[data-test=currency-input]", bid_value)
browser_click(<duration combobox ref>); browser_click(<duration option ref>)

# Both
browser_type("textarea >> nth=0", proposal_text)
# Screening questions: answer each remaining textarea from the JD + profile; show answers in Phase 3
# Attachments: the form has one input[type=file]; click the attach control, then browser_file_upload([cv_path])
# If boosted: set the bid in "Boost your proposal"

browser_take_screenshot(scale: "css", fullPage: true)   # filled form, before sending
browser_click("button.air3-btn-primary")                # Send
# Fixed price shows a confirm modal:
#   browser_click("[role=dialog] input[type=checkbox]")
#   browser_click("[role=dialog] button.air3-btn-primary")
browser_wait_for(text: "Your proposal was submitted")   # or URL leaves /apply/
browser_take_screenshot(scale: "css")                   # success evidence
```

The Send button reads "Send for {N} Connects" — include N in the Phase 3 confirmation (open the form during Phase 2 to read it, without filling anything).

Before clicking Send, read back the form with `browser_evaluate` (rate, rate increase, cover letter length, every textarea filled). Any empty required field or mismatch → stop and report.

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
