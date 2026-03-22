# FLOW Agent Context File
**Version:** 1.1 — March 2026
**Owner:** Markus (markus@go-genie.com)
**Live URL:** https://maspaceship.netlify.app
**GitHub Repo:** https://github.com/markuslyc/PrivateDash (branch: main)
**Auto-deploy:** Netlify auto-deploys within 60–90s of a commit to main

---

## What is FLOW?

FLOW is Markus's personal command center — a single-file HTML app (index.html) that acts as a productivity dashboard. It is **not** a GO-GENIE product; it is Markus's personal operating system for running his life and business. It is hosted on Netlify, protected by Google OAuth login (invite-only), and built to be completely asset-light (no backend, no database — all state in localStorage).

**Design language:** Liquid glass morphism. Atkinson Hyperlegible (primary font, dyslexia-friendly), DM Sans (display), DM Mono (code). Warm dark navy text (#1a2332) on an animated gradient background (sky blue to lavender to mint). Frosted glass cards with backdrop-filter blur. Teal/mint accent colours. Generous line height (1.75), spring-bounce transitions.

---

## File Structure
```
PrivateDash/
├── index.html       <- Main dashboard (the entire app lives here)
├── login.html       <- Google OAuth entry point via Netlify Identity
├── admin.html       <- Admin panel (user approval, role management)
└── _redirects       <- Netlify redirect rules (/ -> /login.html)
```

**Critical:** The entire dashboard is a single HTML file. All JS, CSS, and logic are inline. Do not split into separate files unless explicitly asked.

---

## Auth & Access Control

- **Provider:** Netlify Identity + Google OAuth
- **Google OAuth Client ID:** 387362798277-hljh6f7alu3tf85dpee6pvn4c2c3klk6.apps.googleusercontent.com
- **Authorised origin:** https://maspaceship.netlify.app
- **Init pattern (critical):**
  netlifyIdentity.init({ APIUrl: 'https://maspaceship.netlify.app/.netlify/identity' });
- **Registration:** Invite-only
- **Role system:** Users get approved role via Netlify Identity after admin approval. Admin role = admin. Role check via user.app_metadata.roles.
- **User data scoping:** All localStorage keys are scoped per user email using getUserKey(email) helper.
- **Admin calendar access:** Admin can view approved users Google Calendars. Tokens shared via flow_gcal_token_{emailKey} localStorage pattern.

---

## Current Features (as of March 2026)

### Tabs / Views
- Home: Widget grid — Tasks, Calendar, AI assistant
- Gmail Agent: AI-powered inbox — fetch, summarise, draft replies, send
- Settings: Per-widget visibility, accent colour, layout density, export

### Gmail Agent
- Fetches emails via Gmail API
- Auto-summarises each email into bullet points using Claude (claude-sonnet-4-20250514)
- Point-form reply drafting: user types notes, Claude drafts professional email
- One-click send. Filters: All / Unread / Important

### Gmail to Tasks Integration
- Modal extracts action items from emails using Claude
- Fields: due date, due time, priority, email reminder interval
- Reminders via Google Calendar events with email notification overrides (15min, 1hr, 3hr, 1day, 2day before)

### Settings Tab
- Per-widget visibility toggles
- Accent colour swatches: teal / green / blue / purple / amber / red
- Layout density, profile display, sign out, data export

### Home View Widgets
- tasks-widget: task list with priorities
- cal-widget: Google Calendar integration
- ai-widget: quick AI assistant

---

## Architecture Decisions (Do Not Reverse Without Asking)

1. Single HTML file — all CSS, JS, markup inline in index.html.
2. No backend — all state in localStorage, scoped by user email.
3. Netlify Identity for auth — not Firebase, not Supabase.
4. Claude API for AI features — model: claude-sonnet-4-20250514. API key handled by Anthropic proxy, never hardcoded.
5. Google OAuth via Netlify Identity — do not implement a separate OAuth flow.
6. CSS variables for theming — all colours reference CSS variables.

---

## Known Bugs / Technical Debt (as of March 2026)

1. setLayout() crash on residual biz-widget references — FIXED
2. Home view widgets missing from HTML — FIXED
3. Stray extra closing div in home view section — FIXED
4. .view CSS using position:absolute — FIXED (changed to relative)
5. CM6 editor execCommand corrupts emoji/unicode — KNOWN. Use view.dispatch() method instead.

---

## Deployment Workflow

### Small targeted edits:
Use GitHub web editor CM6 dispatch:
  const view = document.querySelector('.cm-content').cmTile.view;
  const doc = view.state.doc.toString();
  const fixed = doc.replace(OLD, NEW);
  view.dispatch({ changes: { from: 0, to: doc.length, insert: fixed } });
Then commit. Netlify auto-deploys in 60-90s.

### Full file replacements:
Use GitHub Add File -> Upload Files interface.
Do NOT use execCommand('insertText') — corrupts emojis and em-dashes.

### Reading current file:
GitHub raw: https://github.com/markuslyc/PrivateDash/raw/main/index.html

### Verifying deploy:
Check https://maspaceship.netlify.app after 90s. Login with Google to verify auth.

---

## Feature Backlog (Priority Order)

Work top-to-bottom unless Markus specifies otherwise.
After completing each item, update this file and mark it done.

P1 — Calendar: Add & Edit Events (Google Calendar sync) — DONE
P2 — Gmail: Auto-pull tasks + AI reply drafting — PENDING
P3 — Tasks: Attention & Approval queue view — PENDING

---

## Feature Specs

### P1 — Calendar: Add & Edit Events

Goal: Markus can create and edit events in FLOW, synced to Google Calendar in real time.

Requirements:
- Add Event button on cal-widget and Calendar view
- Creation modal: title, date, start time, end time, location (optional), description (optional), guests (optional)
- Click existing event to open edit modal, pre-populated with current values
- Save = write to Google Calendar API (POST for new, PATCH for edit)
- Delete option in edit modal
- After save/edit/delete, refresh calendar view immediately
- Use existing Google OAuth token from login — do not create a new auth flow
- If token expired, prompt re-auth gracefully
- Do NOT build a separate calendar page — extend existing cal-widget only

### P2 — Gmail: Auto-pull tasks + AI reply drafting

Goal: FLOW surfaces emails that need action and helps Markus draft replies.

Auto-pull tasks:
- On Gmail Agent load, scan inbox and classify emails requiring action using Claude
- Extract action items, auto-create tasks with: source email subject, sender, suggested due date, Claude-inferred priority
- Link task back to originating email
- Deduplication: track processed email IDs in localStorage, never re-create

AI reply drafting:
- Each email has a Draft Reply button
- Click opens modal with: original thread, notes field, Generate Draft button
- Claude drafts professional reply from notes + thread context
- User can regenerate, edit inline, send with one click
- Tone options: Professional / Concise / Friendly (default: Professional)

### P3 — Tasks: Attention & Approval Queue

Goal: Markus sees what needs his personal attention or approval, separate from general tasks.

Requirements:
- New task flags: Needs Attention and Needs Approval
- Dedicated section at top of Tasks view: Requires Your Action — flagged tasks only
- Tasks flagged manually OR auto-flagged when pulled from email (P2)
- Each task shows: title, source, due date, one-click Done and Delegate
- Delegate opens quick compose to forward/assign via email
- Badge count on Tasks nav icon for pending items
- Filters: All / Attention / Approval / Done

---

## Agent Behaviour Rules

1. Read this file at the start of every session. Do not rely on memory.
2. Read the current index.html before making any changes. Never edit blind.
3. Make the smallest change that solves the problem. Do not refactor unrelated code.
4. After every deploy, verify the live site loads and auth works.
5. Update this context file after completing any feature or fixing any bug.
6. Never hardcode API keys.
7. Preserve the design language — fonts, colour system, animations.
8. If a change affects auth or login flow, stop and ask Markus first.

---

## Markus's Operating Preferences

- Communication: Direct. No fluff. Show the result, explain only what is essential.
- Autonomy: Full. Make decisions, deploy, report back with what was done and why.
- Escalate only when: change affects auth, data structure, or would break existing features.
- Update format: One-liner summary of what changed + live URL to verify.

---

Last updated: March 2026 — v1.2
Next update: After P2 Gmail auto-pull tasks is complete
