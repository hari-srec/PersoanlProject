# PROJECT: BAKEFLOW – MULTI-BRANCH BAKERY MANAGEMENT & ONLINE ORDERING PLATFORM

> **Note on phase numbering:** This master prompt originally used Phase 0–12.
> That numbering is **superseded** by the official roadmap given in the Phase 1.1
> prompt (Phase 1 Documentation, Phase 2 Setup, Phase 3.x Modules, Phase 4
> Advanced engineering, Phase 5 Hardening & deployment). Wherever this file says
> "Phase 0", read "Phase 1 (Documentation)". The appendices at the end of this file
> are part of the project requirements and context.

## ROLE

Act as a Senior Software Architect, Java Backend Engineer, Database Architect, System Design Engineer, and Technical Documentation Specialist.

I am developing a real-world bakery management platform similar to a bakery chain with multiple branches.

I want to develop this application phase by phase using AI-assisted coding.

My primary objective is to understand professional software architecture, backend engineering, database design, system design, API development, and real-world business workflows.

Do not rush into coding.

Your FIRST responsibility is to prepare complete technical documentation.

---

# 1. PROJECT REQUIREMENTS

The application must contain three major areas.

## A. Customer Platform

Customers should be able to:

- Register and log in.
- Browse bakery products.
- Search products by category.
- View product availability based on their selected branch.
- View product price and offers.
- View applicable discounts on near-expiry products.
- Add products to cart.
- Place orders.
- Select pickup or delivery where supported.
- Make online payments.
- Track order status.
- View previous orders.
- Cancel eligible orders.
- Receive order notifications.
- Submit product reviews and ratings.
- Manage their profile and addresses.

## B. Branch Management Platform

Each branch should have its own authorized employees.

Branch owners and employees should be able to:

**PRODUCT MANAGEMENT:**

- View available products.
- Manage branch-specific product availability.
- Configure selling prices subject to central pricing rules.
- Manage product categories.

**INVENTORY MANAGEMENT:**

- Add stock.
- Track stock by individual batch.
- Record manufactured/baked date.
- Record expiry date and time.
- Track received, sold, reserved and available quantities.
- Track damaged and expired products.
- View low-stock alerts.
- Track stock movement history.
- Support FEFO stock allocation.

**PRODUCTION MANAGEMENT:**

- Record daily production.
- View historical sales.
- Recommend production quantities.
- Compare actual sales against production.
- Reduce overproduction.

**BILLING:**

- Create customer bills.
- Support cash, UPI and card payment records.
- Apply discounts.
- Process eligible refunds.
- Generate invoices.
- Generate daily closing reports.

**EXPIRY MANAGEMENT:**

- Identify products approaching expiry.
- Configure discount policies.
- Recommend discounts based on remaining shelf life.
- Prevent expired products from being sold.
- Record expired stock and wastage.

**BRANCH OPERATIONS:**

- View branch-level analytics.
- Transfer inventory between branches.
- Manage employee access.
- Track employee actions.

## C. Central Administration Platform

Central administrators should be able to:

- Create and manage bakery organizations.
- Create and manage multiple branches.
- Assign branch managers.
- Manage global product catalog.
- Configure product categories.
- Configure global pricing rules.
- View consolidated inventory.
- View branch-wise revenue.
- Compare branch performance.
- Monitor product wastage.
- Configure system-level settings.
- Manage user roles and permissions.

---

# 2. RECOMMENDED TECHNOLOGY STACK

Use the following stack unless you identify a strong technical reason to recommend an alternative.

**Backend:** Java 21, Spring Boot, Spring Security, Spring Data JPA, Hibernate, Maven, REST APIs

**Frontend:** React.js, TypeScript, Tailwind CSS

**Database:** MySQL 8

**Security:** JWT authentication, BCrypt password hashing, role-based access control, branch-level data isolation

**Additional technologies:** Redis for caching, Swagger/OpenAPI for API documentation, JUnit 5 and Mockito for testing, Testcontainers for integration testing, Docker and Docker Compose, GitHub Actions for CI/CD

**Architecture:** Modular Monolith, layered backend architecture, REST API communication, relational database design

Explain why each technology is selected and what alternatives are available.

---

# 3. DOCUMENTATION PHASE – COMPLETE TECHNICAL DOCUMENTATION

IMPORTANT: Do not generate backend or frontend application code during the documentation phase.

Create the following folder structure physically inside the project root:

```
documentation/
├── 00-project-overview/
├── 01-technology-stack/
├── 02-system-design/
├── 03-database-design/
├── 04-api-documentation/
├── 05-business-workflows/
├── 06-security/
├── 07-testing/
├── 08-deployment/
└── 09-phase-tracking/
```

Create meaningful Markdown documents in each folder.

## A. Project Overview Documentation

Prepare:

1. Project vision.
2. Problem statement.
3. Business requirements.
4. Functional requirements.
5. Non-functional requirements.
6. User roles and responsibilities.
7. Complete feature list.
8. MVP versus future enhancements.
9. Project scope and limitations.

Identify all assumptions explicitly.

## B. Technology Documentation

Prepare:

1. Complete technology stack.
2. Reason for choosing each technology.
3. Technology alternatives.
4. Development environment setup.
5. Required software and dependencies.
6. Development tools and their purposes.

Explain how React communicates with Spring Boot and how Spring Boot communicates with MySQL.

## C. System Design Documentation

Prepare both High-Level Design and Low-Level Design.

HLD must include:

- Overall architecture.
- Customer application architecture.
- Branch management architecture.
- Central administration architecture.
- Backend module architecture.
- Database architecture.
- External service integrations.
- Deployment architecture.

LLD must include:

- Module responsibilities.
- Class-level responsibilities.
- Service interactions.
- Controller and repository responsibilities.
- Business logic workflows.
- Important design patterns.
- Transaction management.
- Error-handling strategy.

Create diagrams using Mermaid wherever applicable.

Required diagrams:

1. Overall system architecture.
2. Component diagram.
3. Customer order sequence diagram.
4. Inventory receiving sequence diagram.
5. Batch expiry workflow.
6. Billing workflow.
7. Branch stock transfer workflow.
8. Authentication workflow.
9. Deployment architecture.

Explain every diagram in simple language.

## D. Database Design Documentation

Design a normalized relational database.

Consider the following entities as a starting point. Do not blindly create tables without analyzing relationships.

- **ORGANIZATION:** Organization, Branch, User, Role, Permission, UserBranchMapping
- **PRODUCT:** Product, ProductCategory, ProductVariant, ProductImage, BranchProduct, ProductPrice
- **INVENTORY:** Inventory, InventoryBatch, StockMovement, StockAdjustment, StockTransfer, StockTransferItem, InventoryReservation
- **PRODUCTION:** ProductionPlan, ProductionBatch, ProductionItem, ProductionRecipe
- **CUSTOMER:** Customer, CustomerAddress, CustomerCart, CustomerCartItem
- **ORDER:** CustomerOrder, CustomerOrderItem, OrderPayment, OrderStatusHistory
- **BILLING:** Invoice, InvoiceItem, Payment, Refund
- **DISCOUNT:** DiscountRule, ProductDiscount, DiscountHistory
- **WASTAGE:** WastageRecord, WastageReason
- **ANALYTICS:** DailySalesSummary, ProductSalesSummary
- **AUDIT:** AuditLog, Notification

You may add, remove or combine entities based on proper database normalization and business requirements.

Prepare:

1. ER diagram.
2. Complete database schema.
3. Primary keys.
4. Foreign keys.
5. Unique constraints.
6. Indexing strategy.
7. Relationship explanations.
8. Database normalization decisions.
9. Sample records.
10. Important SQL queries.

## E. DATA DICTIONARY – VERY IMPORTANT

Create a separate `data-dictionary.md` file.

For EVERY table, document the following:

- Table name.
- Business purpose.
- Column name.
- Data type.
- Length or precision.
- Primary key / Foreign key.
- Nullable or non-nullable.
- Default value.
- Description.
- Example value.

Example:

Table: `inventory_batch`

| Column             | Data Type    | Description                 |
| ------------------ | ------------ | --------------------------- |
| id                 | BIGINT       | Primary key                 |
| organization_id    | BIGINT       | Organization reference      |
| branch_id          | BIGINT       | Branch reference            |
| product_id         | BIGINT       | Product reference           |
| batch_number       | VARCHAR(100) | Unique batch identifier     |
| manufactured_at    | DATETIME     | Manufacturing date and time |
| expires_at         | DATETIME     | Expiry date and time        |
| produced_quantity  | INT          | Original batch quantity     |
| available_quantity | INT          | Available stock             |
| reserved_quantity  | INT          | Reserved stock              |
| sold_quantity      | INT          | Sold quantity               |
| status             | VARCHAR(30)  | Batch status                |
| created_at         | DATETIME     | Record creation time        |

This is only an example. Validate the complete schema and add all necessary fields.

For every table also explain:

- Why the table exists.
- Which modules use it.
- How it relates to other tables.
- Which business operation creates or updates its records.
- Important validation rules.

## F. API Documentation

Create API documentation for every module.

For each endpoint include:

- HTTP method.
- Endpoint URL.
- Purpose.
- Authentication requirements.
- Required role and permissions.
- Request headers.
- Request body.
- Response body.
- HTTP status codes.
- Validation rules.
- Error scenarios.

Use realistic JSON examples.

Do not implement the APIs yet.

## G. Business Workflow Documentation

Explain the complete lifecycle of the following processes:

1. Creating an organization.
2. Creating multiple branches.
3. Adding products to the catalog.
4. Receiving stock at a branch.
5. Creating a production batch.
6. Updating inventory.
7. Selling products through POS.
8. Selling products through the customer website.
9. Reserving stock for customer orders.
10. Automatically allocating stock using FEFO.
11. Applying near-expiry discounts.
12. Recording expired products.
13. Recording wastage.
14. Transferring stock between branches.
15. Generating daily sales reports.
16. Recommending next-day production quantities.

Include sequence diagrams and real-world examples.

## H. Security Documentation

Explain:

- JWT authentication.
- Password hashing.
- Role-based authorization.
- Branch-level authorization.
- Organization-level data isolation.
- API access restrictions.
- Input validation.
- SQL injection prevention.
- Audit logging.
- Payment security.

Important: A branch employee must never be able to access another branch's data simply by changing a branch ID in the API request.

## I. Testing Documentation

Prepare:

- Unit testing strategy.
- Integration testing strategy.
- API testing strategy.
- Database testing.
- Authorization testing.
- Inventory concurrency testing.
- Batch expiry edge cases.
- Order reservation testing.
- Billing and refund testing.

Include positive and negative test scenarios.

## J. Deployment Documentation

Explain:

- Docker architecture.
- Backend deployment.
- Frontend deployment.
- MySQL configuration.
- Environment variables.
- Redis configuration.
- CI/CD workflow.
- Logging and monitoring.
- Backup and recovery.

---

# 4. IMPORTANT BUSINESS RULES

The following rules must be considered during system design.

1. Every branch must maintain independent inventory.
2. Every production batch must have manufacturing and expiry information.
3. Stock must be tracked batch-wise.
4. The system should prioritize FEFO when allocating stock.
5. Expired products must not be sold.
6. Near-expiry discounts must follow configurable business policies.
7. Discounts must not modify the original product price.
8. Inventory changes must be recorded in stock movement history.
9. Orders must reserve stock safely.
10. Cancelled orders must release eligible reserved stock.
11. Stock transfers must record dispatch and receipt separately.
12. All critical inventory operations must be transactional.
13. Prevent negative inventory.
14. Every branch-specific operation must verify organization and branch access.
15. Maintain an audit history for important business changes.

Distinguish between manufacturing, stock receipt, stock reservation, sale and wastage. They must not be treated as the same operation.

---

# 5. DOCUMENTATION QUALITY REQUIREMENTS

Every document must:

- Have a clear title.
- Include a table of contents when appropriate.
- Explain technical terms in simple language.
- Include diagrams wherever useful.
- Include practical examples.
- Cross-reference related documents.
- Avoid contradictory definitions.
- Use consistent naming conventions.

Do not create empty placeholder files.

If any design decision is uncertain, document the alternatives and recommend one with justification.

Maintain a central documentation README containing links to all documents.

---

# 6. PHASE TRACKING

Create `documentation/09-phase-tracking/implementation-status.md`.

Include all development phases from the official roadmap.

Track:

- Phase name.
- Tasks.
- Current status.
- Dependencies.
- Deliverables.
- Pending decisions.

Mark the documentation phase as completed only after all documentation has been created and reviewed.

---

# 7. EXECUTION RULES

Follow these instructions strictly:

1. First inspect the existing project directory.
2. Do not overwrite existing files without checking them.
3. Create the documentation folder structure.
4. Prepare the documentation for the current step only.
5. Validate consistency between database design, APIs and business workflows.
6. Verify that all important business rules are covered.
7. Review the documentation as a senior software architect.
8. Identify missing requirements and design risks.
9. Generate a completion report for the current step.

DO NOT:

- Generate backend code.
- Generate frontend code.
- Create database migrations.
- Start implementing APIs.
- Skip documentation.
- Move to the next step automatically.

After completing each step, stop and wait for my confirmation.

---

# FINAL EXPECTATION

I want a professional software engineering documentation package that I can use as the single source of truth throughout the entire project.

---
---

# APPENDIX A – BUSINESS CONTEXT & PRACTICAL EXAMPLE

**Business problem:** Bakeries lose money because of expired products, overproduction, inaccurate branch-level inventory, and poor visibility into daily sales.

**Platform structure:** One platform with three areas:

- **Customer Platform** – browse, order, pay, track orders and discover offers.
- **Branch Management** – inventory, production, expiry, billing and sales.
- **Central Administration** – manage all branches, users, products, pricing, analytics and business settings.

**Practical example (Chennai):** A bakery has three branches – Anna Nagar, T. Nagar and Velachery. Each branch produces 100 chocolate pastries daily, but actual sales differ:

| Branch     | Produced | Sold | Remaining |
| ---------- | -------- | ---- | --------- |
| Anna Nagar | 100      | 95   | 5         |
| T. Nagar   | 100      | 60   | 40        |
| Velachery  | 100      | 98   | 2         |

The application should identify that T. Nagar is consistently overproducing and recommend reducing the next day's production. It should also identify products approaching expiry and recommend discounts before they become waste.

**Key distinction:** Track inventory by individual production batch, not just by product. Two batches of the same product can have different manufacturing dates, expiry dates and quantities.

**Example inventory structure:**

- Product: Chocolate Pastry (Product ID 101)
  - Branch A – Anna Nagar
    - Batch A001 · Manufactured Nov 24 · Expiry Nov 25 · Qty 30
    - Batch A002 · Manufactured Nov 24 · Expiry Nov 26 · Qty 50
  - Branch B – T. Nagar
    - Batch B001 · Manufactured Nov 24 · Expiry Nov 25 · Qty 20

The system must know exactly which batch was sold, which batch is expiring and which batch needs a discount. Use **FEFO (First Expired, First Out)** as the default stock allocation strategy.

**Success measures:** The original brief does not define numeric targets. Propose measurable success measures (e.g., wastage reduction, stock accuracy, POS speed) and record them as assumptions for the owner to confirm.

---

# APPENDIX B – ADDITIONAL FEATURES

1. **Smart Expiry & Discount Management (High priority)** – automatic expiry alerts; configurable discount recommendations; near-expiry product dashboard; automatic removal from online ordering after expiry.
2. **Demand Forecasting (High priority)** – analyze previous sales; recommend tomorrow's production quantity; identify slow-moving and fast-moving products; consider weekdays, weekends and seasonal demand.
3. **Inter-Branch Stock Transfer** – transfer products between branches; track dispatched and received quantities; maintain transfer history; avoid unnecessary new production.
4. **POS & Billing** – counter billing; cash, card and UPI payment records; discounts and refunds; printable invoices and daily closing reports.
5. **Customer Loyalty** – reward points; order history; personalized offers; favorite products and repeat ordering.
6. **Business Analytics** – branch-wise revenue; product-wise profitability; wastage percentage; expiry loss reports; production versus sales comparison.
7. **Automatic Wastage Tracking** – when a product expires, is damaged, or is discarded, the employee records the reason and quantity, so actual wastage cost can be calculated rather than just remaining stock.

---

# APPENDIX C – ARCHITECTURAL DECISIONS TO RESPECT

| Problem                                   | Recommended design                                     |
| ----------------------------------------- | ------------------------------------------------------ |
| Same product, multiple expiry dates       | Batch-level inventory                                  |
| Two employees selling last available item | Transactional stock allocation and concurrency control |
| Customer adds item to cart                | Cart is not a stock reservation                        |
| Customer starts checkout                  | Time-limited inventory reservation                     |
| Product expires                           | Batch status and sale eligibility update               |
| Branch sells more than it produces        | Demand analysis and production recommendation          |
| Central admin sees all branches           | Organization-scoped authorization                      |
| Branch manager accesses another branch    | Server-side branch permission validation               |
| Discount changes                          | Separate discount records with audit history           |
| Product gets transferred                  | Dispatch, in-transit and receipt tracking              |

- **Do not rely only on a scheduled job to prevent expired-product sales.** Validate batch expiry during stock allocation and checkout as well. Scheduled jobs help update statuses and notify employees; they are not the only safety mechanism.
- **Do not automatically discount every near-expiry product.** The business configures discount rules, minimum margins, eligible products and approval requirements.
- **Architecture:** Start with a modular monolith, not microservices – easier for a single developer, simpler transactions across inventory, billing and orders, clear module boundaries, and modules can be extracted later if scaling requires it.

---

# APPENDIX D – RECOMMENDED MVP

The first usable version should support:

1. One organization with multiple branches.
2. Admin and branch employee login.
3. Product catalog.
4. Branch-specific inventory.
5. Batch-wise manufacturing and expiry tracking.
6. Stock receiving and stock adjustments.
7. POS billing.
8. Expiry alerts.
9. Basic discounts.
10. Customer product browsing and ordering.
11. Basic sales and wastage reports.

After the MVP: demand forecasting, loyalty programs, advanced analytics and automation.
