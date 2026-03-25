# GmailDashboard — Context

## What This Is
Email intelligence dashboard for Larry (~65yo entrepreneur, Windows user, not AI-fluent). He opens a URL, sees his emails sorted by importance with one-line summaries. Zero install, minimal setup.

## Current Status
**Phase 1 of 6 — Foundation & Auth**
- Project folder created
- index.html built with full UI: onboarding, dashboard, preferences, print styles
- Needs: Google Cloud OAuth client ID, GitHub Pages deployment

## User: Larry
- ~65, .com bubble era, owns company, community projects, heavy email volume
- Windows + Chrome
- Has Claude Pro subscription; needs separate API key from console.anthropic.com (~$0.30/month)
- UX: large text, high contrast, no jargon, no AI branding, print-friendly

## Setup Flow
1. Larry opens URL
2. Pastes Claude API key (one time)
3. Clicks "Sign in with Google" (one time)
4. Dashboard is live on every subsequent visit

## Plan
`~/OS/plans/active/2026-03-24-gmail-dashboard.md`

## Decisions
- Client-side only architecture (privacy, zero infrastructure)
- Larry provides own Claude API key (self-sufficient, not dependent on James)
- Google OAuth (familiar UX, read-only scope)
- Claude Haiku for classification (fast, cheap, good enough)
- Hybrid classification: rules first (~65%), AI for ambiguous remainder (~35%)
