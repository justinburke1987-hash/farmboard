# FarmBoard — Project Context

> **Purpose of this doc:** Bring Claude Code up to speed instantly so we can iterate on FarmBoard without re-explaining context. Paste this at the start of any new Claude Code session — or Claude's memory system will load it automatically.

---

## What Is FarmBoard?

A lightweight SaaS web app for mid-to-large farming operations to replace the whiteboard/notebook task management system. Farmers and farm hands text or message a dedicated phone number → tasks appear on a shared board → team checks them off → history is preserved seasonally.

**Core insight:** The actual competitor is a whiteboard and a notebook. No existing tool does SMS-in + seasonal history in a farm-specific context.

---

## The Problem It Solves

At a typical operation (example: family farm with 3–6 people), task coordination happens via:
- Physical whiteboard in the shop
- Notebook sheets of paper
- Informal verbal communication

This breaks down during busy seasons (planting, harvest) when coordination across people and equipment is critical. FarmBoard puts the task board on any screen — shop TV, phone, tablet — and lets anyone add tasks by texting a number.

---

## Current Status — Phase 3 In Progress (LIVE)

**Live URL:** https://farmboardapp.com/app

**GitHub repo:** https://github.com/justinburke1987-hash/farmboard

**Local clone:** `C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo`

**Key file:** `app/index.html` — single-file app (vanilla JS + Supabase integration)

**Hosting:** Vercel — connected to GitHub main branch, auto-deploys on every push to main.

### How to push changes

```powershell
# Edit app/index.html locally, then:
cd "C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo"
git add app/index.html
git commit -m "your message"
git push
# Vercel deploys automatically in ~30-60 seconds
```

**Git identity already configured** in the repo:
- `user.email = justinburke1987@gmail.com`
- `user.name = Justin Burke`

**GitHub CLI (`gh`) is installed** at `C:\Program Files\GitHub CLI\gh.exe` and authenticated as `justinburke1987-hash`. To use it in a new terminal session, refresh PATH first:
```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
gh <command>
```

### What is fully working right now

- [x] Supabase Auth — account creation, sign in, session persistence
- [x] Farm auto-creation on first login
- [x] First-time setup screen (farm name + crew members)
- [x] Task board — active jobs left panel, done today right panel
- [x] Priority flag (red dot) — sorts to top
- [x] Task assignment to crew members
- [x] Archive to history — grouped by ISO week
- [x] "SAME WEEK" badge — prior-year groups matching current calendar week highlighted green
- [x] Inbox tab — incoming message log
- [x] Realtime subscriptions — tasks sync across devices via Supabase Realtime
- [x] Settings overlay — farm name, logo upload, crew management
- [x] Simulate SMS button
- [x] Toast notifications
- [x] Live clock + date in header
- [x] Fullscreen button
- [x] Mobile-responsive layout (fixed add bar at bottom on mobile)
- [x] Status bar always visible at bottom (desktop)
- [x] Confirm dialog on task completion (prevents accidental taps)
- [x] Task assignment dropdown per task

### Recent bug fixes (May 2026)

- **TOKEN_REFRESHED silent handling** — Supabase refreshes the auth token every hour. Previously this triggered `loadFarm()` and threw up the loading spinner, interrupting the shop TV. Now token refreshes are silent.
- **Removed getSession() race condition** — The IIFE `getSession()` call at init was racing against `onAuthStateChange` and could push users to the auth screen even after a successful load.
- **Timeouts increased** — `withTimeout` values raised from 8000ms → 20000ms on all Supabase queries. Display fallback raised to 25s.
- **RETRY button** — Loading error now shows a green RETRY button as primary action. "Sign out" is a subtle secondary link so users don't accidentally sign themselves out on a network hiccup.
- **Realtime reconnect on visibility change** — If the WebSocket drops while the shop TV is in the background, it reconnects automatically when the page becomes visible again.

### What is NOT done yet (next up)

- [ ] **Twilio SMS integration** — Real incoming texts → tasks (Phase 3 backend)
- [ ] **Telegram Bot** — Secondary messaging channel
- [ ] **Node.js/Express backend on Railway** — Twilio webhook receiver
- [ ] **Trusted phone number allowlist enforcement** — Currently cosmetic only
- [ ] **Multi-farm / multi-user roles** — Admin vs. viewer (Phase 4)
- [ ] **Stripe billing** — Plan tier enforcement (Phase 5)
- [ ] **Landing page** — farmboard.app or getfarmboard.com

---

## localStorage Keys (legacy — now Supabase)

The original prototype used localStorage. The live app uses Supabase. These keys no longer drive the live app but are preserved in `farmboard_demo.html` for offline demos.

---

## Supabase

**Project URL:** `https://msrduyfwfdpsdrmymvui.supabase.co`

**Anon key:** In `app/index.html` (public/safe — anon key only, RLS enforced)

**Tables:**
- `farms` — `id, owner_id, name, logo_data, created_at`
- `farm_hands` — `id, farm_id, name, phone, created_at`
- `tasks` — `id, farm_id, text, sender_name, source, done, priority, assigned_to, completed_at, archived, created_at`

**Auth:** Supabase Auth — email/password. Session tokens auto-refresh every hour (now handled silently in the client).

**Realtime:** Supabase Realtime subscriptions on the `tasks` table, filtered by `farm_id`.

---

## Phase Roadmap

### Phase 1 — Static Prototype ✅ COMPLETE
Single-file HTML app. All core UI and interactions working.

### Phase 2 — Functional Local App ✅ COMPLETE
localStorage persistence, settings, sender selector, simulate SMS, archive to history, same-week detection, sort control, priority toggle UI.

### Phase 3 — Backend + SMS Integration 🔄 IN PROGRESS
- [x] Supabase project + tables + RLS
- [x] Supabase Auth (email/password)
- [x] Realtime subscriptions
- [x] Frontend live at farmboardapp.com/app on Vercel
- [ ] Node.js/Express backend on Railway
- [ ] Twilio webhook: incoming SMS → task creation
- [ ] Telegram Bot API support

### Phase 4 — Multi-User & Assignment
- [ ] SMS reply confirmation
- [ ] Admin vs. viewer roles
- [ ] Named farm hand profiles linked to phone numbers

### Phase 5 — SaaS Infrastructure
- [ ] Stripe billing + subscription management
- [ ] Plan tier enforcement
- [ ] Landing page + public onboarding

---

## Design System

**Aesthetic:** Industrial/utilitarian — built for a shop TV, not a consumer app. High contrast, readable from across a room. Do NOT introduce new fonts or alter this aesthetic.

**Color Palette (CSS variables):**
```css
--bg: #0f110e           /* near-black green-tinted background */
--surface: #161a14      /* card/panel background */
--surface2: #1e241b     /* hover states, inputs */
--border: #2a3226       /* dividers */
--green: #7ec850        /* primary accent — active, confirmed */
--green-dim: #4a7a2e    /* green backgrounds */
--green-glow: rgba(126,200,80,0.12)
--amber: #e8a030        /* warnings, history badges */
--amber-dim: #7a5218
--red: #e05050          /* priority/urgent */
--text: #e6ecdb         /* primary text */
--text-dim: #a8b596     /* secondary text */
--text-muted: #7d8a6a   /* labels, timestamps */
```

**Typography:**
- Display/headings: `Bebas Neue` (Google Fonts)
- Monospace labels/metadata: `IBM Plex Mono`
- Body/UI: `IBM Plex Sans`

**Layout:** Two-panel split on board (active left, completed right). Tabs for Board / History / Inbox. Status bar always visible at bottom (hidden on mobile).

---

## Tech Stack

### Frontend
- **Current:** Vanilla HTML/CSS/JS — single file `app/index.html`
- **Hosting:** Vercel (free tier, auto-deploy from GitHub main)
- **Realtime:** Supabase Realtime subscriptions

### Backend (Phase 3 — not yet built)
- **Runtime:** Node.js (Express)
- **Hosting:** Railway.app
- **SMS:** Twilio ($1/mo per number + $0.0079/SMS)
- **Messaging:** Telegram Bot API (free, secondary)

### Database
- **Primary:** Supabase (Postgres + Realtime + Auth)

### Auth
- Supabase Auth — email/password for admin

---

## Business Model

**Target customer:** Farms with 2,000+ acres or multi-employee livestock operations. US addressable market ~85,000 operations.

| Tier | Price | Limits |
|---|---|---|
| Starter | $29/mo | 1 phone number, 5 trusted contacts, 90-day history |
| Farm | $49/mo | 1 phone number, unlimited contacts, 2-year history, Telegram |
| Operation | $79/mo | 3 phone numbers, task assignment, multi-user roles |

**Unit economics (Farm tier):** ~$2–3/mo cost per farm. ~94% gross margin. Breakeven at ~10 paying farms.

**GTM:** Ag Facebook groups, county extension offices, co-op partnerships, word of mouth. Justin has 13+ years precision ag background — strong credibility.

---

## Key Decisions Still Open

1. **Telegram vs. SMS priority** — SMS first (universal), Telegram as bonus
2. **One Twilio number per farm vs. shared number with farm codes** — Lean toward one-per-farm (cleaner UX, $1/farm/mo)
3. **Domain name** — `farmboard.app` or `getfarmboard.com` — check availability
4. **React vs. vanilla JS** — Stay vanilla until Phase 3 backend complexity demands it

---

## Current File Structure

```
farmboard/                          ← GitHub repo root
├── app/
│   └── index.html                  ← LIVE app (Supabase-connected)
├── farmboard_demo.html             ← Offline demo (localStorage only, no backend)
├── index.html                      ← Vercel root redirect
├── FARMBOARD_PROJECT_CONTEXT.md    ← This file
└── .gitignore
```

Local clone: `C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo`

---

## Demo Instructions

1. Go to https://farmboardapp.com/app
2. Sign in (or create an account)
3. Check off a task → it moves to Done Today
4. Click the **History** tab → see archived tasks grouped by week, with "SAME WEEK" badge on prior-year matching weeks
5. Hit **📱 SMS** to simulate an incoming text from a farm hand
6. Open **Settings** (⚙) to set the farm name and add crew members
7. Hit **⛶ Fullscreen** to show shop TV mode

**Demo talking points:**
- "Instead of the whiteboard, this is on the TV in the shop"
- "Anyone on the farm texts the number, it shows up here in seconds"
- "This tab shows what you were doing this same week last year — useful for pre-planting prep reminders"
- "History builds up automatically as you use it — it's not just a log, it's seasonal memory"
- "It stays up on the TV all day — anyone can see what's open and what's done"

---

## Notes for Claude Code Sessions

- Always maintain the established color palette and typography — no new fonts, no aesthetic changes
- Keep the codebase in vanilla JS — resist framework complexity until Phase 3 backend warrants it
- The "same week last year" history feature is the key product differentiator — preserve it
- The status bar at the bottom must always be visible on desktop
- The app is designed to stay open all day on a shop TV — never interrupt the running app with loading screens for background operations (token refresh, etc.)
- Push changes via git from the local clone at `C:\Users\justi\OneDrive\Desktop\Farmboard\farmboard-repo`
- Refresh PATH before using `gh` CLI in new PowerShell sessions (see GitHub CLI note above)

---

*Last updated: May 17, 2026 | Creator: Justin Burke | Contact: justin.m.burkepro@gmail.com*
