# France Mandate – Technical Enablement Update (November 2025)

**Source:** CaaS deck, `pipeline/a1-source/product/caas/2026.1 - France Enablement update -2025-11-25/`
**Date:** 25 November 2025
**Presenters:** James Sealey (Director Product), Emma Higgins (Product Owner)

---

## Agenda Topics
- Background: scope and timeline, Invoice Agility
- France AP/AR Integration in Practice Network: transaction data flows, key points, Peppol
- TeN and TeC environments for CTC
- Tungsten e-Invoice Network
- AR and AP Use Cases
- Customer onboarding
- PA Management
- Lifecycle management
- Reporting
- Next Steps / Q&A

---

## French Reform from September 2026

Three pillars:

| Pillar | Detail |
|--------|--------|
| **E-invoicing Receiving** | All businesses must be able to receive e-invoices by Sept 1, 2026 from all providers and accept in any format |
| **E-Reporting** | Payment data, VAT data, and invoice data must be reported to DGFiP via PPF on a scheduled basis |
| **Payment Status** | Status of payments (date, method, amount) must be reported for all invoice categories |

---

## Key Points

- **Tungsten Automation is providing a white-labelled Sovos PDP solution (TA-PDP)** — now known as PA (Plateforme Agréée)
  - Registered under number **0072** with French tax authorities
- **SecNumCloud Certification:** PA environment is hosted in SecNumCloud-certified environment
  - Only EU-based resources permitted to access PA environment
  - Affects Cloud, Technical Support, potentially PS and CSM teams
- **No direct customer access to Sovos PA environment** — all functions presented via Tungsten's own UI

---

## Peppol

- PAs can exchange data over Peppol; some details still to be finalised
- Peppol is **not mandated** — preferred by some PAs, others will use other structured formats
- **Confirmed:**
  - Data still needs to flow through a SecNumCloud-certified PA (Sovos still required)
  - PPF must still be consulted for routing
  - Invoice data exchanged between PAs in **Peppol BIS 3 UBL format**
- **Still unclear:**
  - Format for Lifecycle data exchange between PAs (main assumption: managed by Sovos)
  - Requirements for digital signature on invoices (Peppol uses AS4 for authenticity/integrity)

---

## TeN High-Level Development Timeline

| Milestone | Timing |
|-----------|--------|
| PPF Directory testing | 2025 H1 (Apr/May) |
| AFNOR Spec V1 | 2025 Q3 |
| Interoperability testing can start | 2025 Q4 |
| Grand Pilot testing can start | 2026 Q1 |
| Large and Medium must submit; All must receive | 2026 Q2 |
| Small and micro must submit | 2026 H2 |

**TeN Release features by release:**

| Release | Features |
|---------|----------|
| 2025.3 | Design; MVP Connectivity; Main AP and AR flow (1&2); Customer Config (Create); Main Lifecycle Flow (6) |
| 2026.1 | AP and AR flow (other use cases); Customer Config (Manage); Portal AR flows; Other Lifecycle Flows; Main E-Reporting flow (10) |
| 2026.3 | Other B2B E-Reporting flows; B2C E-Reporting flows |

> Note: Details for what the Pilot contains are not yet published. Expectation is that live invoices can be exchanged between PAs but all reporting to the PPF will be disregarded.

---

## Environments

Same architecture as described in Mar 2026 deck:
- **TeN Production** ↔ Sovos Production ↔ Other PAs Production + French Gov Production
- **TeN OBD** (Customer UAT) ↔ Sovos Sandbox ↔ Other PAs Sandbox + French Gov Test
- **TeN Test0** ↔ TeC Dev/Test (internal)
- TeC Partner (Sign Up live) → TeC Production → TeN Production
- External: PD Production, PD UAT, APS Production

---

## AR/AP Use Cases — Extended Schema

### Mapping Status (as of Nov 2025)
- Gap analysis complete: full FR schema vs. TeN internal mapping (inbound AR and outbound AP)
- First 10 use cases taken through QA process
- Exception: delivery address at line level required dev work; QA to follow post-Nov release

### First 10 Use Cases (in QA):
| Case | Description |
|------|-------------|
| 1 | Multi-order / Multi-delivery *(post Nov. release)* |
| 2 | Invoice already paid by purchaser or third party at time of invoicing |
| 3 | Invoice to be paid by a third party designated at time of invoicing |
| 4 | Invoice partially covered by a third party (subsidy, insurance, etc.) |
| 8 | Invoice to be paid to a third party at time of invoicing (factoring, treasury centralization) |
| 9 | Invoice to a third party who manages order/reception/invoicing (distributor/depositary) *(post Nov. release)* |
| 11 | Invoice with "Addressed to" (INVOICEE) different from buyer |
| 17a | Invoice payable to third-party payment intermediary (e.g., marketplace) |
| 17b | Invoice payable to third party, payment intermediary + invoicing mandate |
| 19a | Invoice issued with billing mandate |
| 20 & 21 | Down-payment invoice and final invoice after down-payment |

### Additional 26 Use Cases
- TeN will support all 26; QA target: early 2026
- Many are niche/industry-specific; expected to be low-volume
- Web Form development to follow after use case review

### Mapping Steps
| Step | Scope | Status |
|------|-------|--------|
| 1 | EN16931 European standard | Basic testing completed June 2025 |
| 2 | First 10 extended use cases (Sovos-supported) | Early 2026 |
| 3 | Additional 26 extended use cases | Early 2026 |

> **Important:** TeN will continue to create a Tungsten Network PDF image, but it will be marked as a **non-tax image** and digital signing will be discontinued for mandate-subject invoices. **Customers must be informed and asked to consider impact on their processing.**

---

## Customer Onboarding

### KYC (Know Your Customer)
- Integral part of the mandate regulatory framework
- Not covered in depth in this deck — likely a separate feature/application
- Will include a team leader toolkit and downstream KYC accreditation process

### Role of the PA (six functions)
1. **Directory management** — onboarding, company details, routing information
2. **E-invoicing transmission** — intermediary for structured e-invoice format with mandatory fields
3. **Data extraction & transmission to DGFiP** — via PPF
4. **Invoice lifecycle** — status updates (submitted, refused, paid) to government platform
5. **E-reporting** — manage and transmit e-reporting data (B2B, B2C aggregated, etc.)

### Directory / Routing
- Routing data held centrally in PPF
- Routing uses SIREN number; example: `1234567891234S-FIN001`
- Data includes: SIREN, account number, company address, invoice ID, invoice type

---

## PA Management (TeN UI)

### Features built/in progress:
- User access / permissions management (PA Admin)
- Landing page: company information and PA management table
- Hierarchy restrictions (prevents duplicate/invalid company hierarchy)
- PA Opt-in for External PA
- Tungsten Automation PA assignment
- e-Address format selection: EDEN, SIREN, SIRET, UPEN, UPEM, UPF
- In-page validations
- Queuing / operation ordering with timestamps
- Status icons: green (success), red (error), white (pending), yellow triangle (warning)
- Bulk VAT / address removal
- Single/multi establishment address removal
- User help & support contextual tooltips

EPIC: https://docusphere.atlassian.net/browse/TNP-43499

---

## Company Hierarchy Rules (France)

### Structure Requirements
1. Each unique company needs a **single head office entity** with:
   - Official company name
   - Valid Intra-Community VAT number (format: `FR` + digits, which encodes SIREN)
   - Head office SIRET number as company registration number
   - Critical: used during KYC and customer onboarding

2. **Establishments** (same VAT, different SIRET) must be **children of the head office entity**
   - Max depth: 1 level (children cannot have nested hierarchy)
   - Each establishment must share the same VAT as its head office
   - Each establishment must have its own SIRET as company registration number

3. Entity address details must be populated accurately and consistently

### Example Structure:
```
[HEAD OFFICE] ACME France HPC Industries
  VAT: FR77501569444  SIRET: 50156944000
  ├── [CHILD] ACME France HPC Industries (LE MEUX)
       VAT: FR77501569444  SIRET: 50156944000036

[HEAD OFFICE] ACME France
  VAT: FR16552119216  SIRET: 55211921602139
  ├── [CHILD] ACME France (ASNIERES-SUR-SEINE)
       VAT: FR16552119555  SIRET: 55211955501271
  └── [CHILD] ACME France (LE PORT)
       VAT: FR16552119555  SIRET: 55211955502071
```

### Hierarchy Implementation Plan
- **Initial setup (PS project):** PS team works with buyers to create all FR companies/establishments in correct hierarchical format in test environment; at Sep 2026 cut-over these replace pre-existing accounts
- **This hierarchy is a prerequisite for all IVA products** — TeN, TeC, PD, etc.
- **Field changes planned:**
  - Label: "VAT registration number" → "Intra-Community VAT number / SIREN number"
  - Label: "Company registration number" → "SIRET number"
  - Field validation: enforce expected alpha/numeric format
  - Field protection: fields locked (read-only) once customer has active e-Invoicing subscription via TA PA

---

## Lifecycle Management

### Existing Invoice Status Service (ISS)
TeN already provides Invoice Status Service for buyers:

**System-derived statuses:** Sent, Accepted, Failed, Delivered

**Buyer-derived statuses:** Received, Processing, Paid, Rejected, Accepted, On Hold

### France Lifecycle — Backend Development (target: end Jan 2026)
7 backend development items:
1. ISS application changes to support French mandate requirements
2. Amend Studio to support outbound mapping configs for ISS
3. Amend Studio to allow inbound mapping config deployment to supplier
4. Create outbound master map: ISS internal structure → CDAR
5. Create inbound master map: CDAR → ISS internal structure
6. Add French-specific invoice status equivalents to ISS internal statuses
7. Turn on ISS services for a supplier via OBi

> Various stories in flight across Oct/Nov/Dec releases

Reference: https://docusphere.atlassian.net/wiki/spaces/TNCA/pages/5282365494/FR+lifecycle+in+CDAR+vs+ISS+internal+structure

---

## Reporting (E-Reporting)

### Types of Reporting

**Basic B2B invoice reporting (for Suppliers):**
- Invoices exchanged between PAs are also reported to PPF
- Covered by Sovos: extracts relevant data during invoice processing and sends to receiving PA

**Aggregated scheduled reporting (based on business tax regime):**
- International invoices sent by suppliers
- International invoices received by buyers
- Payment data (separate from payment lifecycle updates; depends on business VAT registration)
- B2C invoices
- Uses Sovos Scheduler functionality

> **E-Reporting status as of Nov 2025:** Only first drafts of docs from Sovos; requirements not yet defined. Begin scoping end of January 2026; dependent on Sovos documentation detail.

---

## PA Accreditation / Interoperability Testing

- As of Nov 2025, Tungsten Automation has **"provisional / intermediary"** accreditation as PA
- **Registered subject to conditions** = passed stage 1 (all documents provided)
- **Stage 2 (definitive registration):** verification of technical compliance (invoice exchange between PAs, data extraction, etc.) — from end of 2025
- Target: complete PA interoperability testing by **mid-December 2025**
- Led by Cloud Services + Sovos partnership; requires multi-team support

List of approved registered platforms: https://www.impots.gouv.fr/liste-des-plateformes-agreees-immatriculees

---

## Next Steps Timeline (from Nov 2025)

| Date | Milestone |
|------|-----------|
| Nov 26, 2025 | Finalize Annuaire connectivity testing |
| Mid-Dec 2025 | PA interoperability testing (Cloud Services led) |
| End of Dec 2025 | Schema for Lifecycle data (send/receive); First draft PS best practices for France |
| Jan 2026 | PS-supported UAT for mandatory & optional lifecycle; PS-assisted onboarding via CCC API (OBD → Sovos sandbox) |
| End of Jan 2026 | Customer UAT on OBD: e-address construction; EN16931 send/receive; first 10 use cases; lifecycle send/receive |
| Feb 2026 | Target for UI: onboarding & lifecycle (deployment to OBD TBC) |
| End of Feb 2026 | Sending/receiving additional 26 use cases |
| Mar 2026 | Ongoing UAT/testing ahead of Sep 2026 go-live |
| Apr/May 2026 | E-reporting flows |

---

## TeC Development Timeline (TBC)

Same milestone structure as TeN; TeC release schedule mirrors TeN:
- 2025.3, 2026.1, 2026.3 releases
- Features tracked in parallel to TeN (AP/AR flows, lifecycle, e-reporting)

---

## Appendix — Reference Links

| Topic | Link |
|-------|------|
| France e-invoicing and e-reporting | https://docusphere.atlassian.net/wiki/spaces/TNCA/pages/4611997724/ |
| Managing customer hierarchies | https://docusphere.atlassian.net/wiki/spaces/TNCA/pages/5211684873/ |
| PA customer onboarding UI (EPIC) | https://docusphere.atlassian.net/browse/TNP-43499 |
| Support for extended use cases | https://docusphere.atlassian.net/browse/TNP-43812 |
| FR lifecycle in CDAR vs ISS | https://docusphere.atlassian.net/wiki/spaces/TNCA/pages/5282365494/ |
| Lifecycle Jira tickets | TNP-43656, TNP-44160, TNP-43405, TNP-44090 |
