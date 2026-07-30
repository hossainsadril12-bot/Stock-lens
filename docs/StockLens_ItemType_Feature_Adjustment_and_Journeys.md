# StockLens — Feature Adjustment by Item Type, with User Journeys

**Module:** VantaTrack StockLens (Inventory)
**Source of truth:** StockLens PRD v3.0 (July 2026)
**Also draws on:** StockLens UX Research & Feature Analysis, and the Mobbin market review
**Stage:** Define — before wireframes
**Status:** Working document (proposals are marked as proposals)

---

## How to read this document

Read it top to bottom. Each part builds on the one before.

1. The one big idea
2. What the PRD actually says (verified)
3. The problem in plain words
4. The solution: the item-type gate
5. Features adjusted per item type (the main table)
6. Screens the PRD is missing per item type
7. A note on reports
8. User journey stories
9. Design principles
10. Open questions for the product owner
11. What is a proposal vs what is in the PRD

A short rule for the whole document: every idea is followed by a real-life example, so it is easy to picture.

---

## 1. The one big idea

The database already knows the truth. Most problems happen because the **screen does not say what the system knows.**

Example: the system knows a flat has 150 units available, but the screen shows 200 because it counted reserved stock too. The agent promises 200 to a customer. The promise breaks. The system was right; the screen misled the person.

So the design job is not "add more features." It is "make the system speak clearly to the right person."

---

## 2. What the PRD actually says (verified)

The PRD does **not** use the word "industry." It lists **five item types** (Section 3):

1. Real Estate Units
2. Physical Products & Stock
3. Equipment & Assets
4. Digital & Service Items
5. Kits & Bundles

Two exact lines matter most:

- Line 64: "StockLens handles five categories of inventory. Every org can use any or all categories."
- Line 116: "Category (user-defined: e.g. 'Residential', 'Electronics', 'Software')."

Two things follow from this:

- A single company can use **more than one** item type at the same time. Example: a developer sells flats (real estate) and also lends out drills (equipment).
- The PRD word "Category" is a small user tag on an item. It is **not** the item type. We must not reuse the word "Category" for the item-type gate, or the two will clash.

Note on wording: earlier drafts called the item types "industries." That was a plain-language label, not a PRD term. In this document, "item type" is the correct word, and it allows a business to pick several — which fits the PRD's "any or all" rule.

---

## 3. The problem in plain words

The PRD ships **one fixed sidebar of 14 menu items for every business** (Section 10). It is the same for a pharmacy, a real-estate agency, and a software reseller.

That sidebar was written for one item type: Physical Products. It is full of warehouse words — Movements, Transfers, Cycle Counts, Purchase Orders, Suppliers.

Example: a real-estate agency opens the app and sees "Cycle Counts," "Reorder Point," and "Expiry Date." None of it fits a flat. The owner thinks the software is not made for him, and leaves before he ever adds a unit.

So the raw 14-item list causes confusion for three of the five item types.

---

## 4. The solution: the item-type gate

Ask one simple question at setup, then reshape the module to fit the answer.

How it works:

1. The owner logs in the first time.
2. The module asks: **"What do you track?"** and shows the five item types as plain choices.
3. The owner ticks **one or more** (the PRD allows any or all).
4. The module instantly hides, renames, and adds screens to fit only the ticked types.
5. Everyone else in that company from then on sees the tailored version.

Example: the owner ticks only "Real Estate." The words "warehouse," "reorder," and "expiry" never appear again. The word "unit" appears everywhere. The confusion is gone before it starts.

Safety rule: default to a clean single choice, but always allow more than one, because the PRD supports mixing.

This is a **proposal** — the PRD does not contain an item-type gate today.

---

## 5. Features adjusted per item type (main table)

The 14 PRD menu items, adjusted for each item type.

Legend: **Show** = keep as is. **Rename** = keep but change the word. **Hide** = remove from the menu.

| # | PRD menu | Real Estate | Physical Products | Equipment | Digital / Services |
|---|---|---|---|---|---|
| 1 | Dashboard | Show | Show | Show | Show |
| 2 | Items | Rename: Units | Show | Rename: Assets | Rename: Licences |
| 3 | Groups | Rename: Buildings | Show | Rename: Pools | Hide |
| 4 | Locations | Hide (showrooms optional) | Show | Show (optional) | Hide |
| 5 | Movements | Hide | Show | Hide | Hide |
| 6 | Transfers | Hide | Show | Show (optional) | Hide |
| 7 | Returns | Hide | Show | Hide | Hide |
| 8 | Cycle Counts | Hide | Show | Hide | Hide |
| 9 | Purchase Orders | Hide | Show | Show (optional) | Hide |
| 10 | Suppliers | Hide | Show | Show (optional) | Rename: Vendors |
| 11 | Reports | Show (real-estate set) | Show (all 8) | Show (asset set) | Show (expiry/usage set) |
| 12 | Import | Show | Show | Show | Show |
| 13 | Settings | Show | Show | Show | Show |
| 14 | Trash | Show | Show | Show | Show |

How to read it: Physical Products is almost all "Show," because the 14 list was built for it. Real Estate and Digital are mostly "Hide," because most of the warehouse machinery does not fit them.

Example of the shrink: a real-estate agency's sidebar drops from 14 items to about six — Dashboard, Units, Buildings, Reports, Import, Settings (plus Trash and the new Reservations screen below). Much less to fear on day one.

Note on Kits & Bundles: this is not its own sidebar. It rides on top of the others. A kit is a package built from other items — for example, a furnished flat that combines a real-estate unit with equipment. So Kits appears as a capability inside whichever types are turned on, not as a separate menu.

---

## 6. Screens the PRD is missing per item type

The 14 list is not only too big for some item types — it is also **incomplete** for them. These screens do not exist in the PRD and must be added.

| Item type | Screen to add | Why it is needed | Where the PRD hides it today |
|---|---|---|---|
| Real Estate | Reservations (hold a unit for a buyer, with a timer) | The core sales action. A flat is reserved, then sold. | Only a "reserved" movement type and a CRM action. No screen. (Sections 6.3, 7.1) |
| Equipment | Check-out / Check-in | "Who has it" is the product's own headline promise. | Only an "assigned_to" field. No flow. (Section 6.1) |
| Digital / Services | Seat assignment and Renewal / Expiry | Give a licence seat to a person; warn before a subscription dies. | Only "seats" and "expiry_date" fields. No screen. (Section 3.4) |

Example: in the whole PRD, reserving a flat is only a database movement type. A sales agent has no button and no screen for it. That screen must be designed.

---

## 7. A note on reports

The eight reports (Section 7) are also not adjusted per item type. Six of them — Low Stock, Supplier Performance, Stock Aging, ABC Classification, Cycle Count Variance, Movement History — are pure warehouse reports.

Example: a real-estate agency would only use the Item Sales report and maybe Stock Valuation. Showing it the ABC Classification report is noise. So the Reports section also needs per-item-type filtering, not just the sidebar.

---

## 8. User journey stories

These stories show the solution working. No emoji; feelings are written in plain words.

### Journey A — Owner's first login and the item-type gate

Who: the business owner, first time in the app, sitting at a laptop.

| Step | What the owner does | What the owner thinks | Feeling |
|---|---|---|---|
| 1 | Logs in for the first time | "Let me see what this is." | curious |
| 2 | The module asks: "What do you track?" with five simple choices | "It is asking about me first. Good." | relaxed |
| 3 | Ticks "Real Estate" only | "That is my business." | confident |
| 4 | The module reshapes. Warehouse words disappear. Only real-estate words remain. | "This feels made for flats, not a grocery shop." | trust growing |
| 5 | Sees a near-empty screen with one clear next step: "Add your first building" | "I know what to do next." | in control |
| 6 | Imports 60 flats from a spreadsheet | "That saved me hours." | relieved |

What this shows: the item-type gate (Part 4) removed everything that did not fit flats, so confusion never happened.

### Journey B — A sales agent books a flat, and a same-second conflict is handled

Who: a sales agent, in the office, a buyer sitting across the desk. Another agent is in a different room with a different buyer who wants the same flat.

| Step | What the agent does | What the agent thinks | Feeling |
|---|---|---|---|
| 1 | Opens the buyer's record | "Is 12B still free?" | calm |
| 2 | The screen shows a live status: "12B — Available" | "Good, it is free." | relieved |
| 3 | Clicks "Reserve for this buyer" | "Locking it now." | confident |
| 4 | A 48-hour hold timer starts on 12B | "No one can take it now." | secure |
| 5 | The other agent clicks Reserve on 12B the same second and is blocked | (she never sees this — the system handled it) | protected |
| 6 | The other agent sees: "12B was just reserved. Here are 3 free flats on the same floor." | (the other agent) "I can offer 13B instead." | recovering |

What this shows: one true availability number, a reserve action that locks instantly, and a conflict recovery that offers alternatives instead of a cold error.

### Journey C — The manager's morning dashboard

Who: the sales manager (in a small firm, the owner), at a desktop, first thing in the morning.

| Step | What the manager does | What the manager thinks | Feeling |
|---|---|---|---|
| 1 | Opens the dashboard | "What needs me today?" | calm |
| 2 | The top zone shows a short list titled "Needs you today" | "The problems are right here." | relieved |
| 3 | Sees "2 reservations expiring in 4 hours" | "If the buyers do not pay, I lose those holds." | alert |
| 4 | Sees "Agent requests a 5% discount on 12B — waiting for you" | "I can approve that right here." | in control |
| 5 | Approves it without leaving the dashboard | "Done. The agent can close." | satisfied |
| 6 | Sees "Rio Tower — only 3 flats left" | "Nearly sold out. Push the price up." | thinking |
| 7 | Glances at the bottom zone: total units, sold this month | "Numbers look healthy." | content |

What this shows: two zones (urgent on top, statistics below), each urgent item actionable in place, a timer on every hold, and a scarcity signal.

---

## 9. Design principles

Five rules. Test every screen against all five.

1. Show the number the person can act on. Available first, everywhere.
2. Never end a task on a dead end. Every action ends with a clear next step.
3. The second time must be faster. Remember the last values; offer import when bulk work is implied.
4. Urgent beats complete on any glance screen. The dashboard leads with what needs a person.
5. Say what happened, even for invisible work. Confirm saves, syncs, and holds.

Bonus test for every form field: is the person typing something the system already knows? If yes, it is a design failure.

---

## 10. Open questions for the product owner

1. Do we approve the item-type gate as new scope? It is not in the PRD today.
2. Do we approve the three missing screens — Reservations, Check-out / Check-in, Seat assignment?
3. For a business that mixes item types (for example real estate plus equipment), do we show two tailored worlds, or one merged menu?
4. Should the Reports section filter by item type, so each business sees only its relevant reports?
5. The word "Category" already exists in the PRD as an item tag. Agreed that the item-type gate uses a different word to avoid a clash?

---

## 11. What is a proposal vs what is in the PRD

To keep everyone honest:

**In the PRD:**
- Five item types (real estate, physical, equipment, digital, kits).
- One company may use any or all of them.
- A fixed sidebar of 14 menu items for everyone.
- "Category" as a small user-defined tag on an item.

**Our proposals (not yet in the PRD):**
- The item-type gate ("What do you track?") at setup.
- Hiding, renaming, and adding menu items per item type.
- The three missing screens: Reservations, Check-out / Check-in, Seat assignment.
- Per-item-type filtering of the Reports section.

---

*End of document*
