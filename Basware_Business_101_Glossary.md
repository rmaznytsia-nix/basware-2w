**INTERNAL / PERSONAL — WORKING REFERENCE, NOT FOR CLIENT DISTRIBUTION**

# Basware Business 101 + Glossary

Why this exists: the bigger 5-week Discovery SOW has a dedicated Business/Data Analyst role to carry exactly this knowledge. This 2-week embed doesn't. That job falls on you by default — not to become a Basware product expert, but to have enough domain fluency that you can tell, unprompted, when a business definition is off, ask the right follow-up, and not need someone to explain what an "order form" or "gross margin" means mid-workshop.

Two domains matter here and they are *not the same thing* — keep them separate in your head:

1. **Basware's product domain** — what Basware sells (invoice lifecycle management / AP automation). You need this for context and credibility, but it is not what your KPIs measure.
2. **Basware's own corporate/GTM metrics** — ARR, gross margin, and the 16 KPIs still to build. This is Basware-the-company's own SaaS business performance, measured using Basware-the-company's own CRM/CPQ/contract/ERP systems. This is the actual subject of your engagement.

Conflating the two is the single easiest way to sound like you don't know the domain. A "touchless processing rate" question is about domain #1 (their product). A "Contract End Date" question is about domain #2 (their own subscription business).

## Contents

- [1. Basware, the company — fast facts](#1-basware-the-company--fast-facts)
- [2. Product domain 101](#2-product-domain-101-the-thing-basware-sells--context-not-your-kpi-subject)
- [3. Corporate KPI domain 101](#3-corporate-kpi-domain-101-this-is-what-your-engagement-actually-measures)
- [4. Why "Customer," "Contract," and "Partner" are genuinely ambiguous](#4-why-customer-contract-and-partner-are-genuinely-ambiguous-here)
- [5. Working hypothesis: business terms → canonical Gold-layer entities](#5-working-hypothesis-business-terms--canonical-gold-layer-entities)
- [6. Glossary — quick reference](#6-glossary--quick-reference)
  - [Product / operational](#product--operational-domain)
  - [Corporate / GTM KPI](#corporate--gtm-kpi-domain)
  - [Data/architecture](#dataarchitecture-domain-your-own-vocabulary-included-for-completeness)
- [7. Question bank](#7-question-bank--use-this-to-fill-the-ba-gap-in-workshops)
- [8. Databricks technical toolkit — moved out](#8-databricks-technical-toolkit--moved-out)
- [Sources](#sources)

---

## 1. Basware, the company — fast facts

- Invoice Lifecycle Management (ILM) platform: governs every invoice from supplier submission through compliance validation, coding, matching, approval, fraud screening, payment, and archiving — across countries and ERPs, in one flow.
- Core product lines: AP automation, e-invoicing network, e-procurement / procure-to-pay, compliance (continuous transaction controls, tax/e-invoicing mandates).
- Scale: 6,500+ customers, 190+ countries (per Basware's own marketing).
- Go-to-market is **not purely direct** — Basware sells substantially through a Value-Added Reseller (VAR) / channel partner network, alongside direct sales. This is the detail that explains why "partner SAP ID hierarchy" is a real, structural problem for them, not an edge case (see [Why "Customer," "Contract," and "Partner" are genuinely ambiguous](#4-why-customer-contract-and-partner-are-genuinely-ambiguous-here)).
- Recognized as a Leader in Forrester Wave (AP Invoice Automation) and Gartner Magic Quadrant (AP Applications).
- Split organizationally into a Product org and an Enterprise org (Barrett Schiwitz's side) — the data platform you're working on sits in Enterprise.
- 40-year milestone (Feb 2025): platform has processed 2 billion+ invoices and $10.1 trillion in total managed spend to date — useful scale color if it comes up, not a KPI itself.
- Third-party revenue estimate (GetLatka, 2025, **unconfirmed by Basware directly** — treat as background color only): ~$181.6M ARR, ~$544.8M valuation, ~1,700 employees, reported as bootstrapped (conflicts with PitchBook's $398M raised — a sign these third-party estimates aren't fully reliable, don't cite them to the client).

Sources: [Basware Invoice Lifecycle Management](https://www.basware.com/en/why-basware/invoice-lifecycle-management), [About Basware](https://www.basware.com/en/about-basware), [Basware Named a Leader in AP Invoice Automation 2026](https://news.basware.com/en/basware-named-a-leader-in-accounts-payable-invoice-automation-2026), [Strategic Partnering with Basware](https://www.basware.com/en/about-basware/partners/), [Basware Marks 40 Years of Innovation with $10.1 Trillion Spend Managed](https://news.basware.com/en/basware-marks-40-years-of-innovation-with-10.1-trillion-spend-managed-through-its-platform), [Basware Revenue 2025 — GetLatka](https://getlatka.com/companies/basware.com)

---

## 2. Product domain 101 (the thing Basware sells — context, not your KPI subject)

Basware's product moves a supplier's invoice through a pipeline until it's paid and archived. The vocabulary you'll hear in passing, even though it's not what you're measuring:

- **Invoice matching** — verifying a supplier invoice against a purchase order (2-way match: PO + invoice) and/or a goods receipt (3-way match: PO + invoice + receipt) before payment is released. 3-way match exists specifically to catch fraud/discrepancy before money moves.
- **Touchless processing (rate)** — the % of invoices that go from receipt to approval/payment with zero human intervention. This is described in your own notes as Basware's north-star *operational* metric — i.e., how well their product performs for their customers, not a corporate financial metric. It may well be one of the 16 remaining KPIs, in which case it behaves differently from ARR/gross margin (it's an operational/quality metric, not a revenue metric) — don't force it into a financial-KPI mental model if it comes up.
- **SmartCoding** — ML feature that proposes GL coding for non-PO invoices based on historical coding patterns.
- **E-invoicing network** — the infrastructure for sending/receiving structured e-invoices across trading partners, with compliance and archiving built in.
- **Continuous transaction controls (CTC) / e-invoicing mandates** — government-driven requirements (increasingly common across the EU and beyond) that invoices be reported to tax authorities in near-real time, in structured formats. Relevant background if any KPI touches compliance.

Product-domain terms you may hear without needing to own them: **PEPPOL** (a four-corner e-invoice exchange network — sender → sender's access point → receiver's access point → receiver — that avoids bilateral EDI setups), **UBL / EN 16931** (the European structured e-invoice data standard most PEPPOL traffic conforms to), **EDI** (older bilateral electronic document exchange, still common in some corridors), **CIUS** (a country-specific customization of the EN 16931 standard, e.g. XRechnung in Germany).

Sources: [What is invoice matching? — Basware](https://www.basware.com/en/glossary/what-is-invoice-matching), [Touchless Invoice Processing — Basware](https://www.basware.com/en/solutions/ap-automation/touchless-invoice-processing/), [SmartCoding — Basware](https://www.basware.com/en/solutions/ap-automation/smartcoding), [E-Invoicing Glossary — GoRoute](https://goroute.ai/glossary.html), [Peppol Glossary](https://peppolvalidator.com/glossary)

---

## 3. Corporate KPI domain 101 (this is what your engagement actually measures)

ARR and gross margin are **Basware's own SaaS business metrics** — how Basware, as a vendor, is doing selling subscriptions to *its* customers. They live in Basware's CRM (SalesCloud = Salesforce), CPQ, contract repository (M-Files), and ERP (SAP) — not in the AP-automation product telemetry.

**ARR (Annual Recurring Revenue)**
The annualized value of active recurring subscription revenue. Formula (simplified): MRR × 12, or the sum of annualized value across all active contracts. In practice, the hard part is never the formula — it's deciding *which contracts count as active, as of when, and at what value* (multi-year deals, mid-term upsells/downsells, one-time fees excluded, currency, proration on start/end dates). This is exactly where "Contract End Date" ambiguity breaks the number.

**Gross margin**
(Revenue − COGS) / Revenue. For a SaaS vendor, COGS typically = hosting/cloud infrastructure, third-party licensing embedded in delivery, customer support headcount tied to service delivery, payment processing fees. Sales, marketing, G&A, and R&D are *not* COGS — if you see them folded into a gross-margin calculation, that's a modeling red flag worth raising. Industry benchmark for mature SaaS is roughly 73–80%+; useful as a sanity check on whether a number "smells right," not as a target to chase.

**NRR / GRR (likely candidates among the remaining 16 KPIs, watch for them)**
- **GRR (Gross Revenue Retention)** — % of recurring revenue retained from existing customers, counting only churn and downgrades (excludes upsell). Healthy SaaS: ~90%+, best-in-class ~95%+.
- **NRR (Net Revenue Retention)** — same base, but *including* expansion (upsell/cross-sell). NRR > 100% means expansion outpaces churn. A common trap: high NRR can mask serious churn if a few big expansions are covering for many small losses — if this KPI ever comes up, ask whether it's reported alongside GRR, not alone.
- **Basware's own naming, confirmed:** their financial reporting doesn't say "GRR/NRR" — it says **cloud gross renewal rate** and **cloud net renewal rate** (FY2021: 96% gross, 104% net). Use their own vocabulary in workshops, not the generic SaaS-glossary terms, and expect the underlying definition/formula to match GRR/NRR even if the label differs.

**Why these numbers are architecturally hard, specifically at Basware:** every one of them depends on correctly identifying (a) who the customer is, (b) what counts as one contract vs. several, (c) when a contract starts/ends, and (d) how a reseller-fulfilled deal should be attributed. That's not a data engineering problem — it's a business-definition problem that data engineering then has to encode. That's the whole reason this SOW exists.

**Additional candidate KPIs — grounded in Basware's own reporting, not generic SaaS guesswork:**

- **Cloud ARR Order Intake (bookings)** — confirmed as a real, named line item in Basware's own financial statements, distinct from ARR itself (ARR = the recurring-revenue base; order intake = new bookings added in the period). Historical reference point: EUR 17.1M for FY2021. If this is one of the 16, expect it to be built from CPQ/SalesCloud opportunity-close data, not from the same logic as ARR.
- **New logo order intake** — a further split of order intake isolating genuinely new customers from renewals/expansion of existing ones (FY2021: new-logo intake grew ~36% YoY in Q4). Relevant if the 16 KPIs distinguish "growth from new customers" from "growth from existing ones."
- **Reseller share of order intake** — Basware has explicitly reported this split (~18% of FY2021 order intake came through resellers). This is real, historical evidence — not speculation — that the partner/VAR attribution problem in [Why "Customer," "Contract," and "Partner" are genuinely ambiguous](#4-why-customer-contract-and-partner-are-genuinely-ambiguous-here) is materially significant, not an edge case.
- **Adjusted EBITDA** — plausible candidate (common in SaaS earnings reporting, and Basware is a mature enough SaaS vendor to likely track it), but **not independently confirmed** in the sources checked here. Treat as an open hypothesis to validate against the KPI workbook, not a confirmed fact.

**Considered and explicitly excluded from this candidate list** (surfaced by an AI-generated guess at Basware's 18 KPIs, checked and rejected): **LTV, CAC, and the LTV:CAC ratio** — plausible board-level SaaS metrics in general, but they require marketing-spend attribution data that doesn't obviously live in any of the four confirmed source systems (SalesCloud, CPQ, M-Files, SAP); likely out of scope for this specific data-architecture engagement even if Basware tracks them elsewhere. **An "NPS ~97.4%" figure** also surfaced — flagged as very likely wrong or conflated (NPS is a −100-to-+100 scale, not a percentage), don't reuse that number for anything.

Sources: [Datarails — GRR vs NRR](https://www.datarails.com/grr-vs-nrr/), [CloudZero — SaaS Gross Margin](https://www.cloudzero.com/blog/saas-gross-margin/), [Chargebee — SaaS Gross Margin](https://www.chargebee.com/resources/glossaries/saas-gross-margin/), [Basware Financial Statements Bulletin January–December 2021 — Inderes](https://www.inderes.fi/en/releases/basware-financial-statements-bulletin-januarydecember-2021-order-intake-back-to-growth-net-sales-and-ebit-in-line-with-expectations)

---

<a id="4-why-customer-contract-and-partner-are-genuinely-ambiguous-here"></a>
## 4. Why "Customer," "Contract," and "Partner" are genuinely ambiguous here

This isn't sloppiness on Basware's part — their GTM model structurally creates the ambiguity:

**Customer** — Because Basware sells both direct and through VAR/channel partners, "the customer" can mean either the end organization using the software, or the partner who holds the commercial relationship and resells to the end org. If ARR is booked against the wrong level, revenue and renewal tracking silently drift.

**Contract** — At least three systems can each claim to hold "the" contract:
- **SalesCloud (Salesforce/CRM)** — where the opportunity/renewal is tracked, likely has a *renewal date* or *close date*, which is not necessarily the same as the legal end date.
- **CPQ (Configure-Price-Quote)** — generates the quote/order form with its own effective and end dates; this is the commercial terms as *quoted*, which can be amended after signature without CPQ being updated.
- **M-Files** — the document repository holding the actual signed contract; the legally authoritative end date lives here, but only if it's kept in sync with amendments, renewals, and terminations.
None of these is automatically wrong — the real question to ask on the ground is: **which one is updated last, by whom, and does anyone reconcile the three?** That's the actual finding you're being asked to produce, not "which system is right in theory."

**Partner (SAP ID hierarchy)** — SAP is the ERP/billing system of record. When a deal is resold through a VAR partner, SAP's customer/partner master data likely encodes a hierarchy (e.g., partner account → end-customer account, possibly multiple levels for larger resellers). If that hierarchy is flattened or misread, revenue/ARR gets attributed to the wrong node — either double-counted, attributed to the reseller instead of the end customer (or vice versa), or dropped entirely if the linkage breaks. This is a conformed-dimension problem: your canonical model needs a `Partner`/`Account` dimension that can represent hierarchy (parent/child, not just a flat customer list).

**This isn't a theoretical risk — it's confirmed material.** Basware's own FY2021 financial reporting states resellers accounted for roughly 18% of that year's order intake. That's a real, non-trivial share of bookings running through the exact attribution path your model has to get right — worth citing if you need to justify why this ambiguity deserves dedicated resolution time rather than being treated as a minor edge case.

Sources: [Basware Partner Program](https://www.basware.com/en/about-basware/partners/), [Basware Partner Ecosystem Profile — Verdict](https://www.verdict.co.uk/basware-partner-ecosystem-profile/), [Basware Financial Statements Bulletin January–December 2021 — Inderes](https://www.inderes.fi/en/releases/basware-financial-statements-bulletin-januarydecember-2021-order-intake-back-to-growth-net-sales-and-ebit-in-line-with-expectations)

---

## 5. Working hypothesis: business terms → canonical Gold-layer entities

Treat this as a draft to validate on the ground in week 1, not a finished model — but it gives you a starting shape instead of a blank page.

| Business concept | Likely canonical entity/dimension | Open question to validate |
|---|---|---|
| End organization using Basware | `Customer` (or `Account`) dimension | Is this always distinct from the billing/contracting entity? |
| Reseller/VAR | `Partner` dimension, parent-linked to `Customer` | Does SAP already model this as a hierarchy, or as flat rows you'd need to reconstruct? |
| Signed subscription agreement | `Contract` dimension | Single source of truth candidate: M-Files (legal) vs. CPQ (commercial) vs. SalesCloud (CRM view) |
| A specific line of subscribed product/value | `Subscription` / `Order Line` fact, linked to `Contract` and `Product` | Does one Contract roll up multiple Subscriptions (multi-product deals)? |
| ARR | Fact table aggregating active `Subscription` value as of a point in time | What exactly triggers a subscription to stop counting — contract end date, cancellation notice, non-renewal? |
| Gross margin | Fact table joining `Revenue` and `Cost` (hosting, support, COGS) at some grain | Is COGS tracked at customer/contract grain at all today, or only at company level? |
| Touchless processing rate | Product/operational fact, likely per-customer, per-invoice-volume | Separate data domain from GTM — don't merge into the same fact table as ARR without a clear reason |

---

## 6. Glossary — quick reference

### Product / operational domain
| Term | Definition |
|---|---|
| ILM (Invoice Lifecycle Management) | Basware's platform category: governs invoices end-to-end from submission to payment/archiving. |
| AP Automation | Automating accounts-payable workflows: receipt, coding, matching, approval, payment. |
| 2-way match | PO ↔ invoice match (quantity/amount) — no receipt check. |
| 3-way match | PO ↔ invoice ↔ goods receipt match — catches fraud/discrepancy before payment. |
| Touchless processing (rate) | % of invoices processed start-to-finish with zero human intervention; Basware's stated north-star operational metric. |
| SmartCoding | ML-based GL coding proposal for non-PO invoices. |
| E-invoicing network | Infrastructure for structured e-invoice exchange across trading partners, with compliance/archiving. |
| PEPPOL | Four-corner e-invoice exchange network (sender → AP → AP → receiver) avoiding bilateral EDI setups. |
| UBL / EN 16931 | European structured e-invoice data standard most PEPPOL traffic conforms to. |
| EDI | Older bilateral electronic document exchange format, still used in some corridors. |
| CTC (Continuous Transaction Controls) | Government mandates requiring near-real-time invoice reporting to tax authorities in structured formats. |
| VAR (Value-Added Reseller) | Channel partner who resells and implements Basware for end customers. |

### Corporate / GTM KPI domain
| Term | Definition |
|---|---|
| ARR (Annual Recurring Revenue) | Annualized value of active recurring subscription revenue; MRR × 12 in simplified form. |
| MRR | Monthly Recurring Revenue — the monthly-cadence version of the same concept. |
| Gross margin | (Revenue − COGS) / Revenue; COGS = hosting, delivery-linked support, third-party licensing, payment processing. |
| COGS | Cost of goods sold — direct costs of delivering the service; excludes sales/marketing/G&A/R&D. |
| GRR (Gross Revenue Retention) | % of recurring revenue retained from existing customers, counting churn/downgrade only (no expansion). Basware calls this **cloud gross renewal rate** internally (96% FY2021). |
| NRR (Net Revenue Retention) | Same base as GRR but including expansion (upsell/cross-sell); >100% = expanding faster than churning. Basware calls this **cloud net renewal rate** internally (104% FY2021). |
| Cloud ARR Order Intake (bookings) | New cloud bookings added in a period — distinct from ARR itself. Confirmed real Basware line item (EUR 17.1M FY2021). |
| New logo order intake | Order intake from genuinely new customers only, split out from renewal/expansion intake. |
| Reseller share of order intake | % of order intake attributed to VAR/reseller-driven deals (~18% FY2021) — direct evidence the partner-attribution problem in [Why "Customer," "Contract," and "Partner" are genuinely ambiguous](#4-why-customer-contract-and-partner-are-genuinely-ambiguous-here) is material. |
| Adjusted EBITDA | Plausible corporate KPI candidate — common in SaaS earnings reporting, **not independently confirmed** for Basware; validate against the KPI workbook. |
| Churn | Loss of recurring revenue/customers over a period, from cancellation or non-renewal. |
| CPQ (Configure-Price-Quote) | System generating price quotes/order forms with their own effective/end dates — commercial terms as quoted. |
| SalesCloud | Salesforce CRM — where opportunities, renewals, and account relationships are tracked. |
| M-Files | Document/contract repository — likely the legally authoritative source for signed contract terms. |
| SAP | ERP/billing system of record — likely holds partner/customer master data and financial postings. |

### Data/architecture domain (your own vocabulary, included for completeness)
| Term | Definition |
|---|---|
| Canonical / domain model | A single, reusable Gold-layer model organized by business domain (Customer, Contract, Product…) rather than built KPI-by-KPI. |
| Conformed dimension | A dimension (e.g., `Customer`, `Partner`) shared consistently across multiple fact tables/domains, so the same entity means the same thing everywhere. |
| Fact / dimension | Standard Kimball star-schema vocabulary: facts = measurable events (a subscription, a payment); dimensions = the descriptive context around them (who, what, when). |
| Definition of Ready (for a KPI) | The bar a KPI must clear — agreed business definition, known source-to-target lineage, no unresolved ambiguity — before it enters build. |
| Source-to-target lineage | The traceable path from a source system field to its final Gold-layer representation. |

---

## 7. Question bank — use this to fill the BA gap in workshops

Since there's no BA to run structured elicitation for you, borrow their toolkit:

- "Walk me through what happens in [SalesCloud / CPQ / M-Files] when a contract is signed, amended, renewed, or cancelled — in that order." (Surfaces which system is actually updated first and which lags.)
- "If these three systems disagreed on a contract's end date today, who would notice, and how?" (Tests whether reconciliation exists at all.)
- "When a deal goes through a partner, does the ARR get booked against the partner account or the end customer? Has that ever been inconsistent?"
- "Is COGS tracked at a granularity finer than company-wide today (per customer, per contract, per product)?" — if not, gross margin by KPI-relevant dimension may not be buildable yet, which is itself a finding.
- "Show me one contract you're confident is calculated correctly in ARR today, and one you're not sure about — what's different between them?"

---

## 8. Databricks technical toolkit — moved out

The modeling principles, feature catalog, Lakeflow pipeline rules, and source-to-Gold reconciliation method now live in the [Databricks Lakehouse Data Modeling Playbook](./Databricks_Data_Modeling_Playbook.md). This file stays focused on Basware's business domain; that one covers the technical toolkit you design the canonical model with.

---

## Sources

**Company & Strategic Background**
- [Basware Invoice Lifecycle Management](https://www.basware.com/en/why-basware/invoice-lifecycle-management)
- [About Basware](https://www.basware.com/en/about-basware)
- [Basware Named a Leader in AP Invoice Automation 2026](https://news.basware.com/en/basware-named-a-leader-in-accounts-payable-invoice-automation-2026)
- [Strategic Partnering with Basware](https://www.basware.com/en/about-basware/partners/)
- [Basware Partner Program](https://www.basware.com/en/about-basware/partners/)
- [Basware Partner Ecosystem Profile — Verdict](https://www.verdict.co.uk/basware-partner-ecosystem-profile/)

**Product Domain (AP Automation, Invoice Matching, E-Invoicing)**
- [What is invoice matching? — Basware](https://www.basware.com/en/glossary/what-is-invoice-matching)
- [Touchless Invoice Processing — Basware](https://www.basware.com/en/solutions/ap-automation/touchless-invoice-processing/)
- [SmartCoding — Basware](https://www.basware.com/en/solutions/ap-automation/smartcoding)
- [E-Invoicing Glossary — GoRoute](https://goroute.ai/glossary.html)
- [Peppol Glossary](https://peppolvalidator.com/glossary)

**Corporate / SaaS KPI Domain (ARR, Gross Margin, Retention)**
- [Datarails — GRR vs NRR](https://www.datarails.com/grr-vs-nrr/)
- [CloudZero — SaaS Gross Margin](https://www.cloudzero.com/blog/saas-gross-margin/)
- [Chargebee — SaaS Gross Margin](https://www.chargebee.com/resources/glossaries/saas-gross-margin/)
- [Basware Financial Statements Bulletin January–December 2021 — Inderes](https://www.inderes.fi/en/releases/basware-financial-statements-bulletin-januarydecember-2021-order-intake-back-to-growth-net-sales-and-ebit-in-line-with-expectations) (Cloud ARR order intake, gross/net renewal rate, new logo intake, reseller share of order intake)
- [Basware Marks 40 Years of Innovation with $10.1 Trillion Spend Managed](https://news.basware.com/en/basware-marks-40-years-of-innovation-with-10.1-trillion-spend-managed-through-its-platform) (company scale, 2B+ invoices, Feb 2025)
- [Basware Revenue 2025 — GetLatka](https://getlatka.com/companies/basware.com) (third-party estimate, unconfirmed — background color only)
