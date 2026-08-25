**INTERNAL / PERSONAL — WORKING REFERENCE, NOT FOR CLIENT DISTRIBUTION**

# Unity Catalog Metric Views — architecture brief

Checked August 2026. Status authority is Azure Databricks documentation. Launch blogs and partner posts are provenance, not a substitute for the docs. Independent sources are used where they contradict or qualify vendor claims.

**Decision this brief supports:** should Basware encode ARR / gross margin / the remaining KPIs as Unity Catalog metric views, keep them in Power BI / dbt MetricFlow, or split layers? Do not treat this as a build plan for the two-week embed.

Related: modeling do/don't in the [Databricks Lakehouse Data Modeling Playbook](<Databricks_Data_Modeling_Playbook.md>); platform timing in [Databricks platform architecture changes, 2024–2026](<Databricks_Platform_Architecture_2024-2026.md>). The [Basware Embed — 2-Week Playbook](<Basware_Engagement_Playbook.md>) already flags an internal presale risk that Metric Views/Genie may be immature for Basware’s business-logic complexity — this brief is the evidence pack for that question.

## Contents

- [Verdict](#verdict)
- [What a metric view is](#what-a-metric-view-is)
- [Capabilities](#capabilities)
- [Constraints](#constraints)
- [Integration](#integration)
- [Perspectives](#perspectives)
- [Suitability for complex enterprise KPIs](#suitability-for-complex-enterprise-kpis)
- [Working recommendation for this engagement](#working-recommendation-for-this-engagement)
- [Illustrative examples](#illustrative-examples)
  - [Capabilities](#examples-capabilities)
  - [Constraints](#examples-constraints)
  - [Integration](#examples-integration)
  - [Suitability](#examples-suitability)
- [Sources](#sources)

---



## Verdict

Metric views are **GA as a Databricks-native semantic object** (April 2026): define measures once, group by any declared field at query time, govern them in Unity Catalog, and consume them natively from SQL, AI/BI Dashboards, and Genie. That is a real architecture change versus copying `SUM`/`CASE` across reports. ([Metric views overview](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/); [Business Semantics GA](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).)

They are **not yet a drop-in enterprise semantic layer** for a Power BI–centric, multi-fact, RLS/ABAC, share-out-of-Databricks estate:

- Core create/query/govern is GA; Catalog Explorer authoring UI and materialization remain **Preview**. ([GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai); [Materialization](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/materialization).)
- Power BI has **no native Metric View model**. Microsoft removed BI compatibility mode from the Databricks Power BI connector; official workaround is DirectQuery native SQL with `MEASURE()`, or a wrapper view that freezes grain. ([BI tools](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/bi-tools); [Power BI partner page](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/power-bi/); community: [BI compatibility mode removal](https://community.databricks.com/t5/community-articles/databricks-metric-views-power-bi-bi-compatibility-mode-removal/td-p/154977).)
- A governed definition is **not a proof of correctness**. Building `SUM` of an upstream daily distinct count yields a wrong monthly figure; the catalog accepts both the right and the wrong YAML. ([Typedef, Jun 2026](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained).)
- Query-time joins to other tables/metric views are **unsupported**. Multi-fact models need a declared bridge in the metric view source. ([Query](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query); [Joins](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)
- Materialization **cannot** sit on tables with row filters, column masks, or ABAC — it is computed as the owner and would bypass per-user security. ([Materialization](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/materialization).)

**For Basware:** use metric views as the **certified consumption contract on a correct Gold grain**, not as the place that invents Contract End Date or partner/customer identity. If Power BI remains the enterprise BI, plan a dual-path (wrapper views or DAX that must not diverge) until partner native support lands. Tableau’s delegated-semantics integration is **expected late 2026**, not current. ([GA blog partner section](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).)

---



## What a metric view is

A metric view is a Unity Catalog view whose definition is YAML (or SQL DDL wrapping YAML). It separates:

- **Source** — a table-like UC object or a SQL query (tables, views, MVs, streaming tables, foreign tables, even other metric views).
- **Fields (dimensions)** — scalars you may `SELECT`, `WHERE`, and `GROUP BY` at runtime.
- **Measures** — named aggregate expressions (`SUM`, `COUNT(DISTINCT)`, ratios, window measures).
- Optional **joins, filters, parameters, agent metadata, materialization**.

Unlike a standard view, aggregations are **not locked at create time**. You define `total_revenue = SUM(amount)` once; a caller groups by month, customer, or partner and the engine compiles the SQL for that grain. Querying a measure requires `MEASURE(measure_name)` (alias `AGG` on DBR 18.1+). `SELECT *` is not supported. ([Overview](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/); [Query](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query); [Create](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/create).)

Introduced in **Databricks Runtime 16.4**. Current create examples assume **17.3+**. SQL warehouses track the SQL channel, so warehouse users do not pin a cluster DBR. ([Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability); [Create prerequisites](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/create).)

There is no `CREATE METRIC VIEW` statement — it is `CREATE VIEW ... WITH METRICS LANGUAGE YAML`. ([Typedef](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained).)

---



## Capabilities


| Capability                          | What it actually does                                                                                                                                                               | Why it matters in an enterprise KPI program                                                                                                                                                                                                                                       | Illustrative example |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| **Define-once measures**            | Engine compiles `MEASURE()` at the requested `GROUP BY`.                                                                                                                            | Stops the “seventeen revenue views” problem — one certified ARR expression instead of one per report. ([Overview](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/).)                                                                                | [C1](#c1-define-once-measures) |
| **Star and snowflake joins**        | Fact `source` + `LEFT OUTER` dimension joins; nested dimension hops. Default cardinality `many_to_one`.                                                                             | Matches Kimball Gold: `Subscription`/`Contract` fact to `Customer`/`Partner`/`Product`. ([Joins](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)                                                                                            | [C2](#c2-star-and-snowflake-joins) |
| **One-to-many / multi-fact**        | DBR **18.1+**, YAML 1.1. Source is a spine; facts aggregate independently. Multi-fact uses an explicit **bridge** in `source` (CROSS JOIN, UNION of key pairs).                     | Can model “customers and their orders” or two facts sharing dimensions **if** you declare the bridge. Sibling one-to-many branches do not cross-multiply. ([Joins](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)                          | [C3](#c3-one-to-many-and-multi-fact) |
| **Window / semi-additive measures** | Trailing/leading/cumulative windows; `semiadditive: first | last` when the order field is omitted from `GROUP BY`; period-over-period `offset`.                                     | NRR/GRR style period movement, snapshot “last balance” — **if** modeled on the right grain. ([Advanced techniques](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/advanced-techniques).)                                                            | [C4](#c4-window-and-semi-additive-measures) |
| **Composability**                   | Measures can reference other measures.                                                                                                                                              | Ratios (gross margin, NRR) stay DRY inside the YAML. ([Advanced techniques](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/advanced-techniques).)                                                                                                   | [C5](#c5-composability) |
| **Parameters**                      | DBR **18.2+**. Bind values at query time, including window size.                                                                                                                    | One metric view, many report variants (as-of date, currency). ([Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability).)                                                                                        | [C6](#c6-parameters) |
| **UC governance**                   | Same hierarchical `SELECT` model as other views. Owner-only edit unless ownership transferred to a **group** (not for materialized metric views).                                   | Fits an enterprise catalog; collaborative edit is a group-ownership pattern, not Git-native. ([Manage](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/manage).)                                                                                     | [C7](#c7-uc-governance) |
| **Agent metadata**                  | Display names, synonyms, comments, formats (YAML 1.1, DBR 17.3+).                                                                                                                   | This is what makes Genie less of a schema-guessing bot. ([Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability); [GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).) | [C8](#c8-agent-metadata) |
| **Materialization (Preview)**       | Lakeflow pipeline maintains aggregated and/or unaggregated MVs; optimizer rewrites queries when a matching grain exists. Additive vs non-additive is used to avoid illegal rollups. | Performance without a second hand-built agg table — **Preview**, serverless Lakeflow required. ([Materialization](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/materialization).)                                                                 | [C9](#c9-materialization-preview) |
| **Native Databricks consumers**     | AI/BI Dashboards apply `MEASURE()` automatically. Genie spaces can be built on metric views. Alerts, Excel add-in, Google Sheets, JDBC/ODBC metadata.                               | Strongest path if consumption stays on Databricks. ([Query — consume in tools](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query).)                                                                                                              | [C10](#c10-native-databricks-consumers) |


Databricks customer quotes on the GA post (iFood: faster queries, better Genie; Zalando: “promising”) are **vendor-selected**, not independent evaluations. ([GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).)

---



## Constraints

Hard constraints from **current Azure docs**, not folklore:


| Constraint                                                | Implication                                                                                                                                                                                                                                                                                                                                                                                                       | Illustrative example |
| --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| `SELECT *` **forbidden; every measure needs** `MEASURE()` | BI tools that generate `SELECT *` or naive `SUM(column)` against a metric view fail unless they use pass-through SQL, a wrapper view, or BI compatibility mode. ([Query](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query); [BI tools](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/bi-tools).)                                                 | [K1](#k1-select-star-and-measure) |
| **Cannot** `JOIN` **a metric view at query time**         | Error `METRIC_VIEW_JOIN_NOT_SUPPORTED`. Wrap in a CTE after aggregating, or model the join **inside** the YAML. Tableau LOD/TopN/sets often compile to joins and break under BI compatibility mode. ([Query](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query); [BI compatibility limitations](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/bi-metric-view).) | [K2](#k2-cannot-join-a-metric-view) |
| `rely.at_most_one_match` **is not validated**             | If you assert no fan-out and the data has fan-out, **measures are silently wrong**. Same class of risk as a wrong PK on a dimension. ([Joins](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)                                                                                                                                                                               | [K3](#k3-rely-at-most-one-match) |
| **One-to-many: fields cannot come from the many side**    | Order attributes as dimensions require flipping which table is `source`. ([Joins — restrictions](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)                                                                                                                                                                                                                            | [K4](#k4-fields-cannot-come-from-the-many-side) |
| **One aggregation function, one source**                  | `SUM(a) / COUNT(b)` across two facts is OK as two aggs then arithmetic; `SUM(fact1.x + fact2.y)` in one function is not. ([Joins](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins).)                                                                                                                                                                                           | [K5](#k5-one-aggregation-function-one-source) |
| **No Delta Sharing, no data profiling** on metric views   | Cannot share the semantic object itself via Delta Sharing; Lakehouse Monitoring-style profiling does not apply to the metric view object. ([Manage](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/manage).)                                                                                                                                                                        | [K6](#k6-no-delta-sharing-or-profiling) |
| **Materialization vs security**                           | Cannot materialize if the view or sources use RLS, column masks, or ABAC — pre-compute uses owner identity. Group ownership of **materialized** metric views is unsupported. ([Materialization](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/materialization); [Manage](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/manage).)                    | [K7](#k7-materialization-vs-security) |
| **Materialization** `mode: relaxed`                       | Optimizer may serve a stale materialization without checking freshness. `TRIGGER ON UPDATE` is not supported on the metric-view schedule. ([YAML materialization](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/yaml-reference); [BI compatibility — materialized caveats](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/bi-metric-view).)                        | [K8](#k8-materialization-mode-relaxed) |
| **BI compatibility mode is Beta**                         | Session `SET`; DirectQuery/live only. Always `SUM` on measure columns in the BI tool. `AVG` can show `1.0`. Distinct/stddev/median can error or be wrong. Client-side grand totals **wrong for non-additive measures**. Range slicers on measures collapse. ([BI compatibility](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/bi-metric-view).)                                                  | [K9](#k9-bi-compatibility-mode-is-beta) |
| **Runtime fragmentation**                                 | Star joins: 16.4+. Snowflakes, materialization, JDBC metadata: **17.3+**. BI compat, refresh MV: **18.0+**. One-to-many, window offset: **18.1+**. Parameters: **18.2+**. Classic clusters on old DBR cannot see the full model. ([Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability).)                                                     | [K10](#k10-runtime-fragmentation) |
| **Grain lives in the table under the metric view**        | Independent of Databricks: if Gold already stored a daily distinct or a pre-rolled snapshot, `SUM` of that column is valid YAML and a false KPI. ([Typedef](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained).)                                                                                                                                          | [K11](#k11-grain-lives-in-the-table) |


---



## Integration



### Inside Databricks (strong)


| Surface          | Pattern                                                                | Notes                                                                                                                                                                                                                                                                                         | Illustrative example |
| ---------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| SQL / notebooks  | `MEASURE()` + explicit fields                                          | Canonical.                                                                                                                                                                                                                                                                                    | [I1](#i1-sql-and-notebooks) |
| AI/BI Dashboards | Dataset on the metric view                                             | `MEASURE()` applied for you; promote dashboard logic *to* a metric view (GA claim). ([GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai); [Dashboards](https://learn.microsoft.com/en-us/azure/databricks/dashboards/manage/data-modeling/datasets).) | [I2](#i2-ai-bi-dashboards) |
| Genie            | Space on metric views                                                  | Grounded in compiled definitions rather than inferred SQL — still only as correct as the YAML and the Gold grain. ([GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).)                                                                              | [I3](#i3-genie) |
| Alerts           | Threshold on a measure                                                 | Ops, not a semantic substitute. ([Query](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query).)                                                                                                                                                                | [I4](#i4-alerts) |
| dbt              | `materialized='metric_view'` in **dbt-databricks ≥ 1.12.0** (May 2026) | Emerging. Typedef: dbt docs were not yet the authority; compile and read DDL. Metric view does not replace dbt models — it sits on them. ([Typedef on dbt and Unity Catalog metrics](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained).) | [I5](#i5-dbt) |




### External BI (uneven — this is the enterprise gap)

Official patterns, in order of fidelity ([BI tools](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/bi-tools)):

1. **Pass-through / native SQL** with `MEASURE()` — keeps definition in UC. Illustrative example: [I6](#i6-passthrough-native-sql).
2. **Wrapper UC view** that embeds `MEASURE()` at a **fixed** grain — BI sees a table; **you re-lock aggregation**, which is exactly what metric views were invented to avoid. Illustrative example: [I7](#i7-wrapper-uc-view).
3. **BI compatibility mode (Beta)** — rewrite `SUM(measure)` to the YAML; **removed from the Power BI connector** by Microsoft. Still usable from Tableau Initial SQL and other tools that can run session `SET`. ([Power BI](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/power-bi/); [BI compatibility](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/bi-metric-view).) Illustrative example: [I8](#i8-bi-compatibility-mode).

Independent practitioner view (Databricks Community, 2026): a metric view behaves like a **one-big-table** semantic model, not a Power BI star with relationships; multi-fact and time grain are awkward; native Power BI support is not on Microsoft’s connector roadmap as of that article; wrapper + DAX **redefines** measures and weakens “single source of truth.” ([Community: Metric Views with Power BI, part 3](https://community.databricks.com/t5/community-articles/metric-views-with-power-bi-and-tabular-editor-part-3-of-3/td-p/160040).)

Partner timeline from the **GA post** (vendor, not GA of the partner feature):


| Partner                                                       | Stated posture (Apr 2026 post)                                          | Illustrative example |
| ------------------------------------------------------------- | ----------------------------------------------------------------------- | ----------- |
| Sigma, Hex, Omni, Atlan, Monte Carlo, Collibra, Domo, Anomalo | Claimed live or launch-partner integrations.                            | [I9](#i9-claimed-live-partner-integrations) |
| Tableau                                                       | Delegated semantics in the relationship model — **expected late 2026**. | [I10](#i10-tableau-delegated-semantics) |
| ThoughtSpot                                                   | Native Metric Views — **later in 2026**.                                | [I11](#i11-thoughtspot-native-metric-views) |


Treat partner rows as **verify in tenant**, not as architecture you can cite to Basware as already true.

### Semantic-layer alternatives (Discovery, not two-week)

The [semantic-layer alternatives](<Databricks_Data_Modeling_Playbook.md>) in the [Databricks Lakehouse Data Modeling Playbook](<Databricks_Data_Modeling_Playbook.md>) already list dbt Semantic Layer / MetricFlow, Cube, and AtScale. Typedef’s split is accurate: MetricFlow is warehouse-portable and auto-derives time grains; UC metric views are Databricks-deep (Genie, UC grants) and historically need an explicit field per time grain. OSI (Open Semantic Interchange) is a **definition-portability** effort (Snowflake, Databricks, dbt, Salesforce, …), not a correctness compiler. ([Typedef on MetricFlow vs Unity Catalog](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained); [GA blog on OSI](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).) Illustrative example: [I12](#i12-semantic-layer-alternatives).

---



## Perspectives

**1. Databricks’ thesis.** Semantics belong in the catalog, not in each BI tool, because agents multiply metric drift. Open-sourcing the Spark implementation (SPARK-54119) and OSI membership are the lock-in counter-argument. ([GA blog](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai).)

**2. Practitioner thesis.** Native Databricks consumption is ready; Power BI estates are not. Metric views are OBT-shaped; complex enterprise models (multi-fact, heterogenous time) still want a dimensional Gold **plus** a thin metric layer, not a metric view that *is* the warehouse. ([Community Power BI series](https://community.databricks.com/t5/community-articles/metric-views-with-power-bi-and-tabular-editor-part-3-of-3/td-p/160040).)

**3. Correctness thesis (Typedef).** Governance ≠ audit. Additive/non-additive handling in materialization is real and better than most layers; it does not see non-additivity **hidden in an upstream Gold column**. Genie will happily group the wrong measure at month grain and cite the catalog. Define-once makes a wrong number **consistent**. ([Typedef](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained).)

**4. Maturity thesis.** Core object is young as GA (April 2026). Preview UI and Preview materialization, Beta BI compatibility, Power BI connector regression, DBR 18.x for the multi-fact features you need for real SaaS metrics — that is a **2026–2027** platform, not a 2024 semantic layer. Independent LinkedIn summary of the GA split (core GA vs UI/materialization Preview) matches the docs. ([Yasiru Randika](https://www.linkedin.com/posts/yasirurandika_databricks-unitycatalog-dataengineering-activity-7482193124879126528-ba9Y); [Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability).)

---



## Suitability for complex enterprise KPIs

Score against patterns that show up in SaaS finance (ARR, NRR/GRR, partner vs end-customer, multi-system contract dates) — **illustrative of Basware-class problems**, not a statement about Basware’s current implementation.


| Enterprise pattern                                         | Fit                                                                     | Why                                                                                                                                                                                       | Illustrative example |
| ---------------------------------------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| Conformed star: one subscription fact, clean dimensions    | **Strong**                                                              | Default many-to-one joins; Photon + UC. This is the happy path the docs teach.                                                                                                            | [S1](#s1-conformed-star) |
| SCD2 dimensions / point-in-time (“as of contract version”) | **Possible, not automatic**                                             | Window/`semiadditive` and parameters help; you still need Gold that already has valid_from/valid_to. Metric views will not invent Contract End Date authority.                            | [S2](#s2-scd2-and-point-in-time) |
| Multi-grain facts (bookings vs ARR vs invoice)             | **Conditional on DBR 18.1+ and an explicit bridge**                     | Docs require you to enumerate valid dimension combinations. This is real modeling work, not a checkbox.                                                                                   | [S3](#s3-multi-grain-facts) |
| Non-additive KPIs (distinct logos, NRR ratios)             | **Safe on raw grain; unsafe if Gold pre-aggregates or BI reaggregates** | Query-time `COUNT(DISTINCT)` is the point of metric views. Wrapper views, Power BI grand totals, and `SUM` of daily distincts are the failure modes Databricks and Typedef both document. | [S4](#s4-non-additive-kpis) |
| Row/column security, ABAC on PII                           | **Query-time OK; materialize-and-share not OK**                         | Do not promise “fast materialized ARR” and “partner-filtered rows” on the same object.                                                                                                    | [S5](#s5-row-column-security-and-abac) |
| Delta Sharing to partners / other clouds                   | **Poor**                                                                | Metric views are not shareable objects. Share Gold tables; redefine or forgo the semantic object on the other side.                                                                       | [S6](#s6-delta-sharing) |
| Power BI as system of engagement                           | **Weak until native connector support**                                 | Official path is SQL workarounds that either freeze grain or live in DirectQuery native queries most semantic-model developers will not maintain.                                         | [S7](#s7-power-bi-as-system-of-engagement) |
| Genie / AI/BI as system of engagement                      | **Strong, with a correctness caveat**                                   | Best integrated consumer. Agents amplify a wrong grain.                                                                                                                                   | [S8](#s8-genie-and-ai-bi-as-system-of-engagement) |
| Code-reviewed metric lifecycle (Git, PR, dbt)              | **Emerging**                                                            | YAML in Git or dbt `metric_view` materialization. UC owner/group edit is the UI path. Not yet a mature metric-ops product.                                                                | [S9](#s9-code-reviewed-metric-lifecycle) |
| Cross-warehouse semantic portability                       | **Not the job**                                                         | Use MetricFlow / OSI later. UC metric views are the Databricks depth play.                                                                                                                | [S10](#s10-cross-warehouse-semantic-portability) |


**Bottom line for “real complex enterprise use cases”:** Metric views are suitable as the **last governed compile step** on a dimensional Gold model that already got grain, keys, and time semantics right. They are a poor place to hide identity resolution, contract-date arbitration, or partner hierarchy. They are a poor **sole** semantic layer if the firm’s analysts live in Power BI Desktop models. They are a good Genie/AI/BI foundation **after** the KPI definition contract exists.

That is the same conclusion the [Basware Embed — 2-Week Playbook](<Basware_Engagement_Playbook.md>) already hinted at; the docs now make it specific rather than a presale hunch.

---



## Working recommendation for this engagement

1. **Do not block week-1 workshops on metric-view syntax.** Resolve ARR grain, Contract End Date, and partner/customer identity in Gold first.
2. **Prototype one certified KPI** (likely ARR) as a metric view on the candidate Gold star **after** the definition is testable — SQL + AI/BI Dashboard, not Power BI, as the proving consumer.
3. **Record Power BI as an explicit architecture decision:** native Databricks consumption vs wrapper-at-fixed-grain vs keep measures in the PBI dataset until Tableau/PBI native support is real in *this* tenant.
4. **Do not enable materialization** on any metric that needs ABAC/RLS, and do not treat Preview materialization as the HA path for month-end ARR.
5. **Require** `COUNT(DISTINCT)` **/ ratios to sit on atomic Gold**, not on daily rolled facts, unless a window/semiadditive measure is explicitly designed and tested at every grain Genie is allowed to ask.
6. Discovery-phase option, not this SOW: dbt MetricFlow or a portable layer **if** Basware will keep Snowflake or a second warehouse in parallel.

---



## Illustrative examples

SaaS-finance situations (ARR, partners, contract dates, NRR/GRR). Not a claim about Basware’s current implementation. Each item is a **use case** that shows what the row means in practice, then an **issue case** that shows the failure mode, then why that pair *is* the feature.

### Examples: Capabilities

#### C1. Define-once measures

**Use case.** Finance certifies `arr = SUM(subscription_arr)` once. The CFO pack groups by month, the partner review groups by VAR, Genie groups by product family. All three call `MEASURE(arr)`. Changing the churn rule is one YAML edit.

**Issue case.** Before that, `gold_arr_month`, `gold_arr_partner`, and a Power BI measure each implement a slightly different “active on month-end” CASE. The month-end meeting is “which ARR?” not “why did ARR move?”

**Essence.** The engine compiles the same measure at whatever grain you ask. It does not invent a second definition per report.

#### C2. Star and snowflake joins

**Use case.** Fact is `fct_subscription`. YAML left-joins `dim_customer`, then hops to `dim_partner` for partner region. ARR by partner region works without denormalizing partner onto every fact row.

**Issue case.** The team flattens partner, customer, and product onto the fact “because metric views can’t join.” They can — only **declared** joins, at defined cardinality. An undeclared hop does not exist at query time.

**Essence.** Kimball joins live in the YAML, not in every analyst’s FROM clause.

#### C3. One-to-many and multi-fact

**Use case.** Spine is customer-month. One branch is bookings (order intake), one is ARR (active subscriptions). An explicit bridge lists valid customer-month keys so “new logos vs ARR” does not explode.

**Issue case.** Someone joins bookings to ARR in a notebook after reading the metric view, or CROSS JOINs the two facts in Gold with no bridge. Ten contracts times twelve invoices become 120 rows; ARR is 12× high. Or the warehouse is on DBR 18.0 and the YAML feature is missing, so the demo “proves” metric views cannot do SaaS.

**Essence.** Multi-fact is a declared bridge on 18.1+, not a Power BI relationship pane.

#### C4. Window and semi-additive measures

**Use case.** Daily ARR snapshots in Gold. Measure `arr_eop` is semi-additive **last** along date, so “ARR as of 31 Mar” is the month-end snapshot, not the sum of 31 days. A trailing window supports NRR-style movement.

**Issue case.** `SUM(arr_snapshot)` across March returns ~31× true ARR. Or `first`/`last` is set but the order field is left in `GROUP BY`, so every day is its own group and “month-end” never collapses.

**Essence.** Snapshot and period-over-period measures are not additive in time. The YAML has to say so.

#### C5. Composability

**Use case.** `gross_profit` and `revenue` are measures; `gross_margin_pct` is one divided by the other inside the YAML. Dashboards and Genie cannot pick a different ratio formula.

**Issue case.** One tile does `SUM(profit)/SUM(revenue)`. Another averages row-level margins. Both are labeled “gross margin.” The YAML would have forced a single composed measure.

**Essence.** Nested measures are how ratios stay DRY. Composition does not fix a wrong grain underneath.

#### C6. Parameters

**Use case.** One `as_of_date` parameter. The same ARR metric view serves “today” and “as of the last board pack” without cloning YAML.

**Issue case.** Ten objects named `arr_asof_2025_12`, `arr_asof_2026_01`, …. Or the parameter is treated as SCD2 time travel while Gold is Type-1 — every as-of date returns the current Contract End Date.

**Essence.** Parameters bind query-time values. They are not a history engine.

#### C7. UC governance

**Use case.** Ownership sits on a Finance group. Analysts have `SELECT`. A Genie space inherits. A YAML patch does not wait on one named employee.

**Issue case.** A single owner is on leave; Catalog Explorer refuses edits. Or the view is **materialized**, group ownership is unsupported, and a named user is a bus factor for month-end ARR.

**Essence.** Grants are catalog-native. Collaborative edit is group ownership (with a materialization hole), not Git, unless you add dbt.

#### C8. Agent metadata

**Use case.** Display name “Annual Recurring Revenue,” synonyms “cloud ARR,” “ARR.” Genie maps “ARR in DACH” to `MEASURE(arr)`, not to an invoice `amount` column.

**Issue case.** No synonyms. Genie sums `invoice.amount` and answers “ARR is €X.” The number is confident, formatted, and the wrong concept.

**Essence.** Metadata is the Genie contract. Without it the agent guesses the schema.

#### C9. Materialization (Preview)

**Use case.** Lakeflow keeps ARR pre-aggregated at month × product. The optimizer rewrites the exec dashboard to the matching grain. Additive vs non-additive stops an illegal rollup of distinct logos.

**Issue case.** Materialization is enabled on a partner-RLS metric view: blocked, or computed as the owner so a regional VP sees every partner. Or `relaxed` mode serves last Tuesday’s ARR on Friday close.

**Essence.** Preview performance, not a security layer, and not month-end HA.

#### C10. Native Databricks consumers

**Use case.** AI/BI dashboard on the metric view; `MEASURE()` is applied for you. An alert fires if GRR drops below the certified threshold. Finance lives in Databricks.

**Issue case.** An analyst opens the object in Excel/ODBC, expects `SELECT *` like a table, gets an error, and rebuilds ARR in a spreadsheet. Native is first-class; generic SQL clients are not.

**Essence.** The strong path is Databricks in, Databricks out.

### Examples: Constraints

#### K1. SELECT star and MEASURE()

**Use case.** `SELECT month, partner, MEASURE(arr) FROM finance.arr GROUP BY month, partner` returns certified ARR at that grain.

**Issue case.** Power BI or a notebook does `SELECT * FROM finance.arr` “to see the columns,” or `SUM(arr)` treating a measure as a numeric column. The query fails or silently mis-aggregates unless you use pass-through SQL, a wrapper, or BI compatibility mode.

**Essence.** A metric view is not a table. Every measure is a function call.

#### K2. Cannot JOIN a metric view

**Use case.** Partner hierarchy is missing from the YAML. You add the join **inside** the definition, or you aggregate first and join the result in a CTE.

**Issue case.** Tableau LOD / Top N “ARR of the top ten partners” compiles to a JOIN. Error: `METRIC_VIEW_JOIN_NOT_SUPPORTED`. The analyst extracts to CSV.

**Essence.** At query time the object is closed. Joins are modeled in, not bolted on.

#### K3. Rely at most one match

**Use case.** You declare customer → billing partner as many-to-one because “each customer has one partner.”

**Issue case.** SAP has bill-to and sold-to rows for the same customer. The join fans out. ARR doubles. **No error.** The board pack is quietly high. Same class of bug as a wrong dimension PK.

**Essence.** The flag is an assertion, not a uniqueness check.

#### K4. Fields cannot come from the many side

**Use case.** Source grain is customer. Orders are a one-to-many measure: `COUNT(order_id)` is fine.

**Issue case.** You want `GROUP BY order_channel` while source is still customer. Channel lives on the many side; the engine refuses. You must flip `source` to orders (and re-think ARR grain) or keep channel out of this metric view.

**Essence.** Dimensions come from the grain you declared as source, not from any joined table you happen to like.

#### K5. One aggregation function, one source

**Use case.** `arr = SUM(subscription.arr)` and `invoice_count = COUNT(invoice.id)`, then `arr_per_invoice = arr / invoice_count`. Two aggregates, then arithmetic.

**Issue case.** One measure `SUM(subscription.arr + invoice.amount)`. Illegal. The workaround people reach for is pre-joining facts in Gold — which recreates the fan-out this rule exists to prevent.

**Essence.** Combine facts after they have been aggregated, not inside a single `SUM`.

#### K6. No Delta Sharing or profiling

**Use case.** You Delta-Share `gold.fct_subscription` to a partner cloud. They define their own metrics, or they don’t.

**Issue case.** The plan is “share `mv_arr` so VARs consume our certified ARR.” Metric views are not shareable objects. DQ tries to profile the metric view like a table; Lakehouse Monitoring does not apply to it.

**Essence.** Share data. The semantic object stays in this catalog.

#### K7. Materialization vs security

**Use case.** Query-time metric view plus RLS: a Nordic VP sees Nordic ARR only.

**Issue case.** Month-end wants it fast, so materialization is switched on. Databricks refuses because RLS/ABAC/column masks are present — pre-compute runs as the **owner** and would leak. The project stalls on “fast and filtered.”

**Essence.** Viewer identity and owner identity are not the same. Pick query-time security or pre-compute, not both.

#### K8. Materialization mode relaxed

**Use case.** Strict freshness for close: do not serve a materialization unless it matches current data (within documented rules).

**Issue case.** `relaxed` is on for speed. The pipeline lagged. Genie answers Friday’s close question with Wednesday’s ARR and does not tell you.

**Essence.** Relaxed means stale-by-design. `TRIGGER ON UPDATE` is not the metric-view schedule.

#### K9. BI compatibility mode is Beta

**Use case.** Tableau Initial SQL sets the session flag. `SUM(arr)` in the tool is rewritten to the YAML measure. DirectQuery/live only.

**Issue case.** Microsoft removed this from the Power BI connector. Where it still runs: `AVG` can show `1.0`; client grand total of distinct logos is the sum of logos per country; range slicers on measures collapse.

**Essence.** A translation hack with documented lies, not a semantic model.

#### K10. Runtime fragmentation

**Use case.** Serverless SQL warehouse on the current channel: snowflakes, one-to-many, parameters — the full YAML.

**Issue case.** Workshop cluster is classic DBR 16.4. Multi-fact and parameters do not exist there. The demo fails; the room concludes “metric views cannot do SaaS metrics.” The catalog object existed; the runtime did not.

**Essence.** Features are pinned to DBR/SQL channel, not to “we created a metric view.”

#### K11. Grain lives in the table

**Use case.** Gold is at subscription-line grain. `COUNT(DISTINCT customer_id)` is true logos at whatever `GROUP BY` the caller asks.

**Issue case.** Gold already stores `daily_active_customers` or a daily ARR snapshot. YAML says `mau = SUM(daily_active_customers)` or `arr = SUM(daily_arr)`. Valid catalog object; monthly figure is the sum of days. Typedef’s MAU trap is the ARR trap.

**Essence.** Governance does not un-bake an upstream grain. Genie will group the wrong number consistently.

### Examples: Integration

#### I1. SQL and notebooks

**Use case.** A validation notebook: `MEASURE(arr)` grouped by partner vs by product, both reconciling to a control total. This is the canonical API.

**Issue case.** Analyst runs `SELECT *` to “see columns” the way they would in Snowflake, hits the error, and decides the object is broken.

**Essence.** First-class for people who will write `MEASURE()`. Hostile to table-shaped exploration.

#### I2. AI BI Dashboards

**Use case.** Dataset is the metric view. Tiles get `MEASURE()` automatically. A one-off dashboard calculation is promoted *into* the YAML so Genie sees it too.

**Issue case.** A dashboard filter uses a field the metric view never declared — empty tile. Or the real ARR logic stays in the dashboard; Genie never inherits it.

**Essence.** The dashboard is a consumer. If logic stays in the dashboard, you still have two semantic layers.

#### I3. Genie

**Use case.** A Genie space built on `mv_arr`. “ARR by DACH partners this quarter” compiles to `MEASURE(arr)` with the certified joins.

**Issue case.** The same space also indexes raw SalesCloud tables. Genie prefers a column named `revenue`. Or the YAML is wrong and Genie is *confidently* wrong, citing Unity Catalog.

**Essence.** Grounding is not audit. Agents amplify whatever grain you certified.

#### I4. Alerts

**Use case.** Alert when `MEASURE(grr) < 0.96`. Ops notices a certified drop, not a dashboard that someone forgot to refresh.

**Issue case.** “GRR is fine because the alert did not fire” — while the measure is summed snapshots. The alert operationalizes a bad definition.

**Essence.** A threshold on a measure is not a substitute for a KPI contract.

#### I5. dbt

**Use case.** dbt builds `gold_subscription`, then `materialized='metric_view'` in the same project. A PR reviews YAML next to the model.

**Issue case.** Adapter docs lag reality; compiled DDL is not what the author expected. Tests written as if the metric view were a table do not mean what they mean on a table. Or Catalog Explorer edits bypass the PR.

**Essence.** dbt sits *on* Gold models. It does not replace them, and it is not yet the authority over the DDL.

#### I6. Passthrough native SQL

**Use case.** Power BI DirectQuery native query: `SELECT month, MEASURE(arr) FROM … GROUP BY month`. Definition stays in UC.

**Issue case.** Semantic-model developers will not maintain native SQL. The first Import-mode dataset strips the query. Fidelity is highest; adoption is lowest.

**Essence.** This is the official high-fidelity path, not the path most PBI estates will live on.

#### I7. Wrapper UC view

**Use case.** `v_arr_month_product` embeds `MEASURE(arr)` at that grain. Power BI sees an ordinary table.

**Issue case.** Someone groups the wrapper by week. Grain is frozen; you get double aggregation or a useless number. A new grain means a new wrapper. The “seventeen views” problem returns.

**Essence.** A wrapper re-locks aggregation — the thing metric views were invented to avoid.

#### I8. BI compatibility mode

**Use case.** Tableau Initial SQL sets the session flag; `SUM(arr)` is rewritten to the YAML. Live connection only.

**Issue case.** Gone from the Power BI connector. Remaining tools still inherit Beta lies (AVG, distincts, grand totals). A “compatibility” checkbox is treated as native support.

**Essence.** Session translation, not a Power BI model.

#### I9. Claimed live partner integrations

**Use case.** A Hex/Sigma-centric analytics team in *this* tenant queries metric views and it works. Verify, then use it.

**Issue case.** Architecture deck cites the April 2026 GA blog as proof Hex/Collibra/etc. are live *here*. Workshop demo fails. Vendor posture is not an estate fact.

**Essence.** Launch-partner lists are provenance. Check the tenant.

#### I10. Tableau delegated semantics

**Use case.** Late 2026, if it ships: Tableau relationships call UC measures. One definition, Tableau as the visual layer.

**Issue case.** H1 2026 design assumes Tableau already behaves like a Power BI star with delegated metrics. LODs still compile to joins ([K2](#k2-cannot-join-a-metric-view)); the date slips.

**Essence.** “Expected late 2026” is a vendor timeline, not a capability you can put in a current SOW.

#### I11. ThoughtSpot native metric views

**Use case.** Later in 2026, if GA in *this* tenant: search BI on the same UC measures as Genie.

**Issue case.** A Discovery recommendation “ThoughtSpot on metric views” based on the GA post, with no tenant proof and no plan for the gap months.

**Essence.** Same as Tableau: verify before you architect.

#### I12. Semantic-layer alternatives

**Use case.** Discovery, not this embed: MetricFlow or Cube if Snowflake (or a second warehouse) stays in parallel and metrics must travel.

**Issue case.** ARR in UC metric views *and* ARR in MetricFlow, independently edited. OSI is treated as a compiler that will make both correct. It is a definition-portability effort, not an audit.

**Essence.** UC metric views are the Databricks-depth play. Portability is a different product.

### Examples: Suitability

#### S1. Conformed star

**Use case.** One `fct_subscription`, conformed `Customer` / `Partner` / `Product`. Metric view is a thin certified consumption layer. This is the docs’ happy path.

**Issue case.** Gold is still KPI-by-KPI marts with overlapping CASE logic. A metric view is dropped on top and called a canonical model. The star was never built; the YAML cannot conjure it.

**Essence.** Strong **after** Gold is a star, not instead of one.

#### S2. SCD2 and point-in-time

**Use case.** Gold `dim_contract` is SCD2 (`valid_from` / `valid_to`). A parameter `as_of_date` plus a join predicate yields ARR as of a contract version.

**Issue case.** Contract End Date still disagrees across SalesCloud, CPQ, and M-Files. The metric view adds a window on Type-1 current rows and is sold as “history.” There is no history to window.

**Essence.** Point-in-time is a Gold feature. Metric views will not elect an authoritative date.

#### S3. Multi-grain facts

**Use case.** DBR 18.1+ and an explicit bridge: bookings, ARR, and invoices as sibling facts on shared dimensions. Valid grain combinations are enumerated.

**Issue case.** One metric view is asked to *be* the warehouse with no bridge, or the warehouse runtime is 18.0. Bookings × ARR Cartesian product, or a failed demo.

**Essence.** Fit is conditional on runtime **and** modeling work, not a checkbox.

#### S4. Non-additive KPIs

**Use case.** Logos = `COUNT(DISTINCT customer_id)` on atomic Gold. NRR is a ratio of composed measures. Query-time grain is the point.

**Issue case.** Daily distincts pre-aggregated in Gold and `SUM`med ([K11](#k11-grain-lives-in-the-table)); or a wrapper / Power BI grand total sums country logos into a fake global logo count ([K9](#k9-bi-compatibility-mode-is-beta)).

**Essence.** Safe on raw grain. Unsafe the moment someone re-aggregates a non-additive.

#### S5. Row, column security, and ABAC

**Use case.** Query-time RLS: partner-filtered ARR for a channel manager. Correct and governed.

**Issue case.** The same object is materialized “for the board pack” and promised to stay partner-filtered. Databricks will not (must not) pre-compute as the owner and then pretend it is per-user.

**Essence.** Fast materialized ARR and per-user row filters are different objects.

#### S6. Delta Sharing

**Use case.** Share Gold subscription facts to a partner or another cloud. They build what they need.

**Issue case.** “They will consume our metric view.” They cannot. The semantic object does not travel with the share.

**Essence.** Poor for the YAML object; fine for the tables underneath.

#### S7. Power BI as system of engagement

**Use case.** Explicit dual path: Databricks holds the certified measure; Power BI either uses native SQL ([I6](#i6-passthrough-native-sql)), a frozen wrapper ([I7](#i7-wrapper-uc-view)), or DAX that is tested not to diverge.

**Issue case.** The metric view is treated as the Power BI dataset. Import/`SELECT *` fails ([K1](#k1-select-star-and-measure)); compatibility mode is gone from the connector ([I8](#i8-bi-compatibility-mode)). Analysts live in Desktop; the “single source of truth” is a slide.

**Essence.** Weak until a native connector exists in *this* tenant. Plan the workaround; do not assume it.

#### S8. Genie and AI BI as system of engagement

**Use case.** After the KPI contract exists, a certified Genie space on the metric view is the best-integrated consumer ([C10](#c10-native-databricks-consumers), [I3](#i3-genie)).

**Issue case.** Genie is stood up in week 1 to “explore ARR” before grain, Contract End Date, and partner identity are agreed. The agent makes a fiction consistent.

**Essence.** Strong consumer, amplifier of whatever you certified — including a wrong grain.

#### S9. Code-reviewed metric lifecycle

**Use case.** YAML in Git, PR, dbt `metric_view` materialization. Prod matches main.

**Issue case.** Someone edits in Catalog Explorer. No PR. Month-end ARR in prod is not the YAML in the repo. Two sources of truth, both “in Unity Catalog.”

**Essence.** Emerging: Git/dbt path exists; the UI path does not care. Not a mature metric-ops product.

#### S10. Cross-warehouse semantic portability

**Use case.** Do not use UC metric views for this job. If Snowflake remains, Discovery evaluates MetricFlow/OSI ([I12](#i12-semantic-layer-alternatives)).

**Issue case.** Expectation that ARR YAML will round-trip to Snowflake unchanged, or that OSI will prove the number correct. Depth in Databricks is the point; portability is someone else’s layer.

**Essence.** Not the job. Do not score metric views against it.

---



## Sources

**Official — Azure Databricks (status authority)**

- [Unity Catalog metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/)
- [Create a metric view](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/create)
- [Query metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/query)
- [Model metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/basic-modeling)
- [Joins in metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/joins)
- [Advanced techniques (windows, composability)](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/advanced-techniques)
- [Materialization for metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/materialization)
- [Manage metric views](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/manage)
- [Feature availability](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/feature-availability)
- [Use metric views with external BI tools](https://learn.microsoft.com/en-us/azure/databricks/uc-semantics/metric-views/bi-tools)
- [BI compatibility mode (Beta)](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/bi-metric-view)
- [Power BI with Databricks](https://learn.microsoft.com/en-us/azure/databricks/partners/bi/power-bi/)

**Official — Databricks product (provenance)**

- [Announcing GA and open-sourcing of Unity Catalog Business Semantics](https://www.databricks.com/blog/redefining-semantics-data-layer-future-bi-and-ai) (Apr 2026; Preview UI/materialization; partner roadmap; SPARK-54119; OSI)

**Independent / practitioner**

- [What Are Metrics in Unity Catalog — Typedef](https://www.typedef.ai/blog/what-are-metrics-in-unity-catalog-databricks-governed-metric-layer-explained) (22 Jun 2026; dbt adapter 1.12.0; additivity failures; MetricFlow vs UC)
- [Databricks Metric Views — Power BI BI compatibility mode removal — Community](https://community.databricks.com/t5/community-articles/databricks-metric-views-power-bi-bi-compatibility-mode-removal/td-p/154977)
- [Metric Views with Power BI and Tabular Editor (part 3) — Community](https://community.databricks.com/t5/community-articles/metric-views-with-power-bi-and-tabular-editor-part-3-of-3/td-p/160040)
- [Yasiru Randika — GA vs Preview split](https://www.linkedin.com/posts/yasirurandika_databricks-unitycatalog-dataengineering-activity-7482193124879126528-ba9Y) (practitioner summary; corroborates docs, not a primary source)

