# Basware Data

Working materials for a two-week Basware data-architecture engagement. The repository brings together the delivery plan, business and technical reference material, and supporting workshop assets used to define a Databricks lakehouse approach.

> **Confidentiality:** Several files are marked **internal/personal working reference** and are not for client distribution. Treat the whole repository as engagement material unless the document itself states otherwise. Do not upload its contents to unapproved tools or external services.

## Start here

1. Read the [engagement playbook](./engagement/Basware_Engagement_Playbook.md) for the two-week plan, roles, workshop approach, and delivery artifacts.
2. Use the [Basware business glossary](./business/Basware_Business_101_Glossary.md) to align on the business, KPI, and data-domain vocabulary.
3. Use the [data-modeling playbook](./databricks/Databricks_Data_Modeling_Playbook.md) when making modeling, data-quality, lineage, or metric-definition decisions.
4. Use the [Metric Workshop](./engagement/Metric_Workshop.md) in the KPI case clinic — calculation cards, scenario matrix, and visual sequence.
5. Harvest clinic rulings into the [metric specification template](./engagement/metric-specification-template.md). Clinic artifacts are evidence; the specification is the commitment.

## Repository layout

- **[engagement/](./engagement/)** — delivery sequence, case clinic, and metric-specification template
  - [Basware Embed — 2-Week Playbook](./engagement/Basware_Engagement_Playbook.md) — two-week plan, roles, decision rights, and delivery artifacts
  - [Metric Workshop](./engagement/Metric_Workshop.md) — case-clinic sequence, calculation cards, scenario matrix, and acceptance cards
  - [Metric Specification Template](./engagement/metric-specification-template.md) — one metric, one owner, completeness check for the KPI definition contract
- **[business/](./business/)** — Basware domain vocabulary
  - [Basware Business 101 + Glossary](./business/Basware_Business_101_Glossary.md) — terminology for KPI and data-domain conversations
- **[databricks/](./databricks/)** — platform, modeling, and semantic-layer reference
  - [Databricks Lakehouse Data Modeling Playbook](./databricks/Databricks_Data_Modeling_Playbook.md) — modeling patterns, data quality, lineage, and metric-definition decisions
  - [Databricks on Azure — catch-up guide](./databricks/Databricks_on_Azure_Catchup.md) — reading order for Azure Databricks platform docs
  - [Databricks platform architecture changes, 2024–2026](./databricks/Databricks_Platform_Architecture_2024-2026.md) — architecture digest for identity, catalog, compute, pipelines, and the semantic layer
  - [Unity Catalog Metric Views architecture brief](./databricks/Metric_Views_Brief.md) — fitness of the native semantic layer for Basware KPI logic
  - [Snowflake ↔ Databricks terminology cheat sheet](./databricks/Snowflake_Databricks_CheatSheet.md) — live meeting reference for namespacing, RBAC, and pipeline terms
  - [Snowflake to Lakehouse Migration Assessment](./databricks/Snowflake%20to%20Lakehouse%20Migration%20Assessment%205-23.pptx) — migration assessment deck
- **[reference/](./reference/)** — document-type guidance and captured source material
  - [Document Taxonomy](./reference/Document_Taxonomy.md) — how to choose a document type by reader job
  - [KPI Tree guides capture](./reference/kpitree-guides-capture-2026-08-26.md) — local capture of kpitree.co methodology used by the Metric Workshop
  - [KPI Tree metric glossary capture](./reference/kpitree-glossary-metric-pages.md) — local capture of kpitree.co glossary metric definitions

## Working approach

The engagement is organized around making KPI definitions operational and auditable:

- Establish the decision and a shared definition before implementing a metric.
- Map each material component to a source, grain, transformation, owner, and test.
- Profile source variation and record unresolved assumptions or Gold-layer deviations.
- Build from Bronze ingestion through Silver cleaning to a Gold dimensional model and governed semantic metrics.

The [Basware Embed — 2-Week Playbook](./engagement/Basware_Engagement_Playbook.md) includes templates for the [KPI definition contract](./engagement/Basware_Engagement_Playbook.md#appendix-a--kpi-definition-contract), [RAID/decision log](./engagement/Basware_Engagement_Playbook.md#appendix-b--raid-and-decision-log), [evidence pack](./engagement/Basware_Engagement_Playbook.md#appendix-d--kpi-evidence-pack), mapping sheet, profiling request, variation matrix, and Gold deviation register. The [Metric Workshop](./engagement/Metric_Workshop.md) holds the case-clinic sequence, calculation card, scenario matrix, and acceptance-card templates. After the clinic, harvest rulings into the [metric specification template](./engagement/metric-specification-template.md).

## Conventions

- Keep confirmed facts, assumptions, illustrative patterns, and future-discovery options distinct, following the [evidence and claim convention](./databricks/Databricks_Data_Modeling_Playbook.md#evidence-and-claim-convention).
- Maintain traceability from business definition to source fields, transformations, and validation evidence.
- Add new working documents in a format appropriate to their reader and purpose; see the [Document Taxonomy](./reference/Document_Taxonomy.md).
- Keep client-ready outputs separate from internal working notes and clearly label their distribution status.
- Place new files in the folder that matches their reader job, and add them to this README.
