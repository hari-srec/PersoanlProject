# BakeFlow – User Roles and Personas

> **Step:** 1.1 · **Status:** Draft for owner review · **Source:** [master-prompt.md](../master-prompt.md) §1, §3.H

## Table of Contents

1. [Overview](#1-overview)
2. [Data Access Scopes](#2-data-access-scopes)
3. [Role Definitions](#3-role-definitions)
4. [Personas](#4-personas)
5. [High-Level Role vs Capability Matrix](#5-high-level-role-vs-capability-matrix)
6. [Roles Considered but Not Added](#6-roles-considered-but-not-added)
7. [Related Documents](#7-related-documents)

---

## 1. Overview

BakeFlow has two kinds of user. They are stored in separate tables ([ADR-009](../02-system-design/design-decisions.md#adr-009--separate-staff-user-and-customer-account-tables)):

| User type | Roles | Logs in to |
|-----------|-------|------------|
| **Staff user** | SUPER_ADMIN, ORG_ADMIN, BRANCH_MANAGER, BRANCH_STAFF | Branch Management and Central Administration |
| **Customer account** | CUSTOMER | Customer Platform |

The MVP uses a **fixed set of five roles** (AS-22). Each role is a bundle of permissions. The detailed permission matrix (permission codes per endpoint) is produced in step 1.7.

---

## 2. Data Access Scopes

A role decides **what** a user can do. The scope decides **which data** they can do it on.

| Scope | Meaning | Example |
|-------|---------|---------|
| **System** | All organizations on the platform | The platform operator lists all bakery organizations. |
| **Organization** | All branches and data of the user's own organization | The bakery owner sees revenue of Anna Nagar, T. Nagar and Velachery. |
| **Branch** | Only the branch(es) the user is assigned to | A T. Nagar manager sees only T. Nagar stock. |
| **Own data** | Only records that belong to the user personally | A customer sees only their own orders and addresses. |

**The scope is always enforced on the server.** The server works out the user's allowed organization and branches from their identity and stored assignments, never from values sent by the browser (BR-14, NFR-15).

---

## 3. Role Definitions

### 3.1 SUPER_ADMIN – Platform operator

| Aspect | Detail |
|--------|--------|
| **Who** | The person or team running the BakeFlow platform (in this project, the developer). |
| **Scope** | System |
| **Responsibilities** | Create and deactivate organizations (FR-ADM-01). Create the first ORG_ADMIN of each organization. Manage platform-level settings (FR-ADM-16). Support and troubleshooting, view the audit log (FR-ADM-20). |
| **Does not** | Bill customers or change branch stock in day-to-day operation. Every action they take inside an organization is audited. |
| **MVP note** | With one organization (AS-02), this role is used mainly for the initial setup and support. |

### 3.2 ORG_ADMIN – Central administrator / bakery owner

| Aspect | Detail |
|--------|--------|
| **Who** | The bakery owner or head-office manager. |
| **Scope** | Organization (all branches of own organization) |
| **Responsibilities** | Manage branches (FR-ADM-02). Assign branch managers and staff (FR-ADM-03). Manage the global catalog, categories, base prices, tax data and standard cost (FR-ADM-04 to 08). Configure discount rules, wastage reasons and organization settings (FR-ADM-14 to 16). View consolidated reports (FR-ADM-09 to 12, 17, 18). Can act inside any branch of the organization when needed (for example approve an adjustment). |
| **Does not** | See other organizations' data. |

### 3.3 BRANCH_MANAGER – Branch owner / manager

| Aspect | Detail |
|--------|--------|
| **Who** | The person responsible for one branch's results, stock and staff. |
| **Scope** | Branch (assigned branch or branches; usually one) |
| **Responsibilities** | Branch product availability and low-stock thresholds (FR-BR-02). Approve adjustments and discounts (FR-BR-11, 28). Refunds (FR-BR-23). Daily closing (FR-BR-25). Block batches (FR-BR-34). Transfers (FR-BR-37). Manage branch staff (FR-BR-38). Branch reports and audit trail (FR-BR-31, 32, 39). Plus everything BRANCH_STAFF can do. |
| **Does not** | Change base prices, catalog or organization rules. Approve their own adjustments. Access unassigned branches. |

### 3.4 BRANCH_STAFF – Counter staff and bakers

| Aspect | Detail |
|--------|--------|
| **Who** | Cashiers, counter staff, bakers and packers. |
| **Scope** | Branch (assigned branch only) |
| **Responsibilities** | Create POS bills and record payments (FR-BR-20, 21). Apply discounts within the staff limit (FR-BR-22). Record production (FR-BR-15) and stock receipts (FR-BR-05). Record wastage (FR-BR-29, 30). Request adjustments (FR-BR-11). Fulfil online orders (FR-BR-35). View batches, stock history and alerts (FR-BR-06, 07, 12, 14). |
| **Does not** | Refund, approve, change prices, manage staff or see revenue reports. |

### 3.5 CUSTOMER – Online shopper

| Aspect | Detail |
|--------|--------|
| **Who** | A member of the public with a customer account. |
| **Scope** | Own data |
| **Responsibilities** | Browse, cart, checkout, pay, track and cancel own orders (FR-CUS-04 to 17). Manage own profile and addresses (FR-CUS-20, 21). Reviews and loyalty (future). |
| **Does not** | See any other customer's data, any staff screen, or any branch's stock quantities beyond "available / not available / only N left". |

> **Walk-in counter customers** are not users. They do not log in, and POS bills do not require a customer account (AS-16).

---

## 4. Personas

Personas are fictional but realistic people. They help us design screens around real needs.

### Persona 1 – Lakshmi, the bakery owner (ORG_ADMIN)

- **Background:** 46, owns *Chennai Delights Bakery* with three branches. Started with Anna Nagar 15 years ago.
- **Day:** Checks yesterday's revenue on the phone at 7 AM, visits one branch a day, and meets the head baker weekly to set product prices.
- **Pain points:** Gets sales figures from branches on WhatsApp in the evening. Suspects T. Nagar throws away a lot of stock but has no numbers. Cannot tell how much discounts cost.
- **Needs from BakeFlow:** Branch comparison dashboard, wastage in rupees, control over discount rules, and confidence that expired food is never sold.

### Persona 2 – Karthik, the T. Nagar branch manager (BRANCH_MANAGER)

- **Background:** 34, manages T. Nagar with 6 staff. Decides each morning how much to bake.
- **Day:** Opens at 6:30 AM, plans the bake, approves stock corrections, handles customer complaints and refunds, closes the cash drawer at 10 PM.
- **Pain points:** Bakes "the usual 100" because there is no data. Discovers expired trays at closing. Spends 30 minutes on daily closing with a calculator.
- **Needs from BakeFlow:** A near-expiry list at 4 PM with suggested discounts, a production recommendation for tomorrow, a one-click daily closing report, and the option to send spare stock to Velachery.

### Persona 3 – Priya, counter staff at Anna Nagar (BRANCH_STAFF)

- **Background:** 22, works the evening shift at the counter. Comfortable with smartphones, not with complex software.
- **Day:** Bills 150+ customers per shift, with long queues between 5 and 8 PM. Packs online pickup orders between customers.
- **Pain points:** Slow billing screens. Not knowing which tray to sell first. Online orders that are hard to locate in the shop.
- **Needs from BakeFlow:** Keyboard and scanner billing that takes seconds, automatic FEFO (no thinking about batches), and a clear list of online orders to pack, with the batch to pick from.

### Persona 4 – Arun, the baker at Velachery (BRANCH_STAFF)

- **Background:** 40, head baker. Starts at 4 AM.
- **Pain points:** Writes production in a notebook. Expiry dates are guessed.
- **Needs from BakeFlow:** A simple "record what I baked" screen where expiry is filled in automatically from product shelf life, and that works on a tablet with floury hands (large buttons).

### Persona 5 – Meera, the customer (CUSTOMER)

- **Background:** 29, software engineer living in Velachery. Orders cakes and snacks for the office and the weekend.
- **Day:** Browses on a phone during the commute, wants to pick up on the way home.
- **Pain points:** Arrives at the shop to find the wanted item has sold out. Unsure whether "fresh" items are really fresh.
- **Needs from BakeFlow:** Real-time availability at the chosen branch, safe online payment, a clear "ready for pickup" status, visible near-expiry offers (a discount is welcome if the item is still good), and easy cancellation if plans change.

### Persona 6 – Vikram, the platform operator (SUPER_ADMIN)

- **Background:** The developer of BakeFlow.
- **Needs from BakeFlow:** Set up a new organization in minutes, view audit logs to investigate support issues, and change system-level settings safely.

---

## 5. High-Level Role vs Capability Matrix

Legend: ✅ full · 🔹 limited (see note) · 👁 view only · — no access

| Capability | SUPER_ADMIN | ORG_ADMIN | BRANCH_MANAGER | BRANCH_STAFF | CUSTOMER |
|------------|:-----------:|:---------:|:--------------:|:------------:|:--------:|
| Manage organizations | ✅ | 👁 own | — | — | — |
| Manage branches | ✅ | ✅ | 👁 own | — | — |
| Manage staff users & roles | ✅ | ✅ | 🔹 own branch staff only | — | — |
| Global catalog, categories, base price, tax | 👁 | ✅ | 👁 | 👁 | — |
| Branch product availability & thresholds | — | ✅ | ✅ | 👁 | — |
| Branch price override (future) | — | ✅ | 🔹 within band | — | — |
| Record production / stock receipt | — | 🔹 | ✅ | ✅ | — |
| View batches & stock ledger | 👁 | 👁 all branches | 👁 own branch | 👁 own branch | — |
| Stock adjustment | — | ✅ approve | ✅ approve (not own) | 🔹 request | — |
| Block / unblock batch | — | ✅ | ✅ | — | — |
| POS billing & payments | — | 🔹 | ✅ | ✅ | — |
| Manual discount | — | ✅ | ✅ up to manager limit | 🔹 up to staff limit | — |
| Configure discount rules | — | ✅ | 🔹 within org limits (future) | — | — |
| Approve near-expiry discount | — | ✅ | ✅ | — | — |
| Refunds / credit notes | — | ✅ | ✅ | — | — |
| Daily closing | — | 👁 | ✅ | — | — |
| Record wastage | — | 🔹 | ✅ | ✅ | — |
| Inter-branch transfer (future) | — | ✅ | ✅ own branch side | — | — |
| Fulfil online orders | — | 🔹 | ✅ | ✅ | — |
| Branch reports | 👁 | 👁 all branches | 👁 own branch | — | — |
| Consolidated reports & comparison | 👁 | ✅ | — | — | — |
| Audit log | 👁 all | 👁 own org | 👁 own branch | — | — |
| System / organization settings | ✅ system | ✅ org | — | — | — |
| Browse, cart, order, pay, track | — | — | — | — | ✅ own |
| Profile, addresses, reviews | — | — | — | — | ✅ own |

🔹 for ORG_ADMIN on branch operations means they can act in any branch of their organization when needed. Every such action is audited.

---

## 6. Roles Considered but Not Added

| Candidate role | Reason not added in MVP | Revisit |
|----------------|-------------------------|---------|
| CASHIER vs BAKER (split of BRANCH_STAFF) | One staff role is simpler. Small branches have staff doing both jobs. The permission model allows splitting later. | Post-MVP, if owners ask for it (see OQ-21) |
| DELIVERY_AGENT | Delivery is not in the MVP (OQ-05). When it comes, deliveries are done by branch staff (AS-12). | When delivery is enabled |
| ACCOUNTANT / AUDITOR (read-only finance) | Not in the master prompt. ORG_ADMIN covers it. | Suggestion SG-07 |
| Custom roles | Adds complexity (role editor, permission UI). Fixed roles meet MVP needs (AS-22). | Post-MVP |

---

## 7. Related Documents

- [functional-requirements.md](functional-requirements.md)
- [business-requirements.md](business-requirements.md) – BR-14 branch isolation
- [design-decisions.md](../02-system-design/design-decisions.md) – ADR-009
- Detailed permission matrix: *Planned – created in step 1.7*
