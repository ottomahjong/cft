# Website Redesign Plan + Design System + Claude Design Prompt — CrossFit Taylors

**Companion to:** `docs/site-audit.md`, `docs/event-platform-integration-plan.md`, `docs/custom-competition-platform.md`
**Shared tokens:** `docs/brand-design-tokens.css`
**Prepared:** June 2026

## Context

CrossFit Taylors needs a redesign of crossfittaylors.com whose **single goal is converting visitors into members**, while keeping the existing content largely intact. Prior audit work (see `docs/site-audit.md`) found the content is solid but the site under-converts: the primary CTAs dead-end at a contact form, the hero is a generic slogan, the nav is long and confusing, and the design is undifferentiated from every other CrossFit gym site.

Owner decisions that shape this plan:
- **Stay on GoDaddy Websites + Marketing (W+M)** for now (not rebuild on WordPress/Webflow).
- Pursue a **hybrid light/dark** visual direction.
- Build a **unified design system** shared by the marketing site *and* the React/Supabase competition app.
- Use a **distinctive brand look**: heritage/tactical palette (**blacks, army/olive greens, off-white, grays**), historically Futura-heavy, but open to fonts — wanting a **modern serif as the headline driver** that feels unique and does **not** look like a typical gym site.

This document is the redesign plan, the design system spec (reconciled to GoDaddy's real constraints), and a ready-to-paste **Claude Design prompt** to generate the visual design.

**Important platform reality (verified):** GoDaddy W+M lets you set **custom hex theme + per-section colors** (palette is fully doable) but has **no custom font upload** — you choose a primary/body font from GoDaddy's built-in library ("Theme → Fonts → Advanced"). So the ideal fonts below are *design intent*; the build must pick the nearest available faces in GoDaddy's picker. W+M is also **section/template-based** and **unreliable for custom JS embeds** — the design must live within stackable sections, and integrations use links/iframes/branded subdomains.

---

## 1. Goals & guardrails

- **Primary metric:** free-class / intro requests (lead conversion), then trials → members.
- **Keep** all existing content (programs, pricing, FAQ, partner benefits, owner story).
- **Design within GoDaddy W+M:** section-based layouts, custom hex colors, GoDaddy-library fonts, primary/secondary button styles, no custom fonts, minimal/iframe-only embeds.
- **One primary CTA everywhere:** "Claim your free class," with a secondary low-commitment "Free Saturday Community WOD."
- **Distinctive, not generic:** editorial serif + heritage palette so the brand stands out in the CrossFit field.

---

## 2. Design system (brand-aligned, GoDaddy-implementable)

### 2.1 Color palette (enter as custom hex in GoDaddy Theme → Color; set per-section)
| Token | Role | Suggested hex (confirm vs. logo) |
|---|---|---|
| `ink` | Near-black, primary text on light; dark-section background | `#15160F` (warm/green-black) |
| `forest` | Deepest green for dark sections/depth | `#23281A` |
| `olive` | **Primary brand accent** (buttons, links, highlights) | `#565E36` |
| `olive-bright` | Hover/active, small pops | `#6E7843` |
| `bone` | Off-white, primary light background | `#F2EEE3` |
| `paper` | Card/surface on light | `#FAF8F2` |
| `gray-warm` | Muted text, borders, captions | `#8C887B` |
| `gray-line` | Hairlines/dividers | `#D8D3C6` |

Notes: the palette intentionally **drops the old red**. Optional single warm accent (clay/rust `#B4552D`) reserved *only* for urgency moments (e.g., "spots filling") — flagged as an open question, not assumed. Dark sections use `ink`/`forest` bg + `bone` text + `olive` accents (this also bridges to the dark competition app).

### 2.2 Typography (design intent → GoDaddy fallback)
- **Display / headlines (the "driver"):** a **modern, characterful serif** that is uncommon for gyms.
  - Intent: **Fraunces** (soft, contemporary, editorial personality).
  - GoDaddy picks in priority order (use "Advanced" picker; choose first that exists): **Fraunces → DM Serif Display → Playfair Display**.
- **Body / UI:** a **geometric, Futura-style sans** for continuity with the brand.
  - Intent: **Jost** (excellent Futura substitute).
  - GoDaddy fallback order: **Jost → Montserrat → Poppins**.
- **Eyebrow / labels / badges:** keep **League Spartan** (the site's current font) all-caps, letter-spaced — ties old brand to new and adds a third typographic note. Fallback: the body sans, uppercased.
- **Type scale (rem):** display 3.0–3.75 / h1 2.5 / h2 1.875 / h3 1.375 / body 1.0–1.125 / small 0.8125. Headlines mixed-case serif (not all-caps) so the serif's character shows; labels all-caps.

### 2.3 Components / tokens (shared with competition app)
- **Buttons:** Primary = solid `olive` (or `ink`) with `bone` text; Secondary = outline `ink`/`olive`. Map to GoDaddy's primary/secondary button styles. Generous padding, ~6px radius, label sentence/uppercase per type.
- **Cards:** `paper` surface, `gray-line` hairline, soft shadow on light; on dark, `forest` surface with `olive` accents.
- **Pricing tier card:** name, price, "what's included," one CTA; a **"Most popular"** ribbon on the month-to-month Unlimited tier.
- **Spacing:** 8px base scale; section vertical rhythm 64–96px.
- **Imagery direction:** real, warm, candid gym photography — members of varied ages/fitness levels, coaches mid-cue, the Taylors Mill space; matte/filmic grade that flatters the olive/bone palette; avoid stock-y "shredded athlete" clichés. Where photos aren't available yet, use full-bleed `ink`/`forest` color blocks with serif headlines rather than weak stock.
- **Dark-section motif (hybrid):** reserve dark (`ink`/`forest`) for the hero band, the events/competition section, and the footer; keep conversion-critical pages (Pricing, Start Here, Schedule) mostly light (`bone`) for clarity and trust.

### 2.4 Unified with the competition app
The same tokens feed the React/Supabase app (`docs/custom-competition-platform.md`): app runs the **dark** variant (`ink` bg, `bone` text, `olive` accents, League Spartan labels, Jost UI), so site and app feel like one brand. The tokens are provided as importable CSS custom properties in `docs/brand-design-tokens.css`.

---

## 3. Information architecture (streamlined for the buyer's journey)

Collapse the 11-item nav to a focused primary nav; move member-servicing pages into a grouped "Members" menu / footer (from audit §9):

```
Primary nav:  Start Here · Classes · Pricing · About · FAQ · Events        [Free Class ▸]
Members menu: Member Benefits · Resources · Assistance · Merch (Printify)
Footer:       NAP, hours, social, waiver, legal, Members links
```

**Pages & key blocks:**
- **Home** — dark serif hero ("what/who/next" + Free Class CTA) · beginner reassurance ("New to CrossFit? You're the person we built this for") · programs grid (Group, Foundations, 55+, Kids, Bilingual, Open Gym, PT) · social proof (named testimonials) · schedule snapshot · events strip (dark) · location/Taylors Mill · sticky mobile "Free Class" bar.
- **Start Here** (NEW) — the prospect's front door: how it works (Free class → Foundations → Membership), what to expect, free Saturday Community WOD, FAQ highlights, one CTA.
- **Pricing** — keep transparent pricing; add **"Most popular"** + a guided path header + a real "Start free class / enroll" action (not just contact).
- **Classes/Schedule**, **About** (Cameron's story), **FAQ**, **Events** (dark; links to the competition app/registration), **Members** pages, **Contact** (kept, demoted from primary path).

---

## 4. Conversion plumbing to fix during the rebuild (GoDaddy-doable)
- Replace "BOOK NOW → contact form" with a **real lead-capture** (GoDaddy Appointments add-on or an embedded form that auto-replies + tags in Mailchimp).
- Add **Meta Pixel** and define **GA4 conversion events** (free-class request, pricing view, etc.) — currently missing.
- Pricing: add "Most popular" + guided path + actionable CTA.
- Fix the empty "More" nav link and the duplicate H1; tighten LocalBusiness schema.

---

## 5. Build sequence (within GoDaddy W+M)
1. **Theme setup:** enter palette hex; set primary/secondary fonts from the picker (per §2.2 priority); configure primary/secondary button styles; set light baseline + dark section style.
2. **Run the Claude Design prompt (§7)** to produce the design system + high-fidelity mockups of Home, Start Here, and Pricing; use them as the build target.
3. **Build pages** section-by-section to match mockups; restructure nav/IA.
4. **Conversion plumbing** (§4): lead capture, Meta Pixel, GA4 events.
5. **QA:** mobile, contrast/accessibility, NAP consistency, links.

---

## 6. Verification / how we'll know it worked
- **Pre/post GA4:** track free-class requests, pricing-page views, and CTA clicks as events; compare 4-week windows.
- **Heuristic:** 5-second test on the new hero ("what is this / is it for me / what do I do?").
- **Mobile Lighthouse** before/after; **contrast** check on olive/bone/ink combos.
- **Functional:** every page has one working primary CTA that captures a lead (no dead-ends); Meta Pixel fires; events record.

---

## 7. The Claude Design prompt (copy–paste into Claude)

> Paste the block below into Claude (a fresh conversation, "design"/artifact use). It is self-contained. Provide your logo file and any real photos alongside it if you have them.

```
You are a senior brand & web designer. Design a distinctive, conversion-focused
website design system and high-fidelity page mockups for a CrossFit gym. Output
as interactive HTML/CSS artifacts (one design-system page + three responsive page
mockups). The site will be rebuilt on GoDaddy Websites + Marketing, so the design
must use only: custom hex colors (allowed), GoDaddy-library web fonts (no custom
font upload), stackable sections, and primary/secondary button styles. No reliance
on custom JavaScript widgets.

BUSINESS
- CrossFit Taylors — a welcoming, community-first CrossFit gym at Taylors Mill,
  255 Mill Street Unit C, Taylors, SC 29687. Phone 864-907-5772.
  Serves Greenville / Taylors / Greer, SC. Owner: Cameron.
- Audience: local adults, MANY nervous beginners and non-"gym people"; also
  families (Kids 4–12), 55+, and Spanish-speaking members. The brand is warm,
  confident, and inclusive — NOT bro-y or intimidating.
- Tagline to retain as a secondary motif (not the headline): "Effort is a Choice."

GOAL
- Convert visitors into members. One primary CTA repeated everywhere:
  "Claim your free class." Secondary low-commitment CTA: "Free Saturday
  Community WOD." Reassure beginners up front.

BRAND LOOK (make it stand out from typical CrossFit sites)
- Heritage / understated-tactical palette — NO bright red. Use:
  ink/near-black #15160F, deep forest #23281A, primary olive accent #565E36,
  olive-bright #6E7843, off-white "bone" #F2EEE3, card "paper" #FAF8F2,
  warm gray #8C887B, hairline #D8D3C6. (Optional sparing clay accent #B4552D
  only for urgency.)
- TYPOGRAPHY — this is the differentiator. Use a MODERN, characterful SERIF as
  the headline driver (intent: Fraunces; acceptable: DM Serif Display or Playfair
  Display) set mixed-case so its personality shows. Body/UI in a geometric
  Futura-style sans (intent: Jost; acceptable: Montserrat or Poppins). Use
  League Spartan, all-caps + letter-spaced, for small eyebrow labels/badges.
  Editorial serif headlines over a utilitarian geometric sans = the unique,
  non-gym feel we want.
- HYBRID light/dark: most pages light (bone background, ink text, olive accents);
  reserve dark (ink/forest background, bone text, olive accents) for the hero
  band, the events/competition section, and the footer.
- Imagery: warm, candid, real-gym photography (varied ages/levels, coaches
  cueing, the Taylors Mill space), matte/filmic grade. Where no photo, use
  full-bleed ink/forest color blocks with big serif headlines — never weak stock.

DELIVERABLES
1) A DESIGN SYSTEM page: color tokens (with hex + usage), the type scale
   (display/h1/h2/h3/body/label with the fonts above), button styles (primary
   solid olive, secondary outline), card styles (light + dark variants), a
   pricing-tier card with a "Most popular" ribbon, form fields, badges/eyebrows,
   spacing scale (8px base), and notes mapping each choice to GoDaddy's
   capabilities (which font to pick if the ideal isn't in GoDaddy's list).
2) HOME mockup: dark serif hero (clear "what is this / who it's for / what to do
   next" + Free Class CTA), beginner-reassurance block, programs grid (Group,
   Foundations on-ramp, 55+, Kids, Bilingual, Open Gym, Personal Training),
   named-testimonial social proof, schedule snapshot, dark events strip,
   location/community block, footer; plus a sticky mobile "Free Class" bar.
3) START HERE mockup: how it works (Free class → Foundations → Membership),
   what to expect on day one, the free Saturday Community WOD, top FAQs, one CTA.
4) PRICING mockup: transparent tiers — First class FREE; Foundations $60;
   Unlimited month-to-month $175/mo (mark "Most popular"); 6-month $160/mo;
   Annual $150/mo; Bilingual $120/mo; Ageless 55+ $80/mo; 8-class punchcard $120;
   Drop-in $20/class or $50/week; Personal Training "contact." Add a guided
   "New here? Free class → Foundations → membership" header and a clear CTA on
   each tier. Note the 10% discount (first responders/military/teachers/students/
   healthcare/family) and the hardship grant.

CONSTRAINTS
- Fully responsive (design mobile-first; show mobile + desktop).
- Accessible color contrast on all olive/bone/ink combinations.
- Keep it buildable as GoDaddy stackable sections (no bespoke JS).
- The design tokens should also suit a DARK companion web app (a competition
  leaderboard/registration tool) so the two share one identity.

Ask me for the logo and any photos before finalizing; if I don't provide them,
proceed with tasteful placeholders and color-block heroes.
```

---

## 8. Open items needed from the owner
1. **Logo files** (vector/PNG) and the **exact brand hex** for black & army green (to confirm/replace the suggested hexes).
2. **Photography**: any real gym photos to use, or should the mockups use color-block heroes/placeholders?
3. **Confirm fonts available** in your GoDaddy plan's "Advanced" font picker (so we lock the closest match to Fraunces + Jost).
4. **Optional accent**: OK to drop red entirely and (optionally) use a sparing clay/rust for urgency, or keep CTAs strictly olive/ink?
5. **Lead capture**: enable GoDaddy **Appointments** for real free-class booking, or embed a form that auto-replies + tags in Mailchimp?

## 9. Next steps
1. Run the §7 prompt in Claude to generate the design system + Home/Start-Here/Pricing mockups.
2. Configure the GoDaddy theme (palette hex + nearest fonts + button styles) and build pages section-by-section against the mockups.
3. Wire the conversion plumbing (§4) and verify with GA4 events (§6).
The shared tokens for the competition app live in `docs/brand-design-tokens.css`.
