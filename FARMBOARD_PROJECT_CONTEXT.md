# FarmBoard — Project Context

> **Purpose of this doc:** Bring Claude Code up to speed instantly so we can iterate on FarmBoard without re-explaining context. At the start of any new session just say "read the project context" and Claude will pull this file from GitHub via `gh`.

---

## What Is FarmBoard?

A lightweight SaaS web app for mid-to-large farming operations to replace the whiteboard/notebook task management system. Farm hands text a dedicated phone number → tasks appear on a shared board → team checks them off → history is preserved seasonally.

**Core insight:** The actual competitor is a whiteboard and a notebook. No existing tool does SMS-in + seasonal history in a farm-specific context.

---

## The Problem It Solves

At a typical operation (3–6 people), task coordination happens via whiteboards, notebook paper, and verbal communication. This breaks down during busy seasons. FarmBoard puts the task board on any screen — shop TV, phone, tablet — and lets anyone add tasks by texting a number.

---

## Current Status — Phase 3 In Progress (LIVE)

**Live URL:** https://farmboardapp.com/app

**GitHub repo:** https://github.com/justinburke1987-hash/farmboard

**Local clone:** `C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo`

**Key file:** `app/index.html` — entire live app in one file (vanilla JS + Supabase)

**Hosting:** Vercel — connected to GitHub main branch, auto-deploys on every push (~30–60s)

---

## Dev Setup & Push Workflow

### Pushing changes
```powershell
# Edit app/index.html at the local clone, then:
cd "C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo"
git add app/index.html
git commit -m "your message"
git push
# Vercel deploys automatically
```

### Git identity (already configured in this repo — no need to set again on this machine)
- `user.email = justinburke1987@gmail.com`
- `user.name = Justin Burke`

### GitHub CLI (`gh`)
- Installed at: `C:\Program Files\GitHub CLI\gh.exe`
- Authenticated as: `justinburke1987-hash`
- **PATH must be refreshed in each new Claude Code terminal session:**
```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
gh <command>
```

### Setting up on a new computer
1. Install Claude Code (claude.ai/code)
2. Install Git (git-scm.com)
3. `git clone https://github.com/justinburke1987-hash/farmboard`
4. `git config user.email "justinburke1987@gmail.com"` and `git config user.name "Justin Burke"`
5. Install gh CLI: `winget install --id GitHub.cli` then `gh auth login`
6. Open Claude Code in the cloned folder and say "read the project context"

---

## Supabase

**Project URL:** `https://msrduyfwfdpsdrmymvui.supabase.co`

**Anon key:** In `app/index.html` (public/safe — anon key only, RLS enforced)

**Current tables:**
- `farms` — `id, owner_id, name, logo_data, created_at`
- `farm_hands` — `id, farm_id, name, phone, created_at`
- `tasks` — `id, farm_id, text, sender_name, source, done, priority, assigned_to, completed_at, archived, created_at`

**Auth:** Supabase Auth — email/password. One Supabase user = one farm (owner). Session tokens auto-refresh every hour, now handled silently.

**Realtime:** Supabase Realtime subscriptions on `tasks` table, filtered by `farm_id`.

---

## What Is Fully Working Right Now

- [x] Supabase Auth — account creation, sign in, persistent sessions
- [x] Farm auto-creation on first login
- [x] First-time setup screen (farm name + crew members)
- [x] Two-panel task board — Active Jobs (left), Done Today (right)
- [x] Filter Active Jobs by Assign To (crew member dropdown in panel header)
- [x] Add bar — Assignee (who's adding) → task text → Assign To → priority → ADD
- [x] Priority flag (red dot) — sorts to top
- [x] Per-task assignment dropdown (reassign after creation)
- [x] Confirm dialog on task completion (prevents accidental taps)
- [x] History tab — completed jobs grouped by ISO week
- [x] "SAME WEEK" badge — prior-year groups matching current calendar week
- [x] Inbox tab — incoming message log
- [x] Realtime subscriptions — tasks sync across devices instantly
- [x] Settings overlay — farm name, logo upload, crew management
- [x] Toast notifications
- [x] Live clock + date in header
- [x] Fullscreen button (shop TV mode)
- [x] Mobile-responsive layout — fixed add bar at bottom, single-column
- [x] Status bar at bottom (desktop only)

---

## Recent Bug Fixes & Changes (May 2026)

- **TOKEN_REFRESHED handled silently** — Supabase renews auth tokens hourly. Previously triggered a full `loadFarm()` reload, interrupting the shop TV every hour. Now completely silent.
- **Removed getSession() race condition** — The IIFE `getSession()` call at init was racing against `onAuthStateChange` and could push users to the auth screen after a successful load.
- **Timeouts increased** — `withTimeout` values raised 8000ms → 20000ms on all Supabase queries. Display fallback raised to 25s.
- **RETRY button on error** — Loading error now shows RETRY as primary action. "Sign out" is a subtle secondary link — prevents users accidentally signing themselves out on a network hiccup.
- **Realtime reconnect on visibility change** — WebSocket reconnects automatically when the page comes back into focus (important for shop TV).
- **Filter by Assign To** — Crew member filter dropdown in Active Jobs panel header.
- **Assign To on task creation** — Add bar now sets assignment at creation time, not just after.
- **SMS demo button removed** — Was for demo only, now gone.

---

## What Is NOT Done Yet

### Next up — Crew Join Flow (Phase 4, planned)
The goal: one admin account owns the farm, crew members join using a 6-char code without needing individual passwords.

**Planned flow:**
1. Admin generates a join code in Settings (stored in `join_codes` table: `farm_id`, `code`, `is_active`)
2. Crew member goes to `farmboardapp.com/join`, enters code + name + phone number
3. Added to `members` table (`farm_id`, `name`, `phone`, `role`)
4. Device gets a persistent member token stored in localStorage → auto-signs in on return visits
5. Phone number stored at join → links directly to Twilio SMS sender when backend is built

**New tables needed:**
- `join_codes` — `id, farm_id, code (6-char), is_active`
- `members` — `id, farm_id, name, phone, role (enum), token`

**Why this approach:**
- No passwords for crew to forget
- Phone number captured at join = Twilio SMS attribution works automatically
- Shop TV stays signed in as admin all day, unchanged
- If crew member gets a new phone, they re-join with the same code

### Phase 3 remaining — SMS Backend
- [ ] Node.js/Express backend on Railway
- [ ] Twilio webhook: incoming SMS → task creation, attributed via phone number lookup in `members`
- [ ] Telegram Bot API support (secondary)

### Phase 4 — Multi-User
- [ ] Crew join flow with join codes (see above)
- [ ] Role-based access (admin vs. crew)
- [ ] Member phone numbers linked to Twilio SMS identity

### Phase 5 — SaaS Infrastructure
- [ ] Stripe billing + plan tier enforcement
- [ ] Landing page + public onboarding
- [ ] Domain: `farmboard.app` or `getfarmboard.com`

---

## Design System

**Aesthetic:** Industrial/utilitarian — built for a shop TV, not a consumer app. High contrast, readable from across a room. Do NOT introduce new fonts or alter this aesthetic.

**Color Palette:**
```css
--bg: #0f110e           /* near-black green-tinted background */
--surface: #161a14      /* card/panel background */
--surface2: #1e241b     /* hover states, inputs */
--border: #2a3226       /* dividers */
--green: #7ec850        /* primary accent */
--green-dim: #4a7a2e
--amber: #e8a030        /* warnings, history */
--amber-dim: #7a5218
--red: #e05050          /* priority/urgent */
--text: #e6ecdb
--text-dim: #a8b596
--text-muted: #7d8a6a
```

**Typography:** Bebas Neue (display), IBM Plex Mono (labels/meta), IBM Plex Sans (body/UI)

**Layout:** Two-panel board (active left, done right). Tabs: Board / History / Inbox. Status bar always visible at bottom on desktop.

---

## Tech Stack

| Layer | Current | Planned |
|---|---|---|
| Frontend | Vanilla HTML/CSS/JS — `app/index.html` | Stay vanilla through Phase 4 |
| Hosting | Vercel (free tier) | Same |
| Database | Supabase (Postgres + Realtime + Auth) | Same |
| SMS | — | Twilio ($1/mo/number + $0.0079/SMS) |
| Backend | — | Node.js/Express on Railway |
| Auth | Supabase Auth (admin) | + Member token flow (crew) |

---

## Business Model

| Tier | Price | Limits |
|---|---|---|
| Starter | $29/mo | 1 phone number, 5 contacts, 90-day history |
| Farm | $49/mo | 1 number, unlimited contacts, 2-year history, Telegram |
| Operation | $79/mo | 3 numbers, task assignment, multi-user roles |

**Target:** US farms 2,000+ acres (~85,000 operations). Justin has 13+ years precision ag background.
**GTM:** Ag Facebook groups, county extension offices, co-op partnerships.

---

## File Structure

```
farmboard/                          ← GitHub repo root
├── app/
│   └── index.html                  ← LIVE app (Supabase-connected)
├── farmboard_demo.html             ← Offline demo (localStorage, no backend)
├── index.html                      ← Vercel root redirect
├── FARMBOARD_PROJECT_CONTEXT.md    ← This file
└── .gitignore
```

---

## Rules for Claude Code Sessions

- Maintain the color palette and typography — no changes to the aesthetic
- Keep vanilla JS — no frameworks until Phase 3 backend demands it
- The "same week last year" history feature is the key differentiator — preserve it
- The app runs all day on a shop TV — never interrupt the running app with loading screens for background operations (token refresh, realtime reconnect, etc.)
- Always push via git from local clone — direct GitHub API file edits are unreliable
- Refresh PATH before using `gh` in new terminal sessions

---

*Last updated: May 19, 2026 | Creator: Justin Burke | justinburke1987@gmail.com*
