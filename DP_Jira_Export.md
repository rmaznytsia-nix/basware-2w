# Jira Export — Data Platform (DP) Board 34 Backlog

Source: https://data-ai-dlc.atlassian.net/jira/software/projects/DP/boards/34/backlog

Total issues: 7

| Key | Status | Points | Summary |
|---|---|---|---|
| DP-9 | Done | 5 | Setup Medallion Infrastructure (Schemas & DABs) |
| DP-10 | Done | 5 | Ingest Tenant CSV Invoices from Volume to Bronze |
| DP-12 | Done | 5 | Build Silver Layer Cleaning & Transformation |
| DP-13 | To Do | 13 | Implement Gold DWH Star Schema — Dimensional Model Population |
| DP-15 | To Do | 5 | Design dim_customer — Resolve Workflow-Based Semantic Ambiguity |
| DP-16 | To Do | 8 | Implement dim_customer — Extend Gold DWH per SDR-001 Recommendation |
| DP-17 | To Do | 8 | Build Semantic Layer Metrics — Touchless Rate, Spend, and Approval Analytics |

---

## DP-9: Setup Medallion Infrastructure (Schemas & DABs)

- **Type:** Story
- **Status:** Done
- **Story points:** 5
- **Assignee:** Unassigned
- **Created:** 2026-08-13
- **Updated:** 2026-08-14

**Description:**

```
Create the three medallion layer schemas in aidlc_dev catalog and define DABs resources for orchestration. This establishes the container structure for the invoice processing pipeline.

ACCEPTANCE CRITERIA:
- aidlc_dev.bronze schema created with appropriate permissions
- aidlc_dev.silver schema created with appropriate permissions
- aidlc_dev.gold_dwh schema created with appropriate permissions (dimensional model per ADR-005)
- aidlc_dev.gold_semantic schema created with appropriate permissions (calculated metrics per ADR-003; consumed by DP-17)
- DABs job definitions added to src/transformations/ resource files
- databricks.yml includes new resource files and job definitions
- databricks bundle validate -t dev passes
- Unit tests mock schema creation (using pytest fixtures)

IMPLEMENTATION:
- Create SQL DDL under src/transformations/ for schema setup
- Define Databricks SQL tasks in DABs resource files (*.yml)
- Add Python init files to maintain mono-repo structure
- Unit tests should mock Spark/Delta operations
- NOTE: Standardized on the two-schema gold split (gold_dwh / gold_semantic) per ADR-003/ADR-005, which supersedes the single flat "gold" schema referenced in the demo docs

BRANCH: feature/DP-9-setup-medallion-schemas

COMMITS:
DP-9: create medallion layer schemas
DP-9: add dabs resources for medallion jobs

```

---

## DP-10: Ingest Tenant CSV Invoices from Volume to Bronze

- **Type:** Story
- **Status:** Done
- **Story points:** 5
- **Assignee:** Unassigned
- **Created:** 2026-08-13
- **Updated:** 2026-08-14
- **Links:** blocks DP-12

**Description:**

```
Build Auto Loader pipeline to continuously ingest tenant CSV invoices from /Volumes/aidlc_dev/default/tenant_uploads/ into aidlc_dev.bronze.invoices. Source data is pre-staged in volumes (3 tenants: KRN, GDS, STK).

ACCEPTANCE CRITERIA:
- Auto Loader job reads CSVs from managed volume with proper schema inference
- Table aidlc_dev.bronze.invoices created with columns matching invoice schema
- Rows partitioned by tenant_id and invoice_date
- Checkpoint directory created for Auto Loader state management
- All 470 records (KRN 150 + GDS 200 + STK 120) successfully loaded
- Data integrity: no duplication, all columns populated
- Unit tests verify DataFrame schema and record count using local Spark
- DABs job defined and callable via databricks bundle deploy -t dev

IMPLEMENTATION:
- Python code under src/ingestion/basware_invoices/ reads from volumes
- Use PySpark Auto Loader with cloudFiles format for streaming
- Schema defined in code (not inferred, for stability)
- Delta table with PARTITIONED BY (tenant_id, invoice_date)
- Checkpoint stored in Unity Catalog volume
- Unit tests use local Spark + mock volume paths

BRANCH: feature/DP-10-ingest-invoices-to-bronze

COMMITS:
DP-10: add basware invoices ingestion source
DP-10: define bronze invoices table and dabs job

```

---

## DP-12: Build Silver Layer Cleaning & Transformation

- **Type:** Story
- **Status:** Done
- **Story points:** 5
- **Assignee:** Unassigned
- **Created:** 2026-08-14
- **Updated:** 2026-08-14
- **Links:** is blocked by DP-10, blocks DP-13

**Description:**

```
Transform raw invoice data from bronze layer into conformed, deduplicated, and validated silver layer. Silver is the immediate upstream source for all gold DWH loads.

BLOCKING DEPENDENCY: DP-10 (Ingest Tenant CSV Invoices from Volume to Bronze) must be DONE first.

DATA TRANSFORMATION CONTRACT:

Input: aidlc_dev.bronze.invoices (470 raw rows, CSV schema as-is)
- Loose types (all strings from CSV)
- No deduplication
- Raw status codes (may vary by source system)
- Nulls preserved as-is
- No validation applied

Output: aidlc_dev.silver.invoices (470 deduplicated, typed rows)
- Proper data types: DATE, DECIMAL(12,2), BOOLEAN, STRING
- Deduplicated by (tenant_id, invoice_id)
- Status codes standardized: Received|Matched|Approved|Paid|Disputed|Rejected
- Currency codes validated (ISO 4217)
- Null handling: payment_date NULL for unpaid, approved_date NULL for unapproved
- DQ metadata: _source_record_id, _ingested_at, _loaded_at

TRANSFORMATIONS REQUIRED:
1. Type Casting: invoice_date/due_date/payment_date/approved_date to DATE; invoice_amount/tax_amount to DECIMAL(12,2); exception_flag/touchless_flag to BOOLEAN
2. Deduplication: Natural key (tenant_id, invoice_id); keep last by _ingested_at
3. Status Code Standardization: Map source approval_status to canonical set (6 values)
4. Currency Validation: Validate currency_code matches ^[A-Z]{3}$ (ISO 4217)
5. Null Handling: payment_date NULL if approval_status != Paid; approved_date NULL if approval_status IN (Received, Matched)
6. Data Quality Baseline: No duplicate invoices, valid dates, positive amounts, canonical status codes, valid currencies
7. Lineage Tracking: Add _source_system, _source_record_id (SHA256), _ingested_at, _loaded_at

ACCEPTANCE CRITERIA:
- DP-10 Status: DONE (bronze.invoices with 470 records exists)
- silver schema created
- silver.invoices table created with 22 columns (typed correctly)
- All 470 rows loaded, deduplicated, typed, validated
- Status codes standardized to canonical set
- Null handling enforced (payment_date, approved_date)
- Lineage columns populated (_source_record_id, _source_system, _ingested_at, _loaded_at)
- Record count: 470 input to 470 output (no data loss)
- All DQ assertions passing (uniqueness, null handling, domain validation)
- Unit tests pass: pytest tests/unit/transformations/bronze_to_silver/
- DABs resource file created and included in databricks.yml
- databricks bundle validate -t dev passes

BRANCH: feature/DP-12-build-silver-layer-transformation

COMMITS:
DP-12: implement bronze-to-silver type casting and deduplication
DP-12: add status code standardization and null enforcement
DP-12: add lineage tracking and dq validation
DP-12: add unit tests for silver transformation

```

---

## DP-13: Implement Gold DWH Star Schema — Dimensional Model Population

- **Type:** Story
- **Status:** To Do
- **Story points:** 13
- **Assignee:** Unassigned
- **Created:** 2026-08-14
- **Updated:** 2026-08-14
- **Links:** is blocked by DP-12, blocks DP-17

**Description:**

```
Load seven-table Kimball star schema (6 dimensions + 1 fact) into gold_dwh; implement silver-to-gold transformations with full data quality enforcement and consumer views.

BLOCKING DEPENDENCY: DP-12 (Build Silver Layer Cleaning & Transformation) must be DONE first.

DATA SOURCE (Input):
aidlc_dev.silver.invoices (470 deduplicated, typed records)
- All columns properly typed (DATE, DECIMAL, BOOLEAN, STRING)
- Status codes standardized (Received|Matched|Approved|Paid|Disputed|Rejected)
- Null handling enforced (payment_date NULL for unpaid, approved_date NULL for unapproved)
- Lineage columns present (_source_record_id, _ingested_at, _loaded_at)

TARGET SCHEMA: aidlc_dev.gold_dwh (per ADR-003/ADR-005 — supersedes the flat "gold" schema referenced in the demo docs; DP-9 must create this schema before this story starts)

MODEL SPECIFICATIONS:
- Grain: One row per invoice line per tenant; natural key (tenant_id, invoice_id)
- Fact type: Transaction fact (immutable, append-only)
- SCD: Type 1 for all dimensions (overwrite current values)
- Role-playing dimension: dim_date referenced 3 times (invoice_date_sk, due_date_sk, payment_date_sk)
- Natural key traps: dim_supplier and dim_cost_center are composite/tenant-scoped

TABLES (7):
1. dim_tenant (3 rows: KRN, GDS, STK)
2. dim_supplier (~36 rows: 12 per tenant, composite key)
3. dim_date (365 rows: 2026 calendar)
4. dim_invoice_status (6 static rows)
5. dim_cost_center (~17 rows: 5-7 per tenant, composite key)
6. dim_payment_terms (5 static rows)
7. fact_invoice_transaction (470 rows: one per silver record)

DATA QUALITY: 22 rules across Uniqueness (UNQ-001 to UNQ-013), Referential Integrity (REF-001 to REF-008), Nullability (NUL-001 to NUL-013), Completeness (CMP-001 to CMP-007), Cross-table Accuracy (ACC-001 to ACC-003)

KEY ACCEPTANCE CRITERIA:
- DP-9 Status: DONE (aidlc_dev.gold_dwh schema exists)
- DP-12 Status: DONE (silver.invoices with 470 records exists)
- All 7 tables created and populated in gold_dwh per dimensional-model/README.md spec
- Dimension loading order followed (date → tenant/status/terms → supplier/cost_center → fact)
- Surrogate key lookups resolved (no NULL FKs except payment_date_sk)
- Derived measures computed (days_to_approve, days_to_pay)
- All 22 DQ rules passing
- Natural key traps prevented (composite MERGE on supplier_bk, cost_center_bk)
- Role-playing views created (dim_invoice_date, dim_due_date, dim_payment_date)
- Fact cardinality verified (470 rows from silver)
- Clustering configured per section 11.2 (fact by (tenant_sk, invoice_date_sk), dims by natural keys)
- Unit tests pass (cardinality, referential integrity, derived measures, DQ rules, natural key trap prevention)
- DABs resource file created and included in databricks.yml

BRANCH: feature/DP-13-implement-gold-dwh-star-schema

COMMITS:
DP-13: create gold_dwh schema and dimension tables
DP-13: implement dimension loading with SCD1 MERGE
DP-13: implement fact table population and surrogate key lookup
DP-13: implement data quality rules and validation
DP-13: implement role-playing date views and consumer access
DP-13: add unit tests for gold layer

DOCUMENTATION:
- Per-dimension and fact transformation scripts
- DQ assertion queries (all 22 rules)
- Clustering strategy per section 11.2
- Natural key trap prevention commentary
- README covering loading order, MERGE strategy, natural key traps

```

---

## DP-15: Design dim_customer — Resolve Workflow-Based Semantic Ambiguity

- **Type:** Story
- **Status:** To Do
- **Story points:** 5
- **Assignee:** Unassigned
- **Created:** 2026-08-14
- **Updated:** 2026-08-14
- **Links:** blocks DP-16

**Description:**

```
Resolve semantic ambiguity and sparse population of customer_id source field; design dim_customer table or document why it should remain omitted from gold_dwh.

Business Context:
Basware supports both procure-to-pay (tenant is buyer) and order-to-cash (tenant is seller) workflows. The term "customer" is semantically overloaded:
- Reading A: customer = tenant itself (P2P perspective)
- Reading B: customer = transaction counterparty (O2C perspective)

Data Sparsity:
customer_id is sparse by workflow design (not a data quality failure):
- KRN (Krones): 78.7% NULL (P2P-heavy manufacturing)
- GDS (Geodis): 28.5% NULL (O2C-enabled logistics)
- STK (Stockmann): 7.5% NULL (P2P-heavy retail)

ACCEPTANCE CRITERIA:
- Software Design Record (SDR-001-dim_customer-modeling.md) created under docs/design-records/
  - Section 1: Business context (P2P vs. O2C workflows with examples)
  - Section 2: Source analysis (customer_id population % per tenant)
  - Section 3: Semantic ambiguity (two readings documented)
  - Section 4: Three modeling options with pros/cons
    - Option A: Build dim_customer with sentinel row for P2P
    - Option B: Add as degenerate dimension on fact only
    - Option C: Keep omitted; require business clarification
  - Section 5: Recommendation with rationale
  - Section 6: Approvals (business + architect sign-off)

- If Option A chosen: Design dim_customer table (natural key, attributes, SCD type, sentinel strategy)
- If Option B chosen: Document degenerate dimension approach
- If Option C chosen: Document deferral and future work

- Update dimensional-model/README.md section 7 with decision
- Unit test stubs written (assertions on customer_id nullability per workflow)

BRANCH: feature/DP-15-design-dim-customer-semantics

COMMITS:
DP-15: create software design record for dim_customer modeling
DP-15: design dim_customer table [if Option A approved]
DP-15: document degenerate dim_customer approach [if Option B approved]
DP-15: document dim_customer deferral [if Option C approved]

```

---

## DP-16: Implement dim_customer — Extend Gold DWH per SDR-001 Recommendation

- **Type:** Story
- **Status:** To Do
- **Story points:** 8
- **Assignee:** Unassigned
- **Created:** 2026-08-14
- **Updated:** 2026-08-14
- **Links:** is blocked by DP-15, blocks DP-17

**Description:**

```
Implement dim_customer dimension (or document continued omission) per Software Design Record (SDR-001) recommendations and business approval from DP-15.

BLOCKING DEPENDENCY: DP-15 (Design dim_customer SDR) must be DONE with SDR-001 approved before this story starts.

WORKFLOW CONTEXT:
Basware supports procure-to-pay (P2P: tenant is buyer, customer_id NULL) and order-to-cash (O2C: tenant is seller, customer_id populated) workflows.

SDR-001 OUTCOME (from DP-15):
Will recommend ONE of three modeling approaches:

OPTION A: Build dim_customer for O2C invoices; use sentinel row for P2P NULLs
- Natural key: (tenant_id, customer_id)
- Sentinel row for P2P context
- Fact FK always non-NULL

OPTION B: Build dim_customer with nullable FK on fact
- Fact FK nullable: customer_sk IS NULL for P2P, populated for O2C
- Queries handle workflow distinction

OPTION C: Keep omitted; defer O2C to future schema
- Document deferral decision
- Create follow-up story for O2C model

ACCEPTANCE CRITERIA:
- DP-15 Status: DONE with SDR-001 approved

If Option A or B:
- dim_customer table created in aidlc_dev.gold_dwh per SDR specs
- ~30-40 unique customers populated from silver WHERE customer_id IS NOT NULL
- SCD1 MERGE on composite customer_bk key
- All DQ rules (DQ-CUS-001 to DQ-CUS-005) passing
- Fact table FK referential integrity verified
- Unit tests pass
- DABs resource file created and included in databricks.yml
- Consumer views created
- dimensional-model/README.md section 7 updated

If Option C:
- dimensional-model/README.md section 7 updated with deferral decision
- Follow-up story created for O2C dimensional model
- This story marked DONE (decision documented)
- DOWNSTREAM IMPACT: notify DP-17 owner — the ambiguous metrics (touchless_invoice_rate, exception_rate, avg_approval_cycle_time, days_payable_outstanding, total_spend) fall back to Reading A (tenant-grain) only; no customer_name dimension/FK is available for those metric views

BRANCH: feature/DP-16-implement-dim-customer

COMMITS (Option A/B):
DP-16: create dim_customer table and SCD1 MERGE logic
DP-16: implement customer dimension population from silver
DP-16: add customer dimension data quality rules
DP-16: add unit tests and consumer views for dim_customer
DP-16: update dimensional model documentation per SDR-001

COMMITS (Option C):
DP-16: document dim_customer deferral per SDR-001

```

---

## DP-17: Build Semantic Layer Metrics — Touchless Rate, Spend, and Approval Analytics

- **Type:** Story
- **Status:** To Do
- **Story points:** 8
- **Assignee:** Unassigned
- **Created:** 2026-08-14
- **Updated:** 2026-08-14
- **Links:** is blocked by DP-13, is blocked by DP-16

**Description:**

```
Implement semantic layer metrics (5 total) using Databricks Unity Catalog Metric Views. Metrics provide BI-ready aggregations over the gold DWH star schema and are published into aidlc_dev.gold_semantic per ADR-003.

BLOCKING DEPENDENCIES:
- DP-13 (Gold DWH Star Schema) MUST be DONE
- DP-16 (Implement dim_customer, any option) MUST be DONE — the outcome (A/B/C) determines the customer grain available below

TARGET SCHEMA: aidlc_dev.gold_semantic (per ADR-003 — DP-9 must create this schema before this story starts)

OUTCOME-DEPENDENT SCOPE (branches on DP-16's resolved option):

If DP-16 Option A or B (dim_customer exists):
- All 5 metric views include customer_name in their dimensional grain
- Phase 2 "hook" query (touchless_invoice_rate per customer) resolves to Reading B (counterparty)

If DP-16 Option C (dim_customer deferred/omitted):
- Metric views OMIT customer_name from their dimensional grain entirely — no customer_sk FK exists on the fact table
- Phase 2 "hook" query falls back to Reading A (tenant-grain): touchless_invoice_rate grouped by tenant_name only
- Documentation must state explicitly that per-customer (counterparty) analysis is deferred pending DP-16 follow-up

METRICS (5 Total):

1. touchless_invoice_rate (%) — % invoices processed zero manual touch
   - Dimensions: tenant_name, supplier_name, status_category, cost_center_name [+ customer_name if DP-16 Option A/B]
   - Expr: SUM(CASE WHEN touchless_flag THEN 1 ELSE 0 END) / COUNT(*)
   - NOTE: Semantically ambiguous at per-customer grain (customer = tenant P2P vs. counterparty O2C?) — see DP-15/DP-16

2. exception_rate (%) — % invoices requiring manual review
   - Same dimensions as touchless_rate
   - Expr: SUM(CASE WHEN exception_flag THEN 1 ELSE 0 END) / COUNT(*)

3. avg_approval_cycle_time (days) — Average days receipt to approval
   - Dimensions: tenant_name, supplier_name, cost_center_name [+ customer_name if DP-16 Option A/B]
   - Expr: AVG(days_to_approve)
   - NULL handling: invoices not yet approved return NULL

4. days_payable_outstanding (days) — Amount-weighted days to payment
   - Dimensions: tenant_name, supplier_name, cost_center_name [+ customer_name if DP-16 Option A/B]
   - Expr: SUM(invoice_amount * days_to_pay) / NULLIF(SUM(invoice_amount), 0)
   - Amount-weighted (not simple average)

5. total_spend (currency) — Total spend by supplier/cost_center/tenant
   - Dimensions: tenant_name, supplier_name, status_category, cost_center_name [+ customer_name if DP-16 Option A/B]
   - Expr: SUM(invoice_amount)
   - Unambiguous baseline metric

DEMO TICKET SEQUENCE:

Phase 1 (Control): total_spend by supplier & cost_center
- Unambiguous baseline showing end-to-end medallion pipeline working
- Verification: SUM matches fact_invoice_transaction total

Phase 2 (The Hook): touchless_invoice_rate per customer (or per tenant, if DP-16 Option C)
- INTENTIONALLY surfaces semantic ambiguity from DP-15/DP-16
- Questions triggered: Is customer = tenant (P2P) or counterparty (O2C)?
- This is the "define before build" moment: design assumption surfaces in BI layer
- Stakeholders must clarify before metric can be reliably interpreted

ACCEPTANCE CRITERIA:
- DP-9 DONE: aidlc_dev.gold_semantic schema exists
- DP-13 DONE: gold_dwh with fact (470 rows) + 6 base dims
- DP-16 DONE (any option — scope determined by resolved option, see above)
- All 5 metric views created with dimensional joins matching DP-16's outcome
- Phase 1 validated: total_spend by supplier/cost_center
- Phase 2 validated: touchless_invoice_rate at whatever grain DP-16's outcome supports (ambiguity documented either way)
- NULL handling: P2P customers, unpaid invoices, unapproved invoices
- Amount-weighting in DPO verified: (invoice_amount * days_to_pay) / SUM(invoice_amount)
- Unit tests pass: cardinality (470 rows), null handling, metric math, dim consistency
- Manual Phase 1 query: SELECT supplier_name, cost_center_name, SUM(total_spend) → matches fact total
- Manual Phase 2 query surfaces the ambiguity at whichever grain is available
- Documentation complete: README explains demo sequence + semantic ambiguity + DP-16 outcome dependency
- DABs resource file created and included in databricks.yml
- databricks bundle validate -t dev passes
- Power BI integration ready: metric views accessible for semantic model import

BRANCH: feature/DP-17-build-semantic-layer-metrics

COMMITS:
DP-17: implement semantic layer metric views (5 total)
DP-17: add null handling and amount-weighting logic
DP-17: document demo ticket sequence and semantic ambiguity
DP-17: add semantic layer unit tests
DP-17: add semantic layer documentation (README with demo sequence + DP-16 link)

```

---
