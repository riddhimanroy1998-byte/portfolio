# Riddhiman Roy — GTM Engineering, Automation & AI Systems

I find the costliest manual work buried inside messy operational systems and turn it into automation that runs itself. Two domains. **GTM engineering** for B2B SaaS revenue teams: AI agents against the CRM stack, enrichment pipelines, and the data-integrity rules that decide whether any of it can be trusted. And **automation and data infrastructure** at a European B2B e-commerce marketplace operating across 100+ retail platforms. n8n, LLM pipelines, SQL, prompt to production.

![n8n](https://img.shields.io/badge/n8n-workflow_orchestration-EA4B71)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-schema_&_SQL-336791)
![Snowflake](https://img.shields.io/badge/Snowflake-SQL-29B5E8)
![OpenAI](https://img.shields.io/badge/OpenAI_API-LLM_pipelines-412991)
![Claude](https://img.shields.io/badge/Claude-MCP_&_agents-D97757)
![Metabase](https://img.shields.io/badge/Metabase-dashboards-509EE3)
![HubSpot](https://img.shields.io/badge/HubSpot-sequences_&_sync-FF7A59)
![Salesforce](https://img.shields.io/badge/Salesforce-SOQL_&_data_model-00A1E0)

> Confidentiality: my employers and their partners are anonymized, architecture is described at the pattern level, and no internal systems, CRM schema or colleague names are given. In the e-commerce and HR sections every quantitative figure is real. The GTM engineering sections describe method and architecture rather than outcomes, because those numbers belong on a CV and not in a public repo. The workflow files are sanitized illustrations, not drop-in imports.

---

## Live demos

Three interactive tools, each running on synthetic data in the browser with no backend. Open and click around.

| Tool | What it shows |
| --- | --- |
| [**Shortlist** — AI CV screening](https://riddhimanroy1998-byte.github.io/portfolio/cv-screener.html) | Parses a role's requirements, scores every applicant with a weighted model, gates at a 90% confidence bar, returns a ranked shortlist |
| [**Coverage** — GMV-weighted compliance](https://riddhimanroy1998-byte.github.io/portfolio/coverage.html) | Ranks brands by revenue so filling the few that matter clears 80% of exposure fast (modeled on EU GPSR) |
| [**HR automations** — process & architecture](https://riddhimanroy1998-byte.github.io/portfolio/hr-workflows.html) | Leave and payroll preparation plus transit-pass reimbursement, as case studies with flow diagrams |

---

## How I build

A few principles that show up in every project below.

1. Kill the work, do not process it. When a request keeps landing in the queue, I build the source of the request out of existence rather than handling it faster.
2. Match the infrastructure to the task. The same translation job moved from a single LLM call to parallel pipelines to a batch API once I saw the workload was async-tolerant, for a flat 50% cost cut on identical output.
3. Ship v1, then harden with observability. I log found and unfound, pass and fail, from day one, because that is what makes the real failure mode visible later.
4. Confidence-gate the AI. High-confidence outputs auto-pass, borderline cases route to a human. A probabilistic tool should never auto-publish into a near-zero-tolerance step.

---

## Projects

### GTM engineering

Built at a cybersecurity SaaS vendor (digital risk protection / external attack surface
management), selling into security leadership at enterprise accounts. These write-ups are about
method: the root cause, the decision rules, and the tradeoffs.

| # | Project | What it proves |
| --- | --- | --- |
| G1 | Auditing an account qualification process | Finding the bug in a process everyone was using correctly |
| G2 | Two-source enrichment waterfall | Record linkage under conflicting data, and refusing to guess |
| G3 | Eight agents against one stack | Grounding over generation, so output stays checkable |

Full write-ups in [`gtm-engineering-case-studies.md`](gtm-engineering-case-studies.md).

### Automation & data

| # | Project | What it proves | Links |
| --- | --- | --- | --- |
| 1 | Catalogue taxonomy standardization | Designing the data foundation a million products depend on | [code](catalogue-creation-automatic-addition_sanitized.json) |
| 2 | Self-serve export tool | Removing a recurring cross-team bottleneck at the input layer | [code](demander-export-self-serve_sanitized.json) |
| 3 | Multilingual product-page generation | LLM pipelines at catalogue scale, iterated for cost | [code](demand-generation-demander-csv_sanitized.json) |
| 4 | Image-ingestion pipeline | Resilient design with a two-tier fallback | — |
| 5 | CV screening and shortlisting | AI-native recruiting tool, prompt to production | [live demo](https://riddhimanroy1998-byte.github.io/portfolio/cv-screener.html) |
| 6 | Leave and payroll automation | People-ops admin reduced to one click | [live demo](https://riddhimanroy1998-byte.github.io/portfolio/hr-workflows.html) |
| 7 | Transit-pass reimbursement | Per-profile statutory logic, half a day to zero | [live demo](https://riddhimanroy1998-byte.github.io/portfolio/hr-workflows.html) |
| 8 | GMV-weighted GPSR prioritization | Using revenue weighting to make a regulation tractable | [live demo](https://riddhimanroy1998-byte.github.io/portfolio/coverage.html) |
| 9 | Demander-marketplace product-page generation | A five-stage LLM pipeline that turns a catalogue product into a marketplace-ready listing, with per-demander output formats | [deep dive](pipeline-deep-dives.md#product-page-generation-for-demander-marketplaces) |
| 10 | Catalogue creation and maintenance | A continuous, self-healing, idempotent system that builds a 1M+ product catalogue and backfills every missing attribute | [deep dive](pipeline-deep-dives.md#catalogue-creation) |
| 11 | Automated product-match verification | Parallel, attribute-by-attribute LLM comparison that decides whether two catalogue items are the same product | [deep dive](pipeline-deep-dives.md#automated-product-match-verification) |

Detailed write-ups with CV bullet points are in [`portfolio-case-studies.md`](portfolio-case-studies.md).

Deeper per-workflow pipeline walkthroughs, with a diagram for every stage, are in [`pipeline-deep-dives.md`](pipeline-deep-dives.md).
---

### GTM engineering & revenue systems

#### G1. Auditing an account qualification process instead of running it

A sales team ran a top-tier prospect classification with a hard cap on active accounts per rep,
which is what makes the classification work and also what makes a wrong pick expensive. I was
handed the qualification method and a patch to apply it to. I checked whether it worked first.

Two failures, both discarding good accounts while letting already-worked ones through. The
exclusion rule disqualified any account with a prior CRM touch, which silently eliminated the
sectors with the longest sales history and skewed the output by vertical. And the CRM check
could not see what it was being used to detect: activity logged against a contact is stored in
`WhoId`, an account-scoped query reads `WhatId`, so contact-level outreach never appears in an
account-level check. The query returned a clean zero on accounts that had been emailed months
earlier. Nobody was screening carelessly; they were screening correctly against a field that
structurally could not answer the question.

I reframed the exclusion rule around an asymmetric cost: with very few slots per rep, a false
positive burns a quarter and a false negative quietly loses a good account. Three narrow
hard-excludes, everything else stays in play carrying a flag. Verification reads subject lines
rather than dates, because the distinction that decides the outcome is a real two-way exchange
versus a one-way automated sequence, and recency cannot tell those apart. A dormant touchpoint
became a revival angle instead of a rejection.

Stack: Salesforce (SOQL, object model), HubSpot, a four-tier persona architecture, a
reason-coded rejected log.

```mermaid
flowchart TD
    A[Candidate account] --> B[Check rejected log]
    B -->|Previously dropped| Z[Stop, do not re-research]
    B -->|Not seen| C[Account-level CRM check]
    C --> D[Contact-level activity pull via WhoId]
    D --> E[Read subject lines, not just dates]
    E --> F{Two-way exchange?}
    F -->|Yes, open stage, recent| G[Hard exclude: actively worked]
    F -->|One-way sequence only| H[Revival candidate]
    H --> I{DNC or existing customer?}
    I -->|Yes| G
    I -->|No| J[Apply qualifying lenses]
    J --> K[Map contacts to persona tiers]
    K --> L[Fixed-format brief]
    G --> M[Log with reason code]
```

#### G2. A two-source enrichment waterfall that refuses to guess

No contact-data provider is complete, so a second one gets consulted whenever the first comes up
short. The symptom looks like a coverage problem. It is a trust problem: two providers return the
same person with a different phone number, title or employer, one record is stale, and nothing in
either tells you which. Absent a rule the practical default takes whichever field is populated,
which prefers available data over correct data. The failure is not an empty list, it is a
confident one.

I set one provider as the source of record and the second as fallback only, because a blend has
no tiebreaker and a waterfall does. Before any fallback-sourced phone number is trusted, the
company and title on that record have to match the primary. A phone number cannot be validated
on its own terms, only through the identity attached to it, so the cheap fields become the
guardrail on the expensive one. Conflicting data is never silently merged; it is flagged and left
to a human. Contacts are capped per account rather than pulled exhaustively, because a capped
list forces a decision about who matters instead of deferring it to the rep at dial time.

Stack: two commercial data providers in a primary/fallback waterfall, cross-field validation,
tiered buyer profiling by buying role, cross-vendor industry taxonomy mapping.

```mermaid
flowchart TD
    A[Target account] --> B[Primary provider query]
    B --> C{Fields complete?}
    C -->|Yes| H[Accept record]
    C -->|No| D[Fallback provider query]
    D --> E{Primary left field blank?}
    E -->|Yes| F[Low risk: accept fallback value]
    E -->|No, values conflict| G[Cross-check company + title]
    G -->|Match| F
    G -->|Mismatch| X[Flag for human review, do not merge]
    F --> H
    H --> I[Assign buyer tier]
    I --> J[Apply per-account contact cap]
    J --> K[Call-ready list: tier, timezone, hook]
```

#### G3. Eight agents against one stack

An SDR's week is mostly not selling. It is research, CRM hygiene, write-ups, and moving
information between systems that do not talk to each other. The usual response is a longer
checklist; I built tooling against the stack instead.

The design rule is grounding over generation. Each agent reads from a source the team already
trusts rather than from the model's own knowledge, because an account briefing invented by a
model is worse than no briefing, as someone will act on it. The battlecard builder is grounded in
internal positioning content and CRM win/loss history. The outreach generator learns a rep's
voice from their own sent mail, since a rep will not send something that does not sound like
them, which makes voice-matching a precondition for adoption rather than a finishing touch. The
CRM sync dedupes by verified domain rather than company name, because company name is free text
and will cheerfully create a second record for the same company under a different spelling.

The hardest part of each agent was not the model. It was deciding which existing artefact counted
as ground truth.

Stack: Claude API and Claude Skills, HubSpot, Salesforce, Gmail, Slack, Google Drive, LinkedIn
Sales Navigator.

```mermaid
flowchart TD
    A[Trusted source of record] --> B{Which artefact is ground truth?}
    B -->|Internal content + win/loss| C[Battlecard builder]
    B -->|Rep's own sent mail| D[Outreach generator]
    B -->|CRM + engagement + web signals| E[Account summary]
    B -->|Raw call notes| F[AE qualification brief]
    B -->|Verified domain| G[CRM dedupe sync]
    C --> H[Checkable output]
    D --> H
    E --> H
    F --> H
    G --> H
```

---

### Data & catalogue pipelines

#### 1. Catalogue taxonomy standardization

A roughly 1M-product catalogue had grown with no consistent classification: mixed languages, no hierarchy, no canonical list. That broke querying, enrichment, and any mapping onto partner marketplaces, and the work had sat unowned in the backlog. I studied how major retail marketplaces classify products, designed a canonical three-level taxonomy, and modeled the whole hierarchy as a single self-referencing reference table. LLM-driven n8n workflows then backfilled the existing million products and normalize every new one on ingestion. The catalogue reached 100% standardization on levels 1 and 2 and 95 to 96% on level 3, and the taxonomy still governs catalogue creation today.

The judgment was in the iteration: testing a category as top-level then correctly demoting it, and pacing the million-product rollout over weeks to control token cost and workflow stability.

Stack: n8n, PostgreSQL (self-referencing schema, idempotent upserts), LLM standardization. Code sample: [`catalogue-creation-automatic-addition_sanitized.json`](catalogue-creation-automatic-addition_sanitized.json), the four-step idempotent insertion job.

```mermaid
flowchart TD
    A[Research partner-marketplace taxonomies] --> B[Design canonical 3-level taxonomy]
    B --> C[Self-referencing reference table with parent id]
    C --> D[LLM standardization workflow]
    D --> E[Backfill existing ~1M products]
    D --> F[Normalize every new product on ingestion]
```

#### 2. Self-serve export tool

The no-code team was a bottleneck for demander exports, the showcase product files the sales team sent to prospective partners. Each request meant a ticket, a roughly three-day queue, a fragile shared notebook, and manual category cleanup. I built a self-serve n8n form: the sales team picks from the catalogue's canonical category values, which removes category mismatch at the input layer, and the workflow queries the catalogue directly and returns a ready CSV. Turnaround fell from about three days to under 20 minutes, the ticket queue for this request type disappeared entirely, and it ran with zero maintenance after launch.

Stack: n8n (form trigger, parametrized Postgres queries, CSV delivery), PostgreSQL. Code sample: [`demander-export-self-serve_sanitized.json`](demander-export-self-serve_sanitized.json).

```mermaid
flowchart TD
    A[Sales user opens self-serve form] --> B[Pick canonical category values]
    B --> C[Parametrized query against catalogue]
    C --> D[Format rows to export schema]
    D --> E[Convert to CSV]
    E --> F[Deliver file to user]
```

#### 3. Multilingual product-page generation

A large European marketplace client wanted its catalogue opened in five new country-shops, each needing product data in its own language. I built an n8n pipeline that filtered the client's full file to in-stock items by EAN, translated every field into all five languages with an LLM, and auto-generated one localized CSV per shop. The first single-LLM version hallucinated, so I re-architected it into dedicated per-language pipelines running parallel batched calls. Months later I moved the translation step onto the provider's batch API, same output, for a flat 50% token-cost cut. Five country-shops went live within about five days of starting.

Three deliberate iterations: single LLM, then per-language parallel pipelines, then batch API once I recognized the workload was async-tolerant.

Stack: n8n, LLM API (GPT-4o-mini) and its batch API, EAN filtering, CSV ingestion. Related code sample: [`demand-generation-demander-csv_sanitized.json`](demand-generation-demander-csv_sanitized.json), the scheduled incremental per-demander CSV builder that pushes generated rows downstream.

```mermaid
flowchart TD
    A[Client full product file] --> B[Filter to in-stock items by EAN]
    B --> C[Per-language translation pipelines]
    C --> D[Parallel LLM calls in batches]
    D --> E[One localized CSV per country-shop]
    E --> F[Marketplace ingestion]
    D --> G[v3: provider batch API, -50% cost]
```

#### 4. Image-ingestion pipeline

After sourcing produced verified image URLs, there was no fast, safe way to get images into the system and onto the right product. I built an n8n workflow that downloads each image, optionally removes the background via an image API, self-hosts it in object storage, links it to the product, and logs every image as found or unfound so runs are trackable and re-runnable. That observability surfaced a failure mode: a meaningful share of downloads were blocked even with spoofed headers. I designed a two-tier fallback that reroutes blocked fetches through an edge-proxy worker, taking the tool from frequently blocked to reliably capturing images. It is still in production.

I built the observability in from v1, which is what made the failure visible, then scoped the edge worker for a teammate while owning the architecture and integration.

Stack: n8n, object storage (S3), background-removal API, Cloudflare Workers (edge proxy).

```mermaid
flowchart TD
    A[Approved image URLs] --> B[Direct download with browser headers]
    B -->|success| D[Optional background removal]
    B -->|blocked| C[Edge-proxy worker fetch]
    C --> D
    D --> E[Self-host in object storage]
    E --> F[Link to product and log found/unfound]
```

---

### People & HR systems

#### 5. CV screening and shortlisting → [live demo](https://riddhimanroy1998-byte.github.io/portfolio/cv-screener.html)

A three-person People team screened inbound CVs by hand, around ten an hour, which capped how many applicants got a fair read. I built an n8n workflow that extracts each role's requirements, scores every applicant with a structured OpenAI API prompt returning a JSON match score, gates at 90%, and surfaces a ranked shortlist. High-confidence matches auto-surface; borderline cases route to a human. Screening went from about ten CVs an hour to hundreds of applications ranked automatically each day across open roles.

Stack: n8n, OpenAI API (structured JSON output), confidence gating.

```mermaid
flowchart TD
    A[Open role: job description] --> B[Extract required and nice-to-have skills]
    C[Applicant CVs] --> D[OpenAI API: structured prompt, JSON match score]
    B --> D
    D --> E{Score at or above 90%?}
    E -->|yes| F[Auto-surface to ranked shortlist]
    E -->|borderline| G[Route to human review]
```

#### 6. Leave and payroll automation → [live demo](https://riddhimanroy1998-byte.github.io/portfolio/hr-workflows.html)

Every leave request was logged by hand, and at month end the team recalculated each person's days taken, entitlement, and remaining balance, across 80 employees and several leave types. I built an approval-triggered system: a Notion form routes to the team lead, and on approval an n8n workflow logs it and updates a live per-person balance across paid, unpaid, paternity, and maternity leave. At month end, one action compiles every balance into a single CSV formatted for the payroll provider.

```mermaid
flowchart TD
    A[Employee submits leave via Notion form] --> B[Team lead approves]
    B --> C[n8n logs request, updates live balance]
    C --> D[Per-person ledger: paid, unpaid, paternity, maternity]
    E[Month-end trigger] --> F[One-click export]
    D --> F
    F --> G[CSV formatted for payroll provider]
```

#### 7. Transit-pass reimbursement → [live demo](https://riddhimanroy1998-byte.github.io/portfolio/hr-workflows.html)

French employers reimburse 50% of an employee's public-transit pass. Each month the three-person team blocked half a day to open every pass and compute the payout by hand, and the rules differ by profile (student, intern, standard). I built a form where each employee submits their pass once; an n8n workflow reads it, applies the right rule for that profile, computes the reimbursement, and registers it for the pay run. The recurring half-day dropped to zero.

```mermaid
flowchart TD
    A[Employee submits pass once] --> B[n8n reads and classifies profile]
    B --> C{Profile type}
    C -->|student| D[Apply student rule]
    C -->|intern| E[Apply intern rule]
    C -->|standard| F[Apply standard rule]
    D --> G[Compute 50% reimbursement]
    E --> G
    F --> G
    G --> H[Register for pay run]
```

---

### Compliance & influence

#### 8. GMV-weighted GPSR prioritization → [live demo](https://riddhimanroy1998-byte.github.io/portfolio/coverage.html)

When the EU's General Product Safety Regulation became enforceable, marketplaces began blocking listings that lacked full manufacturer and EU-responsible-person data, and coverage was patchy and maintained by hand. Working through thousands of brands alphabetically would waste effort on tiny ones, so I ranked every brand by trailing six-month GMV and worked top-down: cover the brands that carry 80% of revenue and you unblock almost all sellable business while touching a fraction of the catalogue. An automated research step filled most brands; the long tail stayed manual by design. A live dashboard tracked coverage against the 80% target. Manual research dropped from about three hours a week to roughly five minutes, at an automated hit rate near 86%.

Stack: n8n, MCP server, Claude Desktop (web research), Snowflake, Metabase, SQL.

```mermaid
flowchart TD
    A[Warehouse: every brand and 6-month GMV] --> B[Rank brands by revenue]
    B --> C[Automated research: MCP server + Claude Desktop web access]
    C --> D[Structured GPSR fields: manufacturer and EU responsible person]
    D -->|~86% auto-filled| E[Write to database]
    D -->|long tail| F[Manual queue by design]
    E --> G[Coverage dashboard: share of GMV vs 80% target]
```

---

## Code samples

Sanitized, genericized n8n workflows are the three .json files in this repo. Credentials, internal hostnames, schema and table names, and client identifiers are removed or replaced with placeholders. The control flow and SQL and JS logic are faithful to the original; the names are not. These are illustrations, not drop-in imports.

- [`demander-export-self-serve_sanitized.json`](demander-export-self-serve_sanitized.json) — form to taxonomy-constrained query to CSV export.
- [`demand-generation-demander-csv_sanitized.json`](demand-generation-demander-csv_sanitized.json) — scheduled, incremental per-demander CSV builder.
- [`catalogue-creation-automatic-addition_sanitized.json`](catalogue-creation-automatic-addition_sanitized.json) — four-step idempotent catalogue insertion.

---

## Contact

Riddhiman Roy · Paris, France · riddhimanroy1998@gmail.com

CV available on request.
