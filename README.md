# freelance-ops

A Claude Code skill for automated freelance job hunting on Upwork. Scans listings, validates clients against hard filters, drafts proposals, and submits them — you just approve with Y/N.

## Features

- **Scan** — browses Upwork for Flutter & React Native jobs, pre-filters by budget/keywords/payment status
- **Validate** — deep-checks a job: payment verified, client rating ≥4.8, prior hires, budget threshold, negative keywords
- **Propose** — full pipeline: validate → draft proposal → Y/N confirm → send via browser → log
- **Pipeline** — process a queue of job URLs in bulk
- **Tracker** — view proposal history, reply rates, win rates
- **Patterns** — analyze what converts, get recommendations to tune your filters

## Requirements

- [Claude Code](https://claude.ai/code)
- [browser-pilot](https://github.com/korbold/browser-pilot) — MCP server + browser extension that gives Claude access to your authenticated browser sessions
- Upwork account authenticated in browser

## Setup

### 1. Install the skill

```bash
cp -r freelance-ops ~/.claude/skills/
```

### 2. Configure your profile

Edit `~/.claude/skills/freelance-ops/modes/_shared.md`:
- Replace `Your Name`, `Your City`, etc. with your info
- Update portfolio references with your own projects

### 3. Set your CV paths

Edit `~/.claude/skills/freelance-ops/data/filters.json`:

```json
"cv_paths": {
  "flutter": "/absolute/path/to/your/CV_Flutter.pdf",
  "react_native": "/absolute/path/to/your/CV_ReactNative.pdf",
  "default": "/absolute/path/to/your/CV_Flutter.pdf"
}
```

### 4. Tune your filters (optional)

In `data/filters.json`, adjust:
- `budget.fixed_min` — minimum fixed price (default: $300)
- `budget.hourly_min` — minimum hourly rate (default: $15/hr)
- `negative_keywords` — add any phrases that signal bad clients

### 5. Set up browser-pilot

Install [browser-pilot](https://github.com/korbold/browser-pilot) (MCP server + browser extension), then make sure `upwork.com` is in the allowlist. For Workana too:

```json
{ "domains": ["upwork.com", "workana.com"] }
```

## Usage

```
/freelance-ops              → show command menu
/freelance-ops scan         → browse Upwork, populate pipeline
/freelance-ops pipeline     → process queued jobs, send proposals
/freelance-ops {URL}        → validate + propose a single job
/freelance-ops validate     → validate only (no proposal sent)
/freelance-ops tracker      → view proposal history
/freelance-ops patterns     → conversion analysis + filter recommendations
```

## How it works

**Validation hard filters** (any failure = skip, no Connects spent):
1. Payment not verified → skip
2. Client rating < 4.8 → skip
3. Zero prior hires → skip
4. Budget below threshold → skip
5. Negative keywords in description → skip

**Priority score** (orders queue, triggers Boost):
- Client spent > $10k → +2 pts
- Client reviews > 10 → +1 pt
- Description > 200 words → +1 pt
- Score ≥ 4 → Boost proposal

**CV auto-selection:**
- Flutter keywords in JD → Flutter CV
- React Native keywords → RN CV
- Otherwise → default CV

## License

MIT
