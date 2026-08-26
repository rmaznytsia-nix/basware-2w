# Metric Specification Template

Source: https://metric-specification-tem-nk2q3lz.gamma.site/

*Captured from the page on 2026-08-25.*

> One metric. One owner. Clear accountability.

John Wernfeldt  
Managing Director, Northridge Analytics  
john.wernfeldt@northridgeanalytics.com

## Foundation

### Pre-check: Does this metric deserve to exist?

Before investing time in formalizing any metric, you must justify its existence. Too many organizations measure what's easy rather than what's necessary. This pre-check forces discipline before defining a metric's technical specifications. Every metric carries cost computational resources, maintenance burden, stakeholder attention, and potential for misinterpretation. A metric without clear decision support is organizational noise.

The three questions below form a litmus test. If you cannot answer them with specificity and confidence, the metric should not exist. Vague answers like "it helps us understand the business" or "leadership wants visibility" are insufficient. This is where accountability begins with the courage to say no to metrics that serve no purpose.

**What decision does this metric directly support?**

Name the specific business decision. "Monitor performance" is not a decision. "Determine whether to increase ad spend in Region A" is a decision.

**Who makes that decision?**

Identify the decision-maker by name and role. If multiple people are involved, clarify who has final authority.

**What happens if this metric is wrong?**

Describe the downstream impact. If the answer is "nothing," the metric has no consequence and should not exist.

**If these questions cannot be answered clearly, stop here. This metric should not exist.**

## Section 1: Metric Overview

The metric overview establishes the foundational context for everything that follows. This section answers the most basic questions: What is this metric called? Where does it belong in the organization? Why does it exist? These questions seem simple, but ambiguity here cascades into confusion downstream. A metric name that varies across teams creates fragmentation. A metric without a clear business domain becomes organizationally homeless, making ownership disputes inevitable.

The purpose statement is where accountability crystallizes. It forces specificity about decision support. Every metric must serve a decision and that decision must have an owner. If the decision frequency is unclear, stakeholders will have mismatched expectations about refresh rates and timeliness. If no decision depends on the metric, it should be deleted immediately. Metrics are not trophies. They are tools.

**Metric name**

Clear, unambiguous name used consistently across the organization. Avoid abbreviations or jargon that require interpretation.

**Business domain**

Which business area this metric belongs to. Examples: Marketing, Sales, Operations, Finance, Customer Success.

**Purpose**

Primary decision this metric supports

**Decision owner (name and role)**

**Decision frequency (daily, weekly, monthly, quarterly)**

**If no decision depends on this metric, it should not exist.**

## Section 2: Business Definition

The business definition is the most important section of the specification. It describes what the metric represents in plain language language that any stakeholder can understand without technical knowledge. This is not where formulas live. This is not where SQL queries live. This is where meaning lives.

The definition must survive being read in isolation. A finance executive should be able to read this section without context and understand exactly what is being measured. A new team member should be able to apply it to real-world examples without needing clarification. If the definition requires accompanying documentation to make sense, it has failed.

The test of a good business definition is falsifiability. Given a concrete scenario, a stakeholder should be able to evaluate whether it fits the metric definition and answer true or false. Ambiguity here creates downstream inconsistency in interpretation, reporting, and decision-making.

### Rules for writing the definition

- Written for non-technical stakeholders
- Must survive being read in isolation
- No formulas, no SQL, no tool-specific language
- Should be answerable as true or false in examples

**This metric answers the question: "…"**

Complete this sentence with precision. The question should be specific enough that stakeholders immediately understand the metric's scope and boundaries.

## Section 3: Metric Owner

Ownership is the cornerstone of metric governance. Without a single accountable owner, metrics drift. Definitions change quietly. Quality degrades slowly. Disputes about interpretation become endless. The metric owner is not a committee. It is not a team. It is a named individual who has the authority and responsibility to defend the metric, approve changes, and make trade-offs when conflicts arise.

Ownership is not the same as building dashboards or maintaining data pipelines. Those are contributor roles. The owner is accountable for the definition itself what the metric means, how it should be used, and what happens when it breaks. If multiple people claim ownership, or if ownership is unclear, the metric should not be approved. Ambiguity in ownership is ambiguity in accountability, and ambiguity in accountability is organizational risk.

The owner must have the authority to make decisions. They must be willing to stand in front of leadership and defend why the metric is defined the way it is. They must have the organizational standing to reject proposed changes that would undermine the metric's integrity. If the proposed owner lacks this authority, find someone else or do not create the metric.

**Accountable for the definition**

The owner defines what the metric measures and ensures the definition remains consistent.

**Approves changes**

All changes to logic, scope, or source systems require the owner's explicit approval.

**Owns quality trade-offs**

When quality and speed conflict, the owner makes the call based on business priorities.

**Final interpretation authority**

When stakeholders disagree on how to interpret the metric, the owner has the final say.

**Ownership ≠ building dashboards. Ownership ≠ maintaining pipelines. If ownership is disputed, the metric is not approved.**

## Section 4: Contributors (Optional)

Contributors support the metric without owning it. They play essential roles building pipelines, validating logic, maintaining dashboards, reconciling results but they do not have final decision authority. This distinction is critical. When everyone is an owner, no one is an owner.

Contributors may propose changes. They may raise concerns. They may identify quality issues. But they cannot implement changes without the metric owner's approval. This boundary prevents well-intentioned but misaligned changes from fragmenting the metric's definition over time.

**Data Engineering**

Responsible for data availability, pipeline reliability, and system integration. They ensure the metric can be calculated consistently.

**Analytics**

Validates calculation logic, builds reporting, and provides usage insights. They ensure the metric is interpreted correctly.

**Finance**

Provides controls, reconciliation, and audit support. They ensure the metric aligns with financial reporting standards when applicable.

**Business Teams**

Use the metric in decision-making and provide feedback on its relevance. They ensure the metric remains useful.

**Contributors support the metric. They do not own it.**

## Section 5: Calculation Logic

The calculation logic explains how the metric is computed, but not in code. This section is for stakeholders who need to understand the "how" without needing to read SQL queries or data transformations. The goal is transparency at the conceptual level. What gets included? What gets excluded? What assumptions are baked into the logic?

Focus on the structure of the calculation, not the implementation. For example: "Total revenue is the sum of all completed transactions in a given period, excluding refunds and cancellations." That sentence is sufficient. SQL can live elsewhere. The logic should be clear enough that a business stakeholder could mentally walk through examples and understand why certain transactions count and others do not.

Explicit exclusions are especially important. Every metric has boundaries. Naming what is intentionally left out prevents misinterpretation. If your revenue metric excludes returns, say so. If your active user metric excludes internal employees, say so. Silence creates assumptions, and assumptions create misalignment.

1. **Describe the logic conceptually** — Explain how the metric is calculated without SQL or formulas. Focus on what gets measured and how it's aggregated.
2. **State explicit exclusions** — Identify what this metric intentionally does not include. Be specific about edge cases and boundary conditions.
3. **Document key assumptions** — List timing assumptions, aggregation rules, and known edge cases that affect how the metric behaves.

## Section 6: Source Systems

Every metric depends on data from somewhere. This section identifies where that data lives. Naming the source system is not just documentation it's a commitment. It tells contributors where to look when something breaks. It tells stakeholders what systems need to be reliable for this metric to be trustworthy.

If multiple systems contain similar data, explicitly state which one is the source of truth. Conflicts between systems are inevitable. When they happen, the metric owner must know which system wins. Ambiguity here leads to reconciliation exercises that never end.

Source unavailability is a real risk. What happens if the source system goes down? Does the metric stop updating? Does it fall back to a secondary source? Does it escalate to the metric owner? Answering this question in advance prevents chaos during incidents.

**Primary data sources**

System name(s) and tables or objects (if relevant). Be specific enough that data engineers can locate the data without guessing.

**Source of truth**

If multiple systems exist, explicitly state which one wins in conflicts. This eliminates ambiguity during reconciliation.

**Source unavailability**

What happens if the source of truth is unavailable? Describe the fallback plan or escalation process.

## Section 7: Data Quality Expectations

Data quality expectations define "good enough." Not all metrics need perfect data. Some tolerate gaps or delays. Others do not. This section makes quality standards explicit so that contributors know when to escalate and stakeholders know when to trust the metric. Quality without consequences is just hope. If a rule breaks and nothing happens, the rule was never real.

Define thresholds that matter. Completeness: "At least 95% of records must have non-null values." Freshness: "Data must be updated within 24 hours." Valid ranges: "Values must be between 0 and 100." These are not aspirations they are contracts. When a rule breaks, someone should be notified, and there should be a clear consequence.

Not all quality breaks are equal. Some require immediate action. Some require investigation. Some can be logged and reviewed later. The consequence should match the severity. Blocking downstream usage is appropriate when incorrect data would lead to bad decisions. Informing the owner is appropriate when the issue is observable but not critical. Defining these consequences in advance prevents overreaction and underreaction.

**Completeness thresholds**

Minimum acceptable percentage of non-null or valid records

**Valid value ranges**

Boundaries for acceptable values (e.g., percentages between 0-100)

**Freshness expectations**

Maximum acceptable delay between event occurrence and data availability

1. **Quality owner** — Who is notified and expected to act when a rule breaks
2. **Consequence when broken** — Inform only, block reporting, block downstream usage, or escalate to metric owner

**Quality without consequences is just hope.**

## Section 8: Change Control

Metrics evolve. Business context shifts. Source systems change. Definitions get refined. Change is inevitable, but uncontrolled change is chaos. This section establishes the rules for how changes are proposed, evaluated, approved, and communicated. Without change control, metrics become unstable. Stakeholders lose trust. Comparisons across time become meaningless.

Not all changes are equal. Updating documentation or clarifying naming conventions are non-breaking changes. They improve understanding without affecting the data. Changing calculation logic, switching source systems, or modifying scope are breaking changes. They alter what the metric measures. Breaking changes require approval from the metric owner and communication to all downstream consumers.

Silent changes are not allowed. Every change breaking or not—must be documented and communicated. Stakeholders who depend on the metric must know what changed and when. If a metric changes silently, trust erodes. Users will stop relying on it, defeating the purpose of governance.

1. **What counts as a change** — Logic updates, source system changes, definition refinements, scope adjustments
2. **Change classification** — Non-breaking changes (documentation, naming) vs. breaking changes (logic, scope, source)
3. **Approval required** — Who must approve each type of change (usually the metric owner)
4. **Communication** — How changes are communicated and to whom (e.g., email, changelog, dashboard alerts)

**Silent changes are not allowed.**

## Section 9: Usage and Dependencies

Metrics do not exist in isolation. They flow into dashboards, reports, forecasts, and models. Understanding where a metric is used and where it should not be used is essential for managing change. If you do not know who depends on a metric, you cannot assess the impact of changing it. If you do not know what the metric was designed for, you cannot prevent misuse.

Document primary use cases explicitly. Is this metric used for executive reporting? For operational dashboards? For forecasting? For training machine learning models? Each use case has different quality and latency requirements. A metric that is good enough for a weekly executive dashboard may not be good enough for real-time operational alerts. Naming the intended use cases helps contributors prioritize the right quality standards.

Just as important is naming what the metric is not intended for. If a metric measures customer acquisition but should not be used to calculate customer lifetime value, say so. If a metric is designed for internal reporting but should not be shared externally, say so. Boundaries prevent misuse. Misuse creates confusion, erodes trust, and leads to poor decisions.

### Primary use cases

- Dashboards
- Reports
- Forecasts
- AI models
- Operational alerts

**Not intended for**

Decisions or use cases this metric should not be used for. Be explicit about boundaries to prevent misuse.

**Downstream dependencies**

Known consumers impacted by changes. Include teams, dashboards, reports, and systems that rely on this metric.

## Section 10: Review Cadence

Metrics age. What was relevant six months ago may no longer matter. What was accurate a year ago may now be misleading. Regular review ensures that metrics remain useful and that obsolete metrics are removed. Without review, metric catalogs bloat. Teams lose track of what is trusted and what is deprecated.

Set a review frequency based on the metric's importance and volatility. Critical metrics may need quarterly review. Stable metrics may only need annual review. During each review, the metric owner should ask: Is the definition still valid? Is the usage still relevant? Are the quality expectations still realistic?

Sunset triggers define when a metric should be deprecated. If no one has used it in six months, deprecate it. If the underlying source system is being retired, deprecate it. If the decision it supported is no longer being made, deprecate it. Keeping dead metrics alive creates maintenance burden and clutters the catalog.

**Review frequency**

How often the metric is formally reviewed (e.g., quarterly, annually)

**Review scope**

Definition still valid? Usage still relevant? Quality expectations still realistic?

**Sunset triggers**

Conditions under which this metric should be deprecated (e.g., unused for 6 months, source system retired)

## Section 11: Status

Every metric has a lifecycle. It starts as a draft, becomes active when approved, and eventually gets deprecated when it is no longer needed. Tracking status prevents confusion about which metrics are ready for use and which are still under development. Visibility determines who can see the specification and where it is published. Some metrics are internal only. Some are shared across the organization. Some are public-facing. Clarifying visibility prevents inappropriate access or misuse.

**Draft**

Under development, not yet approved for use

**Active**

Approved and in use across the organization

**Deprecated**

No longer maintained, should not be used for new work

**Last reviewed**

Date of most recent formal review

**Next review**

Scheduled date for next review

**Visibility**

Where this specification is published and who can see it

## Approval

Approval is the final step. It transforms a draft specification into a commitment. The metric owner signs their name and accepts accountability. This is not bureaucracy it is clarity. When a metric is challenged, when quality breaks, when changes are proposed, everyone knows who is accountable.

The approval signature is a forcing function. It requires the owner to review the full specification before committing. It creates a paper trail. It signals to the organization that this metric has been vetted and is ready for use.

**Approved by (metric owner)**

Name  
Date

**This specification is a commitment, not documentation.**

## Final Note

If this metric is important enough to be discussed, it is important enough to be owned. No owner means no accountability. No accountability means permanent risk. Metrics without owners drift. Definitions change quietly. Quality degrades slowly. Disputes about interpretation become endless. Trust erodes. Decisions suffer.

This template is not about creating more documentation. It is about creating clarity. Clarity about what a metric measures. Clarity about who owns it. Clarity about what happens when it breaks. Clarity about how it can change. Clarity is the foundation of trust, and trust is the foundation of decision-making.

The organizations that govern metrics well do not have more process. They have more accountability. They know who owns each metric. They know what decisions each metric supports. They know what happens when quality breaks. They review metrics regularly and deprecate the ones that no longer matter. They treat metrics as products, not accidents.

Use this template to bring that discipline to your organization. One metric. One owner. Clear accountability. That is the standard. Anything less is a risk you choose to accept.

> No owner means no accountability. No accountability means permanent risk.
