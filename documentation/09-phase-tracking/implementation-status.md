# BakeFlow – Implementation Status

> **Last updated:** 24-Nov-2026 (step 1.1) · **Owner:** project owner marks steps **Done** after review

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
- **Dependencies** refer to roadmap IDs ([feature-roadmap.md](../00-project-overview/feature-roadmap.md)).
- **Pending decisions** refer to open questions (OQ-xx) in [scope-and-assumptions.md](../00-project-overview/scope-and-assumptions.md#5-open-questions-oq-xx).
- Phase 1 is marked **Done** only after all documentation steps (1.1–1.8) are created and reviewed (master-prompt §6).
- Technical debt is tracked separately in [technical-debt.md](technical-debt.md).

---

## 2. Summary

| Phase | Status | Progress |
|-------|--------|----------|
| Phase 1 – Documentation | In progress | 1.1 in progress, 0 / 8 steps done |
| Phase 2 – Tech stack & project setup | Not started | – |
| Phase 3 – Module implementation | Not started | 0 / 18 modules |
| Phase 4 – Advanced engineering | Not started | – |
| Phase 5 – Hardening & deployment | Not started | – |

---

## 3. Phase 1 – Documentation

| ID | Name | Status | Dependencies | Deliverables | Pending decisions |
|----|------|--------|--------------|--------------|-------------------|
| 1.1 | Project overview, requirements & key decisions | **In progress** | master-prompt.md | `00-project-overview/*` (7 docs), `02-system-design/design-decisions.md`, `09-phase-tracking/*`, `documentation/README.md` | OQ-01 – OQ-22 (owner review) |
| 1.2 | Technology stack documentation | Not started | 1.1 | `01-technology-stack/*`: stack, reasons, alternatives, dev environment, tools, React ↔ Spring Boot ↔ MySQL communication | Frontend structure, image storage (new ADRs) |
| 1.3 | Database design & data dictionary | Not started | 1.1, 1.2 | `03-database-design/*`: ER diagram, schema, keys, indexes, normalisation, `data-dictionary.md`, sample data, SQL queries | OQ-01, OQ-02, OQ-09, OQ-19 |
| 1.4 | Business workflows | Not started | 1.3 | `05-business-workflows/*`: 16 workflows with sequence diagrams and examples | OQ-03, OQ-04, OQ-11, OQ-13, OQ-14 |
| 1.5 | API documentation | Not started | 1.3, 1.4 | `04-api-documentation/*`: endpoints per module with JSON examples, errors, roles | – |
| 1.6 | System design (HLD/LLD) & diagrams | Not started | 1.3–1.5 | `02-system-design/*`: HLD, LLD, 9 required Mermaid diagrams | – |
| 1.7 | Security, testing & deployment documentation | Not started | 1.5, 1.6 | `06-security/*` (incl. permission matrix), `07-testing/*`, `08-deployment/*` | OQ-21 |
| 1.8 | Consistency review & completion report | Not started | 1.1–1.7 | Cross-document review, Phase 1 completion report | All remaining OQs |

---

## 4. Phase 2 – Tech Stack & Project Setup

| ID | Name | Status | Dependencies | Deliverables | Pending decisions |
|----|------|--------|--------------|--------------|-------------------|
| 2 | Skeleton, Docker, Flyway, CI | Not started | Phase 1 Done | Backend and frontend skeletons, Docker Compose, Flyway baseline, GitHub Actions CI, error handling, logging with correlation ID, OpenAPI, Testcontainers base. Cross-cutting: audit capture, idempotency store, money and time utilities | Decisions from 1.2 |

---

## 5. Phase 3 – Module Implementation

| ID | Name | MVP | Status | Dependencies | Deliverables (FRs) | Pending decisions |
|----|------|-----|--------|--------------|--------------------|-------------------|
| 3.1 | Authentication & authorization | Yes | Not started | 2 | FR-ADM-13; auth foundation for FR-CUS-01/02 | OQ-21 |
| 3.2 | Organization, branches & employees | Yes | Not started | 3.1 | FR-ADM-01, 02, 03, 16; FR-BR-38 | OQ-16 |
| 3.3 | Product catalog & pricing | Yes | Not started | 3.2 | FR-ADM-04–08; FR-BR-01–04 | OQ-01, OQ-07, OQ-08, OQ-09 |
| 3.4 | Inventory: batches & stock movement ledger | Yes | Not started | 3.3 | FR-BR-05, 06, 07 | OQ-19 |
| 3.5 | Inventory: FEFO allocation & concurrency | Yes | Not started | 3.4 | FR-BR-08, 09, 10 | OQ-17, OQ-22 |
| 3.6 | Inventory: adjustments, expiry job & alerts | Yes | Not started | 3.5 | FR-BR-11, 12, 13, 14, 34 | OQ-13 |
| 3.7 | Production management | Partial | Not started | 3.4 | FR-BR-15 (MVP), FR-BR-16 | – |
| 3.8 | POS & billing (invoices, GST, refunds) | Yes | Not started | 3.3, 3.5 | FR-BR-20, 21, 23, 24, 25 | OQ-02, OQ-14, OQ-15, OQ-16 |
| 3.9 | Customer accounts & catalog browsing | Yes | Not started | 3.1, 3.3, 3.4 | FR-CUS-01–08, 20, 21; FR-ADM-22 | OQ-20 |
| 3.10 | Cart, checkout, reservation & payment | Yes | Not started | 3.5, 3.9 | FR-CUS-10–14 | OQ-05, OQ-10 |
| 3.11 | Order management & tracking | Yes | Not started | 3.8, 3.10 | FR-CUS-15, 16, 17, 25; FR-BR-35, 36 | OQ-03, OQ-04 |
| 3.12 | Discounts & near-expiry offers | Yes (basic) | Not started | 3.6, 3.8, 3.10 | FR-ADM-14; FR-BR-22, 26, 27, 28; FR-CUS-09 | OQ-11, OQ-12, OQ-15 |
| 3.13 | Wastage management | Yes | Not started | 3.6 | FR-BR-29, 30; FR-ADM-15 | – |
| 3.14 | Inter-branch stock transfer | No | Not started | 3.5, 3.13 | FR-BR-37 | – |
| 3.15 | Notifications & audit trail | Partial | Not started | 3.2 | FR-CUS-18; FR-BR-33, 39; FR-ADM-20 | – |
| 3.16 | Analytics & reports | Partial | Not started | 3.8, 3.11, 3.13 | FR-BR-17, 18, 31, 32; FR-ADM-09–12, 17 | OQ-18 |
| 3.17 | Demand forecasting & production recommendation | No | Not started | 3.7, 3.16 | FR-BR-19; FR-ADM-18 | – |
| 3.18 | Loyalty & reviews | No | Not started | 3.11 | FR-CUS-19, 22, 23, 24; FR-ADM-19, 21 | – |

---

## 6. Phase 4 and Phase 5

| ID | Name | Status | Dependencies | Deliverables | Pending decisions |
|----|------|--------|--------------|--------------|-------------------|
| 4 | Advanced engineering (Redis caching, async events, performance) | Not started | MVP modules | Redis caching, domain events / outbox, query tuning, ledger reconciliation job | Caching and event ADRs |
| 5 | Hardening & deployment | Not started | MVP modules (Phase 4 recommended) | OWASP review, load and concurrency tests, backups and restore drill, monitoring, production deployment, Razorpay live mode | Hosting choice (step 1.7) |

---

## 7. Change Log

| Date | Step | Change |
|------|------|--------|
| 24-Nov-2026 | 1.1 | File created. 1.1 marked In progress, pending owner review. |
