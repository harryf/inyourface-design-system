# MailChimp mapping

MailChimp templates don't understand modern CSS — no custom properties, no `clamp()`, no `:has()`, no flex/grid in older clients. The design-system tokens still map cleanly, but you have to **flatten** them.

## Approach: hand-resolve tokens at template build

For each MailChimp template, take the relevant section of `dist/design-system.css` and:

1. **Replace `var(--token)` with the literal value.**
   - `var(--brand-red)` → `#E53935`
   - `var(--brand-cream)` → `#FFF3E0`
   - `var(--brand-ink)` → `#0F0F10`
2. **Replace `clamp()` with a single fixed value** (use the desktop value).
   - `clamp(1.13rem, 1.07rem + 0.30vw, 1.30rem)` → `18px`
3. **Use inline styles** on every element. MailChimp strips most `<style>` blocks in some clients (Outlook especially).
4. **Use a single body font.** MailChimp won't reliably load Google Fonts in every client. Fall back to `Inter, Arial, sans-serif`. Anton can be used in image-rendered headers if you want comedy-poster type — render the headline as an image with alt text.

## Reusable layout primitives

These design-system pieces translate cleanly to MailChimp HTML emails:

- **Newsletter card** (`.iyf-newsletter-card`) — make it the email hero.
- **Event card** (`.iyf-event-card`) — one card per upcoming show. Stack vertically (don't grid). On mobile-only clients this works automatically. Use `<table role="presentation">` for the outer wrapper to survive Outlook.
- **Ticket button** (`.btn-ticket`) — center it, full-width on mobile. Use a `<a>` styled with `display: inline-block; padding: 14px 22px; background: #E53935; color: #FFF3E0;`.
- **Footer follow chips** (`.iyf-follow-chips`) — render as a single row of small icons in MailChimp (the chip-grid layout doesn't reliably work in email clients).

## Email-specific tweaks

- Min font size for body: **16px** (no smaller). MailChimp's mobile clients zoom out below this.
- Min touch target: **44×44 px** (Apple Mail standard, slightly tighter than the WCAG 48px floor).
- Max content width: **600px** (the universal email width).
- Avoid web fonts in the body; reserve them for image-rendered headers.
- Test in Litmus or MailChimp's preview against Gmail / Apple Mail / Outlook 365 / Outlook desktop / iOS / Android.

## Template skeleton

```html
<table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background:#FFFFFF; font-family: Inter, Arial, sans-serif; color:#0F0F10;">
  <tr><td align="center">
    <table role="presentation" width="100%" style="max-width:600px;" cellpadding="0" cellspacing="0">
      <!-- Hero block (newsletter-card style) -->
      <tr><td style="background:#0F0F10; color:#FFF3E0; padding:32px 24px;">
        <h1 style="font-family: Arial Black, Impact, sans-serif; text-transform:uppercase; letter-spacing:0.02em; color:#FFD54F; margin:0 0 12px;">English stand-up · Zürich</h1>
        <p style="font-size:16px; line-height:1.5; margin:0;">This week's shows + tickets. As always: no algorithm, just dates.</p>
      </td></tr>

      <!-- Event card -->
      <tr><td style="padding:24px; background:#FFF8EE;">
        <table role="presentation" width="100%" cellpadding="0" cellspacing="0">
          <tr>
            <td valign="top" style="width:64px;">
              <table role="presentation" cellpadding="0" cellspacing="0" style="background:#E53935; color:#FFF3E0; width:56px; height:56px; border-radius:8px; text-align:center;">
                <tr><td>
                  <div style="font-family: Arial Black, Impact, sans-serif; font-size:24px; line-height:1;">28</div>
                  <div style="font-size:10px; text-transform:uppercase; letter-spacing:0.1em;">May</div>
                </td></tr>
              </table>
            </td>
            <td valign="top" style="padding-left:12px;">
              <div style="font-size:12px; letter-spacing:0.08em; text-transform:uppercase; color:#6B6B70;">Thu · 19:30 · Robins</div>
              <h2 style="font-family: Arial Black, Impact, sans-serif; text-transform:uppercase; font-size:22px; margin:4px 0 8px;">Comedy Brew</h2>
              <p style="font-size:14px; color:#6B6B70; margin:0 0 16px;">Cold beer, hot takes, done by 10pm.</p>
              <a href="…" style="display:inline-block; background:#E53935; color:#FFF3E0; padding:12px 22px; border-radius:6px; text-decoration:none; font-family: Arial Black, Impact, sans-serif; text-transform:uppercase; letter-spacing:0.04em;">Get tickets →</a>
            </td>
          </tr>
        </table>
      </td></tr>
    </table>
  </td></tr>
</table>
```

That's the design system, hand-resolved for email. The tokens stay the source of truth; the email build is a render target.
