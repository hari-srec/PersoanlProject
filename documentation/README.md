# BakeFlow Documentation

**BakeFlow** is a multi-branch bakery management and online ordering platform: batch-level inventory with FEFO, production, POS billing with GST, expiry and wastage management, and a customer ordering website.

This folder is the **single source of truth** for the project. Code is written only after the documentation for a step has been reviewed.

## Table of Contents

1. [Purpose](#1-purpose)
2. [How to Navigate](#2-how-to-navigate)
3. [Official Roadmap Summary](#3-official-roadmap-summary)
4. [Folder Map](#4-folder-map)
5. [Document Index](#5-document-index)
6. [ID Conventions](#6-id-conventions)

---

## 1. Purpose

This documentation:

- explains **what** BakeFlow must do (requirements) and **why** (business rules and goals),
- records **how** it will be built (decisions, design, database, APIs),
- lets every piece of code and every test be traced back to a requirement, and
- serves as a learning resource for professional software architecture.

The original brief is [master-prompt.md](master-prompt.md). It is **read-only input**. Its Phase 0–12 numbering is superseded by the roadmap below.

## 2. How to Navigate

| If you want to… | Start here |
|-----------------|-----------|
| Understand the product in 5 minutes | [project-vision.md](00-project-overview/project-vision.md) |
| See the rules that must never be broken | [business-requirements.md](00-project-overview/business-requirements.md) |
| Find what a feature must do | [functional-requirements.md](00-project-overview/functional-requirements.md) |
| See quality targets (speed, security…) | [non-functional-requirements.md](00-project-overview/non-functional-requirements.md) |
| Know who can do what | [user-roles-and-personas.md](00-project-overview/user-roles-and-personas.md) |
| See what is built when | [feature-roadmap.md](00-project-overview/feature-roadmap.md) |
| Check assumptions and decisions waiting for the owner | [scope-and-assumptions.md](00-project-overview/scope-and-assumptions.md) |
| See which technologies are used and why | [technology-stack.md](01-technology-stack/technology-stack.md), [technology-justification.md](01-technology-stack/technology-justification.md) |
| Set up your Mac for development | [development-environment.md](01-technology-stack/development-environment.md) |
| Follow naming and structure rules | [coding-conventions.md](01-technology-stack/coding-conventions.md) |
| Understand key technical decisions | [design-decisions.md](02-system-design/design-decisions.md) |
| Check current progress | [implementation-status.md](09-phase-tracking/implementation-status.md) |

**Suggested reading order for newcomers:** vision → business requirements → roles → functional requirements → roadmap → technology stack → coding conventions → design decisions.

## 3. Official Roadmap Summary

| Phase | Content |
|-------|---------|
| **1 – Documentation** | 1.1 Overview, requirements & key decisions · 1.2 Technology stack · 1.3 Database design & data dictionary · 1.4 Business workflows · 1.5 API documentation · 1.6 System design (HLD/LLD) & diagrams · 1.7 Security, testing & deployment · 1.8 Consistency review & completion report |
| **2 – Tech stack & project setup** | Skeleton, Docker, Flyway, CI, audit framework |
| **3 – Module implementation** | 3.1 Auth · 3.2 Organization, branches & employees · 3.3 Catalog & pricing · 3.4 Batches & ledger · 3.5 FEFO & concurrency · 3.6 Adjustments, expiry job & alerts · 3.7 Production · 3.8 POS & billing · 3.9 Customer accounts & browsing · 3.10 Cart, checkout, reservation & payment · 3.11 Orders & tracking · 3.12 Discounts & near-expiry offers · 3.13 Wastage · 3.14 Inter-branch transfer · 3.15 Notifications & audit trail · 3.16 Analytics & reports · 3.17 Forecasting · 3.18 Loyalty & reviews |
| **4 – Advanced engineering** | Redis caching, async events, performance |
| **5 – Hardening & deployment** | Security hardening, load tests, backups, production deployment |

Details, MVP scope and dependencies: [feature-roadmap.md](00-project-overview/feature-roadmap.md).

## 4. Folder Map

```
documentation/
├── README.md                  ← you are here
├── master-prompt.md           ← original brief (read-only)
├── 00-project-overview/       ← vision, requirements, roles, roadmap, scope   (step 1.1)
├── 01-technology-stack/       ← stack, justification, environment, conventions (step 1.2)
├── 02-system-design/          ← ADR log (1.1), HLD/LLD & diagrams             (step 1.6)
├── 03-database-design/        ← ER diagram, schema, data dictionary           (step 1.3)
├── 04-api-documentation/      ← REST API reference per module                 (step 1.5)
├── 05-business-workflows/     ← end-to-end business processes                 (step 1.4)
├── 06-security/               ← authentication, authorization, isolation      (step 1.7)
├── 07-testing/                ← test strategy and scenarios                   (step 1.7)
├── 08-deployment/             ← Docker, CI/CD, monitoring, backups            (step 1.7)
└── 09-phase-tracking/         ← implementation status, technical debt         (step 1.1)
```

## 5. Document Index

### Created

| Document | Folder | Step |
|----------|--------|------|
| [Project vision](00-project-overview/project-vision.md) | 00-project-overview | 1.1 |
| [Business requirements](00-project-overview/business-requirements.md) | 00-project-overview | 1.1 |
| [Functional requirements](00-project-overview/functional-requirements.md) | 00-project-overview | 1.1 |
| [Non-functional requirements](00-project-overview/non-functional-requirements.md) | 00-project-overview | 1.1 |
| [User roles and personas](00-project-overview/user-roles-and-personas.md) | 00-project-overview | 1.1 |
| [Feature roadmap](00-project-overview/feature-roadmap.md) | 00-project-overview | 1.1 |
| [Scope, assumptions and open questions](00-project-overview/scope-and-assumptions.md) | 00-project-overview | 1.1 |
| [Technology stack](01-technology-stack/technology-stack.md) | 01-technology-stack | 1.2 |
| [Technology justification](01-technology-stack/technology-justification.md) | 01-technology-stack | 1.2 |
| [Development environment](01-technology-stack/development-environment.md) | 01-technology-stack | 1.2 |
| [Coding conventions](01-technology-stack/coding-conventions.md) | 01-technology-stack | 1.2 |
| [Design decisions (ADR log)](02-system-design/design-decisions.md) | 02-system-design | 1.1, 1.2 |
| [Implementation status](09-phase-tracking/implementation-status.md) | 09-phase-tracking | 1.1 |
| [Technical debt register](09-phase-tracking/technical-debt.md) | 09-phase-tracking | 1.1 |

### Planned

Document names below are indicative. They are confirmed when each step starts.

| Planned content | Folder | Status |
|-----------------|--------|--------|
| ER diagram, database schema, indexing, normalisation, sample data & queries | 03-database-design | Planned – created in step 1.3 |
| Data dictionary (`data-dictionary.md`) | 03-database-design | Planned – created in step 1.3 |
| Business workflows (16 processes with sequence diagrams) | 05-business-workflows | Planned – created in step 1.4 |
| API documentation per module | 04-api-documentation | Planned – created in step 1.5 |
| High-level design, low-level design and the 9 required diagrams | 02-system-design | Planned – created in step 1.6 |
| Security documentation and detailed permission matrix | 06-security | Planned – created in step 1.7 |
| Testing strategy and scenarios | 07-testing | Planned – created in step 1.7 |
| Deployment documentation | 08-deployment | Planned – created in step 1.7 |
| Phase 1 consistency review & completion report | 09-phase-tracking | Planned – created in step 1.8 |

## 6. ID Conventions

All IDs are permanent and never reused, so later documents can trace back to them.

| Prefix | Meaning | Defined in |
|--------|---------|-----------|
| `BG-xx` | Business goal | business-requirements.md |
| `BR-xx` | Business rule | business-requirements.md |
| `FR-CUS-xx` / `FR-BR-xx` / `FR-ADM-xx` | Functional requirement (customer / branch / central admin) | functional-requirements.md |
| `NFR-xx` | Non-functional requirement | non-functional-requirements.md |
| `AS-xx` | Assumption | scope-and-assumptions.md |
| `OQ-xx` | Open question for the owner | scope-and-assumptions.md |
| `OS-xx` | Out-of-scope item | scope-and-assumptions.md |
| `L-xx` | Known limitation | scope-and-assumptions.md |
| `SG-xx` | Architect suggestion (not in the original brief) | scope-and-assumptions.md |
| `ADR-xxx` | Architecture decision record | design-decisions.md |
| `TD-xx` | Technical debt item | technical-debt.md |

Note: the folders for later steps are created but kept empty until their step. Git does not track empty folders, so they will appear in the repository once they contain documents.
