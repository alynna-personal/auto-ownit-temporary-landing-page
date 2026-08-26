# Auto Ownit — Landing Page

A two-page static site (no build step, no framework) acting as a temporary front
door while the main Auto Ownit website is offline. Its single job is to get a
business owner to a submitted pre-application.

Deploy by uploading the whole folder. `index.html` must sit at the web root.
Nothing to install or compile.

---

## 1. Structure

```
/
├── index.html                 Landing page
├── pre-application.html       Zoho form page
├── css/
│   ├── styles.css             Base + landing page styles
│   └── pre-application.css    Base + form page styles
└── assets/
    ├── Logo.svg               Header + footer logo
    ├── WhatsApp.svg           "Talk With Us" button icon
    ├── telephone.svg
    ├── email.svg
    ├── pin-location.svg       Adelaide + Perth footer icons
    ├── clock.svg              Footer hours icon
    ├── si_Arrow_left.svg      "Back to home" arrow
    ├── hero-bg.png            Hero background — includes the ute
    ├── ABN_CARD.png           Requirement card 1
    ├── DRIVERS_LICENSE_CARD.png
    ├── STEADY_INCOME.png
    ├── cherry-tigo-4.png      Vehicle card 1
    ├── GWM-Cannon-Vanta-1024x1024.png
    ├── 31-1024x1024.png       Vehicle card 3 (Corolla)
    └── 2025 Toyota LandCruiser Prado Altitude 4×4 – Previously Owned.png
```

### How the CSS is split

**Both stylesheets contain a full copy of the same base layer** — the `:root`
tokens, resets, `.btn`, `.announce`, `.site-header`, `.logo`. Below that base,
each file adds its own page-specific rules:

- `styles.css` → hero, stats, requirements, vehicles, quote bar, apply section,
  footer
- `pre-application.css` → page head, form layout, form card, Zoho slot, sidebar

`index.html` loads only `styles.css`. `pre-application.html` loads **both**, so
the duplicated base is applied twice.

> **Gotcha — load order.** `pre-application.html` currently loads
> `pre-application.css` _first_, then `styles.css`. Because the two files share
> identical selectors at identical specificity, **`styles.css` wins every
> conflict.** So if you change a token or a button style in
> `pre-application.css`, nothing happens on that page — the same rule in
> `styles.css` overrides it.
>
> Practical rule while this stands: **make base-layer edits in `styles.css`.**
> Tokens, buttons, header, announcement bar, footer. Only touch
> `pre-application.css` for rules unique to the form page. See §6 for the
> cleanup that removes this trap.

### Design tokens

Everything visual comes from the `:root` block at the top of `styles.css`.
Change a value there and it changes across the site. Never hard-code a hex
further down — add a token instead.

| Token                                                  | Used for                                               |
| ------------------------------------------------------ | ------------------------------------------------------ |
| `--orange` `#f16522`                                   | Primary brand — logo, buttons, prices, eyebrows, icons |
| `--orange-dark`                                        | Solid-button hover only                                |
| `--orange-tint` / `--orange-tint-2` / `--orange-line`  | Icon badges, quote bar, tinted borders                 |
| `--violet` `#4c1d95`                                   | Announcement bar                                       |
| `--violet-accent`                                      | Deposit figures, "Step one" pill, focus rings          |
| `--ink` / `--ink-2` / `--muted` / `--muted-2`          | Text, darkest to faintest                              |
| `--line` / `--surface` / `--surface-2` / `--footer-bg` | Borders and panel backgrounds                          |

Type: **Poppins** (`--display`) for headings, buttons, logo, numbers; **Inter**
(`--body`) for copy. Both from one Google Fonts request, with system fallbacks.

---

## 2. The Zoho form

This is the whole reason `pre-application.html` exists.

**Location:** `pre-application.html`, inside `<div id="zoho-form-embed">`.

```html
<div id="zoho-form-embed">
  <iframe
    aria-label="Pre-Application - Rent to Buy"
    src="https://forms.zohopublic.com.au/autoshare/form/PreapplicationInternationalRenttoBuy/formperma/otz0_BaAhbszRAGZyQAnv-SO26W8uWs7nav88Y6A_OU?zf_rszfm=1"
    title="Pre-Application - Rent to Buy"
    loading="lazy"
  >
  </iframe>
</div>
```

### Replacing the form

Get the new embed from Zoho (**Share → Embed**) and swap the `src` URL only.
Keep the wrapping `<div id="zoho-form-embed">`, the `aria-label`, and the
`title` — screen readers need them. Drop Zoho's inline `style` and `frameborder`
attributes; the CSS handles sizing.

### Height — the thing that breaks

An iframe cannot grow to fit its content. Height is set in
`pre-application.css`:

```css
#zoho-form-embed iframe {
  height: 1100px;
}
```

**After any change to the form's fields, reload the page and check the bottom of
the form isn't clipped.** Adding two or three fields is enough to cut off the
submit button, and there's no error — it just silently disappears below the
iframe edge. Adjust the number until there's minimal dead space.

The `?zf_rszfm=1` on the URL is Zoho's auto-resize flag: it makes the form
_broadcast_ its height via `postMessage`, but **nothing on this page is
listening**, so it currently does nothing. To get true auto-height, grab the
companion resize `<script>` from Zoho's embed panel and paste it directly after
the iframe — then the fixed height can go.

### After any form change, test

1. Submit a real test entry and confirm it lands in Zoho.
2. Trigger a validation error (submit empty) — errors expand the form and are
   the most common cause of clipping.
3. Confirm the post-submit confirmation message is visible without scrolling
   inside the iframe.
4. Check on a real phone.

### Legal copy

The `.form-foot` paragraph under the form ("does not guarantee approval and does
not affect your credit score") is placeholder wording. **Get it confirmed by
whoever owns compliance.**

---

## 3. The Expertease chatbot widget

Loaded in the `<head>` of **both** pages:

```html
<script
  type="module"
  src="https://desaiprodstorage.blob.core.windows.net/botscripts/bots/scripts/daryl/exabot_widget_a7e89359-5c15-4bea-838a-69c8a6be06f3.js"
></script>
```

Notes for whoever maintains this:

- **It's duplicated.** Update the URL in both files or the widget version drifts
  between pages.
- The GUID in the filename is the bot instance. Swapping bots means a new script
  URL from Expertease, not a config change here.
- It's third-party and hosted on Azure Blob Storage. If it 404s or the
  container's access policy changes, the widget silently vanishes — nothing in
  this site's code will tell you. Worth eyeballing after every deploy.
- The widget injects its own floating button, usually bottom-right. If it ever
  collides with the footer artwork, that's the thing to adjust.
- `type="module"` means it's deferred by default, so it won't block render.

---

## 4. Assets

Vehicle photos and requirement cards are wired up two different ways — know
which before you edit:

**In the HTML** (`<img src>`): logo, all icons, the three vehicle photos, the
footer 4WD.

**In `styles.css`** (`background-image`): the hero background and the three
requirement thumbnails (`#abn-card`, `#driver-card`, `#income-card`). To change
one of these, edit the CSS rule, not the HTML — the divs are deliberately empty.

> **The hero vehicle is baked into `hero-bg.png`.** That's why `.hero-grid` is a
> two-column grid with only one child in the markup — the empty right column
> reserves space so the headline doesn't sit on top of the car. If you replace
> `hero-bg.png` with a differently-composed image, re-check `.hero-copy`'s
> `max-width` and the grid ratio.

> **Rename the footer image.**
> `2025 Toyota LandCruiser Prado Altitude 4×4 – Previously Owned.png` contains
> spaces, a `×` multiplication sign, and an `–` en dash. Some servers and CDNs
> will fail to serve it or mangle the URL encoding. Rename to something like
> `footer-prado.png` and update both HTML files. Same advice for
> `31-1024x1024.png`, which is unguessable — `corolla-hybrid.png` saves the next
> person a search.

Add `loading="lazy"` to images below the hero if you touch them.

---

## 5. Routine content updates

**Vehicle prices, deposits, terms** — hard-coded in the three
`<article class="vcard">` blocks in `index.html`. There's no data layer, so each
is a manual edit. Inside a card, `.amount` renders large and orange, `.unit` is
the small dark text beside it — keep that split. Don't forget the
`*based on a 5-year term contract` line when a price changes.

**Adding a fourth vehicle** — `.vehicles` is `repeat(3, 1fr)`. Bump it to 4 and
add a `repeat(2, 1fr)` step in the 1000px media query, or the cards get too
narrow.

**Contact details** — phone and email appear in both footers, the `index.html`
apply card, and the `pre-application.html` sidebar. Search for `478 780 897` and
`sales@autoownit.com.au` and update every hit, including the `wa.me` and
`mailto:` hrefs.

**Header, footer, announcement bar** — duplicated markup across both pages. Edit
one, edit the other. This is the most likely thing to drift.

**When the main site comes back** — delete the `<div class="announce">` block
from both files.

**Uppercasing is CSS's job.** `.eyebrow`, `.pill`, and `.fcol-title` transform
to uppercase in the stylesheet, so write sentence case in the HTML.

---

## 6. Known issues worth cleaning up

Nothing here is breaking the site today. All of it will cost someone an hour
later.

- **Duplicated base CSS.** Extract the shared base (tokens, reset, `.btn`,
  `.announce`, `.site-header`, `.logo`, footer) into a `css/base.css`, strip it
  from both files, and load `base.css` → page CSS in that order. Removes the
  load-order trap in §1 entirely.
- **`#zoho-form-embed iframe` is declared twice** in `pre-application.css` —
  once with `height: 1100px`, and again further down without a height. The
  height still applies because the second block doesn't redeclare it, but the
  next person to change the height will edit one block and not understand why
  nothing moved. Merge them.
- **Dead CSS.** The `.form-placeholder` rules (~40 lines) are left over from
  before the real form was embedded. The markup is gone; delete the CSS.
- **Typo:** `<h3>What you' ll need</h3>` in `pre-application.html` — stray space
  inside the contraction.
- **Contradictory copy.** `index.html`'s apply card and `pre-application.html`'s
  page head both promise a call back _within one business day_, but the numbered
  steps on both pages say an outcome _in up to 3 business days_. Pick one and
  make all four places agree.
- **Missing `<head>` basics.** No `meta description`, no favicon, no Open Graph
  tags on either page. Links shared to WhatsApp — the main CTA channel — will
  preview as a bare URL.
- **`target="_blank"` on `mailto:` links** in both footers does nothing.
  Harmless, but remove it.

---

## 7. Don't regress these

- **No build step.** Don't add npm, a bundler, or a CSS framework for two pages.
  If it outgrows that, moving to Astro or Eleventy is a rewrite decision, not a
  maintenance one.
- **Flat specificity.** Single-class selectors in source order. The only IDs are
  `#zoho-form-embed` (embed hook) and the three `#*-card` thumbnails.
- **Test the three breakpoints:** 1000px (hero and sidebar stack, footer to
  2-up), 760px (cards to single column, stats to 2-up), 560px (footer to 1-up,
  full-width buttons, footer art hidden). The vehicle row and the footer break
  first.
- **Accessibility floor:** visible `:focus-visible` rings — never
  `outline: none`; `prefers-reduced-motion` kills all transitions, so anything
  animated must respect it; decorative SVGs keep `aria-hidden="true"`; one `h1`
  per page, no skipped heading levels.
