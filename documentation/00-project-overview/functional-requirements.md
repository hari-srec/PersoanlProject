# BakeFlow – Functional Requirements

> **Step:** 1.1 · **Status:** Draft for owner review · **Source:** [master-prompt.md](../master-prompt.md) §1, Appendices B and D

## Table of Contents

1. [How to Read This Document](#1-how-to-read-this-document)
2. [Customer Platform – FR-CUS](#2-customer-platform--fr-cus)
3. [Branch Management – FR-BR](#3-branch-management--fr-br)
4. [Central Administration – FR-ADM](#4-central-administration--fr-adm)
5. [Requirement Counts](#5-requirement-counts)
6. [Coverage Check Against master-prompt.md](#6-coverage-check-against-master-promptmd)
7. [Related Documents](#7-related-documents)

---

## 1. How to Read This Document

Each requirement has:

| Column | Meaning |
|--------|---------|
| **ID** | Unique, permanent ID. Prefix: `FR-CUS` (customer), `FR-BR` (branch), `FR-ADM` (central admin). IDs are never reused. |
| **Title / Description** | What the system must let the user do. |
| **Role(s)** | Who performs it. Roles are defined in [user-roles-and-personas.md](user-roles-and-personas.md): `SA` = SUPER_ADMIN, `OA` = ORG_ADMIN, `BM` = BRANCH_MANAGER, `BS` = BRANCH_STAFF, `CU` = CUSTOMER, `SYS` = system (automatic). |
| **Priority** | **Must** = the product is not usable without it. **Should** = important, can follow shortly. **Could** = nice to have. |
| **MVP** | **Yes** if part of the first usable release. Based on Appendix D, see [feature-roadmap.md](feature-roadmap.md#4-mvp-definition). |
| **Module** | The Phase 3 module that implements it. |
| **Source** | Where it comes from: `MP 1.A` = master-prompt §1.A, `App B4` = Appendix B item 4, `App C` = Appendix C, `SG-xx` = an architect suggestion not explicitly in the master prompt, listed in [scope-and-assumptions.md](scope-and-assumptions.md#6-architect-suggestions). |

---

## 2. Customer Platform – FR-CUS

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-CUS-01 | Register account | Create a customer account with name, mobile number, email and password. Email and mobile must be unique within the organization. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-02 | Log in and log out | Log in with email or mobile plus password; log out; stay logged in through token refresh. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-03 | Reset forgotten password | Request a password-reset link by email and set a new password. | CU | Should | Yes (MVP-B) | 3.9 | SG-01 |
| FR-CUS-04 | Select branch | Choose a preferred branch. Availability, prices and offers shown are for that branch. The customer can change branch at any time; the cart is tied to one branch. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-05 | Browse products by category | Browse active products of the selected branch, grouped by category, with image, name, description and price. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-06 | Search products | Search by keyword, filtered by category. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-07 | View branch availability | See whether each product is available at the selected branch, based on **available** quantity in sellable batches. Expired batches, and batches inside the online sale cut-off (AS-27), are excluded automatically. | CU, SYS | Must | Yes | 3.9 | MP 1.A, App B1 |
| FR-CUS-08 | View price | See the selling price (GST-inclusive, AS-06) for the selected branch. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-09 | View offers and near-expiry discounts | See active offers, including near-expiry discounts, with the original price and the discounted price shown separately. | CU | Should | Yes | 3.12 | MP 1.A |
| FR-CUS-10 | Manage cart | Add, update and remove items in a cart for one branch. **The cart does not reserve stock** (ADR-012). Availability is re-checked when the cart is viewed and at checkout. | CU | Must | Yes | 3.10 | MP 1.A, App C |
| FR-CUS-11 | Checkout with reservation | Start checkout: the system reserves stock by FEFO for a limited time (default 15 min, AS-13) and shows the final price. If stock is insufficient, the customer is told which items cannot be supplied. | CU, SYS | Must | Yes | 3.10 | MP 1.A, App C |
| FR-CUS-12 | Choose pickup | Choose pickup at the selected branch (same day, during branch hours, AS-11). | CU | Must | Yes | 3.10 | MP 1.A |
| FR-CUS-13 | Choose delivery | Choose delivery to a saved address where the branch supports delivery (delivery area and fee configured per branch). | CU | Should | No | 3.10 | MP 1.A |
| FR-CUS-14 | Pay online | Pay through Razorpay (UPI, card, net banking). The order is confirmed only after the payment is verified on the server (signature and webhook). If payment fails or times out, the reservation is released. | CU, SYS | Must | Yes | 3.10 | MP 1.A |
| FR-CUS-15 | Track order status | See the current order status and a timeline (for example Placed → Confirmed → Ready → Completed). | CU | Must | Yes | 3.11 | MP 1.A |
| FR-CUS-16 | View order history | List previous orders with items, amounts and status. | CU | Must | Yes | 3.11 | MP 1.A, App B5 |
| FR-CUS-17 | Cancel eligible order | Cancel an order while it is in a cancellable status (AS-31). Reserved stock is released (BR-10) and an online refund is started automatically. | CU | Must | Yes | 3.11 | MP 1.A |
| FR-CUS-18 | Receive order notifications | Get in-app and email notifications when the order status changes. | CU, SYS | Should | No | 3.15 | MP 1.A |
| FR-CUS-19 | Submit reviews and ratings | Rate (1–5) and review products from completed orders only (AS-36). | CU | Could | No | 3.18 | MP 1.A |
| FR-CUS-20 | Manage profile | View and update name, mobile, email and password. | CU | Must | Yes | 3.9 | MP 1.A |
| FR-CUS-21 | Manage addresses | Add, edit, delete and set a default delivery address. Needed only once delivery (FR-CUS-13) is enabled. | CU | Should | No | 3.9 | MP 1.A |
| FR-CUS-22 | Favourites and repeat order | Mark favourite products and re-order a previous order in one step (availability re-checked). | CU | Could | No | 3.18 | App B5 |
| FR-CUS-23 | Loyalty points | Earn points on completed orders and redeem them on later orders. | CU | Could | No | 3.18 | App B5 |
| FR-CUS-24 | Personalised offers | See offers based on order history and favourites. | CU | Could | No | 3.18 | App B5 |
| FR-CUS-25 | View invoice | View and download the GST invoice for a completed order. | CU | Should | No | 3.11 | SG-02 |

---

## 3. Branch Management – FR-BR

All FR-BR requirements are limited to the branches the user is assigned to (BR-14).

### 3.1 Product management

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-01 | View branch products | View catalog products enabled for the branch, with price, category and current available quantity. | BM, BS | Must | Yes | 3.3 | MP 1.B |
| FR-BR-02 | Manage branch product availability | Enable or disable a product for the branch, separately for POS and for online ordering. Set the low-stock threshold. | BM | Must | Yes | 3.3 | MP 1.B |
| FR-BR-03 | Set branch selling price | Override the organization's base price for the branch, only within the band allowed by central pricing rules (FR-ADM-07). The override is audited. | BM | Should | No | 3.3 | MP 1.B |
| FR-BR-04 | Branch category display | Control which organization categories are shown at the branch and in what order. Categories themselves are managed centrally (OQ-08). | BM | Could | No | 3.3 | MP 1.B |

### 3.2 Inventory management

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-05 | Record stock receipt | Receive goods from outside (not baked at the branch), creating a batch with batch number, product, quantity, manufacturing date-time, expiry date-time and cost. Writes `RECEIPT_IN`. | BM, BS | Must | Yes | 3.4 | MP 1.B "Add stock" |
| FR-BR-06 | Track stock by batch | View each batch with received or produced, on-hand, reserved, available, sold, wasted and transferred quantities; status; manufacturing and expiry times. Filter by product, status and expiry. | BM, BS | Must | Yes | 3.4 | MP 1.B |
| FR-BR-07 | View stock movement history | View the append-only ledger per batch, product or date range: type, quantity, user, time and reference document. | BM, BS | Must | Yes | 3.4 | MP 1.B |
| FR-BR-08 | FEFO allocation | When stock is needed (POS line or online reservation), automatically allocate from sellable batches in order of earliest expiry, splitting across batches when needed. | SYS | Must | Yes | 3.5 | MP 1.B, BR-04 |
| FR-BR-09 | Safe concurrent allocation | Two simultaneous sales of the last unit must never both succeed. Stock can never become negative. All stock changes are transactional. | SYS | Must | Yes | 3.5 | BR-09, 12, 13, App C |
| FR-BR-10 | Block expired stock at allocation | Refuse to allocate any batch whose expiry time has passed (or that is blocked), even if the scheduled job has not yet updated its status. | SYS | Must | Yes | 3.5 | BR-05, App C |
| FR-BR-11 | Stock adjustments | Record a correction (increase or decrease) with a mandatory reason and comment. Adjustments above a threshold need branch manager approval, and nobody can approve their own adjustment (OQ-13). Writes `ADJUSTMENT_IN/OUT`. | BS (request), BM (approve) | Must | Yes | 3.6 | App D6 |
| FR-BR-12 | Low-stock alerts | Show an alert when a product's available quantity at the branch falls below its threshold. | SYS, BM, BS | Should | Yes | 3.6 | MP 1.B |
| FR-BR-13 | Automatic expiry status update | A scheduled job marks batches as expired when expiry time passes and removes them from online availability. | SYS | Must | Yes | 3.6 | App B1, App C |
| FR-BR-14 | Expiry and near-expiry alerts | A dashboard of batches that have expired or are approaching expiry (remaining time below a configurable threshold), with quantities and suggested action. | BM, BS | Must | Yes | 3.6 | MP 1.B, App B1, App D8 |
| FR-BR-34 | Block or unblock a batch | Put a batch on hold (for example a suspected quality issue) so it cannot be allocated. Unblocking is audited. | BM | Should | Yes | 3.6 | SG-03 |

### 3.3 Production management

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-15 | Record daily production | Record what was baked: product, quantity, manufacturing date-time, expiry date-time (defaults from product shelf life). This creates a production batch and writes `PRODUCTION_IN`. | BS, BM | Must | Yes | 3.7 | MP 1.B, App D5 |
| FR-BR-16 | Production plan | Enter the planned quantity per product for a day and compare it with what was actually produced. | BM | Should | No | 3.7 | MP 1.B |
| FR-BR-17 | View historical sales | View past sales per product per day for the branch. | BM | Should | Yes | 3.16 | MP 1.B |
| FR-BR-18 | Production vs sales comparison | Compare produced, sold, wasted and remaining quantities per product per day (the Chennai example). | BM, OA | Should | No | 3.16 | MP 1.B, App A |
| FR-BR-19 | Production recommendation | Recommend tomorrow's production quantity per product from sales history, taking weekday and weekend patterns into account. Shown as a recommendation; the manager decides. | SYS, BM | Should | No | 3.17 | MP 1.B, App B2 |

### 3.4 Billing (POS)

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-20 | Create POS bill | Create a counter bill by product search or code. Stock is allocated by FEFO when the bill is finalised. Requires an `Idempotency-Key` (ADR-011). | BS, BM | Must | Yes | 3.8 | MP 1.B, App B4 |
| FR-BR-21 | Record payment methods | Record payment as cash, UPI or card, including a split across methods. For cash, record the amount tendered and the change. Recorded only; no terminal integration (AS-18). | BS, BM | Must | Yes | 3.8 | MP 1.B |
| FR-BR-22 | Apply discounts at POS | Apply active near-expiry discounts automatically to eligible batch lines. Apply a manual discount within the staff limit; above the limit, branch manager approval is needed. The original price is kept (BR-07). | BS, BM | Must | Yes | 3.12 | MP 1.B, App D9 |
| FR-BR-23 | Process refunds | Refund all or some lines of an invoice for an eligible reason, issuing a **credit note** with its own number series. Refunded items are recorded as wastage, not restocked (AS-15). Manager only (OQ-14). | BM | Must | Yes | 3.8 | MP 1.B |
| FR-BR-24 | Generate GST invoice | Issue an invoice with a gap-free number per branch per financial year (ADR-010) and GST breakdown (HSN, rate, CGST/SGST/IGST, ADR-006). Printable and re-printable. Invoices are never edited or deleted. | BS, BM, SYS | Must | Yes | 3.8 | MP 1.B |
| FR-BR-25 | Daily closing report | At day end, summarise sales by payment method, discounts, refunds and taxes. Record counted cash against expected cash and show the variance. The report is locked once closed. | BM | Must | Yes | 3.8 | MP 1.B, App B4 |

### 3.5 Expiry and discount management

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-26 | Branch discount rules | Configure branch-specific near-expiry rules within the limits set by the organization (FR-ADM-14). | BM | Could | No | 3.12 | MP 1.B "Configure discount policies" |
| FR-BR-27 | Near-expiry discount recommendation | For near-expiry batches, recommend a discount % from the matching rule, based on remaining shelf life. The recommendation never goes below the minimum margin. | SYS | Must | Yes | 3.12 | MP 1.B, App C |
| FR-BR-28 | Approve and apply discount | Approve, edit (within limits) or reject a recommended discount. Once approved, it applies to that batch at the POS and online until expiry. Rules may allow auto-approval. | BM | Must | Yes | 3.12 | App C |
| FR-BR-29 | Record expired stock | Record physical disposal of expired batch units as wastage with reason `EXPIRED`. Writes `WASTAGE_OUT`. | BS, BM | Must | Yes | 3.13 | MP 1.B, App B7 |
| FR-BR-30 | Record damaged or discarded stock | Record wastage with a reason (damaged, quality rejection, returned, other), quantity and comment. Wastage cost is calculated from standard unit cost (FR-ADM-08). | BS, BM | Must | Yes | 3.13 | MP 1.B, App B7 |

### 3.6 Branch operations

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-BR-31 | Basic sales and wastage report | Daily and date-range reports for the branch: sales by product, payment method, discounts, wastage quantity and value. | BM | Must | Yes | 3.16 | App D11 |
| FR-BR-32 | Branch analytics dashboard | Trends for revenue, top and slow products, wastage % and near-expiry sell-through. | BM | Should | No | 3.16 | MP 1.B, App B6 |
| FR-BR-33 | Branch notifications | In-app notifications for new online orders, low stock, near-expiry batches and pending approvals. | BM, BS | Should | No | 3.15 | MP 1.B (alerts) |
| FR-BR-35 | Fulfil online orders | View incoming online orders for the branch. Move them through the statuses (confirm, ready for pickup, hand over or complete, cancel with reason). Handover records the sale and issues the invoice (AS-14). | BS, BM | Must | Yes | 3.11 | MP 1.A (implied), SG-13 |
| FR-BR-36 | Handle deliveries | Assign a staff member to a delivery order and mark it out for delivery or delivered. | BM, BS | Should | No | 3.11 | MP 1.A |
| FR-BR-37 | Inter-branch stock transfer | Request, approve, dispatch (source stock ↓, in transit) and receive (destination stock ↑, record actual quantity and any shortfall) transfers. Batch number and expiry are kept. Full transfer history. | BM | Should | No | 3.14 | MP 1.B, App B3, BR-11 |
| FR-BR-38 | Manage employee access | Add, deactivate and reactivate staff for own branch; reset a staff password. Only roles at or below BRANCH_STAFF can be assigned. | BM | Must | Yes | 3.2 | MP 1.B |
| FR-BR-39 | Track employee actions | View the audit trail of actions by branch employees (bills, refunds, discounts, adjustments, wastage). | BM | Should | No | 3.15 | MP 1.B, BR-15 |

---

## 4. Central Administration – FR-ADM

| ID | Title | Description | Role | Priority | MVP | Module | Source |
|----|-------|-------------|------|----------|-----|--------|--------|
| FR-ADM-01 | Manage organizations | Create, update and deactivate bakery organizations (name, legal name, GSTIN, contact). In the MVP a single organization is created by seed data or the SUPER_ADMIN (AS-02). | SA | Must | Yes | 3.2 | MP 1.C |
| FR-ADM-02 | Manage branches | Create, update and deactivate branches (code such as `ANN`, name, address, phone, opening hours, pickup and delivery flags, invoice prefix). | OA | Must | Yes | 3.2 | MP 1.C, App D1 |
| FR-ADM-03 | Assign branch managers | Create staff users and assign them to branches with a role. Re-assign or remove. | OA | Must | Yes | 3.2 | MP 1.C |
| FR-ADM-04 | Manage global product catalog | Create and update products (name, description, category, images, unit, shelf life in hours, active flag, variants per OQ-09). | OA | Must | Yes | 3.3 | MP 1.C, App D3 |
| FR-ADM-05 | Configure categories | Create, rename, order and deactivate product categories for the organization. | OA | Must | Yes | 3.3 | MP 1.C |
| FR-ADM-06 | Set base price and tax data | Set each product's base selling price (GST-inclusive, AS-06), HSN code and GST rate. Price changes are kept as history with effective dates; old prices are never overwritten. | OA | Must | Yes | 3.3 | MP 1.C, ADR-006 |
| FR-ADM-07 | Configure pricing rules | Define how far branches may override the base price (for example ±10 %), per product or category. | OA | Should | No | 3.3 | MP 1.C "global pricing rules" |
| FR-ADM-08 | Set standard unit cost | Maintain a standard cost per product, used for wastage cost, minimum-margin checks and profitability. | OA | Should | Yes | 3.3 | SG-04 (needed by App B6, App C) |
| FR-ADM-09 | Consolidated inventory | View stock across all branches per product: available, reserved and near-expiry. | OA | Should | No | 3.16 | MP 1.C |
| FR-ADM-10 | Branch-wise revenue | Revenue per branch per day or date range, net of refunds. | OA | Must | Yes | 3.16 | MP 1.C, App D11 |
| FR-ADM-11 | Compare branch performance | Side-by-side branch comparison: revenue, wastage %, production vs sales, near-expiry sell-through. | OA | Should | No | 3.16 | MP 1.C, App B6 |
| FR-ADM-12 | Monitor wastage | Organization-wide wastage by branch, product and reason, in quantity and ₹ value. | OA | Must | Yes | 3.16 | MP 1.C, App D11 |
| FR-ADM-13 | Staff authentication and role management | Staff log in, log out, refresh tokens and change password. Admins assign the fixed roles (AS-22). Branch and organization scope is enforced on every request (BR-14). | SA, OA, BM, BS | Must | Yes | 3.1 | MP 1.C, App D2 |
| FR-ADM-14 | Configure near-expiry discount rules | Define organization rules: eligible products or categories, remaining-shelf-life tiers and discount %, maximum discount, minimum margin, approval required (yes/no), and whether the rule applies online. Rule changes are audited. | OA | Must | Yes | 3.12 | MP 1.B/1.C, BR-06, App C |
| FR-ADM-15 | Configure wastage reasons | Maintain the list of wastage reasons (EXPIRED, DAMAGED, QUALITY_REJECTED, RETURNED, TRANSIT_LOSS, OTHER). System reasons cannot be deleted. | OA | Should | Yes | 3.13 | App B7 |
| FR-ADM-16 | Configure system and organization settings | Settings such as reservation timeout, online sale cut-off before expiry, near-expiry threshold, cancellation policy, adjustment approval threshold and staff discount limit. SA manages platform-level settings. | SA, OA | Should | Yes | 3.2 | MP 1.C |
| FR-ADM-17 | Advanced analytics | Product-wise profitability, expiry-loss report, wastage %, slow- and fast-moving products. | OA | Should | No | 3.16 | App B6 |
| FR-ADM-18 | Demand forecasting | Forecast demand per product per branch, accounting for weekday, weekend and seasonal patterns; identify slow- and fast-moving products. Feeds FR-BR-19. | SYS, OA | Should | No | 3.17 | App B2 |
| FR-ADM-19 | Configure loyalty programme | Define points earning and redemption rules. | OA | Could | No | 3.18 | App B5 |
| FR-ADM-20 | Search audit log | Search the organization-wide audit log by user, branch, entity, action and date. | OA, SA | Should | No | 3.15 | BR-15 |
| FR-ADM-21 | Moderate reviews | Hide or show customer reviews that break the content policy. | OA | Could | No | 3.18 | SG-05 |
| FR-ADM-22 | Manage customer accounts | Search customers, view their order history, deactivate abusive accounts. | OA | Could | No | 3.9 | SG-06 |

---

## 5. Requirement Counts

| Area | Total | Must | Should | Could | MVP = Yes |
|------|------:|-----:|-------:|------:|----------:|
| FR-CUS | 25 | 15 | 6 | 4 | 17 |
| FR-BR | 39 | 25 | 12 | 2 | 28 |
| FR-ADM | 22 | 10 | 9 | 3 | 13 |
| **Total** | **86** | **50** | **27** | **9** | **58** |

---

## 6. Coverage Check Against master-prompt.md

Every bullet in master-prompt §1 and Appendices B and D maps to at least one requirement.

| Master-prompt item | Covered by |
|--------------------|-----------|
| **§1.A Customer** – register/login; browse; search by category; branch availability; price & offers; near-expiry discounts; cart; place orders; pickup/delivery; online payment; track; history; cancel; notifications; reviews; profile & addresses | FR-CUS-01/02; 05; 06; 07; 08/09; 09; 10; 11; 12/13; 14; 15; 16; 17; 18; 19; 20/21 |
| **§1.B Product** – view; branch availability; prices within central rules; categories | FR-BR-01; 02; 03 + FR-ADM-07; FR-BR-04 + FR-ADM-05 |
| **§1.B Inventory** – add stock; batch tracking; mfg date; expiry date-time; received/sold/reserved/available; damaged & expired; low-stock alerts; movement history; FEFO | FR-BR-05/15; 06; 05/15; 05/15; 06; 29/30; 12; 07; 08 |
| **§1.B Production** – record; historical sales; recommend; compare; reduce overproduction | FR-BR-15; 17; 19; 18; 16/18/19 |
| **§1.B Billing** – bills; cash/UPI/card; discounts; refunds; invoices; daily closing | FR-BR-20; 21; 22; 23; 24; 25 |
| **§1.B Expiry** – identify near-expiry; configure policies; recommend discounts; prevent expired sales; record expired & wastage | FR-BR-14; 26 + FR-ADM-14; 27/28; 10/13; 29/30 |
| **§1.B Operations** – analytics; transfers; employee access; employee actions | FR-BR-31/32; 37; 38; 39 |
| **§1.C Central admin** – organizations; branches; managers; catalog; categories; pricing rules; consolidated inventory; branch revenue; compare; wastage; system settings; roles & permissions | FR-ADM-01; 02; 03; 04; 05; 06/07; 09; 10; 11; 12; 16; 13 |
| **App B1** Smart expiry | FR-BR-13, 14, 27, 28; FR-CUS-07 |
| **App B2** Forecasting | FR-BR-19, FR-ADM-18 |
| **App B3** Transfer | FR-BR-37 |
| **App B4** POS & billing | FR-BR-20–25 |
| **App B5** Loyalty | FR-CUS-16, 22, 23, 24; FR-ADM-19 |
| **App B6** Analytics | FR-BR-18, 32; FR-ADM-10, 11, 12, 17 |
| **App B7** Wastage tracking | FR-BR-29, 30; FR-ADM-15 |
| **App C** decisions | FR-CUS-10, 11; FR-BR-09, 10, 13, 27, 28, 37; FR-ADM-13, 14 |
| **App D** MVP items 1–11 | See [feature-roadmap.md §4](feature-roadmap.md#4-mvp-definition) |
| **§4 Business rules** | See [business-requirements.md §6](business-requirements.md#6-rule-to-requirement-traceability) |

---

## 7. Related Documents

- [business-requirements.md](business-requirements.md)
- [non-functional-requirements.md](non-functional-requirements.md)
- [user-roles-and-personas.md](user-roles-and-personas.md)
- [feature-roadmap.md](feature-roadmap.md)
- [scope-and-assumptions.md](scope-and-assumptions.md)
