# BakeFlow – Technology Justification

> **Step:** 1.2 · **Status:** Draft for owner review · **Source:** [master-prompt.md](../master-prompt.md) §2, §3.B

## Table of Contents

1. [How to Read This Document](#1-how-to-read-this-document)
2. [Major Choices and Alternatives](#2-major-choices-and-alternatives)
3. [How React Communicates with Spring Boot](#3-how-react-communicates-with-spring-boot)
4. [How Spring Boot Communicates with MySQL](#4-how-spring-boot-communicates-with-mysql)
5. [Where Redis Fits (and Where It Does Not)](#5-where-redis-fits-and-where-it-does-not)
6. [The Full Request Path](#6-the-full-request-path)
7. [Related Documents](#7-related-documents)

---

## 1. How to Read This Document

For each major choice you will find:

- **Chosen** – what we use.
- **Why** – the main reasons.
- **Alternatives** – what else was considered.
- **Trade-offs** – what we give up.
- **Reconsider when** – the signal that would make us revisit the choice.

The full list of technologies and versions is in [technology-stack.md](technology-stack.md). Formal decisions are recorded as ADRs in [design-decisions.md](../02-system-design/design-decisions.md).

---

## 2. Major Choices and Alternatives

### 2.1 MySQL vs PostgreSQL

- **Chosen:** MySQL 8.4 LTS ([ADR-019](../02-system-design/design-decisions.md#adr-019--mysql-84-lts)).
- **Why:** Required by the brief. Very widely used and hosted (AWS RDS, Azure, GCP, all Indian hosting providers). InnoDB gives transactions, row-level locks (`SELECT … FOR UPDATE`, ADR-008), CHECK constraints (enforced since 8.0.16) and good performance for BakeFlow's mostly simple queries.
- **Alternatives:**
  - *PostgreSQL:* richer SQL (partial indexes, `TIMESTAMPTZ`, better JSON, exclusion constraints), stricter defaults. Equally good for BakeFlow.
  - *MariaDB:* MySQL fork, diverging features.
- **Trade-offs:** no true time-zone timestamp type (handled by the UTC convention, ADR-005); no partial indexes (for example "unique only where active"), which need workarounds such as generated columns.
- **Reconsider when:** never for this project, unless hosting forces it.

### 2.2 Flyway vs Liquibase

- **Chosen:** Flyway (ADR-003).
- **Why:** Migrations are plain SQL files: what you write is what runs. Easy to review and to learn real SQL. First-class Spring Boot support.
- **Alternatives:** *Liquibase:* changelogs in XML, YAML or SQL; database-independent changes; built-in rollback; diff tools.
- **Trade-offs:** Flyway's undo (rollback) is a paid feature. We fix mistakes with a new forward migration instead, which is good practice anyway.
- **Reconsider when:** the project must support several database vendors.

### 2.3 Next.js vs Vite SPA

- **Chosen:** A single React SPA built with Vite ([ADR-026](../02-system-design/design-decisions.md#adr-026--single-react-spa-built-with-vite-not-nextjs)).
- **Why:** Most screens are behind a login (POS, inventory, admin), where server-side rendering (SSR) brings no benefit. A static SPA is served by any web server or CDN, with no Node.js server in production. Simpler mental model: React in the browser, Spring Boot for all business logic.
- **Alternatives:**
  - *Next.js:* SSR and static generation (good SEO for public catalog pages), file-based routing, server components. But it adds a Node server and a second backend-like layer, which blurs where business logic lives.
  - *Three separate SPAs* (customer, branch, admin): stronger isolation, but three builds and duplicated shared code. One SPA with lazy-loaded route areas gives most of the isolation benefit.
- **Trade-offs:** public catalog pages are less SEO-friendly (a client-rendered SPA). Acceptable because customers arrive mainly by direct link or by searching the bakery's name.
- **Reconsider when:** organic search traffic for product pages becomes a business goal.

### 2.4 Sessions vs JWT

- **Chosen:** JWT access tokens (short-lived, in memory) plus a refresh token in an HttpOnly cookie ([ADR-023](../02-system-design/design-decisions.md#adr-023--jwt-with-spring-security-resource-server-nimbus-not-jjwt), [ADR-024](../02-system-design/design-decisions.md#adr-024--token-storage-in-the-browser-and-same-origin-deployment)).
- **Why:** The brief specifies JWT. The server stays stateless (NFR-35), so more instances can be added without shared session storage. The same approach works for the SPA and for any future mobile app.
- **Alternatives:**
  - *Server sessions (`JSESSIONID` cookie, Spring Session with Redis or JDBC):* simple, instantly revocable, very secure with HttpOnly cookies. Needs shared session storage for several instances and CSRF protection.
  - *Long-lived JWT in `localStorage`:* simple but readable by any XSS script; rejected.
- **Trade-offs:** an access token cannot be revoked before it expires. This is mitigated by a **15-minute** lifetime (NFR-14) and refresh tokens that are stored server-side (hashed) and can be revoked on logout or deactivation.
- **Reconsider when:** instant revocation of every token becomes a hard requirement.

### 2.5 Redux vs TanStack Query

- **Chosen:** TanStack Query for server state; React context and local state for the little remaining client state ([ADR-027](../02-system-design/design-decisions.md#adr-027--frontend-data-layer-tanstack-query-axios-react-hook-form-zod-no-redux)).
- **Why:** About 90 % of BakeFlow's frontend state is **server data** (products, batches, orders). TanStack Query handles caching, background refresh, retries, loading and error states, and invalidation after changes, with far less code than Redux.
- **Alternatives:**
  - *Redux Toolkit + RTK Query:* powerful, with devtools, but more boilerplate and concepts.
  - *Zustand:* tiny client-state store. Can be added later if client state grows (for example a complex offline POS cart).
- **Trade-offs:** no single global store to inspect; state is spread across query caches and components.
- **Reconsider when:** complex client-only state appears (for example offline POS).

### 2.6 Gradle vs Maven

- **Chosen:** Maven, run through the Maven Wrapper ([ADR-018](../02-system-design/design-decisions.md#adr-018--maven-with-the-maven-wrapper)).
- **Why:** Required by the brief. Declarative XML is verbose but predictable and easy to read for learners. Excellent IDE and CI support. The wrapper pins the Maven version for every developer and for CI.
- **Alternatives:** *Gradle (Kotlin DSL):* faster incremental builds, flexible scripting, build cache.
- **Trade-offs:** slower builds than Gradle on large projects. Irrelevant at BakeFlow's size.
- **Reconsider when:** builds become slow (more than a few minutes) or need complex custom logic.

### 2.7 Spring Boot (and its version line)

- **Chosen:** Spring Boot 4.1.x ([ADR-016](../02-system-design/design-decisions.md#adr-016--spring-boot-41x-on-java-21-lts)). The detailed 4.x vs 3.5 assessment is in [technology-stack.md §3](technology-stack.md#3-spring-boot-4x-vs-35x-assessment).
- **Why Spring Boot at all:** required by the brief; the most common Java backend framework in industry, especially in India; complete ecosystem (security, data, validation, testing, observability).
- **Alternatives:** *Quarkus, Micronaut* (faster startup, lower memory, good for serverless); *Jakarta EE servers*.
- **Reconsider when:** not expected.

### 2.8 Lombok (decided: no)

- **Chosen:** No Lombok ([ADR-021](../02-system-design/design-decisions.md#adr-021--no-lombok-java-records-for-dtos)).
- **Why:** Java 21 **records** replace most Lombok use (request and response DTOs, value objects). JPA entities cannot be records, but their getters are generated once by the IDE and rarely change. Lombok's `@Data`, `@EqualsAndHashCode` and `@ToString` are risky on JPA entities (lazy-loading surprises, infinite recursion, broken `Set` behaviour). Lombok hooks into compiler internals and has had compatibility problems with new JDK and Spring Boot releases, including reports with Spring Boot 4. Explicit code is also clearer for learning.
- **Alternatives:** *Lombok* (less boilerplate; very common in industry projects).
- **Trade-offs:** some extra lines in entity classes.
- **Reconsider when:** entity boilerplate becomes a real burden (unlikely).

### 2.9 MapStruct

- **Chosen:** MapStruct for entity ↔ DTO mapping ([ADR-022](../02-system-design/design-decisions.md#adr-022--mapstruct-for-dto-mapping)).
- **Why:** Generates plain Java at compile time, so it is fast, debuggable and has no reflection. Reports unmapped fields as compile errors. Keeps entities out of API responses (coding convention).
- **Alternatives:** *Manual mapping methods* (zero dependencies, but repetitive and easy to forget a field); *ModelMapper* (reflection, runtime errors, slower).
- **Trade-offs:** one more annotation processor; complex mappings still need hand-written methods.
- **Reconsider when:** the Phase 2 spike shows a blocking Spring Boot 4 incompatibility. Fallback: manual mappers.

### 2.10 Spring Modulith and ArchUnit

- **Chosen:** Both ([ADR-020](../02-system-design/design-decisions.md#adr-020--package-by-module-structure-enforced-by-spring-modulith-and-archunit)).
- **Why:** A modular monolith only stays modular if boundaries are **checked automatically** (NFR-42).
  - Spring Modulith verifies that modules only use each other's public API and that there are no cycles. It generates module documentation and supports module-level tests and application events between modules.
  - ArchUnit adds layer rules **inside** a module (for example controllers must not call repositories; domain must not depend on web classes).
- **Alternatives:** *ArchUnit only* (more manual rules); *Maven multi-module* (hard boundaries through separate jars, but heavier builds and more ceremony for one developer); *discipline only* (erodes).
- **Trade-offs:** a few rules to learn; Spring Modulith's package conventions must be followed.
- **Reconsider when:** modules are extracted into separate services.

### 2.11 Containers and CI

- **Chosen:** Docker + Docker Compose locally, GitHub Actions for CI.
- **Why:** Identical MySQL, Redis and Mailpit versions on every machine and in CI. Testcontainers starts real databases in tests. GitHub Actions is free for public repositories, integrated with pull requests, and its Ubuntu runners have Docker installed, so Testcontainers works there without extra setup.
- **Alternatives:** *locally installed MySQL* (version drift, conflicts); *GitLab CI, Jenkins*.
- **Reconsider when:** the code moves away from GitHub.

---

## 3. How React Communicates with Spring Boot

### 3.1 The basics in plain words

- The **React app** runs in the user's browser. It shows screens and collects input, but it holds **no business rules that matter for correctness**. For example, it may display a calculated total, but the server always recalculates prices, tax and stock.
- The **Spring Boot backend** exposes a **REST API**: a set of URLs (endpoints) such as `/api/v1/branches/3/pos/bills`. Each combines an HTTP **method** (GET = read, POST = create, PUT/PATCH = change, DELETE = remove) with a **resource** (a noun).
- They exchange **JSON** (a text format for data) over **HTTPS** (encrypted HTTP).
- Every protected request carries a **JWT access token** in the `Authorization` header, so the server knows who is calling.

### 3.2 Same origin, so CORS is (almost) not needed

**CORS** (Cross-Origin Resource Sharing) is a browser rule: a page loaded from one origin (scheme + host + port) may only call another origin if that server explicitly allows it.

BakeFlow avoids cross-origin calls ([ADR-024](../02-system-design/design-decisions.md#adr-024--token-storage-in-the-browser-and-same-origin-deployment)):

| Environment | How the browser reaches the API | CORS needed? |
|-------------|---------------------------------|--------------|
| Development | The browser loads the app from Vite at `http://localhost:5173`. Vite's **dev proxy** forwards `/api/*` to Spring Boot at `http://localhost:8080`. The browser only ever talks to port 5173. | No |
| Production | A reverse proxy (Nginx or Caddy) at `https://app.example.in` serves the built React files and forwards `/api/*` to Spring Boot. | No |

Spring Security still has a **strict CORS configuration** (an allow-list of origins, empty by default), so that any accidental cross-origin call is rejected rather than silently allowed.

### 3.3 Authentication flow (simplified)

```mermaid
sequenceDiagram
    participant B as Browser (React)
    participant S as Spring Boot
    B->>S: POST /api/v1/auth/staff/login {username, password}
    S-->>B: 200 {accessToken, expiresIn} + Set-Cookie: refresh token (HttpOnly)
    Note over B: access token kept in memory only
    B->>S: GET /api/v1/branches/3/inventory/batches<br/>Authorization: Bearer <accessToken>
    S-->>B: 200 JSON
    Note over B,S: 15 minutes later the access token expires
    B->>S: GET ... (expired token)
    S-->>B: 401 Unauthorized
    B->>S: POST /api/v1/auth/refresh (cookie sent automatically)
    S-->>B: 200 {new accessToken} + rotated refresh cookie
    B->>S: retry the original GET with the new token
```

The Axios **interceptor** in the React app performs the refresh-and-retry automatically, so screens do not need to care. Endpoint names here are illustrative; the real API is defined in step 1.5.

### 3.4 Standard error format

Every error uses the same JSON shape, based on **RFC 9457 "Problem Details for HTTP APIs"**, which Spring supports natively ([ADR-028](../02-system-design/design-decisions.md#adr-028--rest-api-conventions)):

```json
{
  "type": "https://bakeflow.app/problems/insufficient-stock",
  "title": "Insufficient stock",
  "status": 409,
  "detail": "Only 3 units of Chocolate Pastry are available at Anna Nagar.",
  "instance": "/api/v1/branches/1/pos/bills",
  "code": "INVENTORY_INSUFFICIENT_STOCK",
  "traceId": "8f2c1a7e4b9d4c21",
  "errors": [
    { "field": "lines[0].quantity", "message": "Requested 5, available 3" }
  ]
}
```

- `status` – the HTTP status code (400 validation, 401 not logged in, 403 not allowed, 404 not found, 409 conflict, 422 business rule violation, 500 server error).
- `code` – a stable, machine-readable error code the frontend can switch on.
- `traceId` – the correlation ID, also written to server logs, to find the exact log lines (NFR-45).
- `errors` – field-level problems for forms.

### 3.5 A full request example: creating a POS bill

The cashier at Anna Nagar (branch 1) bills two chocolate pastries and pays with UPI.

**1. React** (POS screen) calls the API layer:

```http
POST /api/v1/branches/1/pos/bills HTTP/1.1
Host: app.example.in
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Idempotency-Key: 6f1d2c3e-8a4b-4c5d-9e6f-7a8b9c0d1e2f
Content-Type: application/json
X-Correlation-Id: 8f2c1a7e4b9d4c21

{
  "lines": [
    { "variantId": 1012, "quantity": 2 }
  ],
  "payments": [
    { "method": "UPI", "amount": "180.00", "reference": "UPI-REF-99812" }
  ]
}
```

**2. Spring Boot**, step by step:

1. **Security filter chain:** validates the JWT signature and expiry and builds the current user (Priya, BRANCH_STAFF, branches [1]).
2. **Idempotency filter:** looks up the key; first time seen, so it continues (ADR-011).
3. **Controller** (`PosBillController`): Jackson converts the JSON into a `CreateBillRequest` record; Bean Validation checks it (quantity > 0 and so on).
4. **Authorization check:** is the user allowed to bill in branch 1? Yes (BR-14). If the URL said branch 2, the result would be `403`.
5. **Service** (`PosBillingService`), inside **one database transaction**:
   - asks the inventory module to **allocate 2 units by FEFO**, which locks batch rows (`SELECT … FOR UPDATE`, ADR-008) and checks expiry (ADR-013);
   - gets the price and HSN tax rate in force today (ADR-006, ADR-014);
   - reserves the next invoice number `ANN/2627/000124` (ADR-010);
   - saves the invoice, lines, payment, stock movements (ledger, ADR-015) and audit entry;
   - commits. If any step fails, **everything is rolled back**.
6. **Response:** MapStruct maps the saved invoice to a `BillResponse` record (never the entity itself).

**3. Response** to React:

```http
HTTP/1.1 201 Created
Location: /api/v1/branches/1/pos/bills/5531
Content-Type: application/json

{
  "id": 5531,
  "invoiceNumber": "ANN/2627/000124",
  "issuedAt": "2026-11-24T11:42:07.512Z",
  "lines": [
    {
      "variantId": 1012,
      "productName": "Chocolate Pastry",
      "batchNumber": "A001",
      "quantity": 2,
      "unitPrice": "90.00",
      "discountAmount": "0.00",
      "lineTotal": "180.00",
      "gstRate": "18.00",
      "taxableValue": "152.54",
      "cgstAmount": "13.73",
      "sgstAmount": "13.73"
    }
  ],
  "totalAmount": "180.00",
  "payments": [ { "method": "UPI", "amount": "180.00" } ]
}
```

Notes:

- `issuedAt` is in **UTC** (`Z`). React displays it as `24-11-2026 05:12 PM` IST (ADR-005, NFR-48, NFR-49).
- Money is sent as **strings** with two decimals, so no precision is lost in JavaScript (ADR-028).
- The GST rate (18 %) is only an example of a rate configured by the admin, not tax advice (AS-05).
- **4. React:** TanStack Query marks the "today's bills" and "stock" queries as stale, so they refresh, and the print view opens (ADR-032).
- If the cashier's network drops and the request is **retried with the same `Idempotency-Key`**, the server returns the same `201` response without creating a second bill (NFR-11).

---

## 4. How Spring Boot Communicates with MySQL

### 4.1 The layers, from the bottom up

| Layer | What it is | Plain-language role |
|-------|-----------|---------------------|
| **MySQL server** | The database process (in Docker locally) | Stores the tables and enforces constraints and locks |
| **JDBC driver** (MySQL Connector/J) | Java library implementing the JDBC standard | Speaks MySQL's network protocol; sends SQL, receives rows |
| **HikariCP** | Connection pool | Opening a database connection is slow, so Hikari keeps a small set (for example 10) open and lends them to requests |
| **JPA / Hibernate** | Object-relational mapper (ORM) | Turns Java objects (entities) into SQL `INSERT/UPDATE/SELECT` and rows back into objects; tracks changes; handles locking |
| **Spring Data JPA** | Repository abstraction | You declare an interface (`InventoryBatchRepository`); Spring generates the implementation, including queries derived from method names |
| **Spring transactions** (`@Transactional`) | Transaction management | Starts a database transaction when a service method begins and commits or rolls back when it ends |
| **Flyway** | Migration tool | At startup, applies any new SQL migration files so the schema matches the code |

### 4.2 Startup sequence

1. Spring Boot reads the datasource settings (URL, user, password from **environment variables**, never hard-coded).
2. HikariCP opens its initial connections.
3. **Flyway** checks its history table (`flyway_schema_history`) and runs any new `V…__*.sql` files in order (ADR-003, [ADR-029](../02-system-design/design-decisions.md#adr-029--database-naming-conventions-and-flyway-versioning)).
4. **Hibernate validates** that every entity matches the schema (`ddl-auto=validate`). A mismatch stops startup, which is intentional.
5. The application starts accepting HTTP requests.

### 4.3 What happens in one transactional request

```mermaid
sequenceDiagram
    participant Svc as Service (@Transactional)
    participant Repo as Repository
    participant H as Hibernate
    participant Pool as HikariCP
    participant DB as MySQL
    Svc->>Pool: begin – borrow a connection, START TRANSACTION
    Svc->>Repo: findSellableBatchesForUpdate(branch, variant)
    Repo->>H: JPQL query with PESSIMISTIC_WRITE lock
    H->>DB: SELECT ... ORDER BY expires_at, id FOR UPDATE
    DB-->>H: rows (now locked for this transaction)
    Svc->>Svc: FEFO allocation in Java, update quantities
    Svc->>Repo: save(stockMovement), save(invoice)
    Note over H: changes kept in the persistence context
    Svc->>H: method ends – flush
    H->>DB: INSERT/UPDATE statements
    Svc->>DB: COMMIT (locks released)
    Svc->>Pool: return connection
```

**Key rules**

- **Transactions belong in the service layer**, never in controllers (detailed in step 1.6).
- **Keep transactions short:** no calls to Razorpay or email while holding locks (NFR-28).
- **Read-only methods** use `@Transactional(readOnly = true)` for efficiency.
- **Open Session in View is disabled** (`spring.jpa.open-in-view=false`), so no lazy-loading happens in controllers. Each service returns exactly the data needed.
- **JDBC time zone is UTC** (`hibernate.jdbc.time_zone=UTC`) so timestamps are stored in UTC (ADR-005).
- **Parameterised queries only.** JPA binds parameters, which prevents SQL injection (NFR-16).

### 4.4 Locking in one sentence each

- **Pessimistic lock** (`SELECT … FOR UPDATE`): "I am changing these batch rows; everyone else waits until I commit." Used for stock quantities.
- **Optimistic lock** (`@Version`): "Save my change only if nobody changed this row since I read it; otherwise fail and retry." Used for other updates (ADR-008).

---

## 5. Where Redis Fits (and Where It Does Not)

Redis is an in-memory key-value store: extremely fast, but its data is not the official record. It arrives in **Phase 4** ([ADR-030](../02-system-design/design-decisions.md#adr-030--redis-for-caching-only-introduced-in-phase-4)).

### 5.1 Planned uses

| Use | Why Redis helps | Safe because… |
|-----|-----------------|---------------|
| Catalog cache (categories, products, prices per branch) | Read very often, changed rarely | Short TTL plus eviction on change; a stale price is never used for billing, because billing reads prices from MySQL |
| Availability summary for customer browsing ("in stock / only 3 left") | Many customers browse at once (NFR-03) | Display only; **checkout always re-checks and reserves in MySQL** |
| Idempotency key look-up (optional) | Fast duplicate detection | The MySQL record stays authoritative |
| Login throttling counters (optional) | Fast counters with expiry | Losing them only resets a counter |

### 5.2 What Redis will NOT be used for

- ❌ **Not the source of truth for stock.** Batch quantities, reservations and the ledger live **only in MySQL**, protected by transactions and row locks. A cache can be stale or lost; stock must never be.
- ❌ **Not for locking stock** (no Redis distributed locks for allocation; ADR-008 uses database locks).
- ❌ **Not for invoices, payments, orders or audit data.**
- ❌ **Not for sessions** (the API is stateless; refresh tokens are stored in MySQL).
- ❌ **Not required to run.** If Redis is down, BakeFlow falls back to MySQL (NFR-12).

**Licence note:** Redis 8 is available under AGPLv3 (among other licences). Using it unmodified as an internal cache is fine. **Valkey** (BSD licence, Linux Foundation) is a drop-in alternative if licensing ever becomes a concern.

---

## 6. The Full Request Path

```mermaid
flowchart LR
    U([User]) --> BR[Browser]
    subgraph FE["React SPA (in the browser)"]
        PG["Page / component"] --> HK["TanStack Query hook"]
        HK --> AX["Axios client<br/>(adds JWT, correlation ID,<br/>refresh on 401)"]
    end
    BR --> PG
    AX -- "HTTPS JSON<br/>/api/v1/..." --> PX["Vite proxy (dev) /<br/>Nginx (prod)"]
    subgraph BE["Spring Boot application"]
        direction LR
        F1["Filters:<br/>correlation ID → security (JWT) →<br/>idempotency"] --> C["Controller<br/>(validation, DTO)"]
        C --> AZ["Authorization<br/>(role + branch scope)"]
        AZ --> S["Service<br/>(@Transactional,<br/>business rules)"]
        S --> R["Repository<br/>(Spring Data JPA)"]
        R --> HB["Hibernate +<br/>HikariCP + JDBC"]
        EH["Global exception handler<br/>→ Problem Details JSON"]
    end
    PX --> F1
    HB -- "SQL" --> DB[(MySQL 8.4)]
    S -. "Phase 4 cache" .-> RD[(Redis)]
    S -. "errors" .-> EH
```

**Explanation:** the user clicks a button. The page calls a TanStack Query hook, which uses the Axios client to send an HTTPS request with the JWT. The proxy forwards `/api` to Spring Boot. Filters add a correlation ID, authenticate the JWT and check idempotency. The controller validates input and converts JSON into a request object. The authorization step checks role and branch scope. The service runs business rules in a transaction and uses repositories, which use Hibernate and JDBC to talk to MySQL. The response travels back as JSON. Any exception becomes a Problem Details error response.

---

## 7. Related Documents

- [technology-stack.md](technology-stack.md)
- [coding-conventions.md](coding-conventions.md)
- [development-environment.md](development-environment.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
- Detailed architecture (HLD/LLD): *Planned – created in step 1.6*
- Security design: *Planned – created in step 1.7*
