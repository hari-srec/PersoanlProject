# BakeFlow – Implementation Status

> **Last updated:** 24-Nov-2026 (step 1.2) · **Owner:** project owner marks steps **Done** after review

## Table of Contents

1. [How to Use This File](#1-how-to-use-this-file)
2. [Summary](#2-summary)
3. [Phase 1 – Documentation](#3-phase-1--documentation)
4. [Phase 2 – Tech Stack & Project Setup](#4-phase-2--tech-stack--project-setup)
5. [Phase 3 – Module Implementation](#5-phase-3--module-implementation)
6. [Phase 4 and Phase 5](#6-phase-4-and-phase-5)
7. [Change Log](#7-change-log)

---

## 1. How to Use This File

- **Status values:** `Not started` · `In progress` · `Done` (Done only after owner review).
- **MVP stage:** `MVP-A` (branch operations), `MVP-B` (online ordering), `No` (post-MVP). See [feature-roadmap.md §4](../00-project-overview/feature-roadmap.md#4-mvp-definition).
- **Dependencies** refer to roadmap IDs ([feature-roadmap.md](../00-project-overview/feature-roadmap.md)).
- **Pending decisions** refer to open questions (OQ-xx) in [scope-and-assumptions.md](../00-project-overview/scope-and-assumptions.md#5-open-questions-oq-xx). OQ-01 to OQ-22 are resolved.
- Phase 1 is marked **Done** only after all documentation steps (1.1–1.8) are created and reviewed (master-prompt §6).
- Technical debt is tracked separately in [technical-debt.md](technical-debt.md).

---

## 2. Summary

| Phase | Status | Progress |
|-------|--------|----------|
| Phase 1 – Documentation | In progress | 1 / 8 steps done (1.1); 1.2 in progress |
| Phase 2 – Tech stack & project setup | Not started | – |
| Phase 3 – Module implementation | Not started | 0 / 18 modules |
| Phase 4 – Advanced engineering | Not started | – |
| Phase 5 – Hardening & deployment | Not started | – |

---

## 3. Phase 1 – Documentation

| ID | Name | Status | Dependencies | Deliverables | Pending decisions |
|----|------|--------|--------------|--------------|-------------------|
| 1.1 | Project overview, requirements & key decisions | **Done** (owner-reviewed 24-Nov-2026) | master-prompt.md | `00-project-overview/*` (7 docs), `02-system-design/design-decisions.md` (ADR-001 – 015), `09-phase-tracking/*`, `documentation/README.md` | None (OQ-01 – 22 resolved) |
| 1.2 | Technology stack documentation | **In progress** | 1.1 | `01-technology-stack/`: `technology-stack.md`, `technology-justification.md`, `development-environment.md`, `coding-conventions.md`; ADR-016 – 034 (Proposed) | OQ-23, OQ-24, OQ-25, OQ-26 |
| 1.3 | Database design & data dictionary | Not started | 1.1, 1.2 | `03-database-design/*`: ER diagram, schema, keys, indexes, normalisation, `data-dictionary.md`, sample data, SQL queries | OQ-23 (adjustment increases) |
| 1.4 | Business workflows | Not started | 1.3 | `05-business-workflows/*`: 16 workflows with sequence diagrams and examples | – |
| 1.5 | API documentation | Not started | 1.3, 1.4 | `04-api-documentation/*`: endpoints per module with JSON examples, errors, roles | – |
| 1.6 | System design (HLD/LLD) & diagrams | Not started | 1.3–1.5 | `02-system-design/*`: HLD, LLD, 9 required Mermaid diagrams | – |
| 1.7 | Security, testing & deployment documentation | Not started | 1.5, 1.6 | `06-security/*` (incl. permission matrix), `07-testing/*`, `08-deployment/*` | Hosting platform, production SMTP provider |
| 1.8 | Consistency review & completion report | Not started | 1.1–1.7 | Cross-document review, Phase 1 completion report | All remaining OQs |

---

## 4. Phase 2 – Tech Stack & Project Setup

| ID | Name | MVP | Status | Dependencies | Deliverables | Pending decisions |
|----|------|-----|--------|--------------|--------------|-------------------|
| 2 | Skeleton, Docker, Flyway, CI | MVP-A | Not started | Phase 1 Done | Backend and frontend skeletons (package-by-module, ADR-020), Maven Wrapper, Docker Compose (MySQL 8.4, Mailpit), Flyway baseline, GitHub Actions CI, error handling (Problem Details), correlation-ID logging, OpenAPI, Testcontainers base test, Spring Modulith/ArchUnit tests. **Audit framework: `audit_log` table and capture mechanism, actor recorded as `system`.** Idempotency store, money and time utilities. **Compatibility spike:** MapStruct, ShedLock, springdoc on Spring Boot 4.1; exact version pins recorded | OQ-24, OQ-25, OQ-26 |

---

## 5. Phase 3 – Module Implementation

Rows are in **build order**: MVP-A first, then MVP-B, then post-MVP modules.

| ID | Name | MVP | Status | Dependencies | Deliverables (FRs) | Pending decisions |
|----|------|-----|--------|--------------|--------------------|-------------------|
| 3.1 | Authentication & authorization | MVP-A | Not started | 2 | FR-ADM-13; auth foundation for FR-CUS-01/02. **Wires the real authenticated user as the audit actor** (replacing `system` for user actions) | – |
| 3.2 | Organization, branches & employees | MVP-A | Not started | 3.1 | FR-ADM-01, 02, 03, 16; FR-BR-38 | – |
| 3.3 | Product catalog & pricing | MVP-A (partial) | Not started | 3.2 | FR-ADM-04–08; FR-BR-01–04 (FR-ADM-07, FR-BR-03, 04 post-MVP) | – |
| 3.4 | Inventory: batches & stock movement ledger | MVP-A | Not started | 3.3 | FR-BR-05, 06, 07 | – |
| 3.5 | Inventory: FEFO allocation & concurrency | MVP-A | Not started | 3.4 | FR-BR-08, 09, 10 | – |
| 3.6 | Inventory: adjustments, expiry job & alerts | MVP-A | Not started | 3.5 | FR-BR-11, 12, 13, 14, 34 | OQ-23 |
| 3.7 | Production management | MVP-A (partial) | Not started | 3.4 | FR-BR-15 (MVP-A), FR-BR-16 | – |
| 3.8 | POS & billing (invoices, GST, credit notes) | MVP-A | Not started | 3.3, 3.5 | FR-BR-20, 21, 23, 24, 25 | – |
| 3.12 | Discounts & near-expiry offers | MVP-A (POS) + MVP-B (online) | Not started | 3.6, 3.8 (MVP-A part); 3.9, 3.10 (MVP-B part) | FR-ADM-14; FR-BR-22, 27, 28 (MVP-A); FR-CUS-09 (MVP-B); FR-BR-26 (post-MVP) | – |
| 3.13 | Wastage management | MVP-A | Not started | 3.6 | FR-BR-29, 30; FR-ADM-15 | – |
| 3.16 | Analytics & reports | MVP-A (basic) | Not started | 3.8, 3.13 | FR-BR-17, 31; FR-ADM-10, 12 (MVP-A); FR-BR-18, 32; FR-ADM-09, 11, 17 (post-MVP) | – |
| 3.9 | Customer accounts & catalog browsing | MVP-B (partial) | Not started | 3.1, 3.3, 3.4 | FR-CUS-01–08, 20 (MVP-B); FR-CUS-21; FR-ADM-22 (post-MVP) | – |
| 3.10 | Cart, checkout, reservation & payment | MVP-B (partial) | Not started | 3.5, 3.9 | FR-CUS-10, 11, 12, 14 (MVP-B); FR-CUS-13 (post-MVP) | – |
| 3.11 | Order management & tracking | MVP-B (partial) | Not started | 3.8, 3.10 | FR-CUS-15, 16, 17; FR-BR-35 (MVP-B); FR-CUS-25, FR-BR-36 (post-MVP) | – |
| 3.14 | Inter-branch stock transfer | No | Not started | 3.5, 3.13 | FR-BR-37 | – |
| 3.15 | Notifications & audit trail (viewing) | No | Not started | 3.2 | FR-CUS-18; FR-BR-33, 39; FR-ADM-20 (audit capture already exists from Phase 2 / 3.1) | – |
| 3.17 | Demand forecasting & production recommendation | No | Not started | 3.7, 3.16 | FR-BR-19; FR-ADM-18 | – |
| 3.18 | Loyalty & reviews | No | Not started | 3.11 | FR-CUS-19, 22, 23, 24; FR-ADM-19, 21 | – |

---

## 6. Phase 4 and Phase 5

| ID | Name | Status | Dependencies | Deliverables | Pending decisions |
|----|------|--------|--------------|--------------|-------------------|
| 4 | Advanced engineering (Redis caching, async events, performance) | Not started | MVP modules | Redis 8 caching (ADR-030), domain events, query tuning, ledger reconciliation job | Cache invalidation rules |
| 5 | Hardening & deployment | Not started | MVP modules (Phase 4 recommended) | OWASP review, load and concurrency tests, backups and restore drill, monitoring, production deployment, Razorpay live mode; Spring Boot 4.2.x upgrade (OQ-25) | Hosting choice (step 1.7) |

---

## 7. Change Log

| Date | Step | Change |
|------|------|--------|
| 24-Nov-2026 | 1.1 | File created. 1.1 marked In progress, pending owner review. |
| 24-Nov-2026 | 1.1 | **Owner review applied; 1.1 marked Done.** Changes: (1) OQ-01 – OQ-22 marked Resolved with final answers and moved into documents as assumptions AS-48 – AS-58 or updates to existing AS items, FRs and ADRs. (2) OQ-01: GST rates stored per HSN code with effective dates (ADR-006, AS-05, FR-ADM-06). (3) OQ-02: ADR-010 format changed to `ANN/2627/000123`, credit-note series `ANNC/2627/00012`, with the 16-character GST reason. (4) OQ-04: no-show refund policy made a configurable organization setting (AS-32, FR-ADM-16). (5) OQ-13 changed: decreases need approval only above 5 units / ₹500, no self-approval, all adjustments audited (AS-54, FR-BR-11); OQ-23 raised for increases. (6) SG-08 accepted; other SGs Proposed – pending owner review. (7) Every MVP requirement labelled MVP-A or MVP-B; build order for both stages added to the roadmap; 3.12 split into POS (MVP-A) and online (MVP-B) parts; 3.16 basic reports depend only on 3.8 and 3.13. (8) Audit framework moved to Phase 2 (actor `system`), with 3.1 wiring the real user; 3.15 is viewing only. |
| 24-Nov-2026 | 1.2 | Step 1.2 started: technology stack, justification, development environment and coding conventions documents; ADR-016 – 034 added as Proposed; OQ-24 – OQ-26 raised. |
