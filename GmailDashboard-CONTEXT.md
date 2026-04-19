# GmailDashboard — Context

## What This Is
AI-powered Gmail intelligence dashboard for Larry (~65yo CEO of a medical data analytics company, also runs community projects like condo board elevator replacement). Single `index.html`, no server, no database. He opens a URL, sees his emails sorted by importance with narrative briefings and interactive topic visualization. Inline delete actions on every email list.

## Current Status
**Ops redesign shipped 2026-04-19 (round 2).** Complete dashboard rebuild after first-round Atelier was deemed "still scattered cards." Mission-control density inspired by OSControlCenter: top mono pulse strip, single-screen no-scroll layout, bubble map hero, 4-pillar triage, sparkline strip at the bottom, fixed right-rail Top Senders column. Briefing card killed (no per-load Claude API cost). Onboarding rebuilt as 3-side-by-side mission-control panel. iCloud bridge + Deep Clean tab from earlier in the day still in place. Larry's Gmail flow preserved.

Live: https://jbuckdev.github.io/gmail-dashboard/
Repo: https://github.com/jbuckdev/gmail-dashboard (public)

## Architecture
- Single `index.html` (~8500 lines, all inline HTML/CSS/JS)
- Provider-aware fetch layer (`mailFetch`, `mailFetchBatchMetadata`, `messageURL`, `isIcloud`):
  - **Gmail path:** Google OAuth2 (readonly → modify lazy upgrade); Gmail API client-side
  - **iCloud path:** local FastAPI bridge at `127.0.0.1:4851` (`~/OS/services/icloud-bridge/`, launchd KeepAlive). App-specific password in macOS Keychain. Bridge emits Gmail-API-shaped JSON so the dashboard's `parseMessage` works unchanged. See `reference/reference_icloud_bridge.md` in memory for full details.
- Claude Haiku API (`claude-haiku-4-5-20251001`) — classification, organic topics, narrative briefing
- GitHub Pages hosting (free HTTPS)
- All processing in user's browser
- Sandboxed-iframe email reader modal renders full HTML email body via `format=full` fetch
- IndexedDB caches: main `inbox-intel-cache` (schema v2, parseMessage shape) + `inbox-intel-deep-clean` (sender-aggregated scans)

## Classification Engine (v3 + hybrid)
- **Stage 1: Noise filter (rules)** — catches ~40-50%: OOO, bounces, calendar responses, platform notifications, Gmail Promotions/Social, noreply newsletters
- **Stage 2: AI analysis (recent only)** — emails from last 30 days go to Claude. AI assigns category (reply/read/fyi/skip), organic topic names, summaries, action items. Topics consistent across batches via "topics used so far" context.
- **Older emails (>30 days): rules-only** — zero API calls. `guessTopicFromRules()` for topics. Free.
- **Private Mode:** rules handle everything, template briefing. Zero external calls.

## Time Ranges
- 6 options: 1D, 1W, 1M, 3M, 6M, 1Y
- Email cap scales: 500 (1D/1W), 2000 (1M), 5000 (3M), 15000 (6M/1Y)
- AI classification only for last 30 days regardless of range (~$0.08-0.10 max)
- Older emails use free rules-only classification
- Gmail API rate limiting handled: retry with backoff, adaptive batch sizes

## Visual Design
- Glassmorphism (backdrop-filter blur) on all cards, panels, overlays
- Animated dot grid canvas background (mouse-reactive, scatter on approach)
- Spotlight cursor effect on all glass surfaces
- Dark mode: deep black (#070a14), semi-transparent card backgrounds, glow effects
- Light mode: "Warm Stone" palette (#e8e6e1), muted accents, subtle noise grain
- Spring easing transitions throughout
- Click-to-expand: clicking any panel opens fullscreen overlay (no expand buttons)

## Briefing System
- **All ranges:** Analytics briefing — stat grid (total, avg/month, replies, topics), monthly volume bars, clickable top senders, topic bars, top domains, busiest day
- **≤30 days:** AI narrative briefing appended below analytics. Reads full email bodies, writes prose summary with clickable [REF:id] links to Gmail.
- **>30 days:** Analytics only (instant, zero API cost)
- Clickable senders everywhere: unexpanded Top Senders rows, expanded sender cards, analytics briefing senders — all use `openSenderEmails()` → slide-over or overlay sidebar with emails + delete actions
- Expanded panel interactivity: sender cards, volume bars, category items all clickable → open email sidebar within overlay

## Delete System (replaced old Cleaning Mode)
- Inline action bar on every email list: "Delete All" + "Select" buttons
- Select mode: click to toggle emails, "Delete X" button appears with count
- Scoped per grouping (section, drill-down, landscape sidebar, sender emails)
- Lazy scope upgrade: `gmail.readonly` → `gmail.modify` on first delete
- `gmail.modify` CANNOT send emails (verified)
- Trash only (30-day Gmail recovery), never permanent delete
- Confirmation for 10+ emails, 3-second countdown for 50+
- `.confirm-overlay` z-index: 10000 (above slide-over at 901, above expanded panels)
- 10-second undo window via toast
- `trashEmails()` uses Gmail `batchModify` API (up to 1000/call), with individual-request fallback
- `ensureModifyScope()` for auth, `ensureValidToken()` refresh between batches
- Failure reporting: user sees count of succeeded vs failed if any drop

## Expanded Panels
- Full-screen modal overlays with cinematic animation
- Per-type enhanced layouts:
  - Briefing: two-column reading layout
  - Email Sections: 2x2 grid, all sections expanded
  - Top Senders: card grid (12), avatar + email + count + bar. **Clickable** — opens sender emails in overlay sidebar via `openSenderEmails(name, email)`
  - Volume: tall stacked bars (240px) with 14-day history + color legend. **Clickable** — each bar calls `openDayEmails(dateKey, label)` to show that day's emails
  - Breakdown: donut ring chart (conic-gradient, animated spin-in) + category list. **Clickable** — each category calls `openCategoryEmails(key)`
  - Action Items: card grid with avatars, subjects, action text
  - Email Landscape: full-width canvas, force-directed physics, legend strip (150px), drill sidebar (320px), pan/zoom
- `showEmailsPanel(title, emailList)` — generic function renders filtered emails in either overlay sidebar or slide-over drill panel (preserves DOM structure, no innerHTML destruction)

## Cache
- IndexedDB (replaced sessionStorage — no size limit)
- Keyed by time range
- Incremental refresh: only fetches new emails, merges with cache

## Features
- Onboarding with privacy choice BEFORE login (Full Analysis vs Private Mode)
- Light/dark theme toggle (respects prefers-color-scheme, flash-prevention)
- Bento grid layout (4-col responsive → 2-col → 1-col)
- Click-to-expand on every panel (fullscreen overlay)
- Hero strip: greeting + 6 range buttons + last updated
- Force-directed bubble map with interactive zoom and click-drag panning
- Back button on bubble map for drill navigation
- Slide-over panel for drill-down email lists
- Top Senders, Volume Chart, Category Breakdown visualizations
- Smart caching with IndexedDB
- Silent OAuth token refresh (auto-renews at 50min, retries on 401)
- Preferences (VIP contacts, blocked senders, text size, privacy mode, Claude key)
- Help & Troubleshooting page
- Comprehensive error handling with Send Report
- Print stylesheet

## Key Config
- OAuth Client ID: `663216675668-u9bnqasnncvurnl2ce47l0do2lvsod0k.apps.googleusercontent.com`
- Authorized origins: `http://localhost:8000`, `https://jbuckdev.github.io`
- Scopes: `gmail.readonly` (default), `gmail.modify` (lazy, for delete)
- Larry added as Google OAuth test user
- Claude model: `claude-haiku-4-5-20251001`

## Privacy
Binary toggle in Preferences:
- **Full Mode:** AI reads emails. Content sent to Claude. Anthropic retains up to 30 days for safety. Never used for training.
- **Private Mode:** Zero external API calls. Rule-based only. Nothing leaves browser.

## Atelier redesign (2026-04-19)

Driven by `~/OS/plans/active/2026-04-19-inbox-intelligence-atelier-redesign.md`.

- **Sidebar workspace layout.** 220px expanded → 64px collapsed (< 1100px) → topbar (< 700px). Tabs: Dashboard / Deep Clean. Bottom: Refresh, Help, Preferences, Theme. Account email at the bottom.
- **Editorial typography.** Fraunces (warm contemporary serif w/ optical sizing) for display + numerals; DM Sans + Instrument Sans for body/UI. Tabular numerals for stats.
- **Burnished ochre accent.** `--accent-primary: #c9892b` (dark) / `#a06a1c` (light) — replaces the generic blue for active state, primary CTAs, hero numerals. Blue retained for links + status.
- **Hero strip.** Editorial date (day-of-week eyebrow + month/day in Fraunces) + greeting + summary + range chips. Hairline rule below.
- **Top Senders centerpiece.** Full-width section (70% bars / 30% summary stats card). Each row: serif name + sans email + horizontal ochre bar + tabular numeral count + % of inbox. Click → opens reader on most recent email from that sender.
- **Email reader modal.** Sandboxed iframe rendering, `format=full` body, Open-unsubscribe-page action button + Trash-this-email button. ESC closes.
- **Deep Clean tab (NEW).** "Years of inbox, one sweep." Range chips 1Y/2Y/3Y/4Y/5Y/All → live scan estimate via `resultSizeEstimate` → Begin scan → progress bar with cancel → sortable filterable sender table (Sender / Domain / Count / Latest / Unsub) → multi-select checkboxes → sticky bulk-action bar (Trash N emails / Block sender / Open unsubscribe / Clear).
- **Static dot grid background** (replaced the old animated canvas that pegged GPU; pure CSS radial-gradient pattern keyed off `--dot-color`).

## Remaining Work
- Phase 7 (Larry onboarding): create Anthropic account, seed VIP contacts, 15-min walkthrough, privacy conversation
- Phase 8 (post-launch tuning) based on Larry's feedback
- Optional: aggressive Phase 4 cleanup of inline delete bars from dashboard cards (deferred — they still work, but cleanup belongs in Deep Clean conceptually)
- Optional: virtual scrolling for sender table when > 500 unique senders

## Decisions
- AI assigns organic topics, not fixed taxonomy — "Elevator Inspection" not "Community"
- Rules only filter obvious noise; AI handles everything that matters
- Binary privacy mode (all or nothing — subject lines are private data too)
- Lazy scope upgrade for delete (don't scare with permissions on first visit)
- gmail.modify not gmail.full (trash but NEVER send or permanent delete)
- claude-haiku-4-5-20251001 (old ID was 404ing — critical fix)
- Briefing API call uses `getClaudeKey()` not stored variable (was `claudeKey` → undefined)
- Error handling: catches 529/overload, "load failed", shows actual error in fallback message
- Hybrid classification: only last 30 days use AI, older = free rules-only
- IndexedDB over sessionStorage for cache (year of emails exceeds 5MB)
- Analytics briefing on all ranges; AI narrative only for ≤30 days
- Inline delete per email list, not global Clean mode
