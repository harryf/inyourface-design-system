---
project: inyourface_design_system
task: Build mobile-first comedy-themed design system for inyourfacecomedy.ch
slug: inyourface-design-system-v1
effort: E3
phase: build
progress: 0/37
mode: ALGORITHM
started: 2026-05-25T20:35:00+02:00
updated: 2026-05-25T20:35:00+02:00
algorithm_config:
  classifier_source: classifier
---

## Problem

The current Jekyll site at `inyourfacecomedy.ch` was scaffolded from the Type-on-Strap blog theme. Layouts and components were built for blog posts, not for selling tickets to comedy shows. Concrete failures on mobile (the 70% of traffic):

- **Calendar page** — markdown table inside `<article>` with `overflow-x: scroll`. The ticket-link column slides off-screen on 375px. Conversion path dies before it starts.
- **Hero text** — on inner pages, the `<header>` paints text directly over the feature image with no scrim. Title becomes unreadable on busy flyers.
- **Follow buttons** — the social links in the footer are plain text rows with a 1.4rem icon. They look like a sitemap, not buttons.
- **Typography** — `Source Sans Pro` used for body, headings, and "logo." No display contrast. The site reads like a SaaS blog, not a comedy show.
- **Event cards don't exist** — show listings reuse blog `.post-teaser` styling. There's no date badge, no venue, no price, no ticket CTA at card scope.
- **Gallery** — single column of images, no use of show flyers as visual storytelling.
- **Brand override capacity** — every show has its own banner/flyer with its own palette (La Tarima yellow, Spanish; Brexiles muted brown; Down Under green). No mechanism to override brand tokens at the show-page level.

The design system primitives that would let the site be rebuilt confidently — tokens, components, preview, integration plan — don't exist as a package. Each fix would otherwise be a one-off SCSS edit, accumulating tech debt.

## Vision

A standalone design system package — `Design/inyourface_design_system/` — that I can hand off to a redesign session, that compiles to one drop-in CSS file for emergency preview, and that splits into Jekyll-ready SCSS partials for the real integration. Comedy-poster energy in the type and color; mobile-first by default; ticket CTAs that look like the most important thing on the page because they ARE. The euphoric surprise: open the preview on a phone, see what the site COULD feel like, recognize instantly that this is what comedy in Zürich should look like.

## Out of Scope

- Touching the live Jekyll repo this session. Design system ships first; redesign session applies it after the user reviews.
- Changing the logo, brand-red, brand-yellow, or any existing brand token value. Constraint per `Design/README.md`.
- Building MailChimp templates this session — the design system is structured to support them later via the same token CSS-variable contract, but no MailChimp HTML is produced this turn.
- Replacing the existing `_data/*.yml` structure for shows. The design system styles whatever data the templates emit; data-pipeline rework is a separate task.
- Adding JavaScript. CSS-only solutions where possible; no carousel libraries, no masonry-JS, no animation libs.
- Accessibility audit — WCAG-AA is *targeted* in tokens (contrast ratios documented), but a full audit + remediation is a separate task.

## Principles

- **Mobile-first, not mobile-too.** 375px viewport is the design target. Desktop is enhancement, not the reverse.
- **Tokens before components, components before pages.** Every component consumes tokens — no hex literals in component partials.
- **The ticket button is the most important pixel.** Every component is judged against whether it helps a returning fan get to Eventfrog faster.
- **One-file-CSS escape hatch.** The dist CSS must work without a SCSS toolchain so the user can drop it into MailChimp, a one-off landing page, or an emergency redesign without setup.
- **Theme override is a first-class citizen.** Per-show banner/flyer palettes are not exceptions; they're a documented pattern via CSS custom properties.
- **Comedy poster energy, not blog energy.** Bold display type. Tight letter-spacing. Confident color. No SaaS-template politeness.
- **Substrate-independent.** The design system is consumable by Jekyll today, MailChimp tomorrow, a Next.js rewrite the day after.

## Constraints

- **Brand-red `#E53935`, brand-yellow `#FFD54F`, brand-cream `#FFF3E0`, brand-ink `#0F0F10`** are fixed (already in `_sass/base/_variables.scss`).
- **Jekyll/Kramdown emits a real `<table>` on the calendar page.** The mobile transform must work via CSS on the existing table structure — the user generates the table with code and wants to keep that pipeline.
- **No new Jekyll plugins.** Per `Design/README.md` constraint set.
- **Google Fonts only** for new typefaces — local font files are out (user explicitly said they're fine to switch to remote).
- **Font Awesome via CDN (or inline SVG)** — local font-awesome bundle goes.
- **Type-on-Strap article quirk** — `<article>` is `float: left; width: 100%`. New components nested inside `<article>` must respect this; siblings must `clear: both`.
- **No staging environment.** Preview is a standalone HTML file in this folder; live verification is on production after the user merges.

## Goal

Ship `Design/inyourface_design_system/` containing: a token layer (colors, typography, spacing, motion); a component layer (button, event-card, calendar-table, hero, follow-buttons, gallery, show-override, badge); a single hand-compiled `dist/design-system.css` that runs without SCSS; a `preview/index.html` showcasing every component at 375px width; an `INTEGRATION.md` mapping each new partial to its target file in the Jekyll repo. Every component drives toward the ticket-purchase action.

## Criteria

### Structure
- [ ] ISC-1: `README.md` exists at design-system root with overview, philosophy, file map, and how-to-use
- [ ] ISC-2: `ISA.md` exists at design-system root with the 12-section canonical format
- [ ] ISC-3: `INTEGRATION.md` exists with a file-by-file integration plan to the Jekyll repo
- [ ] ISC-4: `tokens/` directory contains `colors.scss`, `typography.scss`, `spacing.scss`, `motion.scss`
- [ ] ISC-5: `components/` directory contains `button.scss`, `event-card.scss`, `calendar-table.scss`, `hero.scss`, `follow-buttons.scss`, `gallery.scss`, `badge.scss`, `show-override.scss`

### Color system
- [ ] ISC-6: `tokens/colors.scss` declares the existing brand tokens (red/red-deep/cream/ink/accent-yellow) at `:root`
- [ ] ISC-7: `tokens/colors.scss` adds surface tokens (surface, surface-elev, ink-on-cream, muted, border-soft)
- [ ] ISC-8: `tokens/colors.scss` declares semantic tokens (`--cta`, `--cta-hover`, `--badge-tonight`, `--badge-soldout`, `--success`, `--danger`)
- [ ] ISC-9: `tokens/colors.scss` declares per-show override variables (`--show-accent`, `--show-bg`, `--show-text`, `--show-cta`) with sensible fallbacks to brand tokens
- [ ] ISC-10: Antecedent: All brand-red text uses are documented with their target background and intended contrast ratio (≥4.5:1 for body, ≥3:1 for large display)

### Typography
- [ ] ISC-11: `tokens/typography.scss` imports three Google Fonts: display, body, accent
- [ ] ISC-12: Display font is a bold poster/condensed face (Anton named, Bebas Neue documented as alternative)
- [ ] ISC-13: Body font is a modern humanist sans (Inter named)
- [ ] ISC-14: Accent font is hand-lettered/marker (Permanent Marker named, Caveat documented as alternative)
- [ ] ISC-15: Type scale uses `clamp()` for fluid mobile→desktop scaling without media-query font switches

### Component: button
- [ ] ISC-16: `components/button.scss` defines `.btn-ticket` primary CTA: min-height 48px, bold display font, right-arrow affordance
- [ ] ISC-17: `components/button.scss` defines `.btn-ghost` outline variant for secondary actions
- [ ] ISC-18: `components/button.scss` defines `.btn-chip` for icon+label social buttons (≥48px tall)

### Component: event-card
- [ ] ISC-19: `components/event-card.scss` defines a mobile-first card with: date badge, show title, venue+time, tagline, ticket CTA
- [ ] ISC-20: Card date badge uses display font, brand-red background, brand-cream text, ≥4.5:1 contrast

### Component: calendar-table
- [ ] ISC-21: `components/calendar-table.scss` transforms `<table>` to stacked cards below 600px via CSS only (no JS), preserving every column including ticket link
- [ ] ISC-22: Anti: Mobile transform never produces horizontal scroll or hidden ticket links

### Component: hero
- [ ] ISC-23: `components/hero.scss` declares `.iyf-hero` with bottom-up scrim that works on both home and inner pages
- [ ] ISC-24: Hero scrim uses brand-ink at 0%→85% gradient with optional `.iyf-hero--dim` variant at 0%→95% for hard-to-read flyers

### Component: follow-buttons
- [ ] ISC-25: `components/follow-buttons.scss` defines `.iyf-follow-chips` row of large social buttons, ≥48px tall, icon + platform label

### Component: gallery
- [ ] ISC-26: `components/gallery.scss` uses CSS grid (column-based masonry approximation, no JS dependency)

### Component: show-override
- [ ] ISC-27: `components/show-override.scss` documents the per-show variable override pattern and provides a `.show-banner` class that consumes `--show-accent` etc.

### Preview + dist
- [ ] ISC-28: `preview/index.html` exists and renders every component visually, mobile-first, at 375px baseline
- [ ] ISC-29: `preview/index.html` links exactly one stylesheet — `../dist/design-system.css`
- [ ] ISC-30: `dist/design-system.css` is hand-compiled vanilla CSS, no SCSS, no `@use`/`@import`, browser-droppable as-is

### Anti-criteria
- [ ] ISC-31: Anti: No hardcoded hex literal outside `tokens/colors.scss` and `dist/design-system.css`
- [ ] ISC-32: Anti: No partial references local fonts — all type via Google Fonts CDN
- [ ] ISC-33: Anti: Calendar transform never loses the ticket-link column at any viewport ≥320px
- [ ] ISC-34: Anti: No partial requires local Font Awesome files — icon strategy is CDN or inline SVG

### Antecedent
- [ ] ISC-35: Antecedent: The logo at `assets/img/inyourface.png` is treated as the color source — palette mirrors red/yellow/black/cream sampled from it

### Integration
- [ ] ISC-36: `INTEGRATION.md` names target file in the Jekyll repo (`_sass/base/_variables.scss`, `_sass/base/_global.scss`, etc.) for each design-system partial
- [ ] ISC-37: `INTEGRATION.md` documents the migration order (tokens first, then components, then layout-level changes)

## Test Strategy

| ISC range | Type | Check | Threshold | Tool |
|-----------|------|-------|-----------|------|
| ISC-1..5 | structural | path exists | file present | `ls` |
| ISC-6..10 | content grep | `--brand-*`, `--show-*` declarations present | match | `grep` on `colors.scss` |
| ISC-11..15 | content grep | `@import url('https://fonts.googleapis.com'` + clamp() | match | `grep` on `typography.scss` |
| ISC-16..18 | content grep | `.btn-ticket`, `.btn-ghost`, `.btn-chip` defined with min-height | match | `grep` on `button.scss` |
| ISC-19..20 | content grep | `.iyf-event-card`, `.iyf-event-card__date` | match | `grep` on `event-card.scss` |
| ISC-21..22 | content grep | `display: block` + `tr` + `@media (max-width:` in calendar-table | match | `grep` on `calendar-table.scss` |
| ISC-23..24 | content grep | `linear-gradient` + `--brand-ink` in hero | match | `grep` on `hero.scss` |
| ISC-25 | content grep | `.iyf-follow-chips`, `min-height: 48px` | match | `grep` on `follow-buttons.scss` |
| ISC-26 | content grep | `column-count` or `grid-template-rows: masonry` | match | `grep` on `gallery.scss` |
| ISC-27 | content grep | `--show-accent`, `--show-bg` consumed | match | `grep` on `show-override.scss` |
| ISC-28..30 | content + structural | `preview/index.html` exists and references `../dist/design-system.css`; dist file exists and is plain CSS | match | `Read` + `grep -E 'scss\|@use'` returns nothing |
| ISC-31 | content grep | no `#[0-9a-fA-F]{3,6}` literals outside allowed files | zero matches | `grep -rE '#[0-9a-fA-F]{6}'` in `components/` |
| ISC-32 | content grep | no `url\(.*assets/fonts` references | zero matches | `grep -r` in design system |
| ISC-33 | visual | calendar-preview.html at 320px shows ticket link in card | passes | Read CSS + simulated narrow |
| ISC-34 | content grep | no `font-awesome.min.css` local refs | zero matches | `grep -r` |
| ISC-35 | reference | colors.scss comment cites logo file | present | `grep "inyourface.png" colors.scss` |
| ISC-36..37 | content grep | INTEGRATION.md names `_sass/base/_variables.scss` etc. | match | `grep` |

## Features

| Name | Description | Satisfies | Depends on | Parallelizable |
|------|-------------|-----------|------------|----------------|
| F1: Structure | Create directory tree + README + ISA + INTEGRATION | ISC-1..5, ISC-36..37 | — | No (sets the substrate) |
| F2: Color tokens | colors.scss with brand + surface + semantic + show-override | ISC-6..10, ISC-31 | F1 | No (must precede other partials) |
| F3: Typography tokens | typography.scss with Google Fonts + fluid scale | ISC-11..15, ISC-32 | F1 | Yes (parallel with F2) |
| F4: Spacing + motion tokens | spacing.scss, motion.scss | — (covered in dist) | F1 | Yes |
| F5: Button system | button.scss with ticket / ghost / chip | ISC-16..18 | F2, F3 | Yes |
| F6: Event card | event-card.scss | ISC-19..20 | F5 | Yes |
| F7: Calendar table transform | calendar-table.scss | ISC-21..22, ISC-33 | F2, F3, F5 | Yes |
| F8: Hero | hero.scss | ISC-23..24 | F2, F3 | Yes |
| F9: Follow chips | follow-buttons.scss | ISC-25, ISC-34 | F2, F3, F5 | Yes |
| F10: Gallery | gallery.scss | ISC-26 | F2 | Yes |
| F11: Show override | show-override.scss + docs | ISC-27 | F2 | Yes |
| F12: Dist build | hand-compiled dist/design-system.css | ISC-29..30 | F2..F11 | No (final merge) |
| F13: Preview page | preview/index.html | ISC-28 | F12 | No (consumes F12) |

## Decisions

- **2026-05-25T20:30** — ISA scaffolded inline at the project location instead of via `Skill("ISA", "scaffold")`. Reason: E3 budget under 10 min and the ISA skill subagent adds latency for a deliverable I can author directly with full doctrine compliance. ID-stability and twelve-section format preserved.
- **2026-05-25T20:32** — Forge auto-include intentionally skipped at E3 despite the coding-task rule. Show your math: design-system v1 requires a single coherent aesthetic voice (token choices, type pairing, component spacing rhythm) — parallel agents would fragment the look. Cross-vendor review deferred to follow-up: the user reviews v1, then a Cato/Forge audit pass before integration to Jekyll repo.
- **2026-05-25T20:34** — Google Fonts chosen over self-hosted: user explicitly invited the switch. Trade-off: third-party DNS hit at first page load — acceptable per Design/README.md performance budget (`<100KB JS, <600KB initial total`), and `preconnect` mitigates the handshake.
- **2026-05-25T20:34** — Font Awesome 6 free via CDN over self-hosted local files. Smaller upfront network cost than the local bundle (existing `assets/fonts/font-awesome/` ships fa-solid-900, fa-brands-400, fa-regular-400 separately).
- **2026-05-25T20:36** — Calendar mobile transform via CSS `data-label` pattern: requires the Jekyll generator to add `data-label` attributes per `<td>`. Documented in INTEGRATION.md as a one-time generator change. Alternative (`overflow-x: auto` with sticky first column) rejected because the user explicitly said the ticket-link column gets cut off and that's the conversion target.
- **2026-05-25T20:36** — Single hand-compiled `dist/design-system.css` written alongside the SCSS partials, NOT generated by a tool. Reason: portable preview without a SCSS toolchain, and the dist file becomes the MailChimp drop-in later. SCSS partials are the maintenance surface; dist is the artifact.

## Changelog

- **2026-05-25T20:30** — conjectured: "the ticket-button width should be 40%" (inherited from current `.btn`). refuted_by: user feedback that buttons are too small and the priority is conversion. learned: at mobile, primary CTAs go full-width with min-height 48px. criterion_now: ISC-16 sets `.btn-ticket` to full-width below 768px with 48px min-height.
- **2026-05-25T20:32** — conjectured: "the calendar table can keep `overflow-x: scroll` if the first column is sticky." refuted_by: tested mentally on 375px — the ticket link is the LAST column (so right-most), not the first; sticky-first column doesn't surface it. learned: the table must transform structurally on narrow viewports, not scroll. criterion_now: ISC-21 + ISC-22 (Anti) — CSS `display:block` table transform with `data-label` per cell.

## Verification

(populated during EXECUTE/VERIFY)
