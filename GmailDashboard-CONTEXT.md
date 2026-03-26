# GmailDashboard — Context

## What This Is
AI-powered Gmail intelligence dashboard for Larry (~65yo CEO of a medical data analytics company, also runs community projects like condo board elevator replacement). Single `index.html`, no server, no database. He opens a URL, sees his emails sorted by importance with narrative briefings.

## Current Status
**Phase 3 of 8 complete. Deployed to GitHub Pages.**

Live: https://jbuckdev.github.io/gmail-dashboard/
Repo: https://github.com/jbuckdev/gmail-dashboard (public)

**Next:** Rule engine rewrite — major upgrade from basic header checks (~65% accuracy) to weighted priority scoring with sender reputation, thread awareness, recipient analysis, ESP detection, platform digests, calendar/OOO handling, and Larry-specific topic taxonomy. Research completed, implementation not started.

## Architecture
- Single `index.html` (~3200 lines, all inline HTML/CSS/JS)
- Google OAuth2 (read-only Gmail access via Google Identity Services)
- Gmail API (client-side, fetches email metadata + bodies)
- Claude Haiku API (classification, topic extraction, narrative briefing)
- GitHub Pages hosting (free HTTPS)
- All processing in Larry's browser — no server, no database

## Key Config
- OAuth Client ID: `663216675668-u9bnqasnncvurnl2ce47l0do2lvsod0k.apps.googleusercontent.com`
- Authorized origins: `http://localhost:8000`, `https://jbuckdev.github.io`
- Larry added as Google OAuth test user
- Gmail scope: `gmail.readonly`

## Privacy Architecture
Binary toggle in Preferences:
- **Full Mode:** AI reads emails. Subject + sender + body sent to Claude Haiku. Anthropic retains up to 30 days for safety. Never used for training.
- **Private Mode:** Zero external API calls. Rule-based classification only. Template briefing. Nothing leaves browser.

ZDR (Zero Data Retention) investigated — enterprise-only, no CORS, incompatible with browser architecture. Abandoned.

## User: Larry
- ~65, .com bubble era, owns medical data analytics company
- Active in community: condo board, elevator replacement project
- Windows + Chrome, not AI-fluent
- Has Claude Pro (web only) — needs separate API key from console.anthropic.com (~$0.30/month)
- Privacy conversation needed before first use

## Features Built
- Onboarding (Claude key + Google Sign-In)
- Dark command-center UI (DM Serif Display + DM Sans)
- Hero briefing with 4 stat cards (Reply/Read/FYI/Skip)
- AI narrative briefing (daily/weekly/monthly toggle, clickable email links)
- Smart body fetching (full body for personal emails, snippet for newsletters)
- Force-directed bubble map (topic visualization, click to drill down)
- In-place sidebar drill-down with breadcrumb nav
- Top Senders ranked visualization
- Volume chart (stacked bars by day)
- Category breakdown bars
- Preferences (VIP contacts, blocked senders, text size, time range, privacy mode, Claude key)
- Scroll zones (sidebar scrolls independently only when overflowing)
- ResizeObserver sidebar sync
- Print stylesheet
- Privacy Mode with template briefing

## Decisions
- Single HTML file, no build step (simplicity for deployment + maintenance)
- Binary privacy mode (all or nothing — subject lines are private data too)
- Public repo (no secrets in code, required for free GitHub Pages)
- AI narrative briefing fetches full email bodies for personal emails only
- Force-directed bubble packing (D3-quality physics simulation, vanilla JS)
- Volume + Breakdown cards outside sticky sidebar (prevents scroll collision)

## Plan
`~/OS/plans/active/2026-03-24-gmail-dashboard.md` (8 phases, 2 appendices)

## Remaining Phases
- Phase 4: Deploy updates (after rule engine rewrite)
- Phase 5: Smart caching (incremental refresh, silent token renewal)
- Phase 6: Security ($5 spend cap, "How This Works" page, privacy conversation)
- Phase 7: Larry onboarding (15-min walkthrough)
- Phase 8: Post-launch tuning
