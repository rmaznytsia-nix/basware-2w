# macOS local data-quality and source-mapping lab

**Purpose.** A reproducible, privacy-conscious lab for rapid source discovery, source-to-target mapping, profiling, data-quality analysis, entity resolution, and conflict triage in a brownfield Databricks engagement.

This is intentionally **not** a local substitute for the client's Databricks environment. Use the laptop for small, approved extracts and repeatable analysis; use Databricks Connect and a matching remote cluster for Spark-scale validation. Keep the dependency families below isolated.

> Verified 24 August 2026. Check the linked compatibility documentation before changing Spark, Java, or Databricks Runtime versions.

## 1. What to install, and why

| Capability | Primary tool | Role in this lab | When to use it |
| --- | --- | --- | --- |
| Fast local SQL, files, schema comparisons | DuckDB | Query CSV, Parquet, JSON and Excel-derived files without a server; persist analysis results locally | First pass on extracts; source/target comparisons |
| Exploratory profiling | ydata-profiling | HTML report for types, nulls, distributions, correlations and duplicates | Quick orientation of a small or sampled table |
| Drift and test-style data checks | Evidently | Compare a source, target, or two time periods; track drift and data-quality checks | Baselines and recurring KPI-input checks |
| Scalable constraints and constraint suggestions | Deequ | Spark metrics, suggested constraints, and executable quality rules | Spark-compatible Databricks or a dedicated local Spark sandbox |
| Probabilistic record linkage | Splink | Transparent, reviewable candidate links using DuckDB locally | Discover likely system/entity matches and produce a review queue |
| ML-assisted entity resolution | Zingg | Train and run deduplication/entity-resolution models in Spark | Larger labelled matching exercises; promote to Databricks after validation |
| Schema and contract checks | Pandera + Great Expectations | DataFrame schema assertions and reusable data expectations | Guardrails around mapping transformations |
| Transformation / mapping delivery | dbt-duckdb + SQLMesh (optional) | Version-controlled mapping SQL, tests and generated documentation | Prototyping canonical models before porting to Databricks SQL/dbt |
| Source connectivity | SQLAlchemy, database-specific drivers, Databricks SDK/SQL connector | Read from approved sources and Databricks SQL warehouses | Install only the connectors required for the engagement |
| Metadata / mapping register | DuckDB tables + Markdown/CSV (optionally OpenMetadata in Docker) | A portable inventory, crosswalk, rule catalogue and conflict log | Start lightweight; introduce a catalog only if it will be governed |

**Recommended starting set:** DuckDB, ydata-profiling, Evidently, Splink, Pandera, Great Expectations, dbt-duckdb, JupyterLab, VS Code, Docker Desktop, Databricks CLI/IDE extension. Add Deequ and Zingg in their dedicated Spark environments once the client runtime is known.

## 2. Mac prerequisites

### 2.1 Install base tools

Install [Homebrew](https://brew.sh/) first if it is not already present, then run:

```zsh
brew update
brew install git uv duckdb jq openjdk@8 openjdk@11
brew install --cask visual-studio-code docker
brew install databricks/tap/databricks
```

Open Docker Desktop once and complete its onboarding; it provides an isolated route for Zingg and, later, optional services such as PostgreSQL or OpenMetadata. Do not install it if client policy prohibits container runtimes.

Install these VS Code extensions from the Marketplace:

- Python, Jupyter, SQLTools (or a preferred SQL client), YAML, GitLens.
- **Databricks**, from the Databricks-verified publisher. It can sync a local project and run/debug code against workspace compute. [Databricks IDE extension guidance](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/) applies.

Validate the base installation:

```zsh
uv --version
duckdb --version
java -version
databricks version
docker version
```

### 2.2 Create a safe project structure

Create one repository per engagement. These folders keep raw extracts, generated results, and curated evidence distinct.

```zsh
mkdir -p kpi-data-lab/{data/{incoming,staged,curated},notebooks,sql,src,tests,artifacts/{profiles,drift,links},metadata,docs}
cd kpi-data-lab
git init
printf '.env\ndata/incoming/\ndata/staged/\nartifacts/\n*.duckdb\n.ipynb_checkpoints/\n' > .gitignore
```

Commit only synthetic data and reusable code. Treat `data/incoming/`, report HTML, matching outputs, and a DuckDB database as potentially confidential. Encrypt the Mac, use approved storage, and remove local extracts at the end of the engagement in accordance with the client's retention policy.

## 3. The everyday DuckDB/Python workbench

Use Python 3.11 initially; it has broad library support. `uv` creates a local, reproducible virtual environment and lockfile.

```zsh
uv init --python 3.11
uv add pandas polars pyarrow duckdb sqlalchemy openpyxl python-dotenv \
  jupyterlab ipywidgets plotly "ydata-profiling[notebook,unicode]" \
  evidently splink pandera great-expectations dbt-duckdb \
  databricks-sdk databricks-sql-connector
uv run jupyter lab
```

If ydata-profiling has moved package names or requires a different Python version when you run this, follow its current [installation and migration guidance](https://docs.profiling.ydata.ai/latest/getting-started/installation/) and pin the resolved package in `pyproject.toml`. The `pyspark` ydata extra is deliberately excluded here: it belongs in an isolated Spark environment.

Quickly interrogate a data extract without copying it into a database:

```zsh
duckdb
```

```sql
CREATE OR REPLACE TABLE source_orders AS
SELECT * FROM read_parquet('data/incoming/orders/*.parquet');

DESCRIBE source_orders;
SUMMARIZE source_orders;
SELECT COUNT(*) AS rows, COUNT(DISTINCT customer_id) AS customer_ids
FROM source_orders;
```

For Excel, DuckDB can install its `excel` extension on demand. Only allow extension downloads from approved networks:

```sql
INSTALL excel;
LOAD excel;
SELECT * FROM read_xlsx('data/incoming/source.xlsx', sheet = 'Orders');
```

Useful first notebook outputs:

```python
import pandas as pd
from ydata_profiling import ProfileReport

df = pd.read_parquet("data/incoming/orders.parquet")
ProfileReport(df, title="Orders source profile", explorative=True).to_file(
    "artifacts/profiles/orders-source.html"
)
```

For Evidently, compare a well-understood reference period or system against a current source/target sample, and retain the generated HTML/JSON as evidence. Evidently supports fully local use; its [installation documentation](https://docs.evidentlyai.com/docs/setup/installation) uses `pip install evidently`.

```python
from evidently import Dataset, DataDefinition, Report
from evidently.presets import DataDriftPreset, DataSummaryPreset

definition = DataDefinition()
reference = Dataset.from_pandas(reference_df, data_definition=definition)
current = Dataset.from_pandas(current_df, data_definition=definition)
report = Report([DataSummaryPreset(), DataDriftPreset()])
snapshot = report.run(current_data=current, reference_data=reference)
snapshot.save_html("artifacts/drift/orders-source-vs-target.html")
```

Use the installed package's current API examples if its major version differs; this API is evolving faster than the core workflow.

## 4. Entity and system matching

### Splink: default local matching tool

Splink's DuckDB backend is the best initial choice for laptop-scale record linkage: it is fast, needs no server, produces explainable match probabilities, and supports human review of candidate pairs. Install is already covered above; Splink packages DuckDB/SQLite itself. See the [Splink getting-started guide](https://moj-analytical-services.github.io/splink/getting_started.html).

Use this sequence:

1. Profile each source and normalize candidate fields *without overwriting originals*: case-fold names, trim whitespace, split addresses, normalize phones to E.164, retain source system and source key.
2. Define blocking rules that are generous enough to retain true matches, e.g. normalized postcode + surname prefix, email domain + name, or tax-ID exact match.
3. Configure comparisons and train/estimate the model using domain-reviewed labels where possible.
4. Export candidate links with score, contributing comparisons, source IDs, model version and decision status.
5. Review borderline candidates with the business, then persist approved decisions as a reusable golden-label set.

Do **not** equate a high linkage score with a business merge. It is evidence for a stewardship decision. Retain source identifiers and the many-to-many evidence trail even after a canonical ID is assigned.

### Zingg: Spark-based matching

Zingg is appropriate after the first Splink pass demonstrates a need for larger-scale, trained entity resolution. Its Python API is a PySpark application and its documentation says that local Python programs also require the Zingg CLI; do not expect `pip install zingg` alone to create a working standalone lab. [Zingg Python guidance](https://docs.zingg.ai/latest/working-with-python) and its [single-machine setup](https://docs.zingg.ai/latest/stepbystep/installation/installing-from-release/single-machine-setup) are the authority.

The lowest-friction local smoke test is Docker:

```zsh
docker pull zingg/zingg
docker run --rm -it zingg/zingg bash
./scripts/zingg.sh --phase match --conf examples/febrl/config.json
```

For a native local installation, use the Zingg release matched to **Spark 3.5 and JDK 11**. Set `JAVA_HOME`, `SPARK_HOME`, and `SPARK_MASTER=local[*]` only inside a Zingg-specific shell script; do not make them the globally active Java/Spark settings. Set Zingg's `collectMetrics` configuration to `false` if the engagement prohibits telemetry.

## 5. Dedicated Spark/Deequ environment

Deequ performs profiling, verification, anomaly detection, and constraint suggestion on Spark DataFrames. It is the strongest fit for turning observed source characteristics into a reviewable rule catalogue, then into CI/pipeline checks.

### 5.1 Do not share this environment casually

Deequ's Maven artifact includes the Spark compatibility in its version name. Use a PySpark version and a Deequ artifact with the **same Spark minor version**. For example, an approved Spark 3.5 sandbox can use a coordinate of this form:

```text
com.amazon.deequ:deequ:<approved-deequ-version>-spark-3.5
```

Choose the exact latest compatible release from [Maven Central](https://repo1.maven.org/maven2/com/amazon/deequ/deequ/) and record it in the project. The upstream project's compatibility requirements and releases are published in the [Deequ repository](https://github.com/awslabs/deequ). Java requirements vary by Deequ/Spark combination; older Deequ guidance calls for Java 8, while Zingg's Mac guidance calls for Java 11. This is why the environments remain separate.

Create a Spark 3.5/Deequ shell wrapper (adjust both pinned versions after checking the client runtime):

```zsh
#!/usr/bin/env zsh
# scripts/pyspark-deequ.sh
export JAVA_HOME="$(brew --prefix openjdk@11)/libexec/openjdk.jdk/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
uv run --with "pyspark==3.5.5" pyspark \
  --packages "com.amazon.deequ:deequ:<approved-deequ-version>-spark-3.5"
```

Make it executable with `chmod +x scripts/pyspark-deequ.sh`, replace the placeholder, and launch it. Spark will download Maven packages on first run; have an approved artifact repository/mirror ready if direct internet access is restricted.

### 5.2 Constraint-suggestion operating procedure

1. Profile a representative, approved sample after source extraction and basic normalization.
2. Run Deequ's `ConstraintSuggestionRunner` against the Spark DataFrame.
3. Store every candidate rule with profiler metric, sample window, confidence/support, source version, proposed owner and approval status.
4. Review suggestions with data owners. Common candidates—non-null, uniqueness, type/pattern, numeric range, referential inclusion—are hypotheses, not automatically business rules.
5. Promote approved rules to Deequ checks/DQDL plus the target pipeline's tests; run them in the same Databricks Runtime family used in production.
6. Monitor the metric time series and distinguish an upstream change from a quality defect before rejecting data.

Suggested control columns for the rules register: `rule_id`, `business_kpi`, `source_system`, `table`, `column`, `rule_expression`, `metric`, `baseline`, `threshold`, `severity`, `owner`, `evidence_uri`, `approved_at`, `runtime_version`.

## 6. Databricks connection and parity

### 6.1 CLI and IDE

Authenticate without putting a personal access token in a notebook or Git repository:

```zsh
databricks auth login --host https://<your-workspace-host>
databricks auth profiles
```

Use a named profile if the engagement has more than one workspace. Follow the [Databricks CLI installation instructions](https://docs.databricks.com/aws/en/dev-tools/cli/install) and the client's approved authentication method (prefer OAuth/SSO where available).

Configure the Databricks VS Code extension to select the workspace and a suitable cluster/serverless compute. It can install the matching Databricks Connect package for the selected compute.

### 6.2 Keep remote Spark separate from local Spark

Create a separate environment only after you know the target Databricks Runtime (DBR):

```zsh
uv venv .venv-databricks --python 3.11
source .venv-databricks/bin/activate
uv pip install "databricks-connect==<version-matching-client-DBR>" databricks-sdk
```

Let the extension install/validate the exact version where possible. Databricks Connect runs your local Python control flow while Spark DataFrame work executes remotely; its version and compute requirements are documented in [Databricks Connect for the IDE](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/databricks-connect). Do not install an unrelated local `pyspark`, `delta-spark`, or Deequ JAR into this environment unless it exactly matches the selected runtime.

For a brownfield estate, capture this parity matrix before implementing a reusable check:

| Item | Local DuckDB lab | Local Deequ/Zingg sandbox | Target Databricks |
| --- | --- | --- | --- |
| Python version | Pinned in `pyproject.toml` | Pinned per Spark environment | DBR-supported version |
| Spark/Scala | N/A | Exact minor version pinned | DBR Spark/Scala version |
| Java | N/A | JDK per tool/version | Managed by DBR |
| Deequ/Zingg | N/A | Maven/release coordinate recorded | Cluster library/approved artifact |
| Input access | Approved sample only | Approved sample only | Governed Unity Catalog paths/tables |
| Evidence | Local artefact, no raw sensitive data | Local smoke-test log | Job output, table and catalog lineage |

## 7. Suggested engagement workflow

| Phase | Repeatable output | Local tool(s) | Promotion point |
| --- | --- | --- | --- |
| Inventory | `metadata/source_inventory.csv` | DuckDB, SQLAlchemy | Source list and ownership approved |
| Technical profiling | HTML profile + `column_metrics.parquet` | ydata-profiling, DuckDB, Deequ | Metrics reproduced in Databricks |
| Semantic mapping | `metadata/source_target_crosswalk.csv` | DuckDB, Markdown | Business signs off meaning and grain |
| Candidate matching | `artifacts/links/candidate_links.parquet` | Splink; Zingg if needed | Stewardship accepts/rejects candidates |
| Data-quality rules | `metadata/dq_rule_catalog.csv` | Deequ, Evidently, Pandera/GE | Rule has owner, severity, action |
| Conflict resolution | `metadata/conflict_log.csv` | DuckDB + review workflow | Decision has accountable owner |
| Pipeline hardening | Test suite and monitored metrics | dbt/Databricks + Deequ/Evidently | CI/CD and alert route are verified |

For every proposed KPI, explicitly capture: business definition, grain, dimensions, source-system precedence, reconciliation method, late-arriving-data policy, null/default policy, acceptable variance, quality rules, owner and escalation route. This prevents a technically correct join from becoming an ungoverned enterprise KPI.

## 8. A two-hour first lab

1. Put two approved, de-identified extracts in `data/incoming/` (for example `crm_customer.parquet` and `erp_customer.parquet`).
2. Use DuckDB `DESCRIBE`/`SUMMARIZE` to produce row count, key cardinality, null rate, type, min/max and top values per candidate mapping field.
3. Produce a ydata report for each source; record field-level semantics and grain in `metadata/source_inventory.csv`.
4. Create a canonical customer mapping draft with a preservation rule: never discard `source_system` or native IDs.
5. Run Splink to generate a *review queue*, not automatic merges.
6. Compare the two sources or a prior/current extract in Evidently; save the drift report.
7. On an approved Databricks-compatible sample, run Deequ constraint suggestion; triage suggestions into `accepted`, `needs-business-review`, or `rejected` with reasons.
8. Promote one accepted mapping and two accepted quality rules to a Databricks test/job, then prove results against the same sample.

## 9. Troubleshooting rules of thumb

- **`ClassNotFound`, Scala or Py4J errors:** assume a Spark/Scala/Deequ mismatch first. Recreate the dedicated environment; do not keep adding JARs until it works.
- **Java errors after installing Zingg:** check `JAVA_HOME` in the active terminal. Zingg and Deequ can need different JDKs.
- **Apple Silicon issues:** favour native ARM64 Python packages and Docker images; if a vendor connector is Intel-only, isolate that connector rather than running the whole lab under Rosetta.
- **Large files:** DuckDB/Polars for local columnar analysis; profile samples; send Spark-scale work to Databricks. Avoid loading multi-GB CSVs into pandas merely to make an HTML report.
- **False matches:** improve business labels and blocking/comparison logic; never lower the merge threshold simply to increase match volume.
- **Source credentials:** use an approved secret manager or the OS keychain; `.env` is for local references only and must stay ignored by Git.

## 10. Official references

- [DuckDB installation](https://duckdb.org/install/)
- [ydata-profiling installation](https://docs.profiling.ydata.ai/latest/getting-started/installation/)
- [Evidently installation](https://docs.evidentlyai.com/docs/setup/installation)
- [Splink getting started](https://moj-analytical-services.github.io/splink/getting_started.html)
- [Zingg single-machine setup](https://docs.zingg.ai/latest/stepbystep/installation/installing-from-release/single-machine-setup)
- [Deequ project and compatibility notes](https://github.com/awslabs/deequ)
- [Databricks CLI for macOS](https://docs.databricks.com/aws/en/dev-tools/cli/install)
- [Databricks IDE extension](https://docs.databricks.com/aws/en/dev-tools/vscode-ext/)
