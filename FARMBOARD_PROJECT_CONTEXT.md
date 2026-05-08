# FarmBoard — Project Context

> **Purpose:** Bring a fresh Claude Code session up to speed instantly. Paste this at the start of any new session so we don't have to re-explain context.

---

## Current Status (as of May 7, 2026)

**Live and working:**
- **Landing page:** https://farmboardapp.com
- **App:** https://farmboardapp.com/app — full Supabase-backed multi-tenant app with auth, RLS, realtime, mobile layout
- **Domain:** farmboardapp.com (purchased through Vercel)
- **GitHub:** https://github.com/justinburke1987-hash/farmboard (auto-deploys to Vercel from `main`)
- **Supabase project:** `msrduyfwfdpsdrmymvui` — schema deployed, RLS policies live, anon key embedded in client

**What works end-to-end right now:**
1. User goes to farmboardapp.com → sees landing page with Formspree waitlist
2. User goes to /app → signs up or signs in with email/password
3. First-time users land on a Setup screen → name farm + add farm hands
4. Returning users land on the live board (Active Jobs / Done Today / History / Inbox)
5. Add tasks manually, mark complete (with themed confirm modal), restore from Done Today, view all completed jobs grouped by week in History
6. Settings panel: change farm name, upload farm logo, add/remove farm hands
7. Realtime updates via Supabase `postgres_changes` subscriptions
8. Mobile layout works on iPhone (small + large), desktop layout for shop-TV use

**What's NOT built yet (the remaining roadmap):**
- Backend service (Node.js/Express on Railway)
- Twilio SMS webhook → task creation from texts
- Telegram bot integration
- Task assignment to specific farm hands
- SMS reply confirmations
- Stripe billing + plan-tier enforcement
- Polished onboarding flow + landing-page conversion pass

---

## What FarmBoard Is

A SaaS web app for mid-to-large farming operations to replace the whiteboard-and-notebook task management system. Crew texts/messages a dedicated phone number → tasks appear on a shared board → team checks them off → seasonal history is preserved automatically.

**Core insight:** The actual competitor is a whiteboard and a notebook. No existing tool does SMS-in + seasonal history in a farm-specific context.

**Target customer:** US farms 2,000+ acres or multi-employee livestock operations. ~85,000 addressable.

---

## File Structure (current)

```
FarmBoard/
├── index.html                      ← Landing page (Vercel root)
├── app/
│   └── index.html                  ← The Supabase-powered app
├── farmboard_demo.html             ← Original Phase 1 prototype — DO NOT MODIFY
├── Engelstad Farms logo.png        ← Justin's farm logo asset
├── FARMBOARD_PROJECT_CONTEXT.md    ← this file
└── .git/                           ← deploys to Vercel via main branch
```

**Hard rule:** `farmboard_demo.html` is the original Phase 1 prototype. It is preserved as a historical reference. **Never modify it.** All current work happens in `app/index.html` and `index.html`.

---

## Tech Stack (current)

| Layer | Choice | Status |
|---|---|---|
| Frontend | Vanilla HTML / CSS / JS (no framework) | ✅ Live |
| Hosting | Vercel (auto-deploy from GitHub `main`) | ✅ Live |
| Database | Supabase Postgres | ✅ Live |
| Auth | Supabase Auth (email + password, no email confirmation required) | ✅ Live |
| Realtime | Supabase `postgres_changes` subscriptions | ✅ Live |
| Row Level Security | Explicit per-operation policies on all tables | ✅ Live |
| Waitlist capture | Formspree (`xeenkdvq`) | ✅ Live |
| Backend | Node.js / Express on Railway | ❌ Not started |
| SMS | Twilio | ❌ Not started |
| Messaging | Telegram Bot API | ❌ Not started |
| Billing | Stripe | ❌ Not started |

**Why Supabase (chosen during Step 2):** Postgres relational data fits the multi-tenant farm/hand/task model better than Firebase's document store, and RLS gives us multi-tenancy without server-side enforcement.

---

## Supabase Schema (live)

```sql
-- farms: one per signed-up user (owner)
farms (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id uuid NOT NULL REFERENCES auth.users(id),
  name text NOT NULL DEFAULT 'My Farm',
  logo_data text,                -- base64 data URL, max ~500KB
  created_at timestamptz DEFAULT now()
)

-- farm_hands: roster of named senders for the farm
farm_hands (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  farm_id uuid NOT NULL REFERENCES farms(id),
  name text NOT NULL,
  phone text,                    -- nullable, used later for SMS allowlist
  created_at timestamptz DEFAULT now()
)

-- tasks: jobs on the board
tasks (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  farm_id uuid NOT NULL REFERENCES farms(id),
  text text NOT NULL,
  sender_name text,
  source text,                   -- 'MANUAL' | 'TEXT' | 'TELEGRAM' (future)
  priority boolean DEFAULT false,
  done boolean DEFAULT false,
  completed_at timestamptz,
  archived boolean DEFAULT false, -- legacy column, no longer set by app
  created_at timestamptz DEFAULT now()
)
```

**RLS policies (all three tables):** explicit per-operation policies (`select`/`insert`/`update`/`delete`) plus `GRANT` to the `authenticated` role. The original combined `FOR ALL` policy did not work for INSERT — must be split.

```sql
GRANT USAGE ON SCHEMA public TO anon, authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON public.farms       TO authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON public.farm_hands  TO authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON public.tasks       TO authenticated;
```

`farms` policies key on `owner_id = auth.uid()`. `farm_hands` and `tasks` policies key on `farm_id IN (SELECT id FROM farms WHERE owner_id = auth.uid())`.

---

## Current App Behavior (v1 of /app)

### Auth flow
- Sign up or sign in via email + password
- No email confirmation required (turned OFF in Supabase Auth settings)
- Session persists in localStorage (`sb-msrduyfwfdpsdrmymvui-auth-token`)
- Init logic: stay on loading screen until `getSession()` resolves; route to setup/app if session exists, auth screen if not. **Critical:** `submitAuth()` must NOT call `showView()` itself — onAuthStateChange handles all view routing, otherwise a race clobbers the setup screen.

### Board behavior
- **Active Jobs (left panel):** all `done = false` tasks. Sorted: priority desc, then created_at asc.
- **Done Today (right panel):** all `done = true` tasks where `completed_at` is today. Auto-rolls off at midnight (re-renders every 60s client-side).
- **Tap an active task** → themed confirm modal: "Mark this job as complete?"
- **Tap a done task** → restored to Active immediately, no confirm.
- **Amber RESTORE button** on every done task as a clearer affordance.

### History tab
- Pulls every `done = true` task for the farm (regardless of `archived`).
- Groups by ISO week of `completed_at`, most recent week first.
- Three badge styles: green "THIS WEEK", amber "SAME WEEK · {prior year}", neutral year badge for older weeks.
- Tap any historical task → copies its text to today's add bar.

### Settings panel
- Farm name (free text)
- Farm logo upload (base64 data URL stored in `farms.logo_data`, max 500KB)
- Add/remove farm hands (name + optional phone)
- Sign out button

### Mobile layout (≤700px)
- Whole page scrolls naturally; `html/body` locked to `100vw` with `overflow-x:hidden`
- `.tv` and `.tvf` overridden to `flex:none` so they don't collapse to 0 height inside `.main { flex:none }`
- Active Jobs and Done Today stack vertically (Active on top, Done below)
- Add bar **fixed** to viewport bottom with two rows: row 1 sender + priority dot, row 2 input + ADD button
- Input set to 16px font-size to prevent iOS auto-zoom on focus
- SMS demo button hidden on mobile
- Status bar hidden on mobile (counts already in panel header badges)
- Tighter horizontal padding everywhere

### Confirm modal
- Themed in-app modal (replaces native `confirm()`)
- Bebas Neue header, mono buttons, green-bordered quote box for the task text
- Esc cancels, Enter confirms
- Mobile: buttons stack full-width
- Reusable via `confirmDialog({title, message, quote, okLabel, cancelLabel})` returning a Promise

### Debug overlay
- Hidden by default
- Auto-reveals only on amber/red `dbg()` events (errors or timeouts)
- Useful for diagnosing future production issues without ever cluttering the normal UX

---

## Design System (DO NOT CHANGE WITHOUT REASON)

### Colors
```css
--bg:        #0f110e   /* near-black green-tinted background */
--surface:   #161a14   /* cards/panels */
--surface2:  #1e241b   /* hover, inputs */
--border:    #2a3226   /* dividers */
--green:     #7ec850   /* primary accent */
--green-dim: #4a7a2e   /* green backgrounds */
--amber:     #e8a030   /* warnings, history badges, restore */
--amber-dim: #7a5218
--red:       #e05050   /* priority */
--text:      #e6ecdb   /* primary text (brightened May 2026) */
--text-dim:  #a8b596   /* secondary (brightened May 2026) */
--text-muted:#7d8a6a   /* faintest legible (brightened May 2026) */
```

> Text colors were brightened in May 2026 because the original `#d4dbc8 / #7a8a6a / #4a5a3a` was hard to read in office monitors and on phones in sunlight.

### Typography
- Display/headings: **Bebas Neue**
- Mono labels/timestamps: **IBM Plex Mono**
- Body/UI: **IBM Plex Sans**

### Layout
Two-panel split on desktop (Active left, Done Today right). Single-column stacked on mobile. Tabs: Board / History / Inbox. Status bar bottom (desktop only).

---

## Roadmap — Where We Are

### ✅ Phase 1 — Static Prototype (DONE)
Single-page HTML demo with active/done panels, history, inbox sim, manual add, check off, live clock, fullscreen button, simulated incoming SMS. Lives in `farmboard_demo.html`. Preserved untouched.

### ✅ Phase 2 Step 1 — Demo-Ready (DONE in demo file)
- localStorage persistence ✅
- Settings panel (farm name, hands) ✅
- Mobile layout ✅
- Clear-completed → archive flow (later replaced by automatic Done Today rollover) ✅

### ✅ Phase 2 Step 2 — Real Data Layer (DONE — went straight to Supabase, skipped pure localStorage refactor)
- Supabase tables, RLS, grants ✅
- Auth (email + password) ✅
- Multi-tenant data isolation ✅
- Realtime subscriptions ✅
- Settings persistence in DB ✅
- History grouped by week with same-week-prior-year highlight ✅
- Themed confirm modal, restore button, mobile fixes, contrast fixes ✅
- 12-hour AM/PM clock, MM/DD/YY date on tasks ✅
- Done Today auto-rollover (no manual archive step) ✅

### ⏳ Phase 3 — Backend + SMS (NEXT UP)
- [ ] Railway project, Node.js / Express server scaffolded
- [ ] Twilio account, buy first phone number
- [ ] `POST /webhook/sms` → validate sender against `farm_hands.phone` allowlist → INSERT into `tasks` with `source = 'TEXT'`
- [ ] App already subscribes to realtime, so new SMS tasks appear automatically — minimal frontend work
- [ ] Trusted-number management UI in Settings (phone field already exists on farm_hands)
- [ ] Outbound SMS confirmation reply ("✓ Task added: ...")

### ⏳ Phase 4 — Beta With Real Farms
- [ ] Telegram bot integration (parallel to SMS)
- [ ] Task assignment to a specific farm hand
- [ ] Onboarding refinement
- [ ] Basic analytics (tasks/day, completion rate)

### ⏳ Phase 5 — Monetization
- [ ] Stripe billing + subscription management
- [ ] Plan tier enforcement (history limits, contact limits)
- [ ] Landing-page conversion pass
- [ ] Public signup flow

---

## Pricing & Business Model (unchanged)

| Tier | Price | Key Limits |
|---|---|---|
| Starter | $29/mo | 1 phone number, 5 trusted contacts, 90-day history |
| Farm | $49/mo | 1 phone number, unlimited contacts, 2-year history, Telegram |
| Operation | $79/mo | 3 phone numbers, task assignment, multi-user roles |

Cost per farm ~$2–3/mo. Gross margin ~94%. Breakeven ~10 paying farms.

GTM: Ag Facebook groups, county extension offices, co-op partnerships, word of mouth. Creator has 13+ years precision-ag background.

---

## Notes / Conventions for Claude Code Sessions

- **Never modify `farmboard_demo.html`** — it is preserved as-is.
- **Vanilla JS only** for now — no framework until Phase 3 complexity demands it.
- **Color palette and fonts are locked** — don't introduce new ones without explicit user approval.
- **Priority tasks (red dot) sort to top** of Active.
- **localStorage keys and column names match** so the swap to a backend is clean.
- **Anon key is public by design** — embedded in `app/index.html` is fine. Service-role key (Phase 3) must NEVER be in the frontend.
- **Auto-deploy from `main`** — every push to GitHub `main` deploys to Vercel within ~45 seconds. Test locally / ask user before pushing risky changes.
- **Vercel deploys both `index.html` (root) and `app/index.html` (under /app)** — same project, no monorepo split.
- **Supabase RLS:** if a new table is added, must include both `GRANT` to `authenticated` and explicit per-operation policies. Combined `FOR ALL` does not handle INSERT correctly.
- **Mobile-first:** every layout decision should still work on a small iPhone. Test with `@media(max-width:700px)` block.
- **Themed confirm modal:** route any future destructive confirmations through `confirmDialog()` instead of `confirm()`.

---

## Key Identifiers (for next session)

- **Supabase project URL:** `https://msrduyfwfdpsdrmymvui.supabase.co`
- **GitHub repo:** `justinburke1987-hash/farmboard` (branch: `main`)
- **Vercel project:** auto-named `farmboard`, owned by `justinburke1987-6345s-projects`
- **Formspree ID (waitlist):** `xeenkdvq`
- **Domain:** `farmboardapp.com` (purchased through Vercel May 7, 2026)
- **Creator email:** `justin.m.burkepro@gmail.com`
- **Sign-in test account:** `justinburke1987@gmail.com`

---

*Last updated: May 7, 2026 | Phase 2 Step 2 complete, Phase 3 next | Justin Burke*
