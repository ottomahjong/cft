# Event-Platform Integration Plan
### How to bring PushPress / Competition Corner registration, payment, email, check-in, and leaderboards into crossfittaylors.com

**Companion to:** `docs/site-audit.md` (see §2 and §8)
**Prepared:** June 2026

---

## 0. The one thing to internalize first

You asked to "build PushPress / Competition Corner *into the website build itself*, integrated with GoDaddy's Stripe and database capabilities." Two corrections up front, because they change the whole plan:

1. **GoDaddy Website Builder (Websites + Marketing) is a marketing-site builder, not an application platform.** It gives you pages and a light Online Store. It does **not** give you a programmable database, server-side code, or a reliable place to run third-party JavaScript apps. So you cannot literally *build* a registration/scoring/leaderboard system "into" it.

2. **You don't want to rebuild the engine anyway.** Registration → payment → email → check-in → workout announcements → scheduling → leaderboards → close is *exactly* what PushPress and Competition Corner already are. The smart goal is to **own the front door** (it lives at crossfittaylors.com and feels like your site) while **the platform owns the engine** (it does the hard, regulated, real-time stuff). "Integrated into the website" = embed/redirect, branded to your domain — not a from-scratch rebuild.

The good news: most of what you want is achievable, and one of your instincts is correct — **your own Stripe account can absolutely be the payment rail.** Let's map exactly what each piece can and can't do.

---

## 1. What GoDaddy *can* and *can't* do (verified)

| Capability | GoDaddy Websites + Marketing | Notes |
|---|---|---|
| **Connect your own Stripe** | ✅ Yes — for the **Online Store** | W+M supports Stripe, Square, or PayPal **or** GoDaddy Payments (GoDaddy Payments is mutually exclusive with Stripe/Square). So you can route store sales through *your* Stripe. |
| **Sell tickets/memberships as "products"** | ✅ Possible, but **not yet set up** | The W+M Online Store *can* sell a workshop/event seat as a product, but your store is **not currently configured** — the cart markup on the site is inert, and CFT Merch is an external **Printify** pop-up, not a GoDaddy store. Enabling this means setting up the store and connecting Stripe (Phase 0). |
| **Programmable database** | ❌ No | No user-accessible DB or tables. (GoDaddy's *Managed WordPress / cPanel* hosting has MySQL — but that's a different product, see §5 Depth 3.) |
| **Server-side / custom code** | ❌ No | No backend you control. |
| **Embed third-party JS widgets (PushPress/CompCorner)** | ⚠️ Limited & flaky | W+M has an "Add HTML / custom code" block, but GoDaddy's own docs include a "custom code not displaying properly" troubleshooting page — scripts are sandboxed and often don't render. Simple **iframes** and **links** are the reliable path. **[verify on the live plan/tier]** |
| **Heats, scoring, judging, live leaderboards, check-in, event email automation** | ❌ No | Not what a site builder does. This is the platform's job. |

**Translation:** On GoDaddy *today*, with your own Stripe, you can do **registration → payment** for a simple event (sell a seat). Everything past payment — email sequences, day-of check-in, workout release, leaderboards — needs PushPress or Competition Corner.

---

## 2. What the two platforms give you (verified)

### PushPress (gym-management; also runs your in-house events)
- **Embeddable into your website:** PushPress publishes embeddable **landing pages, public calendar, and "Plans & Events"** widgets, plus "connect your PushPress system directly to your website" — so signup, schedule, and event registration can live *on* crossfittaylors.com via embed code or links.
- **Payments:** runs on **Stripe** under the hood (PushPress Payments). Funds settle to your bank.
- **Covers:** memberships + recurring billing, class booking, **day-of check-in** (incl. a TV check-in app), a branded **member app** with **leaderboards + community feed** (your "workout announcements"), automated **email/SMS**, reporting.
- **Events:** native registration + waivers for free/paid events (workshops, Bring-a-Friend, Community WOD, clinics).
- **Integrations:** Open API + **webhooks + Zapier** → can push to **Mailchimp**, sheets, etc.
- **Pricing:** Core Free $0 · Pro ~$159/mo · Max ~$229/mo. Free migration from other systems.

### Competition Corner (purpose-built competitions)
- **Embeddable into your website:** CompCorner explicitly supports **embedding the registration forms *and* the leaderboard** on your own site (iframe/embed). **[confirm exact embed snippet in their help center]**
- **Payments:** **Stripe**-based payouts to your bank; platform fee ~**4% + $2.00 per ticket** (pay-as-you-sell, no monthly base).
- **Covers the competition-specific engine:** scheduled workout release, **athlete online score submission**, affiliate score validation, **heat/lane scheduling**, athlete **check-in**, **real-time leaderboards** (TV/LED-wall displays), athlete/spectator app + push notifications.

**Rule of thumb:** PushPress = everyday gym + your in-house events. Competition Corner = a real multi-heat throwdown. They are complementary, not either/or.

> **Note on SugarWOD (you already use it):** SugarWOD already posts daily WODs, logs scores, and shows class leaderboards. So your *daily* "workout announcements + leaderboards" goal is partly solved. When you adopt PushPress (which has its own feed/leaderboard), you'll decide to either keep SugarWOD alongside it or consolidate.

---

## 3. The three integration depths (pick per goal, you can climb over time)

### Depth 1 — Sell event tickets as GoDaddy Store products (your Stripe) · **available today**
- **How:** connect your Stripe to the W+M Online Store; create the event as a product with a seat limit.
- **Covers:** registration → payment. That's it.
- **Gaps:** email = manual (Mailchimp), no waiver-in-flow, no check-in, no heats/leaderboard.
- **Use it for:** the *next* workshop/event you run, this month, with zero new tooling. A good way to validate demand before investing.

### Depth 2 — Embed / redirect PushPress or CompCorner from the GoDaddy site · **recommended near-term** ⭐
- **How:** the platform runs the full flow; your site **embeds the widget** (iframe) or **links to a hosted page on your own subdomain** (e.g. `register.crossfittaylors.com`, `app.crossfittaylors.com`) via a GoDaddy DNS CNAME so the URL stays on your brand.
- **Covers:** the **entire lifecycle** — registration → payment (your Stripe) → email → check-in → announcements → scheduling → leaderboards → close.
- **Why this is the sweet spot:** it gives the "on my own website" feel without fighting GoDaddy's embed limits or rebuilding anything. CompCorner embeds reg form + leaderboard; PushPress embeds signup/calendar/events.
- **Caveat:** because W+M JS embeds are flaky, prefer **iframe embeds** or **branded-subdomain redirects** over pasted `<script>` widgets. **[verify which renders cleanly on the live plan]**

### Depth 3 — Rebuild the marketing site on an open platform + deep-integrate · **the true "own-it" end-state**
- **How:** move the marketing site to **WordPress** (this can stay "on GoDaddy" via **GoDaddy Managed WordPress / Managed Ecommerce** — which *does* give you MySQL, Stripe via WooCommerce, and plugins) **or Webflow**. Then embed PushPress/CompCorner cleanly, run your own Stripe, and — only if you have a bespoke need — add a real database.
- **Covers:** everything, seamlessly, with no embed ceiling, better SEO/performance, and full design control.
- **This is where your "GoDaddy Stripe + database" wish is actually granted** — not in the Website Builder, but in GoDaddy's WordPress/cPanel hosting tier.
- **Cost:** a redesign + migration project.

---

## 4. "Can't I just build it myself with Stripe + a database?"

Technically yes — Stripe Checkout + a database (MySQL/Postgres/Supabase) + a small web app can take a registration and a payment, and you'd own 100% of it. But the parts you listed that *matter* — heat/lane scheduling, judge scorecards, athlete score submission + validation, sub-second live leaderboards on a wall display, day-of check-in, and the event email/announcement automations — are a **multi-month custom software build plus permanent maintenance, support, and liability**. PushPress ($0–229/mo) and Competition Corner (4% + $2/ticket) already do all of it, on Stripe, today.

**Recommendation:** do **not** rebuild the engine. Reserve a custom Stripe + database build only for a genuinely bespoke flow the platforms can't express. For 99% of CrossFit Taylors' needs, Depth 2 (now) → Depth 3 (later) is faster, cheaper, and lower-risk.

---

## 5. Recommended architecture

```
                         ┌─────────────────────────────────────────┐
   Visitor / Member ───▶ │  crossfittaylors.com (the "front door")  │
                         │  Phase A: GoDaddy W+M                     │
                         │  Phase B: WordPress/Webflow (deep embed)  │
                         └───────────────┬───────────────────────────┘
                                         │  embed (iframe) / branded subdomain
                 ┌───────────────────────┼───────────────────────────┐
                 ▼                                                   ▼
   ┌─────────────────────────────┐                   ┌────────────────────────────┐
   │ PushPress (system of record)│                   │ Competition Corner          │
   │ members · billing · booking │                   │ (only for real competitions)│
   │ check-in · member app       │                   │ heats · scoring · live board │
   │ leaderboard · email/SMS     │                   │ athlete app · check-in       │
   │ in-house events + waivers   │                   │ embed reg form + leaderboard │
   └──────────────┬──────────────┘                   └──────────────┬──────────────┘
                  │ payments                                         │ payments
                  ▼                                                  ▼
         ┌──────────────────────────  ONE Stripe account  ──────────────────────────┐
         │  unified payouts to your bank · unified reporting · tax/receipts/refunds   │
         └────────────────────────────────────────────────────────────────────────────┘
                  │ webhooks / Zapier
                  ▼
         ┌──────────────────────┐
         │ Mailchimp (email)    │  ← announcements, nurture, post-event follow-up
         └──────────────────────┘
```

**Principles:**
- **One Stripe account** connected to whichever platform is selling, so payouts and reporting stay unified. (Avoid splitting revenue across GoDaddy Payments + Stripe — that fragments your books.)
- **PushPress is the system of record** for people, money, and everyday operations.
- **Competition Corner is added only when you run a true competition**; its pay-per-ticket model means it costs nothing until then.
- **The website embeds/links; it never tries to *be* the platform.**

---

## 6. Phased rollout

| Phase | Goal | Key steps | Effort | When |
|---|---|---|---|---|
| **0 — Prove it on GoDaddy** | First paid event online, no new tools | Connect *your Stripe* to the W+M Online Store; list the next event as a product w/ seat cap; collect waivers via SignNow; promote via Mailchimp; add Meta Pixel + GA4 purchase event to measure | Low | Now |
| **1 — Operations on PushPress** | Members, billing, booking, app | Stand up PushPress (Core Pro); connect Stripe; migrate members/billing; **embed** PushPress signup + public calendar on the site (iframe or `join.crossfittaylors.com`); turn on member app (leaderboards + announcements); wire PushPress→Mailchimp via Zapier; decide SugarWOD keep vs. consolidate | Medium | 1–2 mo |
| **2 — In-house events end-to-end** | Reg→pay→email→check-in | Run the next workshop / Bring-a-Friend / clinic as a **PushPress event**: registration + waiver + your-Stripe payment + automated email; day-of **check-in** via PushPress app | Low–Med | after Phase 1 |
| **3 — Competitions** | Heats + live leaderboard | For a real throwdown: set up **Competition Corner**, connect Stripe, build heats/workouts, **embed reg form + leaderboard** on a `/compete` page or `register.crossfittaylors.com`; run athlete check-in + wall-display leaderboard day-of | Med (per event) | when a comp is on the calendar |
| **4 — Deep integration / rebuild (optional)** | Seamless "own-site" end-state | Rebuild marketing site on **WordPress (incl. GoDaddy Managed WordPress) or Webflow**; deep-embed PushPress/CompCorner; your Stripe via WooCommerce if needed; add a real DB only for bespoke flows | High | when the embed ceiling actually limits you |

You capture **80% of the value by end of Phase 2** without leaving GoDaddy. Phase 4 is only worth it once GoDaddy's embed/SEO limits are demonstrably costing you.

---

## 7. URL / subdomain strategy (so it feels like *your* site)

Point GoDaddy DNS records at the platforms so hosted pages live on your brand:

| Subdomain | Points to | Purpose |
|---|---|---|
| `join.crossfittaylors.com` | PushPress | membership signup / lead capture |
| `app.crossfittaylors.com` or `members.…` | PushPress member portal | booking, account, member app web |
| `register.crossfittaylors.com` | Competition Corner | event/competition registration |
| `leaderboard.crossfittaylors.com` | Competition Corner | live leaderboard / wall display |

(Set these as **CNAME** records in GoDaddy DNS; both platforms support custom/branded domains — **[confirm each platform's custom-domain steps]**.) Where the GoDaddy page allows a clean **iframe**, embed instead of redirect so the visitor never visibly leaves.

---

## 8. Stripe specifics (your payment rail)

- **One Stripe account**, connected to PushPress (and to CompCorner for comps, and optionally to the GoDaddy store). Funds settle to your bank from each.
- **Fees stack:** Stripe's processing fee (~2.9% + 30¢) **plus** the platform's fee (PushPress: included in plan/transparent; CompCorner: 4% + $2/ticket). Decide whether to absorb or pass ticket fees to athletes.
- **Receipts, refunds, disputes, tax** are handled in the platform UI (which calls Stripe). Keep refunds/comp tickets inside the platform so reporting stays correct.
- **Do not run GoDaddy Payments and Stripe in parallel** for events — pick Stripe as the single rail so all event/membership revenue reconciles in one place.

---

## 9. Decisions needed to start

1. **Greenlight PushPress** as the operations/system-of-record platform? (Unblocks Phases 1–2.)
2. **Confirm your Stripe account** is the single payment rail (and switch the GoDaddy store off GoDaddy Payments → Stripe if needed).
3. **SugarWOD:** keep alongside PushPress, or consolidate into PushPress's app for leaderboards/announcements?
4. **Competitions:** is a true multi-heat event on the calendar this year (→ stand up CompCorner in Phase 3), or are in-house workshops enough for now (→ PushPress events suffice)?
5. **End-state site:** commit to a Phase-4 rebuild target (GoDaddy Managed WordPress vs. Webflow vs. stay on W+M with embeds), or defer the decision until after Phase 2?

---

## 10. Cost snapshot (rough, monthly unless noted)

| Item | Cost |
|---|---|
| GoDaddy Websites + Marketing (current) | existing plan |
| Stripe processing | ~2.9% + 30¢ per transaction |
| PushPress Core | $0 (Free) / ~$159 (Pro) / ~$229 (Max) |
| Competition Corner | ~4% + $2.00 per ticket (no base) |
| Mailchimp | existing plan |
| Phase-4 rebuild (optional, one-time) | project-based |

**Net:** you can begin **Phase 0 today at no new monthly cost** (just Stripe fees), and the only committed recurring add is PushPress when you start Phase 1.
