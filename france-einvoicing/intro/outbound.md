# Sending E-Invoices in France — Introduction

## Sending Under the Mandate

### What Does "Sending an E-Invoice" Mean?

- A compliant e-invoice is a structured, machine-readable document — not a PDF attached to an email; it must conform to approved formats (UBL, CII, or Factur-X)
- The invoice is submitted to your PA (Plateforme Agréée), which validates it and routes it to your customer's PA — the recipient's platform delivers it to their system
- The DGFiP receives a copy of the transaction data for VAT monitoring purposes — it does not receive the full invoice document itself
- This is a fundamental departure from current practice: emailing a PDF to a customer is not compliant and will not satisfy the mandate

### Who Must Send, and When?

| Category | Definition | Must Send By |
|---|---|---|
| GE (Grande Entreprise) | 5,000+ employees; >€1.5B turnover or >€2B balance sheet | Sept 1, 2026 |
| ETI (Entreprise de Taille Intermédiaire) | 250–5,000 employees; <€1.5B turnover or <€2B balance sheet | Sept 1, 2026 |
| SME (PME) | Up to 250 employees | Sept 1, 2027 |
| Micro-enterprise | Below SME thresholds | Sept 1, 2027 |

> **Note:** All categories — including SME and micro-enterprises — must be able to **receive** e-invoices by **Sept 1, 2026**, regardless of their sending deadline.

## E-Reporting: The Other Obligation

### E-Invoicing vs. E-Reporting — What's the Difference?

| Obligation | Applies To | What Is Sent | Who Receives It |
|---|---|---|---|
| E-invoicing | Domestic B2B transactions | Full structured invoice document | Recipient's PA → buyer's system |
| E-reporting | B2C sales, cross-border B2B, and payment data | Transaction data only (not the invoice) | DGFiP |

### Why Does E-Reporting Matter for Senders?

- Even if your customer does not receive invoices via a PA — a consumer, or a foreign business outside the mandate — you still have a reporting obligation to the DGFiP for those transactions
- Payment data reporting applies even for domestic B2B invoices: once payment is received or made, that event must be reported separately
- Your PA handles e-reporting submission on your behalf — it is not a separate system you connect to directly, but it is a distinct data flow your PA must be configured to support

## What Do You Need to Do?

### Getting Ready to Send — The Short Version

1. Determine which category your business falls into (GE, ETI, or SME) and confirm your mandatory sending deadline
2. Choose and onboard a PA — it will handle sending, format validation, routing to your customers' PAs, and e-reporting to the DGFiP
3. Connect your ERP or accounts receivable system to your PA so invoices flow out automatically rather than manually
4. Map your invoice data to the required structured formats and French-specific field requirements (CIUS-FR), including mandatory fields not present in most current invoice templates

### The Timeline Is Locked

- In April 2025, the French National Assembly rejected a proposed postponement — there is no further delay expected
- The PLF 2026 (Finance Bill), adopted February 2, 2026, provides the legal anchor confirming the mandate and timeline
- September 1, 2026 is the hard date for GE and ETI — preparation, PA selection, and ERP integration must begin now to avoid a compliance gap
