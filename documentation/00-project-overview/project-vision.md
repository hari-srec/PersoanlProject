# BakeFlow – Project Vision

> **Step:** 1.1 (owner-reviewed) · **Source:** [master-prompt.md](../master-prompt.md) §1, Appendix A, Appendix B

## Table of Contents

1. [Vision Statement](#1-vision-statement)
2. [Problem Statement](#2-problem-statement)
3. [The Chennai Example](#3-the-chennai-example)
4. [Solution Overview](#4-solution-overview)
5. [Target Users](#5-target-users)
6. [Business Value](#6-business-value)
7. [Success Measures](#7-success-measures)
8. [Guiding Principles](#8-guiding-principles)
9. [Related Documents](#9-related-documents)

---

## 1. Vision Statement

**BakeFlow helps a multi-branch bakery sell what it bakes, before it expires.**

It is one platform where:

- **customers** can see what is really available at their nearest branch and order it online,
- **branch teams** know exactly which batch of each product is on the shelf, when it expires and what to bake tomorrow, and
- **central administrators** can see the stock, sales and wastage of every branch on one screen.

---

## 2. Problem Statement

Bakery products have a short shelf life, often measured in hours rather than months. Today, a typical multi-branch bakery has four problems:

| # | Problem | What it looks like day to day | Cost to the business |
|---|---------|-------------------------------|----------------------|
| P1 | **Expired products** | Pastries baked yesterday are found unsold at closing time. Staff are not sure which tray is older. | Direct loss of goods, plus the risk of selling expired food. |
| P2 | **Overproduction** | Each branch bakes the same quantity every day, whatever it actually sells. | Ingredients, labour and energy are spent on products that end up in the bin. |
| P3 | **Inaccurate branch inventory** | Stock is counted per product ("40 pastries"), not per batch. Nobody knows that 30 of them expire tonight. | Wrong decisions on discounts, transfers and production. |
| P4 | **Poor visibility of sales** | The owner gets sales figures late, on paper or in spreadsheets, and cannot compare branches. | Problems such as one branch overproducing go unnoticed for weeks. |

Existing POS tools usually solve billing only. They track stock at **product** level, which cannot tell two batches of the same pastry apart.

---

## 3. The Chennai Example

A bakery has three branches in Chennai: **Anna Nagar**, **T. Nagar** and **Velachery**. Each bakes 100 chocolate pastries a day:

| Branch | Produced | Sold | Remaining | Remaining % |
|--------|---------:|-----:|----------:|------------:|
| Anna Nagar | 100 | 95 | 5 | 5 % |
| T. Nagar | 100 | 60 | 40 | **40 %** |
| Velachery | 100 | 98 | 2 | 2 % |

**What BakeFlow should do with this:**

1. **Spot the pattern.** Over several days it notices that T. Nagar regularly has about 40 % left over and recommends baking fewer pastries tomorrow (forecasting, module 3.17).
2. **Act before the waste happens.** Late in the day, it flags the T. Nagar batches that are close to expiry and suggests a discount based on the bakery's own discount rules (module 3.12).
3. **Move stock where it sells.** Velachery almost sold out, so the T. Nagar manager could send spare stock there instead of Velachery baking more (module 3.14).
4. **Count what is lost.** Any pastries that still expire are recorded as wastage, together with their cost (module 3.13).

### Why batches matter

The same product can sit on the shelf in several batches, each with its own dates and quantity:

```
Chocolate Pastry (Product 101)
├── Anna Nagar
│   ├── Batch A001 · Manufactured 24-Nov · Expires 25-Nov · Qty 30
│   └── Batch A002 · Manufactured 24-Nov · Expires 26-Nov · Qty 50
└── T. Nagar
    └── Batch B001 · Manufactured 24-Nov · Expires 25-Nov · Qty 20
```

When a customer buys a pastry at Anna Nagar, BakeFlow sells from **A001 first**, because it expires first. This rule is called **FEFO (First Expired, First Out)**.

---

## 4. Solution Overview

BakeFlow is a single application with three areas:

| Area | Who uses it | Main purpose |
|------|-------------|--------------|
| **Customer Platform** | Customers (mobile-friendly website) | Browse products by branch, see prices and offers, order, pay online, track orders. |
| **Branch Management** | Branch managers and staff | Inventory by batch, production, POS billing, expiry, wastage, transfers. |
| **Central Administration** | Organization admins | Branches, employees, product catalog, pricing, discount rules, consolidated reports. |

Technically, BakeFlow is a **modular monolith**: one deployable application split into clearly separated modules. See [ADR-001](../02-system-design/design-decisions.md#adr-001--modular-monolith-architecture).

---

## 5. Target Users

| User | Description | Main need |
|------|-------------|-----------|
| Bakery owner / central admin | Runs the organization and all its branches | One view of sales, stock and wastage across branches |
| Branch manager | Responsible for one branch's stock, staff and results | Knows what to bake, discount, transfer and write off |
| Branch staff (counter / baker) | Bills customers, records production and wastage | A fast POS and simple stock screens |
| Customer | Buys from a nearby branch | Sees real availability, orders and pays online, collects quickly |
| Platform operator | Runs the BakeFlow platform itself | Creates organizations and manages system-level settings |

Detailed roles and personas: [user-roles-and-personas.md](user-roles-and-personas.md).

---

## 6. Business Value

| Value | How BakeFlow delivers it | Linked problem |
|-------|--------------------------|----------------|
| Less wastage | Expiry alerts, near-expiry discounts, transfers, accurate wastage records | P1, P2 |
| Right production quantity | Production vs sales comparison and next-day recommendation | P2 |
| Trustworthy stock figures | Stock tracked by batch, with every change recorded in an append-only ledger | P3 |
| Food safety and compliance | Expired batches cannot be sold, whether at the POS or online. GST-compliant invoices. | P1 |
| Faster decisions | Daily closing report, branch-wise revenue and wastage dashboards | P4 |
| A new sales channel | Online ordering that shows real branch stock | – |

---

## 7. Success Measures

The original brief sets no numeric targets. The measures below were proposed in step 1.1 and **accepted by the owner as targets to review after 3 months live** (OQ-18). Each one is recorded as an assumption in [scope-and-assumptions.md](scope-and-assumptions.md#36-success-measure-assumptions) (AS-40 to AS-47).

| ID | Measure | Proposed target | How it is measured |
|----|---------|-----------------|--------------------|
| AS-40 | Wastage reduction | Wastage value (₹) down **30 %** within 3 months of go-live, compared with a 4-week baseline | Wastage report (3.13 / 3.16) |
| AS-41 | Stock accuracy | Physical count matches system stock for **≥ 98 %** of batch lines | Stock adjustment records (3.6) |
| AS-42 | Expired products sold | **0** sales lines allocated from an expired batch | Automated tests plus an audit query |
| AS-43 | POS speed | Bill created in **< 500 ms** (server). A typical 3-item bill completed by staff in **< 30 s** | NFR-01, usability observation |
| AS-44 | Overproduction | Unsold-at-close below **10 %** of production per product per branch (T. Nagar today: 40 %) | Production vs sales report (3.16) |
| AS-45 | Near-expiry sell-through | **≥ 50 %** of discounted near-expiry units sold before expiry | Discount and sales reports (3.12 / 3.16) |
| AS-46 | Forecast accuracy | Mean absolute percentage error (MAPE) **≤ 20 %** after 8 weeks of sales history | Forecasting module (3.17) |
| AS-47 | Online order reliability | **≥ 98 %** of paid orders fulfilled without an "out of stock after payment" cancellation | Order reports (3.11) |

---

## 8. Guiding Principles

1. **Batch, not product.** All stock is tracked per batch ([ADR-002](../02-system-design/design-decisions.md#adr-002--batch-level-inventory-with-fefo-allocation)).
2. **Never sell expired food.** Expiry is checked every time stock is allocated, not only by a background job ([ADR-013](../02-system-design/design-decisions.md#adr-013--expiry-validated-at-allocation-time-not-only-by-scheduled-jobs)).
3. **Every quantity change leaves a trace.** An append-only stock ledger records each change ([ADR-015](../02-system-design/design-decisions.md#adr-015--append-only-stock-movement-ledger-for-every-quantity-change)).
4. **Branches are isolated.** A user only sees the branches they are assigned to, and this is enforced on the server ([BR-14](business-requirements.md#br-14--verify-organization-and-branch-access)).
5. **Simple first.** Start as a modular monolith with a realistic MVP, and grow from there.
6. **Learn by documenting.** This project is also a learning vehicle for professional architecture, so every decision is written down with its reasoning.

---

## 9. Related Documents

- [business-requirements.md](business-requirements.md) – business goals and rules BR-01 to BR-15
- [functional-requirements.md](functional-requirements.md) – FR-CUS, FR-BR and FR-ADM requirements
- [non-functional-requirements.md](non-functional-requirements.md) – measurable quality targets
- [user-roles-and-personas.md](user-roles-and-personas.md)
- [feature-roadmap.md](feature-roadmap.md)
- [scope-and-assumptions.md](scope-and-assumptions.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
