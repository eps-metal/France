# Sending E-Invoices in France — Intermediate

## Choosing Your PA

### What Must a PA Do?

- Certified by DGFiP — only a PA (Plateforme Agréée) may transmit mandatory e-invoices on your behalf
- Handles both outbound sending and inbound receipt, acting as your single point of entry and exit on the network
- Validates every invoice against format rules and CIUS-FR requirements before transmission; rejected invoices never leave your PA
- Queries the Annuaire to identify the recipient's PA and routes the invoice accordingly
- Submits e-reporting data to DGFiP for B2C transactions, cross-border B2B transactions, and payment events
- 101 PAs approved as of January 17, 2026 — selection is a strategic decision, not just a vendor choice

### How to Choose the Right PA

| Criterion | Why It Matters |
|---|---|
| Format support (Factur-X / UBL 2.1 / CII) | Must match what your ERP produces and what your customers' systems expect to receive |
| ERP / AR system connectivity | API or EDI integration method determines implementation effort and ongoing reliability |
| E-reporting capability | Must handle B2C transaction reporting, cross-border B2B reporting, and payment data reporting |
| Service level (SLA, support) | Invoice delivery failures carry legal and commercial consequences — downtime is not a billing dispute, it is a compliance failure |
| Commercial terms | Cost models vary: per-invoice, subscription, or bundled with other services |

## The Outbound Invoice Journey

### Step-by-Step: From Your ERP to Delivery

1. Invoice created in your ERP/AR system in a structured format (Factur-X, UBL 2.1, or CII)
2. Transmitted to your PA via API or EDI
3. Your PA validates: format, EN 16931 semantic rules, CIUS-FR mandatory fields
4. Your PA queries the Annuaire to identify the recipient's PA

### Step-by-Step: Delivery and Status Return

5. Your PA delivers the invoice to the recipient's PA over the certified network
6. Recipient's PA delivers to the recipient's ERP or AP system
7. Status messages (delivered, accepted, rejected) return through the chain to your ERP

### What Your PA Does Before Sending

- Structural validation: confirms the invoice file is well-formed XML (or valid hybrid PDF+XML for Factur-X)
- Semantic validation: checks EN 16931 compliance — required data elements present, amounts consistent, VAT calculations correct
- CIUS-FR field checks: verifies French-specific mandatory fields are present and correctly formatted (SIREN/SIRET, invoice type codes, VAT breakdown by rate)
- Annuaire lookup: confirms the recipient's SIREN/SIRET is registered and returns the identifier of their PA before transmission begins

## E-Reporting Obligations

### Domestic B2B: Covered by the Invoice Flow

- When you send a domestic B2B e-invoice via your PA, the required e-reporting data is extracted and relayed to DGFiP automatically as part of the transmission process
- No separate e-reporting action is required from you for domestic B2B — the PA handles it

### B2C Transactions: What You Must Report

- For sales to consumers, no invoice travels via the PA network — there is no B2C counterpart registered in the Annuaire
- Instead, your PA submits transaction data to DGFiP: sale date, total amount, and VAT breakdown by rate
- This obligation covers all B2C sales regardless of amount or channel
- Your PA handles submission, but must be configured to receive the transaction data from your billing or POS system — this integration is your responsibility to set up

### Cross-Border B2B: What You Must Report

- Invoices issued to businesses established outside France do not travel via the PA network — foreign entities are not registered in the Annuaire
- Your PA still submits transaction data to DGFiP for those sales, covering your French VAT obligations
- Core data fields mirror B2C reporting: seller SIREN, invoice date, amounts, VAT breakdown by rate, and buyer country
- Your PA must be configured to identify cross-border transactions — typically via buyer VAT number format or country code in the invoice data

### Payment Data Reporting

- When a customer pays a B2B, B2C, or B2G invoice, you must report that payment event to DGFiP
- Payment data required: invoice reference, payment date, amount paid, and payment method
- Your PA handles submission once your system signals that payment has been received — the trigger must come from your ERP or cash management system
- The timeline for this obligation mirrors your mandatory sending date — payment reporting is not optional or deferred

## Formats and French Requirements

### Choosing a Format for Outbound

| Format | Type | When to Use |
|---|---|---|
| Factur-X | Hybrid PDF + XML | When customers or operators need a human-readable view alongside automated processing; common in mixed-automation environments |
| UBL 2.1 | Pure XML | Fully automated processing; widely supported in ERP integrations and pan-European trade flows |
| CII | Pure XML | Alternative to UBL; choose based on your PA's native capability and any existing CII infrastructure in your supply chain |

### CIUS-FR: The French Extension

- CIUS-FR is a French-specific conformance profile that extends the European EN 16931 standard with mandatory fields required by French tax law and the DGFiP
- Key additions include: SIREN/SIRET identifiers for both buyer and seller, invoice type codes aligned with French VAT categories, and VAT breakdown by rate with explicit base amounts
- Your ERP must be configured to populate these fields — they are absent from most non-French invoice templates and will not be added automatically by a generic e-invoicing adapter
- Your PA validates CIUS-FR compliance as part of its pre-send checks; an invoice that fails CIUS-FR validation is rejected and never transmitted — fixing it is your responsibility before resubmission
- The authoritative technical reference is DGFiP External Specifications v3.1; the advanced module covers its implementation in detail
