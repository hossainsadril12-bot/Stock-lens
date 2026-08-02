# Design

> Stage: **wireframe / structure pass.** Dark neutral base, no brand color yet. Layout, hierarchy, components, and real demo data land first; the real palette drops in a later `colorize` pass. Status is carried by label + icon + tonal value, not hue, until then.

## Theme

Dark neutral app shell, matching the two inspiration boards (Assini, Tickety): deep near-black canvas, slightly raised panel surfaces, rounded cards, a fixed left sidebar and a top bar. Neutrals only for now (chroma ≈ 0). One reserved accent slot (`--accent`) is a placeholder neutral today and becomes the brand color in the color pass.

## Color palette

OKLCH, dark theme, neutral ramp (all chroma ~0):

```
--bg:        oklch(0.17 0 0)   /* app canvas, near-black */
--surface-1: oklch(0.21 0 0)   /* sidebar / top bar */
--surface-2: oklch(0.24 0 0)   /* cards / panels */
--surface-3: oklch(0.28 0 0)   /* raised: inputs, hover rows, popovers */
--border:    oklch(0.32 0 0)   /* hairline dividers, card edges */
--ink:       oklch(0.97 0 0)   /* primary text, big numbers */
--ink-2:     oklch(0.78 0 0)   /* secondary text, labels */
--ink-3:     oklch(0.60 0 0)   /* muted text — still ≥4.5:1 on --bg */
--accent:    oklch(0.80 0 0)   /* PLACEHOLDER — becomes brand color later; used for primary action + current selection only */
--accent-ink:oklch(0.17 0 0)   /* text on accent fill */
```

Semantic status (neutral now, hue in the color pass): each status renders as a pill with a **label + dot/icon + tonal weight**, so it reads without color — `success/positive`, `warning`, `danger`, `info`, `neutral`. Reserve `--ok --warn --danger --info` tokens, currently mapped to distinguishable neutral values.

Light theme is a later toggle (both inspirations show one); tokens are authored so a `[data-theme="light"]` override can flip them.

## Typography

- **One family: Inter** (variable), `system-ui` fallback. No display/body pairing — product UI.
- **Fixed rem scale, not fluid** (product rule): 0.75 / 0.8125 / 0.875 / 1 / 1.125 / 1.25 / 1.5 / 2 / 2.5rem. Ratio ~1.2.
- Big dashboard numbers use `font-variant-numeric: tabular-nums` and weight 600–700; labels 0.8125rem, `--ink-2`.
- Data/tables may run denser than 65–75ch; prose blocks respect the measure. `text-wrap: balance` on headings.

## Components (from the two inspirations)

- **App shell:** fixed left **sidebar** (logo, grouped nav — Primary / Secondary, active pill, collapse toggle to icon-rail) + **top bar** (search with ⌘K, theme toggle, notification bell with count = S7, profile). CSS grid: `[sidebar] [main]`, main = `[topbar] / [content]`.
- **Sign-in / industry chooser:** the entry screen. Lists the five PRD industries (real_estate, physical, equipment, digital, kit) as selectable cards; the chosen industry shapes the app.
- **KPI stat card:** label + big tabular number + delta (`↑12% vs previous`) + inline mini **sparkline**. Row of 3–4, `repeat(auto-fit, minmax(240px, 1fr))`.
- **Urgent band ("Needs you today"):** the dashboard's top priority zone — actionable rows (low-stock, PO to approve) each with time context ("low for 3 days") and an **inline action**. Larger and darker than the stats below.
- **Primary chart panel:** a large bar/area chart (stock over time / financial result), with a range selector.
- **List/table panel:** the adaptive **Items table** — columns + status vocabulary reshape per chosen industry (physical → quantity/available/reorder; real_estate → unit/floor/status; equipment → assigned-to/condition; digital → seats/expiry). Search + **category filter** chips, available-first number, own-location default with an All-locations toggle.
- **Status pill, delta indicator, sparkline, icon button, segmented control (tabs), dropdown/select** (use native `<dialog>`/popover or `position: fixed` to escape overflow), **skeleton** loaders.
- **Auth screens** (login / sign-up / onboarding) — centred card, brand mark, demo-role quick buttons.
- **Item detail** — available/on-hand/reserved stat trio, stock-by-location table, key-value details, scan stock in/out, movement history.
- **Item form** — smart fields that change per item type (shared `shared.module.css` form primitives).
- **Transfers** — list (route hub→sub, transport = vehicle + driver, status) + new-transfer form; **Purchase Orders** list + new; **generic table/panel/stat/kv/form primitives** in `shared.module.css` reused across POs, Transfers, Suppliers, Locations, Categories, Reports, Settings.

Every interactive component ships all states: default, hover, focus-visible, active, disabled, loading, error, selected. Empty states teach the interface.

## Layout & responsive

Structural, not fluid type (product rule):

- **≥1200px:** full sidebar (240px) + multi-column content.
- **900–1200px:** sidebar collapses to a 64px icon rail; KPI row wraps.
- **<900px:** sidebar becomes an off-canvas drawer behind a menu button; top bar condenses; KPI cards and panels stack single-column; the items table switches to a stacked card list.

Semantic z-index scale: `--z-dropdown / --z-sticky / --z-drawer-backdrop / --z-drawer / --z-modal / --z-toast / --z-tooltip`.

## Motion

150–250ms, ease-out (quart/expo). Motion conveys state only: sidebar collapse, drawer slide, row hover, tab underline, skeleton shimmer, number/chart mount. No orchestrated page-load sequence. `@media (prefers-reduced-motion: reduce)` → crossfade/instant on every one.

## Stack

Next.js (App Router) + React, TypeScript. **Drizzle ORM + SQLite** (single local file) with a seed script planting the demo company (paper warehouse + Rio Tower + a little equipment/digital) so the client sees real queried data. CSS via CSS Modules / plain CSS with the token variables above. Icons: one set (lucide-react).
