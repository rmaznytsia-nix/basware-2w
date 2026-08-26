# Databricks on Azure — complete architecture materials and fast catch-up guide

**Target audience:** Data Architect refreshing Azure Databricks platform knowledge.  
**Goal:** Establish current, practical judgement across governance, engineering, operations, delivery automation and migration modernisation.

> This is the complete registry of materials identified in the earlier browser, Partner Portal and GitHub review. Items are retained even where they are older, cloud-generic, third-party, or lower priority; use the relevance column to decide reading order.

## Contents

- [Start here](#start-here)
- [Catch up quickly and effectively](#catch-up-quickly-and-effectively)
- [1. Official platform and Azure guidance](#1-official-platform-and-azure-guidance)
- [2. Databricks Partner Portal delivery materials](#2-databricks-partner-portal-delivery-materials)
- [3. Databricks GitHub reference implementations](#3-databricks-github-reference-implementations)
- [4. Additional public materials previously identified](#4-additional-public-materials-previously-identified)
- [Practical architecture recommendations](#practical-architecture-recommendations)
- [Databricks Azure Best Practices](#databricks-azure-best-practices)
  - [Reliability](#reliability)
  - [Security](#security)
  - [Cost Optimization](#cost-optimization)
  - [Operational Excellence](#operational-excellence)
  - [Performance Efficiency](#performance-efficiency)
- [Fast-reference source map](#fast-reference-source-map)

---

## Start here

Read these four first. They provide the highest-value current baseline before diving into product details or example code.

1. [Unity Catalog best practices — Azure](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/best-practices) — governance, identity, storage and compute guardrails.
2. [Azure Databricks Well-Architected guide](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-databricks) — Azure-specific architecture review lens.
3. [terraform-databricks-examples](https://github.com/databricks/terraform-databricks-examples) — executable Azure patterns for platform controls.
4. [bundle-examples](https://github.com/databricks/bundle-examples) — Git-based, repeatable deployment patterns for jobs and pipelines.



## Catch up quickly and effectively

1. **Map the current platform vocabulary.** Focus on Unity Catalog, Lakeflow, serverless compute, Declarative Automation Bundles, AI/BI, and the Azure control plane. Treat older terms such as DLT and Asset Bundles as entry points, then reconcile them with current documentation.
2. **Establish architecture guardrails before reviewing code.** Use the Well-Architected and Unity Catalog material to anchor identity, network isolation, storage, governance, auditability, resilience and cost discussions.
3. **Use examples as an architecture microscope.** Review the Terraform Azure examples for Private Link, VNet injection, Unity Catalog grants and data-exfiltration protection. Compare each pattern with the estate's current implementation.
4. **Learn the current delivery model.** Use Bundle examples to understand environment promotion, source control, deployment of jobs/pipelines, and ownership of production assets.
5. **Use partner kits as practical review criteria.** The Unity Catalog Migration Kit and FDE assurance assets are useful checklists when standardising a platform, upgrading from Hive Metastore, or strengthening an enterprise governance model.
6. **Use AI assistance after understanding the intent.** Agent Skills can reinforce platform practices in AI-assisted development, but human review remains essential for identity, permissions, networking, data classification, production promotion and cost decisions.



## 1. Official platform and Azure guidance

These are the architecture authority. AWS-labelled Databricks pages are included only where the principle is platform-neutral; translate implementation specifics to Azure services and controls.


| Material                                                                                                                                  | Quick annotation                                                                                                   | Freshness | Relevance |
| ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | --------- | --------- |
| [Azure Databricks Well-Architected guide](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-databricks)       | Azure-specific lens for reliability, security, cost, operations, performance, Azure Policy and Advisor.            | Feb 2026  | Very high |
| [Unity Catalog best practices — Azure](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/best-practices)   | Metastore and identity design; catalogs/schemas; managed storage; external locations; sharing; compute guardrails. | Jul 2026  | Very high |
| [Data engineering best practices — Azure](https://learn.microsoft.com/en-us/azure/databricks/data-engineering/best-practices)             | Starting point for Lakeflow, modelling, joins, streaming state, data quality and observability.                    | Current   | Very high |
| [Data and AI governance](https://docs.databricks.com/aws/en/lakehouse-architecture/data-governance/best-practices)                        | Operating model, lineage, audit logging, metadata quality and enterprise standards.                                | Jul 2026  | Very high |
| [Reliability best practices](https://docs.databricks.com/aws/en/lakehouse-architecture/reliability/best-practices)                        | Delta ACID, expectations, retries, streaming recovery, autoscaling, disaster recovery and metastore recovery.      | Jul 2026  | Very high |
| [Operational excellence](https://docs.databricks.com/aws/en/lakehouse-architecture/operational-excellence/best-practices)                 | Environment isolation, catalog strategy, standardised compute, automated jobs and monitoring.                      | Jul 2026  | Very high |
| [Developer best practices](https://docs.databricks.com/aws/en/developers/best-practices)                                                  | Version control, environment management, developer tooling, CI/CD and managed deployment.                          | Jul 2026  | High      |
| [Notebook engineering best practices](https://docs.databricks.com/aws/en/notebooks/best-practices)                                        | Production notebook expectations; particularly useful for notebook-heavy estates.                                  | Jun 2026  | High      |
| [Interoperability and usability](https://docs.databricks.com/aws/en/lakehouse-architecture/interoperability-and-usability/best-practices) | Open formats, integration, secure sharing, self-service and trusted data products.                                 | Aug 2026  | High      |
| [Classic compute configuration](https://docs.databricks.com/aws/en/compute/cluster-config-best-practices)                                 | Baseline for evaluating classic clusters before identifying appropriate serverless candidates.                     | Current   | High      |




## 2. Databricks Partner Portal delivery materials

These translate product guidance into assessment, migration and delivery assets. Use them in accordance with the Databricks partner agreement and stated restrictions.


| Material                                                                                                              | Quick annotation                                                                                                                       | Freshness                | Relevance |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | --------- |
| [Unity Catalog Migration Service Delivery Kit](https://partners.databricks.com/s/unity-catalog-migration-sdk-revised) | Early assessment, upgrade planning, migration checklists, UCX, implementation plan, troubleshooting and closeout templates.            | Sep 2025 release notes   | Very high |
| [Data Governance Playbook](https://partners.databricks.com/s/data-governance-playbook)                                | Field patterns for Unity Catalog design, permission modelling, security, Delta Sharing and Clean Rooms.                                | Current portal asset     | Very high |
| [FDE Assurance Offerings](https://partners.databricks.com/s/fde-assurance-offerings)                                  | Well-Architected Lakehouse, migration and Unity Catalog assurance; a useful quality bar for architecture review.                       | Current portal asset     | Very high |
| [FDE Service Delivery Kits](https://partners.databricks.com/s/FDE-SDK)                                                | Delivery kits for Azure lakehouse migration, Unity Catalog, governance, observability, lakehouse build, warehouse migration and MLOps. | Current portal catalogue | Very high |
| [Migrate & Modernize Program](https://partners.databricks.com/s/migrate-modernize-program)                            | Modernisation framing for source activation, ETL/code translation, BI modernisation and Azure Synapse migration.                       | FY27 / 2026              | Very high |
| [Lakebridge](https://partners.databricks.com/s/lakebridge)                                                            | Migration-scoping method: analyse source estates, evaluate conversion effort and identify assurance needs.                             | Current                  | High      |
| [Technical Playbooks](https://partners.databricks.com/s/technical-playbooks)                                          | Data warehouse implementation practices; relevant to DBSQL, EDW and semantic/reporting workloads.                                      | Current portal asset     | High      |
| [Azure Databricks and Microsoft Fabric](https://partners.databricks.com/s/azure-databricks-and-msft-fabric)           | Learning material for estates where Fabric coexists with Azure Databricks.                                                             | Undated                  | High      |
| [Developing your Center of Excellence](https://partners.databricks.com/s/developing-your-center-of-excellence)        | CoE maturity model, rubric, self-assessment and playbook for operating model and adoption capability.                                  | Current portal asset     | High      |




## 3. Databricks GitHub reference implementations

Use these as learning assets and implementation patterns. Prefer maintained repositories; use older blueprints to form review questions and validate them against current Azure and Databricks documentation.


| Repository                                                                                                           | Quick annotation                                                                                                                                                       | Freshness        | Relevance |
| -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | --------- |
| [terraform-databricks-examples](https://github.com/databricks/terraform-databricks-examples)                         | Best Azure executable reference: lakehouse, Private Link, VNet injection, data-exfiltration protection, Unity Catalog/Entra sync, grants, Overwatch and CI/CD.         | Aug 2026         | Very high |
| [bundle-examples](https://github.com/databricks/bundle-examples)                                                     | Declarative Automation Bundles examples for jobs, SQL, dbt, Lakeflow, MLOps and knowledge bases; strong Git-based multi-environment deployment reference.              | Aug 2026         | Very high |
| [databricks-agent-skills](https://github.com/databricks/databricks-agent-skills)                                     | Current platform guidance encoded for coding agents: core platform, data discovery, Bundles, Jobs, Lakeflow, serverless migration, model serving and AI/BI.            | Active / 19 tags | Very high |
| [terraform-databricks-lakehouse-blueprints](https://github.com/databricks/terraform-databricks-lakehouse-blueprints) | Composable Azure architecture: hub-and-spoke, VNet injection, secure connectivity and Unity Catalog modules. Use as a comparison blueprint, not a copy/paste template. | Jan 2024         | High      |
| [delta-live-tables-notebooks](https://github.com/databricks/delta-live-tables-notebooks)                             | Runnable Lakeflow / former DLT patterns for CDC, snapshots, quality, metadata-driven pipelines and serverless benchmarks.                                              | Sep 2025         | High      |
| [notebook-best-practices](https://github.com/databricks/notebook-best-practices)                                     | Hands-on patterns for Repos, shared code, tests, scheduled Jobs and CI/CD around notebook workloads.                                                                   | Jul 2024         | High      |
| [terraform-provider-databricks](https://github.com/databricks/terraform-provider-databricks)                         | Authoritative resource semantics for deciding what should be automated and governed as code.                                                                           | Aug 2026         | High      |
| [dbt-databricks](https://github.com/databricks/dbt-databricks)                                                       | Useful when dbt is present; supports assessment of transformation, SQL engineering and CI/CD conventions.                                                              | Aug 2026         | Medium    |




## 4. Additional public materials previously identified

These sources were identified in the initial browser review. They are retained for coverage, but are generally supporting references rather than the starting point for architecture decisions.


| Material                                                                                                                                                                                        | Quick annotation                                                                                                                     | Freshness             | Relevance |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | --------------------- | --------- |
| [Security, compliance and privacy best practices](https://docs.databricks.com/aws/en/lakehouse-architecture/security-compliance-and-privacy/best-practices)                                     | Platform guidance covering security controls, data protection and compliance considerations; map cloud-specific controls to Azure.   | Current docs page     | High      |
| [Azure Databricks getting-started best practices](https://learn.microsoft.com/en-us/azure/databricks/getting-started/best-practices)                                                            | Microsoft Learn navigation page linking to best-practice topics by capability.                                                       | Current               | High      |
| [Unity Catalog best practices — Databricks on AWS](https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices)                                                             | Cloud-neutral Unity Catalog counterpart to the Azure page; useful when following cross-cloud links in the docs.                      | Current docs page     | High      |
| [Data engineering best practices — Databricks on AWS](https://docs.databricks.com/aws/en/data-engineering/best-practices)                                                                       | Cross-cloud counterpart to the Azure data-engineering page; useful for platform-neutral patterns.                                    | Current docs page     | High      |
| [Data Pipeline Best Practices](https://www.databricks.com/blog/data-pipeline-best-practices)                                                                                                    | Databricks blog material on pipeline architecture, modern pipeline patterns and deployment.                                          | Current blog post     | High      |
| [Secure your Data + AI Platform using best practices](https://www.databricks.com/trust/security-features/best-practices)                                                                        | Databricks security posture and feature overview; helpful framing for security discussions.                                          | Current security page | High      |
| [Databricks Best Practices blog category](https://www.databricks.com/blog/category/data-strategy/best-practices?categories=best-practices)                                                      | Live index of best-practice articles. Useful for topic exploration after the core architecture sources.                              | Live index            | Medium    |
| [Best Practices for Databricks Notebooks](https://www.databricks.com/blog/2022/06/25/software-engineering-best-practices-with-databricks-notebooks.html)                                        | Databricks blog article on applying software engineering discipline to notebooks.                                                    | Jun 2022              | Medium    |
| [5 Best Practices for Databricks Workspaces](https://www.databricks.com/blog/2022/03/10/functional-workspace-organization-on-databricks.html)                                                   | Functional workspace organisation patterns. Useful historical context; reconcile with Unity Catalog and current deployment guidance. | Mar 2022              | Medium    |
| [Apache Spark overview](https://docs.databricks.com/aws/en/spark/)                                                                                                                              | Foundation reference for Spark concepts and runtime behaviour relevant to workload assessment.                                       | Current docs page     | Medium    |
| [Databricks Certified Data Engineer Professional exam guide](https://www.databricks.com/sites/default/files/2026-07/databricks-certified-data-engineer-professional-exam-guide-july-3-2026.pdf) | Current competency outline; useful as a structured checklist of engineering topics to refresh.                                       | Jul 2026              | Medium    |
| [Databricks certification — Data Engineer](https://www.databricks.com/learn/training/certification#data-engineer)                                                                               | Certification catalogue and learning route for data-engineering role alignment.                                                      | Current               | Medium    |
| [Build agents on Databricks](https://docs.databricks.com/aws/en/agents/)                                                                                                                        | Official agent-development documentation; relevant when AI-agent workloads are in scope.                                             | Current docs page     | Medium    |
| [Databricks AI Security Framework (DASF)](https://www.databricks.com/blog/introducing-databricks-ai-security-framework-dasf)                                                                    | Security framework for AI workloads; use when governance includes agentic or GenAI capabilities.                                     | Current blog post     | Medium    |
| [Databricks Architecture Best Practices — Dateonic](https://dateonic.com/databricks-architecture-best-practices/)                                                                               | Third-party architecture overview. Use as a comparison perspective, not as an authority.                                             | Undated               | Low       |
| [11 Common Databricks Mistakes Beginners Make](https://blog.devgenius.io/11-common-databricks-mistakes-beginners-make-best-practices-for-data-management-and-coding-e3c843bad2b0)               | Practitioner checklist of data-management and coding pitfalls. Useful as a lightweight review prompt.                                | Undated               | Low       |
| [Databricks Best Practices: Revamped](https://medium.com/@matt_weingarten/databricks-best-practices-revamped-b039f61c2267)                                                                      | Practitioner perspective on platform best practices. Validate all product-specific advice against current documentation.             | Undated               | Low       |
| [Databricks: Best Practices — Performance Optimization, Unity Catalog and more](https://rihab-feki.medium.com/databricks-best-practices-28771ccf8f3f)                                           | Practitioner article centred on performance and Unity Catalog. Treat as supplemental and validate current guidance.                  | Undated               | Low       |




## Practical architecture recommendations

- Create one intentional Unity Catalog design: clear metastore ownership, catalog/schema conventions, least-privilege grants and governed external locations.
- Manage platform controls as code where practical: account/workspace configuration, identity, Unity Catalog grants, jobs and environment promotion.
- Keep experimentation and production distinct: exploration can remain flexible, but production logic should be versioned, testable, observable and deployable.
- Define a compute posture: assess classic cluster configurations, then identify serverless candidates and apply policies/templates for remaining classic workloads.
- Demand operational evidence: audit logs, lineage, data-quality checks, job recovery, cost attribution and resilience testing should be demonstrable.



## Databricks Azure Best Practices

Source: [Architecture best practices for Azure Databricks](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-databricks) (Microsoft Azure Well-Architected Framework)

### Reliability

- [Reliability design principles](https://learn.microsoft.com/en-us/azure/well-architected/resiliency/principles) — foundational resilience guidance
- [Design review checklist for Reliability](https://learn.microsoft.com/en-us/azure/well-architected/reliability/checklist) — starting checklist for the pillar
- [Service limits](https://learn.microsoft.com/en-us/azure/databricks/resources/) — workspace/compute/network quota constraints
- [FMA](https://learn.microsoft.com/en-us/azure/well-architected/reliability/failure-mode-analysis) — failure mode analysis method
- [Comprehensive monitoring](https://learn.microsoft.com/en-us/azure/well-architected/reliability/monitoring-alerting-strategy) — health monitoring/alerting strategy
- [DR](https://learn.microsoft.com/en-us/azure/well-architected/reliability/disaster-recovery) — disaster recovery planning
- [Autoscaling](https://learn.microsoft.com/en-us/azure/databricks/compute/configure#autoscaling) — dynamic cluster node scaling
- [Multiregion DR setup](https://learn.microsoft.com/en-us/azure/databricks/admin/disaster-recovery) — cross-region deployment for mission-critical workloads
- [Cluster pools](https://learn.microsoft.com/en-us/azure/databricks/compute/pool-index) — prewarmed instances to cut startup time
- [Time travel](https://learn.microsoft.com/en-us/azure/databricks/delta/history) — Delta Lake point-in-time recovery
- [Diagnostic logs](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/audit-logs) — centralized monitoring via Azure Monitor
- [Serverless SQL warehouses](https://learn.microsoft.com/en-us/azure/databricks/compute/sql-warehouse/warehouse-types#serverless-sql-warehouses) — always-available analytics compute
- [Retry policies](https://learn.microsoft.com/en-us/azure/databricks/jobs/configure-task#retries) — job-level automatic retries
- [Virtual network injection](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/vnet-inject) — custom routing/redundancy at network layer
- [Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/) — governance with backup/metadata sync
- [Lakeflow Spark Declarative Pipelines](https://learn.microsoft.com/en-us/azure/databricks/ldp/) — fault-tolerant declarative pipelines
- [Azure REST APIs](https://docs.databricks.com/api/azure/workspace/workspace) — automated workspace backup
- [Structured streaming](https://learn.microsoft.com/en-us/azure/databricks/structured-streaming/concepts) — checkpointing for exactly-once processing
- [Automatic cluster restart](https://learn.microsoft.com/en-us/azure/databricks/compute/configure) — restart policies for continuity
- [Instance pools](https://learn.microsoft.com/en-us/azure/databricks/compute/pools) — VM diversity for provisioning resilience



### Security

- [Security design principles](https://learn.microsoft.com/en-us/azure/well-architected/security/security-principles)
- [Design review checklist for Security](https://learn.microsoft.com/en-us/azure/well-architected/security/checklist)
- [Azure Databricks security baseline](https://learn.microsoft.com/en-us/security/benchmark/azure/baselines/azure-databricks-security-baseline) — CIS-style security benchmark
- [Identity and access management](https://learn.microsoft.com/en-us/azure/well-architected/security/identity-access) — control/data plane auth model
- [Virtual network injection](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/vnet-inject) — network isolation for workspaces
- [Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/databricks/security/auth/) — SSO/MFA integration
- [Conditional access policies](https://learn.microsoft.com/en-us/azure/databricks/archive/azure-admin/conditional-access) — access based on risk signals
- [Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/) — centralized data governance
- [Customer-managed keys](https://learn.microsoft.com/en-us/azure/databricks/security/keys/customer-managed-keys) — encryption key control via Key Vault
- [Key Vault-backed secret scopes](https://learn.microsoft.com/en-us/azure/databricks/security/secrets) — centralized credential management
- [IP access lists](https://learn.microsoft.com/en-us/azure/databricks/security/network/front-end/ip-access-list) — allow/deny by network origin
- [Secure cluster connectivity](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/secure-cluster-connectivity) — no public IPs on clusters
- [Private Link endpoints](https://learn.microsoft.com/en-us/azure/databricks/security/network/front-end/front-end-private-connect) — private control-plane access
- [Enhanced security and compliance](https://learn.microsoft.com/en-us/azure/databricks/security/privacy/enhanced-security-compliance) — HIPAA/regulated workload features
- [Audit logging](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/audit-logs) — track user/system activity
- [OAuth 2.0 machine-to-machine authentication](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/auth/oauth-m2m) — replaces personal access tokens
- [SAT](https://github.com/databricks-industry-solutions/security-analysis-tool) — Databricks Security Analysis Tool for config assessment
- [Service principal authentication](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/auth/) — non-human identity for automation
- [Microsoft Entra ID credential passthrough](https://learn.microsoft.com/en-us/azure/databricks/security/credential-passthrough/adls-passthrough) — ADLS access without service principals
- [Cluster hardening](https://learn.microsoft.com/en-us/azure/databricks/security/network/classic/secure-cluster-connectivity) — SSH restriction/runtime controls
- [CI/CD pipeline integration](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/ci-cd/) — automated security scanning in pipelines



### Cost Optimization

- [Cost Optimization design principles](https://learn.microsoft.com/en-us/azure/well-architected/cost-optimization/principles)
- [Design review checklist for Cost Optimization](https://learn.microsoft.com/en-us/azure/well-architected/cost-optimization/checklist)
- [Job clusters](https://learn.microsoft.com/en-us/azure/databricks/jobs/compute#how-do-i-configure-compute-for-jobs) — avoid idle all-purpose cluster costs
- [Cluster autoscaling](https://learn.microsoft.com/en-us/azure/databricks/compute/configure#autoscaling) — match nodes to demand
- [Automatic termination](https://learn.microsoft.com/en-us/azure/databricks/compute/configure#automatic-termination) — shut down idle interactive clusters
- [Serverless SQL warehouses](https://learn.microsoft.com/en-us/azure/databricks/compute/sql-warehouse/) — consumption-based SQL compute
- [Cluster pools](https://learn.microsoft.com/en-us/azure/databricks/compute/pool-best-practices) — faster starts, no idle DBU charge
- [Delta Lake optimization features](https://learn.microsoft.com/en-us/azure/databricks/optimizations/) — OPTIMIZE/Z-ORDER/VACUUM for storage efficiency
- [Compute policies](https://learn.microsoft.com/en-us/azure/databricks/admin/clusters/policies) — restrict instance types/sizes
- [Databricks system tables](https://learn.microsoft.com/en-us/azure/databricks/admin/usage/system-tables) — DBU/usage tracking
- [Microsoft Cost Management integration](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/quick-acm-cost-analysis) — spend visibility
- [Databricks reserved capacity](https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/prepay-databricks-reserved-capacity) — prepaid DBCU discounts
- [Compute types](https://learn.microsoft.com/en-us/azure/databricks/compute/) — workload-specific compute selection
- [VACUUM commands](https://learn.microsoft.com/en-us/azure/databricks/delta/vacuum) — automated data lifecycle cleanup
- [Standard tier](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/account) — lower-cost tier for non-prod
- [Serverless jobs](https://learn.microsoft.com/en-us/azure/databricks/jobs/run-serverless-jobs) — for variable/intermittent workloads
- [Databricks usage monitoring](https://learn.microsoft.com/en-us/azure/databricks/admin/usage/) — proactive cost alerts
- [Photon acceleration](https://learn.microsoft.com/en-us/azure/databricks/compute/photon) — faster compute reduces cost
- [Lakeflow Connect Free Tier](https://aka.ms/adblakeflowfree) — free daily DBUs for SaaS ingestion



### Operational Excellence

- [Operational Excellence design principles](https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/principles)
- [Design review checklist for Operational Excellence](https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/checklist)
- [Databricks Asset Bundles](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/bundles/) — IaC-style deployment/version control
- [Operational runbooks](https://learn.microsoft.com/en-us/azure/databricks/lakehouse-architecture/operational-excellence/) — step-by-step incident guidance
- [Backup and recovery procedures](https://learn.microsoft.com/en-us/azure/well-architected/reliability/disaster-recovery) — protecting configs/code/data
- [Diagnostic settings](https://learn.microsoft.com/en-us/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs) — send logs to Log Analytics
- [ARM templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/) — consistent workspace deployment
- [Git repositories](https://learn.microsoft.com/en-us/azure/databricks/repos/) — Databricks Repos source control
- [Cluster rightsizing](https://learn.microsoft.com/en-us/azure/databricks/compute/cluster-config-best-practices) — automated sizing from metrics
- [Unity Catalog audit logging](https://learn.microsoft.com/en-us/azure/databricks/admin/account-settings/audit-logs#uc) — governance activity trail
- [Lakeflow Spark Declarative Pipelines (expectations)](https://learn.microsoft.com/en-us/azure/databricks/ldp/expectations) — automated data quality rules
- [Databricks REST API](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/api/latest/workspace) — workspace artifact backup automation
- [Workspace folder hierarchies](https://learn.microsoft.com/en-us/azure/databricks/workspace/workspace-assets) — naming/organization conventions
- [Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/) — tagging strategy for spend tracking
- [Service principal authentication](https://learn.microsoft.com/en-us/azure/databricks/security/auth/) — automated integration auth
- [Cluster life cycle policies](https://learn.microsoft.com/en-us/azure/databricks/administration-guide/clusters/policies) — auto-termination/idle timeout rules
- [Azure Monitor alert rules](https://learn.microsoft.com/en-us/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs) — proactive incident notification
- [RBAC](https://learn.microsoft.com/en-us/azure/databricks/security/auth/access-control) — environment separation/access control



### Performance Efficiency

- [Performance Efficiency design principles](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/principles)
- [Design review checklist for Performance Efficiency](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/checklist)
- [Architecture strategies for selecting the right services](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/select-services) — matching services to workload
- [Comprehensive performance monitoring](https://learn.microsoft.com/en-us/azure/well-architected/performance-efficiency/collect-performance-data) — bottleneck identification
- [Memory-optimized instance types](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview#memory-optimized) — E/M-series VMs for large datasets
- [Cluster autoscaling policies](https://learn.microsoft.com/en-us/azure/databricks/compute/configure#autoscaling) — performance under variable demand
- [OPTIMIZE commands with Z-ordering](https://learn.microsoft.com/en-us/azure/databricks/delta/tutorial#z-order) — faster data skipping on queries
- [Delta Cache](https://learn.microsoft.com/en-us/azure/databricks/optimizations/disk-cache) — SSD caching for repeated access
- [Photon engine](https://learn.microsoft.com/en-us/azure/databricks/compute/photon) — vectorized SQL/DataFrame execution
- [Spark executor memory settings](https://learn.microsoft.com/en-us/azure/databricks/spark/conf) — tuning executor/driver memory
- [Data partitioning strategies](https://learn.microsoft.com/en-us/azure/databricks/optimizations/) — partition pruning for scan reduction
- [Parquet file format](https://learn.microsoft.com/en-us/azure/databricks/optimizations/) — columnar storage with compression
- [Serverless SQL warehouses](https://learn.microsoft.com/en-us/azure/databricks/compute/sql-warehouse/) — instant scaling BI compute
- [AQE](https://learn.microsoft.com/en-us/azure/databricks/optimizations/aqe) — adaptive query execution optimizations
- [Cluster pools](https://learn.microsoft.com/en-us/azure/databricks/compute/pool-index) — reduce startup latency
- [Optimization tasks](https://learn.microsoft.com/en-us/azure/databricks/optimizations/) — scheduled file compaction/VACUUM
- [Structured streaming trigger intervals](https://learn.microsoft.com/en-us/azure/databricks/structured-streaming/concepts) — latency vs. throughput tuning
- [GPU-enabled clusters](https://learn.microsoft.com/en-us/azure/databricks/compute/configure) — NC/ND/NV VMs for ML workloads



## Fast-reference source map


| Need                               | Best starting source                                                           |
| ---------------------------------- | ------------------------------------------------------------------------------ |
| Target Azure architecture          | Azure Well-Architected guide + `terraform-databricks-examples`                 |
| Unity Catalog / governance         | Unity Catalog best practices + Data Governance Playbook + UC Migration Kit     |
| Engineering and pipeline practices | Azure data engineering best practices + `bundle-examples` + Lakeflow notebooks |
| Legacy notebook remediation        | Notebook engineering best practices + Developer best practices                 |
| Migration readiness                | Migrate & Modernize Program + Lakebridge + FDE Service Delivery Kits           |
| Operating model and adoption       | Operational excellence + Developing your Center of Excellence                  |


