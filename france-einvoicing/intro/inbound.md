# Receiving E-Invoices in France — Introduction

## What Is E-Invoicing?

### From Paper to Structured Data

- An e-invoice is a document exchanged in a structured, machine-readable format — not a scanned PDF or an email attachment
- Traditional invoices (paper, PDF, email) require manual data entry; structured e-invoices are read and processed directly by software
- Structured data enables straight-through processing: no re-keying, no OCR errors, automatic matching against purchase orders
- Compliance becomes auditable by design — every field is discrete, timestamped, and traceable

### Why Is France Mandating It?

- The VAT gap — the difference between VAT owed and VAT collected — costs France billions annually; real-time invoice data lets the DGFiP detect fraud and underreporting earlier
- EU Directive 2014/55/EU established a European baseline for e-invoicing in public procurement; France is extending the obligation to all domestic B2B transactions
- Structured invoices modernise finance operations: faster payment cycles, reduced disputes, lower processing costs for both buyers and suppliers
- The DGFiP gains near-real-time visibility into transaction data, enabling more accurate VAT pre-filling and audit targeting

## The French Mandate at a Glance

### Who Is Affected?

| Category | Definition | Must Receive By | Must Send By |
|---|---|---|---|
| GE (Grande Entreprise) | 5,000+ employees; >€1.5B turnover or >€2B balance sheet | Sept 1, 2026 | Sept 1, 2026 |
| ETI (Entreprise de Taille Intermédiaire) | 250–5,000 employees; <€1.5B turnover or <€2B balance sheet | Sept 1, 2026 | Sept 1, 2026 |
| SME (PME) | Up to 250 employees | Sept 1, 2026 | Sept 1, 2027 |
| Micro-enterprise | Below SME thresholds | Sept 1, 2026 | Sept 1, 2027 |

### What Does "Receiving an E-Invoice" Mean?

- A structured invoice document arrives through a certified platform (PA) — not via email or a supplier portal login
- Your organisation must be technically capable of accepting it: your ERP or accounts payable system needs to handle structured data, not just display a PDF
- The invoice contains machine-readable fields (supplier ID, line items, VAT amounts, payment terms) that your system can validate and post automatically
- To receive invoices at all, you must be registered with a PA (Plateforme Agréée) and listed in the national directory (Annuaire) — without this, suppliers have no valid address to send to

## What Do You Need to Do?

### Getting Ready to Receive — The Short Version

1. Choose a PA (Plateforme Agréée) — 101 are approved as of January 2026; this platform acts as your certified inbox for all incoming e-invoices
2. Register your business in the national directory (Annuaire) via your PA — this is how sending platforms know where to deliver your invoices
3. Ensure your ERP or AP system can receive and process the structured formats (UBL, CII, or Factur-X) that will arrive from your suppliers

### Key Term: PA (Plateforme Agréée)

- A PA (Plateforme Agréée) is a platform certified by the DGFiP to exchange e-invoices between businesses; it handles transmission, format validation, and reporting obligations on your behalf
- All e-invoices under the French mandate travel PA to PA — invoices do not arrive by email; if you have no PA, you cannot receive compliant invoices
- As of January 17, 2026, 101 PAs are listed and approved; you choose one based on your ERP integration, commercial terms, and service requirements
