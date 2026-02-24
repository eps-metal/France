# Receiving E-Invoices in France — Intermediate

## The E-Invoicing Ecosystem

### The Three Pillars: PA, PPF, DGFiP

| Pillar | Role | What It Does |
|---|---|---|
| PA (Plateforme Agréée) | Certified exchange platform | Sends, receives, and validates invoices; submits e-reporting data to DGFiP on your behalf |
| PPF (Portail Public de Facturation) | Government portal | Maintains the Annuaire national directory; relays data copies to DGFiP |
| DGFiP | French tax authority | Receives e-reporting data for VAT monitoring and compliance enforcement |

### The Y-Model: How Invoices Actually Travel

The Y-model describes the path every B2B invoice takes:

1. Sender's ERP generates a structured invoice and hands it to the Sender's PA
2. Sender's PA queries the Annuaire to discover which PA serves the recipient
3. Sender's PA delivers the invoice directly to the Recipient's PA
4. Recipient's PA makes the invoice available to the Recipient's ERP

The "Y" shape: two legs — the sender's PA and the recipient's PA — converge at the Annuaire lookup point.

**Critical point:** The PPF is off to the side. It maintains the Annuaire and relays data copies to DGFiP, but it is not in the invoice delivery path. Invoices never pass through the PPF.

### The Annuaire: The Routing Directory

- The Annuaire is the national directory of French businesses and their designated PAs, maintained by the PPF
- When a business onboards with a PA, that PA registers the business's SIREN in the Annuaire, pointing to itself as the routing destination
- Sending PAs query the Annuaire in real time to discover the correct destination PA before delivering any invoice
- Without an Annuaire entry, a business cannot receive e-invoices — registration via a PA is a prerequisite for participation

## The Inbound Invoice Journey

### Step-by-Step: From Sender to Your System

1. Supplier creates a structured invoice (Factur-X, UBL 2.1, or CII) in their ERP
2. Supplier's ERP transmits the invoice to the Supplier's PA
3. Supplier's PA validates the invoice format and CIUS-FR field conformance; rejects if invalid
4. Supplier's PA queries the Annuaire with your SIREN to identify your PA
5. Supplier's PA delivers the invoice securely to your PA
6. Your PA validates the invoice and makes it available in your AP system or ERP
7. Your system acknowledges receipt; the acknowledgement status flows back through the PA network to the supplier

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
| Factur-X | Hybrid PDF + embedded XML | Yes — PDF layer is human-readable | Mixed environments where some parties still need visual review alongside automated processing |
| UBL 2.1 | Pure XML | No | Fully automated machine-to-machine processing; common in procurement and ERP integrations |
| CII (Cross Industry Invoice) | Pure XML | No | Alternative to UBL; prevalent in certain industries and some legacy system integrations |

### The French Standard: EN 16931 + CIUS-FR

- All three formats must conform to the European semantic standard EN 16931, which defines the common data model for structured invoices across EU member states
- CIUS-FR (Core Invoice Usage Specification — France) extends EN 16931 with mandatory French-specific fields such as VAT identification details and payment terms required by the DGFiP
- Format and CIUS-FR conformance validation is performed by the sending PA before the invoice is routed — malformed invoices are rejected before they ever reach your system

## Your Obligations as a Receiver

### E-Reporting: Payment Data Reporting

- E-reporting is a separate obligation from e-invoicing: when you pay a B2B invoice, you must report that payment event to DGFiP
- This payment data reporting applies even though the invoice itself was already transmitted via PA-to-PA exchange — the payment event is distinct and must be declared independently
- Your PA handles the submission to DGFiP automatically when triggered by the payment confirmation in your system
- The obligation applies to all company sizes from their respective mandatory sending deadlines (GE + ETI from 1 September 2026; SMEs and micro-enterprises from 1 September 2027)

### What You Do Not Need to Do

- You do not submit e-reports for invoices you receive — outbound e-reporting (transaction data) is the sender's obligation, not yours
- Your PA handles all format and CIUS-FR validation before an invoice reaches your system; you are not responsible for validating inbound invoice structure
- Archiving obligations are covered at the advanced level; your PA may offer certified archiving as a managed service
