# CrossFit Taylors — Full Site Audit & Redesign Brief

**Site:** https://crossfittaylors.com/
**Prepared:** June 2026
**Goal:** Prepare the site for a redesign that keeps the content largely the same but materially improves conversion of visitors → members, and lay out a path toward owner-hosted event registration (registration → payment → email → day-of check-in → workout announcements → scheduling → leaderboards → close).

---

## 1. Methodology & scope

This audit is based on two evidence sources:

1. **The homepage export** (saved HTML, CSS, JS, and asset references) — the real markup, navigation, calls-to-action, scripts, tracking, and integrations.
2. **The actual inner-page exports** you provided in a second pass: **Pricing, Schedule, FAQ, Member Benefits, and Membership Resources**. These replaced my earlier search-index reconstruction with real page code and corrected a key finding (pricing — see §4.2).
3. **The public search index + business listings** (CrossFit affiliate directory, Yelp, Facebook, Instagram, Wellhub) for the few pages still not exported (About, Contact, Events, Membership Assistance).

**What I verified directly (from code):** platform/hosting, navigation, homepage + 5 inner pages' copy and CTAs, the full pricing table, analytics/tracking, third-party integrations, structured data, and image/heading structure.

**What I still inferred (from the index, not code):** the exact layout of About, Contact, Events, and Membership Assistance. Flagged as **[verify]** where relevant.

> A logical next step is a hands-on pass with login access to the GoDaddy editor plus live Lighthouse/PageSpeed, GA4, and Search Console data, to quantify current conversion rates. This document is structured so it stays useful either way.

---

## 2. Platform & hosting reality check

You weren't sure what GoDaddy can do transactionally. The export answers this clearly.

**The site is built on GoDaddy Website Builder (the "Websites + Marketing" product).** Evidence in the code:

- Asset paths under `ceph-p3-01/website-builder-data-prod/...`
- Images served through GoDaddy's `isteam` image CDN and `blobby` object store
- A "Made with GoDaddy" / `godaddy.com/websites/website-builder` link in the footer

**What this means for transactions and plugins — the important part:**

| Capability | Status on the current site | Notes |
|---|---|---|
| **Online Store / cart** | **Already active** | The **Events** and **CFT Merch** pages use GoDaddy's commerce cart (`olsPage=cart` in the markup). So you *already* have the ability to take payments for tickets and merch. |
| **Email marketing** | **Active** | A **Mailchimp** signup is linked from the homepage. (GoDaddy also has its own email marketing; you're currently using Mailchimp.) |
| **Waivers** | **Active** | Waivers run through **SignNow** (external link). |
| **Workout tracking / leaderboards** | **Active (SugarWOD)** | The Pricing page lists "Access To Workout & Progress Tracking via **SugarWOD**" as a member benefit. SugarWOD already gives members daily WOD posting, score logging, PRs, and class leaderboards — so part of your "workout announcements + leaderboards" goal already exists. Important for the platform decision in §8. |
| **Appointment / class booking** | **Not used** | GoDaddy offers an "Appointments" booking add-on, but it is **not** wired up. The primary "BOOK NOW" button just goes to the contact form. |
| **Analytics** | **GA4 installed** (`G-M5DZNS31LJ`) | No Meta/Facebook pixel detected (`fbq` is absent) → **no retargeting** is possible today. |
| **Custom plugins / code** | **Very limited** | GoDaddy Website Builder is a closed, template-based builder. You cannot install arbitrary plugins (unlike WordPress) or run a custom server-side registration/leaderboard app on it. Custom HTML embeds are restricted. |

**Bottom line:** GoDaddy can handle *simple* transactions (sell a ticket, sell a shirt, collect an email). It **cannot** host the full competition platform you described (heats, athlete score submission, live leaderboards, day-of check-in, automated event email flows). That is a separate class of software — see Section 8.

---

## 3. Site inventory (page map)

From the navigation in the export plus the search index, the current sitemap is:

```
/                       Home
/about-us               About Us (owner story – "Cameron", coaches, facility)
/schedule               Group Class Schedule
/pricing                Membership Pricing (no public prices – "contact us")
/faq                    FAQ
/events                 Events (uses GoDaddy store cart)
/member-benefits        Member Benefits (referral $60/$60, Victory Grips code, etc.)
/membership-resources-1 Membership Resources
/membership-assistance  Membership Assistance (My Compassion Connection grants, hold/cancel)
/cft-merch              CFT Merch (store)
/contact-us             Contact Us (the de-facto conversion endpoint)
/privacy-policy         Privacy Policy
/terms-and-conditions   Terms & Conditions
```

Plus a **"More"** nav item that currently points to `#` (an empty/placeholder link — see findings).

**Programs offered (from Schedule + Pricing pages)** — a strong, broad roster that the nav doesn't fully surface: group CrossFit (all levels, M–Sat), **Foundations/Fundamentals** on-ramp (4×60-min), **Open Gym** (Thursdays), **Spanish/bilingual classes** (M/W/F 6:30pm), **Ageless Athletes 55+** (T/Th 7:30am), **CrossFit Kids** (ages 4–7 and 8–12), **personal training** (Troli Train), drop-ins, and the free **Saturday Community WOD**.

**Observations on IA:**
- The nav is **long and flat (11 top-level items)**. "Member Benefits," "Membership Resources," and "Membership Assistance" are three near-synonymous items that dilute attention and confuse non-members.
- The slug `membership-resources-1` carries GoDaddy's auto-dedupe `-1` suffix, suggesting a page was duplicated/re-created at some point. Minor, but worth cleaning up for SEO/tidiness.
- There is **no single "Get Started" / "Start Here" destination** — the most important page a prospective member needs.

---

## 4. Conversion findings (prioritized)

Severity: 🔴 high impact on conversion · 🟡 medium · 🟢 polish

### 🔴 4.1 The primary conversion action doesn't actually convert
The hero and header CTAs ("**BOOK NOW**", "**MORE INFO**", "**GET IN TOUCH**") all route to **`/contact-us`** — a generic contact form. There is **no real booking step**. A motivated visitor who wants to claim the "FIRST CLASS FREE" offer has to fill out a form and then *wait for a human to reply*. Every hour of delay bleeds intent.

> **Recommendation:** Make the #1 CTA a real, self-serve "Claim your free class" flow that captures name/email/phone *and* lets them pick a class time (or at least immediately confirms next steps + adds them to a nurture email). On GoDaddy this can be the **Appointments** add-on; long-term it's the gym-management platform's booking widget (Section 8).

### 🟡 4.2 Pricing is fully transparent (good!) — but it's a long, unguided list with no way to act
**Correction to my first-pass audit:** the Pricing page **does** list complete, transparent pricing. This is a genuine strength — keep it. For the record, the current table is:

| Option | Price |
|---|---|
| First class (Greenville County residents) | Free |
| Foundations (on-ramp) | $60 |
| Unlimited — month-to-month | $175/mo |
| Unlimited — 6-month (billed monthly) | $160/mo |
| Unlimited — annual (billed monthly) | $150/mo |
| Spanish/Bilingual classes (M/W/F 6:30pm) | $120/mo |
| Ageless Athletes 55+ (T/Th 7:30am) | $80/mo |
| 8-Class Punchcard (no expiration) | $120 |
| Drop-in — one class | $20 |
| Drop-in — one week | $50 |
| Personal training (Troli Train) | "Contact for scheduling" |

Plus a 10% discount (first responders, military, teachers, full-time students, healthcare, family in-household) and the My Compassion Connection grant. So the problem is **not** transparency — it's two things:

1. **No way to act on the page.** There are no "Join" / "Sign up" / "Buy" buttons on any tier — every path still routes to *contact us* or *sign a waiver*. A ready-to-buy visitor has nowhere to convert. (This is the same dead-end as §4.1.)
2. **Choice overload, no guidance.** Eleven options with no "Most popular" flag, no recommended starting path, and no framing of *which plan fits whom*. A newcomer can't tell that the intended journey is likely *free class → Foundations → Unlimited*.

> **Recommendation:** Keep the transparent prices. Add (a) a real online enroll/checkout or "Start free class" button on the primary tiers, (b) a "Most popular" badge on the month-to-month Unlimited (or annual) plan, and (c) a one-line "New here? Start with a free class → Foundations → membership" path at the top so the list reads as a journey, not a menu.

### 🔴 4.3 Weak, generic value proposition above the fold
The hero is an animated banner repeating **"Effort is a Choice - CrossFit Taylors"** four times, with the supporting line *"CrossFit Taylors is a gym located in the Greenville, SC area. Our focus is to use a positive group class environment to help you make health and fitness a priority."*

This is brand-y but doesn't answer the visitor's three questions in 5 seconds: *What is this? Is it for someone like me? What do I do next?* "Effort is a Choice" is a slogan, not a value proposition.

> **Recommendation:** Lead with a benefit-driven headline + specific, low-friction offer + one primary button. Example structure (keep your voice): **"Coached group fitness in Taylors that actually fits your life."** → sub: *"Beginner-friendly CrossFit classes 6 days a week. Your first class is free."* → **[Claim my free class]**. Keep "Effort is a Choice" as a secondary brand motif, not the headline.

### 🟡 4.4 Three overlapping "membership" nav items
"Member Benefits," "Membership Resources," and "Membership Assistance" compete for the same mental slot and mostly serve *existing* members (referral codes, partner discounts, hold/cancel, hardship grants). They clutter the nav a prospect uses to evaluate the gym.

> **Recommendation:** Consolidate into a single **"Members"** area (or a footer "Current Members" group). Reserve the primary nav for the buyer's journey: **Start Here · Classes/Schedule · Pricing · About · FAQ · Events**.

### 🟡 4.5 Social proof is present but underleveraged
There's a **"Reviews for CrossFit Taylors near Greenville, SC"** section on the homepage — good. But there are **no testimonials with names/faces/results**, no transformation stories, and the reviews aren't surfaced near the CTAs where decisions happen.

> **Recommendation:** Add 2–3 named member stories ("I'd never lifted a barbell…") near the hero and the pricing CTA. Pull your best Google/Facebook reviews. Faces + first names + a specific result outperform a generic review carousel.

### 🟡 4.6 "Free first class" and "free Saturday Community WOD" offers are buried/scattered
The homepage shows **"FIRST CLASS FREE!!! COME TRY US OUT!!!"** and the Saturday **Community WOD** (free for members and non-members). These are your strongest acquisition hooks, but they're styled as shout-y all-caps banners rather than a clear, repeated, clickable offer. The all-caps with triple exclamation points reads as low-trust.

> **Recommendation:** Make "First class free" a calm, confident, repeated CTA (hero, mid-page, footer, and a sticky mobile bar). Give the free Saturday Community WOD its own small "no commitment way to try us" card — it's a fantastic top-of-funnel offer for nervous beginners.

### 🟡 4.7 No clear path for the beginner who's intimidated
CrossFit's biggest conversion barrier is "I'm not fit enough / it looks scary." The FAQ addresses some of this, but the homepage doesn't *reassure* up front. The "NOW OFFERING SPANISH SPEAKING CLASSES" and "55+" programs are real differentiators that are easy to miss.

> **Recommendation:** Add a short "Is this for me?" / "New to CrossFit?" reassurance block (scaling for all fitness levels, Foundations on-ramp, supportive community, Spanish-speaking classes, 55+ program). These inclusivity angles are genuine competitive advantages — feature them.

### 🟢 4.8 "More" nav points to nothing
The **"More"** menu item links to `#` (placeholder). Either populate it or remove it.

### 🟢 4.9 Duplicate H1 / heading hygiene
The page renders the **same H1 ("Effort is a Choice - CrossFit Taylors") twice**, and the event banner repeats the text 4×. Multiple identical H1s hurt SEO and accessibility (screen-reader users get a confusing outline).

---

## 5. Technical / SEO / accessibility findings

### SEO
- ✅ Title and meta description are present and reasonable on the homepage.
- ✅ A `LocalBusiness` JSON-LD block exists — good instinct. **But it's incomplete/malformed:** the whole address is jammed into `streetAddress` (no `addressLocality`, `addressRegion`, `postalCode`), and it's **missing** `openingHours`, `priceRange`, `sameAs` (Facebook/Instagram), and `aggregateRating`. Fixing this improves the Google Business knowledge panel and local pack presence. **[verify across other pages]**
- ⚠️ **No `canonical` tag** on the homepage.
- ⚠️ Most imagery on GoDaddy is rendered as **CSS background images**, not `<img>` tags (only 5 `<img>` tags in the whole homepage). Background images are invisible to search engines and screen readers and can't carry alt text. This caps your image-SEO and accessibility ceiling — a known GoDaddy limitation.
- 🟡 Inner-page titles follow a decent pattern ("Membership Pricing | Upstate South Carolina Gym"), but should consistently include the **city + "CrossFit"** for local intent ("CrossFit in Taylors / Greer / Greenville SC").

### Performance
- The page is a ~195 KB minified HTML document with multiple GoDaddy widget JS bundles, a cookie-consent widget, GA4, and the reviews widget. GoDaddy sites are typically **mid-pack on Core Web Vitals** (heavy framework JS, lots of background-image loading). **[verify with live PageSpeed Insights — mobile score is the one that matters for local conversion]**

### Tracking & measurement
- ✅ **GA4** is installed (`G-M5DZNS31LJ`).
- 🔴 **No Meta/Facebook Pixel** (no `fbq`). You cannot retarget website visitors or build lookalike audiences on Instagram/Facebook — a big miss for a local gym that's active on social.
- 🔴 **No conversion events defined** that I can see — form submits, "book free class," and store purchases should fire GA4 events so you can actually measure visitor→lead→member. Right now you're flying blind on conversion rate.

### Accessibility (quick heuristics)
- All-caps shouting copy ("FIRST CLASS FREE!!!") is harder for screen readers and reads as low-trust.
- Background-image-only visuals = no alt text for non-decorative images.
- **[verify]** color contrast, focus states, and form labels in a live pass.

### Legal / trust
- ✅ Privacy Policy + Terms present; ✅ cookie-consent banner present.
- ✅ NAP (name/address/phone) consistent in footer: *255 Mill Street, Unit C, Taylors, SC 29687 · 864-907-5772*. (Note: one external listing shows "250 Mill St." — make sure NAP is identical everywhere for local SEO. **[verify]**)

---

## 6. What's working (don't break these in the redesign)

- **Transparent, well-structured pricing** with options for every budget (drop-ins, punchcard, 55+, bilingual, multiple commitment terms) — a real trust asset; keep it.
- A genuinely strong, broad **program & offer set**: first class free, free Saturday Community WOD, Foundations on-ramp, CrossFit Kids, Spanish-speaking and 55+ classes, month-to-month (no contracts), referral program ($60/$60), and a rich **partner-benefits page** (Onward PT, Victory Grips, 2POOD, TYR, Revival Cleaning, member shower) plus hardship/grant assistance.
- A real **community story** (owner Cameron, Taylors Mill location, events like Murph, Bring-a-Friend Week, member hangouts) and a solid, reassuring **FAQ** (glossary, "you don't need to be in shape," scaling, safety).
- **SugarWOD already in use** for workout posting, score logging, and leaderboards — a head start on the "workout announcements + leaderboards" goal.
- GA4 in place; commerce cart enabled; Mailchimp collecting emails; waivers digital (SignNow). Decent baseline local SEO footprint.

The redesign is mostly about **reorganizing and sharpening** existing content around a clear conversion path — not rewriting the gym's story.

---

## 7. Prioritized roadmap

### Phase 0 — Quick wins on the current GoDaddy site (days, no migration)
1. Rewrite the hero: benefit headline + "first class free" + one primary button.
2. Turn "BOOK NOW" into a real lead capture (GoDaddy **Appointments** add-on, or at minimum an embedded form that auto-replies and tags the lead in Mailchimp).
3. Keep the transparent pricing; add a "Most popular" badge, a guided "free class → Foundations → membership" path, and a real enroll/checkout (or "Start free class") button on the primary tiers.
4. Add a sticky mobile "Claim free class" bar.
5. Add the **Meta Pixel** and define **GA4 conversion events** (lead form, free-class request, store purchase).
6. Fix the LocalBusiness schema (full address, hours, `sameAs`, `priceRange`).
7. Fix the empty **"More"** link and de-duplicate the H1.
8. Add 2–3 named testimonials near the CTAs.

These alone should move the needle before any platform change.

### Phase 1 — Redesign (information architecture + messaging)
- New sitemap and a dedicated **"Start Here"** page (Section 9 below).
- Consolidate the three "membership" pages into a single member area.
- Beginner reassurance block + inclusivity differentiators surfaced.
- Decide platform direction (Section 8): stay on GoDaddy for the marketing site, or move the marketing site too.

### Phase 2 — Member-management + event platform (the big goal)
- Stand up a gym-management platform (recommendation: **PushPress**) for memberships, billing, class booking, the member app, and **event registration**.
- Decide whether competitions run on PushPress events or a dedicated competition tool (**Competition Corner**).

---

## 8. Event registration & member-management strategy (the headline ask)

You want to eventually run, **on your own site**, the full lifecycle:
**registration → payment → email/announcements → day-of check-in → workout announcements → event scheduling → leaderboards → close.**

Reality: **GoDaddy Website Builder cannot do this.** It's a marketing-site builder with a light store. The capabilities you listed are exactly what dedicated gym-management and competition platforms exist to provide. The two examples you named sit in two different categories:

### The two categories
**A) Gym-management platform (run the whole business + light events)** — e.g. **PushPress**, Wodify, Zen Planner, Glofox.
- Memberships, recurring billing, class scheduling/booking, **check-in (incl. a TV check-in app)**, a branded **member app** with **leaderboards** and a community feed for **workout/PR announcements**, automated **email/SMS**, reporting.
- Handles **event registration + waivers + paid/free events** natively.
- **PushPress pricing (public):** Core Free $0/mo, Core Pro ~$159/mo, Core Max ~$229/mo; payments via Stripe. Free migration from Mindbody/Zen Planner/Wodify/etc.
- **Best for:** running the gym day-to-day *and* hosting your in-house events, smaller competitions, "bring a friend" sign-ups, workshops (you already list a Squat Workshop, Murph, member hangouts — perfect fit).

**B) Dedicated competition platform (serious, multi-heat competitions)** — e.g. **Competition Corner**.
- Purpose-built for functional-fitness *competitions*: registration, **scheduled workout release**, **athlete online score submission**, affiliate scoring validation, **heat/lane scheduling**, athlete check-in, **real-time leaderboards** (TV/LED-wall displays), and a spectator/athlete app.
- **Pricing model:** per-ticket platform fee (~4% + $2.00/ticket) — you pay as you sell, no monthly base.
- **Best for:** a real throwdown/competition with heats, lanes, judges, and a live leaderboard wall — the parts a gym-management tool only does lightly.

### Where SugarWOD fits
You **already run SugarWOD** for daily workout posting, score logging, PRs, and class leaderboards — so the "workout announcements + leaderboards" piece of your goal is partly solved for *class* programming today. Two implications:
- For **everyday class life**, SugarWOD already covers workout announcements and leaderboards; you mainly need to connect it to booking/billing and surface it on the site.
- For **events/competitions**, SugarWOD is not a registration/heat/scoring engine — that's the gap PushPress (light) or Competition Corner (full) fills. Note PushPress has its own member-app feed/leaderboard, so when you adopt a management platform you'll decide whether to keep SugarWOD alongside it or consolidate.

### Recommended architecture
For CrossFit Taylors' stated goals, the cleanest path is:

1. **Adopt PushPress as the system of record** for members, billing, class booking, check-in, the member app (leaderboards + announcements), and **most events** (workshops, Community WOD signups, Bring-a-Friend, paid clinics). This single move also fixes Findings 4.1 and 4.2 (real booking + a place to publish pricing) and gives you the email/SMS automation you want.
2. **Embed PushPress's booking/registration widgets into the website** so registration *feels* like it lives on crossfittaylors.com. You don't need to abandon the marketing site to do this.
3. **Add Competition Corner only when/if you run a true multi-heat competition.** Don't pay for competition infrastructure you don't yet need; its per-ticket model means it costs nothing until you sell tickets.

### The website question this raises
GoDaddy Website Builder is **restrictive about custom embeds**, which makes deeply integrating a PushPress booking flow awkward. So a key redesign decision is:

- **Option 1 — Keep GoDaddy as the marketing site, link/embed PushPress where allowed.** Lowest effort, lowest cost, but the integration is shallow and you're capped by GoDaddy's CWV/SEO/embed limits.
- **Option 2 — Rebuild the marketing site on a platform with full embed/code control (WordPress, Webflow, or Squarespace) and integrate PushPress cleanly.** More effort, but removes the ceiling on SEO, performance, embeds, and design, and makes "run registration on my own site" genuinely seamless.

> **My recommendation:** Plan for **Option 2 as the redesign target**, because your end-state goals (own-site registration, leaderboards, event flows) repeatedly bump into GoDaddy's walls. But sequence it: do the Phase-0 quick wins on GoDaddy now (cheap, fast lift), adopt PushPress for operations, and rebuild the marketing site on a more open platform as the actual "redesign." That order captures most of the conversion gains early while you stage the bigger move.

---

## 9. Proposed redesign sitemap (content stays, structure sharpens)

```
/                  Home — strong hero + free-class CTA, beginner reassurance,
                   offers, social proof, schedule snapshot, events, location
/start-here        NEW — the prospect's front door: how it works, Foundations
                   on-ramp, first-class-free + Saturday Community WOD, what to
                   expect, FAQ highlights, → book free class
/schedule          Classes & Schedule (live booking widget)
/pricing           Pricing — show prices/plans + 55+ + personal training + intro offer
/about             About Us — Cameron's story, coaches, facility, community
/events            Events — registration via the gym/competition platform
/faq               FAQ
/store             CFT Merch
─ Members ─ (grouped, secondary nav/footer)
   /members/benefits      (referral, partner discounts)
   /members/resources
   /members/assistance    (hold/cancel, hardship grants)
/contact           Contact (kept, but no longer the primary conversion path)
```

Every prospect-facing page gets **one clear primary CTA** ("Claim your free class") repeated consistently, plus the secondary low-commitment option ("Try the free Saturday Community WOD").

---

## 10. Decisions needed from the owner

1. **Pricing transparency:** publish prices, publish a "starting at," or intentionally gate behind a free-intro booking? (Affects Pricing page + hero CTA.)
2. **Platform direction for the marketing site:** stay on GoDaddy (cheapest, capped) vs. rebuild on WordPress/Webflow/Squarespace (more open, better end-state). See Section 8.
3. **Operations platform:** green-light PushPress (or a competitor) as the member/billing/booking/events system of record?
4. **Competitions:** is a true multi-heat competition on the near-term roadmap (→ Competition Corner), or just workshops/in-house events (→ PushPress events are enough for now)?
5. **Branding:** keep "Effort is a Choice" — and if so, as the headline or as a secondary motif?

---

## 11. Evidence appendix (from the homepage export)

- **Platform:** GoDaddy Website Builder — asset paths `website-builder-data-prod`, `isteam` image CDN, `blobby` store, footer `godaddy.com/websites/website-builder` link.
- **Commerce active:** `/events` and `/cft-merch` use GoDaddy store cart (`data-page-query="olsPage=cart"`).
- **Analytics:** GA4 `G-M5DZNS31LJ` via `googletagmanager.com/gtag/js`. No `fbq` (no Meta pixel).
- **Integrations:** Mailchimp (`mailchi.mp/...`) for email; SignNow (`signnow.com/s/...`) for waivers; **SugarWOD** for workout/score tracking & leaderboards (named on Pricing page).
- **Pricing page (verified from export):** Foundations $60; Unlimited month-to-month $175/mo, 6-mo $160/mo, annual $150/mo; Spanish/Bilingual $120/mo; Ageless 55+ $80/mo; 8-class punchcard $120; drop-in $20/class, $50/week; personal training "contact"; 10% discount; grant program. **No per-tier "join/buy" buttons** — all links go to Schedule, Membership Assistance, or SignNow.
- **Schedule page (verified):** full class times M–Sat; Open Gym Thursday (5:30–6:30am, 9–10am, 12–1pm, 4:30–7pm); Spanish M/W/F 6:30pm; Ageless 55+ T/Th 7:30am; **CrossFit Kids (4–7, 8–12)**; Fundamentals 4×60-min "contact for scheduling."
- **FAQ page (verified):** What is CrossFit, what to bring, "first class free for Greenville County residents," safety/scaling, "do I need to be in shape? No," class structure, CrossFit glossary (AMRAP/EMOM/Metcon/Tabata), external resources.
- **Member Benefits (verified):** referral $60/$60; partner discounts — Onward PT (10% eval w/ Dr. Ryan Savage), Victory Grips (code CFTAYLORS), 2POOD (CROSSFITTAYLORS10), TYR (SE20971WA), Revival Cleaning (15%); Troli Train personal training + podcast; member shower.
- **Membership Resources (verified):** grant application, membership hold (1–12 weeks/yr) and cancellation request forms (CFT replies in 2–3 business days).
- **Nav (11 items):** Home, About Us, Schedule, Pricing, FAQ, Events, Member Benefits, Membership Resources, Membership Assistance, CFT Merch, More (`#`).
- **CTAs:** "BOOK NOW" → `/contact-us`; "MORE INFO" → `/contact-us#…`; "GET IN TOUCH" → `/contact-us`; "SIGN WAIVER HERE" → SignNow.
- **Hero/H1:** "Effort is a Choice - CrossFit Taylors" (rendered twice).
- **Key copy:** "FIRST CLASS FREE!!! COME TRY US OUT!!!"; "Getting Started Is The Hardest Part"; "NOW OFFERING SPANISH SPEAKING CLASSES!!"; Community WOD (free, Saturdays 10–11am); events (Murph, Bring-a-Friend Week, Member Hangout @ Unity Park, Squat Workshop w/ Ryan of Onward).
- **Structured data:** `LocalBusiness` JSON-LD present but incomplete (address not broken into locality/region/postal; no hours, `sameAs`, `priceRange`, or rating).
- **NAP:** 255 Mill Street, Unit C, Taylors, SC 29687 · 864-907-5772. Social: facebook.com/crossfittaylors, instagram.com/crossfit_taylors.
- **Images:** only 5 `<img>` tags; most visuals are CSS background images (no alt text, not crawlable).
