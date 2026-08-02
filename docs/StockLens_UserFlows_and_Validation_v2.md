# StockLens — User Flows and Validation (v2, corrected)

**Module:** VantaTrack StockLens (Inventory)
**Source of truth:** StockLens PRD v3.0 (July 2026)
**Built on:** StockLens UX Research & Feature Analysis, and the Mobbin market review
**Stage:** Ideate — user flows (after Empathize and Define)
**Status:** Working document. Proposals are marked as proposals.
**Supersedes:** `StockLens_UserFlows_and_Validation.md` (v1). This version fixes five mistakes found in an audit against the PRD — see "Corrections in this version" below.

---

## Corrections in this version (read this first)

v1 had five mistakes. All are fixed here.

| # | v1 mistake | The truth (with PRD proof) | Fix in v2 |
|---|---|---|---|
| 1 | "Scanning and offline are mobile concerns." | Scanning works on desktop too — browser camera and USB/Bluetooth scanner (PRD 6.7) — and is **Phase 1** in the PRD. Offline is a network condition and is **not in the PRD at all**. | Reworded. Scanning is a real Phase 1 feature, just sequenced after the dashboard slice. Offline is a proposal. |
| 2 | Real-estate reserve flow shown as buildable in the first ("dashboard-first") slice. | Reserve, sold sync, and the conflict all run through StockLens ↔ Pulse (PRD 7.1), which is **Phase 3**. Phase 1 is core inventory only, "no cross-product connections yet." | Added a Phase alignment section. Real-estate reserve/conflict marked Phase 3. |
| 3 | Manager "approves a 5% discount" on the dashboard. | StockLens has only two approval types: PO approval (6.6) and transfer approval (6.12). Both are hidden for real estate. Discount/deal approvals live in Pulse, not StockLens. | Removed. Real-estate deal approvals are noted as Pulse, not StockLens. |
| 4 | "Available-first number" for real estate. | A flat is a status (available / reserved / sold), not a quantity (PRD 6.1 status table). Available-first as a big number is a physical-stock pattern. | Split: real estate shows a **status**; physical shows a **number**. |
| 5 | 60 flats imported via CSV shown as a clean happy path. | The research doc (A4) deferred bulk real-estate creation and logged a risk: property agents may not manage CSV (also PRD open question 9). | Flagged as the deferred, risk-logged path. |

---

## The Figma user flow board

All four user flow diagrams live on one FigJam board.

**Open the board:** https://www.figma.com/board/mKE9wm2SlpnCd3s5KrIN9m

| Flow on the board | Title banner | PRD item_type |
|---|---|---|
| Real Estate | REAL ESTATE — User Flow | real_estate |
| Physical Products | PHYSICAL PRODUCTS — User Flow | physical |
| Equipment and Assets | EQUIPMENT & ASSETS — User Flow | equipment |
| Digital and Services | DIGITAL & SERVICES — User Flow | digital |

**Note:** the board diagrams still show the v1 real-estate dashboard (with the invented "approve discount" and a hold timer as if native). They need a follow-up edit to match this v2 text. Flagged, not yet applied.

---

## 1. The one big idea

The database already knows the truth. Most problems happen because the **screen does not say what the system knows.**

Example: the system knows a flat is reserved, but the screen still shows it as free. An agent promises it to a buyer. The promise breaks. The system was right; the screen misled the person.

So the design brief is not "add features." It is "make the system speak."

---

## 2. Why we split flows by item type

The PRD lists five item types (Section 3): real estate, physical, equipment, digital, kits. It also says (line 64): "Every org can use any or all categories."

But the PRD ships **one fixed 14-item sidebar for everyone** (Section 10), written for physical products. For the other item types it is confusing.

Our proposal: an **item-type gate** at setup. It asks "What do you track?", then hides, renames, and adds screens to fit only the chosen types. The user may tick more than one, because the PRD allows any or all.

Kits is not a separate flow. It is an add-on that rides on top of the others (for example, a furnished flat mixes real estate and equipment).

---

## 3. Phase alignment — what is buildable when (new, important)

This is the section v1 was missing. It maps each flow to the PRD build phases so no one expects Phase 3 work inside a Phase 1 slice.

| PRD phase | What it gives | Which of our flow parts live here |
|---|---|---|
| Phase 1 — Core Inventory | Item CRUD (all 5 types), groups, locations, status, barcode/QR generation and scanning, CSV import, basic dashboard. No cross-product. | Item-type gate; add buildings/items; unit or item list with status; basic dashboard; scanning. |
| Phase 2 — Stock Operations | Movements, multi-location, low-stock alerts, suppliers, POs (create/send/receive), stock reports. | Physical flow: receive delivery (partial receipt), stock in/out, low-stock → create PO. |
| Phase 3 — Cross-Product | StockLens ↔ Pulse (reserve, sold, availability gating), ↔ Spend, ↔ Works. | Real estate flow: reserve a unit, sold sync, the same-second conflict, holds. |
| Phase 4 — Advanced | Batch/serial, kitting, forecast reorder, transfer orders, returns, cycle counts, custom fields, audit log. | Equipment serial/service depth; batch/expiry; count mode. |

**Key consequence:** the real-estate reserve-and-conflict experience is **Phase 3**, because it needs Pulse. In **Phase 1**, a real-estate org can only add buildings and units, see the unit list with a status, and view a basic dashboard. Reserving through the app comes later. This must be clear to the product owner before wireframes.

---

## 4. Each user flow in plain words

All four flows share the same shape:

> Open app → login check → item-type gate → setup → dashboard hub → the core action → status lifecycle

### 4.1 Real Estate (item_type: real_estate)

- **Phase 1 part — setup and view:** item-type gate → tick Real Estate → sidebar becomes Units, Buildings, Reservations (proposed), Reports, Import, Settings → add first building → add units. Bulk import of many flats via CSV is the **deferred, risk-logged** path (research A4): property agents may not handle CSV, so this needs validation or a simpler bulk tool.
- **Phase 1 part — status, not a number:** a unit shows one **status** big — Available, Reserved, or Sold. (Real estate has no quantity; the "available-first number" pattern is for physical stock only.)
- **Phase 3 part — reserve a unit (needs Pulse):** open unit → if Available, reserve for a buyer → the unit becomes Reserved. A 48-hour hold timer is **our proposal** (the PRD reservation is a movement type driven by a Pulse deal, with no timer).
- **Phase 3 part — conflict:** two agents reserve the same unit at once. First wins. Loser sees "just reserved" plus nearby alternatives — never a cold error. This is the double-selling problem the PRD names in Section 2.
- **Approvals note:** a real-estate manager's deal approvals (discounts, deal sign-off) live in **Pulse (the CRM), not StockLens.** StockLens itself only approves POs and transfers, which are hidden for real estate. So a pure StockLens real-estate dashboard has no approvals of its own.
- **Lifecycle:** Available → Reserved → Sold (deal Won in Pulse). Hold released or deal Lost returns it to Available.

### 4.2 Physical Products (item_type: physical)

- **Setup (Phase 1):** tick Physical Products → full warehouse menus stay on → create a location → import or add items → set reorder point.
- **Dashboard (Phase 1/2):** low stock alerts, pending POs, stock value.
- **Receive a delivery (Phase 2):** from a low-stock alert, one-click Create PO → receive delivery → "How many arrived?" (partial receipt is the default) → scan items → confirm → stock in recorded.
- **Available-first as a number (Phase 1):** the item list shows a large Available count per item (the Shopify pattern the review validated).
- **Stock out (Phase 2):** search item → open → record stock out → the system checks the reorder point.
- **Lifecycle:** In stock → Low stock → Out of stock; reorder returns it to In stock.

Example: 18 boxes arrive, not 20. The form asks "how many arrived?" instead of forcing "confirm 20."

### 4.3 Equipment and Assets (item_type: equipment)

- **Setup (Phase 1):** tick Equipment → sidebar becomes Assets, Pools, Check-out (proposed) → add assets with serial and condition.
- **Dashboard:** in use now, available now, overdue returns, needs service.
- **Check out / check in (proposed screen; assignment exists only as a field today, PRD 6.1):** open asset → if Available, check out to a person or project → In use. Check in → Available.
- **Maintenance (Phase 4 depth):** needs service → Under maintenance → service done → Available.
- **Lifecycle:** Available, In use, Under maintenance, Retired (PRD 6.1).

### 4.4 Digital and Services (item_type: digital)

- **Setup (Phase 1):** tick Digital → sidebar becomes Licences, Vendors → add licences with key, seats, expiry (fields from PRD 6.1 attributes).
- **Dashboard:** expiring soon, seats used vs free.
- **Seat assignment and renewal (proposed screens; only fields exist today):** open licence → if a seat is free, assign it to a person → Assigned. Renew before expiry keeps it Active.
- **Lifecycle:** Active, Assigned, Expired, Cancelled (PRD 6.1).

---

## 5. Validation — checked against the Mobbin review

| Decision | Review verdict | In our flows | Correct now? |
|---|---|---|---|
| Available-first (S3) | Validated | Physical: big Available number. Real estate: big status. | Yes — split by type |
| Multi-field, typo-tolerant search (S2) | Validated | Search step in each flow | Yes (spec at wireframe) |
| Question-first onboarding (S1) | Validated | The item-type gate is the first screen | Yes |
| Two-zone dashboard (Problem 6) | Validated | Top "Needs you today" + bottom stats | Yes |
| Setup progress, not empty widgets | Validated | Single dashboard + setup banner | Yes |
| One-tap scan, continuous, manual fallback (S9) | Validated in pieces | Physical/Equipment scan (Phase 1 feature) | Yes — not "mobile-only" |
| "Not found → create it" (protect it) | Refine | Keep at scan wireframe | Yes |
| Task ends with confirmation + next step (S4) | Keep | Confirmation steps | Yes |
| Viewer sees hidden, not disabled | Sound | Design principle | Yes |
| Act inline on the dashboard (differentiator) | Protect | Physical: create PO / reorder inline. Real estate: reserve-related actions arrive with Pulse in Phase 3. | Corrected — no invented discount approval |

---

## 6. The three hard gaps and their status

| Hard gap | Status | Note |
|---|---|---|
| Same-second conflict (two agents, one unit) | Designed (Phase 3) | Needs Pulse; lives in the real-estate flow |
| Partial or over delivery | Designed (Phase 2) | Physical flow "How many arrived?" |
| Offline / signal loss | Not designed | A network condition, not in the PRD; a proposal for later |

---

## 7. Design principles

1. Show the thing the person can act on — a number for stock, a status for a unit.
2. Never end a task on a dead end. Every action ends with a clear next step.
3. The second time must be faster. Remember values; offer import when bulk work is implied.
4. Urgent beats complete on any glance screen.
5. Say what happened, even for invisible work.

Bonus test for every field: is the person typing something the system already knows? If yes, it is a design failure.

---

## 8. What we still owe before wireframes

1. **User validation.** 3 to 5 interviews with real inventory managers (research doc and review both recommend this; not done).
2. **B2B benchmark** for partial-delivery and conflict flows (Zoho, Cin7, inFlow, Sortly).
3. **Phase-1 reality check with the product owner:** confirm that a real-estate Phase 1 is a catalogue of units with status only, and that reserve/conflict wait for Phase 3 (Pulse).
4. **Update the Figma board** to match this v2 (remove the invented discount approval; label the real-estate reserve parts as Phase 3).
5. **Offline path** — add when and if it is approved.

---

## 9. What is a proposal vs what is in the PRD

**In the PRD:** five item types; any-or-all use; a fixed 14-item sidebar; "Category" as a small item tag; scanning in Phase 1; PO and transfer approvals; the Pulse deal-to-unit lifecycle in Phase 3.

**Our proposals (not in the PRD):** the item-type gate; hide/rename/add per item type; the three added screens (Reservations, Check-out/Check-in, Seat assignment); per-item-type report filtering; the 48-hour reservation hold timer; the offline sync queue.

---

*End of document*
