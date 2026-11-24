# BakeFlow – Non-Functional Requirements

> **Step:** 1.1 (owner-reviewed) · **Source:** [master-prompt.md](../master-prompt.md) §2, §4, Appendix C

## Table of Contents

1. [Purpose and Conventions](#1-purpose-and-conventions)
2. [Performance](#2-performance)
3. [Availability and Reliability](#3-availability-and-reliability)
4. [Security and Privacy](#4-security-and-privacy)
5. [Data Integrity](#5-data-integrity)
6. [Concurrency](#6-concurrency)
7. [Auditability](#7-auditability)
8. [Scalability and Capacity](#8-scalability-and-capacity)
9. [Usability](#9-usability)
10. [Maintainability and Observability](#10-maintainability-and-observability)
11. [Localisation](#11-localisation)
12. [Related Documents](#12-related-documents)

---

## 1. Purpose and Conventions

Non-functional requirements (NFRs) describe **how well** the system must work, rather than what it does. Every NFR has a measurable target so it can be tested in Phase 5.

- **p95** means 95 % of requests are at least this fast.
- Latency targets are measured **at the server** (from request received to response sent), under the capacity defined in NFR-32, unless stated otherwise.
- **Priority** uses the same Must/Should/Could scale as [functional-requirements.md](functional-requirements.md).
- **Verified in** shows when the NFR is checked: the module (3.x) or phase (4, 5).

---

## 2. Performance

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-01 | POS bill creation is fast | Finalising a POS bill of up to 10 lines (FEFO allocation, invoice number, GST calculation, save): **p95 < 500 ms**, p99 < 1 s | Must | 3.8, 5 |
| NFR-02 | Product search on POS is instant | Product search by name or code: **p95 < 200 ms** | Must | 3.3, 5 |
| NFR-03 | Catalog browsing is fast | Catalog and availability API for one branch: **p95 < 300 ms**. Customer page on a 4G phone: **first contentful paint < 2.5 s** | Must | 3.9, 4, 5 |
| NFR-04 | Checkout reservation is fast | Checkout reservation of up to 10 lines: **p95 < 800 ms** | Must | 3.10, 5 |
| NFR-05 | General API latency | All other read APIs **p95 < 300 ms**; write APIs **p95 < 600 ms** | Should | 5 |
| NFR-06 | Reports do not block operations | Daily reports for one branch for up to 31 days: **< 3 s**. Report queries must not hold locks on inventory tables (pre-aggregated summaries or read-only queries) | Should | 3.16, 4 |
| NFR-07 | Background jobs are timely | Expiry status job runs **every 5 minutes** (AS-28). Expired reservations released **within 1 minute** of timeout | Must | 3.6, 3.10 |

---

## 3. Availability and Reliability

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-08 | Availability during business hours | **99.5 %** monthly availability between 06:00 and 23:00 IST (about 2.6 hours of downtime per month at most). Planned maintenance outside these hours | Should | 5 |
| NFR-09 | Backups | Full database backup **daily** plus binary logs. **RPO ≤ 15 minutes**, **RTO ≤ 4 hours**. Restore tested **at least monthly** | Must | 5 |
| NFR-10 | Payment resilience | No paid order is lost if the browser closes after payment: server-side **webhook reconciliation** confirms or refunds **100 %** of captured payments within 15 minutes | Must | 3.10 |
| NFR-11 | Safe retries | Repeating a billing or payment request with the same `Idempotency-Key` creates **no duplicate** invoice, payment or stock movement (ADR-011) | Must | 3.8, 3.10 |
| NFR-12 | Graceful degradation | If Redis is unavailable (Phase 4 onward), the system **keeps working** from the database, with only slower responses | Should | 4 |

---

## 4. Security and Privacy

Detailed design is in step 1.7.

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-13 | Password storage | BCrypt with cost factor **≥ 12**. Plain-text passwords are never stored or logged | Must | 3.1 |
| NFR-14 | Token-based authentication | JWT access token lifetime **≤ 15 min**. Refresh token **≤ 7 days**, rotated on use and revocable on logout or deactivation | Must | 3.1 |
| NFR-15 | Branch and organization isolation | **100 %** of branch-scoped endpoints enforce server-side scope checks. Automated authorization tests show that changing `branchId` or `organizationId` in a request returns **403 or 404**, never data (BR-14) | Must | 3.1, 3.2, every module |
| NFR-16 | OWASP Top 10 | **No high or critical findings** from dependency scanning and a manual OWASP Top 10 review before go-live. Parameterised queries only (no string-built SQL) | Must | 5 |
| NFR-17 | Transport security | HTTPS only (TLS 1.2+) in every non-local environment. HSTS enabled | Must | 5 |
| NFR-18 | Payment data | BakeFlow **never stores card numbers, CVV or UPI PINs**. Razorpay handles card data, keeping BakeFlow out of PCI-DSS card-data scope. Webhook signatures are verified on **100 %** of callbacks | Must | 3.10 |
| NFR-19 | Brute-force protection | Login is locked or throttled after **5 failed attempts in 15 minutes** per account | Should | 3.1 |
| NFR-20 | Personal data protection | Customer personal data (name, phone, email, addresses) is visible only to its owner and to authorised staff of that organization. It is masked in logs. Accounts can be deleted or anonymised on request, keeping invoices as the law requires (aligned with India's DPDP Act 2023) | Should | 3.9, 5 |

---

## 5. Data Integrity

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-21 | No negative stock | **0** batches with negative on-hand, reserved or available quantity, enforced by application checks **and** database CHECK constraints (BR-13) | Must | 3.4, 3.5 |
| NFR-22 | Ledger matches batches | For every batch, the sum of ledger movements equals the stored quantities. A reconciliation query finds **0 mismatches**, checked by an automated test and a nightly check (ADR-015) | Must | 3.4, 4 |
| NFR-23 | Money precision | All money stored as `DECIMAL(12,2)` and handled as `BigDecimal` with explicit rounding (HALF_UP to 2 decimals at line level). **0** `float`/`double` money fields (ADR-004) | Must | All |
| NFR-24 | Invoice continuity | Invoice numbers are **gap-free and unique** per branch per financial year, even under concurrent billing. Invoices are **never deleted or edited**; corrections go through credit notes (ADR-010) | Must | 3.8 |
| NFR-25 | Schema control | **100 %** of schema changes go through versioned Flyway migrations. Hibernate `ddl-auto=validate` in every environment (ADR-003) | Must | 2 |

---

## 6. Concurrency

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-26 | No overselling | A Testcontainers test runs **50 concurrent** allocation requests for a batch with 10 units: exactly **10 units** are allocated in total, **0** oversold, and the ledger is consistent (ADR-008) | Must | 3.5 |
| NFR-27 | No deadlocks under normal load | Batch rows are always locked in a **consistent order** (by batch ID). A load test with 20 concurrent POS terminals and online checkouts on the same products shows **0 unrecovered deadlocks**. Transient lock failures are retried **up to 3 times** | Must | 3.5, 5 |
| NFR-28 | Short transactions | Inventory transactions hold row locks for **< 200 ms (p95)**. External calls (for example Razorpay) are **never** made while holding a stock lock | Must | 3.5, 3.10 |

---

## 7. Auditability

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-29 | Audit coverage | **100 %** of the audited actions in BR-15 produce an audit record with: who (user ID and role), when (UTC), where (organization, branch), what (action, entity type and ID), and before/after values for changes | Must | Phase 2 (framework, actor `system`), 3.1 (real actor), 3.15 (viewing) |
| NFR-30 | Tamper resistance | Audit log and stock ledger are **append-only**: the application database user has no UPDATE or DELETE rights on these tables in production | Should | 5 |
| NFR-31 | Retention | Invoices, credit notes, payments, audit log and stock ledger are kept for **at least 8 years** (covers the GST record-keeping requirement of 72 months from the annual return due date). Notifications are kept for 90 days | Must | 5 |

---

## 8. Scalability and Capacity

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-32 | MVP capacity | Without degradation beyond the targets above: **1 organization, 10 branches, 50 staff users, 1,000 active products, 2,000 POS bills/day, 500 online orders/day, 30 POS terminals and 300 concurrent customer sessions at peak** (AS-30) | Must | 5 |
| NFR-33 | Growth headroom | Supports **50 branches** and **5× the MVP transaction volume** with no schema redesign, by scaling hardware, adding Redis caching and adding a read replica | Should | 4 |
| NFR-34 | Multi-tenant ready | Every business table carries `organization_id`, so further organizations can be added without migration (AS-02) | Must | 3.2 onward |
| NFR-35 | Stateless application | The backend keeps no user session in memory (JWT), so **more than one instance** can run behind a load balancer | Should | 4 |

---

## 9. Usability

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-36 | POS speed for staff | A 3-item cash bill can be completed in **≤ 30 seconds** by trained staff. Fully **keyboard-operable** (search, quantity, pay, print) and works with USB barcode scanners (keyboard input) | Must | 3.8 |
| NFR-37 | Mobile-friendly customer site | Usable from **360 px** screen width. Tap targets ≥ 44 px. Checkout in **≤ 4 screens** from cart | Must | 3.9, 3.10 |
| NFR-38 | Clear errors | Every API error returns a consistent error body (code, message, field errors). Users never see stack traces. Stock errors name the product and the quantity actually available | Must | 2, all |
| NFR-39 | Accessibility | Customer site meets **WCAG 2.1 AA** for colour contrast, labels and keyboard navigation | Should | 3.9, 5 |
| NFR-40 | Printable documents | Invoices print on **80 mm thermal** and **A4** layouts from the browser | Should | 3.8 |

---

## 10. Maintainability and Observability

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-41 | Test coverage | Service layer line coverage **≥ 70 %** overall and **≥ 90 %** for inventory allocation, billing, refunds and discount calculation | Must | Every module |
| NFR-42 | Module boundaries | Modules talk to each other only through public service interfaces. **0** violations, checked by automated architecture tests (tool decided in step 1.2) (ADR-001) | Should | 2 onward |
| NFR-43 | API documentation | **100 %** of endpoints documented in OpenAPI (Swagger) and kept in sync with code | Must | Every module |
| NFR-44 | CI quality gate | Every push runs build, unit tests and integration tests in CI. Merging to `main` requires a **green pipeline** | Must | 2 |
| NFR-45 | Structured logging | JSON logs with a **correlation ID** per request, user ID and branch ID. No secrets or personal data in logs | Should | 2, 5 |
| NFR-46 | Health and metrics | Health, readiness and basic metrics endpoints (request latency, error rate, DB pool) are available to monitoring | Should | 5 |

---

## 11. Localisation

| ID | Requirement | Measurable target | Priority | Verified in |
|----|-------------|-------------------|----------|-------------|
| NFR-47 | Currency | Single currency **INR**, shown as `₹1,23,456.00` with Indian digit grouping (AS-01) | Must | All UIs |
| NFR-48 | Time zone | All timestamps **stored in UTC** and **displayed in Asia/Kolkata (IST)**. "Business day", daily reports and financial year (1 Apr–31 Mar) are based on the **IST calendar** (ADR-005, AS-21) | Must | All |
| NFR-49 | Date and number format | Dates displayed as `DD-MM-YYYY` and times as 12-hour `hh:mm AM/PM` in IST | Should | All UIs |
| NFR-50 | Language | UI in **English** only. Text is kept in resource files so other languages (for example Tamil) can be added later without code changes | Should | All UIs |

---

## 12. Related Documents

- [functional-requirements.md](functional-requirements.md)
- [business-requirements.md](business-requirements.md)
- [design-decisions.md](../02-system-design/design-decisions.md)
- [scope-and-assumptions.md](scope-and-assumptions.md)
- Security details: *Planned – created in step 1.7*
- Testing strategy: *Planned – created in step 1.7*
