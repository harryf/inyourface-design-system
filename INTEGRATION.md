# Integration Plan — Design System → Jekyll Repo

> **Backsync notes (2026-05-25)** — what the design system absorbed *back* from the integration. Read these before re-applying anything; the v1 spec drifted on a few points during implementation:
>
> | What v1 specified | What v1.1 ships | Why |
> |---|---|---|
> | `.iyf-hero` defaulted to `justify-content: flex-end` + `min-height: 60vh` | Centered + `min-height: auto` by default | The flex-end + 60vh combo left a big empty band above the h1 on every page; folded the compact behaviour into the base selector. `.iyf-hero--centered` collapsed to a no-op alias |
> | `.iyf-hero--compact` carried its own `align-items: center; text-align: center; min-height: auto` | Same behaviour but inherited from `.iyf-hero` | Compact stays for tighter padding + smaller title only |
> | Calendar `tbody td:nth-child(3) a` used `var(--brand-ink)` | `var(--text)` | Brand-ink is always dark; broke in dark mode (Type-on-Strap renders dark by default on inyourfacecomedy.ch) |
> | Calendar mobile col-4 used `var(--brand-ink-soft)` | `var(--muted)` (and `html[data-theme="dark"]` overrides `--muted` to soft cream) | Same dark-mode fix |
> | Event-card tagline used `var(--brand-ink-soft)` | `var(--muted)` | Same dark-mode fix |
> | `show-override.scss` shipped with `[data-show="jokesjokes"]` + `jokes-jokes-jokes` | Adds `[data-show="jokesjokesjokes"]` (real permalink) | The actual Jekyll permalink is `/jokesjokesjokes/`; neither original selector matched |
> | `tokens/colors.scss` had no `--cta-bg` token | Legacy aliases `--cta-bg` / `--cta-bg-hover` carry over | Type-on-Strap legacy `.btn` consumers reference them; kept until everything renders via `.btn-ticket` / `.btn-ghost` |
> | Step 7 (Gallery) originally said "wrap the masonry div" | Replace the JS-Masonry markup entirely with the `.iyf-gallery` CSS-columns layout | Coexistence created visual fights; the design system component is JS-free |
> | Step 3 + Step 8 specified as separate work | Folded into one commit | Both touch `_layouts/post.liquid`; separating them is churn |

> **v1.2 — Jump to Next Show (2026-05-25)** — additive UX on the calendar.
>
> | Surface | Provided by design system | Provided by consuming app |
> |---|---|---|
> | Subtle row accent | `tr[data-next-show="true"]` — yellow tint mixed into `--surface-elev` + inset left-edge yellow accent. Theme-aware. Mobile-card variant overrides `border-left` to yellow. | — |
> | Click flash | `tr.is-jump-flash` runs `@keyframes iyfJumpFlash` for 1.4s, gated on `prefers-reduced-motion: no-preference`. | — |
> | Button | `.btn-ghost.btn-ghost--on-dark` (already shipped) is the intended variant. | The host renders the button inside its hero. |
> | "Which row is next?" | — | Vanilla JS: parse `.iyf-month-heading` year + each row's first cell, find first `<tr>` whose date `>= localMidnight(today)`, set `data-next-show="true"`, wire click → `scrollIntoView({behavior: reducedMotion ? 'auto' : 'smooth', block: 'center'})` + toggle `.is-jump-flash`. |
> | All-past calendar | — | Host JS hides the button (`button.hidden = true`). |
>
> Reference JS implementation: `Design/inyourfacecomedy/assets/js/jump-to-next-show.js` (116 lines, zero deps, IIFE + `'use strict'`).

> **v1.3 — Comedians directory (2026-05-26)** — new surface at `/comedians/` and `/comedians/<slug>/`. Sourced from Grist via a Ruby cron script that lives in the consuming Jekyll repo, not in the design system.
>
> | Surface | Provided by design system | Provided by consuming app |
> |---|---|---|
> | Comedian grid | `.iyf-comedian-grid` — mobile-first responsive grid (2 / 3 / 4 columns at <480 / ≥768 / ≥1024). | Hub page iterates `site.comedians \| sort: "title"`. |
> | Comedian card | `.iyf-comedian-card` — 1:1 photo crop, object-fit:cover, hover lift mirroring `.iyf-event-card`. Placeholder gradient when no photo. | Markup: anchor wrapping `__media` (img or `__media--placeholder`) + `__name`. |
> | Profile layout | `.iyf-comedian-profile` — header (photo+name, stacks under 768px), bio prose, socials row, back link. | Markup ships from `_layouts/comedian.liquid`. |
> | Social link chips | `.iyf-social-link` + `--instagram / --tiktok / --facebook / --x / --youtube / --website` modifiers. Tap-target ≥44px (uses `--touch-min`). Hover paints brand colour. | Liquid conditional renders only platforms with a non-empty URL. |
> | Photo size budget | — | Ruby script resizes via `sips` until file is ≤95KB; saved to `assets/img/comedians/<slug>.<ext>`. |
> | Privacy filter | — | Allowlist of public fields in the script; Phone/Email never written. `Live=false` deletes the slug page and its photo. |
>
> Reference implementations:
> - SCSS: `components/comedian-card.scss` + `components/comedian-profile.scss` (mirrored at `_sass/components/_comedian-card.scss` + `_comedian-profile.scss` in the Jekyll repo).
> - Cron script: `Design/inyourfacecomedy/script/sync-comedians.rb` (Ruby, stdlib + `sips` only, no gem deps).


A file-by-file map for landing this design system inside `Design/inyourfacecomedy/`. The order matters — tokens first, then components, then layout edits. Verify after every step with `Skill("Interceptor")` at 320 / 375 / 768 / 1440 px.

The Jekyll repo path below is shorthand for `Design/inyourfacecomedy/`. The design-system path is `Design/inyourface_design_system/`.

## Migration order

1. **Tokens** (no visual change — sets the substrate)
2. **Buttons** (single visible upgrade, low blast radius)
3. **Hero** (fixes the inner-page legibility issue)
4. **Calendar** (highest-leverage UX win — ship to its own branch + verify on prod)
5. **Footer follow chips** (replaces the small text rows the brief flagged)
6. **Event card** (new component — used by home upcoming + show pages)
7. **Gallery** (replaces single-column gallery on `/moments/`)
8. **Show overrides** (per-page palette via `<article class="show-banner" data-show="…">`)

## Step 1 — Tokens

| Design-system file | Jekyll target | Action |
|---|---|---|
| `tokens/colors.scss` | `_sass/base/_variables.scss` | **Merge.** Existing `:root` block already has `--brand-red` / `--brand-red-deep` / `--brand-cream` / `--brand-ink` / `--accent-yellow`. Append the new tokens (`--brand-yellow-hot`, surface-*, semantic, show-*) inside the same `:root`. |
| `tokens/typography.scss` | NEW `_sass/base/_typography.scss` | Create new partial. Import it from `_sass/type-on-strap.scss` immediately after `_variables.scss`. |
| `tokens/spacing.scss` | `_sass/base/_variables.scss` | Append the `:root` block. The existing `$break`, `$sm-break` etc. SCSS vars stay; new tokens are additive. |
| `tokens/motion.scss` | NEW `_sass/base/_motion.scss` | Create new partial. Import after `_typography.scss`. |

In `_sass/type-on-strap.scss`, add the imports near the top, after `_variables`:

```scss
@import "base/typography";
@import "base/motion";
```

In `_includes/default/head.liquid` add the Google Fonts link:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Anton&family=Inter:wght@400;500;600;700&family=Permanent+Marker&display=swap" rel="stylesheet">
```

Replace the local Font Awesome bundle (`assets/fonts/font-awesome/*`) with the CDN link — same `<link>` block:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
```

Now drop `assets/fonts/font-awesome/` and `assets/fonts/source-sans-pro/` from the repo. Verify on prod after deploy.

**Update `_variables.scss` SCSS vars:**

```scss
$font-family-main:     'Inter', system-ui, Helvetica, Arial, sans-serif;
$font-family-headings: 'Anton', 'Bebas Neue', 'Oswald', Impact, sans-serif;
$font-family-logo:     'Anton', 'Bebas Neue', 'Oswald', Impact, sans-serif;
```

Now Type-on-Strap's existing `h1..h6` selectors automatically pick up Anton.

## Step 2 — Buttons

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/button.scss` | `_sass/base/_global.scss` | **Replace** the existing `.btn` + `.btn-secondary` blocks with the design-system definitions. Class names change: `.btn` → `.btn-ticket`, `.btn-secondary` → `.btn-ghost`. |

Then grep-replace usages across templates:

```bash
cd Design/inyourfacecomedy
rg -l '\.btn\b' --type=liquid --type=html --type=md
# Update each .btn class to .btn-ticket (or .btn-ghost for the outline variant)
```

Files to update (per current repo state):

- `_includes/default/footer.liquid` — three `.btn` instances become `.btn-ticket`
- `_layouts/post.liquid` — primary "Get Tickets" link → `.btn-ticket--xl`
- `pages/1_follow.md` — `.btn` in Mailchimp section → `.btn-ticket`
- `pages/2_calendar.md` — leave the markdown table alone; the calendar transform handles it
- `_includes/blog/post_info.liquid` — any `.btn` becomes `.btn-ghost`

## Step 3 — Hero (collapses the two implementations)

Currently there are two hero code paths:

- `<div id="main" class="call-out">` on home (`_layouts/home.liquid`)
- `<header id="main">` inside `<article class="feature-image">` on inner pages (`_layouts/page.liquid` and `_layouts/post.liquid`)

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/hero.scss` | NEW `_sass/components/_hero.scss` | Create new partial. Import from `_sass/type-on-strap.scss`. |
| `_layouts/home.liquid` | Edit | Replace `<div id="main" class="call-out">` with `<header class="iyf-hero iyf-hero--centered" style="background-image: url('{{ page.feature-img | relative_url }}');">`. Use `<p class="iyf-hero__eyebrow">…</p>` for the kicker, `<h1 class="iyf-hero__title">` for the headline. |
| `_layouts/page.liquid` | Edit | Replace `<header id="main">` inside `<article class="feature-image">` with `<header class="iyf-hero iyf-hero--compact" style="…">`. Same structure: eyebrow / title / subtitle / actions. |
| `_layouts/post.liquid` | Edit | Same swap as `page.liquid`, with the addition of `<a class="btn-ticket btn-ticket--xl">Get Tickets</a>` in `.iyf-hero__actions` when `page.ticket_url` exists. |
| `_sass/layouts/_blog.scss` | Edit | Remove the `.call-out` block (now covered by `.iyf-hero`). Keep `.posts`. |
| `_sass/layouts/_posts.scss` | Edit | Remove the `.feature-image header` background-size cascade (lines 125–142). The new hero handles its own background-size. |

## Step 4 — Calendar (highest-leverage UX win)

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/calendar-table.scss` | NEW `_sass/components/_calendar-table.scss` | Create new partial. Import from `_sass/type-on-strap.scss`. |
| `pages/2_calendar.md` | Edit | Wrap each markdown table block in `<div class="iyf-calendar" markdown="1">…</div>`. Kramdown processes the inner markdown; the CSS handles everything else. The generator (whatever produces this file) needs to emit the wrapper around each table. |
| `pages/2_calendar.md` | Edit | Replace each `## May 2026` H2 with `<h2 class="iyf-month-heading">May 2026</h2>` (the design system style). Replace the italic flavour line with `<p class="iyf-month-flavor">May the laughs…</p>`. |

The generator that creates this calendar table needs to update its template to emit:

```liquid
<h2 class="iyf-month-heading">{{ month_name }}</h2>
<p class="iyf-month-flavor">{{ month_flavor }}</p>
<div class="iyf-calendar" markdown="1">

| Date | Day | Show | Info | Tickets |
|------|-----|------|------|---------|
| May 5 | Tue | [Jokes, Jokes, Jokes](…) | Local comics … | [Get Tickets](…) |
| …

</div>
```

The blank lines around the table are important — Kramdown needs them.

## Step 5 — Footer follow chips + newsletter

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/follow-buttons.scss` | NEW `_sass/components/_follow-buttons.scss` | Create + import. |
| `_includes/default/footer.liquid` | Edit | Replace `.footer-mailchimp` block with `<div class="iyf-newsletter-card">…</div>`. |
| `_includes/default/footer.liquid` | Edit | Replace `.footer-social .social-list ul` with `<ul class="iyf-follow-chips">…</ul>` (the existing Liquid loop over `site.social.*` stays — only the wrapping markup + classes change). |
| `_sass/includes/_footer.scss` | Edit | Remove the `.social-list`, `.footer-mailchimp .btn`, `.footer-block .btn` rules — now handled by the design-system components. Keep `.footer-grid` and `.footer-meta`. |
| `pages/1_follow.md` | Edit | The "Channels" section becomes `<ul class="iyf-follow-chips iyf-follow-chips--light">…</ul>` (light variant for the body surface). |

## Step 6 — Event card (new component)

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/event-card.scss` | NEW `_sass/components/_event-card.scss` | Create + import. |
| NEW `_includes/event-card.liquid` | Create | A reusable Liquid include accepting `post` (or a `_data/shows.yml` row). Emits the `.iyf-event-card` markup. |
| `_layouts/home.liquid` | Edit | Add an "Upcoming shows" section using `{% include event-card.liquid post=show %}` in a loop over the next 3 show posts. |
| Each show page (`_layouts/post.liquid`) | Edit | Add an event-card at the top of the body for the next occurrence, alongside the hero. |

The event-card include reads `post.next_event_date`, `post.venue_slug`, `post.ticket_url`, `post.price_chf`, `post.tagline` — all already present in the existing post frontmatter.

## Step 7 — Gallery

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/gallery.scss` | NEW `_sass/components/_gallery.scss` | Create + import. |
| `_includes/gallery.html` | Edit | Wrap the existing image loop in `<div class="iyf-gallery">` and emit each image inside a `<figure>` with an optional `<figcaption>`. |
| `pages/3_gallery.md` | Edit | Optional: add a `<div class="iyf-flyer-strip">` above the masonry gallery showing the next 4 show flyers as a horizontal scroller. |

## Step 8 — Show overrides

| Design-system file | Jekyll target | Action |
|---|---|---|
| `components/show-override.scss` | NEW `_sass/components/_show-override.scss` | Create + import. |
| `_layouts/post.liquid` | Edit | Wrap the `<article>` in `<article class="show-banner" data-show="{{ page.permalink \| replace: '/','' \| strip }}">`. The `data-show` is derived from the permalink (e.g. `/comedybrew/` → `data-show="comedybrew"`), which matches the selectors in the SCSS partial. |

The hero, event-card, and ticket button on each show page now pick up that show's palette automatically. Brand tokens untouched.

## Verification per step

After each step:

```bash
# Local serve
cd Design/inyourfacecomedy
bundle exec jekyll serve
# Visit http://127.0.0.1:4000

# After merge to master + Netlify auto-deploy
# Verify on prod at the 4-width grid:
Skill("Interceptor", "open inyourfacecomedy.ch and capture at 320, 375, 768, 1440")
```

Specific gates:

| Step | Gate |
|------|------|
| 1 — tokens | No visual regression. Fonts visibly change to Anton + Inter + Permanent Marker. |
| 2 — buttons | All ticket buttons full-width on mobile, comfortable on desktop. Hover state visible. |
| 3 — hero | Title legible over both home flyer and any inner-page feature image. Verify with two visually busy show flyers. |
| 4 — calendar | At 375px, every row is a card. **Every row shows the "Get Tickets" button without horizontal scroll.** |
| 5 — footer | Follow chips are obviously buttons, not text links. Newsletter card stands out. |
| 6 — event card | Home page surfaces 3 upcoming shows above the fold on mobile. |
| 7 — gallery | Masonry grid renders even without `grid-template-rows: masonry` (column fallback works). |
| 8 — show override | Each show page reads its own palette without leaking into the rest of the site. |

## Notes

- **Brand tokens are immutable.** Do not change `--brand-red`, `--brand-yellow`, `--brand-cream`, `--brand-ink` values. Adjust `--show-*` only.
- **Type-on-Strap article quirk.** `<article>` is `float: left; width: 100%`. The new hero + event-card components are siblings to or children of `<article>`. When in doubt, add `clear: both` to anything that follows an `<article>`.
- **Branch per step.** Each migration step lands on its own branch + PR (one branch per task per the project `CLAUDE.md` § Branch workflow). Don't batch all eight into one PR.
- **Kramdown gotcha.** When you wrap a markdown table in a `<div>`, add `markdown="1"` to the div so Kramdown processes the inner content. Test before merge.
