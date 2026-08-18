# Hike Your Mind

Shopify theme for **Hike Your Mind** — a guided-hiking business in Italy, Latvia and Portugal.

This is not a normal merch store, and that shapes the whole codebase: **a hike is a booking, not a purchase.** Each departure is a product, reserved with a **20% deposit** taken at checkout, with the balance charged 60 days before the trip. Trip facts (dates, location, group size, itinerary) live in metafields and metaobjects. A conventional Shop section (backpacks, caps, notebooks) sits alongside it.

Built on Shopify's [Skeleton theme](https://github.com/Shopify/skeleton-theme) (v0.2.0). Design source: the `HIKE_YOUR_MIND` Figma file, six templates at 1440 desktop / 402 mobile — Home, Hikes, Detailed Hike, Create a Hike, Reviews, Shop.

---

## Getting started

### Prerequisites

- [Shopify CLI](https://shopify.dev/docs/api/shopify-cli) — the entire dev loop
- [Shopify Liquid VS Code extension](https://shopify.dev/docs/storefronts/themes/tools/shopify-liquid-vscode) — highlighting, linting, Liquid docs

There is **no `package.json`, no `node_modules`, no build step.** Clone and run.

### Run it

```bash
shopify theme dev --store <your-store>.myshopify.com
# → http://127.0.0.1:9292/
```

There is deliberately **no `shopify.theme.toml`** and no CI workflow in this repo — pass `--store` (and `--theme` when working against a specific unpublished theme) on the command line.

### Check it

```bash
shopify theme check
```

`.theme-check.yml` extends `theme-check:recommended`. Run it before every commit.

> **Theme check is not sufficient on its own.** It validates syntax and schema, not runtime behaviour. A Liquid filter-chain error in `sections/footer.liquid` passed both theme-check and the JSON schema validator and only surfaced when the page was actually rendered. **Always load the affected pages in `shopify theme dev` before committing.**

---

## Where things are

Standard Shopify layout. `templates/`, `layout/`, `config/`, `locales/`, `assets/`, `sections/`, `blocks/`, `snippets/` mean what they mean in [the theme architecture docs](https://shopify.dev/docs/storefronts/themes/architecture).

What matters here is which files are **ours** and which are still untouched Skeleton scaffolding:

### Built for this project

| File | Role |
|---|---|
| `snippets/css-variables.liquid` | **The design token layer.** Type scale, colours, spacing, motion. Start here. |
| `snippets/fonts.liquid` | `@font-face` for the two self-hosted brand faces |
| `assets/critical.css` | Reset, base type, focus ring, `.visually-hidden`, the section grid, responsive `--page-margin` |
| `sections/header.liquid` | Two-row desktop header; mobile `<details>` drawer |
| `sections/footer.liquid` | Oversized wordmark, Instagram call-out, three detail columns, copyright |
| `blocks/testimonials.liquid` | Testimonial carousel, sourced from metaobjects. Reused on 4 templates. |
| `snippets/nav-menu.liquid` | Renders a link list; underlines the active item |
| `snippets/language-selector.liquid` | LV / ENG switcher, zero JavaScript |
| `layout/theme.liquid`, `layout/password.liquid` | Font preloads, token render order |
| `config/settings_schema.json` | Brand defaults: cream `#FFF8E5` ground, black ink, 79px desktop margin, square inputs |
| `locales/en.default.json` | All user-facing strings |

### Still Skeleton scaffolding — expect to rewrite

`sections/product.liquid` (a bare `<select>` + add-to-cart), `sections/cart.liquid` (an unstyled `<table>`), `sections/collection.liquid`, `sections/blog.liquid`, `sections/article.liquid`, `sections/search.liquid`, `sections/404.liquid`, `sections/page.liquid`, `blocks/text.liquid`, `blocks/group.liquid`, `sections/hello-world.liquid`, `sections/custom-section.liquid`.

`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `LICENSE.md` and `assets/shoppy-x-ray.svg` are inherited from Skeleton upstream and do not describe this project's process.

---

## Design system

### Tokens

Everything visual reads from CSS custom properties defined once in `snippets/css-variables.liquid`, inlined into `<head>` before `critical.css`. **Do not hardcode a colour or a font size in a section.** If you need a value that isn't there, add a token.

```
--font-display--family   Spectral        headings, tagline, Instagram call-out
--font-body--family      Rethink Sans    body, navigation, UI
--font-size-display/body/ui/small/quote  + matching line height and tracking
--color-background       #FFF8E5         cream ground (theme setting)
--color-foreground       #000000         ink (theme setting)
--color-surface          #FFFAEC         cards
--color-hairline         rgba(0,0,0,.5)  0.5px rules
--page-width / --page-margin / --size-icon / --duration-* / --easing-base
```

Only the two colours a merchant may reasonably want to change and the layout measurements are wired to theme settings. The rest are brand constants and are intentionally **not** editable in the theme editor. Typography is brand-locked, so there is no `font_picker` setting.

`--page-margin` is set in `critical.css`, not in the token snippet: 30px on mobile, and the theme setting (79px) from 750px up. It lives there because `critical.css` loads *after* the inlined `<style>` block and so wins the cascade.

> Figma is essentially untokenized (four variables in the whole file). Token values were derived by **measuring the design**, not by importing variables. When you add one, confirm the value against the specific Figma node with `get_design_context` — don't eyeball it from a screenshot.

### Fonts

Both faces are Google Fonts / OFL and self-hosted from `assets/` — no font CDN, no `preconnect`:

- **Spectral** — static, weight 400 only
- **Rethink Sans** — variable, declared `font-weight: 400 700`

Each ships as two woff2 subsets, `latin` and `latin-ext`, split by `unicode-range`. **`latin-ext` carries the Latvian diacritics (ā, č, ē, ģ, ī, ķ, ļ, ņ, š, ū, ž)** — it is fetched on demand and must not be dropped. Only the `latin` subsets are preloaded in `layout/theme.liquid`.

The `HIKE YOUR MIND` wordmark is a vector (`assets/logo-wordmark.svg`, `logo-wordmark-large.svg`), not a webfont, which sidesteps licensing the condensed display face entirely. Handwritten script accents in the design should ship as SVG for the same reason.

Icons are inlined via `inline_asset_content` and normalized to `currentColor`, so they inherit text colour.

### Layout

Skeleton's `.shopify-section` grid is kept as-is: three columns (`--content-grid`), children constrained to the centre column by default, `.full-width` to break out edge-to-edge. The Hikes listing and Home hero are full-bleed; everything else is a centred column.

---

## Conventions

- **No build tooling.** CSS and JS go in per-component `{% stylesheet %}` / `{% javascript %}` tags inside the `.liquid` file that uses them. Shopify concatenates them into `compiled_assets/styles.css` and `compiled_assets/block-scripts.js`. Only genuinely global CSS belongs in `assets/critical.css`.
- **Liquid is not rendered inside `{% stylesheet %}` / `{% javascript %}`.** Pass dynamic values via inline `style="--x: {{ … }}"` and read them as custom properties.
- **Every user-facing string goes through `{{ 'key' | t }}`** and into `locales/en.default.json`. English only — Latvian comes from Translate & Adapt. Schema labels go into `locales/en.default.schema.json` as `t:` keys. Sentence case throughout.
- **LiquidDoc header on every snippet** and on any block rendered statically, documenting params with `@param` and an `@example`.
- **Prefer zero JavaScript.** The mobile drawer is `<details scroll-lock>` (the lock is CSS via `html:has(...)`), the "Read more" is `<details>` + `:has()`, the language switcher is a plain `{% form 'localization' %}`. The only custom element in the theme is `<testimonial-carousel>`.
- **Accessibility is not a later phase.** Icon-only controls carry `.visually-hidden` labels, active nav items use `aria-current`, focus is visible everywhere, motion respects `prefers-reduced-motion`.
- **Guard on empty content.** Sections that read metaobjects or optional settings must render *nothing* rather than an orphan heading — see the `{% if testimonials.size > 0 %}` wrapper in `blocks/testimonials.liquid`. The theme has to be installable before the content model exists.
- One gotcha worth knowing: a **filter chain cannot be used inline as a `t:` argument.** Resolve it with `assign` first (`sections/footer.liquid:70`).

---

## Content model

Nothing below exists in the store yet — creating these definitions in the Shopify admin is the first task of Phase 2.

**Metafields**, namespace `hike`, on the hike products, all with storefront access:

| Key | Type | Feeds |
|---|---|---|
| `start_date`, `end_date` | date | "11–18 May", duration, balance-due date |
| `country` | single-line text | listing filter, "Tuscany, Italy" |
| `location` | single-line text | spec table |
| `walking_per_day` | number | "approx. 22 km per day" |
| `group_size` | integer | "max. 12 people" |
| `what_to_expect`, `included`, `cancellation_policy` | rich text | detail-page tabs |
| `itinerary` | list of `hike_day` refs | Itinerary tab |

**Metaobjects:**

- `hike_day` — `day` (integer), `title`, `body` (rich text), `image`
- `testimonial` — `author`, `photo`, `body` (rich text), `date`, `hike` (product ref, optional) — already consumed by `blocks/testimonials.liquid`

Duration and the balance-due date are **derived in Liquid** from `start_date` / `end_date`. Never store them twice.

---

## The deposit — read this before touching the hike product page

Shopify's [deferred purchase options](https://shopify.dev/docs/storefronts/themes/pricing-payments/preorder-tbyb/add-preorder-tbyb-to-your-theme) require **five** theme-side pieces, and Skeleton has none of them:

1. Selling-plan selector on the product page (`product.selling_plan_groups`)
2. JS syncing the hidden `selling_plan` input when the variant or plan changes
3. Selling-plan display on each cart line item (`line_item.selling_plan_allocation`)
4. Amount due today in the cart, from `selling_plan.checkout_charge`
5. Selling-plan display on customer order pages

Two rules:

- The reservation-summary figures (full price / pay today / remaining balance / total) come from `selling_plan_allocation` + `checkout_charge`. **Metafields supply trip facts; Shopify supplies money.** No hand-rolled arithmetic.
- Don't bake prices into selling-plan names ("€420 deposit") — currency switching and rounding will break them. Name the plan `20% deposit` and render the euro figures from the allocation.

Isolate all of it in `snippets/hike-purchase.liquid` — the only file that should know *how* money is taken. If the purchase-options app changes, one file changes.

Shopify Payments supports Latvia, so the deferred balance charge is viable.

---

## Where the work stands

**Phase 1 — Foundation: done** (commit `07ace3f`). Tokens, self-hosted fonts, layout, header, footer, testimonials carousel, locales.

**Next up:**

| Phase | Scope |
|---|---|
| **2 — Commerce core** *(riskiest)* | Metafield + metaobject definitions; `templates/collection.hikes.json` with the country filter; `templates/product.hike.json` — gallery, reservation summary, spec table, tabs — plus the five deferred-purchase pieces above |
| **3 — Home** | Mosaic hero, the three expanding link rows, Create-a-Hike CTA, My Story |
| **4 — Shop** | Product grid with SALE badges and quick-add, product page, cart |
| **5 — Pages** | Create a Hike (native `{% form 'contact' %}`), Reviews, My Story |
| **6 — Polish** | LV translations, a11y pass, Lighthouse, a real test booking on the Bogus Gateway |

### Blocked on the merchant, not on code

1. **Purchase-options app** (Downpay or equivalent) installed, and Shopify Payments active — blocks all of Phase 2's deposit work.
2. **Search & Discovery app** installed, for the country filter.
3. **Real navigation menus.** Header and footer currently point at the store's default `main-menu` (Home / Catalog / Contact). The design's menu — Hikes / My Story / Create a Hike / Shop / Reviews — is an admin change, not a code change.
4. **Markets + Translate & Adapt** enabled. The language switcher renders nothing while the store has one language.
5. Clarification of what **"Rentals"** means commercially — a product type, or a different fulfilment flow?

### Open judgment calls

Decisions made to unblock Phase 1, any of which the team may want to overrule:

- The design uses Inter in the footer; it renders in Rethink Sans for consistency.
- The design labels the switcher "ENG"; it renders the ISO code, "EN".
- The design reads "Folow Us on Instagram"; the typo is not reproduced.
- The closing tagline renders on every template, because `footer-group` is global. The design omits it on Shop only. Clearing the setting hides it everywhere.
- The design's four near-blacks are consolidated into three tokens.

---

## Verification checklist

Before calling a phase done:

- [ ] `shopify theme check` clean
- [ ] Pages actually loaded in `shopify theme dev` — static checks miss runtime Liquid errors
- [ ] Compared side by side against the Figma frame at **1440** and **402**
- [ ] Empty states exercised: 0, 1, and many entries
- [ ] Keyboard path walked through every interactive component
- [ ] WCAG 2.1 AA contrast held on the cream ground

---

Derived from Shopify's [Skeleton theme](https://github.com/Shopify/skeleton-theme), MIT licensed — see [LICENSE.md](./LICENSE.md).
