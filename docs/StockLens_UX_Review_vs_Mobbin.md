# StockLens UX Doc — Review & Mobbin Benchmark

**Reviews:** `StockLens_UX_Research_Feature_Analysis.md`
**Method:** Each major design decision in the doc is checked against how shipping
inventory / seller apps actually solve the same problem (references pulled from
Mobbin, iOS). Verdicts: ✅ validated by market · 🔧 refine · 🕳️ no reference found.
**Reviewer:** Design review (Claude)

---

## 1. Verdict in one line

The document is unusually strong — it reasons from journeys, not feature lists,
and most of its headline decisions match what the best real apps already ship.
The Mobbin scan confirms three of the four "🔴 blocks design" bets (**available-first**,
**multi-field search**, **question-first onboarding**) and the two-zone dashboard.
The gaps are exactly where the doc already flagged uncertainty: **receiving /
partial delivery**, **offline**, and the **scan failure path** — none of which the
consumer apps on Mobbin cover well, because they're warehouse-B2B problems.

---

## 2. Decision-by-decision benchmark

| Doc decision | Market reference | Verdict |
|---|---|---|
| **S3 — Available is the primary number** (Decision #3) | **Shopify Inventory** lists each variant with a single **"Available"** column header and a large right-aligned number (`99`); **Shopify Products** subtitles items **"495 available · 5 variants"** — available first, on-hand not shown at the glance level | ✅ Strongly validated. This is the exact pattern the highest-volume commerce app in the set uses. |
| **S2 — Multi-field, typo-tolerant search** (R1) | **alias** search placeholder reads **"Brand, Name, SKU"** — a single box spanning multiple fields, mirroring the doc's "name + SKU + barcode + serial + batch in one box" | ✅ Validated as a pattern. Typo-tolerance itself isn't visible in a screenshot — still a build requirement, not a layout one. |
| **S1 — Question-first onboarding, hide irrelevant features** (A1) | **Linear** onboarding asks one question at a time with an example placeholder (*"What's a project your team are working on? e.g. Q4 budget…"*); **Linear web** setup uses *"How large is your company?" / "What is your role?"* dropdowns to branch; **Expensify** / **Slack** do staged workspace creation | ✅ Validated. The "3 questions then one next step" shape is standard. Borrow Linear's *one-question-per-screen + concrete placeholder* execution. |
| **Two-zone dashboard: "Needs you today" large + stats small** (Problem 6) | **eBay Selling** puts a **"Tasks · 1"** actionable block above small stat tiles (ACTIVE/ORDERS/UNSOLD) and a **"Jump to…"** row; **Whatnot** leads with a reassurance banner (*"Nice work! Your account is in good health"*) | ✅ Validated. eBay is almost a literal implementation of the doc's dashboard. Note it still doesn't let you *act inline* — see gap below. |
| **Phase-1 dashboard shows setup progress, not empty widgets** (Journey 5) | **eBay** shows a persistent **"Set up payouts"** info banner until the setup step is done — a live "unfinished setup" nudge on the home surface | ✅ Validated. Use a dismiss-on-complete banner, not a separate onboarding checklist screen. |
| **R2 / S9 — One-tap scan, continuous mode, manual fallback** | **DICK's** puts a **"SCAN AN ITEM OR A BARCODE"** button directly in search (one tap); **HBX** offers **"Enter barcode number"** + **"Scan barcode from photo"** fallbacks; **UNIQLO** keeps a running **"Scan history"** list under the live camera (continuous-scan pattern) | ✅ Validated in pieces. No single app combines all three — StockLens would be differentiated by doing so. |
| **R2 — "Not found → create it?" recovery** | **UNIQLO** scan returns a dead-end **"Product not found → OK"** alert — the anti-pattern; **Thrive Market** does better with steady-hand guidance and shows *alternatives* on a failed match | 🔧 Market mostly gets this *wrong*. The doc's "offer to create the item" is better than every reference found — protect it. |
| **R6 — Partial receipt as the default form** (Journey 4) | No consumer app in the scan covers goods-receiving / PO check-in | 🕳️ No reference. Look outside Mobbin's consumer set (Shopify POS receiving, Sortly, Zoho Inventory, Zoho/SAP warehouse apps). |
| **S4 — Every task ends in a confirmation + next step** | Partially visible (eBay task chips, scan-history confirmation) but not as an explicit "success screen with next action" | 🔧 Weakly referenced. Keep it; it's a principle, not a screen you can copy. |
| **Viewer sees hidden, not disabled, controls** (Decision #4) | Not directly observable in the pulled flows | 🕳️ No reference. Sound principle regardless ("never show a door that is locked"). |

---

## 3. What the references add that the doc doesn't yet say

1. **List-row anatomy (from Shopify Inventory).** The doc says "available is primary,
   on-hand grey" but never specifies the *row*. Shopify's row = thumbnail · name ·
   grey sub-label (variant/color) · large right-aligned Available number. StockLens's
   equivalent sub-label is **location or group**, and the right-aligned number is
   **Available**. Specify this row now — it's "the spine" the doc itself prioritizes.

2. **Status-grouped entry list (from alias).** alias opens inventory as **counts by
   status** ("For Sale 2 / In Review 1 / Not For Sale 0") *above* the item list. For
   StockLens this maps cleanly to **Low / OK / Out** or **by location** — a way to make
   the empty/early state feel structured instead of a blank list (helps Problem 2).

3. **One-question-per-screen onboarding (from Linear).** The doc says "3 questions" but
   a single screen with 3 fields will feel like a form. Linear's execution — one big
   question, one input, a concrete example placeholder, immediate Next — is the version
   that produces the "small win before minute five" the doc is chasing.

4. **Persistent setup banner (from eBay).** Cleaner than a dashboard that morphs between
   a "Phase-1 version" and "full version." One dashboard + a dismiss-on-complete banner
   is less to build and less to explain than two dashboard variants.

---

## 4. Anti-patterns spotted in the references — things to *avoid*

- **UNIQLO "Product not found" dead-end.** Confirms the doc's Problem 8. A failed scan
  must branch (create / manual entry / retry), never a lone "OK."
- **eBay dashboard can't act inline.** eBay surfaces tasks but bounces you to another
  screen to act — exactly the friction the doc calls out in Journey 5 step 5–6. StockLens's
  differentiator is **approve / reorder from the dashboard row**. Don't lose that.
- **Equal-weight stat tiles (eBay/Whatnot).** Both show rows of same-size metric tiles.
  Fine for a *reports* screen, but the doc is right that a *glance* screen needs one
  urgent zone with visual dominance, not a grid of equals.

---

## 5. Gaps the benchmark could NOT close (flag for the PRD owner)

These three are the doc's hardest problems *and* the ones with no good consumer-app
reference — which is a signal they need dedicated design + validation, not pattern reuse:

1. **Receiving a partial/over delivery** (Journey 4) — the whole flow, incl. scan-loop,
   batch/expiry capture, and the PO→Spend sync confirmation.
2. **Offline / signal-loss during a scan run** (Q8) — "3 items waiting to sync" queue.
3. **Simultaneous-claim conflict** (X7 / S12) — two agents pass availability in the same
   second. Thrive Market hints at "alternatives on failure," but the real-estate
   double-booking case is unique to VantaTrack.

**Recommendation:** for these three, benchmark against B2B inventory tools directly
(Shopify POS receiving, Sortly, Zoho Inventory, Cin7, inFlow) rather than Mobbin's
consumer catalogue, and prioritise them in the "Validate — 3–5 interviews" step the
doc already schedules.

---

## 6. Suggested edits to the source doc

The source is immutable to me (I don't hand-edit it), so these are proposed, not applied:

- **§10 / Decision log** — add a row anatomy spec for the Items list (Shopify pattern).
- **§10 Onboarding** — note "one question per screen" as the execution standard (Linear).
- **§7 Problem 6** — add "act inline on the dashboard" as an explicit differentiator vs.
  eBay's bounce-to-screen behaviour.
- **Decision #? (new)** — one dashboard + dismiss-on-complete setup banner, replacing the
  "two dashboards (Phase-1 vs full)" split, if PM agrees.
- **Open questions** — Q8 (offline scan) and the receiving flow should be tagged as
  "no market pattern — design from scratch + validate."

---

## 7. Reference index (Mobbin, iOS)

- Shopify — Inventory detail (Available-first): https://mobbin.com/flows/dc2e69d0-3b01-43fe-8015-861d2d8ae65e
- Shopify — All products ("495 available"): https://mobbin.com/flows/9a40be1b-3642-485b-81ad-a6c582111611
- alias — Inventory ("Brand, Name, SKU" + status counts): https://mobbin.com/flows/7a072d17-0d2d-4784-b60a-6e7cd9b08dbd
- alias — Item status detail: https://mobbin.com/flows/61bccf49-10bc-4b66-8165-49bbd4d2acd7
- DICK's Sporting Goods — Scan from search: https://mobbin.com/flows/8302db41-8c68-4b9b-8c58-3a74b26bedb6
- Thrive Market — Scanning an item barcode: https://mobbin.com/flows/22667bf1-54df-4bc1-a812-fca38a0eaf8c
- HBX — Scan barcode (manual + photo fallback): https://mobbin.com/flows/9a82b32e-e55f-4009-a422-bbd24f145927
- UNIQLO — Scanning a barcode (scan history + not-found error): https://mobbin.com/flows/bbcbb1c8-77ec-4c85-880a-f3320f371e1a
- Linear Mobile — Onboarding (one-question-per-screen): https://mobbin.com/flows/628034a6-be45-450e-99d3-fb86e4571661
- Slack — Creating a workspace: https://mobbin.com/flows/5908e7aa-8a4f-4594-8b23-7c8623eb0a66
- Expensify — Creating a new workspace: https://mobbin.com/flows/47071c46-a703-4c46-9b06-55b9be710008
- eBay — Selling dashboard (Tasks + stat tiles + setup banner): https://mobbin.com/flows/8b0f54aa-a2bc-4d0c-9de2-f531719fa034
- Whatnot — Account health: https://mobbin.com/flows/d3b5b8e4-6a99-4857-93c5-0b24842c56c9

*End of review*
