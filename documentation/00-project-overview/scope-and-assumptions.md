# BakeFlow – Scope, Assumptions and Open Questions

> **Step:** 1.1 (owner-reviewed) · **Last updated:** step 1.2 · **Source:** [master-prompt.md](../master-prompt.md), Appendices A–D, owner decisions on OQ-01 to OQ-22

## Table of Contents

1. [In Scope](#1-in-scope)
2. [Out of Scope](#2-out-of-scope)
3. [Assumptions (AS-xx)](#3-assumptions-as-xx)
4. [Known Limitations](#4-known-limitations)
5. [Open Questions (OQ-xx)](#5-open-questions-oq-xx)
6. [Architect Suggestions](#6-architect-suggestions)
7. [Related Documents](#7-related-documents)

---

## 1. In Scope

Everything in master-prompt §1 (Customer Platform, Branch Management, Central Administration) and Appendix B, delivered according to [feature-roadmap.md](feature-roadmap.md):

- Multi-branch organization setup with role-based, branch-isolated access.
- Central product catalog with variants, categories, prices with history, and GST data (HSN codes with effective-dated rates).
- Batch-level inventory with FEFO allocation, stock ledger, adjustments, expiry handling and alerts.
- Production recording (MVP), planning and forecasting (later).
- POS billing with GST invoices, payment records, refunds through credit notes, and daily closing.
- Customer website: browse by branch, cart, checkout with reservation, Razorpay payment, pickup (delivery later), order tracking and cancellation.
- Configurable near-expiry discounts with approval.
- Wastage recording with reasons and cost.
- Inter-branch transfers (post-MVP).
- Audit trail, notifications, analytics, forecasting, loyalty and reviews (per roadmap).
- Documentation, automated tests, Docker-based deployment and CI/CD.

## 2. Out of Scope

These are **not** planned in the current roadmap. Any of them can be proposed later as a new module.

| # | Item | Reason |
|---|------|--------|
| OS-01 | Raw-material / ingredient inventory and recipe costing | Large separate domain. The `ProductionRecipe` entity in master-prompt §3.D is deferred (AS-08). |
| OS-02 | Supplier management and purchase orders | Not requested. Stock receipt records incoming goods without a PO (AS-08). |
| OS-03 | Central kitchen / factory distributing to branches | Each branch bakes its own products (AS-07). Can be modelled later as a "branch" that only transfers. |
| OS-04 | Custom or made-to-order products (birthday cake orders, pre-orders) | Online orders are fulfilled from existing stock (AS-10). |
| OS-05 | Scheduled pickup or delivery time slots | Same-day pickup only (AS-11). |
| OS-06 | Third-party delivery aggregators (Swiggy, Zomato) and delivery partner apps | Delivery by own staff (AS-12). |
| OS-07 | Offline POS mode | Requires local storage and sync conflict handling (AS-17). |
| OS-08 | Card terminal / UPI QR device integration at POS | Payments are recorded, not processed, at the POS (AS-18). |
| OS-09 | SMS, WhatsApp and mobile push notifications | In-app and email only (AS-20). |
| OS-10 | Native Android / iOS apps | Mobile-friendly web only (AS-29). |
| OS-11 | Multi-currency, multi-language UI, multiple time zones | AS-01, AS-21. |
| OS-12 | Payroll, attendance and HR | Not requested. |
| OS-13 | Full accounting (ledger, GST return filing, e-invoicing / IRN) | BakeFlow produces invoices and reports. Filing is done in accounting software. B2C retail invoices are generally outside e-invoicing. |
| OS-14 | Credit sales / customer "khata" accounts | Every bill is fully paid at the time of sale (AS-33). |
| OS-15 | Selling by weight (loose items priced per kg) | Integer unit quantities only (AS-39). |

## 3. Assumptions (AS-xx)

An assumption is something we treat as true without proof. Items marked **Confirmed** were confirmed or decided by the owner in the step 1.1 review. The resolved open question is shown in brackets.

### 3.1 Business and organization

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-01 | Single currency: **INR**. | Money fields would need a currency column. |
| AS-02 | **One organization** in the MVP, but the schema is multi-tenant ready: every business table has `organization_id` (NFR-34). | Low. Design already supports more. |
| AS-07 | **Each branch bakes its own products.** There is no central kitchen in the MVP. Bought-in goods are added through stock receipt. | Central kitchen would need transfer-based supply flows (OS-03). |
| AS-08 | No raw-material, recipe or supplier management. | New modules required (OS-01, OS-02). |
| AS-23 | A staff user can be assigned to **more than one branch** (for example a manager covering two). The role applies per assignment. | If strictly one branch per user, the model gets simpler. |
| AS-24 | Customer accounts belong to **one organization**. A customer can order from any branch of that organization. | A platform-wide customer identity would need a different model. |
| AS-29 | All user interfaces are **browser-based**. The POS runs in a desktop or tablet browser; the customer site is mobile-friendly web. | Native apps would add a separate roadmap. |
| AS-30 | **MVP volumes:** up to 10 branches, 50 staff, 1,000 products, 2,000 POS bills/day, 500 online orders/day, 30 POS terminals and 300 concurrent customer sessions at peak (NFR-32). | Larger volumes would bring Phase 4 forward. |

### 3.2 Tax, pricing and money

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-03 | **Confirmed (OQ-16).** All branches are in **one state (Tamil Nadu)**, so sales are intra-state and taxed as **CGST + SGST**. IGST fields exist (ADR-006) but are not used in the MVP. | Inter-state branches need place-of-supply logic and IGST. |
| AS-04 | **Confirmed (OQ-16).** The organization is a **regular GST registrant** (not composition scheme) with **one GSTIN for Tamil Nadu**. Each branch has its **own invoice series** (allowed under GST). | Composition scheme would issue a "bill of supply" with no GST. |
| AS-05 | **Confirmed (OQ-01).** GST rates are **data, never hard-coded**. Each product references an **HSN code**. Each HSN code has a GST rate history with **effective dates**, maintained by the admin. Invoice lines snapshot the rate in force at the time of sale. BakeFlow does not decide tax law. | – |
| AS-06 | **Confirmed (OQ-01).** Selling prices are **GST-inclusive** (the shelf price is what the customer pays). Tax is back-calculated per invoice line. | – |
| AS-25 | A **standard unit cost** per product is maintained by the admin (FR-ADM-08) and used for wastage cost, minimum margins and profitability. No actual cost from recipes. | Profitability figures are estimates only. |
| AS-33 | Every sale is **fully paid** at the time of sale (split payment allowed). No credit sales. | Would need receivables (OS-14). |
| AS-37 | **Confirmed (OQ-02, SG-08 accepted).** Refunds after an invoice is issued produce a **credit note** in a **separate gap-free series** per branch per financial year (for example `ANNC/2627/00012`). The original invoice is never edited ([ADR-010](../02-system-design/design-decisions.md#adr-010--gap-free-invoice-numbering-per-branch-per-financial-year)). | – |
| AS-49 | **Confirmed (OQ-07).** The organization base price is the default. Branch price overrides are allowed only within a ±% band per product or category set by ORG_ADMIN. **Post-MVP.** | – |
| AS-53 | **Confirmed (OQ-12).** **Discounts do not stack.** The single best eligible discount applies per line. A manual discount replaces an automatic one only with manager approval. | – |
| AS-55 | **Confirmed (OQ-14).** Refunds: BRANCH_MANAGER or ORG_ADMIN only, within **7 days** of the invoice, through the original payment method, always with a credit note. | – |
| AS-56 | **Confirmed (OQ-15).** When FEFO takes one line's quantity from batches with **different prices or discounts**, the invoice line is **split** per batch and price. | – |

### 3.3 Inventory, production and expiry

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-09 | **Every** sellable item is batch-tracked, including bought-in packaged goods (using the supplier's dates). | Non-batch items would need a second stock model. |
| AS-15 | Refunded or returned food is **never put back** into sellable stock. It is recorded as wastage with reason `RETURNED`. | Restocking would need a quality-check step. |
| AS-26 | A batch is **near-expiry** when its remaining shelf life is below a threshold configured per category (default proposal: **25 % of shelf life or 4 hours, whichever is greater**). | Only changes defaults. |
| AS-27 | **Online sale cut-off:** a batch is not offered online if it expires within **2 hours** (configurable), so online customers receive products with reasonable life left. Near-expiry batches are offered online only when a discount rule allows it (AS-52). | Customers could receive almost-expired items. |
| AS-28 | The **expiry status job runs every 5 minutes**. It is not the main safety mechanism; allocation-time checks are (ADR-013). | – |
| AS-39 | **Confirmed (OQ-19).** Stock quantities are **whole units** (pieces, packs). Items sold by weight are set up as fixed packs (for example "Cookies 250 g"). | Decimal quantities would be needed everywhere. |
| AS-50 | **Confirmed (OQ-08).** **Categories are organization-level only**, managed by ORG_ADMIN. Branches control only which categories are shown and in what order (FR-BR-04). | – |
| AS-51 | **Confirmed (OQ-09).** **Product variants are modelled from day one.** Every product has at least one variant (SKU). Price, HSN reference, batches and stock are held at variant level. | – |
| AS-52 | **Confirmed (OQ-11).** Each near-expiry discount rule has an "approval required" flag, **default ON** (branch manager approves), and an "apply online" flag, **default OFF**. A discount never takes the price below the minimum margin. | – |
| AS-54 | **Confirmed, changed by owner (OQ-13).** A **stock decrease** needs BRANCH_MANAGER approval **only when it exceeds 5 units or ₹500** (both thresholds configurable). Smaller decreases apply immediately. **Nobody approves their own adjustment.** **Every adjustment is audit-logged**, whether or not it needed approval. Whether the same thresholds apply to **increases** is open (OQ-23); until then the same thresholds apply. | – |
| AS-57 | **Confirmed (OQ-17).** For physical vs system batch mismatch at the POS, the MVP uses system FEFO plus shelf practice (older batch in front). A manager-only "sell from batch X" override with a reason is a later suggestion (SG-11). | – |

### 3.4 Orders, payments and customers

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-10 | Online orders are fulfilled **from existing stock** (made-to-stock). | Made-to-order needs a production link (OS-04). |
| AS-11 | **Pickup** means same-day pickup at the selected branch before branch closing. No time slots. | Slots need capacity planning (OS-05). |
| AS-12 | When delivery is enabled, it is done by the **bakery's own staff** within a delivery area per branch. | Aggregators need integrations (OS-06). |
| AS-13 | **Confirmed (OQ-10).** Checkout reservation lasts **15 minutes** (configurable per organization). If payment does not succeed in time, the reservation is released. | – |
| AS-14 | **Confirmed (OQ-03).** **Online order stock flow:** reserved at checkout → reservation kept after payment until handover → `SALE_OUT` and invoice issued **at handover** (pickup or delivery). Cancellation before handover only releases the reservation and refunds the payment; no credit note is needed. | – |
| AS-16 | **Walk-in POS customers do not need an account.** A phone number may be captured optionally. | – |
| AS-18 | At the POS, cash, UPI and card payments are **recorded only**. They are processed on the bakery's existing devices. | Device integration would be added (OS-08). |
| AS-19 | **Razorpay test mode** throughout development. The owner provides a live merchant account before go-live (ADR-007). | – |
| AS-31 | **Confirmed (OQ-04).** A customer may cancel an order with a full refund until it reaches **READY_FOR_PICKUP** (or OUT_FOR_DELIVERY). A branch may cancel any time before handover with a full refund. | – |
| AS-32 | **Confirmed with change (OQ-04).** **No-show:** a pickup order not collected by branch closing time is cancelled automatically by a job. Its stock is released (if still sellable) or recorded as wastage. The **no-show refund policy is a configurable organization setting** (FR-ADM-16), for example full refund, partial refund (a percentage) or no refund. **MVP default: full refund.** No-show counts per customer are tracked. | – |
| AS-36 | Reviews can be submitted only for products in the customer's **completed** orders. | – |
| AS-38 | After a successful payment, the reservation is **extended until the pickup deadline** (branch closing on the same day) instead of expiring after 15 minutes. | – |
| AS-48 | **Confirmed (OQ-05).** **Pickup only in the MVP.** Delivery (addresses, delivery area, fee, assignment) comes right after the MVP. | – |
| AS-58 | **Confirmed (OQ-20).** **Customer password reset by email is in MVP-B**, so simple SMTP email sending is part of MVP-B ([ADR-033](../02-system-design/design-decisions.md#adr-033--email-via-spring-mail-mailpit-in-development)). | – |

### 3.5 Platform, security and operations

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-17 | The POS needs an **internet connection**. No offline mode (OS-07). | A connectivity loss stops billing (see limitation L-01). |
| AS-20 | Notifications are **in-app and email** only (OS-09). | – |
| AS-21 | UI in **English**. Display time zone **Asia/Kolkata**. "Business day" is the IST calendar day. Financial year is **1 April – 31 March**. | – |
| AS-22 | **Confirmed (OQ-21).** A **fixed set of five roles** (SUPER_ADMIN, ORG_ADMIN, BRANCH_MANAGER, BRANCH_STAFF, CUSTOMER), backed by a permission table so custom roles can be added later. BRANCH_STAFF is **not** split into cashier and baker in the MVP. | A role editor would be needed earlier. |
| AS-34 | Product images are stored outside the database. **Decided in step 1.2:** local file storage behind an interface in development and MVP, S3-compatible storage later ([ADR-031](../02-system-design/design-decisions.md#adr-031--file-storage-behind-an-interface-local-disk-first)). | – |
| AS-35 | Financial and audit records are kept for **8 years** (NFR-31). | – |

### 3.6 Success measure assumptions

**Confirmed (OQ-18).** These are **targets to review after 3 months live**. Details are in [project-vision.md §7](project-vision.md#7-success-measures).

| ID | Measure | Target |
|----|---------|--------|
| AS-40 | Wastage reduction | Wastage value down **30 %** within 3 months of go-live versus a 4-week baseline |
| AS-41 | Stock accuracy | **≥ 98 %** of batch lines match physical counts |
| AS-42 | Expired products sold | **0** |
| AS-43 | POS speed | Server **< 500 ms**; 3-item bill by staff **< 30 s** |
| AS-44 | Overproduction | Unsold-at-close **< 10 %** of production per product per branch |
| AS-45 | Near-expiry sell-through | **≥ 50 %** of discounted near-expiry units sold before expiry |
| AS-46 | Forecast accuracy | MAPE **≤ 20 %** after 8 weeks of history |
| AS-47 | Online order reliability | **≥ 98 %** of paid orders fulfilled without stock-out cancellation |

---

## 4. Known Limitations

| ID | Limitation | Mitigation |
|----|-----------|------------|
| L-01 | No offline POS: an internet outage stops counter billing. | Reliable connection plus a mobile hotspot backup. Paper bills entered afterwards as back-dated sales by a manager (procedure defined in step 1.4). |
| L-02 | **System FEFO may differ from the physical item picked.** The system allocates the earliest-expiring batch, but staff may hand over an item from a newer tray. | Shelf practice and the batch shown on the packing list (AS-57). A manager FEFO override is a future suggestion (SG-11). |
| L-03 | Wastage cost and profitability use standard cost, not actual cost (AS-25). | Admin reviews standard costs periodically. |
| L-04 | Forecasting needs at least 8 weeks of sales history before it is useful (AS-46). | Production plans stay manual until then. |
| L-05 | One organization in the MVP. Multi-organization onboarding UI is minimal. | Schema is ready (NFR-34). |
| L-06 | A single application instance in the MVP is a single point of failure. | Stateless design (NFR-35) allows several instances later. Backups (NFR-09). |

---

## 5. Open Questions (OQ-xx)

**Status values:** `Resolved` (the owner decided; the final answer is recorded in the referenced document) · `Open` (waiting for the owner; documents follow the recommended answer until decided).

### 5.1 Resolved in the step 1.1 review

| ID | Question | Status | Final answer | Recorded in |
|----|----------|--------|--------------|-------------|
| OQ-01 | GST-inclusive or exclusive prices? | Resolved | **Inclusive.** Tax is back-calculated per line. The GST rate is stored per HSN code with **effective dates** and is never hard-coded; rates can change. | AS-05, AS-06, ADR-006, FR-ADM-06 |
| OQ-02 | Invoice number format (GST 16-character limit) | Resolved | **`ANN/2627/000123`** (15 characters, maximum 16). Credit notes use a **separate series**, for example `ANNC/2627/00012`. | ADR-010, AS-37 |
| OQ-03 | When is an online order sold and invoiced? | Resolved | Reserve at checkout; keep the reservation after payment; record `SALE_OUT` and issue the invoice **at handover**. | AS-14 |
| OQ-04 | Cancellation and no-show policy | Resolved | Customer cancels with a full refund until READY_FOR_PICKUP. No-shows are auto-cancelled at closing. The **no-show refund policy is a configurable organization setting**, default full refund in the MVP. | AS-31, AS-32, FR-ADM-16 |
| OQ-05 | Delivery in the MVP? | Resolved | **No.** Pickup only. | AS-48 |
| OQ-06 | Two-stage MVP? | Resolved | **Yes.** MVP-A (branch operations), then MVP-B (online ordering). | [feature-roadmap.md §4](feature-roadmap.md#4-mvp-definition), FR MVP labels |
| OQ-07 | Branch price overrides | Resolved | Within a ±% band set by ORG_ADMIN; post-MVP. | AS-49, FR-ADM-07, FR-BR-03 |
| OQ-08 | Category ownership | Resolved | Organization-level only; branches control visibility and order. | AS-50, FR-ADM-05, FR-BR-04 |
| OQ-09 | Product variants in the MVP? | Resolved | **Yes, from day one.** | AS-51, FR-ADM-04 |
| OQ-10 | 15-minute checkout reservation | Resolved | **Yes**, configurable. | AS-13 |
| OQ-11 | Near-expiry discount approval and online display | Resolved | Approval flag default ON; online flag default OFF; never below minimum margin. | AS-52, FR-ADM-14 |
| OQ-12 | Discount stacking | Resolved | **No stacking.** | AS-53 |
| OQ-13 | Stock adjustment approval | Resolved (**changed**) | Decreases need manager approval **only above 5 units or ₹500** (configurable). No self-approval. All adjustments are always audit-logged. | AS-54, FR-BR-11 |
| OQ-14 | Refund rules | Resolved | Manager or org admin only, within 7 days, original payment method, credit note always. | AS-55, FR-BR-23 |
| OQ-15 | Mixed-price batches in one line | Resolved | Split the invoice line. | AS-56 |
| OQ-16 | GST registration | Resolved | Regular registration, single GSTIN in Tamil Nadu, invoice series per branch. | AS-03, AS-04, ADR-006 |
| OQ-17 | Physical vs system batch mismatch | Resolved | System FEFO plus shelf practice in the MVP; manager override later. | AS-57, L-02 |
| OQ-18 | Success measures | Resolved | Accepted as **targets to review after 3 months live**. | AS-40 – AS-47, project-vision.md §7 |
| OQ-19 | Whole units vs weight | Resolved | Whole units; loose items as fixed packs. | AS-39 |
| OQ-20 | Customer password reset in the MVP? | Resolved | **Yes, in MVP-B.** | AS-58, FR-CUS-03 |
| OQ-21 | Split BRANCH_STAFF into CASHIER and BAKER? | Resolved | **Not in the MVP.** | AS-22 |
| OQ-22 | Pessimistic lock **and** `@Version` on batches | Resolved | **Keep both.** Pessimistic locks for quantity changes, `@Version` for other batch updates. | ADR-008 |

### 5.2 Open (raised in step 1.2)

| ID | Question | Recommended answer | Affects |
|----|----------|--------------------|---------|
| OQ-23 | Your OQ-13 decision covers stock **decreases**. Should **increases** above 5 units or ₹500 also need manager approval? | **Yes, use the same thresholds.** Unapproved large increases could hide theft (for example inflating stock before recording a "sale" that never happened) and distort stock accuracy. | AS-54, FR-BR-11 |
| OQ-24 | The brief specifies **Java 21**. **Java 25** has been an LTS release since September 2025 and is supported by Spring Boot 4. Stay on 21? | **Stay on Java 21 for now**, as specified. Re-evaluate at Phase 5 (Java 25 brings performance gains but no feature BakeFlow needs). | [ADR-016](../02-system-design/design-decisions.md#adr-016--spring-boot-41x-on-java-21-lts) |
| OQ-25 | **Spring Boot 4.1.x** (mature, latest patch 4.1.3) or **4.2.0** (released 19-Nov-2026, five days ago)? | **4.1.x now**, upgrade to 4.2.x before Phase 5 once springdoc-openapi and Spring Modulith publish releases confirmed for 4.2. | ADR-016 |
| OQ-26 | Is your Mac **personal** or **company-managed**? Docker Desktop is free for personal use but needs a paid subscription for commercial use in larger organizations. | Personal Mac: **Docker Desktop**. Company-managed Mac: **Colima** (free, open source), or check your company's Docker licence. | [development-environment.md](../01-technology-stack/development-environment.md) |

---

## 6. Architect Suggestions

Features or rules **not explicitly in master-prompt.md** that this documentation adds or proposes.

| ID | Suggestion | Where used | Status |
|----|-----------|-----------|--------|
| SG-01 | Customer password reset by email | FR-CUS-03 | Proposed – pending owner review (note: OQ-20 places FR-CUS-03 in MVP-B) |
| SG-02 | Customer can view and download the invoice | FR-CUS-25 | Proposed – pending owner review |
| SG-03 | Batch block / quality hold | FR-BR-34 | Proposed – pending owner review |
| SG-04 | Standard unit cost per product, needed for wastage cost, minimum margin (App C) and profitability (App B6) | FR-ADM-08 | Proposed – pending owner review |
| SG-05 | Review moderation | FR-ADM-21 | Proposed – pending owner review |
| SG-06 | Customer account administration | FR-ADM-22 | Proposed – pending owner review |
| SG-07 | Read-only accountant / auditor role | Roles §6 | Proposed – pending owner review |
| SG-08 | Credit notes for refunds, with their own gap-free series | FR-BR-23, AS-37, ADR-010 | **Accepted** |
| SG-09 | Cash reconciliation (expected vs counted) in daily closing | FR-BR-25 | Proposed – pending owner review |
| SG-10 | Online sale cut-off before expiry | FR-CUS-07, AS-27 | Proposed – pending owner review |
| SG-11 | Manager override of FEFO with a reason | L-02, AS-57 | Proposed – pending owner review |
| SG-12 | Nightly ledger-vs-batch reconciliation check | NFR-22 | Proposed – pending owner review |
| SG-13 | Online order fulfilment screen for branches (implied by online ordering) | FR-BR-35 | Proposed – pending owner review |

---

## 7. Related Documents

- [project-vision.md](project-vision.md)
- [business-requirements.md](business-requirements.md)
- [functional-requirements.md](functional-requirements.md)
- [non-functional-requirements.md](non-functional-requirements.md)
- [feature-roadmap.md](feature-roadmap.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
- [technology-stack.md](../01-technology-stack/technology-stack.md)
