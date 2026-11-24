# BakeFlow – Architecture Decision Records (ADR Log)

> **Steps:** 1.1 (ADR-001 – 015, Accepted) · 1.2 (ADR-016 – 034, Proposed until owner review) · **Source:** Phase 1.1 prompt, [master-prompt.md](../master-prompt.md) Appendix C

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
17. [ADR-016 – Spring Boot 4.1.x on Java 21 LTS](#adr-016--spring-boot-41x-on-java-21-lts)
18. [ADR-017 – Dependency version policy](#adr-017--dependency-version-policy)
19. [ADR-018 – Maven with the Maven Wrapper](#adr-018--maven-with-the-maven-wrapper)
20. [ADR-019 – MySQL 8.4 LTS](#adr-019--mysql-84-lts)
21. [ADR-020 – Package-by-module structure enforced by Spring Modulith and ArchUnit](#adr-020--package-by-module-structure-enforced-by-spring-modulith-and-archunit)
22. [ADR-021 – No Lombok; Java records for DTOs](#adr-021--no-lombok-java-records-for-dtos)
23. [ADR-022 – MapStruct for DTO mapping](#adr-022--mapstruct-for-dto-mapping)
24. [ADR-023 – JWT with Spring Security Resource Server (Nimbus), not JJWT](#adr-023--jwt-with-spring-security-resource-server-nimbus-not-jjwt)
25. [ADR-024 – Token storage in the browser and same-origin deployment](#adr-024--token-storage-in-the-browser-and-same-origin-deployment)
26. [ADR-025 – ShedLock for scheduled jobs](#adr-025--shedlock-for-scheduled-jobs)
27. [ADR-026 – Single React SPA built with Vite, not Next.js](#adr-026--single-react-spa-built-with-vite-not-nextjs)
28. [ADR-027 – Frontend data layer: TanStack Query, Axios, React Hook Form, Zod; no Redux](#adr-027--frontend-data-layer-tanstack-query-axios-react-hook-form-zod-no-redux)
29. [ADR-028 – REST API conventions](#adr-028--rest-api-conventions)
30. [ADR-029 – Database naming conventions and Flyway versioning](#adr-029--database-naming-conventions-and-flyway-versioning)
31. [ADR-030 – Redis for caching only, introduced in Phase 4](#adr-030--redis-for-caching-only-introduced-in-phase-4)
32. [ADR-031 – File storage behind an interface; local disk first](#adr-031--file-storage-behind-an-interface-local-disk-first)
33. [ADR-032 – Invoice printing: browser print first, server PDF later](#adr-032--invoice-printing-browser-print-first-server-pdf-later)
34. [ADR-033 – Email via Spring Mail; Mailpit in development](#adr-033--email-via-spring-mail-mailpit-in-development)
35. [ADR-034 – Razorpay integration through the official Java SDK behind PaymentGateway](#adr-034--razorpay-integration-through-the-official-java-sdk-behind-paymentgateway)
36. [How to Add a New ADR](#how-to-add-a-new-adr)

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
| ADR-002 | Batch-level inventory with FEFO allocation | Accepted | BR-03, BR-04 | OQ-15, OQ-17 (resolved) |
| ADR-003 | Flyway for all schema changes; ddl-auto=validate | Accepted | NFR-25 | – |
| ADR-004 | DECIMAL(12,2) / BigDecimal for money | Accepted | NFR-23 | – |
| ADR-005 | UTC storage, Asia/Kolkata display | Accepted | NFR-48 | – |
| ADR-006 | GST support (HSN, effective-dated rates, CGST/SGST/IGST) | Accepted (amended 1.1 review) | FR-BR-24, FR-ADM-06 | OQ-01, OQ-16 (resolved) |
| ADR-007 | Razorpay payment gateway (test mode in dev) | Accepted | FR-CUS-14, NFR-18 | – |
| ADR-008 | Pessimistic locking + @Version on batches | Accepted | BR-09, BR-12, BR-13, NFR-26–28 | OQ-22 (resolved: keep both) |
| ADR-009 | Separate staff-user and customer tables | Accepted | BR-14 | – |
| ADR-010 | Gap-free invoice numbering per branch per FY | Accepted (amended 1.1 review) | NFR-24 | OQ-02 (resolved: `ANN/2627/000123`) |
| ADR-011 | Idempotency-Key for billing and payment APIs | Accepted | NFR-11 | – |
| ADR-012 | Cart ≠ reservation; time-limited reservation at checkout | Accepted | BR-09, BR-10 | – |
| ADR-013 | Expiry validated at allocation time | Accepted | BR-05 | – |
| ADR-014 | Discounts stored separately | Accepted | BR-07 | – |
| ADR-015 | Append-only stock movement ledger | Accepted | BR-08, NFR-22 | – |
| ADR-016 | Spring Boot 4.1.x on Java 21 LTS | Proposed | NFR-16, NFR-44 | OQ-24, OQ-25 |
| ADR-017 | Dependency version policy | Proposed | NFR-16, NFR-44 | – |
| ADR-018 | Maven with the Maven Wrapper | Proposed | NFR-44 | – |
| ADR-019 | MySQL 8.4 LTS | Proposed | NFR-09, NFR-21 | – |
| ADR-020 | Package-by-module structure enforced by Spring Modulith and ArchUnit | Proposed | NFR-42 | – |
| ADR-021 | No Lombok; Java records for DTOs | Proposed | NFR-41 | – |
| ADR-022 | MapStruct for DTO mapping | Proposed | NFR-38 | – |
| ADR-023 | JWT with Spring Security Resource Server (Nimbus), not JJWT | Proposed | NFR-14, BR-14 | – |
| ADR-024 | Token storage in the browser and same-origin deployment | Proposed | NFR-14, NFR-17 | – |
| ADR-025 | ShedLock for scheduled jobs | Proposed | NFR-07, NFR-35 | – |
| ADR-026 | Single React SPA built with Vite, not Next.js | Proposed | NFR-03, NFR-37 | – |
| ADR-027 | Frontend data layer: TanStack Query, Axios, React Hook Form, Zod; no Redux | Proposed | NFR-36, NFR-37 | – |
| ADR-028 | REST API conventions | Proposed | NFR-38, NFR-43, NFR-11 | – |
| ADR-029 | Database naming conventions and Flyway versioning | Proposed | NFR-21, NFR-23, NFR-25 | – |
| ADR-030 | Redis for caching only, introduced in Phase 4 | Proposed | NFR-03, NFR-12 | – |
| ADR-031 | File storage behind an interface; local disk first | Proposed | AS-34 | – |
| ADR-032 | Invoice printing: browser print first, server PDF later | Proposed | NFR-40, FR-BR-24 | – |
| ADR-033 | Email via Spring Mail; Mailpit in development | Proposed | AS-58, FR-CUS-03 | – |
| ADR-034 | Razorpay integration through the official Java SDK behind PaymentGateway | Proposed | FR-CUS-14, NFR-18 | – |

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
  - ⚠️ **Concern:** when one line draws from a discounted and a full-price batch, pricing must be split. **Resolved (OQ-15):** invoice lines are split per batch and price (AS-56).
  - ⚠️ **Concern:** the system's FEFO choice may differ from the item physically handed over (**L-02**). **Resolved (OQ-17):** system FEFO plus shelf practice in the MVP (AS-57).

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
  - ⚠️ MySQL `DATETIME` has no time zone, so the UTC convention must be enforced in configuration and tested. **Decided in step 1.2:** columns use `DATETIME(6)` holding UTC values ([coding-conventions.md](../01-technology-stack/coding-conventions.md#43-data-types)).

## ADR-006 – GST support on products and invoices

- **Status:** Accepted
- **Context:** Indian retail sales need GST invoices. Bakery items have different HSN codes and GST rates.
- **Decision:** Each product variant references an **HSN code**. GST rates are stored **per HSN code with effective dates** (`effective_from`, optional `effective_to`), so the rate applied to a sale is the one in force at the time of sale. **Rates are never hard-coded**; when the government changes a rate, the admin adds a new effective-dated rate. Each invoice line stores a **snapshot** of HSN, rate, taxable value and **CGST, SGST and IGST amounts**, so later rate changes never alter old invoices. The invoice stores totals per tax component and the organization's GSTIN. *(Amended in the step 1.1 review, OQ-01.)*
- **Alternatives considered:**
  - *Tax only at invoice level:* cannot handle mixed-rate bills.
  - *Look up current rates at reporting time:* wrong when rates change.
  - *No GST in the MVP:* invoices would not be legally usable.
- **Consequences:**
  - ✅ Legally structured invoices; correct tax reports per rate.
  - ✅ Rate changes need no code change or deployment; history stays correct.
  - ⚠️ All branches are in Tamil Nadu (AS-03, confirmed), so every sale is intra-state (CGST + SGST). IGST columns exist but stay zero in the MVP. Supporting IGST properly would need place-of-supply logic.
  - ⚠️ **Resolved (OQ-16):** regular GST registration, a single GSTIN in Tamil Nadu, an invoice series per branch (AS-04).
  - ⚠️ Rate lookup must use the IST business date of the sale (ADR-005), and overlapping effective periods for the same HSN code must be rejected.
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
  - ⚠️ **Concern (minor):** two locking mechanisms on the same row can confuse developers. Rule of thumb: pessimistic for quantity changes, `@Version` for everything else. A pessimistic update also increments the version. **Resolved (OQ-22):** keep both mechanisms.

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
- **Decision:** Keep a **sequence row per (branch, document type, financial year)**. When an invoice is issued, the row is locked (`SELECT … FOR UPDATE`), incremented and used **in the same transaction** as the invoice insert. A rollback therefore rolls back the number too, leaving no gap. Invoices are never deleted; corrections are made with credit notes, which have their **own separate gap-free series** (AS-37, SG-08 accepted).
- **Format (amended in the step 1.1 review, OQ-02):**

  | Document | Pattern | Example | Length |
  |----------|---------|---------|-------:|
  | Tax invoice | `{BRANCH}/{FY}/{6-digit counter}` | `ANN/2627/000123` | 15 |
  | Credit note | `{BRANCH}C/{FY}/{5-digit counter}` | `ANNC/2627/00012` | 15 |

  `{BRANCH}` is the 3-letter branch code; `{FY}` is the financial year as `2627` for 2026-27. **Reason:** GST rules (Rule 46 of the CGST Rules) limit an invoice serial number to **16 characters** (letters, digits, `-` and `/`). The earlier example `ANN/2026-27/000123` was 18 characters and would not have been compliant. The 6-digit counter allows 999,999 invoices per branch per year (about 2,700 a day), far above expected volume. The counter resets on 1 April (IST).
- **Alternatives considered:**
  - *Database auto-increment:* gaps on rollback, global not per branch.
  - *Generate numbers in memory:* breaks with several instances or restarts.
  - *Assign numbers in a nightly batch:* customers need a numbered invoice immediately.
- **Consequences:**
  - ✅ Legally compliant continuous series per branch.
  - ⚠️ The sequence row is a contention point per branch. Billing at one branch is serialised briefly; fine at about 2,000 bills/day.
  - ✅ **Resolved (OQ-02):** the format stays within the 16-character limit.
  - ⚠️ Branch codes must be exactly 3 letters and unique per organization, so the format never exceeds 16 characters.

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

## ADR-016 – Spring Boot 4.1.x on Java 21 LTS

- **Status:** Proposed (step 1.2)
- **Context:** The brief asks for Java 21 and Spring Boot. On 24-Nov-2026: Spring Boot **3.5.x open-source support ended on 30-Jun-2026** (last release 3.5.16); **4.1.3** is the latest 4.1 patch (20-Nov-2026); **4.2.0** was released on 19-Nov-2026. springdoc-openapi 3.x, Testcontainers 2.x, Flyway (with `spring-boot-starter-flyway` + `flyway-mysql`) and Spring Modulith 2.x all target Spring Boot 4.
- **Decision:** Use **Spring Boot 4.1.x** (pin 4.1.3 or the latest 4.1 patch at Phase 2 start) on **Java 21 LTS** (Eclipse Temurin). Upgrade to 4.2.x before Phase 5, once springdoc-openapi and Spring Modulith confirm support.
- **Alternatives considered:**
  - *3.5.x:* well known, but no free security patches; a later 3 → 4 migration (Jakarta EE 11, Jackson 3, Hibernate 7, Security 7) would cost more than starting on 4.
  - *4.2.0:* newest, but too fresh for third-party libraries.
  - *Java 25 LTS:* supported by Boot 4, but the brief specifies 21 and nothing in BakeFlow needs 25 (OQ-24).
- **Consequences:**
  - ✅ Supported, modern baseline for the whole project.
  - ⚠️ Many tutorials still show Spring Boot 3 code (for example `com.fasterxml.jackson` imports, `RestTemplate`), so learning material must be checked against version 4.
  - ⚠️ MapStruct and ShedLock compatibility must be confirmed in a Phase 2 spike.
  - ⚠️ Open: 4.1 vs 4.2 (OQ-25) and Java 21 vs 25 (OQ-24).

## ADR-017 – Dependency version policy

- **Status:** Proposed (step 1.2)
- **Context:** Unpinned versions make builds unrepeatable. Brand-new releases sometimes have bugs or, in rare supply-chain attacks, malicious code.
- **Decision:** Pin exact versions (Maven explicit versions; npm `save-exact` plus a committed lock file). Let the Spring Boot BOM manage the libraries it covers; override only with a documented reason. Adopt the latest stable release that has full ecosystem compatibility. Wait 14 days before adopting new releases, except for security fixes. Dependabot opens weekly update PRs. Use LTS runtimes (Java, Node, MySQL). Record each verification date in [technology-stack.md](../01-technology-stack/technology-stack.md#5-version-policy).
- **Alternatives considered:** *version ranges or `latest`* (unrepeatable); *never upgrading* (security risk).
- **Consequences:** ✅ Reproducible builds and controlled upgrades. ⚠️ Upgrades need regular attention (weekly Dependabot review).

## ADR-018 – Maven with the Maven Wrapper

- **Status:** Proposed (step 1.2)
- **Context:** The brief specifies Maven. Different Maven versions on different machines cause "works on my machine" problems.
- **Decision:** Use Maven, always run through the **Maven Wrapper** (`./mvnw`) committed to the repository. No global Maven installation is required.
- **Alternatives considered:** *globally installed Maven* (version drift); *Gradle* (faster and flexible, but not required and less readable for learning).
- **Consequences:** ✅ The same Maven version locally and in CI. ⚠️ The first run downloads Maven, which needs internet access.

## ADR-019 – MySQL 8.4 LTS

- **Status:** Proposed (step 1.2)
- **Context:** The brief requires MySQL 8. **MySQL 8.0 reached end of life on 30-Apr-2026.** MySQL 8.4 is the current Long-Term Support line; 9.x releases are short-lived "Innovation" releases.
- **Decision:** Use **MySQL 8.4 LTS** (Docker image `mysql:8.4`), InnoDB, `utf8mb4` / `utf8mb4_0900_ai_ci`.
- **Alternatives considered:** *8.0* (end of life); *9.x Innovation* (short support windows); *PostgreSQL* (not requested).
- **Consequences:** ✅ Years of security fixes; native arm64 images for Apple Silicon. ⚠️ 8.4 removed some legacy defaults (for example the `mysql_native_password` plugin is disabled by default), so clients must use `caching_sha2_password`, which Connector/J supports.

## ADR-020 – Package-by-module structure enforced by Spring Modulith and ArchUnit

- **Status:** Proposed (step 1.2)
- **Context:** ADR-001 chose a modular monolith. Without automatic checks, module boundaries erode (NFR-42).
- **Decision:** One top-level package per module (`com.bakeflow.<module>`), with sub-packages `api`, `application`, `domain`, `infrastructure`. The **module root package is the published interface** for other modules (facade interfaces, records, events); all sub-packages are internal. **Spring Modulith** verifies module boundaries and cycles in a test; **ArchUnit** enforces layer rules inside modules. Full rules in [coding-conventions.md §2](../01-technology-stack/coding-conventions.md#2-java-backend-structure).
- **Alternatives considered:** *package-by-layer* (`controllers`, `services`, … across the whole app: boundaries invisible); *Maven multi-module* (hard boundaries, but more build ceremony for one developer); *ArchUnit only*.
- **Consequences:** ✅ Boundaries are visible and tested; modules can be extracted later. ⚠️ The word `api` means the HTTP layer, while the module's published API is its root package. This is documented to avoid confusion.

## ADR-021 – No Lombok; Java records for DTOs

- **Status:** Proposed (step 1.2)
- **Context:** Lombok reduces boilerplate, but works by hooking into compiler internals. Its `@Data` and `@EqualsAndHashCode` are risky on JPA entities. Problems with Lombok on Spring Boot 4 projects have been reported.
- **Decision:** **Do not use Lombok.** Request and response DTOs, events and value objects are **Java records**. Entities have explicit (IDE-generated) getters and only the setters or business methods they need.
- **Alternatives considered:** *Lombok everywhere*; *Lombok for entities only*.
- **Consequences:** ✅ No annotation-processor surprises; explicit, debuggable code; no entity `equals`/`hashCode` traps. ⚠️ Somewhat longer entity classes.

## ADR-022 – MapStruct for DTO mapping

- **Status:** Proposed (step 1.2)
- **Context:** Entities must never be returned from the API, so entity-to-DTO mapping is needed throughout.
- **Decision:** Use **MapStruct 1.6.x** (compile-time generated mappers, Spring component model), configured so that **unmapped target properties are compile errors**. Mappers live in each module's `application` package.
- **Alternatives considered:** *hand-written mappers* (no dependency, repetitive); *ModelMapper* (reflection-based, runtime errors).
- **Consequences:** ✅ Fast, type-safe mapping. ⚠️ An open report of a bean-creation issue on Spring Boot 4.0.2 must be checked in the Phase 2 spike; fallback is hand-written mappers.

## ADR-023 – JWT with Spring Security Resource Server (Nimbus), not JJWT

- **Status:** Proposed (step 1.2)
- **Context:** The brief requires JWT. BakeFlow both issues and validates its own tokens.
- **Decision:** Use **Spring Security's OAuth2 Resource Server** support and its **Nimbus JOSE + JWT** encoder and decoder (`NimbusJwtEncoder`, `NimbusJwtDecoder`) for issuing and validating tokens. Sign with **HS256** and a secret of at least 256 bits from an environment variable. Claims: subject (user ID), principal type (`STAFF`/`CUSTOMER`, ADR-009), organization ID, roles, issued-at and expiry. Branch assignments are **looked up server-side**, not trusted from the token (BR-14).
- **Alternatives considered:**
  - *JJWT 0.13.x:* popular and simple, but an extra dependency duplicating built-in support.
  - *RS256 (asymmetric keys):* needed only when other services validate tokens without the secret.
  - *External identity provider (Keycloak):* too heavy for now.
- **Consequences:** ✅ Versions managed by Spring Boot; standard Spring Security integration. ⚠️ HS256 means the same secret signs and verifies, so it must be kept out of code and rotated if leaked. Switch to RS256 if tokens are ever validated elsewhere.

## ADR-024 – Token storage in the browser and same-origin deployment

- **Status:** Proposed (step 1.2; security details elaborated in step 1.7)
- **Context:** Where the SPA keeps tokens decides exposure to XSS (script injection) and CSRF (forged cross-site requests).
- **Decision:** The **access token** (15 minutes) is held **in memory only** (JavaScript variable or React context) and sent in the `Authorization` header. The **refresh token** is an opaque random value in an **HttpOnly, Secure, SameSite=Strict cookie** scoped to the refresh endpoint path; it is stored **hashed** in MySQL and rotated on every use. The SPA and API are served from the **same origin** (Vite dev proxy locally, a reverse proxy in production), so normal operation needs no CORS. CORS stays configured with an explicit, empty-by-default allow-list.
- **Alternatives considered:** *tokens in `localStorage`* (readable by any injected script); *everything in cookies with server sessions* (stateful, needs CSRF tokens on all writes).
- **Consequences:** ✅ Injected scripts cannot read the refresh token; SameSite=Strict blocks cross-site use of the cookie. ⚠️ A page reload loses the in-memory access token, so the app calls `/auth/refresh` on start-up. ⚠️ The refresh endpoint must also check the `Origin` header.

## ADR-025 – ShedLock for scheduled jobs

- **Status:** Proposed (step 1.2)
- **Context:** Scheduled jobs (batch expiry, reservation release, no-show cancellation, nightly reconciliation) must not run twice at the same time once more than one backend instance is running (NFR-35).
- **Decision:** Use **ShedLock** with its JDBC lock provider (a `shedlock` table in MySQL) on every `@Scheduled` job, from module 3.6 onward. Jobs are also written to be **idempotent**, so a repeated run causes no harm.
- **Alternatives considered:** *no locking* (fine for one instance, unsafe later); *Quartz clustered scheduler* (heavier); *database advisory locks written by hand*.
- **Consequences:** ✅ Safe horizontal scaling. ⚠️ ShedLock 7.x compatibility with Spring Boot 4.1 must be confirmed in the Phase 2 spike.

## ADR-026 – Single React SPA built with Vite, not Next.js

- **Status:** Proposed (step 1.2)
- **Context:** Three user areas (customer, branch, admin). Most screens are behind a login; public pages are a branch catalog.
- **Decision:** **One** React + TypeScript SPA built with **Vite**, with three **lazy-loaded route areas** (`/shop`, `/branch`, `/admin`), so staff code is not downloaded by customers and the other way round. It is deployed as static files behind the reverse proxy.
- **Alternatives considered:** *Next.js* (SSR and SEO, but adds a Node server and a second logic layer); *three separate SPAs* (stronger isolation, triple builds).
- **Consequences:** ✅ Simple build and hosting; one shared design system. ⚠️ Limited SEO for public catalog pages; revisit if organic search becomes important.

## ADR-027 – Frontend data layer: TanStack Query, Axios, React Hook Form, Zod; no Redux

- **Status:** Proposed (step 1.2)
- **Context:** Most frontend state is server data; forms are central (POS, stock receipt, admin).
- **Decision:** **TanStack Query** for all server state; **Axios** as the single HTTP client with interceptors (token, correlation ID, refresh, error conversion); **React Hook Form + Zod** for forms and validation; React context for the little global client state (current user, selected branch). **No Redux.**
- **Alternatives considered:** *Redux Toolkit / RTK Query*; *SWR*; *Formik + Yup*; *fetch without a library*.
- **Consequences:** ✅ Less code; consistent loading, error and caching behaviour. ⚠️ Developers must learn query keys and invalidation.

## ADR-028 – REST API conventions

- **Status:** Proposed (step 1.2)
- **Context:** Consistent APIs are easier to use, document and test. Step 1.5 needs fixed rules.
- **Decision:** `/api/v1` prefix; plural `kebab-case` resources; branch-scoped URLs `/api/v1/branches/{branchId}/…` with server-side verification; organization taken from the token, never the URL; `camelCase` JSON; **money as decimal strings**; ISO-8601 **UTC** timestamps; `YYYY-MM-DD` business dates; zero-based pagination with a fixed page envelope; **RFC 9457 Problem Details** errors extended with `code`, `traceId` and `errors`; `Idempotency-Key` header; **404 for resources of another organization** (no existence leak). Full rules in [coding-conventions.md §5](../01-technology-stack/coding-conventions.md#5-rest-api-conventions).
- **Alternatives considered:** *money as JSON numbers* (risk of binary floating-point rounding in JavaScript); *custom error envelope* (non-standard); *GraphQL* (more flexible queries, more complexity).
- **Consequences:** ✅ Predictable APIs; standard error format supported by Spring. ⚠️ The frontend must parse money strings for display, and never for authoritative arithmetic.

## ADR-029 – Database naming conventions and Flyway versioning

- **Status:** Proposed (step 1.2)
- **Context:** Step 1.3 needs fixed naming and type rules. The MVP build order (MVP-A modules 3.12 and 3.16 before MVP-B modules 3.9–3.11) breaks any migration numbering tied to module numbers.
- **Decision:** Singular `snake_case` tables; `id BIGINT` primary keys; `<table>_id` foreign keys; constraint prefixes `pk_` (documentation only, since MySQL names primary keys `PRIMARY`), `fk_`, `uk_`, `idx_`, `ck_`; standard columns `created_at`, `created_by`, `updated_at`, `updated_by`, `version`, with append-only tables keeping only the `created_*` columns; `DECIMAL(12,2)` money; `DATETIME(6)` UTC timestamps; `VARCHAR(30)` status values enforced by check constraints (no MySQL `ENUM`). Flyway files are named **`V<yyyyMMddHHmm>__<module>_<description>.sql`**. Full rules in [coding-conventions.md §4](../01-technology-stack/coding-conventions.md#4-database-conventions).
- **Alternatives considered:** *plural table names*; *sequential migration numbers* (collide on parallel branches); *module-number versions* (out-of-order failures); *MySQL `ENUM`* (alters the table to change, sorts by position).
- **Consequences:** ✅ Consistent schema that matches entity names. ⚠️ The 64-character identifier limit needs consistent abbreviations for long constraint names.

## ADR-030 – Redis for caching only, introduced in Phase 4

- **Status:** Proposed (step 1.2)
- **Context:** The brief lists Redis. Stock correctness depends on transactions and row locks (ADR-008), which Redis cannot provide together with MySQL data.
- **Decision:** Introduce **Redis 8** in **Phase 4** through Spring's cache abstraction and Spring Data Redis, for catalog and availability-display caching (and optionally idempotency look-ups and rate-limit counters). Redis is **never** the source of truth for stock, reservations, invoices, payments or orders, and never used for stock locks. The application must work, more slowly, if Redis is down (NFR-12).
- **Alternatives considered:** *Redis from Phase 2* (complexity before it is needed); *Caffeine in-process cache* (simpler, but not shared across instances; may be used for small reference data); *Valkey* (BSD-licensed drop-in alternative to Redis).
- **Consequences:** ✅ Simpler MVP; a clear rule for what may be cached. ⚠️ Cache invalidation rules must be defined in Phase 4.

## ADR-031 – File storage behind an interface; local disk first

- **Status:** Proposed (step 1.2; resolves AS-34)
- **Context:** Product images must not be stored in the database (AS-34). Hosting is not chosen yet (step 1.7).
- **Decision:** A `FileStorage` interface in `shared` with a **local-disk implementation** (a Docker volume) for development and MVP, and an **S3-compatible implementation** (AWS S3, Cloudflare R2, or MinIO for local testing) added when hosting is chosen. The database stores only the file key and metadata. Uploads are validated (image types only, size limit, re-encoded or checked).
- **Alternatives considered:** *images in MySQL BLOBs* (bloats backups); *S3 from day one* (cloud account needed early).
- **Consequences:** ✅ No cloud dependency during development; the swap is a configuration change. ⚠️ A local volume must be backed up and is a single point of failure until moved to object storage.

## ADR-032 – Invoice printing: browser print first, server PDF later

- **Status:** Proposed (step 1.2)
- **Context:** Counter staff need printed invoices on 80 mm thermal printers or A4 (NFR-40). Customers may later download PDF invoices (FR-CUS-25, post-MVP).
- **Decision:** **MVP:** invoices are HTML views with **print CSS** for 80 mm and A4, printed with the browser's print dialog (the thermal printer is installed as a system printer). **Post-MVP:** server-side PDFs rendered from the same HTML template with **OpenHTMLtoPDF**.
- **Alternatives considered:** *iText* (AGPL or commercial licence); *JasperReports* (heavy designer tooling); *ESC/POS direct printer commands* (fast, but needs a local agent or WebUSB); *client-side PDF libraries* (inconsistent output).
- **Consequences:** ✅ No server PDF dependency in the MVP; one template for screen and paper. ⚠️ Printing goes through the browser dialog (kiosk-mode printing can be configured per counter PC).

## ADR-033 – Email via Spring Mail; Mailpit in development

- **Status:** Proposed (step 1.2)
- **Context:** Password reset is in MVP-B (AS-58); notifications come later (3.15). Development must never send real emails.
- **Decision:** Use **Spring Mail** (SMTP) behind a small `EmailSender` interface. In development and tests, emails go to **Mailpit** (SMTP on port 1025, web inbox on port 8025). The production SMTP provider is chosen in step 1.7. Emails are sent **after** the transaction commits (transactional event listener), never inside stock-locking transactions.
- **Alternatives considered:** *a provider's HTTP API SDK directly* (ties code to one vendor); *MailHog* (unmaintained; Mailpit is its active successor).
- **Consequences:** ✅ Safe local testing; provider-independent. ⚠️ Production deliverability (SPF, DKIM, DMARC) must be set up in step 1.7.

## ADR-034 – Razorpay integration through the official Java SDK behind PaymentGateway

- **Status:** Proposed (step 1.2; extends ADR-007)
- **Context:** ADR-007 chose Razorpay. We need order creation, payment signature verification, webhook signature verification and refunds.
- **Decision:** Use the official **`razorpay-java`** SDK, wrapped by BakeFlow's `PaymentGateway` interface in the `payment` module. Signatures are verified with the SDK's utilities. In tests, Razorpay's HTTP API is replaced by **WireMock**. Keys come from environment variables (test keys in development).
- **Alternatives considered:** *calling the REST API with `RestClient`* (full control, more code); *a third-party wrapper library*.
- **Consequences:** ✅ Less code; officially maintained signature checks. ⚠️ The SDK brings its own HTTP and JSON libraries, so check for dependency conflicts when it is added (module 3.10).

---

## How to Add a New ADR

1. Copy the structure of an existing ADR and use the next number (ADR-016 …).
2. Set **Status: Proposed** and discuss it.
3. When agreed, change it to **Accepted** and add it to the index.
4. To change an earlier decision, write a new ADR and mark the old one `Superseded by ADR-xxx`. Never delete ADRs.

**Expected upcoming ADRs:** async event mechanism details (Phase 4), hosting platform (step 1.7).
