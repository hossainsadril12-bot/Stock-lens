# Product

## Register

product

## Users

Three real roles inside one company (the demo company is **Anwar Supplies & Properties** — a paper/printing warehouse plus a 24-unit residential block, Rio Tower):

- **Anwar, 48 — Admin/Owner.** Excellent with Excel, suspicious of software. Two opposite contexts: **Setup** (once, laptop, patient, ~15 min of goodwill) and **Daily** (phone, ~40 seconds, standing, impatient). Only person who approves POs above threshold. Uses everything.
- **Rahim, 26 — Staff.** Warehouse floor, standing, one hand on a box, noisy bay, weak signal, types with typos, someone always waiting. Searches, scans, moves stock, raises reorders. Cannot delete, price, or approve.
- **Salam, 55 — Viewer.** Outside investor, read-only, twice a month, remembers nothing between visits. Wants one number then leaves.

Job to be done: **track everything the company owns and sells across five item types**, and — for the admin daily flow — answer "is anything wrong today?" in one glance and act on it without leaving the screen.

## Product Purpose

VantaTrack StockLens is the inventory operations module of the VantaTrack platform. It tracks five item types — **real_estate, physical, equipment, digital, kit** (PRD §3) — with stock derived per location from a movement ledger, batch/serial tracking, forecast reorder points, purchase orders with approval, and CSV/bulk import. The **industry is chosen during onboarding (right after sign-in)**; the app then shapes its table columns, KPIs, status vocabulary, and required fields to that industry (PRD §3.1–3.5 field/status sets). Success = a brand-new owner reaches a real win in under five minutes, and a returning owner acts on the single most urgent thing in under fifteen seconds.

**Locations & distribution.** The business runs a **main hub warehouse (Dhaka) plus sub-warehouses by city** (Chittagong, Sylhet, Khulna). Stock moves out to the branches via **the company's own transport (vehicle + driver)** — an internal *transfer*, not a supplier purchase and not a courier. This is PRD §6.12 Transfer Orders, simplified to the owner's real workflow (no approval gate, own transport).

**Roles.** Three roles gate every action, in the UI and on the server: **Admin** (everything, incl. PO approval), **Staff** (create/edit items, raise POs, dispatch/receive transfers, stock moves — no delete, no approval), **Viewer** (read-only). Viewers see hidden, not disabled, controls.

## Brand Personality

Trustworthy, plain-spoken, fast. Three words: **honest, quick, calm.** The one-line brief for the whole product: *"The system knows more than it says. Our job is to make it speak."* Every screen states the number the user can act on, ends every task with proof and a next step, and treats the messy real world (partial deliveries, typos, lost signal) as the normal case — not an error afterthought.

## Anti-references

- The **ten-equal-widgets dashboard** — everything the same size, so nothing has priority and the owner postpones. Urgent must beat complete on any glance screen.
- Generic **SaaS-cream** admin templates: warm off-white body, identical icon+heading+text card grids, tiny tracked uppercase eyebrows over every section.
- Over-decorated product chrome: gradient text, glass cards as default, side-stripe accent borders, ghost-card (1px border + wide soft shadow), over-rounded 32px+ cards.
- Showing three stock numbers side by side (on hand / reserved / available) and making the user choose — that is the exact confusion that causes a broken promise to a customer.

## Design Principles

1. **Show the number they can act on.** Available is the primary figure everywhere; on hand secondary/grey; reserved on detail only.
2. **Never end a task on a dead end.** Every save/import/receive lands on a confirmation that names the result and offers the next step.
3. **The second time must be faster.** Pre-filled "add another like this", remembered values, bulk generators/import where repetition is detected.
4. **Urgent beats complete on any glance screen.** The dashboard has a physical priority zone on top; statistics are demoted below.
5. **Say what happened invisibly.** Background work (sync, expense-to-finance, ledger writes) surfaces one calm line; offline shows a visible waiting-to-sync count.

## Accessibility & Inclusion

WCAG 2.1 AA. Body text ≥4.5:1, large text ≥3:1, placeholders held to 4.5:1 (no washed-out muted gray). Status must never be conveyed by color alone — always a label/icon too (color-blind safe), which suits the wireframe stage where color comes last. Full keyboard paths and visible focus on every interactive element. `prefers-reduced-motion` honored on every transition. Desktop-first for launch across all roles; the phone-first staff experience is a later phase (PRD/Decision 7).
