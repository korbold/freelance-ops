# freelance-ops — Tracker Mode

Display proposal history with stats and pending follow-ups.

## Steps

### 1. Read tracker

Read `~/.claude/skills/freelance-ops/data/tracker.json`.

If array is empty:
```
No proposals sent yet. Run /freelance-ops scan to find jobs.
```
Stop.

### 2. Calculate 30-day stats

Filter entries where `date` is within last 30 days.

Count by status:
- `sent` = still pending (no response yet)
- `replied` = client responded
- `hired` = won the job
- `rejected` = explicit rejection
- `ghosted` = sent > 5 days ago with no response

Calculate:
- Reply rate = (replied + hired + rejected) / total sent × 100
- Win rate = hired / total sent × 100

### 3. Display

```
Proposals — last 30 days
──────────────────────────────────────────
Sent: {N}  |  Replied: {N}  |  Hired: {N}  |  Ghosted: {N}
Reply rate: {X}%  |  Win rate: {X}%

PENDING REPLY (sent > 48h ago, status: sent):
{for each: "- {job_title} ({date}) — {bid} bid"}

RECENT WINS:
{for each hired: "- {job_title} ({date}) — {bid}"}

ALL PROPOSALS (newest first):
{date} | {status emoji} | {job_title} | {bid} | {cv_used}
```

Status emojis: sent=⏳ replied=💬 hired=✅ rejected=❌ ghosted=👻

### 4. Update stale entries

For any entry with status `sent` and date > 5 days ago: update status to `ghosted` in tracker.json automatically.
