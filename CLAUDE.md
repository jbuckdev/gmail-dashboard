# GmailDashboard

Single-page email intelligence dashboard for Larry. No backend, no build step.

## Stack
- Single `index.html` (HTML + CSS + JS, all inline)
- Google Identity Services (OAuth2 for Gmail read-only access)
- Gmail API (client-side, fetches email metadata)
- Claude Haiku API (email classification, called from browser)
- Hosted on GitHub Pages (free HTTPS)

## Architecture
- All processing happens in the user's browser
- Claude API key stored in localStorage (user provides their own)
- Google OAuth token managed by Google Identity Services
- No server, no database, no build pipeline

## Key Files
- `index.html` — the entire application
- `CLAUDE.md` — this file
- `GmailDashboard-CONTEXT.md` — project memory

## Deployment
1. Replace `YOUR_CLIENT_ID.apps.googleusercontent.com` in index.html with real Google OAuth client ID
2. Push to GitHub
3. Enable GitHub Pages on the repo
4. Add Larry's Gmail as a test user in Google Cloud Console

## Plan
`~/OS/plans/active/2026-03-24-gmail-dashboard.md`
