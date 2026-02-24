# France E-Invoicing Content Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Produce 8 Markdown files covering France's mandatory B2B e-invoicing regime across four progressive levels (Intro, Intermediate, Advanced, Product), split by inbound (AP) and outbound (AR) flows, structured for use in slide presentations.

**Architecture:** Process-first organization — inbound and outbound are the primary axis; each level assumes the reader has absorbed the previous level. Product level is stub-heavy on first pass with `[STUB: topic]` markers. All regulatory content reflects the current 2026 state of the mandate (PA terminology, Y-model, PLF 2026).

**Reference:** Design doc at `docs/plans/2026-02-24-france-einvoicing-design.md`

**Tech Stack:** Markdown, presentation-friendly heading hierarchy (H1 = deck title, H2 = section/chapter, H3 = individual slide)

---

## Regulatory Reference (read before writing any file)

These facts must be consistent across ALL files:

- **PA** (Plateforme Agréée) = the current term for approved invoice exchange platforms (formerly called PDP — do not use PDP)
- **Y-model**: invoices travel PA → PA directly; PPF is NOT an exchange hub
- **PPF** (Portail Public de Facturation): maintains the Annuaire (national directory) + relays data to DGFiP
- **Annuaire**: central directory where each business registers its PA; used to route invoices
- **E-invoicing**: applies to domestic B2B transactions — structured invoice via PA
- **E-reporting**: separate obligation covering B2C, cross-border B2B, and payment data for B2B/B2C/B2G
- **Formats**: Factur-X (hybrid PDF+XML), UBL 2.1 (pure XML), CII (pure XML) — all EN 16931 + CIUS-FR
- **DGFiP**: French tax authority; also the French Peppol authority
- **Chorus Pro**: B2G only — unchanged
- **Sept 1, 2026**: All businesses must RECEIVE; GE + ETI must SEND + e-report
- **Sept 1, 2027**: SMEs + micro-enterprises must SEND + e-report
- **GE**: 5,000+ employees; >€1.5B turnover or >€2B balance sheet
- **ETI**: 250–5,000 employees; <€1.5B turnover or <€2B balance sheet
- **SME/micro**: below ETI thresholds
- **PLF 2026**: Finance Bill adopted February 2, 2026 — legal anchor
- **101 PAs**: listed January 17, 2026

---

## Slide Structure Convention

Every file must follow this heading convention so slides map cleanly:

```
# [File Title]           ← deck/chapter title slide
## [Section Name]        ← section divider slide
### [Slide Title]        ← individual slide
[body content]           ← slide body (bullets, tables, short prose)
```

Keep slide bodies concise: 3–6 bullets, or a small table, or 2–3 short paragraphs max. If a topic needs more space, break it into two slides (two H3s).

---

## Task 1: Create directory structure

**Files:**
- Create: `france-einvoicing/intro/inbound.md` (empty)
- Create: `france-einvoicing/intro/outbound.md` (empty)
- Create: `france-einvoicing/intermediate/inbound.md` (empty)
- Create: `france-einvoicing/intermediate/outbound.md` (empty)
- Create: `france-einvoicing/advanced/inbound.md` (empty)
- Create: `france-einvoicing/advanced/outbound.md` (empty)
- Create: `france-einvoicing/product/inbound.md` (empty)
- Create: `france-einvoicing/product/outbound.md` (empty)

**Step 1: Create all directories**

```bash
mkdir -p france-einvoicing/intro france-einvoicing/intermediate france-einvoicing/advanced france-einvoicing/product
```

**Step 2: Verify structure**

```bash
find france-einvoicing -type d
```

Expected output:
```
france-einvoicing/intro
france-einvoicing/intermediate
france-einvoicing/advanced
france-einvoicing/product
```

**Step 3: Commit**

```bash
git add france-einvoicing/
git commit -m "chore: scaffold france e-invoicing content directory structure"
```

---

## Task 2: Write intro/inbound.md

**File:** `france-einvoicing/intro/inbound.md`

**Audience assumption:** Reader knows nothing about e-invoicing or the French mandate.

**Required coverage (acceptance criteria):**
- [ ] What e-invoicing is (simple definition, contrast with PDF/paper)
- [ ] Why France is mandating it (VAT fraud prevention, modernisation)
- [ ] What "receiving an e-invoice" means in everyday terms
- [ ] Who is affected and when (all businesses must receive by Sept 2026)
- [ ] What a business needs to do to get ready — high level (choose a PA, register in Annuaire)
- [ ] Key term introduced: PA (Plateforme Agréée)

**Step 1: Write the file using this structure**

```markdown
# Receiving E-Invoices in France — Introduction

## What Is E-Invoicing?

### From Paper to Structured Data
[3–4 bullets: what e-invoicing is, contrast with PDF/email, why structured data matters]

### Why Is France Mandating It?
[3–4 bullets: VAT gap, fraud prevention, EU directive, modernisation]

## The French Mandate at a Glance

### Who Is Affected?
[Table: company size category / definition / must receive by / must send by]

### What Does "Receiving an E-Invoice" Mean?
[3–4 bullets: a structured document arrives via an approved platform, not email;
your system must be able to accept it; you need a registered platform (PA)]

## What Do You Need to Do?

### Getting Ready to Receive — The Short Version
[Numbered list: 1) Choose a PA, 2) Register in the national directory (Annuaire),
3) Ensure your ERP/AP system can accept the structured formats]

### Key Term: PA (Plateforme Agréée)
[2–3 bullets: what a PA is, why you need one, that 101 are approved as of Jan 2026]
```

**Step 2: Verify acceptance criteria**

Read through the written file and check each item in the acceptance criteria list above. Every box must be checked before proceeding.

**Step 3: Commit**

```bash
git add france-einvoicing/intro/inbound.md
git commit -m "feat: add intro/inbound.md — receiving e-invoices introduction"
```

---

## Task 3: Write intro/outbound.md

**File:** `france-einvoicing/intro/outbound.md`

**Audience assumption:** Reader has read intro/inbound.md and understands what e-invoicing is.

**Required coverage (acceptance criteria):**
- [ ] What "sending an e-invoice" means in everyday terms
- [ ] Who must send and by when (GE + ETI: Sept 2026; SME: Sept 2027)
- [ ] Clear definition of GE, ETI, SME/micro size thresholds
- [ ] What e-reporting is and how it differs from e-invoicing
- [ ] High-level steps to prepare for outbound (choose PA, connect ERP, configure formats)
- [ ] Why this is different from just sending a PDF by email

**Step 1: Write the file using this structure**

```markdown
# Sending E-Invoices in France — Introduction

## Sending Under the Mandate

### What Does "Sending an E-Invoice" Mean?
[3–4 bullets: structured format, sent via PA, not email; PA routes to recipient's PA;
government gets a copy of the data — not the full invoice]

### Who Must Send, and When?
[Table: GE definition + Sept 2026 / ETI definition + Sept 2026 / SME + Sept 2027 /
micro-enterprise + Sept 2027; note: ALL must receive by Sept 2026]

## E-Reporting: The Other Obligation

### E-Invoicing vs. E-Reporting — What's the Difference?
[Table: e-invoicing (domestic B2B, full structured invoice via PA) vs.
e-reporting (B2C, cross-border B2B, payment data — transaction data only to DGFiP)]

### Why Does E-Reporting Matter for Senders?
[3–4 bullets: even if your customer doesn't receive via PA (B2C, foreign),
you still have a reporting obligation to DGFiP]

## What Do You Need to Do?

### Getting Ready to Send — The Short Version
[Numbered list: 1) Know which category your business falls into,
2) Choose and onboard a PA, 3) Connect your ERP/AR system to the PA,
4) Map your invoice data to the required formats and fields]

### The Timeline Is Locked
[2–3 bullets: April 2025 postponement rejected; Sept 2026 is confirmed;
PLF 2026 adopted Feb 2026 as legal anchor — time to act now]
```

**Step 2: Verify acceptance criteria**

Check each item in the acceptance criteria list. Every box must be checked.

**Step 3: Commit**

```bash
git add france-einvoicing/intro/outbound.md
git commit -m "feat: add intro/outbound.md — sending e-invoices introduction"
```

---

## Task 4: Write intermediate/inbound.md

**File:** `france-einvoicing/intermediate/inbound.md`

**Audience assumption:** Reader understands what e-invoicing is and that a PA is required. Now learning how the ecosystem actually works.

**Required coverage (acceptance criteria):**
- [ ] The Y-model explained: PA-to-PA exchange, PPF as directory only
- [ ] The Annuaire: what it is, how routing works, how a business registers
- [ ] Step-by-step journey of an inbound invoice from sender's PA to recipient's system
- [ ] E-reporting obligations on the receiving side (payment data reporting)
- [ ] Format overview: Factur-X vs UBL 2.1 vs CII — when each is used, what they look like
- [ ] Distinction between e-invoicing and e-reporting clearly established

**Step 1: Write the file using this structure**

```markdown
# Receiving E-Invoices in France — Intermediate

## The E-Invoicing Ecosystem

### The Three Pillars: PA, PPF, DGFiP
[Diagram description or table: PA (exchange platform) / PPF (directory + data hub) /
DGFiP (tax authority receiving e-reporting data)]

### The Y-Model: How Invoices Actually Travel
[Describe the Y-model flow with clear steps:
Sender ERP → Sender's PA → [Annuaire lookup] → Recipient's PA → Recipient ERP;
PPF sits off to the side receiving data copies — not in the invoice path]

### The Annuaire: The Routing Directory
[3–4 bullets: what it is, who maintains it (PPF), how businesses register their PA,
how a PA looks up the recipient's PA before sending]

## The Inbound Invoice Journey

### Step-by-Step: From Sender to Your System
[Numbered flow: 1) Sender creates structured invoice in ERP,
2) Sender's PA validates and routes via Annuaire lookup,
3) Invoice delivered to your PA,
4) Your PA validates and makes available to your ERP/AP system,
5) Status acknowledgements flow back to sender]

### Invoice Status Lifecycle (Overview)
[Table or list: Deposited / Received / Rejected / Under processing / Paid —
explain what each means at a high level]

## Formats You Will Receive

### Factur-X, UBL 2.1, and CII — What Are They?
[Table: format name / type (hybrid/pure XML) / typical use case / human-readable?]

### The French Standard: EN 16931 + CIUS-FR
[2–3 bullets: all formats must conform to the European standard EN 16931;
CIUS-FR adds French-specific mandatory fields; your PA handles format validation]

## Your Obligations on the Receiving Side

### E-Reporting: Payment Data
[3–4 bullets: when you pay a B2B invoice, you must report payment data to DGFiP;
this applies regardless of the transaction type; your PA handles submission;
timeline mirrors the sending obligation by company size]
```

**Step 2: Verify acceptance criteria**

Check each item. Every box must be checked.

**Step 3: Commit**

```bash
git add france-einvoicing/intermediate/inbound.md
git commit -m "feat: add intermediate/inbound.md — ecosystem and inbound flow"
```

---

## Task 5: Write intermediate/outbound.md

**File:** `france-einvoicing/intermediate/outbound.md`

**Audience assumption:** Reader understands the Y-model and PA ecosystem from intermediate/inbound.md.

**Required coverage (acceptance criteria):**
- [ ] How to choose a PA (what to look for, that 101 are approved)
- [ ] Full outbound Y-model flow: ERP → PA → Annuaire → recipient PA → recipient
- [ ] E-reporting for B2C transactions
- [ ] E-reporting for cross-border B2B transactions
- [ ] Payment data reporting obligation for outbound senders
- [ ] Format choice: when to use Factur-X vs UBL vs CII
- [ ] CIUS-FR: what it adds and why it matters for outbound

**Step 1: Write the file using this structure**

```markdown
# Sending E-Invoices in France — Intermediate

## Choosing Your PA

### What Is a PA and What Must It Do?
[3–4 bullets: certified by DGFiP; handles sending, receiving, and e-reporting;
validates invoices against business rules; routes via Annuaire; 101 approved as of Jan 2026]

### How to Choose the Right PA
[Table or bullets: key criteria — format support, ERP connectivity,
e-reporting capability, SLA, cost model, existing relationship]

## The Outbound Invoice Journey

### Step-by-Step: From Your ERP to Your Customer
[Numbered flow:
1) Invoice created in ERP/AR system,
2) Transmitted to your PA (API or EDI),
3) PA validates format + CIUS-FR fields,
4) PA queries Annuaire for recipient's PA,
5) PA delivers to recipient's PA,
6) Status updates returned to your system]

### What Your PA Does Before Sending
[3–4 bullets: format validation, schematron rules check,
Annuaire lookup, secure transmission, acknowledgement handling]

## E-Reporting Obligations

### Domestic B2B: Covered by E-Invoicing
[1–2 bullets: when you send a domestic B2B e-invoice via PA, e-reporting is handled automatically]

### B2C Transactions: What You Must Report
[3–4 bullets: for sales to consumers, no invoice goes via PA;
instead, transaction data is submitted to DGFiP;
includes amount, VAT, date; your PA handles submission]

### Cross-Border B2B: What You Must Report
[3–4 bullets: invoices to foreign businesses don't go via the PA network;
transaction data must still be e-reported to DGFiP;
same data fields as B2C e-reporting]

### Payment Data Reporting
[3–4 bullets: when a customer pays a B2B invoice, you report payment receipt to DGFiP;
applies to B2B, B2C, and B2G; your PA handles the submission]

## Formats and French Requirements

### Choosing a Format for Outbound
[Table: Factur-X (PDF+XML, human-readable, good for mixed audiences) /
UBL 2.1 (pure XML, machine-to-machine) /
CII (pure XML, alternative to UBL)]

### CIUS-FR: The French Extension
[3–4 bullets: what CIUS-FR is (French-specific mandatory fields on top of EN 16931);
examples of CIUS-FR fields (SIREN number, payment terms specifics);
your ERP must map to these fields; your PA validates them]
```

**Step 2: Verify acceptance criteria**

Check each item. Every box must be checked.

**Step 3: Commit**

```bash
git add france-einvoicing/intermediate/outbound.md
git commit -m "feat: add intermediate/outbound.md — outbound flow, e-reporting, formats"
```

---

## Task 6: Write advanced/inbound.md

**File:** `france-einvoicing/advanced/inbound.md`

**Audience assumption:** Reader understands the ecosystem and flows. Now needs technical and regulatory depth for implementation or compliance work.

**Required coverage (acceptance criteria):**
- [ ] DGFiP External Specifications v3.1 as the technical reference — what it contains
- [ ] Full invoice status lifecycle with technical state names
- [ ] Validation rules: schematron, 50+ business rules, what failure looks like
- [ ] Error handling and rejection flows — what happens when an invoice is rejected
- [ ] Peppol interoperability: DGFiP as French Peppol authority, implications
- [ ] Connectivity options: AS2/AS4 (EDI) and REST API
- [ ] Archiving obligations: retention period, format requirements

**Step 1: Write the file using this structure**

```markdown
# Receiving E-Invoices in France — Advanced

## Technical Reference

### DGFiP External Specifications v3.1
[3–4 bullets: what the spec covers (API endpoints, status codes, data schemas,
business rules); where to find it; version history note (v3.0 introduced PA terminology);
this is the authoritative source — not vendor docs]

## Invoice Validation

### Schematron Validation: The 50+ Business Rules
[3–4 bullets: what schematron validation is; when it runs (at PA on receipt);
examples of rules (mandatory fields, VAT calculation checks, SIREN format);
your PA returns a structured error if validation fails]

### What a Validation Failure Looks Like
[Table or bullets: error code / meaning / who handles it (PA vs. your system)]

## Invoice Status Lifecycle

### Status States: Full Technical View
[Table: status name / trigger / who sets it / what action is required]
[States to cover: Deposited, Received, Rejected, Refused, Acknowledged,
Under Query, Cancelled, Paid]

### Status Flow Diagram (Description)
[Describe the state machine: happy path and rejection/refusal paths]

## Error Handling and Disputes

### Rejection vs. Refusal — What Is the Difference?
[Table: Rejection (technical — fails validation, never legally an invoice) vs.
Refusal (business — valid invoice, recipient disputes it)]

### Handling a Rejected Inbound Invoice
[Numbered steps: notification from PA, identify error, notify sender,
sender resubmits corrected invoice]

## Connectivity and Interoperability

### AS2/AS4 vs. REST API — When to Use Which
[Table: AS2/AS4 (EDI, high-volume, existing EDI infrastructure) vs.
REST API (modern, event-driven, lower setup overhead)]

### Peppol Interoperability
[3–4 bullets: DGFiP is the French Peppol authority;
cross-border invoices from Peppol network can reach French recipients via PA;
Peppol BIS 3.0 is compatible with UBL 2.1;
implications for multinational supply chains]

## Archiving Obligations

### Legal Archiving Requirements
[3–4 bullets: invoices must be archived for 10 years (French tax law);
archive must preserve the original structured format plus the human-readable version;
PA may offer archiving as a service — verify in your PA contract;
DGFiP can request access during audit]
```

**Step 2: Verify acceptance criteria**

Check each item. Every box must be checked.

**Step 3: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "feat: add advanced/inbound.md — technical depth for inbound"
```

---

## Task 7: Write advanced/outbound.md

**File:** `france-einvoicing/advanced/outbound.md`

**Audience assumption:** Reader understands the ecosystem and flows at a deep level. Needs full technical and legal detail for outbound implementation.

**Required coverage (acceptance criteria):**
- [ ] Outbound technical specs: what the DGFiP spec requires of senders
- [ ] CIUS-FR mandatory fields: complete list with data types and sources
- [ ] Schematron validation on outbound: what your PA checks before accepting
- [ ] E-reporting data structure: exactly what fields are required for each type
- [ ] Payment data reporting: scope, timing, field requirements
- [ ] Penalties and compliance risk: what happens if you don't comply
- [ ] PLF 2026 legal detail: key provisions affecting senders

**Step 1: Write the file using this structure**

```markdown
# Sending E-Invoices in France — Advanced

## Technical Outbound Requirements

### What DGFiP External Specifications v3.1 Requires of Senders
[3–4 bullets: format conformance, CIUS-FR field population, transmission via PA,
e-reporting data submission; reference the spec version explicitly]

### CIUS-FR Mandatory Fields for Outbound Invoices
[Table: field name / description / format/type / source in typical ERP]
[Cover key fields: SIREN/SIRET of buyer and seller, invoice type code,
payment terms, VAT breakdown by rate, legal mentions required by French law]

## Outbound Validation

### What Your PA Validates Before Accepting Your Invoice
[Table: validation layer / what is checked / error if it fails]
[Layers: structural (well-formed XML), semantic (EN 16931 rules),
CIUS-FR (French extension rules), business rules (schematron)]

### Common Outbound Validation Failures
[Table: error type / likely cause / how to fix]

## E-Reporting Data Requirements

### B2C E-Reporting: Required Fields
[Table: field / description / mandatory?]
[Fields: seller SIREN, transaction date, amount ex-VAT, VAT amount by rate,
transaction type, payment method]

### Cross-Border B2B E-Reporting: Required Fields
[Table: same structure as B2C table, noting differences]

### Payment Data Reporting: Required Fields and Timing
[Table: field / description]
[Fields: invoice reference, payment date, payment amount, payment method]
[Note timing: must be reported within defined period of payment receipt]

## Legal Framework

### PLF 2026: Key Provisions for Senders
[3–4 bullets: confirms Sept 2026 / Sept 2027 timeline;
establishes the Annuaire as legally binding routing mechanism;
defines PA certification requirements;
confirms e-reporting scope including payment data]

### Penalties for Non-Compliance
[Table: violation / penalty]
[Cover: failure to issue e-invoice, failure to e-report, late e-reporting,
inaccurate e-reporting; note DGFiP enforcement approach]

## Peppol for Outbound

### Sending to Cross-Border Recipients via Peppol
[3–4 bullets: DGFiP as French Peppol authority;
French businesses can reach EU Peppol recipients through their PA;
PA must support Peppol BIS 3.0 / UBL 2.1 for cross-border;
e-reporting still required for these transactions]
```

**Step 2: Verify acceptance criteria**

Check each item. Every box must be checked.

**Step 3: Commit**

```bash
git add france-einvoicing/advanced/outbound.md
git commit -m "feat: add advanced/outbound.md — technical depth for outbound"
```

---

## Task 8: Write product/inbound.md

**File:** `france-einvoicing/product/inbound.md`

**Audience assumption:** Reader understands the mandate thoroughly. Now needs to understand how Tungsten products implement or support the inbound flow. This file is stub-heavy on first pass.

**Stub convention:** Use `> **[STUB: topic description]** — content to be added.` for every stub section.

**Required coverage (acceptance criteria):**
- [ ] Overview: where Tungsten AP products sit in the inbound Y-model flow
- [ ] PA connectivity: is Tungsten a PA, or does it connect to a PA?
- [ ] Inbound invoice receipt: how the invoice gets from PA into Tungsten
- [ ] Format handling: which formats Tungsten supports natively
- [ ] Validation and enrichment before ERP handoff
- [ ] ERP integration points
- [ ] Exception handling and operator workflow
- [ ] E-reporting obligations handled by or supported by Tungsten

**Step 1: Write the file using this structure**

```markdown
# Receiving E-Invoices — Tungsten Products

## Where Tungsten Sits in the Inbound Flow

### Tungsten AP and the Y-Model
> **[STUB: Diagram and narrative showing where Tungsten AP products sit in the
> inbound Y-model flow — PA role, connection point, handoff to ERP]**

### PA Role: Is Tungsten a Certified PA?
> **[STUB: Clarify whether Tungsten operates as a PA, partners with a PA,
> or both — and what this means for customers]**

## Receiving the Invoice

### How an Inbound E-Invoice Enters Tungsten
> **[STUB: Describe the technical integration between the PA and Tungsten —
> API call, EDI message, polling, or event-driven; include connection protocols]**

### Supported Inbound Formats
> **[STUB: List which formats Tungsten AP accepts natively (Factur-X, UBL 2.1, CII)
> and whether format conversion is handled internally or by the PA]**

## Processing the Invoice

### Validation and Enrichment
> **[STUB: Describe what Tungsten validates beyond PA-level schematron checks —
> e.g., PO matching, supplier master validation, duplicate detection;
> and what data enrichment occurs before ERP handoff]**

### Routing and ERP Handoff
> **[STUB: Describe how validated invoices are routed to the correct ERP instance/entity;
> which ERP integrations are supported; what the handoff mechanism is (API, file, message)]**

## Exception Handling

### When an Invoice Fails Validation or Matching
> **[STUB: Describe the operator workflow for exceptions —
> what the user sees, what actions they can take (approve, reject, query sender),
> and how status updates flow back through the PA to the sender]**

### Handling Rejections and Refusals
> **[STUB: Distinguish how Tungsten handles technical rejections vs.
> business refusals; workflow for notifying the sender via the PA status mechanism]**

## E-Reporting Support

### Payment Data Reporting on the Inbound Side
> **[STUB: Describe how Tungsten supports or automates the payment data
> reporting obligation — does it trigger e-reporting when a payment is posted
> in the ERP, or is this handled separately?]**
```

**Step 2: Verify acceptance criteria**

Confirm every required topic is present — either as written content or a clearly scoped stub.

**Step 3: Commit**

```bash
git add france-einvoicing/product/inbound.md
git commit -m "feat: add product/inbound.md — Tungsten AP inbound (stub-heavy)"
```

---

## Task 9: Write product/outbound.md

**File:** `france-einvoicing/product/outbound.md`

**Audience assumption:** Same as product/inbound.md. Stub-heavy first pass.

**Required coverage (acceptance criteria):**
- [ ] Overview: where Tungsten AR products sit in the outbound Y-model flow
- [ ] PA connectivity for outbound
- [ ] Format generation: which formats Tungsten AR can produce
- [ ] CIUS-FR field mapping from ERP source data
- [ ] E-reporting: how Tungsten handles B2C, cross-border, and payment data
- [ ] ERP integration points for outbound
- [ ] Status tracking and acknowledgement back to ERP

**Step 1: Write the file using this structure**

```markdown
# Sending E-Invoices — Tungsten Products

## Where Tungsten Sits in the Outbound Flow

### Tungsten AR and the Y-Model
> **[STUB: Diagram and narrative showing where Tungsten AR products sit
> in the outbound Y-model — ERP source, Tungsten role, PA connection, delivery]**

### PA Role for Outbound
> **[STUB: Clarify PA connectivity for outbound — same PA as inbound,
> separate PA options, or customer-configurable; what Tungsten handles vs.
> what the PA handles]**

## Generating the E-Invoice

### Supported Outbound Formats
> **[STUB: List which formats Tungsten AR can generate (Factur-X, UBL 2.1, CII);
> describe the format selection logic — customer preference, PA capability,
> or explicit configuration]**

### CIUS-FR Field Mapping
> **[STUB: Describe how Tungsten maps ERP source fields to CIUS-FR mandatory fields;
> which fields require customer-specific configuration;
> how SIREN/SIRET and other French-specific data is sourced]**

## Connecting to the PA

### Outbound PA Integration
> **[STUB: Describe the technical connection from Tungsten to the outbound PA —
> API, EDI (AS2/AS4), or both; authentication mechanism;
> how submission acknowledgements are handled]**

## E-Reporting

### B2C E-Reporting from Tungsten
> **[STUB: Describe how Tungsten generates and submits B2C e-reporting data —
> is this automated from AR transactions, or a separate process?
> Which data fields are sourced from where?]**

### Cross-Border B2B E-Reporting
> **[STUB: Same as B2C stub — how cross-border transactions are identified
> and how e-reporting data is generated and submitted]**

### Payment Data Reporting (Outbound)
> **[STUB: Describe how Tungsten reports payment receipt data to DGFiP —
> triggered by ERP payment posting, or manual process?
> Which PA handles the submission?]**

## Closing the Loop

### Status Tracking and ERP Acknowledgement
> **[STUB: Describe how delivery and acceptance statuses from the recipient's PA
> are received by Tungsten and surfaced back to the ERP and AR operator;
> how failed deliveries are handled]**

### Operator Visibility and Audit Trail
> **[STUB: Describe what visibility Tungsten provides to AR operators —
> dashboards, alerts, per-invoice status; audit trail for compliance purposes]**
```

**Step 2: Verify acceptance criteria**

Confirm every required topic is present — either as written content or a clearly scoped stub.

**Step 3: Commit**

```bash
git add france-einvoicing/product/outbound.md
git commit -m "feat: add product/outbound.md — Tungsten AR outbound (stub-heavy)"
```

---

## Final Step: Verify Complete File Tree

After all tasks are complete, run:

```bash
find france-einvoicing -name "*.md" | sort
```

Expected output:
```
france-einvoicing/advanced/inbound.md
france-einvoicing/advanced/outbound.md
france-einvoicing/intermediate/inbound.md
france-einvoicing/intermediate/outbound.md
france-einvoicing/intro/inbound.md
france-einvoicing/intro/outbound.md
france-einvoicing/product/inbound.md
france-einvoicing/product/outbound.md
```

Then commit the plan doc if not already committed:

```bash
git add docs/plans/
git commit -m "docs: add france e-invoicing design and implementation plan"
```
