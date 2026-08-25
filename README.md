# Basware Data & AI Discovery Lab

Working materials for a two-week Basware data-architecture engagement. The repository brings together the delivery plan, business and technical reference material, backlog context, and supporting workshop assets used to define and demonstrate a Databricks lakehouse approach.

> **Confidentiality:** Several files are marked **internal/personal working reference** and are not for client distribution. Treat the whole repository as engagement material unless the document itself states otherwise. Do not upload its contents to unapproved tools or external services.

## Start here

1. Read the [engagement playbook](Basware_Engagement_Playbook_Ruslan.md) for the two-week plan, roles, workshop approach, and delivery artifacts.
2. Use the [Basware business glossary](Basware_Business_101_Glossary.md) to align on the business, KPI, and data-domain vocabulary.
3. Review the [Data Platform Jira export](DP_Jira_Export.md) to understand the current medallion, Gold DWH, and semantic-layer backlog.
4. Use the [data-modeling playbook](Databricks_Data_Modeling_Playbook.md) when making modeling, data-quality, lineage, or metric-definition decisions.

## Repository guide

| Area | Files | Purpose |
| --- | --- | --- |
| Engagement delivery | [Basware_Engagement_Playbook_Ruslan.md](Basware_Engagement_Playbook_Ruslan.md), [AI_DLC_Demo_Runbook.html](AI_DLC_Demo_Runbook.html) | Delivery sequence, workshop methods, decision artifacts, and demo runbook. |
| Business context | [Basware_Business_101_Glossary.md](Basware_Business_101_Glossary.md), [Client_Meeting_Discovery_Questions.docx](Client_Meeting_Discovery_Questions.docx), [Internal_Presale_Risk_Notes.docx](Internal_Presale_Risk_Notes.docx) | Basware terminology, discovery prompts, and internal commercial context. |
| Data-platform backlog | [DP_Jira_Export.md](DP_Jira_Export.md) | Snapshot of the Data Platform board: Bronze/Silver foundations, Gold DWH, customer dimension, and semantic metrics work. |
| Databricks architecture and modeling | [Databricks_Data_Modeling_Playbook.md](Databricks_Data_Modeling_Playbook.md), [databricks-azure-architecture-catchup.md](databricks-azure-architecture-catchup.md), [Databicks_innovations.md](Databicks_innovations.md), [Snowflake_Databricks_CheatSheet.docx](Snowflake_Databricks_CheatSheet.docx) | Architecture refresh, modeling patterns, platform capability research, and migration reference. |
| Local analysis environment | [macos-data-quality-lab-setup.md](macos-data-quality-lab-setup.md) | macOS setup for profiling, source-to-target mapping, entity matching, and Databricks parity. |
| Presentation and supporting artifacts | `*.pptx`, `*.pdf`, [document_taxonomy.md](document_taxonomy.md) | Technical vision, demo and migration presentations, reference PDFs, and guidance for choosing document types. |

## Working approach

The engagement is organized around making KPI definitions operational and auditable:

- Establish the decision and a shared definition before implementing a metric.
- Map each material component to a source, grain, transformation, owner, and test.
- Profile source variation and record unresolved assumptions or Gold-layer deviations.
- Build from Bronze ingestion through Silver cleaning to a Gold dimensional model and governed semantic metrics.

The playbook includes templates for the KPI definition contract, RAID/decision log, evidence pack, mapping sheet, profiling request, variation matrix, and Gold deviation register.

## Local setup

This repository does not contain a runnable application or a project-level dependency manifest. If you need a local environment for analysis, follow [macos-data-quality-lab-setup.md](macos-data-quality-lab-setup.md). It covers the suggested DuckDB/Python workbench, entity-resolution tools, a separate Spark/Deequ environment, and Databricks CLI/IDE setup.

## Conventions

- Keep confirmed facts, assumptions, illustrative patterns, and future-discovery options distinct, following the conventions in the modeling playbook.
- Maintain traceability from business definition to source fields, transformations, and validation evidence.
- Add new working documents in a format appropriate to their reader and purpose; see [document_taxonomy.md](document_taxonomy.md).
- Keep client-ready outputs separate from internal working notes and clearly label their distribution status.

## Current scope

The tracked Jira snapshot identifies completed foundation work for the medallion infrastructure, Bronze tenant CSV ingestion, and Silver transformation. It also lists Gold DWH dimensional modeling, customer-semantic clarification, and semantic metrics as remaining work. Treat the export as a point-in-time reference and validate status in Jira before planning against it.
