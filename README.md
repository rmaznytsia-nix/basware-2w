# Basware Data

Working materials for a two-week Basware data-architecture engagement. The repository brings together the delivery plan, business and technical reference material, and supporting workshop assets used to define a Databricks lakehouse approach.

> **Confidentiality:** Several files are marked **internal/personal working reference** and are not for client distribution. Treat the whole repository as engagement material unless the document itself states otherwise. Do not upload its contents to unapproved tools or external services.

## Start here

1. Read the [engagement playbook](<Basware_Engagement_Playbook.md>) for the two-week plan, roles, workshop approach, and delivery artifacts.
2. Use the [Basware business glossary](<Basware_Business_101_Glossary.md>) to align on the business, KPI, and data-domain vocabulary.
3. Use the [data-modeling playbook](<Databricks_Data_Modeling_Playbook.md>) when making modeling, data-quality, lineage, or metric-definition decisions.



## Repository guide


| Area                                  | Documents                                                                                                                                                                                                                                                                                              | Purpose                                                                                                                 |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Engagement delivery                   | [Basware Embed — 2-Week Playbook](<Basware_Engagement_Playbook.md>)                                                                                                                                                                                                                                   | Delivery sequence, workshop methods, and decision artifacts.                                                            |
| Business context                      | [Basware Business 101 + Glossary](<Basware_Business_101_Glossary.md>)                                                                                                                                                                                                                               | Basware terminology for KPI and data-domain conversations.                                                              |
| Databricks architecture and modeling  | [Databricks Lakehouse Data Modeling Playbook](<Databricks_Data_Modeling_Playbook.md>), [Databricks on Azure — catch-up guide](<Databricks_on_Azure_Catchup.md>), [Databricks platform architecture changes, 2024–2026](<Databricks_Platform_Architecture_2024-2026.md>), [Unity Catalog Metric Views architecture brief](<Metric_Views_Brief.md>), [Snowflake ↔ Databricks terminology cheat sheet](<Snowflake_Databricks_CheatSheet.md>) | Architecture refresh, 2024–2026 platform changes, modeling patterns, metric-view fitness, and Snowflake-to-Databricks terminology mapping. |
| Presentation and supporting artifacts | [Snowflake to Lakehouse Migration Assessment](<Snowflake%20to%20Lakehouse%20Migration%20Assessment%205-23.pptx>), [Document Taxonomy](<Document_Taxonomy.md>)                                                                                                                             | Migration assessment deck and guidance for choosing document types.                                                     |




## Working approach

The engagement is organized around making KPI definitions operational and auditable:

- Establish the decision and a shared definition before implementing a metric.
- Map each material component to a source, grain, transformation, owner, and test.
- Profile source variation and record unresolved assumptions or Gold-layer deviations.
- Build from Bronze ingestion through Silver cleaning to a Gold dimensional model and governed semantic metrics.

The [Basware Embed — 2-Week Playbook](<Basware_Engagement_Playbook.md>) includes templates for the [KPI definition contract](<Basware_Engagement_Playbook.md>), [RAID/decision log](<Basware_Engagement_Playbook.md>), [evidence pack](<Basware_Engagement_Playbook.md>), mapping sheet, profiling request, variation matrix, and Gold deviation register.

## Conventions

- Keep confirmed facts, assumptions, illustrative patterns, and future-discovery options distinct, following the [evidence and claim convention](<Databricks_Data_Modeling_Playbook.md>).
- Maintain traceability from business definition to source fields, transformations, and validation evidence.
- Add new working documents in a format appropriate to their reader and purpose; see the [Document Taxonomy](<Document_Taxonomy.md>).
- Keep client-ready outputs separate from internal working notes and clearly label their distribution status.

