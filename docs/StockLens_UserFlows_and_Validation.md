# StockLens — User Flows and Validation

**Module:** VantaTrack StockLens (Inventory)
**Source of truth:** StockLens PRD v3.0 (July 2026)
**Built on:** StockLens UX Research & Feature Analysis, and the Mobbin market review
**Stage:** Ideate — user flows (after Empathize and Define)
**Status:** Working document. Proposals are marked as proposals.

---

## The Figma user flow board

All four user flow diagrams live on one FigJam board, each with a large title naming its item type.

**Open the board:** https://www.figma.com/board/mKE9wm2SlpnCd3s5KrIN9m

| Flow on the board | Title banner | PRD item_type |
|---|---|---|
| Real Estate | REAL ESTATE — User Flow | real_estate |
| Physical Products | PHYSICAL PRODUCTS — User Flow | physical |
| Equipment and Assets | EQUIPMENT & ASSETS — User Flow | equipment |
| Digital and Services | DIGITAL & SERVICES — User Flow | digital |

The title banner shows the exact PRD value (for example `item_type: real_estate`) so a developer can match each flow straight to the database column `stocklens.items.item_type`.

---

## How to read this document

Read it top to bottom.

1. The one big idea
2. Why we split flows by item type
3. Each user flow in plain words
4. Validation — how every decision was checked against the Mobbin review
5. The three hard gaps and their status
6. Design principles
7. Traceability — solution IDs to flow steps
8. What we still owe before wireframes
9. Status and next step

Every idea has a real-life example, so it is easy to picture. No emoji, no icons.

---

## 1. The one big idea

The database already knows the truth. Most problems happen because the **screen does not say what the system knows.**

Example: the system knows a flat has 150 units available, but the screen shows 200 because it counted reserved stock too. The agent promises 200 to a customer. The promise breaks. The system was right; the screen misled the person.

So the design brief is not "add features." It is "make the system speak."

---

## 2. Why we split flows by item type

The PRD lists five item types (Section 3): real estate, physical products, equipment, digital, and kits. It also says (line 64): "Every org can use any or all categories."

But the PRD ships **one fixed 14-item sidebar for everyone** (Section 10). That sidebar was written for physical products. For the other item types it is confusing.

Our proposal: an **item-type gate** at setup. It asks one question — "What do you track?" — then hides, renames, and adds screens to fit only the chosen types.

Example: a real-estate agency ticks Real Estate. The words "warehouse," "reorder," and "expiry" never appear. The word "unit" appears everywhere. Confusion is gone before it starts.

Because of this, each item type gets its own user flow. Kits is not a separate flow; it is an add-on that rides on top of the others (for example, a furnished flat mixes real estate and equipment).

---

## 3. Each user flow in plain words

All four flows follow the same shape, so they are easy to compare:

> Open app → login check → item-type gate → setup → dashboard hub → the core daily action → status lifecycle

### 3.1 Real Estate (item_type: real_estate)

- **Setup:** item-type gate → tick Real Estate → sidebar becomes Units, Buildings, Reservations, Reports, Import, Settings → add first building → import 60 flats or add manually → units created.
- **Dashboard hub:** top zone "Needs you today" (holds expiring, approvals waiting, tower nearly sold out); bottom zone pipeline stats (Available, Reserved, Sold). Every urgent item acts in place.
- **Core action — reserve a unit:** search 12B → open unit → status shown big → if Available, reserve for buyer → start a 48-hour hold timer → lock → confirmation.
- **Conflict branch:** two agents reserve the same unit in the same second. First wins. Loser sees "12B just reserved" plus three nearby flats — never a cold error.
- **Lifecycle:** Available → Reserved → Sold (deal Won). Hold expires or deal Lost sends it back to Available.

Example: a buyer wants 12B, the agent reserves it, a timer starts, and no one else can take it.

### 3.2 Physical Products (item_type: physical)

- **Setup:** tick Physical Products → full warehouse menus stay on → create a location → import or add items → set reorder point.
- **Dashboard hub:** low stock alerts, pending POs, stock value.
- **Core action — receive a delivery:** from a low-stock alert, one-click Create PO → receive delivery → "How many arrived?" (partial receipt is the default) → scan items → confirm receipt → stock in recorded.
- **Core action — stock out:** search item (list shows the Available number) → open item → record stock out → the system checks the reorder point.
- **Lifecycle:** In stock → Low stock → Out of stock. Reorder brings it back to In stock.

Example: 18 boxes arrive, not 20. The form asks "how many arrived?" instead of forcing "confirm 20."

### 3.3 Equipment and Assets (item_type: equipment)

- **Setup:** tick Equipment → sidebar becomes Assets, Pools, Check-out → add assets with serial and condition.
- **Dashboard hub:** in use now, available now, overdue returns, needs service.
- **Core action — check out and check in:** search asset (list shows who has it) → open asset → if Available, check out to a person or project → In use. When it comes back, check in → Available.
- **Maintenance:** needs service → send to maintenance → Under maintenance → service done → Available.
- **Lifecycle:** Available, In use, Under maintenance, Retired.

Example: the list shows "Drill 3 — with Karim," so the manager instantly knows who has it.

### 3.4 Digital and Services (item_type: digital)

- **Setup:** tick Digital → sidebar becomes Licences, Vendors → add licences with key, seats, and expiry.
- **Dashboard hub:** expiring soon, seats used vs free.
- **Core action — assign a seat:** search licence (list shows seats and expiry) → open licence → if a seat is free, assign it to a person → Assigned. Unassign frees the seat.
- **Renewal:** expiring soon → renew → Active.
- **Lifecycle:** Active, Assigned, Expired, Cancelled.

Example: a subscription is about to expire; the dashboard warns before it dies, and one renew keeps it Active.

---

## 4. Validation — checked against the Mobbin review

The Mobbin review checked our headline decisions against how the best shipping apps solve the same problem. This table maps each validated decision to where it now lives in our flows.

| Decision | Review verdict | Market reference | In our flows |
|---|---|---|---|
| Available is the primary number (S3) | Validated | Shopify shows a large right-aligned "Available" number | Physical: "Item list shows Available number"; Real Estate: status shown big |
| Multi-field, typo-tolerant search (S2) | Validated | alias search "Brand, Name, SKU" in one box | Search step in each flow (spec to add at wireframe) |
| Question-first onboarding, hide irrelevant (S1) | Validated | Linear one question at a time | The item-type gate is the first screen in all four flows |
| Two-zone dashboard, urgent on top (Problem 6) | Validated | eBay tasks block above small stat tiles | Every flow: top zone "Needs you today" + bottom zone stats |
| Setup progress, not empty widgets | Validated | eBay "Set up payouts" banner until done | Single dashboard hub, not two dashboards |
| One-tap scan, continuous, manual fallback (S9) | Validated in pieces | DICK's, HBX, UNIQLO | Physical: scan step (full detail deferred with mobile) |
| "Not found → create it" recovery | Refine — protect it | Market mostly gets this wrong | To keep at scan wireframe |
| Every task ends with a confirmation + next step (S4) | Keep — it is a principle | eBay task chips (partial) | Confirmation steps in each flow |
| Viewer sees hidden, not disabled | Sound principle | No reference | Design principle, applied at wireframe |
| Act inline on the dashboard (differentiator) | Protect it | eBay bounces you away — do not copy | Real Estate: approve, extend hold, adjust price all return to the dashboard |

Plain reading: the review validated our biggest bets, and our flows contain them. The one thing the review told us to protect — acting inline on the dashboard instead of bouncing to another screen — is built into the Real Estate flow.

---

## 5. The three hard gaps and their status

The review flagged three problems with no app to copy. They need fresh design and user testing.

| Hard gap | Status in our work | Note |
|---|---|---|
| Same-second conflict (two agents, one unit) | Designed | Real Estate flow has the "First wins / Loser sees alternatives" branch |
| Partial or over delivery ("18 arrived, not 20") | Designed | Physical flow uses "How many arrived?" as the default receive step |
| Offline or signal loss during a scan | Not designed | A mobile problem; deferred with the mobile app. Add later. |

Two of three are handled. The third is low priority now because we are dashboard-first and desktop-first; scanning and offline are mobile concerns.

---

## 6. Design principles

Five rules. Test every screen against all five.

1. Show the number the person can act on. Available first, everywhere.
2. Never end a task on a dead end. Every action ends with a clear next step.
3. The second time must be faster. Remember the last values; offer import when bulk work is implied.
4. Urgent beats complete on any glance screen. The dashboard leads with what needs a person.
5. Say what happened, even for invisible work. Confirm saves, syncs, and holds.

Bonus test for every form field: is the person typing something the system already knows? If yes, it is a design failure.

---

## 7. Traceability — solution IDs to flow steps

The research doc named twelve solutions (S1 to S12). This shows where each one appears in a flow.

| ID | Solution | Where it lives now |
|---|---|---|
| S1 | Onboarding path (question-first) | The item-type gate, all flows |
| S2 | Multi-field search | Search step, all flows |
| S3 | Available-first number | Item and unit lists |
| S4 | Success screen with next step | Confirmation steps |
| S5 | "Add another like this" | Real Estate and Physical manual-add path |
| S6 | Editable CSV preview | "Fix red rows on screen" in setup |
| S7 | Notification centre | Dashboard top zone (Phase 2) |
| S8 | Backup approver / escalation | Approvals in the dashboard (Phase 2) |
| S9 | Continuous scan | Physical scan step (mobile, later) |
| S10 | Count mode | Not in flows yet (Phase 4) |
| S11 | Equipment checkout / return | The whole Equipment flow |
| S12 | Conflict recovery with alternatives | Real Estate conflict branch |

---

## 8. What we still owe before wireframes

Honest guardrails, not stop signs:

1. **User validation.** The research doc and the review both say: do 3 to 5 interviews with real inventory managers before wireframes. We have not done this yet.
2. **B2B benchmark for the hard flows.** Compare the partial-delivery and conflict flows against real B2B tools (Zoho Inventory, Cin7, inFlow, Sortly), as the review advised.
3. **Offline path.** Add the "items waiting to sync" queue when the mobile app returns.

---

## 9. Status and next step

| Stage | Status |
|---|---|
| Empathize — personas, feature analysis, journeys | Complete (personas are assumptions until validated) |
| Define — problems, principles | Complete |
| Item-type gate and per-item-type feature adjustment | Proposed (needs product-owner approval) |
| Ideate — user flows for all four item types | Complete (this document + the Figma board) |
| Validate — 3 to 5 interviews | Recommended, not done |
| Prototype — wireframes | Next |

**Immediate next step:** review these flows with the product owner, then begin wireframes for the Real Estate flow (dashboard-first), since that is the lead CRM case.

---

## What is a proposal vs what is in the PRD

**In the PRD:** five item types; a company may use any or all; a fixed 14-item sidebar; "Category" as a small item tag.

**Our proposals (not yet in the PRD):** the item-type gate; hiding, renaming, and adding menu items per item type; the three added screens (Reservations, Check-out/Check-in, Seat assignment); per-item-type filtering of reports.

---

*End of document*
