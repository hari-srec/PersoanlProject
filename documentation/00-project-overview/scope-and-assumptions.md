# BakeFlow – Scope, Assumptions and Open Questions

> **Step:** 1.1 · **Status:** Draft for owner review · **Source:** [master-prompt.md](../master-prompt.md), Appendices A–D

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
- Central product catalog, categories, prices with history, GST data.
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
| OS-15 | Selling by weight (loose items priced per kg) | Integer unit quantities only in the MVP (AS-39, OQ-19). |

## 3. Assumptions (AS-xx)

An assumption is something we treat as true without proof. If any is wrong, the design may need to change, so **please confirm or correct each one**.

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
| AS-03 | All branches are in **one state (Tamil Nadu)**, so sales are intra-state and taxed as **CGST + SGST**. IGST fields exist (ADR-006) but are not used in the MVP. | Inter-state branches need place-of-supply logic and IGST. |
| AS-04 | The organization is a **regular GST registrant** (not composition scheme) with one GSTIN for the state. Each branch has its **own invoice series** (allowed under GST). | Composition scheme issues a "bill of supply" with no GST charged; invoices would change (OQ-16). |
| AS-05 | The admin configures HSN code and GST rate per product. BakeFlow does **not** decide tax law (rates for bread, cakes and so on vary). | – |
| AS-06 | Selling prices are **GST-inclusive** (the shelf price is what the customer pays). Tax is back-calculated on the invoice (OQ-01). | Exclusive prices change every price calculation and display. |
| AS-25 | A **standard unit cost** per product is maintained by the admin (FR-ADM-08) and used for wastage cost, minimum margins and profitability. No actual cost from recipes. | Profitability figures are estimates only. |
| AS-33 | Every sale is **fully paid** at the time of sale (split payment allowed). No credit sales. | Would need receivables (OS-14). |
| AS-37 | Refunds after an invoice is issued produce a **credit note** in a separate gap-free series per branch per financial year (format proposed in OQ-02). The original invoice is never edited (ADR-010). | – |

### 3.3 Inventory, production and expiry

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-09 | **Every** sellable item is batch-tracked, including bought-in packaged goods (using the supplier's dates). | Non-batch items would need a second stock model. |
| AS-15 | Refunded or returned food is **never put back** into sellable stock. It is recorded as wastage with reason `RETURNED`. | Restocking would need a quality-check step. |
| AS-26 | A batch is **near-expiry** when its remaining shelf life is below a threshold configured per category (default proposal: **25 % of shelf life or 4 hours, whichever is greater**). | Only changes defaults. |
| AS-27 | **Online sale cut-off:** a batch is not offered online if it expires within **2 hours** (configurable), so online customers receive products with reasonable life left. Near-expiry batches can still be offered online when a discount rule allows it (OQ-11). | Customers could receive almost-expired items. |
| AS-28 | The **expiry status job runs every 5 minutes**. It is not the main safety mechanism; allocation-time checks are (ADR-013). | – |
| AS-39 | Stock quantities are **whole units** (pieces, packs). Items sold by weight are set up as fixed packs (for example "Cookies 250 g"). | Decimal quantities would be needed everywhere (OQ-19). |

### 3.4 Orders, payments and customers

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-10 | Online orders are fulfilled **from existing stock** (made-to-stock). | Made-to-order needs a production link (OS-04). |
| AS-11 | **Pickup** means same-day pickup at the selected branch before branch closing. No time slots. | Slots need capacity planning (OS-05). |
| AS-12 | When delivery is enabled, it is done by the **bakery's own staff** within a delivery area per branch. | Aggregators need integrations (OS-06). |
| AS-13 | Checkout reservation lasts **15 minutes** (configurable). If payment does not succeed in time, the reservation is released (OQ-10). | Only changes the default. |
| AS-14 | **Online order stock flow:** reserved at checkout → reservation kept after payment until handover → `SALE_OUT` and invoice issued **at handover** (pickup or delivery). Cancellation before handover only releases the reservation and refunds the payment; no credit note is needed (OQ-03). | Changes order, invoice and refund design. |
| AS-16 | **Walk-in POS customers do not need an account.** A phone number may be captured optionally. | – |
| AS-18 | At the POS, cash, UPI and card payments are **recorded only**. They are processed on the bakery's existing devices. | Device integration would be added (OS-08). |
| AS-19 | **Razorpay test mode** throughout development. The owner provides a live merchant account before go-live (ADR-007). | – |
| AS-31 | A customer may cancel an order until it reaches **READY_FOR_PICKUP** (or OUT_FOR_DELIVERY), with a full refund (OQ-04). | Changes the cancellation policy only. |
| AS-32 | **No-show:** a pickup order not collected by branch closing time is cancelled automatically by a job. Its stock is released (if still sellable) or wasted, and the refund follows the no-show policy (OQ-04). | – |
| AS-36 | Reviews can be submitted only for products in the customer's **completed** orders. | – |
| AS-38 | After a successful payment, the reservation is **extended until the pickup deadline** (branch closing on the same day) instead of expiring after 15 minutes. | – |

### 3.5 Platform, security and operations

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| AS-17 | The POS needs an **internet connection**. No offline mode (OS-07). | A connectivity loss stops billing (see limitation L-01). |
| AS-20 | Notifications are **in-app and email** only (OS-09). | – |
| AS-21 | UI in **English**. Display time zone **Asia/Kolkata**. "Business day" is the IST calendar day. Financial year is **1 April – 31 March**. | – |
| AS-22 | A **fixed set of five roles** (SUPER_ADMIN, ORG_ADMIN, BRANCH_MANAGER, BRANCH_STAFF, CUSTOMER), backed by a permission table so custom roles can be added later. | A role editor would be needed earlier. |
| AS-34 | Product images are stored outside the database (file or object storage). Choice made in step 1.2. | – |
| AS-35 | Financial and audit records are kept for **8 years** (NFR-31). | – |

### 3.6 Success measure assumptions

The brief defines no numeric targets. These are proposed in [project-vision.md §7](project-vision.md#7-success-measures). **Please confirm or adjust.**

| ID | Measure | Proposed target |
|----|---------|-----------------|
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
| L-02 | **System FEFO may differ from the physical item picked.** The system allocates the earliest-expiring batch, but staff may hand over an item from a newer tray. | Shelf practice (older batch in front) and the batch shown on the packing list. A manager FEFO override is a future suggestion (SG-11, OQ-17). |
| L-03 | Wastage cost and profitability use standard cost, not actual cost (AS-25). | Admin reviews standard costs periodically. |
| L-04 | Forecasting needs at least 8 weeks of sales history before it is useful (AS-46). | Production plans stay manual until then. |
| L-05 | One organization in the MVP. Multi-organization onboarding UI is minimal. | Schema is ready (NFR-34). |
| L-06 | A single application instance in the MVP is a single point of failure. | Stateless design (NFR-35) allows several instances later. Backups (NFR-09). |

---

## 5. Open Questions (OQ-xx)

Each question has a **recommended answer**. Until you decide, the design uses the recommendation.

| ID | Question | Recommended answer | Affects |
|----|----------|--------------------|---------|
| OQ-01 | Are selling prices **GST-inclusive** or exclusive? | **Inclusive.** This is normal Indian retail practice: the shelf price equals the price paid, and tax is back-calculated per line. | AS-06, 1.3, 3.3, 3.8 |
| OQ-02 | The ADR-010 example `ANN/2026-27/000123` is **18 characters**, but GST rules limit an invoice number to **16 characters**. Which format? | **`ANN/2627/000123`** (15 characters): branch code (3), FY as `2627`, 6-digit counter. Credit notes: **`ANNC/2627/00012`** (15 characters). | ADR-010, 3.8 |
| OQ-03 | For online orders, **when is stock counted as sold and when is the invoice issued**? | Reserve at checkout; keep the reservation after payment; record `SALE_OUT` and issue the invoice **at handover** (AS-14). Cancellation before handover needs no credit note. Under GST, tax on goods is due at supply, not on advance receipt. | AS-14, 3.10, 3.11, 3.8 |
| OQ-04 | Cancellation and **no-show** policy? | Customer can cancel with a full refund until READY_FOR_PICKUP. Branch can cancel any time before handover with a full refund. No-shows are auto-cancelled at closing with a **full refund in the MVP**; no-show counts are tracked so a penalty can be introduced later. | AS-31, AS-32, 3.11 |
| OQ-05 | Is **delivery** part of the MVP? | **No.** Pickup only in the MVP. Delivery (addresses, area, fee, assignment) comes right after. | FR-CUS-13, 21, FR-BR-36 |
| OQ-06 | Should the MVP be released in **two stages** (MVP-A branch operations, MVP-B online ordering)? | **Yes.** It reduces risk for a single developer and delivers wastage visibility earlier. See [feature-roadmap.md §4.2](feature-roadmap.md#42-recommended-staged-mvp-for-a-single-developer). | Roadmap |
| OQ-07 | How may branches **override prices**? | Organization base price is the default. Branch override allowed within a ±% band per product or category set by ORG_ADMIN. **Post-MVP.** | FR-ADM-07, FR-BR-03 |
| OQ-08 | The master prompt lets both branches and central admin "manage categories". Who owns categories? | **Organization-level only (ORG_ADMIN).** Branches control visibility and display order (FR-BR-04). This avoids duplicate or conflicting category trees. | FR-ADM-05, FR-BR-04 |
| OQ-09 | Should **product variants** (for example Black Forest Cake 500 g / 1 kg) exist in the MVP? | **Yes, modelled from day one.** Every product has at least one variant (SKU). Price, batches and stock are kept at variant level. Adding variants later would mean migrating batches, prices and invoices. | 1.3, 3.3 onward |
| OQ-10 | Is a **15-minute** checkout reservation right? | **Yes**, configurable per organization (AS-13). | 3.10 |
| OQ-11 | Should near-expiry discounts be **auto-applied or approved**, and shown **online**? | Each rule has an "approval required" flag, **default ON** (manager approves). Discounts never go below the minimum margin. Online display is a per-rule flag, **default OFF**, so online customers mostly get fresh stock. | FR-ADM-14, FR-BR-27/28, AS-27 |
| OQ-12 | Can discounts **stack** (near-expiry + manual)? | **No stacking.** The single best eligible discount applies per line. A manual discount replaces an automatic one only with manager approval. | 3.12 |
| OQ-13 | What **stock adjustment** needs approval? | All decreases and any adjustment above **5 units or ₹500** need BRANCH_MANAGER approval. Nobody approves their own adjustment. Thresholds configurable. | FR-BR-11 |
| OQ-14 | **Refund** rules? | BRANCH_MANAGER or ORG_ADMIN only. Within **7 days** of the invoice. Refund through the original payment method. Credit note always issued. | FR-BR-23, AS-37 |
| OQ-15 | If FEFO takes one line's quantity from a **discounted batch and a full-price batch**, how is it priced? | **Split the invoice line** per batch and price, so each line shows its own discount. The customer sees a clear breakdown. | 3.8, 3.12 |
| OQ-16 | Please confirm the **GST registration type** and GSTIN set-up (one GSTIN for Tamil Nadu, invoice series per branch). | Regular registration, one GSTIN, series per branch (AS-04). | ADR-006, 3.8 |
| OQ-17 | How to handle **physical vs system batch mismatch** at the POS (L-02)? | MVP: system FEFO plus shelf practice. Later: a manager-only "sell from batch X" override with a reason (SG-11). | 3.5, 3.8 |
| OQ-18 | Do you accept the proposed **success measures** AS-40 to AS-47? | Accept as starting targets and review after 3 months of live data. | Vision |
| OQ-19 | Are **whole-unit quantities** enough, or must some items be sold **by weight**? | Whole units in the MVP (AS-39). Loose weight items become fixed packs. | 1.3 (quantity data types) |
| OQ-20 | Should **customer password reset by email** be in the MVP? | **Yes, with MVP-B.** Customers cannot be reset manually by staff easily. This needs simple SMTP email sending in MVP-B. FR-CUS-03 is marked MVP on this basis. | FR-CUS-03 |
| OQ-21 | Should BRANCH_STAFF later be **split** into CASHIER and BAKER? | Not in the MVP. Revisit after branch feedback. | Roles |
| OQ-22 | Is **ADR-008** (pessimistic lock **and** `@Version`) wanted on the same rows? | Keep both, as accepted. Pessimistic locks protect allocation. `@Version` protects other batch updates (for example blocking, metadata edits) that do not take a row lock. | ADR-008 |

---

## 6. Architect Suggestions

Features or rules **not explicitly in master-prompt.md** that this documentation adds or proposes. They are listed here so the owner can accept or reject them.

| ID | Suggestion | Where used | Status |
|----|-----------|-----------|--------|
| SG-01 | Customer password reset by email | FR-CUS-03 | Proposed (OQ-20) |
| SG-02 | Customer can view and download the invoice | FR-CUS-25 | Proposed |
| SG-03 | Batch block / quality hold | FR-BR-34 | Proposed (MVP) |
| SG-04 | Standard unit cost per product, needed for wastage cost, minimum margin (App C) and profitability (App B6) | FR-ADM-08 | Proposed (MVP) |
| SG-05 | Review moderation | FR-ADM-21 | Proposed |
| SG-06 | Customer account administration | FR-ADM-22 | Proposed |
| SG-07 | Read-only accountant / auditor role | Roles §6 | Idea only |
| SG-08 | Credit notes for refunds, with their own gap-free series | FR-BR-23, AS-37 | Proposed (MVP) – required for GST-correct refunds |
| SG-09 | Cash reconciliation (expected vs counted) in daily closing | FR-BR-25 | Proposed (MVP) |
| SG-10 | Online sale cut-off before expiry | FR-CUS-07, AS-27 | Proposed (MVP) |
| SG-11 | Manager override of FEFO with a reason | L-02, OQ-17 | Idea only |
| SG-12 | Nightly ledger-vs-batch reconciliation check | NFR-22 | Proposed (Phase 4) |
| SG-13 | Online order fulfilment screen for branches (implied by online ordering) | FR-BR-35 | Proposed (MVP) |

---

## 7. Related Documents

- [project-vision.md](project-vision.md)
- [business-requirements.md](business-requirements.md)
- [functional-requirements.md](functional-requirements.md)
- [non-functional-requirements.md](non-functional-requirements.md)
- [feature-roadmap.md](feature-roadmap.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
