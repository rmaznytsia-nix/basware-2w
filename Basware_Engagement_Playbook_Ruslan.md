**INTERNAL / PERSONAL — WORKING PLAYBOOK, NOT FOR CLIENT DISTRIBUTION**

# Basware Embed — 2-Week Playbook (Data Architect)

Context: this 2-week SOW (you + 1 Data Engineer, embedded in Basware's Cresco team) is explicitly framed as validation — "to validate N-iX's expertise and get a head start on the Discovery project." Every deliverable you produce here is really pre-sale collateral for the $58,880 / 5-week Discovery SOW N-iX already pitched. Treat this as a two-week audition that should end with Basware asking "when can the full team start."

Client contacts on this SOW: **Anthony Kirkland** (Enterprise Data Product Manager) and **Nishant Redekar** (Director Enterprise Architecture) — governance line. Barrett Schiwitz was the intro-call contact; confirm at kickoff whether he's still involved day-to-day or whether Kirkland/Redekar are now your primary counterparts.

---

## 1. Role design — use specialist capacity deliberately

This embed has complementary roles: the Data Architect concentrates on business semantics, canonical-model decisions, evidence, and stakeholder alignment; the certified Data Engineer concentrates on Databricks implementation and delivery mechanics. This is the fastest route to a credible, useful outcome in two weeks.

**Division of labor (agree this explicitly with him before Wednesday):**
- **You own:** business definitions, KPI semantics, source-to-Gold lineage narrative, canonical/domain model design, data governance framing, the two named ambiguities (Contract End Date, SAP ID hierarchy), the KPI walkthrough summary, the definition-process recommendations.
- **He owns:** platform mechanics — Unity Catalog specifics, Delta Live Tables / Lakeflow pipelines, cluster vs. SQL Warehouse questions, CI/CD and branching, anything requiring him to read/write Databricks code directly for his story delivery.
- **Shared:** joint summary, closing readout, cross-feeding findings (e.g., his dev-lifecycle observations may explain *why* the Gold layer got built KPI-by-KPI — that's a gift to your data-model narrative).

**Architectural contribution:** dimensional modeling (star schema, conformed dimensions, Kimball-style Gold layer) is technology-agnostic and transfers directly into a Databricks Gold-layer design. The preparation target is a common working vocabulary for productive architecture conversations: Bronze/Silver/Gold, Unity Catalog, Lakeflow, Metric Views, Genie, and the implementation decisions that should be verified with the Data Engineer.

**How to use the companion technical reference:** this playbook tells you what to do and deliver; [Databricks_Data_Modeling_Playbook.md](Databricks_Data_Modeling_Playbook.md) provides the detailed options when the walkthrough raises a modeling, reconciliation, Lakeflow, or future-Discovery question. Do not duplicate its technical research here. Use its routing guide to get to the right section, then record the chosen pattern and evidence in the KPI contract and decision log below.

**Working Databricks context for the engagement** (target for Tuesday night):
- Bronze/Silver/Gold in Delta Lake terms, Unity Catalog namespacing (catalog.schema.table), Delta Live Tables vs. plain notebooks, Databricks SQL Warehouses, and what Genie / Metric Views actually are (Basware's presale conversation already floated these as the semantic-layer tool — you should be able to discuss their fit without deep hands-on experience).
- Use your own Snowflake↔Databricks cheat sheet as a live reference during meetings — don't hide it, a data architect cross-walking terminology in real time reads as rigor, not weakness.
- Internal flag (never say this to Basware): the presale risk notes already question whether Metric Views/Genie are mature enough for Basware's actual business-logic complexity. If that risk surfaces during your walkthrough, note it in your internal findings — don't assert it to the client as fact, validate it against what you actually see.

---

## 1A. KPI elicitation strategy — turn documented definitions into tested operating definitions

The KPI catalogue is an important starting point, not proof that a KPI is implementable. Position the work constructively: **preserve the existing definition, make its evidence explicit, and close the gaps that only appear when business meaning meets source data and Gold implementation.** Do not frame this as correcting Accenture or reopening every KPI.

### Working principle

Business stakeholders should not be asked to interpret raw records unaided. The architecture/engineering pair prepares a small, readable evidence pack; business validates business meaning, scenarios, and exceptions; source owners validate field lifecycle and semantics; engineering validates implementability. Use **Appendix H** for the protocol and **Appendix I** for the evidence artifacts.

### What success looks like for the selected KPI

- The current definition is classified as **confirmed**, **partially evidenced**, or **assumption-dependent** rather than simply “documented.”
- Every material formula component has a stated grain, source field, transformation, owner, and test case.
- The team can explain the KPI through representative business scenarios and safely redacted record examples—not just SQL or workbook prose.
- Source-to-target mappings have a confidence/status, profiling evidence, and an owner for unresolved cases.
- Country/source variation is visible and explicitly handled, deferred, or ruled out for the selected KPI.
- Gold shortcuts are recorded as deviations with a reason, consumer impact, and a pragmatic convergence path; no full Gold rebuild is implied.

### Tone and workshop framing

Use questions such as: “What decision should this KPI change?”, “Show us three real business situations it must handle,” “Which source event makes this value change?”, and “What would make Finance reject this result?” Avoid asking stakeholders to validate column names, raw rows, or implementation detail. This keeps the workshop at the correct level while producing testable evidence.

---

## 2. Prep window: Monday Aug 24 + Tuesday Aug 25

You have two working days before Wednesday's kickoff. Spend them on things that pay off immediately in week 1 — not on becoming a Databricks expert.

### Monday Aug 24
- [ ] **Chase the KPI workbook now, don't wait for kickoff.** The SOW says the KPI semantic workbook (Overview / Entity Catalogue / Source System Map / KPI Relationships) and the 18 KPI design pages "will be made available at kickoff" — email Kirkland today asking to get read access *before* Wednesday so Tuesday can be spent pre-reading instead of Wednesday being your first look.
- [ ] **Push to pre-select the KPI.** Officially "the Cresco team will agree the specific KPI during week 1," but nothing stops you proposing one now. Strong case for **ARR** specifically: it's the one with a known, documented failure (the ARR logic error caught late in build) — you already have the story, the "5 Whys" root-cause framing is pre-built from the pitch deck, and fixing/explaining it directly reinforces the exact narrative that sold the bigger Discovery phase. If ARR is already "done" and off-limits, second choice: whichever of the 16 backlog KPIs touches Contract or Customer, since that's where the two named ambiguities live.
- [ ] **Chase access provisioning today**, not Wednesday morning. Databricks workspace, source control, Confluence (Cresco Design Stream) — access delays are the single most likely thing to eat your week 1. Ask Kirkland directly who owns provisioning and when it kicks off.
- [ ] **Databricks speed-run (2–3 hrs, AI-assisted).** Use Claude/Cursor as a tutor framed around what you already know: "explain Unity Catalog to someone who knows Snowflake RBAC and Snowflake namespacing cold." Do the same for Delta Live Tables vs. notebooks, and for Genie/Metric Views. Update your cheat sheet with anything new.
- [ ] **Re-read your own presale trail** (memory, intro-call notes, presale risk notes, discovery questions, the pitch deck) so you don't contradict what N-iX already told Basware. In particular: the pitch already named the two ambiguities, the ARR incident, and the "design first, build once" pitch — stay consistent with that language throughout.

### Tuesday Aug 25
- [ ] **Build the KPI walkthrough template now**, so Wednesday–Friday is pure listening/filling, not template design. Start with the **KPI Definition Contract** in **Appendix A**, then add source-to-Gold lineage, current Gold implementation, known gaps/ambiguities, and open questions. Draft it as a living document you fill in real time in front of stakeholders — visible artifact-building signals exactly the discipline N-iX is selling ("define before build").
- [ ] **Prepare the KPI elicitation pack in Appendix H and the evidence artifacts in Appendix I.** Pre-select three business scenarios the KPI must handle (normal, exception, and country/source variant), a short profiling request for the Data Engineer, and a source-to-target mapping sheet. This makes the first workshop evidence-led without asking business stakeholders to interpret raw records.
- [ ] **Pre-draft hypotheses for the two named ambiguities** so the workshops are validation, not discovery from zero:
  - *Contract End Date* — three candidate sources of truth (SalesCloud, M-Files, CPQ). Before Wednesday, write down the obvious question set: which system is contractually authoritative, do the three ever disagree today, has anyone measured how often, what's the business process that updates each one and in what order.
  - *Partner SAP ID hierarchy* — write down: is this a parent/child rollup problem (reseller → end customer), where does the hierarchy actually live today (SAP master data vs. somewhere else), what breaks downstream when it's wrong (likely revenue/KPI misattribution across the hierarchy).
- [ ] **Start the two running documents you'll keep open all two weeks:** a live glossary ("customer," "contract," and whatever else comes up) and a RAID/**decision** log using **Appendix B**. These aren't busywork — they *are* the "definition process improvement" deliverable if you populate them well, so start them today, empty is fine, structure is what matters.
- [ ] **Sync with your Data Engineer colleague.** Confirm the division of labor above, agree on shared vocabulary (you don't want to contradict each other in a room), and agree who says what in the kickoff.
- [ ] **Logistics:** calendar invites, timezone overlap check (Finland/UK/India — Cresco team is distributed), Basware Slack/Teams access request, confirm the AI-tooling ground rules apply to this SOW too (the bigger SOW draft has an AI-tooling clause — confirm Cursor/Claude/Gemini use is explicitly fine here, not assumed).

---

## 3. Week 1 (Wed Aug 26 – Fri Aug 28, continuing Mon Aug 31)

**Wed Aug 26 — Kickoff.**
- [ ] Confirm scope, access, and points of contact (per SOW governance section) — but also use this meeting to close three specific open items instead of leaving them vague:
  1. **Lock the KPI** for your walkthrough today, not later in the week.
  2. **Resolve the interim-readout date conflict in the SOW itself** — the deliverables table says the interim readout is due "end of week 1," but the timeline narrative places it "mid-week" under week 2. Don't let this slip through unresolved; ask directly and get a date on the calendar.
  3. Identify the actual **business SMEs** who can sign off on definitions (you'll need them by week 2, book them now while calendars are open).
- [ ] Learn the real org chart fast: who on Cresco is analytics engineer vs. BI engineer vs. the internal senior data engineer mentioned in the bigger SOW, and how Kirkland/Redekar/Schiwitz relate to each other and to the team day-to-day.
- [ ] **Agree decision rights for the selected KPI** using **Appendix C**. Put names and decision dates in the log before leaving kickoff.

**Thu Aug 27 – Fri Aug 28 — KPI onboarding.**
- [ ] Business definition interview(s), source-to-Gold lineage walkthrough, current Gold implementation review — fill your template live. Build the **KPI evidence pack** in **Appendix D** as you go.
- [ ] **Run a KPI case clinic, not a generic requirements interview.** Present the three prepared business scenarios and a readable, safely redacted record journey. Ask business to validate the outcome and exceptions; ask source owners to validate field lifecycle; ask the Data Engineer to profile the relevant source fields and mapping coverage. Capture results in Appendices H and I.
- [ ] **Use AI on the actual Databricks code**, not just for orientation: point Claude/Cursor at the real notebooks/dbt/DLT logic behind this KPI and have it explain the pipeline back to you. This is the highest-leverage use of AI available this week: it accelerates architecture review of unfamiliar Spark/SQL and lets the Data Engineer spend specialist capacity on implementation and delivery.
- [ ] Populate the glossary and RAID log every day, don't batch it for Friday night.
- [ ] Draft the **KPI walkthrough summary** incrementally — have a rough version by Thursday night, polished by Friday, not started Friday afternoon.

**Fri Aug 28 (or wherever the kickoff lands the date) — Interim readout + KPI walkthrough summary due.**
- [ ] Structure the readout the way the N-iX pitch deck already framed things for Basware — "what we heard / our point of view," "where the gap is" style tables. Basware has already seen and responded to that framing in the sales deck; reusing it signals the sales pitch and the delivery are the same substance, not a bait-and-switch.
- [ ] Deliver the KPI walkthrough summary: business definition, lineage, current implementation, evidence pack, and named decisions — in enough detail that it stands alone (this is literally the SOW's acceptance bar: "enough detail for the Cresco team to act without further clarification"). Use the **Appendix D** contents and mark every conclusion as **confirmed**, **hypothesis to validate**, **illustrative pattern**, or **decision pending**; the companion Data Modeling Playbook defines these labels.

**Weekend Aug 29–30 — your regroup buffer.** Use it to: catch up on anything Databricks-specific that came up and you didn't fully follow, tidy the glossary/RAID log, and pre-draft the structure of the week-2 deliverables (definition-process recommendations, Gold-layer improvement list) so week 2 is refinement, not a blank page.

---

## 4. Week 2 (Mon Aug 31 – Fri Sep 4)

- [ ] Turn the KPI walkthrough into two concrete outputs:
  1. **Definition process improvement recommendations** — keep this lightweight and adoptable. Cresco has no dedicated BA support and a small team (5 internal + 3 contractors); don't propose a heavyweight governance framework they can't sustain. Demonstrate the minimum definition lifecycle in **Appendix E** with the selected KPI.
  2. **Gold Layer data model improvements** — entity structure, relationships, domain alignment, framed around what the KPI walkthrough actually surfaced (don't invent a full canonical model from scratch in a week; point at the specific gaps you saw).
- [ ] **Triage the two existing custom-KPI Gold implementations** using Appendix I's deviation register. Identify the recurring pattern, the reason it was needed, consumer impact, and the smallest convergence action; recommend a staged path rather than refactoring both implementations in this SOW.
- [ ] **Explicit resolution path for both named ambiguities** — these are called out by name in the SOW, treat them as non-negotiable deliverables:
  - Contract End Date (SalesCloud vs. M-Files vs. CPQ) — state which system should be authoritative and why, or the concrete next step to decide if you can't resolve it outright in two weeks.
  - Partner SAP ID hierarchy — same treatment.
- [ ] **Use the least-complex valid path for the SAP partner issue.** First decide whether it is a deterministic hierarchy/role mapping, a known-entity source reconciliation problem, or a genuine entity-matching problem. Use the detailed decision pattern in the Data Modeling Playbook §7; do not recommend probabilistic matching merely because several systems are involved.
- [ ] **Add a safety and acceptance plan for any proposed change** using **Appendix F**. Reference Data Modeling Playbook §§2A and 4 for technical control patterns.
- [ ] Sync with the Data Engineer's dev-lifecycle findings for the **joint summary** — look for one or two places where his platform observations and your model observations point at the same root cause (e.g., "KPI-by-KPI builds happened because there was no shared model *and* no branching discipline to catch it" — a joint story is more persuasive than two parallel reports).
- [ ] Build the **joint summary + closing readout** in the approved N-iX client-deliverable template — don't hand over a plain memo. Reuse the pitch deck's visual language (Assess → Stabilize → Build → Scale, "Where the Gap Is" tables) so the closing readout reads as a natural continuation of the sales narrative.
- [ ] **End the joint summary with an explicit next step**, not an open question. Since the 5-week Discovery SOW ($58,880) is already drafted and priced, the joint summary should point straight at it (or a variant informed by what you actually found) — this is the single highest-leverage move for turning "impressed" into "prolonged engagement." Don't make Basware ask what's next; tell them.
- [ ] Self-review pass: before anything goes external, have AI (or your colleague) sanity-check it against the acceptance criterion — "enough detail to act without further clarification from N-iX."

---

## 5. AI leverage checklist — use it everywhere it saves time or raises quality

**Apply the AI/data-handling and scope guardrails in Appendix G** before using AI with Basware materials.

- Reading unfamiliar Databricks/Spark/dbt/DLT code and having it explained back in architecture terms, so specialist engineering time stays focused on implementation and delivery.
- Nightly: turning raw interview notes into polished glossary entries, lineage descriptions, and the walkthrough summary draft.
- Pre-meeting: generating a tailored question set per stakeholder role before each workshop.
- Cheat-sheet extension: capture any new Databricks term you hit live, ask AI to translate it against your Snowflake mental model on the spot.
- Self-QA pass on every deliverable before it goes to Basware.
- If the moment allows it, a 5-minute live mini-demo of the "define before build" AI DLC loop using the Contract End Date or ARR ambiguity as the example — you already have a working reference build in `data-ai-dlc`, and the pitch deck already planned exactly this kind of demo. Low effort, high memorability, directly reinforces the sales narrative. Don't force it if there's no natural opening.

---

## 6. Impression management

**Scope discipline is part of the impression.** Apply the stop-rules in **Appendix G**: make the selected KPI excellent, leave an explicit resolution path for the named open items, and frame broader options as next-step Discovery work.

- Open the KPI conversation already knowing the ARR story and the two named ambiguities. Informed questions from minute one show that the architecture stream is prepared to focus on business meaning, lineage, and decisions while the engineering stream handles platform mechanics.
- Keep the internal presale risk notes (Metric Views/Genie doubts, team-adoption risk, "is this really an architecture problem or an analytical-maturity problem") private. If they turn out to be real, surface them diplomatically and hedged — validate, don't assert — consistent with how you've handled client communication so far.
- Stay visibly coordinated with your Data Engineer colleague — shared vocabulary, no contradicting each other, cross-referencing each other's findings. Two people who read as one aligned team looks far more "enterprise" than two people working in silos.
- Every deliverable here is implicitly sales collateral for the bigger Discovery phase — hold it to the same bar as the brand deck, not to "internal working note" quality.

---

## 7. Pre-flight checklist for Wednesday morning

- [ ] KPI candidate proposed (ARR first choice)
- [ ] Access requests sent (Databricks, repo, Confluence, Slack/Teams)
- [ ] KPI Definition Contract / walkthrough template ready
- [ ] Glossary + RAID/decision log docs created (empty is fine)
- [ ] Hypotheses drafted for Contract End Date and SAP ID hierarchy
- [ ] Databricks terms speed-run done (Unity Catalog, Delta Lake, DLT, Genie/Metric Views)
- [ ] Synced with Data Engineer colleague on division of labor and kickoff talking points
- [ ] Re-read own presale trail for consistency
- [ ] Interim-readout date conflict flagged as a kickoff question

---

## Appendix A — KPI Definition Contract

Create one contract for the selected KPI. It is the working artifact for the walkthrough and the reusable example of the proposed definition process.

| Field | Capture | Example — ARR |
|---|---|---|
| KPI name and business purpose | The decision it supports and the accountable business owner. | **Annual Recurring Revenue (ARR)** — measures contracted recurring revenue for executive planning and commercial performance; owner: _named commercial-finance owner_. |
| Definition and formula | Plain-language definition plus the agreed calculation expression. | Annualized recurring contract value for eligible active subscriptions at the reporting cut-off; exclude one-time implementation, usage, and non-recurring services. |
| Grain | What one row or observation represents. | One contract subscription line for one effective period, linked to customer, product, currency, and partner. |
| Time semantics | Effective date, reporting period, timezone, snapshot/cut-off rule, and late-arrival treatment. | Month-end snapshot; include a line when its effective start is on/before month end and its agreed contract end is after month end; document the late-arrival correction window. |
| Inclusions and exclusions | Explicit business population, statuses, products, geographies, and exception rules. | Include active, recurring customer subscriptions; exclude prospects, cancelled lines effective before cut-off, one-off fees, internal/test customers, and non-recurring products. |
| Source-of-record by attribute | Authoritative source for every disputed attribute, including Contract End Date if relevant. | Contract End Date: **decision pending** between M-Files, CPQ, and SalesCloud; contract amount/currency/product: _confirm source and fallback rule_. |
| Gold representation | Gold entities, fields, relationships, and metric/report implementation. | `fact_subscription` joined to `dim_contract`, `dim_customer`, `dim_product`, `dim_partner`, and calendar; served through the approved ARR metric/report definition. |
| Freshness and consumers | Update cadence, expected latency, dependent reports, and downstream owners. | Monthly close plus agreed intra-month refresh; consumers: executive ARR dashboard, finance planning, and commercial reporting; owners: _name each_. |
| Assumptions and open decisions | Link to the relevant Appendix B entries. | Link to Contract End Date authority and SAP partner-attribution decisions; state the interim rule, if a report must run before approval. |
| Validation | Control totals, representative edge cases, variance tolerance, and expected results. | Reconcile to approved finance control total; test renewals, cancellations, amendments, reseller deals, overlapping contracts, missing end dates, and currency conversion; agree allowable variance. |
| Approval and version | Decision owner, approvers, date, definition version, and change summary. | Business KPI owner approves; source and BI owners review; `ARR v1.0`, approved date _TBD_; record the change from prior calculation, if any. |

## Appendix B — RAID and Decision Log

Maintain this log daily. An unresolved decision must have an owner, due date, interim assumption, and stated consequence.

| Type | Ambiguity, risk, assumption, issue, or decision | Options / current state | Evidence needed | Owner | Due date | Interim assumption | Impact if unresolved | Outcome / link |
|---|---|---|---|---|---|---|---|---|
| Decision | Contract End Date authority | SalesCloud / M-Files / CPQ | Field semantics, update process, disagreement sample, KPI impact | _Name_ | _Date_ | _If needed_ | ARR definition cannot be approved | _Decision record_ |
| Decision | SAP partner hierarchy | Deterministic hierarchy / reconciliation / entity matching | IDs, hierarchy extract, role rules, unmatched-rate evidence | _Name_ | _Date_ | _If needed_ | Attribution remains uncertain | _Decision record_ |

## Appendix C — KPI Decision Rights

Confirm named people for these roles in kickoff. The Data Architect recommends; accountable Basware owners decide.

| Role | Accountable for | Required contribution |
|---|---|---|
| Business KPI owner | Approving business definition, intended use, materiality, and acceptance | Signs the Definition Contract. |
| Source-system owner | Meaning, lifecycle, quality, and latency of source attributes | Confirms or disputes source-of-record proposals. |
| Cresco Analytics / BI owner | Consumer semantics and report impact | Confirms report compatibility and validation cases. |
| Cresco Data / Analytics Engineer | Gold implementation, tests, controls, and release feasibility | Confirms implementation and rollback plan. |
| Data Architect | Options, lineage, evidence, impact, and recommended resolution | Maintains the contract and decision log; does not self-approve business semantics. |
| Enterprise Data Product Manager / Enterprise Architecture | Resolving cross-domain deadlocks and prioritizing next steps | Escalation point when accountable owners cannot decide. |

## Appendix D — KPI Evidence Pack

The Friday walkthrough summary must contain or link to each applicable item below. It is evidence, not a generic assessment.

- Current KPI/report output and the report/dashboard owner.
- Current business wording, formula, and implementation location.
- Source → Bronze/Silver → Gold → metric/report lineage.
- Grain, key, time semantics, and source-to-target mapping.
- Sample reconciliation results: population size, disagreement categories/counts, and representative records or safely redacted examples.
- Current data-quality/control checks and observed gaps.
- Open decisions, owner, due date, and interim assumption from Appendix B.
- Existing versus proposed definition/model change, affected consumers, and recommended action.
- Clear status labels: **confirmed**, **hypothesis to validate**, **illustrative pattern**, or **decision pending**.

## Appendix E — Minimum KPI Definition Lifecycle

Demonstrate this lifecycle with the selected KPI. The handover recommendation is the process plus its completed example, not a new heavyweight governance framework.

1. **Propose** — business owner states the intended KPI and decision it supports.
2. **Evidence** — collect lineage, source semantics, current implementation, and reconciliation evidence.
3. **Review** — business, source, BI, and engineering owners review the Definition Contract and options.
4. **Decide** — record the approval, interim assumption, or escalation in Appendix B.
5. **Specify or implement** — define the required Gold-model, metric, and pipeline/control change.
6. **Validate** — execute agreed examples, control totals, and variance checks.
7. **Version and communicate** — version the definition, record consumer impact, and publish the approved result in the team’s existing workbook/documentation location.

## Appendix F — Proposed Change Safety and Acceptance Plan

Complete this for any recommendation that may alter a KPI, Gold entity, or pipeline.

| Control | Record before promotion |
|---|---|
| Baseline | Approved control period, current KPI result, and source/Gold versions used. |
| Edge cases | Record-level cases covering nulls, duplicates, late changes, hierarchy changes, and relevant business exceptions. |
| Acceptance | Expected result and acceptable KPI variance; named approver. |
| Consumer impact | Reports, extracts, models, and owners affected by the change. |
| Promotion | Dev/staging/production path, implementation owner, and release window. |
| Rollback | Trigger, owner, rollback action, and how the prior definition/result is restored. |
| Monitoring | Data-quality checks, reconciliation/control total, alert owner, and post-release observation window. |

## Appendix G — AI/Data Handling and Scope Guardrails

### AI/data handling

- Do not put production rows, credentials, customer information, or other restricted material into an external AI tool without explicit approval and sanitization.
- AI may work from approved code, metadata, schema, lineage, and redacted examples.
- A human verifies every AI-generated explanation, recommendation, or client-facing artifact before use.
- Record material AI-assisted assumptions or transformations when they affect a deliverable conclusion.

### Scope stop-rules

- Do not review all 18 KPIs; make the selected KPI exemplary.
- Do not rebuild Gold or re-platform integrations within this SOW.
- Do not initiate a vendor/tool evaluation unless selected-KPI evidence shows it is material; frame such work as a Discovery option.
- Do not call an ambiguity an architecture defect before evidence establishes whether it is a business definition, source process, data-quality, hierarchy, or implementation issue.
- Do not recommend probabilistic entity matching before Appendix C owners and the Data Modeling Playbook §7 classification path establish that deterministic options are inadequate.

## Appendix H — KPI Elicitation Protocol for Fragmented Evidence

Use this protocol when a KPI is documented but its business meaning, source behavior, or Gold implementation has not been jointly tested. The aim is not to discredit prior work; it is to convert a documented definition into an agreed, evidence-backed operating definition.

### 1. Establish the decision before discussing data

Ask the business KPI owner:

- What decision changes when this KPI moves?
- Who acts on it, and at what cadence?
- What result would be surprising or unacceptable to Finance/commercial leadership?
- Which three business situations must it handle correctly: normal case, exception, and country/source variant?

Record the answers in Appendix A. Business validates outcomes and exceptions—not column names or raw records.

### 2. Run a case clinic with readable evidence

For each scenario, prepare a one-page case card: business event, safely redacted record journey, current KPI result, expected result, relevant source events, and open question. Work through the case with the right people:

| Participant | Validate |
|---|---|
| Business owner / SME | Business outcome, inclusion/exclusion, exception handling, materiality. |
| Source-system owner | Meaning, lifecycle, update timing, and reliability of the candidate attributes. |
| Data Engineer | Actual availability, profiling result, mapping feasibility, transformation/control implication. |
| Data Architect | Grain, lineage, model impact, evidence status, and recommendation. |

Do not ask business to interpret unprepared raw extracts. If a record is needed, translate it into a readable journey and preserve a link to the controlled technical evidence.

### 3. Make every material component testable

For each formula term and material rule, record: business wording, grain, candidate source field, transformation, data-quality risk, country/source applicability, evidence status, owner, and a test case. Use Appendix I's mapping sheet. “Documented” becomes **confirmed** only when the business outcome, source semantics, and implementation path agree.

### 4. Treat data quality as a repeatable classification problem

Do not handle every incident as a one-off. Classify it first:

| Category | Typical signal | Default response |
|---|---|---|
| Missing/late data | Nulls or arrivals after reporting cut-off | Define completeness/freshness rule, exception route, and monitoring threshold. |
| Invalid value | Value violates format, range, or allowed status | Expectation, quarantine/rejection path, and source-owner feedback. |
| Duplicate / conflicting record | More than one candidate for the same business key | Deterministic survivorship/reconciliation rule, or escalation if semantics are unresolved. |
| Mapping gap | No reliable source field or target transformation | Mark mapping as unresolved; do not silently derive a substitute. |
| Country/source variation | Same business concept follows different process or code set | Record a variation rule, confirm whether it is legitimate, and test it separately. |

### 5. Finish each clinic with one of four outcomes

- **Confirmed rule** — ready to specify/implement and validate.
- **Interim rule** — usable for a stated period with named assumption, owner, and expiry date.
- **Decision required** — evidence and options are ready; accountable owner must choose.
- **Discovery item** — not safely resolvable in this SOW; define scope, impact, and recommended next step.

## Appendix I — Mapping, Profiling, Variability, and Gold Deviation Artifacts

### I.1 Source-to-target mapping evidence sheet

Create one row for every material source attribute or derived component of the selected KPI.

| Business term / KPI component | Gold target and grain | Candidate source field(s) | Profiling checks and result | Transformation / precedence rule | DQ risk | Country/source variation | Confidence/status | Owner and evidence link |
|---|---|---|---|---|---|---|---|---|
| Contract End Date | `dim_contract.contract_end_date`; one contract effective period | M-Files / CPQ / SalesCloud | Completeness, disagreement rate, update lag, values by country | Decision pending; no silent fallback | Conflicting values, late updates | Confirm country process differences | Decision required | _Named owner / query or sample_ |

### I.2 Profiling request to the Data Engineer

Keep profiling focused on the selected KPI and request evidence that answers a decision:

- Row count and distinct business-key count by source and country.
- Null/completeness rate, allowed values, duplicates, and freshness/update-lag distribution for candidate fields.
- Cross-source match rate and disagreement rate by country, product, and material status.
- Representative samples for each disagreement category, redacted for business review where required.
- Existing pipeline transformations, expectation results, and known failure/retry patterns.

### I.3 Country and source-variation matrix

Do not assume a global source rule. Capture the selected KPI's relevant scope explicitly.

| Country / region | Source system / process variant | Same business definition? | Mapping or rule difference | Evidence | Owner | Treatment |
|---|---|---|---|---|---|---|
| _Example country_ | _Source/process_ | Yes / No / Unknown | _Difference_ | _Link_ | _Name_ | Standardize / parameterize / defer |

### I.4 Gold deviation register

Use this only to understand the two existing custom KPI implementations and identify a staged convergence path. It is not a mandate to refactor them in this SOW.

| KPI / Gold object | Custom pattern | Why it was introduced | Reusable pattern missing | Consumer impact | Risk | Smallest next action | Discovery follow-up |
|---|---|---|---|---|---|---|---|
| _KPI 1_ | _Custom table/join/calculation_ | _Known reason or hypothesis_ | _Conformed dimension, fact, metric, or pipeline pattern_ | _Reports/users_ | _Consistency/maintenance/DQ_ | Document, test, or extract a shared component | _Yes / No_ |
