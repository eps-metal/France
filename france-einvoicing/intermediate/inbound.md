# Receiving E-Invoices in France — Intermediate

## The E-Invoicing Ecosystem

### The Three Pillars: PA, PPF, DGFiP

| Pillar | Role | What It Does |
|---|---|---|
| PA (Plateforme Agréée) | Certified exchange platform | Sends, receives, and validates invoices; submits e-reporting data to DGFiP on your behalf |
| PPF (Portail Public de Facturation) | Government portal | Maintains the Annuaire directory; receives structural relay copies from PAs and forwards to DGFiP |
| DGFiP | French tax authority | Receives e-reporting data for VAT monitoring and compliance enforcement |

### The Y-Model: How Invoices Actually Travel

The Y-model describes the path every B2B invoice takes:

1. Sender's ERP generates a structured invoice and hands it to the Sender's PA
2. Sender's PA queries the Annuaire to discover which PA serves the recipient
3. Sender's PA delivers the invoice directly to the Recipient's PA
4. Recipient's PA makes the invoice available to the Recipient's ERP

The "Y" shape: two legs — the sender's PA and the recipient's PA — converge at the Annuaire lookup point. The convergence is the directory resolution step; once the recipient's PA is identified, the invoice travels the resolved PA-to-PA delivery channel (the stem of the Y).

**Critical point:** The PPF is off to the side. It maintains the Annuaire and relays data copies to DGFiP, but it is not in the invoice delivery path. Invoices never pass through the PPF.

### The Annuaire: The Routing Directory

- The Annuaire is the national directory of French businesses and their designated PAs, maintained by the PPF
- When a business onboards with a PA, that PA registers the business's SIREN in the Annuaire, pointing to itself as the routing destination
- Sending PAs query the Annuaire in real time to discover the correct destination PA before delivering any invoice
- Without an Annuaire entry, a business cannot receive e-invoices — registration via a PA is a prerequisite for participation

### Registering in the Annuaire

- Registration is performed by your PA on your behalf — you do not register directly with DGFiP or PPF; your PA submits your SIREN/SIRET and connectivity details to the Annuaire
- Your PA will require your SIREN (company-level identifier) and every SIRET (establishment-level identifier) that will receive invoices
- If multiple SIRETs share the same PA, provide routing preferences at registration so the PA can route by establishment
- Once registered, your entry is visible to all other PAs on the network — any PA routing an invoice to your SIREN/SIRET will find your PA automatically and route accordingly
- For group structures with multiple SIRETs, each establishment must have its own Annuaire entry even if all establishments are served by the same PA

## The Inbound Invoice Journey

### Step-by-Step: From Sender to Your System

1. Supplier creates a structured invoice (Factur-X, UBL 2.1, or CII) in their ERP
2. Supplier's ERP transmits the invoice to the Supplier's PA
3. Supplier's PA validates the invoice format and CIUS-FR field conformance; rejects if invalid
4. Supplier's PA queries the Annuaire with your SIREN to identify your PA
5. Supplier's PA delivers the invoice securely to your PA
6. Your PA validates the invoice and makes it available via API or portal for retrieval
7. Your AP system ingests the invoice; acknowledgement status flows back through the PA network to the supplier

### Invoice Status Lifecycle — Overview

| Status | Meaning | Who Sets It |
|---|---|---|
| Deposited | Invoice submitted to the sending PA | Sender's PA |
| Received | Invoice delivered to the recipient's PA | Recipient's PA |
| Acknowledged | Recipient's system confirms successful receipt | Recipient system (via your PA) |
| Rejected | Failed technical or format validation | PA (either side) |
| Refused | Recipient disputes the invoice on business grounds | Recipient (via your PA) |
| Paid | Payment event reported to DGFiP | Recipient (via your PA) |

## Formats You Will Receive

### Factur-X, UBL 2.1, and CII — What Are They?

| Format | Type | Human-Readable? | Typical Use Case |
|---|---|---|---|
| Factur-X | Hybrid PDF + embedded XML | Yes — PDF layer is human-readable | Mixed environments; PDF layer supports manual review |
| UBL 2.1 | Pure XML | No | Fully automated machine-to-machine ERP integration |
| CII (Cross Industry Invoice) | Pure XML | No | Alternative to UBL; legacy systems and select industries |

### The French Standard: EN 16931 + CIUS-FR

- All three formats must conform to the European semantic standard EN 16931, which defines the common data model for structured invoices across EU member states
- CIUS-FR (Core Invoice Usage Specification — France) extends EN 16931 with mandatory French-specific fields such as VAT identification details and payment terms required by the DGFiP
- Format and CIUS-FR conformance validation is performed by the sending PA before the invoice is routed — malformed invoices are rejected before they ever reach your system

## Your Obligations as a Receiver

### E-Reporting: What It Covers

- E-reporting is a separate, parallel obligation to e-invoicing — it applies to transaction types that do not go through the PA-to-PA invoice network
- E-reporting covers three categories: **B2C sales** (seller reports transaction data to DGFiP), **cross-border B2B sales** (seller reports data for invoices to foreign businesses), and **payment data** (payer reports when a B2B/B2C/B2G invoice is settled)
- As a receiver of domestic B2B invoices, your relevant e-reporting obligation is the payment data category — the other two (B2C, cross-border) are the sender's responsibility

### E-Reporting: Your Payment Data Obligation

- When you **pay** a B2B invoice, you must report that payment event to DGFiP — this is distinct from the invoice receipt itself
- Your PA handles the submission automatically when triggered by the payment confirmation in your system
- The obligation applies from your respective mandatory date: GE + ETI from September 1, 2026; SMEs and micro-enterprises from September 1, 2027 *(dates current as of publication — verify against the official DGFiP schedule before compliance planning)*

### What You Do Not Need to Do

- You do not submit e-reports for invoices you receive — outbound e-reporting (transaction data) is the sender's obligation, not yours
- Your PA handles all format and CIUS-FR validation before an invoice reaches your system; you are not responsible for validating inbound invoice structure
- Archiving obligations are covered at the advanced level; your PA may offer certified archiving as a managed service

## Transition Period

### September 2026 to September 2027: Dual-Mode Operation

| Period | Who Must Send | Who Must Receive |
|---|---|---|
| Before September 1, 2026 | No one (mandate not yet active) | No one mandated |
| September 1, 2026 onwards | GE + ETI | All businesses — GE, ETI, SME, micro |
| September 1, 2027 onwards | All businesses | All businesses (already required) |

- **Critical asymmetry**: all businesses must receive e-invoices from September 1, 2026 even if their own sending deadline is September 1, 2027 — SMEs and micro-enterprises cannot defer their PA receipt configuration
- An SME receiving invoices from a large supplier must be PA-connected from September 1, 2026; if they are not, they are non-compliant — not the sender
- During the transition window (Sept 2026 – Sept 2027), any SME customer not yet registered in the Annuaire is non-compliant from September 1, 2026 — DGFiP has not defined a formal fallback channel for this scenario; monitor your PA's guidance on how to handle unroutable recipients

### PA Decertification and Migration

- DGFiP has authority under PLF 2026 to revoke PA certification — a decertified PA cannot legally transmit invoices or e-reports; all in-flight transmissions halt
- You bear legal responsibility for ensuring your PA remains certified — verify against the official DGFiP PA registry periodically; the list changes as new PAs are added or existing ones are reviewed
- Migration to a new PA requires: (1) sign with new PA, (2) new PA registers your SIREN/SIRET in the Annuaire, (3) old PA removes the entry, (4) update your API/EDI connection — coordinate steps 2 and 3 tightly to avoid a routing gap
- Ensure your PA contract specifies notice obligations in the event of decertification and provides for assisted migration — this is a key procurement point during PA selection, not an afterthought

## What's Next

### From Here to the Advanced Level

- You now understand the full inbound ecosystem: the Y-model, Annuaire routing, invoice journey, formats, and receiver obligations
- The advanced level goes deeper: the DGFiP External Specifications v3.1, the full invoice status lifecycle with technical state names, and how validation failures are handled
- It also covers connectivity options (AS2/AS4 vs. REST API), Peppol interoperability for cross-border supply chains, and your legal archiving obligations
