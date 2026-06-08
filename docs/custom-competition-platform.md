# Custom Competition Platform — Prototype Assessment & Build Roadmap

**Companion to:** `docs/site-audit.md` and `docs/event-platform-integration-plan.md`
**Subject:** the `cft-app` prototype (Vite + React 19) you built for in-house competition management
**Prepared:** June 2026

---

## 0. The decision this reflects

You've decided you want to **build your own competition platform that lives on crossfittaylors.com**, rather than run it on PushPress or Competition Corner. This document assesses the prototype you built and lays out the path from "promising mockup" to "real platform that isn't manual."

**Scope note:** this is specifically about the **competition / event engine**. The separate question of day-to-day *gym operations* (memberships, recurring billing, class booking) is not the same problem — there, buying PushPress is still very likely the better call (see `event-platform-integration-plan.md`). Building your own competition tool and buying gym-ops software are not mutually exclusive.

---

## 1. What you've built — a genuinely good start

`cft-app` is a single-file React (Vite) app with three views:

- **Schedule** — WODs with heat times, teams grouped per heat, plus a "Team Heat Groups" reference accordion.
- **Leaderboard** — ranked results table with medals for the top 3 and per-WOD score columns.
- **Admin** (password-gated) — *Team Profiles* (edit team/athlete names, division, heat, lane) and *Score Entry* (enter each team's score per WOD; totals auto-calc).

What's right about it:
- **The mental model is correct:** teams → heats/lanes → WODs → scores → leaderboard, with an admin gate. That's the real shape of a competition.
- **The design is legitimately nice** — dark theme, color-coded heats, division badges, podium medals, search, responsive tables, a "sync status" affordance. This is the hard-to-fake part, and you nailed the UX.
- **A Supabase project already exists** (its URL is in `src/supabase.js`). That's exactly the backend you'll build on.

You proved the interface. That's worth a lot. The gap now is everything *behind* the interface.

---

## 2. Honest technical state: it's a front-end mockup, not yet an app

Right now the prototype is a UI with no backend. Specifically:

### 🔴 It won't run as-is — three blocking issues
1. **Hooks aren't imported.** `App.jsx` only does `import React from 'react'` but uses `useState` and `useCallback` throughout → `ReferenceError: useState is not defined`. Fix: `import React, { useState, useCallback } from 'react'`.
2. **`App.jsx` imports itself and mounts twice.** Lines 1–7 of `App.jsx` import `CFTCompApp` *from `./App.jsx`* (itself) and call a second `ReactDOM.createRoot(...).render(...)` — but `main.jsx` already mounts the app via `index.html`. Those 7 lines are a copy-paste artifact and should be deleted.
3. **`@supabase/supabase-js` is not installed.** `src/supabase.js` imports it, but it's absent from `package.json` and `node_modules`. So even the Supabase client can't load.

### 🔴 There is no persistence
`App.jsx` never references `supabase.js` — all teams and scores live in **in-memory React state**. Consequences:
- The **"Save / Sync" button is cosmetic**: `handleSync` just runs `setTimeout(() => setSyncStatus("synced"), 1200)`. It writes nothing.
- A **page refresh wipes everything.**
- The "live" leaderboard is live **only inside one browser tab** — a judge on a phone and the TV on the wall would not see each other's data.

### 🟡 Everything is hardcoded in source — *this is your "too manual on entry"*
The roster (`INITIAL_TEAMS`), the WODs, heat times, divisions, and lanes are all hardcoded constants. **Running a new event means editing code and redeploying.** That's the friction you're feeling, and §3 is the fix.

### 🟡 The scoring is a placeholder
`computeLeaderboard` sums raw numeric scores (`total += parseFloat(score)`) and ranks one combined list, "more is better." Real functional-fitness scoring needs:
- **Per-WOD scoring types** (a 16-min *time* where lower is better vs. *reps* where higher is better vs. *load*),
- **Placing points** (rank within each WOD → points; sum points; lowest/highest wins by ruleset),
- **Per-division leaderboards** (RX shouldn't be ranked against Scaled),
- **Tiebreakers**.
Today, summing a time and a rep count produces a meaningless total.

### 🟡 Security is demo-grade
The admin password (`cft2025`) and the Supabase anon key are committed in client source. Anyone can read the JS and unlock admin. Fine for a fun in-house board; **not** fine once registration data, emails, or payments are involved (see §9).

> None of this is a knock — it's exactly the right stage to be at. You validated the experience; now it needs a backend and a front door.

---

## 3. The one problem to solve first: kill manual entry

"Too manual on the entry side" has a precise fix: **self-service registration that writes straight into the same database the app reads from.**

```
Athlete registers (+ pays)  →  a `registrations` row is created
        →  admin assigns heats/lanes (manual or auto-balance)
        →  the SAME roster powers Schedule, Score Entry, and Leaderboard
```

No retyping, no editing `INITIAL_TEAMS`, no redeploy. The roster becomes data, not code. Everything else in the roadmap builds on this.

---

## 4. Target architecture (build, on your site)

| Layer | Choice | Why |
|---|---|---|
| **Frontend** | Your React app, refactored, hosted at `compete.crossfittaylors.com` (or embedded at `/compete`) | Reuse what you built; keep it independent of the marketing-site platform |
| **Database + backend** | **Supabase** (you already have a project) | Postgres + Auth + **Realtime** + Row-Level Security + Edge Functions — covers persistence, login, live updates, and serverless payment handling in one |
| **Payments** | **Stripe** Checkout, confirmed by a Supabase **Edge Function** webhook | Your own Stripe account is the rail; webhook flips a registration to `paid` |
| **Live updates** | **Supabase Realtime** | Judge enters a score on a phone → wall-display leaderboard updates instantly across devices |
| **Auth** | **Supabase Auth** (magic link for athletes; admin/judge roles via RLS) | Replaces the client-side password with real, enforceable roles |

### Schema sketch
```
events         (id, name, date, divisions[], status)
registrations  (id, event_id, team_name, athlete1, athlete2, email, phone,
                division, payment_status, stripe_session_id, created_at)
teams          (id, event_id, registration_id, heat, lane)      -- assignment layer
wods           (id, event_id, name, order, cap,
                scoring_type [time|reps|load|points], ascending bool)
scores         (id, event_id, team_id, wod_id, raw_value,
                judge_id, validated bool, entered_at)
profiles       (user_id, role [admin|judge|athlete])
```
The leaderboard becomes a computed view: for each WOD, rank teams *within division* by `raw_value` honoring `scoring_type`, award placing points, sum, break ties. The current "sum of raw values" gets replaced here.

---

## 5. Phased build plan

| Phase | Goal | Key work | Outcome |
|---|---|---|---|
| **1 — Make it real** | Running + persistent | Fix the 3 blocking bugs; install `@supabase/supabase-js`; create tables; replace in-memory state with Supabase reads/writes; make "Save" actually upsert; enable **Realtime** on `scores` | Your *current* event, but data survives refresh and the leaderboard is truly live on every device/the wall |
| **2 — Kill manual entry** | Self-serve roster | Public **registration form** → writes `registrations`; admin **"Assign heats & lanes"** screen (manual drag + an auto-balance button) | Athletes enter their own info; you stop editing code |
| **3 — Payments** | Registration → paid | **Stripe Checkout** on registration; Edge Function webhook sets `payment_status=paid`; capacity caps, waitlist, confirmation email (Mailchimp/Resend), refunds | Registration→payment→roster, hands-off |
| **4 — Real scoring** | Correct results | Per-WOD scoring types, **placing points**, **per-division leaderboards**, tiebreakers, judge-enter → admin-validate workflow; public vs. judge views | Trustworthy, rule-correct standings |
| **5 — Day-of + close** | Run the event | Check-in view, athlete announcements/notifications, heat-schedule reminders, **TV/wall leaderboard mode**, export results, archive/"close" event with final standings | The full lifecycle you described |
| **6 — Multi-event + hardening** | Reusable & secure | Make `event` first-class so you reuse it forever; real admin/judge auth + **RLS**; move secrets to env vars | A durable platform, not a one-off |

**You get the biggest relief at the end of Phase 2** (registration replaces hand-entry) and a genuinely "live" board at the end of Phase 1.

---

## 6. Putting it on crossfittaylors.com

This is the same mechanism described in `event-platform-integration-plan.md` (Depth 2/3): host the app on a **branded subdomain** (`compete.crossfittaylors.com`) via a GoDaddy DNS **CNAME**, or **iframe-embed** it into a page. The competition app is fully independent of whatever the marketing site runs on, so you can ship it now and it keeps working if/when you move the marketing site off GoDaddy later.

---

## 7. Build-vs-buy — an honest take, since you've chosen "build"

What you gain by building: **full control, your brand end-to-end, no per-ticket fees, exactly your scoring rules, and a reusable owned asset.**

What you take on (eyes open):
- You become the **software vendor**: you own scoring correctness, payment edge cases (refunds, disputes, capacity), PII/security, and **event-day reliability** — a crash at 9:00am on comp day is yours to fix.
- The things Competition Corner gives for ~4% + $2/ticket — validated scoring, bulletproof live leaderboards, an athlete app, and support — are non-trivial to match.

Pragmatic guidance:
- Building is very reasonable **if** you'll run **several events a year** (amortizes the effort) and want a distinctive, owned product. For a once-a-year throwdown, buying is cheaper and lower-risk.
- A sensible hybrid: **build the front door and the simple in-house events** (registration, schedule, leaderboard for your own throwdowns/Bring-a-Friend/Community-WOD-style events), and keep **Competition Corner in your back pocket** for a large or sanctioned competition. Not either/or.

---

## 8. What I can do next (just say go)

1. **Make the prototype run** — fix the three blocking bugs (5-minute change), so you can actually demo it.
2. **Stand up the Supabase schema and wire the app to it** — real persistence + Realtime, so "Save" works and the leaderboard is live across devices (Phase 1).
3. **Scaffold the registration form + Stripe Checkout** — the actual cure for manual entry (Phases 2–3).

Tell me which to start with and I'll begin. (If you want, I can also commit a cleaned-up copy of the prototype into this repo so we version it from here.)

---

## 9. Security to-dos (do these regardless of pace)

- The **Supabase anon key** in `src/supabase.js` is only safe to expose **if Row-Level Security is enabled and correct on every table.** Confirm RLS is on before any real registration/PII/payment data goes in. If the project ever ran with RLS off, **rotate the key.**
- **Replace the client-side admin password** (`cft2025`) with Supabase Auth + an `admin`/`judge` role enforced server-side via RLS. A password baked into shipped JavaScript protects nothing.
- Move secrets/config to **environment variables**, not source.
