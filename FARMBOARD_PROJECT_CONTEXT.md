# FarmBoard — Project Context

> **Purpose of this doc:** Bring Claude Code up to speed instantly so we can iterate on FarmBoard without re-explaining context. Paste this at the start of any new Claude Code session.

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

## Core Feature Set (MVP → V1)

### Phase 1 — Static Prototype (WHERE WE ARE NOW)
- [x] Single-page HTML app with dark industrial aesthetic
- [x] Active tasks panel + completed today panel
- [x] History tab (shows same-week tasks from prior years)
- [x] Inbox log tab (shows incoming message simulation)
- [x] Manual task add via input bar
- [x] Check off / uncheck tasks
- [x] Live clock, fullscreen button
- [x] Simulated incoming SMS task after 6 seconds

**File:** `farmboard.html` — pure HTML/CSS/JS, no dependencies, opens in any browser.

### Phase 2 — Functional Local App (NEXT TARGET)
- [ ] Real data persistence (localStorage for now, then database)
- [ ] Multiple named senders/farm hands stored in settings
- [ ] Allowlist of trusted phone numbers per farm account
- [ ] Task timestamps and sorting
- [ ] Clear completed tasks (archive, not delete)
- [ ] Settings panel (farm name, trusted numbers, display preferences)
- [ ] Mobile-responsive layout for phone use

### Phase 3 — Backend + SMS Integration
- [ ] Node.js or FastAPI backend
- [ ] Twilio webhook: incoming SMS → task creation
- [ ] Telegram Bot API support
- [ ] Supabase database (tasks, farms, users, phone allowlist)
- [ ] Real-time updates via Supabase subscriptions or WebSocket
- [ ] Farm account creation + simple auth (PIN or email/password)

### Phase 4 — Multi-User & Assignment
- [ ] Named farm hand profiles linked to phone numbers
- [ ] Task assignment to specific people
- [ ] Basic notifications (SMS reply confirmation)
- [ ] Admin vs. viewer roles

### Phase 5 — SaaS Infrastructure
- [ ] Stripe billing integration
- [ ] Tiered plans (Starter / Farm / Operation)
- [ ] Onboarding flow
- [ ] Multi-farm account support

---

## Design System

**Aesthetic:** Industrial/utilitarian — built for a shop TV, not a consumer app. High contrast, readable from across a room.

**Color Palette (CSS variables):**
```css
--bg: #0f110e           /* near-black green-tinted background */
--surface: #161a14      /* card/panel background */
--surface2: #1e241b     /* hover states, inputs */
--border: #2a3226       /* dividers */
--green: #7ec850        /* primary accent — active, confirmed */
--green-dim: #4a7a2e    /* green backgrounds */
--amber: #e8a030        /* warnings, history badges */
--red: #e05050          /* priority/urgent */
--text: #d4dbc8         /* primary text */
--text-dim: #7a8a6a     /* secondary text */
--text-muted: #4a5a3a   /* labels, timestamps */
```

**Typography:**
- Display/headings: `Bebas Neue` (Google Fonts)
- Monospace labels/metadata: `IBM Plex Mono`
- Body/UI: `IBM Plex Sans`

**Layout:** Two-panel split on board view (active left, completed right). Tabs for Board / History / Inbox. Status bar at bottom.

---

## Tech Stack Decisions

### Frontend
- **Framework:** Vanilla HTML/CSS/JS for MVP → React when complexity warrants it
- **Hosting:** Vercel (free tier handles this easily)
- **Real-time:** Supabase Realtime subscriptions (no extra infrastructure)

### Backend
- **Runtime:** Node.js (Express) or Python (FastAPI) — decide at Phase 3
- **Hosting:** Railway.app (~$5–20/mo, simple deploys)
- **SMS:** Twilio ($1/mo per number + $0.0079/SMS)
- **Messaging:** Telegram Bot API (free)

### Database
- **Primary:** Supabase (Postgres + Realtime + Auth)
  - Free tier: 500MB storage, 2GB bandwidth
  - Pro tier: $25/mo — handles thousands of farms comfortably
- **Schema (planned):**
  ```
  farms (id, name, created_at, plan_tier)
  farm_phones (id, farm_id, twilio_number, telegram_bot_token)
  trusted_contacts (id, farm_id, phone_number, name, active)
  tasks (id, farm_id, text, sender_name, sender_phone, source, 
         created_at, completed_at, assigned_to, archived)
  ```

### Auth
- Supabase Auth (email/password to start, PIN mode for shop display)

---

## Business Model

**Target customer:** Farms with 2,000+ acres or multi-employee livestock operations. US addressable market ~85,000 operations.

**Pricing tiers:**
| Tier | Price | Key Limits |
|---|---|---|
| Starter | $29/mo | 1 phone number, 5 trusted contacts, 90-day history |
| Farm | $49/mo | 1 phone number, unlimited contacts, 2-year history, Telegram |
| Operation | $79/mo | 3 phone numbers, task assignment, multi-user roles |

**Unit economics (Farm tier):**
- Cost per farm: ~$2–3/mo (hosting share + Twilio)
- Gross margin: ~94%
- Breakeven: ~10 paying farms

**Go-to-market:** Ag Facebook groups, county extension offices, co-op partnerships, word of mouth. Creator has 13+ years precision ag background — strong credibility advantage.

---

## Current File Structure

```
farmboard/
├── farmboard.html          ← MVP prototype (complete, shareable now)
├── FARMBOARD_PROJECT_CONTEXT.md   ← this file
```

**Planned structure (Phase 2+):**
```
farmboard/
├── index.html              ← entry point
├── app.js                  ← main app logic
├── styles.css              ← extracted styles
├── data/
│   └── store.js            ← localStorage abstraction → swap for Supabase later
├── components/
│   ├── TaskList.js
│   ├── TaskItem.js
│   ├── HistoryView.js
│   ├── InboxLog.js
│   └── Settings.js
├── backend/                ← Phase 3
│   ├── server.js
│   ├── routes/
│   │   ├── webhooks.js     ← Twilio/Telegram incoming
│   │   └── tasks.js
│   └── db/
│       └── supabase.js
└── FARMBOARD_PROJECT_CONTEXT.md
```

---

## Development Roadmap

### Step 1 — Demo-Ready (This Week)
**Goal:** A version you can show your father-in-law and brother-in-law on a laptop or phone without any server needed.

- [ ] Improve the HTML prototype with localStorage persistence (tasks survive page refresh)
- [ ] Add a Settings panel: farm name, farm hand names
- [ ] Add ability to simulate "incoming text" with a sender name selector
- [ ] Mobile layout pass — make it usable on a phone screen
- [ ] Add a "clear completed" button that archives to history

**How to share for demo:** Just send the `farmboard.html` file. They open it in Chrome. No hosting needed. Works completely offline.

### Step 2 — Real Data Layer
**Goal:** Tasks persist properly, multi-session, groundwork for backend.

- [ ] Refactor to component-based JS (no framework yet, just clean modules)
- [ ] localStorage store with schema matching planned Supabase schema
- [ ] Settings persistence (farm name, contacts list)
- [ ] History grouped by week/year properly
- [ ] "Same week last year" history computation from stored data

### Step 3 — Backend + SMS (First Real Users)
**Goal:** Actual texts create actual tasks.

- [ ] Set up Railway project + Node.js Express server
- [ ] Twilio account + buy first phone number
- [ ] Webhook handler: POST /webhook/sms → validate sender → create task
- [ ] Supabase project + tables + Row Level Security
- [ ] Frontend connects to Supabase for real-time task list
- [ ] Simple farm account creation (email + password)
- [ ] Trusted number allowlist management in Settings

### Step 4 — Beta With Real Farms
**Goal:** 2–5 farms using it daily for 30 days.

- [ ] Telegram bot integration
- [ ] Task assignment (assign to a named farm hand)
- [ ] SMS confirmation reply ("✓ Task added: Check oil on JD tractor")
- [ ] Onboarding flow (account setup, add your number, test it)
- [ ] Basic analytics (tasks/day, completion rate)

### Step 5 — Monetization
**Goal:** Stripe billing, public launch.

- [ ] Stripe integration + subscription management
- [ ] Plan tier enforcement (history limits, contact limits)
- [ ] Landing page (farmboard.app or similar)
- [ ] Public signup flow

---

## Key Decisions Still Open

1. **React vs. vanilla JS** — Stay vanilla until Phase 3 complexity demands it. Keep the barrier to iteration low.
2. **Node vs. Python backend** — Node preferred for Twilio/Supabase ecosystem alignment.
3. **One Twilio number per farm vs. shared number with farm codes** — One number per farm is cleaner UX but costs $1/farm/mo. Shared number with a prefix (`[FARM]`) is cheaper but worse UX. Lean toward one-per-farm.
4. **Telegram vs. SMS priority** — SMS first (universal), Telegram as bonus. Some farm crews already use it.
5. **Domain name** — `farmboard.app` or `getfarmboard.com` — check availability.

---

## Demo Instructions (Showing the HTML Prototype)

1. Send `farmboard.html` to the person via email, text, AirDrop, etc.
2. They open it in **Chrome or Safari** (any modern browser)
3. It works completely offline — no internet needed after file opens
4. To demo SMS-in: wait 6 seconds after opening, a task from "Tom" auto-appears
5. Click any task to check it off
6. Type in the bottom bar + hit Add or Enter to add tasks manually
7. Click the History tab to show seasonal memory feature
8. Hit the ⛶ Fullscreen button to show shop TV mode

**Talking points for demo:**
- "Instead of the whiteboard, this is on the TV in the shop"
- "Anyone on the farm texts the number, it shows up here in seconds"
- "This tab shows what you were doing this same week last year — useful for pre-planting prep reminders"
- "Eventually you can check things off from your phone too"

---

## Notes for Claude Code Sessions

- Always maintain the established color palette and typography — do not introduce new fonts or change the dark industrial aesthetic
- Keep the codebase in vanilla JS as long as possible — resist framework complexity until Phase 3
- localStorage keys should match the planned Supabase schema column names — makes the swap clean
- The "same week last year" history feature is a key differentiator — make sure it always works correctly with real date logic
- Priority tasks (red dot) should always sort to top of active list
- The status bar at the bottom is always visible — it's the at-a-glance health indicator

---

*Last updated: May 2026 | Creator: Justin Burke | Contact: justin.m.burkepro@gmail.com*
