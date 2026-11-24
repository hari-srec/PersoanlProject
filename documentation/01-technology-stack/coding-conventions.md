# BakeFlow – Coding Conventions

> **Step:** 1.2 · **Status:** Draft for owner review · **Applies to:** all code from Phase 2 onward, and to the database design in step 1.3

## Table of Contents

1. [Purpose and Principles](#1-purpose-and-principles)
2. [Java Backend Structure](#2-java-backend-structure)
3. [Java Coding Rules](#3-java-coding-rules)
4. [Database Conventions](#4-database-conventions)
5. [REST API Conventions](#5-rest-api-conventions)
6. [Frontend Conventions](#6-frontend-conventions)
7. [Git Conventions](#7-git-conventions)
8. [Traceability in Code and Tests](#8-traceability-in-code-and-tests)
9. [Related Documents](#9-related-documents)

---

## 1. Purpose and Principles

Conventions make the code base look as if one careful person wrote it. They are checked automatically wherever possible: Spring Modulith, ArchUnit, Spotless, ESLint, Prettier and CI.

Principles:

1. **Consistency over personal taste.** If a rule exists, follow it; if it is wrong, change the rule here first.
2. **Explicit over clever.** Readable code wins over short code (this is a learning project).
3. **Boundaries are real.** Modules talk only through published interfaces ([ADR-020](../02-system-design/design-decisions.md#adr-020--package-by-module-structure-enforced-by-spring-modulith-and-archunit)).
4. **The server is the authority.** Prices, tax, stock and permissions are decided on the backend; the frontend only displays them.

---

## 2. Java Backend Structure

### 2.1 Package-by-module

The base package is `com.bakeflow`. Each business module is a **top-level package** under it. Spring Modulith treats each one as an application module.

| Package | Roadmap | Contents |
|---------|---------|----------|
| `com.bakeflow.shared` | 2 | Cross-cutting building blocks: money and time helpers, base exception, error codes, Problem Details handler, idempotency support, security context access. *Shared* type in Spring Modulith (usable by all) |
| `com.bakeflow.audit` | 2, 3.15 | Audit framework (capture, actor resolution); audit viewing later |
| `com.bakeflow.auth` | 3.1 | Login, tokens, refresh, password hashing, roles and permissions |
| `com.bakeflow.organization` | 3.2 | Organization, branch, staff user, branch assignment, settings |
| `com.bakeflow.catalog` | 3.3 | Category, product, variant, HSN code and GST rates, prices, branch product availability |
| `com.bakeflow.inventory` | 3.4–3.6 | Batch, stock ledger, FEFO allocation, reservations, adjustments, expiry job, alerts |
| `com.bakeflow.production` | 3.7 | Production records and plans |
| `com.bakeflow.billing` | 3.8 | POS bills, invoices, credit notes, document sequences, payment records, daily closing |
| `com.bakeflow.customer` | 3.9 | Customer accounts, profile, addresses, password reset |
| `com.bakeflow.ordering` | 3.10–3.11 | Cart, checkout, customer orders, status history, no-show handling |
| `com.bakeflow.payment` | 3.10 | Online payment gateway integration (Razorpay), webhooks, refunds via gateway |
| `com.bakeflow.discount` | 3.12 | Discount rules, recommendations, approvals, applied discounts |
| `com.bakeflow.wastage` | 3.13 | Wastage records and reasons |
| `com.bakeflow.transfer` | 3.14 | Inter-branch transfers |
| `com.bakeflow.notification` | 3.15 | In-app and email notifications |
| `com.bakeflow.reporting` | 3.16 | Reports and summaries |
| `com.bakeflow.forecasting` | 3.17 | Demand forecasting, production recommendations |
| `com.bakeflow.loyalty` | 3.18 | Loyalty points, favourites, reviews |

Module names are singular nouns in lower case. `order` is avoided as a name because it is a SQL reserved word and too generic; `ordering` is used instead.

### 2.2 Inside a module

```
com.bakeflow.inventory
├── StockAllocationApi.java        ← published interface for other modules (module root = public API)
├── AllocationRequest.java         ← published records used by that interface
├── AllocationResult.java
├── StockReserved.java             ← published domain event
├── api/                           ← HTTP layer (REST controllers)
│   ├── InventoryBatchController.java
│   └── dto/
│       ├── CreateStockReceiptRequest.java
│       └── InventoryBatchResponse.java
├── application/                   ← use cases, @Transactional services, mappers
│   ├── StockReceiptService.java
│   ├── StockAllocationService.java   (implements StockAllocationApi)
│   └── InventoryBatchMapper.java     (MapStruct)
├── domain/                        ← entities, enums, value objects, pure business logic
│   ├── InventoryBatch.java
│   ├── BatchStatus.java
│   ├── StockMovement.java
│   └── FefoAllocator.java
└── infrastructure/                ← technical adapters
    ├── persistence/
    │   └── InventoryBatchRepository.java  (Spring Data JPA)
    └── scheduling/
        └── BatchExpiryJob.java
```

| Layer | Responsibility | May depend on |
|-------|----------------|---------------|
| **module root** | The module's **published interface** for other modules: facade interfaces (`XApi`), records they use, and domain events | Its own `application` (implementation) |
| `api` | REST controllers, request and response DTOs, input validation annotations, HTTP status choices. **No business logic** | `application`, `api.dto`, `shared` |
| `application` | Use cases; **transaction boundaries** (`@Transactional`); authorization checks for branch scope; orchestration; MapStruct mappers from entities to response DTOs | `domain`, `infrastructure`, `api.dto`, other modules' **root** packages, `shared` |
| `domain` | JPA entities, enums, value objects, domain services with pure logic (for example FEFO ordering), domain exceptions | `shared` only |
| `infrastructure` | Spring Data repositories, external clients (Razorpay), schedulers, module configuration | `domain`, `application`, `shared` |

**Important terminology note:** in this project the sub-package `api` means the **HTTP/REST layer**. The module's **published API for other modules** is the **module root package**. Spring Modulith treats every sub-package as internal, so other modules can never reach `api`, `application`, `domain` or `infrastructure` directly.

**Rules checked by tests:**

- **Spring Modulith** (`ApplicationModules.verify()`): no access to another module's internal packages; no cyclic dependencies between modules.
- **ArchUnit:**
  - controllers do not use repositories;
  - `domain` does not depend on `api`, `application`, `infrastructure` or Spring Web;
  - `@Transactional` appears only in `application`;
  - entities are never returned from controllers;
  - no class in `api` is called by another layer except `api.dto` records.

**Why services return response DTOs:** mapping happens inside the transaction, so lazy-loaded data is available and Open Session in View can stay disabled (see [technology-justification.md §4.3](technology-justification.md#43-what-happens-in-one-transactional-request)). This is a pragmatic choice that avoids a third set of DTO classes.

### 2.3 Class naming

| Kind | Pattern | Example |
|------|---------|---------|
| REST controller | `XController` | `PosBillController` |
| Application service | `XService` | `PosBillingService` |
| Published module interface | `XApi` | `StockAllocationApi` |
| Spring Data repository | `XRepository` | `InventoryBatchRepository` |
| JPA entity | Domain noun, **no suffix** | `InventoryBatch`, `Invoice` |
| Enum | Noun | `BatchStatus`, `PaymentMethod` |
| Request DTO (record) | `XRequest` (verb + noun for commands) | `CreateBillRequest`, `ApproveAdjustmentRequest` |
| Response DTO (record) | `XResponse` | `InvoiceResponse`, `InventoryBatchResponse` |
| Paged response | `PageResponse<T>` (shared) | `PageResponse<InvoiceResponse>` |
| MapStruct mapper | `XMapper` | `InvoiceMapper` |
| Domain event (record, past tense) | `XHappened` style | `StockReserved`, `InvoiceIssued` |
| Scheduled job | `XJob` | `BatchExpiryJob`, `ReservationReleaseJob` |
| Configuration | `XConfig`; typed properties `XProperties` | `SecurityConfig`, `ReservationProperties` |
| Exception | `XException` | `InsufficientStockException` |
| Unit test | `XTest` | `FefoAllocatorTest` |
| Integration test (Testcontainers) | `XIT` | `StockAllocationConcurrencyIT` |
| Architecture test | `ArchitectureTest`, `ModularityTest` | – |

Methods: verbs in camelCase (`allocateStock`, `issueInvoice`). Booleans: `isX`, `hasX`, `canX`. Constants: `UPPER_SNAKE_CASE`.

---

## 3. Java Coding Rules

| Topic | Rule |
|-------|------|
| **No entities in API responses** | Controllers return only `XResponse` records. Entities never leave the `application` layer |
| DTOs | Java **records**, immutable, with Bean Validation annotations on request records |
| Lombok | Not used ([ADR-021](../02-system-design/design-decisions.md#adr-021--no-lombok-java-records-for-dtos)) |
| Dependency injection | **Constructor injection** only, fields `private final`; no `@Autowired` on fields |
| Transactions | `@Transactional` on public `application` service methods only; `readOnly = true` for queries; no external HTTP calls inside stock-locking transactions (NFR-28) |
| Money | `BigDecimal`, scale 2, `RoundingMode.HALF_UP`; never `double`/`float`; compare with `compareTo`, not `equals` (ADR-004) |
| Time | `Instant` for timestamps (UTC); `LocalDate` for IST business dates; zone constant `ZoneId.of("Asia/Kolkata")` in `shared`; never `java.util.Date` or `LocalDateTime` for stored instants (ADR-005). Use an injectable `Clock` so tests can control time |
| Null handling | Return `Optional` only from look-up methods; never pass `Optional` as a parameter; mark packages `@NullMarked` (JSpecify, supported by Spring Boot 4) |
| Enums in JPA | `@Enumerated(EnumType.STRING)` always, never ordinal |
| Entities | `equals`/`hashCode` based on ID with the standard Hibernate-safe pattern; no bidirectional relationships unless needed; lazy loading by default |
| Exceptions | Business errors extend a shared `BakeflowException` carrying an **error code** (`INVENTORY_INSUFFICIENT_STOCK`); a single `@RestControllerAdvice` converts them to Problem Details (ADR-028). Never swallow exceptions |
| Logging | SLF4J with placeholders (`log.info("Bill {} issued", id)`); no personal data, tokens or passwords in logs (NFR-45) |
| Configuration | Typed `@ConfigurationProperties` records; secrets only from environment variables |
| Formatting | Enforced by the **Spotless** Maven plugin with **Palantir Java Format** (4-space indent, 120 columns); exact plugin versions are chosen in Phase 2 |
| Comments | Explain *why*, not *what*. Public module interfaces (`XApi`) have Javadoc |

---

## 4. Database Conventions

These rules are binding for step 1.3 ([ADR-029](../02-system-design/design-decisions.md#adr-029--database-naming-conventions-and-flyway-versioning)).

### 4.1 Names

| Rule | Example |
|------|---------|
| Tables and columns in **`snake_case`**, lower case, English | `inventory_batch`, `expires_at` |
| **Singular** table names | `branch`, `invoice_line`, `stock_movement` |
| Avoid SQL reserved words by qualifying the name | `customer_order` (not `order`), `staff_user` (not `user`) |
| Join tables: both entity names, alphabetical where no better name exists | `staff_user_branch` (or a meaningful name such as `branch_assignment`) |
| Primary key column is always `id` | `id` |
| Foreign key column = `<referenced_table>_id` | `branch_id`, `inventory_batch_id` |
| Two foreign keys to the same table get a role prefix | `source_branch_id`, `destination_branch_id`, `approved_by_staff_user_id` |
| Booleans start with `is_` or `has_` | `is_active`, `has_delivery` |
| Timestamps end with `_at`; business dates end with `_date` | `created_at`, `business_date` |

**Why singular table names:** a table row represents one thing; the name matches the JPA entity class (`InventoryBatch` → `inventory_batch`); foreign keys read naturally (`inventory_batch_id`); it matches the master prompt's own example (`inventory_batch`); and it avoids irregular plurals (`category`/`categories`, `wastage`). Both styles are common; consistency is what matters.

### 4.2 Standard columns

Every **business table** has `organization_id` (NFR-34), except platform-level tables such as `organization` itself.

| Table kind | Standard columns |
|-----------|------------------|
| **Mutable** (master and transactional data that can change: product, batch, order) | `id`, `created_at`, `created_by`, `updated_at`, `updated_by`, `version` |
| **Append-only** (never updated: `stock_movement`, `audit_log`, `order_status_history`, invoice lines after issue) | `id`, `created_at`, `created_by` |

- `created_by` / `updated_by` are `VARCHAR(64)` **actor keys**: `STAFF:12`, `CUSTOMER:88` or `SYSTEM` (matching the audit framework, where Phase 2 records `SYSTEM` and 3.1 adds real users). Business columns that must reference a specific person (for example who approved an adjustment) use a real foreign key, such as `approved_by_staff_user_id`.
- `version` is `BIGINT NOT NULL DEFAULT 0`, used by JPA `@Version` (ADR-008).
- Master data is **deactivated**, not deleted (`is_active` or a status). Financial and stock records are **never deleted**.

### 4.3 Data types

| Data | MySQL type | Java type | Notes |
|------|-----------|-----------|-------|
| Primary / foreign keys | `BIGINT` (`AUTO_INCREMENT` for PK) | `Long` | Signed, to match Java `long` |
| Money | `DECIMAL(12,2)` | `BigDecimal` | ADR-004 |
| Percentages (GST rate, discount %) | `DECIMAL(5,2)` | `BigDecimal` | `18.00` means 18 % |
| Quantities | `INT` | `int` / `Integer` | Whole units (AS-39); `CHECK (x >= 0)` where negative is impossible |
| Timestamps | **`DATETIME(6)`**, holding **UTC** | `Instant` | Microsecond precision; never `TIMESTAMP` (2038 limit, session-time-zone conversion) (ADR-005) |
| Business dates (IST day) | `DATE` | `LocalDate` | For example `business_date` of a daily closing |
| Short codes | `VARCHAR(n)` | `String` | Branch code `VARCHAR(3)`, HSN code `VARCHAR(8)` |
| Names, titles | `VARCHAR(150)` | `String` | Adjust per column in the data dictionary |
| Email | `VARCHAR(254)` | `String` | Stored lower-case |
| Phone | `VARCHAR(15)` | `String` | E.164 format, for example `+919876543210` |
| Status / type | `VARCHAR(30)` | Java `enum` | See 4.5 |
| Long text | `TEXT` | `String` | Only for free text (descriptions, comments) |
| Structured snapshots | `JSON` | `String` / mapped type | Only for audit before/after values and similar |
| Idempotency keys, external IDs | `VARCHAR(64)` | `String` | UUIDs as text |

Character set `utf8mb4`, collation `utf8mb4_0900_ai_ci`, engine InnoDB for every table.

### 4.4 Constraint and index names

| Object | Pattern | Example |
|--------|---------|---------|
| Primary key | `pk_<table>` (**documentation only**) | `pk_inventory_batch` |
| Foreign key | `fk_<table>_<column>` | `fk_inventory_batch_branch_id` |
| Unique constraint | `uk_<table>_<col1>[_<col2>…]` | `uk_inventory_batch_branch_id_batch_number` |
| Index | `idx_<table>_<col1>[_<col2>…]` | `idx_inventory_batch_branch_id_variant_id_expires_at` |
| Check constraint | `ck_<table>_<rule>` | `ck_inventory_batch_quantities_non_negative` |

- **MySQL always names a primary key `PRIMARY`** and ignores any name given. The `pk_` name is used in ER diagrams and the data dictionary for consistency only.
- MySQL identifiers have a **64-character limit**. If a name would exceed it, shorten words consistently (`quantity → qty`, `organization → org`) and note the abbreviation in the data dictionary.
- Every foreign key column gets an index (InnoDB creates one automatically if missing; we name it explicitly).

### 4.5 Status and enum columns

- Stored as `VARCHAR(30)` with **UPPER_SNAKE_CASE** values: `ACTIVE`, `EXPIRED`, `READY_FOR_PICKUP`.
- Allowed values are **documented in the data dictionary** (step 1.3) and **enforced** by a check constraint, for example `ck_inventory_batch_status CHECK (status IN ('ACTIVE','BLOCKED','EXPIRED','DEPLETED'))`, mirrored by a Java enum.
- Adding a value needs a Flyway migration that updates the check constraint, plus the enum change, in the same pull request.
- MySQL `ENUM` columns are **not** used: changing them alters the table and they sort by position, not by name.

### 4.6 Flyway migrations

| Rule | Detail |
|------|--------|
| Location | `src/main/resources/db/migration` |
| File name | `V<yyyyMMddHHmm>__<module>_<description>.sql`, for example `V202612011030__inventory_create_inventory_batch.sql` |
| Why timestamps, not 1, 2, 3 or module numbers | Modules are built in MVP-A order (3.12 and 3.16 before 3.9–3.11). Module-number versions such as `V3_9_…` would be lower than versions already applied, and Flyway would reject them as out of order. Timestamps always increase in the order work is done |
| One change per file | Create one table (with its constraints and indexes) or one alteration per migration |
| Never edit an applied migration | Fix forward with a new migration |
| Reference data | In migrations (for example system wastage reasons): `V…__wastage_seed_system_reasons.sql` |
| Development sample data | Separate location `db/devdata`, enabled only in the `dev` profile; never in production |
| SQL style | Keywords UPPER CASE, identifiers lower case, one column per line, explicit constraint names |

---

## 5. REST API Conventions

Detailed endpoints come in step 1.5. These rules apply to all of them ([ADR-028](../02-system-design/design-decisions.md#adr-028--rest-api-conventions)).

### 5.1 URLs

| Rule | Example |
|------|---------|
| Prefix and version | `/api/v1/...` (a breaking change creates `/api/v2/...`) |
| **Plural nouns**, `kebab-case` for multiple words | `/products`, `/stock-adjustments` |
| Identifiers as path segments | `/products/{productId}` |
| Organization is **never** in the URL for organization users; it comes from the token | `/api/v1/products` (organization from the JWT) |
| **Branch-scoped** resources sit under the branch, which is **always verified server-side** (BR-14) | `/api/v1/branches/{branchId}/inventory/batches`, `/api/v1/branches/{branchId}/pos/bills` |
| State transitions that are not plain updates use a **verb sub-resource** with `POST` | `POST /api/v1/branches/{branchId}/stock-adjustments/{id}/approve` |

**URL families**

| Family | Prefix | Who |
|--------|--------|-----|
| Authentication | `/api/v1/auth/...` | Everyone |
| Platform administration | `/api/v1/platform/...` | SUPER_ADMIN |
| Organization-level | `/api/v1/{resource}` (for example `/branches`, `/products`, `/discount-rules`, `/reports/...`) | ORG_ADMIN (some read access for branch roles) |
| Branch-scoped | `/api/v1/branches/{branchId}/...` | Branch roles and ORG_ADMIN |
| Customer self-service | `/api/v1/me/...` (`/me/profile`, `/me/cart`, `/me/orders`) | CUSTOMER (own data only) |
| Public storefront (no login) | `/api/v1/storefront/...` (`/storefront/branches/{branchId}/products`) | Anyone |
| Webhooks | `/api/v1/webhooks/razorpay` | Razorpay (signature-verified) |

### 5.2 JSON

| Rule | Example |
|------|---------|
| Property names in `camelCase` | `"expiresAt"`, `"availableQuantity"` |
| Enum values in `UPPER_SNAKE_CASE` | `"status": "READY_FOR_PICKUP"` |
| IDs as JSON numbers | `"id": 5531` |
| **Money as strings** with two decimals | `"totalAmount": "180.00"` |
| Quantities as integers | `"quantity": 2` |
| Timestamps as **ISO-8601 UTC** with `Z`, field name ends in `At` | `"issuedAt": "2026-11-24T11:42:07.512Z"` |
| Business dates as `YYYY-MM-DD` (IST day), field name ends in `Date` | `"businessDate": "2026-11-24"` |
| Absent optional values | Sent as `null` (fields are not omitted), so response shapes are predictable |

### 5.3 Pagination, sorting, filtering

- **Request:** `?page=0&size=20&sort=expiresAt,asc` (page numbers start at 0; default size 20; maximum 100; several `sort` parameters allowed).
- **Response:**

```json
{
  "content": [ { "id": 1 } ],
  "page": { "number": 0, "size": 20, "totalElements": 57, "totalPages": 3 }
}
```

- **Filters** are query parameters named after fields: `?status=ACTIVE&variantId=1012`. Ranges use `From`/`To` suffixes: `?businessDateFrom=2026-11-01&businessDateTo=2026-11-24`, `?expiresAtTo=2026-11-25T00:00:00Z`.

### 5.4 Headers and status codes

| Header | Use |
|--------|-----|
| `Authorization: Bearer <token>` | Every protected request |
| **`Idempotency-Key: <uuid>`** | **Required** on POST requests that create bills, payments, refunds, checkout reservations and orders (ADR-011) |
| `X-Correlation-Id` | Optional on requests (generated if missing); always returned on responses and written to logs |
| `Location` | On `201 Created`, the URL of the new resource |

| Status | Meaning |
|--------|---------|
| 200 OK / 201 Created / 204 No Content | Success |
| 400 Bad Request | Malformed request or field validation failure |
| 401 Unauthorized | Not logged in, or token expired |
| 403 Forbidden | Logged in but not allowed (for example a branch not assigned to the user) |
| 404 Not Found | Does not exist, **or belongs to another organization** (existence is not revealed) |
| 409 Conflict | State conflict: insufficient stock, concurrent change, idempotency key in progress |
| 422 Unprocessable Content | Valid request but violates a business rule (for example refund window passed), or idempotency key reused with a different body |
| 429 Too Many Requests | Rate limit (login throttling) |
| 500 Internal Server Error | Unexpected error (details only in logs) |

Errors use RFC 9457 Problem Details with `code`, `traceId` and `errors` (example in [technology-justification.md §3.4](technology-justification.md#34-standard-error-format)). Error codes follow `<MODULE>_<ERROR>`, for example `INVENTORY_INSUFFICIENT_STOCK`, `BILLING_REFUND_WINDOW_EXPIRED`, `AUTH_INVALID_CREDENTIALS`.

---

## 6. Frontend Conventions

### 6.1 Folder structure

```
frontend/src/
├── app/                 # App.tsx, router.tsx, providers.tsx (QueryClient, auth), error boundary
├── areas/               # route trees and layouts per area – lazy-loaded
│   ├── shop/            #   customer platform  (/shop/...)
│   ├── branch/          #   branch management  (/branch/...)
│   └── admin/           #   central admin      (/admin/...)
├── features/            # one folder per business feature
│   └── inventory/
│       ├── api/         #   inventoryApi.ts (HTTP calls), inventoryKeys.ts (query keys)
│       ├── hooks/       #   useBatchesQuery.ts, useCreateStockReceiptMutation.ts
│       ├── components/  #   BatchTable.tsx, ExpiryBadge.tsx
│       ├── pages/       #   BatchListPage.tsx
│       ├── schemas/     #   stockReceiptSchema.ts (Zod)
│       ├── types.ts
│       └── index.ts     #   public exports of the feature
├── shared/
│   ├── components/ui/   # Button, Input, Dialog, Table … (Tailwind-styled primitives)
│   ├── components/layout/
│   ├── lib/             # httpClient.ts, queryClient.ts, money.ts, dates.ts, apiError.ts
│   ├── hooks/
│   └── types/           # api.generated.ts (from OpenAPI), common types
├── styles/index.css     # Tailwind entry
└── test/                # setup.ts, MSW handlers
```

Features mirror backend modules where sensible (`inventory`, `billing`, `ordering`). A feature may import from `shared` and from another feature's `index.ts` only, never from its internal files.

### 6.2 Naming

| Kind | Pattern | Example |
|------|---------|---------|
| Component file and name | `PascalCase.tsx` | `BatchTable.tsx` → `BatchTable` |
| Page component | `XPage` | `PosBillingPage` |
| Hook | `useX.ts` | `useCurrentBranch.ts` |
| Query hook / mutation hook | `useXQuery` / `useXMutation` | `useBatchesQuery`, `useCreateBillMutation` |
| API module | `xApi.ts` with verb functions | `billingApi.createBill()` |
| Query key factory | `xKeys` | `inventoryKeys.batches(branchId, filters)` |
| Zod schema | `xSchema` | `createBillSchema` |
| Types / interfaces | `PascalCase`, no `I` prefix | `InventoryBatch` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_PAGE_SIZE` |
| Tests | `X.test.ts(x)` next to the file | `BatchTable.test.tsx` |
| Exports | **Named exports** only; route modules expose `Component` for React Router lazy loading | `export function BatchTable() {}` |

### 6.3 API layer rules

1. **One Axios instance** (`shared/lib/httpClient.ts`), base URL `/api/v1`. Interceptors: attach the access token, add `X-Correlation-Id`, refresh once on `401` then retry, and convert Problem Details into an `ApiError` (with `code`, `status`, `fieldErrors`).
2. **Only `features/*/api` files call `httpClient`.** Components never call Axios directly; they use query or mutation hooks.
3. **TanStack Query** for all server data. Query keys come from key factories. Mutations invalidate the affected keys.
4. **Idempotency:** mutation hooks for bills, payments, refunds and checkout generate one `Idempotency-Key` (`crypto.randomUUID()`) per user action and reuse it on retries.
5. **Money:** kept as strings from the API. Formatted with `Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' })`. Never used for authoritative arithmetic.
6. **Dates:** displayed with `Intl.DateTimeFormat('en-IN', { timeZone: 'Asia/Kolkata', … })` as `DD-MM-YYYY` and `hh:mm AM/PM` (NFR-49).
7. **Types:** generated from the backend OpenAPI spec (`openapi-typescript`) into `shared/types/api.generated.ts`; never edited by hand.
8. **Environment variables:** only `VITE_`-prefixed, and only non-secret values (anything in the frontend bundle is public).
9. **Tokens:** the access token is held in memory (auth context); nothing goes in `localStorage` (ADR-024).

### 6.4 Styling and UI

- Tailwind utility classes; reusable look-and-feel lives in `shared/components/ui`.
- No inline `style` attributes except for truly dynamic values.
- Mobile-first for `/shop` (works from 360 px, NFR-37); keyboard-first for the POS (NFR-36).
- Semantic HTML and labelled form fields (NFR-39).
- TypeScript `strict: true`; no `any` (ESLint rule); Prettier formats on save and in CI.

---

## 7. Git Conventions

### 7.1 Branches

| Branch | Purpose | Example |
|--------|---------|---------|
| `main` | Always releasable; protected (changes via pull request with green CI) | – |
| `docs/<step>-<topic>` | Documentation steps | `docs/1.2-technology-stack` |
| `phase/<n>-<topic>` | Whole-phase work | `phase/2-setup` |
| `module/<id>-<name>` | One Phase 3 module | `module/3.1-auth`, `module/3.12-discounts` |
| `fix/<short-description>` | Bug fixes | `fix/fefo-tie-breaker` |
| `chore/<short-description>` | Tooling, dependency updates | `chore/bump-spring-boot-4.1.4` |

Large modules may use short-lived sub-branches (`module/3.8-pos-invoice-numbering`) that merge into the module branch.

### 7.2 Commit messages – Conventional Commits

```
<type>(<scope>): <summary in imperative mood, max 72 characters>

<body: why the change was made, not what – optional>

Refs: FR-BR-20, BR-13, ADR-010
```

- **Types:** `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`, `chore`, `revert`.
- **Scope:** the module (`inventory`, `billing`), or `docs`, `infra`, `frontend`, `deps`.
- **Breaking changes:** add `!` after the scope (`feat(api)!: …`) and a `BREAKING CHANGE:` footer.

Examples:

```
feat(inventory): allocate stock by FEFO with row locking
fix(billing): keep invoice counter gap-free on rollback
docs(1.2): add technology stack and coding conventions
test(inventory): add 50-thread oversell concurrency test
chore(deps): bump spring-boot to 4.1.4
```

Pull requests are **squash-merged**, so each PR becomes one Conventional Commit on `main`.

### 7.3 Tags

| Tag | When | Example |
|-----|------|---------|
| `step-<id>` (annotated) | When a roadmap step is accepted by the owner | `step-1.1`, `step-1.2`, `step-2`, `step-3.1` |
| `v<major>.<minor>.<patch>` (Semantic Versioning) | Releases | `v0.1.0` = MVP-A complete; `v0.2.0` = MVP-B complete; `v1.0.0` = production go-live; `v0.1.1` = patch |

### 7.4 Never commit

`.env` files, secrets, keys, real customer data, build output (`target/`, `dist/`, `node_modules/`) and OS files such as `.DS_Store`. A `.gitignore` covering these is created in Phase 2.

---

## 8. Traceability in Code and Tests

- Tests that prove a business rule or requirement reference its ID, for example `@Tag("BR-13")` plus `@DisplayName("BR-13: stock never goes negative under concurrent sales")`.
- Commit footers reference requirement and ADR IDs (`Refs:`).
- Error codes and log messages never contain requirement IDs. They are for users and operators, not for documentation.

---

## 9. Related Documents

- [technology-stack.md](technology-stack.md)
- [technology-justification.md](technology-justification.md)
- [design-decisions.md](../02-system-design/design-decisions.md) – ADR-020, ADR-021, ADR-022, ADR-028, ADR-029
- Database design and data dictionary: *Planned – created in step 1.3*
- API documentation: *Planned – created in step 1.5*
