# BakeFlow – Business Requirements

> **Step:** 1.1 (owner-reviewed) · **Source:** [master-prompt.md](../master-prompt.md) §1, §4, Appendices A–C

## Table of Contents

1. [Purpose](#1-purpose)
2. [Business Goals](#2-business-goals)
3. [Key Concepts and Glossary](#3-key-concepts-and-glossary)
4. [Distinguishing Stock Operations](#4-distinguishing-stock-operations)
5. [Business Rules BR-01 to BR-15](#5-business-rules-br-01-to-br-15)
6. [Rule-to-Requirement Traceability](#6-rule-to-requirement-traceability)
7. [Related Documents](#7-related-documents)

---

## 1. Purpose

This document lists the **business goals** BakeFlow must achieve and the **business rules** it must always respect.

- A **business goal** describes an outcome the business wants.
- A **business rule** is a constraint that must never be broken, whichever screen or API is used.

Later documents (database design, APIs, workflows, tests) refer back to these IDs.

---

## 2. Business Goals

| ID | Goal | Linked problem ([project-vision.md](project-vision.md#2-problem-statement)) | Success measure |
|----|------|------|-----|
| BG-01 | Reduce wastage caused by expiry and overproduction | P1, P2 | AS-40, AS-44, AS-45 |
| BG-02 | Know the exact stock of every batch in every branch at any time | P3 | AS-41 |
| BG-03 | Never sell expired products, whether at the counter or online | P1 | AS-42 |
| BG-04 | Bill counter customers quickly with GST-compliant invoices | – | AS-43 |
| BG-05 | Plan each day's production from real sales data | P2 | AS-44, AS-46 |
| BG-06 | Give central management same-day visibility of sales, stock and wastage across branches | P4 | – |
| BG-07 | Open an online ordering channel that only sells stock that really exists | – | AS-47 |
| BG-08 | Keep a trustworthy history of who changed what, for accountability | – | – |

---

## 3. Key Concepts and Glossary

| Term | Plain-language meaning |
|------|------------------------|
| **Organization** | A bakery business (for example "Chennai Delights Bakery") that owns one or more branches. |
| **Branch** | One physical shop or outlet of the organization, such as Anna Nagar. |
| **Product** | An item in the central catalog, for example Chocolate Pastry. |
| **Batch** | A group of units of one product made or received together. A batch shares one manufacturing time and one expiry time. |
| **FEFO** | *First Expired, First Out*. When selling, take stock from the batch that expires soonest. |
| **On-hand quantity** | Units physically present in the branch for a batch. |
| **Reserved quantity** | On-hand units that are held for a customer order and must not be sold to anyone else. |
| **Available quantity** | Units that can still be sold: *on-hand − reserved*, and only for batches that are not expired or blocked. |
| **Near-expiry** | A batch whose remaining shelf life has fallen below a configured threshold. |
| **Stock movement ledger** | An append-only history of every quantity change. Rows are never edited or deleted. |
| **POS** | Point of Sale: the counter billing screen. |
| **GST / HSN** | India's Goods and Services Tax / Harmonized System of Nomenclature code used to classify goods for tax. |
| **Credit note** | A document issued when an invoice is partly or fully refunded. The original invoice is never changed. |

---

## 4. Distinguishing Stock Operations

The master prompt requires that manufacturing, stock receipt, reservation, sale and wastage **must not be treated as the same operation**. This step adds transfer and adjustment to that list.

Each operation has its own meaning, trigger, permission and ledger record. This prevents mistakes such as recording an expired pastry as a "sale with zero price", or a recount as "production".

| Operation | What it means | Who does it (typical) | Effect on batch quantities | Ledger movement (indicative name, finalised in step 1.3) | Module |
|-----------|---------------|------------------------|----------------------------|---------------------------------------|--------|
| **Manufacturing (production)** | The branch bakes products. A **new batch** is created with manufacturing and expiry date-time. | Branch staff (baker) | New batch: on-hand = produced quantity | `PRODUCTION_IN` | 3.7 |
| **Stock receipt** | Goods arrive from outside (for example packaged drinks or bought-in items) and a new batch is created with the supplier's dates. This is not baking. | Branch staff | New batch: on-hand = received quantity | `RECEIPT_IN` | 3.4 |
| **Stock reservation** | Units are held for a customer order during checkout and until handover. They stay on the shelf but cannot be sold to anyone else. | System (at checkout) | reserved ↑, available ↓, on-hand unchanged | `RESERVE` / `RESERVATION_RELEASE` | 3.5, 3.10 |
| **Sale** | Units leave the branch to a customer, through a POS bill or an online order handover. | Branch staff / system | on-hand ↓ (and reserved ↓ if the units were reserved) | `SALE_OUT` | 3.8, 3.11 |
| **Transfer** | Units move from one branch to another in **two separate steps**: dispatch at the source and receipt at the destination. | Branch managers | Source on-hand ↓ at dispatch; destination batch on-hand ↑ at receipt. Any shortfall is recorded as transit loss. | `TRANSFER_OUT` / `TRANSFER_IN` | 3.14 |
| **Adjustment** | Corrects the system to match a physical count (for example a miscount or data-entry error). It is not a loss event. | Staff or manager records; manager approves decreases above 5 units / ₹500 (AS-54) | on-hand ↑ or ↓ with a mandatory reason | `ADJUSTMENT_IN` / `ADJUSTMENT_OUT` | 3.6 |
| **Wastage** | Units are destroyed or discarded because they expired, were damaged, failed quality checks or were returned. They carry a **cost**. | Branch staff | on-hand ↓, wastage cost recorded | `WASTAGE_OUT` | 3.13 |

**Key clarifications**

- **Expiry is a status change, not a movement.** When a batch passes its expiry time it becomes unsellable straight away. The units stay on-hand until staff physically discard them and record **wastage** with reason `EXPIRED`.
- **Refunds do not put food back on the shelf.** Refunded items are recorded as wastage with reason `RETURNED` (AS-15).
- **The cart is not a reservation.** Adding to cart changes nothing in inventory ([ADR-012](../02-system-design/design-decisions.md#adr-012--cart-is-not-a-reservation-time-limited-reservation-at-checkout)).

---

## 5. Business Rules BR-01 to BR-15

Each rule below gives the rule text from the master prompt, why it matters and how BakeFlow enforces it.

### BR-01 – Independent inventory per branch

**Rule:** Every branch must maintain independent inventory.

**Why it matters:** Anna Nagar's stock cannot be sold at Velachery. Customers must see the stock of the branch they selected, and each manager is accountable for their own shelf.

**How:** Every batch belongs to exactly one branch. Stock can only move between branches through a transfer (BR-11).

### BR-02 – Production batches carry manufacturing and expiry information

**Rule:** Every production batch must have manufacturing and expiry information.

**Why it matters:** FEFO, near-expiry discounts and expiry blocking all depend on these dates. A batch without an expiry time could be sold forever.

**How:** Manufacturing date-time and expiry date-time are mandatory, and expiry must be later than manufacturing. Expiry can default from the product's configured shelf life but can be overridden. The same rule applies to received batches (AS-09).

### BR-03 – Batch-wise stock tracking

**Rule:** Stock must be tracked batch-wise.

**Why it matters:** Two batches of the same pastry can expire on different days. Product-level totals hide this and cause waste.

**How:** All quantities (on-hand, reserved, sold, wasted) are kept per batch. Product totals are calculated from batches ([ADR-002](../02-system-design/design-decisions.md#adr-002--batch-level-inventory-with-fefo-allocation)).

### BR-04 – FEFO allocation

**Rule:** The system should prioritise FEFO when allocating stock.

**Why it matters:** Selling the oldest stock first reduces expiry losses.

**How:** When stock is allocated for a POS bill or an online order, eligible batches are picked in order of earliest expiry. One sale line can draw from several batches.

### BR-05 – Expired products must not be sold

**Rule:** Expired products must not be sold.

**Why it matters:** This is food safety and a legal and reputational risk. It is the most important rule in the system.

**How:** Expiry is checked **at the moment of allocation**, at POS billing and at checkout, as well as by a scheduled job that updates batch status ([ADR-013](../02-system-design/design-decisions.md#adr-013--expiry-validated-at-allocation-time-not-only-by-scheduled-jobs)). Expired batches are removed from online availability automatically.

### BR-06 – Near-expiry discounts follow configurable policies

**Rule:** Near-expiry discounts must follow configurable business policies.

**Why it matters:** Discounting everything near expiry would cut margins needlessly. The business decides which products, how much, the minimum margin and whether approval is needed.

**How:** Discount rules are configured by remaining shelf life, eligible products or categories, maximum discount, minimum margin and an approval flag. The system **recommends** discounts. It applies them automatically only when a rule explicitly allows it.

### BR-07 – Discounts never change the original price

**Rule:** Discounts must not modify the original product price.

**Why it matters:** If a discount overwrote the price, the true price would be lost, reports would be wrong and the discount could "stick" after it ended.

**How:** Discounts are stored as separate records. Invoice lines store the original price, the discount amount and the final price separately ([ADR-014](../02-system-design/design-decisions.md#adr-014--discounts-stored-separately-original-price-never-modified)).

### BR-08 – Every inventory change is recorded

**Rule:** Inventory changes must be recorded in stock movement history.

**Why it matters:** Without history, stock differences cannot be explained or audited.

**How:** Every quantity change writes a row to the append-only stock movement ledger, with type, quantity, batch, user, time and reference document ([ADR-015](../02-system-design/design-decisions.md#adr-015--append-only-stock-movement-ledger-for-every-quantity-change)).

### BR-09 – Orders reserve stock safely

**Rule:** Orders must reserve stock safely.

**Why it matters:** Without a reservation, an item could be sold at the counter while an online customer is paying for it. The customer would then pay for something that no longer exists.

**How:** A time-limited reservation is created at checkout, inside a transaction with row locking ([ADR-008](../02-system-design/design-decisions.md#adr-008--pessimistic-locking-for-stock-allocation-plus-optimistic-locking-on-batches), [ADR-012](../02-system-design/design-decisions.md#adr-012--cart-is-not-a-reservation-time-limited-reservation-at-checkout)). Reserved units cannot be allocated to anyone else.

### BR-10 – Cancelled orders release eligible reserved stock

**Rule:** Cancelled orders must release eligible reserved stock.

**Why it matters:** Stock stuck in dead reservations becomes unsellable and is eventually wasted.

**How:** Cancellation, payment failure and reservation timeout all release the reservation. Units go back to *available* only if their batch is still sellable (not expired or blocked). Otherwise they stay unavailable and are handled as expiry.

### BR-11 – Transfers record dispatch and receipt separately

**Rule:** Stock transfers must record dispatch and receipt separately.

**Why it matters:** Goods in a delivery van belong to neither shelf. If dispatch and receipt were one step, losses in transit would be invisible.

**How:** A transfer goes through Requested → Dispatched (source stock ↓, goods *in transit*) → Received (destination stock ↑, quantity actually received). Any difference is recorded as a transit loss. Batch identity and expiry are kept across the transfer.

### BR-12 – Critical inventory operations are transactional

**Rule:** All critical inventory operations must be transactional.

**Why it matters:** A half-finished operation (for example a ledger row written but batch quantity not updated) corrupts stock figures.

**How:** Allocation, reservation, sale, transfer, adjustment and wastage each run in a single database transaction. Either everything is saved or nothing is.

### BR-13 – No negative inventory

**Rule:** Prevent negative inventory.

**Why it matters:** Negative stock means the system has sold units that do not exist. This is always an error.

**How:** Validation runs before every reduction, under a row lock, and database CHECK constraints act as a final safety net ([ADR-008](../02-system-design/design-decisions.md#adr-008--pessimistic-locking-for-stock-allocation-plus-optimistic-locking-on-batches)).

### BR-14 – Verify organization and branch access

**Rule:** Every branch-specific operation must verify organization and branch access.

**Why it matters:** A T. Nagar employee must never see or change Anna Nagar data, even by editing a branch ID in an API request.

**How:** On every request, the server checks the user's organization and branch assignments against the target resource. The client is never trusted. Details are in step 1.7 security documentation.

### BR-15 – Audit history for important changes

**Rule:** Maintain an audit history for important business changes.

**Why it matters:** This provides accountability (who gave that discount?), fraud detection and dispute resolution.

**How:** Important actions are written to an audit log with user, time, action, entity and before/after values. Examples are price changes, discount approvals, refunds, adjustments, wastage, user and role changes, and transfers. The audit framework (`audit_log` table and capture mechanism) is built in **Phase 2**, recording the actor as `system`. Module **3.1** connects it to the real authenticated user, so every MVP module is audited from the start. Every stock adjustment is audited, whether or not it needed approval (AS-54). See [feature-roadmap.md](feature-roadmap.md#phase-2--tech-stack--project-setup).

---

## 6. Rule-to-Requirement Traceability

| Rule | Functional requirements | ADRs | Main module(s) |
|------|-------------------------|------|----------------|
| BR-01 | FR-BR-05, FR-BR-06, FR-BR-37 | ADR-002 | 3.4 |
| BR-02 | FR-BR-05, FR-BR-15 | ADR-002 | 3.4, 3.7 |
| BR-03 | FR-BR-06 | ADR-002 | 3.4 |
| BR-04 | FR-BR-08, FR-BR-20, FR-CUS-11 | ADR-002 | 3.5 |
| BR-05 | FR-BR-10, FR-BR-13, FR-BR-14, FR-CUS-07 | ADR-013 | 3.5, 3.6 |
| BR-06 | FR-BR-27, FR-BR-28, FR-ADM-14 | ADR-014 | 3.12 |
| BR-07 | FR-BR-22, FR-CUS-09, FR-ADM-06 | ADR-014 | 3.3, 3.12 |
| BR-08 | FR-BR-07 | ADR-015 | 3.4 |
| BR-09 | FR-CUS-11, FR-BR-09 | ADR-008, ADR-012 | 3.5, 3.10 |
| BR-10 | FR-CUS-17, FR-BR-35 | ADR-012 | 3.10, 3.11 |
| BR-11 | FR-BR-37 | ADR-015 | 3.14 |
| BR-12 | FR-BR-09 (and every stock-changing FR) | ADR-008 | 3.4–3.14 |
| BR-13 | FR-BR-09 | ADR-008 | 3.5 |
| BR-14 | FR-ADM-13, FR-BR-38 | ADR-009 | 3.1, 3.2 |
| BR-15 | FR-BR-39, FR-ADM-20 | ADR-015 | Phase 2 (framework), 3.1 (real actor), 3.15 (viewing) |

---

## 7. Related Documents

- [functional-requirements.md](functional-requirements.md)
- [non-functional-requirements.md](non-functional-requirements.md)
- [scope-and-assumptions.md](scope-and-assumptions.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
