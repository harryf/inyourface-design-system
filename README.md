# IN YOUR FACE — Design System v1.1

> **v1.1 — backsynced from the live integration on 2026-05-25.** Hero defaults
> changed from "flex-end + 60vh" to "centered + content-height" because the
> shipped Jekyll site rendered with a big empty band above the h1. Calendar +
> event-card text now read theme-aware tokens (`--text`, `--muted`) so the
> components stay legible in dark mode (the `inyourfacecomedy.ch` site sets
> `color_theme: dark`). See `INTEGRATION.md` "Backsync notes" for the full
> diff between v1 spec and what shipped.

Mobile-first design system for **inyourfacecomedy.ch**. Comedy-poster energy in the type, logo-derived palette, ticket-CTA as the most important pixel on every page.

The design system is the package; applying it to the Jekyll repo is a separate session (`Roadmap/Sprint-1/`).

## What you get

```
inyourface_design_system/
├── README.md                 ← you are here
├── ISA.md                    ← canonical articulation (ideal-state artifact)
├── INTEGRATION.md            ← file-by-file plan for the Jekyll repo
├── tokens/
│   ├── colors.scss           ← brand + surface + semantic + show-override
│   ├── typography.scss       ← Anton + Inter + Permanent Marker, fluid scale
│   ├── spacing.scss          ← 8px scale, radii, breakpoints, touch floor
│   └── motion.scss           ← easing curves, durations, prefers-reduced-motion
├── components/
│   ├── button.scss           ← .btn-ticket / .btn-ghost / .btn-chip / .btn-link
│   ├── event-card.scss       ← the atomic event card
│   ├── calendar-table.scss   ← Jekyll <table> → mobile cards
│   ├── hero.scss             ← readable overlay, home + inner pages
│   ├── follow-buttons.scss   ← big social chips, newsletter card
│   ├── gallery.scss          ← masonry + flyer-strip
│   ├── badge.scss            ← tonight / sold-out / new / lang
│   └── show-override.scss    ← per-show palette via CSS custom properties
├── dist/
│   └── design-system.css     ← single hand-compiled drop-in CSS file
├── preview/
│   └── index.html            ← open in browser to see every component
└── guidelines/               ← TODO (this folder is a placeholder)
```

## Quick preview

```bash
open preview/index.html
```

The preview frame is 480px wide so it reads as a phone. Resize the window past 900px to see the desktop layout.

## Philosophy

1. **The ticket button is the most important pixel.** Every component is judged against whether it helps a returning fan get to Eventfrog faster. If a design choice fights conversion, the choice is wrong.
2. **Mobile-first, not mobile-too.** 375px viewport is the design target. Desktop is enhancement, not the reverse. 70% of traffic is mobile per `01-ANALYTICS-BASELINE.md`.
3. **Tokens before components, components before pages.** No hex literal lives in a component partial. Adding a colour means adding a token first.
4. **Theme override is first-class.** Per-show banner/flyer palettes (La Tarima yellow-on-brown, Down Under green, Pulp Non-Fiction noir) override `--show-*` variables — brand tokens stay untouched.
5. **One-file CSS escape hatch.** `dist/design-system.css` is hand-compiled vanilla CSS. No SCSS toolchain needed to use it — drop it into a one-off page, a MailChimp template, an emergency redesign.
6. **Comedy poster energy, not blog energy.** Bold display type, tight letter-spacing, confident colour. No SaaS-template politeness.
7. **Substrate-independent.** Tokens are CSS custom properties on `:root`. Consumable by Jekyll today, MailChimp tomorrow, a Next.js rewrite the day after.

## Type pairing

| Role | Family | Use case | Fallback |
|------|--------|----------|----------|
| Display | **Anton** | H1, H2, event titles, badges, button labels | Bebas Neue, Oswald, Impact |
| Body | **Inter** | Body copy, meta, nav, form labels | system-ui, -apple-system |
| Accent | **Permanent Marker** | Taglines, "Tonight!" callouts, month flavour text | Caveat, Brush Script MT |

All three load via a single Google Fonts request:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Anton&family=Inter:wght@400;500;600;700&family=Permanent+Marker&display=swap" rel="stylesheet">
```

## Colour system

The brand palette is sampled from `assets/img/inyourface.png`:

- `--brand-red`  `#E53935` — logo ring
- `--brand-red-deep`  `#B71C1C` — pressed / hover
- `--brand-yellow`  `#FFD54F` — logo face
- `--brand-yellow-hot`  `#FFB300` — "Tonight" badge
- `--brand-cream`  `#FFF3E0` — logo inner ring, paper feel
- `--brand-ink`  `#0F0F10` — logo outline, body ink

**Per-show overrides** are scoped to `.show-banner[data-show="…"]` and only re-declare `--show-*` variables. Brand tokens never get touched. Add a new show palette in `components/show-override.scss`. `data-show` selectors must match the actual Jekyll **permalink** (`/comedybrew/` → `comedybrew`, `/jokesjokesjokes/` → `jokesjokesjokes`). The shipped file ships kebab + un-kebabed aliases for the seven recurring shows.

**Dark mode contract.** The host site can render in dark mode via Type-on-Strap's `data-theme="dark"` attribute. The design-system's `html[data-theme="dark"]` block remaps `--surface-elev`, `--ink-on-cream`, `--muted`, `--border-soft`, and `--border-strong` so any component that reads those tokens stays readable. Components MUST use `--text` / `--muted` / `--show-text` for body text — never the always-dark `--brand-ink` / `--brand-ink-soft` primitives.

```html
<article class="show-banner" data-show="la-tarima">
  <section class="iyf-hero">…</section>
  <article class="iyf-event-card">…</article>
</article>
```

The hero and card pick up La Tarima's yellow + brown palette automatically.

## Components at a glance

- **`.btn-ticket`** — primary CTA. Min-height 48px. Full-width on mobile (the most important button gets the whole row). `--xl` and `--sm` size variants. Built-in `→` arrow.
- **`.btn-ghost`** — outline secondary. `--on-dark` variant for hero usage.
- **`.btn-chip`** — icon + label, for social buttons. Always brand-ink on light or brand-cream on dark.
- **`.iyf-event-card`** — date badge + title + tagline + ticket footer. Reads `--show-*` so it themes per show.
- **`.iyf-calendar`** — wraps the existing Markdown table. CSS-only mobile transform: each `<tr>` becomes a stacked card under 768px with the ticket link as a full-width button at the bottom.
- **`.iyf-hero`** — replaces both `.call-out` and `.feature-image header`. Bottom-up scrim + radial vignette. **Centered + content-height by default** (no enforced min-height; background image shows behind the text only). `--dim` for busy flyers (used on show pages — mobile title font caps at `--fs-xl`). `--compact` for inner pages (tighter padding + smaller title). `--centered` is now a no-op alias kept for templates that still emit it.
- **`.iyf-follow-chips`** — replaces the footer's `<ul class="social-list">` text rows.
- **`.iyf-newsletter-card`** — replaces the inline footer Mailchimp anchor block.
- **`.iyf-gallery`** — masonry grid; `.iyf-flyer-strip` is the horizontal-scroll variant for the home page.
- **`.iyf-badge`** — `--tonight` / `--soldout` / `--new` / `--lang` / `--muted`.

## Integration path

Read `INTEGRATION.md` for the file-by-file integration plan. Migration order:

1. Tokens — `tokens/colors.scss` + `tokens/typography.scss` merge into `_sass/base/_variables.scss` and a new `_sass/base/_typography.scss`.
2. Components — copy each `components/*.scss` into `_sass/components/` and add `@import` lines to `_sass/type-on-strap.scss`.
3. Layouts — touch one template at a time. Calendar first (highest-leverage), then footer, then hero, then show pages.
4. Verify after each step via `Skill("Interceptor")` at 320 / 375 / 768 / 1440px.

## Anti-criteria (what this design system promises NOT to do)

- ❌ No hex literals outside `tokens/colors.scss` and `dist/design-system.css`.
- ❌ No reliance on local font files. Everything loads via Google Fonts.
- ❌ No reliance on local Font Awesome. CDN or inline SVG.
- ❌ No JavaScript dependency for any component (carousel-strip uses native scroll-snap; gallery uses CSS columns).
- ❌ No mobile transform that hides the ticket-link column.

## What this design system intentionally does NOT cover (this session)

- Live MailChimp templates — the token CSS-variable contract makes them straightforward later, but no template HTML is shipped here.
- A full accessibility audit. Contrast for tokens is documented; component-by-component audit + remediation is a separate task per `Design/README.md` Sprint-1+ backlog.
- Replacing the existing `_data/*.yml` show schema. The components style whatever data the templates emit.

## Where to next

Open `preview/index.html`. Resize from 375 to 1024px. If it looks right, the next session is `Roadmap/Sprint-1/01-design-system-tokens/` — wiring tokens + a single component (probably the calendar) into the Jekyll repo and verifying on production.
