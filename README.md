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
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) in extension mode + the "Playwright Extension" from the Chrome Web Store — drives your logged-in Chrome with trusted clicks (needed for Upwork's dropdowns)
- Upwork account authenticated in browser
- Optional fallback: [browser-pilot](https://github.com/korbold/browser-pilot)

## Setup

### 1. Install the skill

```bash
cp -r freelance-ops ~/.claude/skills/
```

### 2. Create your data files

Your personal data lives in gitignored files under `data/`. Create them from the templates:

```bash
cd ~/.claude/skills/freelance-ops/data
cp profile.example.md profile.md
cp filters.example.json filters.json
cp tracker.example.json tracker.json
cp pipeline.example.md pipeline.md
```

Then edit `data/profile.md` with your name, hourly bid tiers and portfolio projects.

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

### 5. Set up Playwright MCP

1. Install the "Playwright Extension" in Chrome.
2. Add the MCP server (user scope, so every project sees it):

```bash
claude mcp add --scope user playwright-extension -- npx -y @playwright/mcp@latest --extension
```

3. Restart Claude Code. On the first browser call, Chrome asks you to approve the connection and pick the tab Claude may use.

Playwright writes snapshots to `.playwright-mcp/` in the working directory — add it to `.gitignore`.

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
