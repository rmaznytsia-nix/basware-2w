# Snowflake ↔ Databricks / Azure Terminology Cheat Sheet

*Quick-reference mapping for pre-sale and scoping conversations*

---

## Terminology Comparison

| Concept | Snowflake | Databricks / Azure |
|---------|-----------|-------------------|
| **Compute** | Virtual Warehouse | Cluster (job / all-purpose) or SQL Warehouse |
| **Storage format** | Proprietary micro-partitions | Delta Lake (open Parquet + transaction log) |
| **Namespace** | Database.Schema.Table | Catalog.Schema.Table (Unity Catalog) |
| **Governance** | Roles, RBAC, Row Access Policies, Dynamic Data Masking | Unity Catalog: RBAC, Row/Column filters, masks — also governs files, ML models, dashboards |
| **Clustering / pruning** | Clustering Keys, micro-partition pruning | Liquid Clustering (preferred, DBR13+) or ZORDER + file stats |
| **Table maintenance** | Automatic (managed service) | OPTIMIZE (compaction) + VACUUM (retention cleanup) — must be scheduled |
| **Ephemeral tables** | TRANSIENT tables | Delta tables with short VACUUM retention |
| **Caching** | Result cache + warehouse cache | Delta cache + Photon engine acceleration |
| **Cost control** | Auto-suspend / auto-resume warehouse | Cluster auto-termination; Serverless SQL Warehouses |
| **Materialized views** | Materialized Views / Dynamic Tables | Delta Live Tables / Lakeflow declarative pipelines |
| **Transformation engine** | SQL-first (Snowpark for code) | Spark-first (PySpark/Scala/SQL); SQL Warehouses run on Photon-Spark |
| **ELT tooling** | dbt (native support) | dbt (native support) + Lakeflow Pipelines / Delta Live Tables |
| **Ingestion / CDC** | Snowpipe, Snowpipe Streaming, external stages | Lakeflow Connect (DB + SaaS mirroring/CDC), Auto Loader |
| **Streaming** | Snowpipe Streaming, Dynamic Tables | Structured Streaming, Lakeflow Pipelines (streaming mode) |
| **Semi-structured data** | VARIANT column type | Native JSON/struct columns in Delta; VARIANT type also added in DBR |
| **Data sharing** | Secure Data Sharing / Marketplace | Delta Sharing (open protocol) + Databricks Marketplace |
| **Operational / OLTP** | Unistore (Hybrid Tables) | Lakebase (new operational Postgres-compatible DB, GA 2026) |
| **Cross-platform query** | External Tables / Iceberg Tables | Lakehouse Federation (query external DBs from Unity Catalog) |
| **BI layer tie-in** | Native connectors (Tableau, Power BI, etc.) | Power BI Direct Lake via Mirrored Unity Catalog → OneLake (Fabric) |

*Prepared for technical pre-sale use — verify current feature names (Lakeflow, Lakebase, OneLake Federation) against latest Databricks/Microsoft docs before quoting version-specific capabilities.*

---

## Reference Architectures — Azure / Databricks Lakehouse

*Four patterns to frame the scoping conversation, with differentiators and watch-outs*

---

### 1. Classic Azure Databricks Lakehouse (single-platform)

**Use when:** Client wants one governed platform for engineering, ML, and BI; no hard Fabric/Power BI mandate; team can adopt Databricks SQL for reporting.

**Flow:**
```
ADLS Gen2 (raw) 
  → Lakeflow Connect / Auto Loader 
  → Bronze (Delta) 
  → Silver (Delta, dbt or Lakeflow Pipelines) 
  → Gold (Delta, dbt marts) 
  → Databricks SQL Warehouse 
  → BI tool
```

**Differentiators:**
- Single control plane: Unity Catalog governs tables, files, ML models, dashboards together.
- Best story for ML/AI workloads alongside analytics (MLflow, Lakebase for operational AI apps).
- Simplest cost model — one platform, one billing unit (DBUs).

**Watch-outs:**
- Requires the client to standardize BI on Databricks SQL or accept a connector hop to Power BI/Tableau.
- Team needs Spark literacy for anything beyond SQL-only pipelines.

---

### 2. Databricks + Microsoft Fabric (federated / "best of both")

**Use when:** Power BI is the strategic BI standard, or the org is already invested in Fabric capacity; Databricks remains system of record for engineering/ML.

**Flow:**
```
ADLS Gen2 
  → Databricks Bronze/Silver/Gold (Delta) 
  → Mirrored Unity Catalog 
  → OneLake 
  → Power BI Direct Lake (zero-copy)
```

**Differentiators:**
- No data duplication: Fabric reads the same Delta files Databricks writes, via OneLake shortcuts or Unity Catalog mirroring (GA).
- OneLake Catalog Federation (preview) extends governed access further across platforms.
- Lets each team keep its preferred tool — engineers in Databricks, BI/finance in Fabric/Power BI.

**Watch-outs:**
- Unity Catalog governance (row/column security) does NOT automatically apply when Fabric reads underlying Delta files directly — replicate policies in Fabric's permission model if governance is critical.
- Fabric Capacity Units and Databricks DBUs are not directly comparable — cost story needs a workload-based POC, not a spec-sheet comparison.
- Two platforms = two operational surfaces to monitor, patch, and support.

---

### 3. Brownfield migration architecture (Snowflake/legacy → Databricks)

**Use when:** Client has an existing Snowflake, Synapse, or on-prem warehouse and wants to migrate incrementally rather than a big-bang cutover.

**Flow:**
```
Legacy source (Snowflake / SQL Server / Oracle / Teradata) 
  → Lakeflow Connect (CDC/mirroring) 
  → Bronze (Delta) 
  → Silver/Gold (reuse existing dbt models where possible) 
  → coexistence period 
  → cutover
```

**Differentiators:**
- Lakeflow Connect has native mirroring for 9+ source systems including Snowflake, SQL Server, Oracle, Teradata, Redshift, Synapse — reduces custom ingestion build.
- Existing dbt investment transfers almost directly — same layering (stg/int/fct/dim), different execution engine.
- Supports a phased/coexistence cutover: run Gold in both platforms during validation before flipping consumers.

**Watch-outs:**
- Don't rebuild Silver/Gold logic in notebooks if dbt models already exist — port them, don't reinvent.
- Validate query pattern parity (Spark's cost-based optimizer behaves differently from Snowflake's for skewed joins) before committing to timelines.
- Plan for a real coexistence window — brownfield migrations rarely tolerate a single cutover weekend.

---

### 4. Streaming / near-real-time lakehouse

**Use when:** Client needs sub-minute freshness (fraud detection, operational dashboards, IoT) rather than batch-only reporting.

**Flow:**
```
Event source (Kafka/Event Hubs/CDC) 
  → Structured Streaming or Lakeflow Pipelines (streaming mode) 
  → Bronze/Silver (continuous) 
  → Gold (near-real-time aggregates) 
  → serving layer
```

**Differentiators:**
- Structured Streaming + Delta gives exactly-once, ACID-compliant streaming writes — no separate speed layer needed (no Lambda architecture).
- Lakeflow Pipelines can express streaming and batch logic in the same declarative framework, reducing dual-maintenance.
- Lakebase available if the use case needs low-latency operational reads/writes alongside the analytical stream.

**Watch-outs:**
- Streaming DBU cost is materially higher than batch — always model cost of always-on clusters vs. micro-batch/serverless options.
- Small-file problem is real on high-frequency streaming writes — plan OPTIMIZE/compaction cadence from day one.
