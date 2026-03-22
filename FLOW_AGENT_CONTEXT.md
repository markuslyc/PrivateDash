# FLOW Agent Context File
**Version:** 1.0 — March 2026  
**Owner:** Markus (markus@go-genie.com)  
**Live URL:** https://maspaceship.netlify.app  
**GitHub Repo:** https://github.com/markuslyc/PrivateDash (branch: main)  
**Auto-deploy:** Netlify auto-deploys within 60–90s of a commit to main

---

## What is FLOW?

FLOW is Markus's personal command center — a single-file HTML app (index.html) that acts as a productivity dashboard. It is **not** a GO-GENIE product; it is Markus's personal operating system for running his life and business. It is hosted on Netlify, protected by Google OAuth login (invite-only), and built to be completely asset-light (no backend, no database — all state in localStorage).

**Design language:** Liquid glass morphism. Atkinson Hyperlegible (primary font, dyslexia-friendly), DM Sans (display), DM Mono (code). Warm dark navy text (#1a2332) on an animated gradient background (sky blue → lavender → mint). Frosted glass cards with backdrop-filter blur. Teal/mint accent colours. Generous line height (1.75), spring-bounce transitions.

---

## File Structure

```
PrivateDash/
├── index.html       ← Main dashboard (the entire app lives here)
├── login.html       ← Google OAuth entry point via Netlify Identity
├── admin.html       ← Admin panel (user approval, role management)
└── _redirects       ← Netlify redirect rules (/ → /login.html)
```

**Critical:** The entire dashboard is a single HTML file. All JS, CSS, and logic are inline. Do not split into separate files unless explicitly asked.

---

## Auth & Access Control

- **Provider:** Netlify Identity + Google OAuth
- **Google OAuth Client ID:** `387362798277-hljh6f7alu3tf85dpee6pvn4c2c3klk6.apps.googleusercontent.com`
- **Authorised origin:** `https://maspaceship.netlify.app`
- **Init pattern (critical):**
  ```javascript
  netlifyIdentity.init({
    APIUrl: 'https://maspaceship.netlify.app/.netlify/identity'
  });
  ```
- **Registration:** Invite-only (set in Netlify Identity settings)
- **Role system:** Users get `approved` role via Netlify Identity after admin approval. Admin role = `admin`. Role check via `user.app_metadata.roles`.
- **User data scoping:** All localStorage keys are scoped per user email using `getUserKey(email)` helper to avoid data collisions between users.
- **Admin calendar access:** Admin can view approved users' Google Calendars. Tokens shared via `flow_gcal_token_{emailKey}` localStorage pattern set during login.

---

## Current Features (as of March 2026)

### Tabs / Views
| Tab | Description |
|-----|-------------|
| **Home** | Widget grid: Tasks, Calendar, AI assistant |
| **Gmail Agent** | AI-powered inbox — fetch, summarise, draft replies, send |
| **Settings** | Per-widget visibility, accent colour, layout density, export |

### Gmail Agent (detailed)
- Fetches emails via Gmail API
- Auto-summarises each email into bullet points using Claude (`claude-sonnet-4-20250514`)
- Point-form reply drafting: user types notes → Claude drafts professional email
- One-click send
- Filters: All / Unread / Important

### Gmail → Tasks Integration
- Modal extracts action items from emails using Claude
- Fields: due date, due time, priority, email reminder interval
- Reminders via Google Calendar events with email notification overrides (15min, 1hr, 3hr, 1day, 2day before)

### Settings Tab
- Per-widget visibility toggles
- Accent colour swatches: teal / green / blue / purple / amber / red
- Layout density options
- Profile display + sign out
- Data export

### Home View Widgets
- `tasks-widget` — task list with priorities
- `cal-widget` — Google Calendar integration
- `ai-widget` — quick AI assistant

---

## Architecture Decisions (Do Not Reverse Without Asking)

1. **Single HTML file** — all CSS, JS, and markup inline in index.html. Keeps deployment simple and avoids CORS/import issues on Netlify.
2. **No backend** — all state in localStorage, scoped by user email.
3. **Netlify Identity for auth** — not Firebase, not Supabase. Do not switch providers.
4. **Claude API for AI features** — model: `claude-sonnet-4-20250514`. API key is handled by the Anthropic proxy (not hardcoded).
5. **Google OAuth via Netlify Identity** — do not implement a separate Google OAuth flow.
6. **CSS variables for theming** — all colours reference CSS variables so accent colour swaps work globally.

---

## Known Bugs / Technical Debt (as of March 2026)

| # | Bug | Status | Notes |
|---|-----|--------|-------|
| 1 | `setLayout()` crash on residual `biz-widget` references | Fixed | Businesses section removed, widget removed from DOM |
| 2 | Home view widgets missing from HTML | Fixed | `tasks-widget`, `cal-widget`, `ai-widget` added |
| 3 | Stray extra `</div>` in home view section | Fixed | Was ejecting all views from DOM as orphans |
| 4 | `.view` CSS using `position:absolute` | Fixed | Changed to relative — was causing views to render off-screen |
| 5 | CM6 editor `execCommand('insertText')` corrupts emoji/unicode | Known | Use `view.dispatch()` method instead for GitHub editor edits |

---

## Deployment Workflow (Agent Instructions)

### For small targeted edits:
Use the GitHub web editor CM6 dispatch method:
```javascript
const view = document.querySelector('.cm-content').cmTile.view;
const doc = view.state.doc.toString();
const fixed = doc.replace(OLD_STRING, NEW_STRING);
view.dispatch({ changes: { from: 0, to: doc.length, insert: fixed } });
```
Then commit. Netlify auto-deploys in 60–90s.

### For full file replacements:
Use GitHub's **Add File → Upload Files** interface. Do NOT use `execCommand('insertText')` — it corrupts multi-byte Unicode characters (emojis, em-dashes).

### Reading the current file:
GitHub raw endpoint: `https://github.com/markuslyc/PrivateDash/raw/main/index.html`  
Note: GitHub CSP blocks `fetch()` calls from within the editor page itself.

### Verifying deploy:
Check `https://maspaceship.netlify.app` after 90s. Login with Google to verify auth flow.

---

## Feature Backlog (Priority Order)

> **Agent instruction:** Work through this list top-to-bottom unless Markus specifies otherwise. After completing each item, update this file to mark it done and move to the next.

| Priority | Feature | Status | Notes |
|----------|---------|--------|-------|
| 1 | **Calendar: Add & Edit Events (Google Calendar sync)** | Done | Deployed March 2026 |
| 2 | **Gmail: Auto-pull tasks + AI reply drafting** | Done | Deployed March 2026 |
| 3 | **Tasks: Attention & Approval queue view** | Done | Deployed March 2026 |
| 4 | **GO-GENIE Email Agent (Full Inbox Command Centre)** | Done | Deployed March 2026 |

---

## Feature Specs

### P1 — Calendar: Add & Edit Events (Google Calendar sync)

**Goal:** Markus can create new events and edit existing events directly in FLOW, with changes syncing to Google Calendar in real time.

**Requirements:**
- "Add Event" button on the cal-widget and Calendar view
- Event creation modal: title, date, start time, end time, location (optional), description (optional), guests (optional)
- Click an existing event to open edit modal — pre-populated with current values
- Save = write to Google Calendar via Calendar API (POST for new, PATCH for edit)
- Delete option in edit modal
- After save/edit/delete, refresh the calendar view immediately (no manual reload)
- Use existing Google OAuth token (already obtained during login) — do not create a new auth flow
- Error handling: if token is expired, prompt re-auth gracefully

**Do not build:** A separate calendar page. This extends the existing cal-widget and calendar view only.

---

### P2 — Gmail: Auto-pull tasks + AI reply drafting

**Goal:** FLOW automatically surfaces emails that need action and helps Markus draft replies without him having to think.

**Requirements:**

*Auto-pull tasks from email:*
- On Gmail Agent load, scan inbox for emails that require action (use Claude to classify)
- Extract action items and auto-create tasks in the task list with: source email subject, sender, suggested due date, priority (Claude-inferred)
- Link task back to the originating email (clicking task opens the email)
- Deduplication: don't re-create tasks for emails already processed (track by email ID in localStorage)

*AI reply drafting:*
- In Gmail Agent, each email has a "Draft Reply" button
- User clicks → modal opens with: the original email thread, a notes field ("what do you want to say?"), a "Generate Draft" button
- Claude generates a professional reply based on notes + thread context
- User can regenerate, edit inline, then send with one click
- Tone options: Professional / Concise / Friendly (default: Professional)

---

### P3 — Tasks: Attention & Approval Queue

**Goal:** Markus can see at a glance what requires his personal attention or approval, separate from general tasks.

**Requirements:**
- New task type/flag: "Needs Attention" and "Needs Approval" (in addition to existing priority levels)
- Dedicated section at the top of the Tasks view: "🔴 Requires Your Action" — shows only flagged tasks
- Tasks can be flagged manually OR auto-flagged when pulled from email (P2 feature)
- Each task in this queue shows: title, source (email/manual), due date, a one-click "Done" and "Delegate" action
- "Delegate" opens a quick compose to forward/assign via email
- Badge count on the Tasks nav icon showing number of pending attention/approval items
- Filters: All / Attention / Approval / Done

---

## Agent Behaviour Rules

1. **Read this file at the start of every session.** Do not rely on memory.
2. **Read the current index.html before making any changes.** Never edit blind.
3. **Make the smallest change that solves the problem.** Do not refactor unrelated code.
4. **After every deploy, verify the live site loads and auth works.**
5. **Update this context file** after completing any feature or fixing any bug — add to Known Bugs if new issues are found, update backlog status, increment version number.
6. **Never hardcode API keys.** Auth and API keys are handled externally.
7. **Preserve the design language.** Do not change fonts, colour system, or animation style unless explicitly asked.
8. **If a change would affect auth or the login flow, flag it to Markus before proceeding.**

---

## Markus's Operating Preferences

- **Communication style:** Direct. No fluff. Show the result, explain only what's essential.
- **Autonomy level:** Full. Make decisions, deploy, report back with what was done and why.
- **Escalate only when:** A change affects auth, data structure, or would break existing features.
- **Preferred format for updates:** One-liner summary of what changed + live URL to verify.

---

---

### P4 — GO-GENIE Email Agent (Full Inbox Command Centre)

**Goal:** A dedicated tab in FLOW that processes Markus's GO-GENIE inbox end-to-end — summarising, triaging, actioning, and replying — with Markus approving before anything is sent or committed.

**Inbox:** GO-GENIE work email (markus@go-genie.com) via Gmail API.

**Tab name:** "Email Agent" — sits in the main FLOW navigation.

---

**Section 1 — Inbox Feed**
- On load, fetch the last 50 unread emails from the GO-GENIE inbox
- Each email displayed as a card showing: sender, subject, time received, and a 2–3 line AI summary (Claude-generated, auto-runs on load)
- Colour-coded triage badge on each card (Claude-assigned):
  - 🔴 Action Required — needs Markus to do something
  - 🟡 Approval Needed — needs Markus to sign off
  - 🟢 FYI — informational, no action needed
  - 📅 Calendar — contains a meeting, event, or deadline
- Click a card to expand the full email thread
- Filters across the top: All / Action / Approval / FYI / Calendar

---

**Section 2 — Action Panel (appears when email is expanded)**
Each expanded email shows four smart action buttons:

1. **Create Task** — extracts action items from the email using Claude, opens a pre-filled task creation modal (title, due date, priority, source email linked). Saves to FLOW task list with "Needs Attention" or "Needs Approval" flag.

2. **Draft Reply** — opens reply modal with:
   - Full email thread shown for context
   - Notes field: "What do you want to say?" (Markus types bullet points)
   - Tone selector: Professional / Concise / Friendly (default: Professional)
   - "Generate Draft" button → Claude writes full professional reply
   - Markus can edit inline or click "Regenerate"
   - "Send for Approval" button → shows final preview with To/Subject/Body before sending
   - Markus clicks "Confirm & Send" → email sent via Gmail API
   - Nothing is ever sent without Markus's explicit confirmation

3. **Add to Calendar** — Claude extracts date, time, and event details from the email, opens pre-filled calendar event creation modal (feeds into P1 Calendar feature). Markus reviews and confirms before saving to Google Calendar.

4. **Mark FYI / Archive** — marks email as read and archives it. No Claude needed.

---

**Section 3 — Meeting Summary Executor**
- Detect emails that contain meeting summaries, action items from meetings, or follow-up notes
- Claude extracts: decisions made, action items assigned to Markus, deadlines mentioned
- Displays a structured "What You Need To Do" panel:
  - Each action item shown with suggested due date and priority
  - One-click "Add to Tasks" per item
  - One-click "Schedule Follow-up" per item (creates calendar event)
- Markus reviews the full list and confirms which items to action

---

**Section 4 — Approval Queue**
- Dedicated sub-tab within Email Agent: "Awaiting My Approval"
- Aggregates all emails tagged 🟡 Approval Needed
- Each item shows: what is being requested, who is requesting, deadline if mentioned
- Actions: Approve (drafts confirmation reply for Markus to review) / Reject (drafts decline reply) / Delegate (opens forward modal)
- Nothing sent without Markus's confirmation

---

**Technical requirements:**
- Gmail API scope: gmail.readonly + gmail.send + gmail.modify (for archive/read)
- Use existing Google OAuth token from FLOW login — do not create new auth flow
- All processed email IDs stored in localStorage to avoid reprocessing
- Claude model: claude-sonnet-4-20250514
- All AI calls happen client-side via Anthropic proxy (no backend)
- Every send/calendar/task action requires explicit Markus confirmation — no auto-send ever

**Do not build:** A full email client. This is a triage and action tool only. Composing new emails from scratch is out of scope.

---

*Last updated: March 2026 — v1.4 P1/P2/P3/P4 all done*
