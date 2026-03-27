# GmailDashboard — Context

## What This Is
AI-powered Gmail intelligence dashboard for Larry (~65yo CEO of a medical data analytics company, also runs community projects like condo board elevator replacement). Single `index.html`, no server, no database. He opens a URL, sees his emails sorted by importance with narrative briefings and interactive topic visualization. Includes Cleaning Mode to trash/archive emails directly.

## Current Status
**Feature-complete + Phase 5 done. Deployed to GitHub Pages. Tested with James's Gmail. Ready for Larry onboarding.**

Live: https://jbuckdev.github.io/gmail-dashboard/
Repo: https://github.com/jbuckdev/gmail-dashboard (public)

## Architecture
- Single `index.html` (~4000 lines, all inline HTML/CSS/JS)
- Google OAuth2 (readonly for dashboard, lazy upgrade to modify for cleaning)
- Gmail API (client-side)
- Claude Haiku API (`claude-haiku-4-5-20251001`) — classification, organic topics, narrative briefing
- GitHub Pages hosting (free HTTPS)
- All processing in Larry's browser

## Classification Engine (v3)
- **Stage 1: Noise filter (rules)** — catches ~40-50%: OOO, bounces, calendar responses, platform notifications (Healthchecks.io, Slack, GitHub, PagerDuty, etc.), Gmail Promotions/Social, noreply newsletters
- **Stage 2: AI analysis** — everything else goes to Claude. AI assigns category (reply/read/fyi/skip), organic topic names (not from a fixed list), summaries, action items. Topics are consistent across batches via "topics used so far" context.
- **Private Mode:** rules handle everything, `guessTopicFromRules()` for topics, template briefing. Zero external calls.

## Features
- Onboarding with privacy choice BEFORE login (Full Analysis vs Private Mode)
- Dark command-center UI (DM Serif Display + DM Sans)
- 50/50 split layout: left (briefing, emails, senders, charts) / right (large bubble map + drill)
- Expand button on every panel (fullscreen overlay toggle)
- Hero briefing with 4 stat cards
- AI narrative briefing (daily/weekly/monthly, reads full email bodies, clickable links)
- Smart body fetching (full body for important emails, snippet for noise)
- Force-directed bubble map (organic AI topics, drill-down, scaled for wide view)
- Top Senders, Volume Chart, Category Breakdown visualizations
- Cleaning Mode (click/drag select, trash, archive, undo, lazy scope upgrade)
- Smart filters in Cleaning Mode: Newsletters, Promotions, Social (bulk select)
- Selectable emails in bubble/category drill-down during Cleaning Mode
- Auto-scroll in both main panel and drill panel during drag-select
- Smart caching: sessionStorage incremental refresh (only fetches new emails)
- Silent OAuth token refresh (auto-renews at 50min, retries on 401)
- Preferences (VIP contacts, blocked senders, text size, privacy mode, Claude key)
- Help & Troubleshooting page
- Comprehensive error handling with Send Report for unfixable errors
- Print stylesheet (retained but Print button removed)

## Key Config
- OAuth Client ID: `663216675668-u9bnqasnncvurnl2ce47l0do2lvsod0k.apps.googleusercontent.com`
- Authorized origins: `http://localhost:8000`, `https://jbuckdev.github.io`
- Scopes: `gmail.readonly` (default), `gmail.modify` (lazy, for cleaning)
- Larry added as Google OAuth test user
- Claude model: `claude-haiku-4-5-20251001`

## Privacy
Binary toggle in Preferences:
- **Full Mode:** AI reads emails. Content sent to Claude. Anthropic retains up to 30 days for safety. Never used for training.
- **Private Mode:** Zero external API calls. Rule-based only. Nothing leaves browser.
- ZDR investigated — enterprise-only, no CORS, incompatible. Abandoned.

## Cleaning Mode
- Lazy scope upgrade: `gmail.readonly` → `gmail.modify` only when Clean clicked
- `gmail.modify` CANNOT send emails (verified against API docs)
- Trash only (30-day Gmail recovery), never permanent delete
- Click to select, drag to select range, auto-scroll near edges
- Select All on FYI and Skip sections only
- Smart filters: Newsletters, Promotions, Social — one-click bulk select via action bar
- Selectable emails in bubble/category drill-down (click bubble → select emails)
- Auto-scroll in drill panel sidebar during drag-select
- Action bar always visible in cleaning mode (shows filters + count)
- Confirmation for 10+ emails, 3-second countdown for 50+
- 10-second undo window
- API response checking with specific error messages

## Remaining Work
- Phase 7: Larry onboarding (create Anthropic account, seed VIP contacts, 15-min walkthrough, privacy conversation)
- Phase 8: Post-launch tuning (based on Larry's feedback)

## Decisions
- AI assigns organic topics, not fixed taxonomy — "Elevator Inspection" not "Community"
- Rules only filter obvious noise; AI handles everything that matters
- Binary privacy mode (all or nothing — subject lines are private data too)
- Lazy scope upgrade for cleaning (don't scare with permissions on first visit)
- gmail.modify not gmail.full (trash/archive but NEVER send or permanent delete)
- claude-haiku-4-5-20251001 (old ID was 404ing — critical fix)
- Send Report opens Gmail compose in browser (not local mail app)
