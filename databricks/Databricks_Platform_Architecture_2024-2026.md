**INTERNAL / PERSONAL — WORKING REFERENCE, NOT FOR CLIENT DISTRIBUTION**

# Databricks platform architecture changes, 2024–2026

Why this exists: the previous one-table “innovations” list was not an architecture brief. It mixed Data + AI Summit announcements with GA, treated Azure as if it matched AWS, copied vendor claims (“eliminates lock-in”, “up to 80%”, “up to 40%”) as facts, and omitted the changes that actually alter a 2026 Azure lakehouse: serverless compute and networking, predictive optimization / liquid clustering, Lakeflow as a product (not a June 2024 GA), metric views, ABAC, and UniForm/Iceberg interoperability.

This file is a **working architecture digest** for the Basware Azure Databricks engagement. It is scoped to changes that alter identity, catalog, storage layout, compute, network, pipeline shape, semantic layer, or consumption — not a product changelog.

Companion files: modeling decisions live in the [Databricks Lakehouse Data Modeling Playbook](./Databricks_Data_Modeling_Playbook.md); reading order for Azure platform docs lives in [Databricks on Azure — catch-up guide](./Databricks_on_Azure_Catchup.md).

Checked against Databricks and Azure Databricks documentation **as of August 2026**. Preview status and Azure lag change; verify in the tenant before putting a preview into a design.

## Contents

- [How to read this](#how-to-read-this)
- [Verdict on the previous list](#verdict-on-the-previous-list)
- [The 2026 default Azure Databricks architecture](#the-2026-default-azure-databricks-architecture)
- [1. Governance and catalog](#1-governance-and-catalog)
- [2. Storage and table design](#2-storage-and-table-design)
- [3. Compute and Azure networking](#3-compute-and-azure-networking)
- [4. Pipelines and delivery](#4-pipelines-and-delivery)
- [5. Semantic layer and consumption](#5-semantic-layer-and-consumption)
- [6. Identity, security, and AI control plane](#6-identity-security-and-ai-control-plane)
- [7. Migration tooling](#7-migration-tooling)
- [What the old list got wrong](#what-the-old-list-got-wrong)
- [Explicitly not here](#explicitly-not-here)
- [Sources](#sources)

---



## How to read this


| Column                          | Meaning                                                                                                         |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **When**                        | First material date. Announce ≠ GA. Where they diverge, both are shown.                                         |
| **Status (Aug 2026)**           | Production-readiness on **Azure Databricks**, not the press-release cloud.                                      |
| **Why it changes architecture** | What you would design differently vs. a 2023 Hive Metastore / classic-cluster / ZORDER / Jobs+notebooks estate. |
| **Basware**                     | Relevance to this two-week embed (SalesCloud, CPQ, M-Files, SAP, ARR/margin, Snowflake→lakehouse).              |


Status labels: **GA** (design against it), **Public Preview** (name it, do not depend on it unless the tenant already uses it), **Private Preview / Beta** (do not put it in the target architecture), **Azure lag** (GA elsewhere, not the Azure default yet).

Vendor percentages (80% conversion, 40% faster, 40% storage) are **marketing measurements**, not design inputs.

---



## The 2026 default Azure Databricks architecture

If you are designing a new governed lakehouse on Azure in 2026, the default shape is no longer “workspace + Hive Metastore + classic jobs clusters + ZORDER + BI extract”.

1. **Unity Catalog is mandatory**, not optional. Serverless, predictive optimization, metric views, ABAC, MVs/STs, and most Lakeflow features require it.
2. **Serverless-first compute** for SQL, notebooks, jobs, and Lakeflow pipelines, with **account-level Network Connectivity Configurations** for private access to Azure data. Classic VNet injection remains for workloads serverless cannot run.
3. **UC managed Delta tables**, `CLUSTER BY AUTO`, predictive optimization (OPTIMIZE / VACUUM / ANALYZE), Zstandard compression. Do not design a partition/ZORDER runbook as the default.
4. **Lakeflow** as one product: Connect (managed CDC/SaaS ingest) → Spark Declarative Pipelines (`AUTO CDC`, expectations, streaming tables / materialized views) → Jobs. Deploy with **Declarative Automation Bundles** (formerly Databricks Asset Bundles).
5. **Metric views** as the governed semantic layer for ARR / gross margin / the remaining KPIs. AI/BI Dashboards and Genie consume those definitions; they do not replace them.
6. **ABAC + governed tags** for row/column security at catalog/schema scale, instead of a view-per-policy forest.
7. **Lakebridge** for warehouse SQL conversion (Snowflake in this engagement); do not treat the “80% automated” claim as a plan.

Hive Metastore, hand-written `MERGE` for ordered CDC, per-table ZORDER jobs, and a second semantic layer in Power BI *only* are 2023 patterns. They still exist; they are no longer the architecture you propose.

---



## 1. Governance and catalog


| Change                                  | When                                                                    | Status (Aug 2026)                                                                                                                                                                    | Why it changes architecture                                                                                                                                                                                                                                                                           | Basware                                                                                                    |
| --------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Unity Catalog OSS + Iceberg REST**    | Announce / OSS: Jun 2024. Iceberg REST read GA called out at DAIS 2025. | GA on Azure for UC itself (since 2021–22). OSS is an ecosystem catalog, **not** a replacement for commercial UC.                                                                     | Open catalog APIs and Iceberg REST let non-Databricks engines read UC tables without copying data. Commercial UC still holds lineage, Delta Sharing, ABAC, metric views, and Azure integrations. “Eliminates vendor lock-in” is overstated: you escape *format* lock-in more than *platform* lock-in. | Medium. Relevant if Fabric, Snowflake, or another engine must read Gold. Not a reason to skip UC on Azure. |
| **Delta UniForm / Iceberg reads**       | GA Jun 2024 (Delta 3.2 / DBR 14.3).                                     | GA. Constraints: UC table, column mapping, no deletion vectors on Iceberg-compat v2 path.                                                                                            | One Parquet copy, Delta + Iceberg metadata. Changes the interoperability design: you do not need a dual-write lake.                                                                                                                                                                                   | Medium. Same as above.                                                                                     |
| **ABAC on Unity Catalog**               | Beta at DAIS 2025. Public Preview 3 Nov 2025. **GA 28 Apr 2026.**       | GA. Requires DBR 16.4+ on classic compute. Policies on tables / MVs / STs, not on views directly. GA changed evaluation on views to **session user** (breaking for some Beta users). | Tag-driven row filters and column masks inherit from catalog/schema. This is the scalable alternative to per-table RLS/masking and secure-view sprawl. Pair with governed tags and data classification.                                                                                               | High if KPI data includes PII or partner-vs-customer splits that must be filtered consistently.            |
| **Governed tags + data classification** | Beta DAIS 2025; Public Preview with ABAC.                               | Public Preview / GA depending on surface — **verify in tenant**.                                                                                                                     | Auto-tagging sensitive columns so ABAC policies attach to new tables without a ticket.                                                                                                                                                                                                                | Medium. Useful; do not block the Gold model on preview classification.                                     |


Docs: [Unity Catalog best practices](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/best-practices) · [ABAC](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/abac/) · [Iceberg reads / UniForm](https://learn.microsoft.com/en-us/azure/databricks/delta/iceberg-reads)

Announce: [UC OSS, 12 Jun 2024](https://www.databricks.com/company/newsroom/press-releases/databricks-open-sources-unity-catalog-creating-industrys-only-open) · [UniForm Iceberg GA](https://www.databricks.com/blog/delta-lake-universal-format-uniform-iceberg-compatibility-now-ga) · [ABAC blog (PP-era; status now GA)](https://www.databricks.com/blog/how-scale-data-governance-attribute-based-access-control-unity-catalog)

---



## 2. Storage and table design


| Change                                                  | When                                                                                                                            | Status (Aug 2026)                                                            | Why it changes architecture                                                                                                                                     | Basware                                                                                    |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **Liquid clustering**                                   | DBR 13.3+; **GA with DBR 15.4 LTS**.                                                                                            | GA. Recommended over Hive-style partitions and ZORDER for UC managed tables. | Clustering keys can change without rewriting the whole table. Partition explosion and ZORDER maintenance stop being the default physical-design job.            | High. Contract/subscription facts should not be born partitioned by a guessed date column. |
| **Automatic liquid clustering (**`CLUSTER BY AUTO`**)** | Public Preview with Predictive Optimization; DBR 15.4+ for AUTO.                                                                | GA for UC managed Delta.                                                     | Databricks picks and evolves clustering keys from query patterns. Default for new managed tables should be AUTO, not a workshop debate about partition columns. | High.                                                                                      |
| **Predictive optimization**                             | Default **on for new accounts from 11 Nov 2024**. Gradual enablement of existing accounts targeted to complete **by Aug 2026**. | GA. Runs OPTIMIZE, VACUUM, ANALYZE on UC managed tables.                     | Deletes the standing “weekly OPTIMIZE job” from the operating model. Check whether Basware’s account is enrolled; if not, enable at catalog/schema for Gold.    | High.                                                                                      |
| **Zstandard default on new UC managed tables**          | Rolled out through 2025; recapped in the Jan 2026 DBSQL year-in-review.                                                         | GA as default for **new** managed tables. Existing tables need migration.    | Storage codec is a platform default, not a tuning exercise. The “up to 40% storage saving” is a vendor comparison to older codecs — treat as directional.       | Medium. Cost hygiene, not a model change.                                                  |
| **VARIANT (semi-structured)**                           | PP with DBR 15.3 (2024). Later GA including shredding.                                                                          | GA. Prefer over JSON strings.                                                | Bronze landing for messy SaaS payloads (Salesforce, files) without freezing a struct schema on day one.                                                         | Medium. Useful for SalesCloud / M-Files payloads; Gold still wants typed attributes.       |
| **Materialized views and streaming tables in DBSQL**    | Public Preview 2024. **GA 10 Oct 2024** (AWS/Azure).                                                                            | GA. Unity Catalog + serverless.                                              | Incremental ingest and pre-aggregated Gold become first-class UC objects, not only DLT pipeline datasets. Analysts can own some Silver/Gold in SQL.             | High. Candidate for KPI-serving aggregates once definitions stabilize.                     |


Docs: [Liquid clustering](https://learn.microsoft.com/en-us/azure/databricks/tables/clustering) · [Predictive optimization](https://learn.microsoft.com/en-us/azure/databricks/optimizations/predictive-optimization) · [Standalone materialized views](https://learn.microsoft.com/en-us/azure/databricks/ldp/dbsql/materialized) · [VARIANT ingest](https://learn.microsoft.com/en-us/azure/databricks/ingestion/variant)

Announce: [Automatic liquid clustering](https://www.databricks.com/blog/announcing-automatic-liquid-clustering) · [MVs and STs GA](https://www.databricks.com/blog/announcing-general-availability-materialized-views-and-streaming-tables-databricks-sql) · [2025 DBSQL year-in-review (Zstd, engine)](https://www.databricks.com/blog/2025-review-databricks-sql-faster-every-workload)

---



## 3. Compute and Azure networking


| Change                                                                     | When                                                                                                                                                                                | Status (Aug 2026)                                                                                                  | Why it changes architecture                                                                                                                                                      | Basware                                                                               |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Serverless compute for notebooks, jobs, DLT/Lakeflow**                   | **GA 15 Jul 2024** on Azure.                                                                                                                                                        | GA where the workspace is UC-enabled and in a supported region. Default for new jobs/notebooks in many workspaces. | Removes cluster policy as the first design question for ETL. Changes identity (shared access / serverless), library strategy, and network (no classic VNet for these workloads). | High. Target compute for the embed unless a workload is serverless-incompatible.      |
| **Network Connectivity Configuration (NCC)**                               | Rolled out 2024–2025 as the serverless private-access model on Azure.                                                                                                               | GA. Account-level, regional; attach to workspaces; managed private endpoints.                                      | **This** is the Azure serverless network architecture, not VNet injection. Classic compute still needs VNet/NSG/Private Link of its own.                                         | High. Any production design that uses serverless + ADLS/SQL must include NCC.         |
| **Azure Network Security Perimeter +** `AzureDatabricksServerless` **tag** | Enforcement date **9 Jun 2026** for storage accounts that still allowlist old serverless subnet IDs.                                                                                | Binding for Azure storage firewalls.                                                                               | If Basware allowlisted Databricks serverless subnets on storage, that pattern is expired. Onboard NSP and the service tag.                                                       | High if their storage firewalls are already customized. **Verify.**                   |
| **Private Network Gateway**                                                | Announce DAIS 2026 (Jun).                                                                                                                                                           | **Private Preview on Azure.**                                                                                      | Intended to replace “one private endpoint per resource” with a single serverless-to-private-network path. Do not design the 2-week or Discovery target around it.                | Low until GA. Use NCC.                                                                |
| **Lakebase (serverless Postgres / OLTP)**                                  | Announce DAIS 2025. Azure PP default-on Aug 2025. AWS GA Feb 2026; Azure GA followed (Mar 2026 in secondary reporting). Autoscaling default for new instances from **12 Mar 2026**. | GA on Azure in supported regions — **verify region**.                                                              | Adds an OLTP plane next to the lakehouse (apps, serving, reverse ETL via synced tables). Not required for ARR/margin Gold.                                                       | Low for this SOW. Future Discovery if they want operational serving of KPI snapshots. |


Docs: [Serverless compute](https://learn.microsoft.com/en-us/azure/databricks/compute/serverless/) · [Serverless network / NCC](https://learn.microsoft.com/en-us/azure/databricks/security/network/serverless-network-security/) · [Serverless Private Link](https://learn.microsoft.com/en-us/azure/databricks/security/network/serverless-network-security/serverless-private-link) · [Lakebase](https://learn.microsoft.com/en-us/azure/databricks/oltp/instances/)

Announce: [Serverless notebooks/jobs/DLT GA](https://www.databricks.com/blog/announcing-general-availability-serverless-compute-notebooks-workflows-and-delta-live-tables) · [DAIS 2026 security (AIM, CBI, Private Network Gateway)](https://www.databricks.com/blog/whats-new-databricks-platform-security-and-compliance-data-ai-summit-2026)

---



## 4. Pipelines and delivery


| Change                                                                 | When                                                               | Status (Aug 2026)                                                                                                               | Why it changes architecture                                                                                                                                     | Basware                                                                   |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Declarative Automation Bundles** (formerly Databricks Asset Bundles) | **GA 23 Apr 2024** (CLI 0.218.0). Renamed **16 Mar 2026**.         | GA. This is the CI/CD unit for jobs, pipelines, SQL, ML.                                                                        | Production assets are a Git-deployed bundle with environment targets, not workspace-click jobs. “Repos + jobs UI” is no longer the recommended promotion model. | High. Delivery standard for anything that survives the two weeks.         |
| **Lakeflow announced**                                                 | DAIS **Jun 2024**. Connect “preview soon”.                         | Announcement only at that date.                                                                                                 | Brand umbrella: Connect + Pipelines + Jobs. Do not cite June 2024 as the date the estate could adopt it.                                                        | —                                                                         |
| **Lakeflow product GA**                                                | DAIS **Jun 2025**.                                                 | GA: Connect (connector-by-connector), Declarative Pipelines, Jobs. DLT code remains compatible.                                 | One data-engineering product instead of Auto Loader + DLT + Workflows as three mental models. Jobs are the renamed Workflows.                                   | High.                                                                     |
| **DLT → Lakeflow Spark Declarative Pipelines**                         | Rebrand ~DAIS 2025; open Spark Declarative Pipelines in Spark 4.1. | GA. `import dlt` still works; `from pyspark import pipelines as dp` is current. `AUTO CDC` is **Lakeflow-only**, not OSS Spark. | Vocabulary, Python API, and “what is open vs Databricks-only” change. CDC remains a Databricks pipeline feature.                                                | High. Use current names in workshops.                                     |
| **Lakeflow Connect — Salesforce**                                      | GA **2 Apr 2025** (Salesforce Platform / Workday Reports).         | GA. Connector matrix still mixed GA/preview — check Salesforce vs SAP.                                                          | Managed, UC-governed, serverless ingest from SalesCloud. Strong **buy** alternative to a custom Salesforce CDC job. SAP is **not** implied.                     | High for SalesCloud. SAP still likely Fivetran/partner — see [ingestion / CDC connectors](./Databricks_Data_Modeling_Playbook.md#ingestion--cdc-connectors) in the [Databricks Lakehouse Data Modeling Playbook](./Databricks_Data_Modeling_Playbook.md). |
| **Lakeflow Connect — SQL Server**                                      | Azure RN **26 Aug 2025** GA (Databricks blog ~Sep 2025).           | GA. CDC / change tracking / SCD2 options.                                                                                       | Relevant if any SQL Server sits in front of SAP extracts or staging.                                                                                            | Medium.                                                                   |
| `AUTO CDC` **/** `AUTO CDC FROM SNAPSHOT`                              | Evolution of `APPLY CHANGES` through 2025–26.                      | GA on serverless or Pro/Advanced Lakeflow pipelines. Not in Apache Spark Declarative Pipelines.                                 | Ordered CDC and snapshot-diff SCD1/SCD2 without hand-written MERGE. Sequencing column becomes an architecture decision (source change time, not ingest time).   | High. Contract/Customer history.                                          |


Docs: [Bundles](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/bundles/) · [Lakeflow pipelines](https://learn.microsoft.com/en-us/azure/databricks/ldp/) · [Where is DLT?](https://learn.microsoft.com/en-us/azure/databricks/ldp/concepts/where-is-dlt) · [Lakeflow Connect](https://learn.microsoft.com/en-us/azure/databricks/ingestion/lakeflow-connect/) · [AUTO CDC](https://learn.microsoft.com/en-us/azure/databricks/ldp/cdc)

Announce: [Lakeflow intro (Jun 2024, archived)](https://www.databricks.com/blog/introducing-databricks-lakeflow) · [Lakeflow GA (Jun 2025)](https://www.databricks.com/blog/announcing-general-availability-databricks-lakeflow) · [Connect Salesforce GA](https://www.databricks.com/blog/announcing-general-availability-lakeflow-connect) · [Bundles GA](https://www.databricks.com/blog/announcing-general-availability-databricks-asset-bundles)

---



## 5. Semantic layer and consumption


| Change                                 | When                                                                                                                | Status (Aug 2026)                                                                                 | Why it changes architecture                                                                                                                                                                                            | Basware                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **AI/BI Dashboards**                   | DAIS Jun 2024: **GA on AWS/Azure**.                                                                                 | GA. No extra BI seat license for view-only.                                                       | Native dashboarding on DBSQL without extracting to a second warehouse. Does **not** remove the need for a metric definition.                                                                                           | Medium. Fine for internal ops views; Power BI may remain the enterprise standard.                 |
| **AI/BI Genie**                        | Jun 2024: Public Preview. Subsequent quality/tooling releases 2025–26.                                              | Generally usable; treat certified answers + metric views as the trust path.                       | Conversational BI on governed data. The 2024 claim that you do not need a semantic model is **obsolete**: metric views are that model, and Genie should call them.                                                     | Medium. Useful demo; dangerous if ARR is defined in prompt instructions instead of a metric view. |
| **Unity Catalog metric views**         | Introduced DBR 16.4 (2025 PP). **Business Semantics GA Apr 2026**; core implementation being open-sourced in Spark. | GA. Queryable from SQL, dashboards, Genie, and external BI.                                       | **This is the semantic-layer architecture change of the period.** Measures and dimensions defined once, governed in UC, compiled at query time. Replaces “KPI = a Gold table + a Power BI measure that might diverge”. | **High.** Default home for ARR, gross margin, and the remaining 16 KPIs — with the caveats in the [Unity Catalog Metric Views architecture brief](./Metric_Views_Brief.md). |
| **Databricks Apps**                    | Public Preview Oct 2024 (blog; Azure RN treats Nov 2024 as PP). **GA 13 May 2025** on Azure.                        | GA. Serverless, UC, OAuth.                                                                        | First-party place for Streamlit/Dash/React apps instead of a separate App Service.                                                                                                                                     | Low for two weeks. Possible Discovery if they want a KPI explorer.                                |
| **Databricks One / Genie consumer UI** | Announce Jun 2025. Workspace-level GA early 2026. Later folded into the broader Genie experience (One URL kept).    | GA for workspace consumer access. Account-level / “Genie One” features still moving — **verify**. | Entitlement model: **Consumer access** vs workspace/SQL. Business users should not need a cluster UI.                                                                                                                  | Medium for operating model; low for the Gold model itself.                                        |


Docs: [Metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/) · [Databricks Apps](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/databricks-apps/) · [Genie One](https://learn.microsoft.com/en-us/azure/databricks/genie-one/genie) · [Consumer access](https://learn.microsoft.com/en-us/azure/databricks/ai-bi/consumers/)

Announce: [AI/BI intro](https://www.databricks.com/blog/introducing-aibi-intelligent-analytics-real-world-data) · [Metric views / Business Semantics GA](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai) · [Apps intro (PP)](https://www.databricks.com/blog/introducing-databricks-apps) · [Apps GA](https://www.databricks.com/blog/announcing-general-availability-databricks-apps)

---



## 6. Identity, security, and AI control plane


| Change                                                                     | When                                                                                                                  | Status (Aug 2026)                                                                   | Why it changes architecture                                                                                                                                                                    | Basware                                                                       |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Automatic Identity Management (AIM)**                                    | Native on **Azure Databricks** earlier; **GA for Entra on AWS/GCP at DAIS 2026**. Okta AIM Public Preview on AWS/GCP. | On Azure: already the Entra path. Do not treat June 2026 as “AIM arrives on Azure”. | Account-level auto-provisioning of users, groups, service principals from the IdP. Reduces SCIM-script architecture.                                                                           | Medium. Confirm they are on AIM vs legacy SCIM.                               |
| **Context-Based Ingress**                                                  | DAIS 2026.                                                                                                            | **Public Preview** on AWS, Azure, GCP.                                              | Lets Genie/Dashboards/Apps be exposed with network+identity+scope policies without opening the whole workspace. Interesting for consumer access; not a GA control.                             | Low until GA.                                                                 |
| **Unity AI Gateway**                                                       | GA **Aug 2026**.                                                                                                      | GA.                                                                                 | Central spend, routing, and guardrails for models, agents, MCP, coding tools. Extends UC to AI traffic.                                                                                        | Low for this SOW unless they are already running agents on the same platform. |
| **Compliance expansion (serverless certifications, HITRUST, ISMAP, etc.)** | Rolling 2025–26; DAIS 2026 recap.                                                                                     | Region- and offering-specific.                                                      | Matters only if Basware has a regulatory overlay that blocked serverless. Check the [Trust Center](https://www.databricks.com/trust) + region matrix; do not copy the blog list into a design. | Verify.                                                                       |


Announce: [DAIS 2026 security](https://www.databricks.com/blog/whats-new-databricks-platform-security-and-compliance-data-ai-summit-2026) · [Unity AI Gateway GA](https://www.databricks.com/blog/unity-ai-gateway-generally-available)

---



## 7. Migration tooling


| Change                                                         | When                                                                         | Status (Aug 2026)                                                                      | Why it changes architecture                                                                                                                                                                                                          | Basware                                                                                                                          |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **BladeBridge acquisition / AI-assisted warehouse conversion** | Called out in DBSQL Feb 2025 roundup.                                        | Productized into Lakebridge.                                                           | Databricks started buying conversion IP rather than leaving SQL translation to SI custom tools.                                                                                                                                      | Medium (context).                                                                                                                |
| **Lakebridge**                                                 | **Jun 2025**. Free for customers/partners. Analyzer + converter + validator. | GA as a foundations toolkit. “Up to 80% automation” and “2× faster” are vendor claims. | The architecture implication is: Snowflake (and other EDW) SQL is converted and reconciled, not rewritten from a blank page. Still needs a semantic/model redesign — Lakebridge does not invent a canonical Customer/Contract model. | **High** given the Snowflake→lakehouse assessment already in this repo. Use Analyzer output as evidence, not as the Gold design. |


Announce: [Lakebridge](https://www.databricks.com/blog/introducing-lakebridge-free-open-data-migration-databricks-sql) · Partner kit also listed in [Databricks Partner Portal delivery materials](./Databricks_on_Azure_Catchup.md).

---

A complete *product* inventory for 2024–2026 would be the monthly [Azure Databricks release notes](https://learn.microsoft.com/en-us/azure/databricks/release-notes/product/). That is the wrong artifact for architecture.

---



## Sources

**Docs (status authority — prefer these)**

- [Unity Catalog best practices — Azure](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/best-practices)
- [ABAC — Azure](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/abac/)
- [Metric views — Azure](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/)
- [Liquid clustering — Azure](https://learn.microsoft.com/en-us/azure/databricks/tables/clustering)
- [Predictive optimization — Azure](https://learn.microsoft.com/en-us/azure/databricks/optimizations/predictive-optimization)
- [Iceberg reads / UniForm — Azure](https://learn.microsoft.com/en-us/azure/databricks/delta/iceberg-reads)
- [Serverless compute — Azure](https://learn.microsoft.com/en-us/azure/databricks/compute/serverless/)
- [Serverless network / NCC — Azure](https://learn.microsoft.com/en-us/azure/databricks/security/network/serverless-network-security/)
- [Lakeflow pipelines — Azure](https://learn.microsoft.com/en-us/azure/databricks/ldp/)
- [AUTO CDC — Azure](https://learn.microsoft.com/en-us/azure/databricks/ldp/cdc)
- [Lakeflow Connect — Azure](https://learn.microsoft.com/en-us/azure/databricks/ingestion/lakeflow-connect/)
- [Declarative Automation Bundles — Azure](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/bundles/)
- [Databricks Apps — Azure](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/databricks-apps/)
- [Lakebase — Azure](https://learn.microsoft.com/en-us/azure/databricks/oltp/instances/)
- [Azure Databricks product release notes](https://learn.microsoft.com/en-us/azure/databricks/release-notes/product/) (Nov 2024 predictive opt default; Jul 2024 serverless GA; Oct 2024 MV/ST GA; May 2025 Apps GA; Nov 2025 ABAC PP; Apr 2026 ABAC GA)

**Announcements (provenance, often archived — not status authority)**

- [UC OSS, 12 Jun 2024](https://www.databricks.com/company/newsroom/press-releases/databricks-open-sources-unity-catalog-creating-industrys-only-open)
- [Lakeflow intro, Jun 2024](https://www.databricks.com/blog/introducing-databricks-lakeflow)
- [Lakeflow GA, Jun 2025](https://www.databricks.com/blog/announcing-general-availability-databricks-lakeflow)
- [Lakeflow Connect Salesforce GA](https://www.databricks.com/blog/announcing-general-availability-lakeflow-connect)
- [AI/BI intro, Jun 2024](https://www.databricks.com/blog/introducing-aibi-intelligent-analytics-real-world-data)
- [Business Semantics / metric views GA](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai)
- [Apps intro (PP)](https://www.databricks.com/blog/introducing-databricks-apps)
- [Apps GA](https://www.databricks.com/blog/announcing-general-availability-databricks-apps)
- [Bundles GA](https://www.databricks.com/blog/announcing-general-availability-databricks-asset-bundles)
- [Lakebridge](https://www.databricks.com/blog/introducing-lakebridge-free-open-data-migration-databricks-sql)
- [ABAC (written at Public Preview)](https://www.databricks.com/blog/how-scale-data-governance-attribute-based-access-control-unity-catalog)
- [2025 DBSQL year-in-review](https://www.databricks.com/blog/2025-review-databricks-sql-faster-every-workload)
- [DAIS 2026 security](https://www.databricks.com/blog/whats-new-databricks-platform-security-and-compliance-data-ai-summit-2026)
- [Unity AI Gateway GA](https://www.databricks.com/blog/unity-ai-gateway-generally-available)
- [What’s new with Databricks SQL, Feb 2025](https://www.databricks.com/blog/whats-new-databricks-sql-february-2025) (kept as provenance for the old Git-SQL row; not an architecture source)

