# VantaTrack StockLens — Product Requirements Document (PRD)

**Version:** 3.0 **Date:** July 2026 **Status:** Pre-Development **Scope:** VantaTrack StockLens — Inventory Management Module **URL:** `stocklens.vantatrack.io` **App location:** `apps/stocklens/` in VantaTrack monorepo **Database schema:** `stocklens` in vantatrack-production Neon project

---

## Executive Summary

VantaTrack StockLens is the inventory management module of the VantaTrack platform. It gives agencies and SMEs a single place to track everything they own and sell — real estate units, physical products, equipment, digital items, and service packages.

StockLens is not a standalone stock counter. It is a living inventory layer connected to the rest of VantaTrack. When a deal closes in Pulse, StockLens marks the unit sold. When a project starts in Works, StockLens allocates the required stock. When stock is purchased, Spend logs the expense. Everything stays in sync automatically through one database transaction — no manual updates across multiple tools.

Version 3.0 upgrades StockLens from a stock counter into a full inventory operations system: stock is now tracked per location rather than pinned to one, batches/expiry and serial numbers are supported, reorder points are forecast from actual consumption instead of a fixed number, kits can be assembled from component stock, and every quantity or price change is auditable.

**The guiding headline:**

> _"Know exactly what you have, where it is, and who has it — at all times."_

---

## 1. Product Overview

|Parameter|Specification|
|---|---|
|Product Name|VantaTrack StockLens|
|Product Type|SaaS — Inventory Management Module|
|Target Audience|Agencies and SMEs using VantaTrack platform|
|Value Proposition|Unified, forecast-driven inventory across all item types — multi-location, auditable, connected to sales, projects, and finance|
|Platform|Responsive web app (desktop-optimised, mobile-friendly)|
|Deployment|`stocklens.vantatrack.io` via Vercel|
|Schema|`stocklens` in shared Neon PostgreSQL project|
|Auth|Shared VantaTrack JWT via `packages/auth`|

---

## 2. Problem Statement

Businesses today manage their inventory across disconnected tools:

- Real estate agencies track available units in Excel
- Product businesses use separate stock software that doesn't know about their CRM
- Projects consume stock with no link to what's left in the warehouse
- Purchases are logged manually in accounting tools with no connection to stock levels
- No one knows total inventory value without running a manual report
- Reorder decisions rely on gut feel instead of actual consumption rate
- Stock scattered across multiple locations can't be seen in one place, so staff over-order at one site while another sits overstocked
- Nobody can say who changed a price or deleted an item last month

This creates critical problems:

1. **Double selling** — two agents create deals on the same unit because availability isn't shared
2. **Stock surprises** — projects run out of materials mid-delivery because no one checked
3. **Financial blind spots** — no clear view of how much capital is tied up in inventory
4. **Manual reconciliation** — every report requires cross-referencing multiple tools
5. **Reactive reordering** — fixed thresholds trigger too late or too early because they ignore actual sales velocity
6. **No accountability** — item and price edits aren't traceable to a person or a time

VantaTrack StockLens solves this by making inventory a connected, forecast-aware, fully auditable layer of the platform — not another isolated tool.

---

## 3. Item Types

StockLens handles five categories of inventory. Every org can use any or all categories:

### 3.1 Real Estate Units

Flats, plots, floors, commercial spaces, parking bays. Grouped by building, block, or project. Each unit has physical attributes (area, floor, bedrooms), pricing, and a lifecycle status (available → reserved → sold).

### 3.2 Physical Products & Stock

Tangible goods that are bought, stored, and sold or consumed. Tracked by quantity **per location** (see 6.9). Movements are logged (stock in, stock out, transfer). Has a reorder point that can be fixed or forecast from consumption history (see 6.4).

Supports optional batch/lot tracking with expiry dates for perishable or regulated goods, and individual serial numbers for high-value or warrantied units (see 6.10). Can be purchased in one unit of measure and stocked/sold in another, e.g. bought by the carton, sold by the piece (see 6.16).

### 3.3 Equipment & Assets

Items the company owns and uses — vehicles, machinery, tools, devices. Not sold, but tracked for availability, assignment, and condition. Can be assigned to projects or team members. Can carry a serial number for warranty and service history.

### 3.4 Digital & Service Items

Software licences, subscriptions, service packages, templates, digital products. No physical quantity — tracked by availability, assignment, and expiry date.

### 3.5 Kits & Bundles

Composite items assembled from other StockLens items — e.g. a "Starter Package" made of three physical products, or a furnished-flat bundle combining a real estate unit with equipment. A kit is its own catalogue entry with its own price; assembling or selling it automatically deducts the component items' stock (see 6.11).

---

## 4. User Roles

|Role|Who|Permissions|
|---|---|---|
|StockLens Admin|Business owner, operations manager|Full access — create, edit, delete items, manage locations, suppliers, POs, transfers, returns, cycle counts, custom fields, reports, settings. Only Admins can be assigned as PO approvers.|
|StockLens Staff|Warehouse staff, agents, field team|Stock in/out, item lookup, barcode scanning, view stock levels, create movement entries, submit transfer/return/cycle-count requests — cannot delete items, change pricing, or approve POs above the approval threshold|
|StockLens Viewer|Managers, executives, clients|Read-only — view stock levels, item details, reports — no write access|

Roles are assigned per user per org via the gateway's `shared.users` role system. A user can be StockLens Staff in one org and StockLens Viewer in another if they belong to multiple orgs.

---

## 5. Core Features

### 6.1 Item Management

The central catalogue of everything an org tracks in StockLens.

**Creating an item:**

Every item has a core set of fields plus type-specific attributes stored in a flexible JSONB column:

```
Core fields (all item types):
  Name (required)
  Item type: real_estate | physical | equipment | digital | kit
  Category (user-defined: e.g. "Residential", "Electronics", "Software")
  SKU / Item code (optional, auto-generated if blank)
  Description
  Images (multiple uploads)
  Currency: BDT | USD
  Purchase price (cost to acquire)
  Selling price (price charged to buyer)
  Margin (auto-calculated: selling price - purchase price)
  Purchase unit of measure / stock unit of measure / conversion factor (physical items — see 6.16)
  Tags (colour-coded, user-defined)
  Custom fields (org-defined per item type — see 6.15)
  Notes
  Created by, Created at

Type-specific attributes (JSONB):
  Real estate:   { bedrooms, bathrooms, area_sqft, floor, block, facing, furnished }
  Physical:      { unit_of_measure, weight, dimensions, reorder_point, tracking_mode: none|batch|serial }
  Equipment:     { serial_number, condition, assigned_to, assigned_at, last_serviced }
  Digital:       { licence_key, seats, expiry_date, assigned_to, vendor_url }
  Kit:           { component_item_ids }
```

Note: a physical item no longer stores a single `location_id` or a single `quantity`. Both are derived — quantity per location is calculated from the movement ledger (see 6.9), and reorder points can be fixed or auto-forecast (see 6.4).

**Item statuses:**

|Type|Possible Statuses|
|---|---|
|Real estate|available, reserved, sold, off_market|
|Physical|in_stock, low_stock, out_of_stock, discontinued|
|Equipment|available, in_use, under_maintenance, retired|
|Digital|active, assigned, expired, cancelled|
|Kit|active, discontinued|

**Soft delete:** Deleting an item moves it to Trash. StockLens Admin can restore or permanently delete from Trash. Items linked to active deals, projects, transfers, or kits cannot be deleted until the link is resolved. Every delete, restore, price change, or status change is written to the audit log (see 6.17).

---

### 6.2 Item Groups & Locations

**Item Groups** Organise related items into logical containers:

- Real estate: Building → Block → Floor → Unit
- Physical: Product Line → Category → Individual SKU
- Equipment: Equipment Pool → Type → Individual Asset

Groups are hierarchical and unlimited in depth. Items can belong to one group.

**Storage Locations / Warehouses** Orgs can create as many locations as they need — or none if a single location is sufficient:

```
Location fields:
  Name (e.g. "Warehouse A", "Block B Showroom", "Head Office")
  Address
  Type: warehouse | showroom | office | site | virtual
  Manager (linked to shared.users)
  Notes
```

Stock is tracked per item per location (see 6.9). Stock moves between locations via ad hoc movements or formal Transfer Orders (see 6.12).

---

### 6.3 Stock Movement Tracking

Every change in physical stock quantity is logged as a movement record. This builds a complete history of what entered and left the inventory, at which location.

**Movement types:**

|Type|Description|Effect on Quantity|
|---|---|---|
|stock_in|Stock received from supplier or transfer|+ increase|
|stock_out|Stock sold, consumed, or dispatched|- decrease|
|transfer|Moved between two locations|neutral (deducted from source, added to destination)|
|adjustment|Manual correction (count discrepancy, damage write-off)|+ or -|
|return|Item returned from client, project, or supplier|+ or - (see 6.13)|
|reserved|Quantity held for a pending deal or project|held (not deducted until confirmed)|
|released|Reservation cancelled — quantity returned to available|released|
|kit_assembly|Component stock consumed to build a kit unit|- on components, + on kit|

**Movement record fields:**

```sql
stocklens.movements
  id, org_id, item_id,
  movement_type,
  quantity,
  location_id,              -- the location this movement affects
  from_location_id,        -- null for stock_in
  to_location_id,          -- null for stock_out
  batch_id,                -- nullable, links to stocklens.batches
  serial_id,               -- nullable, links to stocklens.serials
  reference_type,          -- 'purchase_order' | 'deal' | 'project' | 'transfer_order' | 'return' | 'cycle_count' | 'manual'
  reference_id,            -- links to PO, deal, project, transfer, return, or count
  unit_cost,               -- cost per unit at time of movement
  total_cost,              -- quantity x unit_cost
  notes,
  performed_by,            -- shared.users reference
  performed_at
```

Current stock level for any item — overall or at a specific location — is always calculated as the sum of its movements, never stored as a single editable number. This prevents manual tampering and ensures the history is always accurate.

---

### 6.4 Low Stock Alerts & Smart Reorder

Each physical stock item has a reorder point per location — either a fixed quantity or auto-forecast from consumption history.

**Fixed mode:** Alert fires when stock at a location falls at or below the configured `reorder_point`.

**Forecast mode:** StockLens calculates a rolling average daily consumption per item per location from the last 30/60/90 days of `stock_out` movements, multiplies by the supplier's `average_lead_time_days`, and adds a configurable safety margin to suggest a reorder point and quantity automatically. The suggestion recalculates weekly; Admins can accept it, override it, or revert to fixed mode per item.

**Alert behaviour:**

- When current stock at a location falls at or below its reorder point, item status changes to `low_stock` for that location
- StockLens Admin and Staff receive an in-app notification
- Alerts can also fire to email, Slack, or Microsoft Teams (via incoming webhook), configurable per org in Settings
- Item appears in the Low Stock report with a red indicator and, in forecast mode, a suggested reorder quantity
- If stock reaches zero at a location, status changes to `out_of_stock` there
- One-click "Create PO" pre-fills the supplier, item, and suggested quantity

**Alert settings per item:**

```
Reorder mode:        Fixed | Forecast
Reorder point:       [quantity] (fixed mode) or auto-calculated (forecast mode)
Alert channels:      In-app (always) + Email + Slack + Teams (opt-in per org)
Alert recipients:    StockLens Admin (always) + optional additional users
Alert frequency:     Once (until restocked) — no repeated daily alerts for the same item/location
```

---

### 6.5 Supplier & Vendor Management

StockLens maintains a supplier directory used when creating purchase orders.

**Supplier record:**

```sql
stocklens.suppliers
  id, org_id,
  name (required),
  contact_name,
  email, phone,
  address,
  payment_terms,           -- e.g. "Net 30", "50% advance"
  average_lead_time_days,
  currency_preference,     -- BDT | USD
  notes,
  is_active,
  created_at, updated_at
```

**Items link to preferred suppliers** — each item can have a default supplier so purchase orders pre-fill supplier details automatically, and so forecast reorder suggestions can use the supplier's real lead time.

---

### 6.6 Purchase Orders (PO)

When stock needs to be replenished, StockLens Admin (or Staff, if permitted) creates a Purchase Order.

**PO lifecycle:**

```
Draft → Pending Approval → Sent → Confirmed → Partially Received → Fully Received | Cancelled | Rejected
```

**Approval workflow:** Orgs can set a PO amount threshold in Settings. POs at or above the threshold move to `Pending Approval` and require sign-off from a designated StockLens Admin approver before they can be sent to the supplier. Below the threshold, POs skip straight to `Sent`. Every approval or rejection is written to the audit log.

**PO fields:**

```sql
stocklens.purchase_orders
  id, org_id,
  po_number,               -- VT-PO-2026-00123 (auto-generated)
  supplier_id,
  status,
  requires_approval,
  approved_by, approved_at,
  expected_delivery_date,
  delivery_location_id,    -- which warehouse to receive into
  currency,
  line_items JSONB,        -- [{ item_id, item_name, quantity, unit_cost, total, uom }]
  subtotal, tax, total,
  notes, internal_notes,
  created_by, created_at, updated_at

stocklens.po_receipts
  id, po_id, org_id,
  received_at,
  line_items JSONB,        -- [{ item_id, quantity_received, batch_number, expiry_date, serial_numbers, condition_note }]
  received_by,
  notes
```

**Receiving stock against a PO:** When a PO is marked as received (fully or partially), StockLens automatically creates `stock_in` movement records for each line item at the linked delivery location — creating batch or serial records where the item requires them (see 6.10).

**Cross-product connection — Spend:** When a PO is marked Fully Received, StockLens fires a cross-product transaction that creates an expense record in `spend.expenses` with the PO total, supplier name, and line items as the reference. Finance team sees the stock purchase automatically in their expense tracker without manual entry.

---

### 6.7 Barcode & QR Code Support

**Barcode scanning (stock in/out):**

- Every item, batch, or serial unit can have a barcode or QR code assigned (manually entered or auto-generated)
- Staff can use a device camera to scan items during stock movements, transfers, cycle counts, and returns
- Scanning opens the item directly and pre-fills the relevant form
- Supports standard formats: EAN-13, UPC-A, QR Code, Code 128

**QR code generation:**

- StockLens generates a printable QR code label for any item, batch, or serialised unit
- Label includes: item name, SKU, location, batch/serial number (if applicable), QR code
- Labels can be bulk-printed for an entire location, item group, or received PO line

**Implementation note:** Camera-based scanning uses the browser's native camera API — no external hardware required. A physical barcode scanner (USB/Bluetooth HID device) also works as it inputs keystrokes directly into the scan field.

---

### 6.8 Bulk CSV Import

For orgs migrating from spreadsheets or other systems.

**Import flow:**

1. Download template CSV (different template per item type; batch/serial columns included where relevant)
2. Fill in item data
3. Upload CSV — StockLens validates each row before importing
4. Preview: shows valid rows (green) and errors (red) with reason
5. Confirm import — valid rows are created, errored rows are skipped
6. Download error report for rows that failed

**Import history** is logged showing file name, row count, success count, error count, and the user who performed it.

---

### 6.9 Multi-Location Stock Visibility

Replaces the single "current location" model with true per-location stock tracking.

- Every physical item's stock is the sum of its movements, grouped by `location_id` — an item can simultaneously have stock at any number of locations
- Item detail view shows a **Stock by Location** table: quantity on hand, reserved, and available at each location
- Reorder points, low stock alerts, and forecast suggestions are evaluated **per location**, not just globally
- Dashboard and reports can be filtered to one location or rolled up across all locations
- Moving stock between locations is done via an ad hoc `transfer` movement (quick, single-step) or a formal Transfer Order (see 6.12) for larger or cross-team movements

---

### 6.10 Batch, Lot & Serial Tracking

For physical items where `tracking_mode` is set to `batch` or `serial`.

**Batch/lot tracking** (perishables, regulated goods, anything with expiry):

```sql
stocklens.batches
  id, org_id, item_id,
  batch_number,
  manufacture_date, expiry_date,
  quantity_received, quantity_remaining,
  location_id,
  supplier_id, po_id,
  created_at
```

- Stock movements for batch-tracked items must reference a `batch_id`
- Expiry alerts fire on a configurable window (e.g. 30 days before expiry), separate from low-stock alerts
- Expired batches are automatically excluded from "available" stock counts and flagged in the Stock Aging report
- FEFO (first-expiry-first-out) is the default suggestion when picking stock for an order, overridable by staff

**Serial number tracking** (high-value or warrantied units):

```sql
stocklens.serials
  id, org_id, item_id,
  serial_number,
  status,                  -- in_stock | reserved | sold | in_use | returned | retired
  batch_id,                -- nullable
  location_id,
  sold_to_deal_id,         -- nullable, links to pulse.deals
  created_at, updated_at
```

- Each serialised unit is tracked individually through its own lifecycle, independent of the item's aggregate quantity
- Supports warranty and service history lookups by serial number
- Equipment items (3.3) reuse this same table for asset-level tracking

---

### 6.11 Kitting & Bill of Materials

For kit/bundle items (3.5).

```sql
stocklens.kit_components
  id, org_id, kit_item_id,     -- references the kit's own stocklens.items row
  component_item_id,
  quantity_required,
  created_at
```

- A kit is defined once with its list of components and quantities (its "bill of materials")
- **Assembling** a kit creates `kit_assembly` movements: component stock is deducted (`stock_out`) at the assembly location and kit stock is added (`stock_in`)
- **Selling** a kit directly (without pre-assembly) deducts all components' stock in one transaction, keeping component-level inventory accurate without a separate assembly step
- Kit availability is capped by whichever component has the least stock relative to its required quantity — the item view shows "buildable quantity" for quick reference

---

### 6.12 Transfer Orders

A formal, auditable alternative to ad hoc transfer movements — for larger quantities, cross-team handoffs, or orgs that want approval before stock leaves a location.

**Lifecycle:** `Requested → Approved → In Transit → Received | Rejected`

```sql
stocklens.transfer_orders
  id, org_id, transfer_number,   -- VT-TR-2026-00045 (auto-generated)
  from_location_id, to_location_id,
  status,
  line_items jsonb,              -- [{ item_id, quantity, batch_id, serial_ids }]
  requested_by, approved_by,
  notes,
  created_at, updated_at
```

- Requesting a transfer reserves the requested quantity at the source location so it can't be double-allocated while in transit
- Approval creates a `transfer` movement pair (stock_out at source, stock_in at destination) once marked Received
- Transfer history is visible from both the source and destination location views

---

### 6.13 Returns & RMA Management

Handles stock coming back into the system from two directions:

**Customer returns** — linked to a Pulse deal or sale. Staff records the returned item, quantity, and condition; StockLens routes it to `restock` (stock_in at the receiving location), `write_off` (adjustment, removed from available stock), or `send_back` (queued for supplier return).

**Supplier returns** — linked to a Purchase Order. Used when received stock is faulty or incorrect; creates a `stock_out` movement and, optionally, a corresponding note on the PO for a credit note or replacement.

```sql
stocklens.returns
  id, org_id,
  return_type,             -- 'customer' | 'supplier'
  reference_type, reference_id,   -- 'deal' | 'purchase_order'
  item_id, quantity, batch_id, serial_id,
  condition_note,
  disposition,              -- 'restock' | 'write_off' | 'send_back'
  performed_by, created_at
```

---

### 6.14 Cycle Counts & Stock Audits

Scheduled or ad hoc physical counts to catch and correct drift between system and shelf.

```sql
stocklens.cycle_counts
  id, org_id, location_id,
  status,                   -- 'scheduled' | 'in_progress' | 'completed'
  scheduled_date, completed_at,
  created_by

stocklens.cycle_count_lines
  id, cycle_count_id,
  item_id, batch_id,
  expected_quantity, counted_quantity, variance,
  notes
```

- A count can cover a whole location, a single item group, or an ABC-classified subset (e.g. count all "A" items monthly, "C" items quarterly)
- Staff scan or manually enter counted quantities against the expected quantities StockLens pre-fills from the ledger
- On completion, variances above zero generate `adjustment` movements automatically, each tagged back to the cycle count for audit purposes
- Cycle Count Variance report shows accuracy rate per location and per counter over time

---

### 6.15 Custom Fields

Lets each org extend the catalogue without waiting on engineering.

```sql
stocklens.custom_fields
  id, org_id, item_type,
  field_name, field_type,   -- text | number | date | dropdown | boolean
  options jsonb,             -- for dropdown type
  is_required,
  created_at
```

- Configured per org, per item type, in Settings
- Appears on the item form alongside core fields and is included in CSV import/export templates and filters

---

### 6.16 Unit of Measure Conversion

For physical items purchased and stocked/sold in different units — e.g. bought by the carton (12 pieces), stocked and sold by the piece.

- Each item defines a `purchase_uom`, a `stock_uom`, and a `conversion_factor`
- Purchase Orders are entered in `purchase_uom`; on receipt, StockLens automatically converts to `stock_uom` when creating the `stock_in` movement
- Reports and stock levels always display in `stock_uom` for consistency; PO totals still reflect `purchase_uom` pricing

---

### 6.17 Audit Log & Change History

A separate, immutable trail from the movement ledger — covering non-quantity changes: price edits, status changes, deletions, permission changes, PO approvals, and settings changes.

```sql
stocklens.audit_log
  id, org_id,
  entity_type, entity_id,   -- 'item' | 'purchase_order' | 'supplier' | 'settings' | ...
  field_name, old_value, new_value,
  changed_by, changed_at
```

- Visible from the item, PO, or supplier detail view as a "History" tab
- Exportable per entity or per date range for compliance or dispute resolution
- Write-only from the application layer — no update or delete endpoint exists for this table

---

## 6. Cross-Product Connections

### 7.1 StockLens ↔ Pulse (CRM) — The Deal-to-Unit Lifecycle

Already defined in the Master Architecture PRD. Summary:

- Pulse deal creation → agent links a specific StockLens item (e.g. flat, or a serialised unit) to the deal
- Availability check runs before linking — prevents double allocation
- Company optionally marks item as `reserved` when deal enters negotiation
- Deal marked Won → StockLens item auto-updates to `sold` in one database transaction
- Deal marked Lost → StockLens item releases back to `available`
- Customer returns against a won deal flow into StockLens Returns (see 6.13)
- Pulse deal view shows linked unit details
- StockLens item view shows linked deal details

**Pulse Projects module** (Real Estate vertical) pulls unit data directly from StockLens once StockLens is live — replacing current manual entry in the CRM Projects module.

---

### 7.2 StockLens ↔ Spend (Expenses) — Purchase-to-Expense Sync

When a Purchase Order in StockLens is marked Fully Received:

```
StockLens PO marked received →
  packages/db/crossProduct.js creates expense record in spend.expenses →
  Expense: supplier name, PO number, total amount, currency, line items →
  Spend shows it as a categorised business expense automatically →
  shared.events logs 'po.received' for audit trail
```

If Spend is not activated for the org, this step is silently skipped.

---

### 7.3 StockLens ↔ Works (Projects) — Stock Allocation to Projects

When a project is created or updated in Works, project managers can allocate stock items from StockLens:

```
Works project needs 50 units of Item X →
  Project manager selects items from StockLens within Works (from a specific location) →
  StockLens creates 'reserved' movement for 50 units against project_id →
  StockLens available quantity decreases by 50 at that location (reserved, not consumed yet) →
  As project progresses, stock is marked consumed (stock_out movement) →
  On project completion, unused reserved stock is released back to available
```

Works project view shows: allocated items, quantities used vs reserved, estimated material cost. StockLens item view shows: which projects are currently using this item.

---

### 7.4 StockLens → Lens (Reporting) & Command (Dashboard)

StockLens feeds the following data to Lens and Command:

- Total inventory value (sum of all items × purchase price)
- Items by status (available, reserved, sold, low stock), per location and rolled up
- Stock movement volume over time
- Top sold items this month
- Supplier spend summary
- Forecast accuracy (suggested vs actual reorder timing)
- Dead stock and aging value at risk

These are read via cross-schema SQL queries in `packages/db/` — no extra API calls needed.

---

## 7. Reporting

All reports are accessible under the Reports section of StockLens. Each report is filterable by date range, item type, category, location, and supplier. All reports export to CSV.

### 8.1 Stock Valuation Report

Total value of current inventory broken down by:

- Item type
- Category
- Storage location

Shows: quantity on hand, purchase price per unit, total value, selling price, potential revenue, margin.

### 8.2 Movement History Report

Full log of every stock movement across all items. Filterable by movement type, item, location, date range, and performed by.

Shows: date, item, type, quantity, from/to location, batch/serial (if applicable), reference (PO/deal/project/transfer/return/count), performed by.

### 8.3 Low Stock Report

All items currently at or below their reorder point, per location.

Shows: item name, SKU, location, current quantity, reorder point (fixed or forecast), suggested reorder quantity, default supplier, last restocked date. One-click "Create PO" button per row.

### 8.4 Supplier Performance Report

Per-supplier summary of purchase history.

Shows: supplier name, total POs, total spend, average lead time vs expected, on-time delivery rate, last order date.

### 8.5 Item Sales Report

For items linked to Pulse deals — shows sold items, sale price, buyer, date, agent, and margin.

### 8.6 Stock Aging & Dead Stock Report

Flags slow-moving and non-moving inventory by value at risk.

Shows: item, location, quantity, days since last movement, batch expiry (if applicable), inventory value tied up, ABC class. Sortable to surface highest-value dead stock first.

### 8.7 ABC Inventory Classification Report

Auto-classifies physical items into A (top ~20% of value/velocity), B, and C bands, recalculated monthly. Used to prioritise cycle count frequency and reorder attention — A items counted most often, C items least.

### 8.8 Cycle Count Variance Report

Per cycle count: expected vs counted quantity, variance value, and accuracy rate by location and by counter over time.

---

## 8. Database Schema

```sql
-- Items
stocklens.items
  id uuid PK,
  org_id uuid REFERENCES shared.orgs(id),
  item_type text,                -- 'real_estate' | 'physical' | 'equipment' | 'digital' | 'kit'
  name text NOT NULL,
  category text,
  sku text UNIQUE,
  description text,
  images jsonb,
  tags jsonb,
  custom_fields jsonb,
  attributes jsonb,              -- type-specific fields, incl. tracking_mode: none|batch|serial
  status text,
  group_id uuid REFERENCES stocklens.item_groups(id),
  supplier_id uuid REFERENCES stocklens.suppliers(id),
  purchase_price numeric,
  selling_price numeric,
  currency text DEFAULT 'BDT',
  reorder_mode text DEFAULT 'fixed',   -- 'fixed' | 'forecast'
  reorder_point integer,               -- physical items only, per-location overrides in stocklens.reorder_points
  purchase_uom text, stock_uom text, uom_conversion_factor numeric,
  barcode text,
  qr_code text,
  -- Cross-product links
  deal_id uuid,                  -- pulse.deals reference
  project_id uuid,               -- works.projects reference
  -- Meta
  created_by uuid REFERENCES shared.users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz

-- Per-location reorder overrides (optional; falls back to items.reorder_point)
stocklens.reorder_points
  id, org_id, item_id, location_id,
  reorder_mode, reorder_point, suggested_quantity, last_calculated_at

-- Item groups (hierarchical)
stocklens.item_groups
  id, org_id, parent_group_id,   -- self-referencing for hierarchy
  name, type, description,
  attributes jsonb,
  created_at

-- Storage locations
stocklens.locations
  id, org_id,
  name, type,                    -- 'warehouse'|'showroom'|'office'|'site'|'virtual'
  address, manager_id,
  notes, is_active, created_at

-- Stock movements
stocklens.movements
  id, org_id, item_id,
  movement_type,
  quantity,
  location_id, from_location_id, to_location_id,
  batch_id, serial_id,
  reference_type, reference_id,
  unit_cost, total_cost,
  notes, performed_by, performed_at

-- Batch / lot tracking
stocklens.batches
  id, org_id, item_id, batch_number,
  manufacture_date, expiry_date,
  quantity_received, quantity_remaining,
  location_id, supplier_id, po_id,
  created_at

-- Serial number tracking (physical + equipment)
stocklens.serials
  id, org_id, item_id, serial_number,
  status, batch_id, location_id,
  sold_to_deal_id, created_at, updated_at

-- Kit bill of materials
stocklens.kit_components
  id, org_id, kit_item_id, component_item_id,
  quantity_required, created_at

-- Transfer orders
stocklens.transfer_orders
  id, org_id, transfer_number,
  from_location_id, to_location_id,
  status, line_items jsonb,
  requested_by, approved_by,
  notes, created_at, updated_at

-- Returns / RMA
stocklens.returns
  id, org_id, return_type,
  reference_type, reference_id,
  item_id, quantity, batch_id, serial_id,
  condition_note, disposition,
  performed_by, created_at

-- Cycle counts
stocklens.cycle_counts
  id, org_id, location_id, status,
  scheduled_date, completed_at, created_by

stocklens.cycle_count_lines
  id, cycle_count_id, item_id, batch_id,
  expected_quantity, counted_quantity, variance, notes

-- Custom fields
stocklens.custom_fields
  id, org_id, item_type, field_name, field_type,
  options jsonb, is_required, created_at

-- Audit log (non-quantity changes)
stocklens.audit_log
  id, org_id, entity_type, entity_id,
  field_name, old_value, new_value,
  changed_by, changed_at

-- Suppliers
stocklens.suppliers
  id, org_id, name, contact_name,
  email, phone, address,
  payment_terms, average_lead_time_days,
  currency_preference, notes,
  is_active, created_at, updated_at

-- Purchase orders
stocklens.purchase_orders
  id, org_id, po_number,
  supplier_id, status,
  requires_approval, approved_by, approved_at,
  expected_delivery_date,
  delivery_location_id,
  currency,
  line_items jsonb,
  subtotal, tax, total,
  notes, internal_notes,
  created_by, created_at, updated_at

-- PO receipts
stocklens.po_receipts
  id, po_id, org_id,
  received_at, line_items jsonb,
  received_by, notes

-- Import history
stocklens.import_history
  id, org_id, file_name,
  item_type, total_rows,
  success_count, error_count,
  error_details jsonb,
  imported_by, imported_at
```

---

## 9. Technical Stack

Consistent with all VantaTrack products:

|Layer|Technology|
|---|---|
|Frontend|React.js (v18) + Vite|
|Styling|Tailwind CSS v4|
|Routing|Wouter|
|State|TanStack Query|
|Backend|Node.js + Express|
|ORM|Drizzle ORM|
|Database|PostgreSQL (Neon) — `stocklens` schema|
|Auth|Shared JWT via `packages/auth`|
|Cross-product|`packages/db/crossProduct.js`|
|File uploads|Multer (images)|
|CSV parsing|csv-parser, xlsx|
|Barcode|`html5-qrcode` (camera scanning)|
|QR generation|`qrcode` npm package|
|Email|Nodemailer via SMTP (low stock, expiry, approval alerts)|
|Chat alerts|Slack / Microsoft Teams incoming webhooks|
|Scheduled jobs|Cron worker for forecast recalculation, expiry checks, cycle count reminders|

---

## 10. UI Structure

```
Sidebar navigation:
  Dashboard          ← StockLens home — stock overview, alerts, recent movements
  Items              ← Full item catalogue with filters (incl. kits)
  Groups             ← Item group hierarchy
  Locations          ← Warehouse / location management, stock-by-location view
  Movements          ← Full movement log
  Transfers          ← Transfer order list and creation
  Returns            ← Customer and supplier returns / RMA
  Cycle Counts       ← Scheduled and in-progress physical counts
  Purchase Orders    ← PO list, creation, and approvals
  Suppliers          ← Supplier directory
  Reports            ← All 8 reports
  Import             ← CSV bulk import
  Settings           ← Categories, tags, custom fields, alert channels, PO approval threshold
  Trash              ← Soft-deleted items
```

### Dashboard widgets:

```
Total items                Items by status (donut chart)
Total inventory value      Low stock alerts (red banner if any)
Movements today             Recent activity feed
Pending POs / approvals     Top items by movement this month
Dead stock value at risk    Batches expiring within 30 days
```

---

## 11. Build Phases

### Phase 1 — Core Inventory (Ship first)

- Item CRUD (all 5 types, incl. kits)
- Item groups and locations
- Status management
- Barcode/QR generation and scanning
- CSV bulk import
- Basic dashboard

**Done when:** Orgs can add and manage inventory independently. No cross-product connections yet.

---

### Phase 2 — Stock Operations

- Movement tracking (stock in, stock out, transfer, adjustment)
- Multi-location stock visibility
- Low stock alerts (fixed mode) and email notifications
- Supplier management
- Purchase Orders (create, send, receive)
- Movement history report
- Low stock report
- Stock valuation report

**Done when:** Full stock lifecycle works — PO to receipt to movement to alert, across multiple locations.

---

### Phase 3 — Cross-Product Connections

- StockLens ↔ Pulse: deal-to-unit lifecycle (reservation, sold sync, availability gating)
- StockLens ↔ Spend: PO receipt → auto expense creation
- StockLens ↔ Works: project stock allocation and consumption
- StockLens → Lens + Command: inventory metrics feeds
- Supplier performance report
- Item sales report

**Done when:** StockLens is fully connected to all relevant products. Real estate companies can manage their full unit-to-deal lifecycle.

---

### Phase 4 — Advanced Inventory Operations

- Batch/lot tracking with expiry alerts and FEFO suggestions
- Serial number tracking for physical and equipment items
- Kitting & Bill of Materials
- Smart/forecast-based reorder suggestions
- Transfer Orders with approval workflow
- Returns & RMA management
- Cycle counts and stock audits
- Custom fields
- Audit log & change history
- PO approval workflow
- Slack / Teams alert channels
- Unit of measure conversion
- Stock aging, ABC classification, and cycle count variance reports

**Done when:** StockLens matches the operational depth expected of a dedicated inventory management system, not just a stock ledger.

---

## 12. Non-Goals (Explicitly Out of Scope for V1)

- Double-entry accounting or balance sheet integration
- Automated payment to suppliers
- Real-time GPS tracking of assets
- IoT sensor integration for automated stock counting
- Native mobile app (responsive web only)
- Multi-currency conversion (BDT and USD stored separately, no live FX)
- Marketplace or e-commerce storefront
- Automated demand forecasting beyond simple moving-average velocity (no ML-based forecasting in V1)

---

## 13. Definition of Done — Production

StockLens is production-ready when:

- All 5 item types (incl. kits) can be created, edited, and soft-deleted
- Stock movements are logged and current levels — overall and per location — are always calculated from movement history
- Low stock alerts fire correctly in fixed and forecast mode, via in-app, email, and optional Slack/Teams channels
- Purchase Orders flow from Draft through approval (where required) to Received with automatic stock_in movements on receipt
- Batch/lot items track expiry and surface FEFO suggestions; serial-tracked items maintain individual lifecycle status
- Kits can be assembled or sold with correct component stock deduction
- Transfer Orders move stock between locations with reservation, approval, and receipt steps
- Returns (customer and supplier) route correctly to restock, write-off, or send-back
- Cycle counts generate accurate adjustment movements and a variance report
- CSV import handles 1000+ rows without timeout
- Barcode scanning works on mobile camera without hardware
- StockLens ↔ Pulse deal lifecycle works end-to-end (reserve, won, lost, return)
- StockLens ↔ Spend PO expense sync works when both products are active
- StockLens ↔ Works project stock allocation works when Works is active
- All 8 reports generate and export to CSV correctly
- Role-based access enforces StockLens Admin / Staff / Viewer permissions correctly, including PO approver restriction
- Every price, status, deletion, and approval change is recorded in the audit log
- `requireProduct('stocklens')` returns 403 for orgs without StockLens activated

---

_Document Owner: VantaTrack Product Team_ _Next Review: When Phase 1 development begins_ _Related documents: VantaTrack Master Architecture PRD, VantaTrack Suite MD_
