# BakeFlow – Architecture Decision Records (ADR Log)

> **Step:** 1.1 · **Status:** All decisions below are **Accepted** · **Source:** Phase 1.1 prompt, [master-prompt.md](../master-prompt.md) Appendix C

## Table of Contents

1. [What Is an ADR?](#what-is-an-adr)
2. [Decision Index](#decision-index)
3. [ADR-001 – Modular monolith architecture](#adr-001--modular-monolith-architecture)
4. [ADR-002 – Batch-level inventory with FEFO allocation](#adr-002--batch-level-inventory-with-fefo-allocation)
5. [ADR-003 – Flyway for all schema changes; Hibernate ddl-auto=validate](#adr-003--flyway-for-all-schema-changes-hibernate-ddl-autovalidate)
6. [ADR-004 – DECIMAL(12,2) / BigDecimal for all money values](#adr-004--decimal122--bigdecimal-for-all-money-values)
7. [ADR-005 – Store timestamps in UTC; display in Asia/Kolkata](#adr-005--store-timestamps-in-utc-display-in-asiakolkata)
8. [ADR-006 – GST support on products and invoices](#adr-006--gst-support-on-products-and-invoices)
9. [ADR-007 – Razorpay as payment gateway](#adr-007--razorpay-as-payment-gateway)
10. [ADR-008 – Pessimistic locking for stock allocation plus optimistic locking on batches](#adr-008--pessimistic-locking-for-stock-allocation-plus-optimistic-locking-on-batches)
11. [ADR-009 – Separate staff-user and customer account tables](#adr-009--separate-staff-user-and-customer-account-tables)
12. [ADR-010 – Gap-free invoice numbering per branch per financial year](#adr-010--gap-free-invoice-numbering-per-branch-per-financial-year)
13. [ADR-011 – Idempotency-Key header for billing and payment APIs](#adr-011--idempotency-key-header-for-billing-and-payment-apis)
14. [ADR-012 – Cart is not a reservation; time-limited reservation at checkout](#adr-012--cart-is-not-a-reservation-time-limited-reservation-at-checkout)
15. [ADR-013 – Expiry validated at allocation time, not only by scheduled jobs](#adr-013--expiry-validated-at-allocation-time-not-only-by-scheduled-jobs)
16. [ADR-014 – Discounts stored separately; original price never modified](#adr-014--discounts-stored-separately-original-price-never-modified)
17. [ADR-015 – Append-only stock movement ledger for every quantity change](#adr-015--append-only-stock-movement-ledger-for-every-quantity-change)
18. [How to Add a New ADR](#how-to-add-a-new-adr)

---

## What Is an ADR?

An **Architecture Decision Record** captures one important technical decision: the problem (context), what was decided, the alternatives considered and the consequences, good and bad. ADRs are never deleted. If a decision changes, a new ADR **supersedes** the old one, so the history of reasoning is kept.

**Status values:** `Proposed` (under discussion) · `Accepted` (in force) · `Superseded by ADR-xxx` · `Deprecated`.

Where the architect has a concern about an accepted decision, it is written under **Consequences → Concerns** and raised as an open question in [scope-and-assumptions.md](../00-project-overview/scope-and-assumptions.md#5-open-questions-oq-xx).

---

## Decision Index

| ID | Title | Status | Related requirements | Concern raised |
|----|-------|--------|----------------------|----------------|
| ADR-001 | Modular monolith architecture | Accepted | NFR-42, NFR-35 | – |
| ADR-002 | Batch-level inventory with FEFO allocation | Accepted | BR-03, BR-04 | OQ-15, OQ-17 |
| ADR-003 | Flyway for all schema changes; ddl-auto=validate | Accepted | NFR-25 | – |
| ADR-004 | DECIMAL(12,2) / BigDecimal for money | Accepted | NFR-23 | – |
| ADR-005 | UTC storage, Asia/Kolkata display | Accepted | NFR-48 | – |
| ADR-006 | GST support (HSN, rate, CGST/SGST/IGST) | Accepted | FR-BR-24, FR-ADM-06 | OQ-16 |
| ADR-007 | Razorpay payment gateway (test mode in dev) | Accepted | FR-CUS-14, NFR-18 | – |
| ADR-008 | Pessimistic locking + @Version on batches | Accepted | BR-09, BR-12, BR-13, NFR-26–28 | OQ-22 |
| ADR-009 | Separate staff-user and customer tables | Accepted | BR-14 | – |
| ADR-010 | Gap-free invoice numbering per branch per FY | Accepted | NFR-24 | **OQ-02** |
| ADR-011 | Idempotency-Key for billing and payment APIs | Accepted | NFR-11 | – |
| ADR-012 | Cart ≠ reservation; time-limited reservation at checkout | Accepted | BR-09, BR-10 | – |
| ADR-013 | Expiry validated at allocation time | Accepted | BR-05 | – |
| ADR-014 | Discounts stored separately | Accepted | BR-07 | – |
| ADR-015 | Append-only stock movement ledger | Accepted | BR-08, NFR-22 | – |

---

## ADR-001 – Modular monolith architecture

- **Status:** Accepted
- **Context:** BakeFlow has about 18 business modules (inventory, billing, orders and so on) and is built by a **single developer**. Many operations cross modules in one transaction: a POS sale allocates stock (inventory), creates an invoice (billing) and records discounts (discount). Master-prompt Appendix C recommends starting with a modular monolith.
- **Decision:** Build **one deployable Spring Boot application** organized into clearly separated modules (for example `catalog`, `inventory`, `billing`, `order`). Each module owns its tables and exposes a public service interface. Other modules may not reach into its internal classes or repositories. One MySQL database is shared, with table ownership per module.
- **Alternatives considered:**
  - *Microservices:* independent scaling and deployment, but distributed transactions (sagas), network failures, service discovery and much more DevOps. Too costly for one developer and current volumes.
  - *Traditional layered monolith (no module boundaries):* simplest to start, but tends to become a "big ball of mud" where everything depends on everything.
- **Consequences:**
  - ✅ Simple ACID transactions across inventory, billing and orders.
  - ✅ One build, one deployment, easy local development.
  - ✅ Modules can be extracted into services later if needed.
  - ⚠️ Boundaries must be enforced by discipline and automated architecture tests (NFR-42), otherwise they erode.
  - ⚠️ The whole application scales as one unit; acceptable for NFR-32 volumes.

## ADR-002 – Batch-level inventory with FEFO allocation

- **Status:** Accepted
- **Context:** Bakery items expire within hours or days. Two batches of the same product can have different expiry times (Appendix A: A001 expires 25-Nov, A002 expires 26-Nov). Product-level stock cannot say which units to sell first, discount or discard.
- **Decision:** Stock is held **per batch** (branch + product/variant + batch number + manufacturing time + expiry time + quantities). Allocation for any sale or reservation selects sellable batches **ordered by earliest expiry time** (FEFO), with batch ID as a tie-breaker, splitting quantity across batches when needed. Product-level totals are calculated from batches.
- **Alternatives considered:**
  - *Product-level stock:* simple, but fails BR-03 and BR-04.
  - *FIFO (first in, first out):* uses manufacturing or receipt time. It is wrong when a later batch has a shorter shelf life (for example bought-in items).
  - *Manual batch selection by staff:* slow at the counter and error-prone.
- **Consequences:**
  - ✅ Exact knowledge of what expires when; enables discounts, alerts and wastage per batch.
  - ⚠️ One sale line may map to several batches, so sale and order lines need a line-to-batch allocation detail.
  - ⚠️ **Concern:** when one line draws from a discounted and a full-price batch, pricing must be split. Recommendation: split invoice lines per batch and price (**OQ-15**).
  - ⚠️ **Concern:** the system's FEFO choice may differ from the item physically handed over (**L-02, OQ-17**).

## ADR-003 – Flyway for all schema changes; Hibernate ddl-auto=validate

- **Status:** Accepted
- **Context:** Letting Hibernate create or alter tables automatically (`ddl-auto=update`) can silently create wrong columns, never drops anything and is not repeatable across environments.
- **Decision:** Every schema change is a **versioned Flyway SQL migration** (`V1__init.sql`, `V2__…`) committed with the code. Hibernate runs with `spring.jpa.hibernate.ddl-auto=validate`, so startup **fails** if entities and schema disagree.
- **Alternatives considered:**
  - *Hibernate `update`/`create`:* fast for prototypes, unsafe for real data.
  - *Liquibase:* equally capable (XML/YAML/SQL changelogs, rollback support). Flyway's plain SQL is simpler to learn and review.
- **Consequences:**
  - ✅ Repeatable schema in every environment, reviewed like code, testable with Testcontainers.
  - ⚠️ Migrations already applied must never be edited; fixes go in a new migration.
  - ⚠️ Changes must be backward-compatible during deployment (for example add a column, deploy, then remove the old one later).

## ADR-004 – DECIMAL(12,2) / BigDecimal for all money values

- **Status:** Accepted
- **Context:** Floating-point numbers (`float`, `double`) cannot represent values like 0.10 exactly, which causes rounding errors in bills and tax.
- **Decision:** All money columns are `DECIMAL(12,2)` (up to ₹9,999,999,999.99), held in Java as `BigDecimal`. Rounding is explicit: `RoundingMode.HALF_UP` to 2 decimals, applied at **line level** for tax and discount, then summed. Percentages (GST rate, discount %) use `DECIMAL(5,2)`.
- **Alternatives considered:**
  - *`double`:* rejected because of rounding errors.
  - *Integer paise (`BIGINT`):* exact and fast, but every display and calculation must convert, and it is easy to get wrong. Razorpay uses paise, so conversion happens only at that boundary.
- **Consequences:**
  - ✅ Exact, auditable amounts that match printed invoices.
  - ⚠️ The rounding rules (line-level vs invoice-level) must be documented once (step 1.3 / 1.5) and used everywhere.
  - ⚠️ Conversion to and from paise is needed at the Razorpay boundary.

## ADR-005 – Store timestamps in UTC; display in Asia/Kolkata

- **Status:** Accepted
- **Context:** Expiry times, reservations and reports depend on correct time. Mixing server-local and user-local times causes off-by-5:30 bugs.
- **Decision:** All timestamps are **stored in UTC** and handled as `Instant` in Java. The JVM and JDBC connection are set to UTC. Times are **converted to Asia/Kolkata only for display** and for interpreting user input. "Business day", daily reports and financial year are computed on the **IST calendar** (AS-21).
- **Alternatives considered:**
  - *Store in IST:* simple while all users are in India, but breaks with servers in other regions, cloud defaults and any future expansion.
  - *Store with offset (`TIMESTAMP WITH TIME ZONE`):* MySQL has no such type.
- **Consequences:**
  - ✅ Unambiguous times, safe server placement.
  - ⚠️ Daily boundaries must be converted: the IST day 25-Nov runs from 24-Nov 18:30 UTC to 25-Nov 18:30 UTC. Reports and FY logic must use a shared helper.
  - ⚠️ MySQL `DATETIME` has no time zone, so the UTC convention must be enforced in configuration and tested. Column type (`DATETIME(3)` vs `TIMESTAMP`) is decided in step 1.3.

## ADR-006 – GST support on products and invoices

- **Status:** Accepted
- **Context:** Indian retail sales need GST invoices. Bakery items have different HSN codes and GST rates.
- **Decision:** Each product (variant) stores an **HSN code** and **GST rate**. Each invoice line stores a **snapshot** of HSN, rate, taxable value and **CGST, SGST and IGST amounts**, so later rate changes never alter old invoices. The invoice stores totals per tax component and the organization's GSTIN.
- **Alternatives considered:**
  - *Tax only at invoice level:* cannot handle mixed-rate bills.
  - *Look up current rates at reporting time:* wrong when rates change.
  - *No GST in the MVP:* invoices would not be legally usable.
- **Consequences:**
  - ✅ Legally structured invoices; correct tax reports per rate.
  - ⚠️ **Concern:** all branches are expected to be in Tamil Nadu (AS-03), so every sale is intra-state (CGST + SGST). IGST columns exist but stay zero in the MVP. Supporting IGST properly would need place-of-supply logic.
  - ⚠️ **Concern:** the design assumes a regular GST registration. A composition-scheme business issues a "bill of supply" without GST (**OQ-16**).
  - ⚠️ BakeFlow stores rates configured by the admin; it does not decide tax law (AS-05).

## ADR-007 – Razorpay as payment gateway

- **Status:** Accepted
- **Context:** Online orders need Indian payment methods (UPI, cards, net banking) with good developer documentation and a free test mode.
- **Decision:** Use **Razorpay**, in **test mode** during development (AS-19). Flow: the server creates a Razorpay order; the customer pays in Razorpay Checkout; the server **verifies the payment signature**; a **webhook** reconciles payments where the browser never returned. Refunds use the Razorpay Refund API. All Razorpay calls sit behind a `PaymentGateway` interface.
- **Alternatives considered:** PayU, Cashfree, Stripe India, PhonePe PG. All are viable. Razorpay was chosen for wide adoption, clear documentation and easy test mode.
- **Consequences:**
  - ✅ Card and UPI data never touch BakeFlow servers (NFR-18).
  - ⚠️ The webhook endpoint must be public, signature-verified and idempotent.
  - ⚠️ Vendor dependency is limited by the `PaymentGateway` abstraction.
  - ⚠️ Razorpay calls are made **outside** stock-lock transactions (NFR-28).

## ADR-008 – Pessimistic locking for stock allocation plus optimistic locking on batches

- **Status:** Accepted
- **Context:** Two cashiers, or a cashier and an online checkout, may try to take the last unit of a batch at the same moment (Appendix C). Without control, both succeed and stock goes negative (BR-13).
- **Decision:**
  1. **Allocation** (POS, checkout reservation, transfer dispatch, wastage, adjustment) reads candidate batches with **`SELECT … FOR UPDATE`** inside a transaction. This locks those rows until commit, so a competing transaction waits and then sees the updated quantity.
  2. Batch entities also carry a **`@Version`** column (optimistic locking). Any update made without a row lock (for example blocking a batch or editing metadata) fails cleanly if someone else changed the row first.
  3. Rows are locked in a **consistent order** (by batch ID) to avoid deadlocks. Transactions are short. Deadlock or lock-timeout errors are retried up to 3 times (NFR-27, 28).
  4. Database **CHECK constraints** (quantities ≥ 0) are the final safety net.
- **Alternatives considered:**
  - *Optimistic locking only:* fine with low contention, but at busy counters the same popular batch is hit often, causing many retries.
  - *Atomic conditional update* (`UPDATE … SET qty = qty - n WHERE qty >= n`): very efficient for one row, but FEFO across several batches needs to read and decide first.
  - *Application-level or distributed locks (Redis):* extra infrastructure, and the database is already the source of truth.
- **Consequences:**
  - ✅ No overselling, proven by a concurrency test (NFR-26).
  - ⚠️ Lock waits under high contention. Mitigated by short transactions and no external calls while locked.
  - ⚠️ **Concern (minor):** two locking mechanisms on the same row can confuse developers. Rule of thumb: pessimistic for quantity changes, `@Version` for everything else. A pessimistic update also increments the version (**OQ-22**).

## ADR-009 – Separate staff-user and customer account tables

- **Status:** Accepted
- **Context:** Staff and customers differ greatly. Staff have roles, branch assignments and an organization. Customers have addresses, carts, orders and loyalty. Their login rules, numbers and privacy needs also differ.
- **Decision:** Two tables, `staff_user` and `customer` (names finalised in 1.3), with separate authentication endpoints. The JWT carries a **principal type** (`STAFF` or `CUSTOMER`). Staff endpoints reject customer tokens and the other way round.
- **Alternatives considered:**
  - *One `user` table with a type column:* less duplication, but many nullable columns and a higher risk of a customer getting staff permissions through a bug.
  - *External identity provider (Keycloak, Auth0):* powerful but heavy for this project's learning goals and scale.
- **Consequences:**
  - ✅ Strong separation; a mistake in customer code cannot grant staff access.
  - ⚠️ A person who is both an employee and a customer has two accounts (acceptable).
  - ⚠️ Some authentication code (password hashing, token issuing) is shared through common components.

## ADR-010 – Gap-free invoice numbering per branch per financial year

- **Status:** Accepted
- **Context:** GST rules require invoice numbers that are consecutive and unique per financial year (series allowed). Auto-increment IDs leave gaps after a rollback and are shared across branches.
- **Decision:** Keep a **sequence row per (branch, document type, financial year)**. When an invoice is issued, the row is locked (`SELECT … FOR UPDATE`), incremented and used **in the same transaction** as the invoice insert. A rollback therefore rolls back the number too, leaving no gap. Invoices are never deleted; corrections are made with credit notes, which have their own series (AS-37). Example format: `ANN/2026-27/000123`.
- **Alternatives considered:**
  - *Database auto-increment:* gaps on rollback, global not per branch.
  - *Generate numbers in memory:* breaks with several instances or restarts.
  - *Assign numbers in a nightly batch:* customers need a numbered invoice immediately.
- **Consequences:**
  - ✅ Legally compliant continuous series per branch.
  - ⚠️ The sequence row is a contention point per branch. Billing at one branch is serialised briefly; fine at about 2,000 bills/day.
  - ⚠️ **Concern:** GST rules limit invoice numbers to **16 characters**. The example `ANN/2026-27/000123` is **18 characters**. Recommended compliant format: `ANN/2627/000123` (15 characters) (**OQ-02**).

## ADR-011 – Idempotency-Key header for billing and payment APIs

- **Status:** Accepted
- **Context:** Networks fail. A cashier might click "Pay" twice, or the browser might retry after a timeout. Without protection this creates duplicate invoices, double stock deduction or double refunds.
- **Decision:** Endpoints that create invoices, payments, refunds, checkout reservations and order placements require an **`Idempotency-Key`** header (a client-generated UUID). The server stores key + user + endpoint + request hash + response. A repeat with the same key returns the **stored response** without running the operation again. The same key with a different request body returns `422`. Keys expire after **24 hours**.
- **Alternatives considered:**
  - *Rely on UI button disabling:* does not cover network retries.
  - *Natural unique constraints only:* not every operation has a natural key.
- **Consequences:**
  - ✅ Safe retries (NFR-11).
  - ⚠️ Needs a key store table (and possibly Redis later) plus a clean-up job.
  - ⚠️ Concurrent requests with the same key must be handled (the second one waits or gets `409`).

## ADR-012 – Cart is not a reservation; time-limited reservation at checkout

- **Status:** Accepted
- **Context:** If adding to cart reserved stock, abandoned carts would lock stock away from counter customers. If nothing is ever reserved, a customer could pay for stock that was just sold at the counter (Appendix C).
- **Decision:** Adding to cart **does not touch inventory**. Availability is re-checked when the cart is viewed. **At checkout**, the system reserves stock by FEFO (ADR-002, ADR-008) for a limited time (default **15 minutes**, AS-13). On successful payment, the reservation is **extended until the pickup deadline** (AS-38) and finally converted to a sale at handover (AS-14). On payment failure, timeout, cancellation or no-show, it is **released** (BR-10). A scheduled job releases expired reservations within 1 minute (NFR-07).
- **Alternatives considered:**
  - *Reserve on add-to-cart:* stock hoarding.
  - *No reservation, check at payment:* customers pay for unavailable items, causing refunds and complaints.
- **Consequences:**
  - ✅ Fair to counter customers; safe for online customers.
  - ⚠️ A customer may see "available" in the cart and find it gone at checkout. The UI must explain this clearly.
  - ⚠️ A payment that succeeds just after the reservation expired must be handled: re-reserve if possible, otherwise refund automatically.

## ADR-013 – Expiry validated at allocation time, not only by scheduled jobs

- **Status:** Accepted
- **Context:** A scheduled job that marks batches expired runs periodically. Between runs, a batch can be past its expiry time but still have status `ACTIVE`. Appendix C explicitly warns against relying only on jobs.
- **Decision:** Every allocation query includes **`expires_at > now()`** (and status not blocked), regardless of stored status. Online availability uses `expires_at > now() + online cut-off` (AS-27). The scheduled job (every 5 minutes, AS-28) **also** updates status, removes items from online listings and raises alerts, but it is not the safety mechanism.
- **Alternatives considered:**
  - *Job only:* leaves a window of up to one job interval in which expired food can be sold.
  - *Database event scheduler:* same window problem, and logic hidden in the database.
- **Consequences:**
  - ✅ BR-05 is guaranteed at the moment of sale (AS-42).
  - ⚠️ Status and reality may briefly disagree; screens should derive "expired" from `expires_at`, not only from status.
  - ⚠️ Index design must support `(branch, product, expires_at)` lookups (step 1.3).

## ADR-014 – Discounts stored separately; original price never modified

- **Status:** Accepted
- **Context:** If a near-expiry discount overwrote the product price, the original price would be lost, reports would show wrong revenue and the discount might stay after the batch is gone (BR-07).
- **Decision:** Prices live in a **price table with effective dates** (history kept). Discounts are **separate records**: rules, batch-level applied discounts with approver and validity, and manual line discounts. Invoice and order lines store **original unit price, discount amount, discount reference and final price** separately. Every discount change is audited (Appendix C).
- **Alternatives considered:**
  - *Overwrite the price:* loses information (rejected by BR-07).
  - *Store only the final price on lines:* cannot report discount cost.
- **Consequences:**
  - ✅ Accurate discount-cost reporting and full history.
  - ⚠️ More tables and joins; price calculation needs one shared "pricing service".

## ADR-015 – Append-only stock movement ledger for every quantity change

- **Status:** Accepted
- **Context:** BR-08 requires every inventory change to be recorded. Stock differences must be explainable ("why does B001 show 18 when we counted 17?").
- **Decision:** Every quantity change writes a row to a **stock movement ledger**: type (for example `PRODUCTION_IN`, `RECEIPT_IN`, `RESERVE`, `RESERVATION_RELEASE`, `SALE_OUT`, `TRANSFER_OUT`, `TRANSFER_IN`, `ADJUSTMENT_IN/OUT`, `WASTAGE_OUT`), batch, quantity, the resulting quantities, user, time (UTC) and reference document (invoice, order, transfer, adjustment, wastage). Rows are **never updated or deleted**. Batch quantity columns are kept as a fast **current-state cache**, updated in the **same transaction** as the ledger insert.
- **Alternatives considered:**
  - *Quantities only, no history:* fails BR-08.
  - *Full event sourcing (state rebuilt from events only):* powerful but complex for a first project; reads become harder.
  - *Database triggers writing history:* hides business context (who, why, reference).
- **Consequences:**
  - ✅ Complete, explainable stock history; supports audits and reconciliation.
  - ⚠️ The ledger grows fast (about 10,000+ rows/day at MVP volumes). It needs good indexes and, later, partitioning or archiving.
  - ⚠️ Batch cache and ledger could drift through a bug, so a reconciliation check is needed (NFR-22, SG-12).
  - ⚠️ Production database permissions should prevent UPDATE and DELETE on the ledger (NFR-30).

---

## How to Add a New ADR

1. Copy the structure of an existing ADR and use the next number (ADR-016 …).
2. Set **Status: Proposed** and discuss it.
3. When agreed, change it to **Accepted** and add it to the index.
4. To change an earlier decision, write a new ADR and mark the old one `Superseded by ADR-xxx`. Never delete ADRs.

**Expected upcoming ADRs (step 1.2 onward):** frontend structure (one React app vs separate apps), JWT storage strategy in the browser, image storage, module boundary testing tool, caching strategy (Phase 4), async event mechanism (Phase 4).
