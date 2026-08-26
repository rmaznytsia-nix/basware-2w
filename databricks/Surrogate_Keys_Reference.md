**INTERNAL / PERSONAL — WORKING REFERENCE, NOT FOR CLIENT DISTRIBUTION**

# Surrogate keys — reference guide

Lookup for Gold-layer key generation during modeling and Lakeflow design. Not a tutorial. Parent: [Data Modeling Playbook](./Databricks_Data_Modeling_Playbook.md).

**Claim labels** follow the playbook: **Illustrative pattern** unless a source is cited. Multi-source identity (SalesCloud / CPQ / M-Files) is **Hypothesis to validate** until evidence exists.

---

## Contents

- [When to open this](#when-to-open-this)
- [What the two sources actually say](#what-the-two-sources-actually-say)
- [Three jobs (do not collapse)](#three-jobs-do-not-collapse)
- [Choose by constraint](#choose-by-constraint)
- [Generation options](#generation-options)
- [Substitution and storage](#substitution-and-storage)
- [SCD2: entity vs version](#scd2-entity-vs-version)
- [Lakeflow pairing](#lakeflow-pairing)
- [Data Vault 2.0](#data-vault-20)
- [Hard rules](#hard-rules)
- [Sources](#sources)

---

## When to open this

| Question | Go to |
|---|---|
| Natural key vs surrogate? | [What the two sources actually say](#what-the-two-sources-actually-say) then [Choose by constraint](#choose-by-constraint) |
| Will a full refresh break fact joins? | Column **Reload-safe?** in [Generation options](#generation-options) |
| Hash vs IDENTITY vs mapping table? | [Generation options](#generation-options) |
| SCD2 needs a new key? | [SCD2: entity vs version](#scd2-entity-vs-version) |
| Dim is an MV, fact is a streaming table? | [Lakeflow pairing](#lakeflow-pairing) |
| How does Data Vault 2.0 mint keys? | [Data Vault 2.0](#data-vault-20) |

---

## What the two sources actually say

Do not merge these into one rule.

| | Kimball (Ch. 10–12) | Databricks Lakeflow dimensional modeling |
|---|---|---|
| **Join key** | Always a warehouse surrogate integer. Natural key is an **attribute**, never the fact–dim FK. | **Prefer the natural key** when it is stable and usable (clusters and joins well). Surrogate only if the source reuses or mutates IDs. |
| **Why a surrogate** | Multi-source collision, key reuse, reassignment (e.g. employee number recycled). Warehouse owns identity. | Same failure modes; introduce a SK only when they apply. |
| **How to mint** | Next sequential integer (`max(sk)+1`). Durable key still sequence-based. Reload survival via a **persisted NK→SK map**. | If you mint a SK: **deterministic, order-preserving** from the stable NK so a rebuild keeps fact FKs valid. **Avoid `sha2(nk)`** (random → bad liquid clustering / Z-order). **`IDENTITY` only** if append-only and **never full-refreshed**. |
| **Hash keys** | Not prescribed. Meets his *intent* (anonymous, non-smart). Standard in dbt / Data Vault 2.0 (modern, not Kimball text). Closest Kimball text is hash-encoding for archive audit, not key gen. | Explicitly reject hash surrogates such as `sha2(natural_key)` for file locality. |

Official Lakeflow examples join `fact_orders.customer_id` to `dim_customer` with **no surrogate**.

---

## Three jobs (do not collapse)

A generator is often asked to do three jobs. Most failures come from using one mechanism for all three.

| Job | Meaning | Kimball | Databricks | Lakehouse extra |
|---|---|---|---|---|
| **Identity** | Same entity → same key after rebuild, late facts, parallel writers | Anonymous int + map or durable key | Deterministic function of NK, or IDENTITY if never rebuilt | Stateless preferred (no map to lose) |
| **Versioning** | SCD2: new version ≠ same as entity | New sequential SK **plus** durable entity key | AUTO CDC `KEYS` + `__START_AT` / `__END_AT` | Do not hash NK alone and call it the version PK |
| **Locality** | Adjacent rows in adjacent files | Compact integers (1990s join/index) | Order-preserving key; not a random hash | **Clustering columns can differ from the join PK** |

---

## Choose by constraint

Pick the row that matches what you **refuse to operate**. Default for Lakeflow Gold: stable single-source NK as join key.

| You refuse… | Use |
|---|---|
| Extra key machinery; source NK is stable and single-system | **Natural key as PK/FK** (Databricks default; Kimball risk register must be closed by source contract) |
| Mapping tables; dims and incremental facts must survive **full refresh** | **Deterministic SK**: namespaced order-preserving encoding if the NK is numeric; else **hash entity key + liquid-cluster on NK/date**, not on the hash |
| Compact integers + classic fact SK lookup + SCD2 version rows | **Sequence or IDENTITY + never-refreshed NK→SK map** (Kimball). Not IDENTITY on a rebuilt MV |
| dbt / Data Vault-style simplicity + anonymity | **`HASH(source_system, nk)` for entity**; SCD2 via AUTO CDC intervals (optional separate version SK) |
| File locality as the #1 Gold lever | Do not cluster on `sha2`. Order-preserving SK **or** cluster on something ordered |
| Cross-system “same Contract, three IDs” | **Mastering / durable map first** — no hash of a single NK can invent a match rule |

---

## Generation options

| Option | Mechanism | Reload-safe? | Kimball | Databricks | Pros | Cons |
|---|---|---|---|---|---|---|
| **Natural key as PK** | Source id is the join key | Yes if NK is stable | **Rejected** as join key | **Preferred** if stable | Zero state; best clustering; matches Lakeflow examples | Fails on reuse/reassignment; cannot conform multi-source; SCD2 still needs a version discriminator |
| **Sequential int** `max+1` | ETL counter per dim | **No** unless map persisted or history replayed in original order | Canonical (Ch. 11) | Not described; same family as IDENTITY | Compact; anonymous; SCD2-friendly | Load-order dependent; unsafe for distributed writers; full refresh remints FKs |
| **`GENERATED ALWAYS AS IDENTITY`** | Delta assigns `BIGINT` on insert | **No** — rebuild reassigns IDs | Platform equivalent of sequential int | Allowed **only** if append-only and **never full-refreshed** | Native, compact, no hash function | Wrong pairing with MVs (they recompute); two pipelines cannot independently assign the same id |
| **Durable key** | Meaningless int per **entity**, distinct from SCD2 version SK | Only if the registry survives | Ch. 10: EDW owns customer keys across conflicting sources | Not named | Facts can hang off “this customer across history” | Two keys to govern; still stateful if sequential; requires a mastering rule |
| **NK→SK map table** | Persisted `natural_key → surrogate_key` | **Yes if the map is never dropped** | Ch. 11 fast lookup; Ch. 12 metadata catalog | Not in the dim-modeling page; own it as a table **outside** dim full-refresh | Compact ints; placeholder rows for late facts | The map **is** production state; contention; not an MV |
| **Hash** `sha2` / `md5` / UUID v4 | `HASH(nk)` or `HASH(source \|\| nk)` | **Yes** (stateless) | Intent yes, text no. dbt `generate_surrogate_key`; DV 2.0 | **Avoid as the clustering/join SK** (`sha2` scatters files) | Parallel writers; MV refresh; namespaced multi-source | Locality; wide keys; collisions (SHA-256 operationally fine, MD5 poorer); PII not anonymized by hashing; recipe change remints all keys |
| **Order-preserving deterministic SK** | Function of stable NK that preserves sort order | **Yes** if purely a function of NK (not `row_number()` over a changing set) | Not his mechanism | **Recommended** when a SK is needed. Algorithm **not published** | Reload + locality | Underspecified; `row_number()` looks like this and behaves like IDENTITY |
| **Hash/int PK + separate clustering** | Join on hash or int; `CLUSTER BY (nk, date)` | Depends on PK method | Not developed (indexes were on SK) | Docs assume join key ≈ clustering key | Resolves hash-vs-locality | Must actually set clustering; point lookup by SK is random (usually fine) |
| **UUID v7 / time-sortable** | Time-ordered unique id, not a function of NK | **No** unless persisted | — | — | Better locality than SHA-256 | Wrong for conformed entity keys; use for append-only event ids |

**Order-preserving implementations** (gap-fill — not in either source):

| Recipe | Reload-safe? | Notes |
|---|---|---|
| Numeric NK as `BIGINT`, or `source_id << 48 \| nk` | Yes | Compact; Kimball would call high-bit source a “smart key”; Databricks likes locality |
| Fixed-width sortable string / binary of the NK | Yes | Wider joins |
| `row_number() OVER (ORDER BY nk)` | **No** for incremental facts | Late NKs that sort in the middle renumber everything |

---

## Substitution and storage

| Pattern | Kimball | Lakeflow / lakehouse |
|---|---|---|
| **NK→SK map** | Two-column table in memory during fact load | Streaming or managed table **you never full-refresh**. Not an MV |
| **Cache the dimension** | Ch. 11: skip the extra map if ETL caches the dim | Broadcast / lookup join `fact` → current `dim` on NK |
| **SK pipeline** | Multithreaded replace NK→SK on each fact (Subsystem #14) | Stream-static join, or compute deterministic SK in the fact `SELECT` (no lookup) |
| **Late-arriving fact** | Pause, insert placeholder dim row, return SK | Awkward in declarative pipelines. Deterministic SK can land the FK **before** the dim row (orphan until dim catches up) |
| **Instance-independent key service** | Subsystem #10: mint ints for distributed clients | Avoid in Spark; hash or namespaced encoding is the stateless substitute |

---

## SCD2: entity vs version

| | Entity identity | Version identity |
|---|---|---|
| **Must** | Survive attribute change | Change when a Type 2 attribute changes |
| **Kimball** | Durable key | New sequential SK per version |
| **Databricks AUTO CDC** | `KEYS` columns | `__START_AT` / `__END_AT`; PK conceptually = keys + interval |
| **Hash** | `HASH(source, nk)` | Separate `HASH(nk, valid_from)` **or** skip version SK and as-of join the interval |

Query current SCD2 row: `__END_AT IS NULL`. Point-in-time `D`: `__START_AT <= D AND (__END_AT > D OR __END_AT IS NULL)`.

`dim_date`: Databricks — generate with `sequence()` / `explode()`, do not ingest. Classic exception: `yyyyMMdd` int is a **smart key that is order-preserving and clustering-optimal**.

---

## Lakeflow pairing

Recommended Gold shape: **dimensions = materialized views** (or streaming + AUTO CDC SCD2); **facts = streaming tables**.

That pairing means dims **recompute** and facts **append**. Reminting dimension SKs breaks historical fact FKs.

| Dim dataset | Compatible SK |
|---|---|
| MV, may full-refresh | Natural key, or deterministic SK (encoded NK or hash). **Not IDENTITY** |
| Streaming, append-only, never full-refresh | IDENTITY or sequence OK |
| Streaming + AUTO CDC SCD2 | Business `KEYS` + intervals; IDENTITY only if you accept the refresh caveat |

Treat full refresh as a controlled procedure (watermark / stateful logic changes). Short-retention sources + full refresh → silently incomplete table **and** reminted IDENTITY.

---

## Data Vault 2.0

DV 2.0 answers the same problem by **splitting identity from history** and making the entity key a **stateless hash of the business key**. It does not mint Kimball integers, and it does not use `IDENTITY`.

### What DV 2.0 actually does with keys

| Object | Key | Survives full refresh? |
|---|---|---|
| **Hub** | `HASH(business_key)` — entity only | Yes. Same BK → same hub key after rebuild, late facts, parallel writers |
| **Link** | `HASH(bk_a \|\| bk_b \|\| …)` in a fixed order | Yes. Relationship identity is a function of the contributing BKs |
| **Satellite** | Parent hash + `load_date` (and dependent child key if needed) | Yes. Attribute change does **not** remint the hub |

DV 1.0 used sequences. DV 2.0 dropped them because sequences fail the three jobs above: they need a map, they do not parallelize, and a rebuild remints FKs.

Natural keys stay on the hub as attributes. The hash is the join key everywhere in the vault.

### Mapped to the three jobs

**Identity.** Hash of the BK is deterministic. No NK→SK map, no `max+1`, no `IDENTITY`. A Gold fact (or a link) that stored `contract_hk` yesterday still joins after you drop and rebuild the hub.

**Versioning.** SCD2 is not a new hub key. Hubs are insert-only. History lives in satellites: new row when `hashdiff` of the payload changes. Entity key and version key are different tables, so you never hash the NK alone and call it a version PK.

**Locality.** This is the Databricks collision. Random hashes scatter files. DV 2.0 does not solve clustering; it ignores it (indexes on hash in MPP warehouses were acceptable). On a lakehouse the compatible pattern is already in [Hard rules](#hard-rules): **join on the hash, liquid-cluster on BK / load_date / business date**.

### Against this guide's questions

**Natural key vs surrogate?** Both. BK is retained; the PK/FK is a hash surrogate. That is closer to Kimball's *intent* (anonymous, warehouse-owned) than to the Lakeflow default (join on stable NK). It is the constraint-table row: “dbt / Data Vault-style simplicity + anonymity.”

**Will a full refresh break fact joins?** Not if facts/links store hub/link hash keys derived from BKs. Rebuild is a function of the same preimage. Recipe change (`MD5` → `SHA-256`, extra delimiter, adding `source_system` later) remints everything — same warning as the hash row in [Generation options](#generation-options).

**Hash vs IDENTITY vs mapping table?** DV 2.0 refuses IDENTITY and refuses a sequence map. The map is replaced by the hash function. The remaining stateful object is **not** a key mint: it is a **same-as link** (or Business Vault bridge) when two BKs mean one real-world entity.

**SCD2 needs a new key?** No new entity key. New satellite row. Optional `load_end_date` on the satellite if you end-date; many vaults stay insert-only and compute current/PIT at read time (same idea as `__END_AT IS NULL`).

**Dim = MV, fact = streaming?** Compatible, because the dim key is a function of the BK. An MV that recomputes hubs still emits the same `hk`. IDENTITY on that MV would be the anti-pattern DV 2.0 was designed to avoid.

### What DV 2.0 does not solve

- **File locality** — still `CLUSTER BY` the BK/date, not the hash.
- **A business match rule** — same-as is explicit; hashing does not merge identities.
- **PII** — hashing a customer number is not anonymization ([Hard rules](#hard-rules) #6).
- **Gold usability** — analysts still want a star. Typical lakehouse: Raw Vault in Silver (hash keys), Gold dims/facts either keep those hash FKs or project order-preserving/NK keys *as a function of the same BK* so Gold refresh still does not break.

**One-line summary:** DV 2.0 makes the join key a reload-safe hash of the business key and puts SCD2 in satellites instead of reminting the entity — not `IDENTITY` or a dropped map.

---

## Hard rules

1. **Do not put `IDENTITY` on a rebuildable Gold dimension** (especially an MV). Databricks and Kimball reload math agree.
2. **Do not use `row_number()` as a “deterministic order-preserving key”** unless facts rebuild whenever the dim rebuilds.
3. **Do not cluster on `sha2`**. Hash for identity is compatible with Databricks **if** liquid clustering is on NK/date.
4. **Do not hash a mastered identity** until the match rule is stable — a changed match remints like a dropped map.
5. **Include `source_system` in any hash preimage** when NKs collide across systems. Version the recipe; adding a component later remints all keys.
6. **Hashing PII is not anonymization.** Kimball “anonymous” means “not a smart business code.”
7. **Never expose a SK as a smart key** or let business logic depend on its bit pattern.
8. **Date facts:** prefer a date dimension key (or `yyyyMMdd`), not a raw timestamp as the only time grain (Kimball Ch. 10).
9. **Keep the natural key on the dimension as an attribute** even when the PK is a surrogate (Kimball minimum column set).

**Pattern that satisfies both sources without dropping either:** entity key = deterministic function of `(source, natural key)`, order-preserving when the NK allows it; otherwise hash the entity key and liquid-cluster on NK and date; SCD2 = AUTO CDC intervals (optional version SK); cross-system mastering = an explicit durable map (stateful on purpose — it stores the business rule, not a counter).

---

## Sources

| Source | Role |
|---|---|
| [Dimensional modeling in Lakeflow pipelines](https://learn.microsoft.com/en-us/azure/databricks/ldp/best-practices/dimensional-modeling) | Natural-key preference; hash warning; order-preserving deterministic SK; IDENTITY refresh caveat |
| Kimball Group, *The Data Warehouse Toolkit* / ETL companion — Ch. 10–12 | Sequential SK; reject natural/smart join keys; durable key; NK→SK map; SK pipeline; late-arriving facts; instance-independent key service |
| dbt `generate_surrogate_key`; Data Vault 2.0 hash keys | Stateless hub/link hashes; satellites for history — [Data Vault 2.0](#data-vault-20) |
| Playbook [Appendix C — Kimball technical columns](./Databricks_Data_Modeling_Playbook.md#appendix-c--kimball-technical-columns) | Minimum `surrogate_key` / `natural_key` / `durable_key` on Gold dims; IDENTITY only if declared on `CREATE STREAMING TABLE` |
| Playbook [§3 Lakeflow pipeline design](./Databricks_Data_Modeling_Playbook.md#3-lakeflow-pipeline-design) | Prefer stable NK; SK only if source IDs reused/mutable; full refresh as recovery procedure |
