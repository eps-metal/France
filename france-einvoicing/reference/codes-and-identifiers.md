# France E-Invoicing: Codes & Identifiers Reference

---

## CDAR Message Codes

| Code | Status | Meaning | Direction |
|------|--------|---------|-----------|
| 200 | Delivered | Invoice transmitted to recipient PA (PAr) | PAe → PPF |
| 210 | Refused | Business refusal by buyer | PAr → PPF |
| 212 | Paid | Payment confirmed, reported to DGFiP | PAe → PPF |
| 213 | Rejected | Technical validation failure at receiving PA | PAr → PPF |

---

## Invoice Lifecycle Statuses

| French | English | Trigger | Who Sets It |
|--------|---------|---------|-------------|
| Déposée | Deposited | Invoice submitted to sending PA | Sender's PA |
| Reçue | Received | Delivered to recipient PA | Recipient's PA |
| Approuvée | Approved | Recipient confirms receipt | Recipient (via PA) |
| Rejetée | Rejected | Failed technical/format validation | PA (either) |
| Refusée | Refused | Recipient disputes on business grounds | Recipient (via PA) |
| En Anomalie | Under Query | Issue raised needing clarification | Recipient (via PA) |
| Annulée | Cancelled | Withdrawn by mutual agreement | Sender (via PA) |
| Encaissée | Payment Reported | Payment event reported to DGFiP | Recipient (via PA) |

---

## Invoice Type Codes (BT-3)

| Code | Type | Notes |
|------|------|-------|
| 380 | Commercial invoice | Standard B2B invoice |
| 381 | Credit note | Must reference original via BT-25 |
| 383 | Debit note | Upward adjustment; must reference original via BT-25 |
| 384 | Corrective invoice | Replaces original; carries new BT-1 |
| 389 | Self-billed invoice | Only valid code for self-billing (UC 19b) |

---

## E-Reporting Flow Codes

| Flow | Scope | Aggregation | Notes |
|------|-------|-------------|-------|
| 1 | Domestic B2B invoices | Per-invoice, automatic | Fires as side-effect of every domestic invoice transmission — zero supplier effort |
| 8 | Domestic B2B invoice data | Per-invoice, automatic | PA → PPF relay; extracted during transmission |
| 9 | Domestic B2B status/lifecycle data | Per-event, automatic | PA → PPF relay; status changes forwarded as they occur |
| 10 | Non-domestic reporting (umbrella) | Aggregated on schedule | Covers all sub-flows below |
| 10.1 | International B2B + B2C invoicing data | Aggregated | Data not already covered by flows 8/9 |
| 10.2 | Payment data | Aggregated | For e-invoices, flows 8/9, and 10.1; triggered by payment events in ERP |
| 10.3 | B2C transaction data | Aggregated | Transactions where no invoice was issued |
| 10.4 | B2C transaction payment data | Aggregated | Payment data for invoiceless B2C transactions |

---

## Identifier Schemes

| Scheme | Format | Used For |
|--------|--------|----------|
| 0009 | 9-digit SIREN | Company-level identification |
| 0010 | 14-digit SIRET | Establishment-level identification |

SIREN = first 9 digits of SIRET. SIRET = SIREN + 5-digit NIC (establishment code).

---

## EN 16931 / CIUS-FR Field Reference

| BT Code | Field Name | Description | CIUS-FR Mandatory? |
|---------|------------|-------------|-------------------|
| BT-1 | Invoice Number | Fingerprinted by SIREN + number; must be unique | Yes |
| BT-3 | Document Type Code | 380 / 381 / 383 / 384 / 389 | Yes |
| BT-9 | Due Date | Payment due date | Yes |
| BT-20 | Payment Terms | Free-text payment terms | Yes |
| BT-22 | Invoice Note | Mandatory French legal mentions (e.g., late-payment penalties clause) | Yes |
| BT-25 | Preceding Invoice Reference | Reference to original invoice | Conditional (required for 381/383/384) |
| BT-29 | Seller SIREN | Seller identification, scheme 0009 | Yes |
| BT-30 | Seller SIRET | Seller establishment, scheme 0010 | Yes |
| BT-46 | Buyer SIREN | Buyer identification, scheme 0009 | Yes (domestic B2B) |
| BT-49 | Buyer SIRET | Buyer establishment, scheme 0010 | Yes (domestic B2B) |
| BT-116 | Taxable Amount | Net amount per VAT category (in BG-23) | Yes |
| BT-117 | VAT Amount | Tax amount per VAT category (in BG-23) | Yes |
| BT-119 | VAT Rate | Applicable rate per VAT category (in BG-23) | Yes |
| BG-23 | VAT Breakdown Group | One group per distinct VAT category/rate combination | Yes |
| BG-24 | Additional Supporting Documents | Attachments, referenced documents | Optional |

---

## CIUS-FR Schematron Validation Rules

| Property | Value |
|----------|-------|
| Pattern | `BR-FR-CTC-NNN` (e.g., BR-FR-CTC-001) |
| Current version | 1.3.0 (FNFE-MPE) |
| Rule count | ~50+ rules |
| Scope | SIREN format, BT-3 code list, VAT accuracy, BT-25 presence, legal mentions |
| Failure behavior | Any single failing rule = full invoice rejection |

---

## Format Standards

| Code | Description |
|------|-------------|
| EN 16931 | European semantic standard for structured invoices |
| CIUS-FR | Core Invoice Usage Specification — France; extends EN 16931 with French-specific rules |
| UBL 2.1 | Pure XML format (OASIS) |
| CII | Pure XML format (UN/CEFACT Cross Industry Invoice) |
| Factur-X | Hybrid PDF/A-3 + embedded XML; human-readable and machine-readable |
| Peppol BIS 3 | PA-to-PA exchange profile; UBL 2.1 based |

---

## Penalties (PLF 2026)

Subject to implementing decree.

| Violation | Penalty | Annual Cap |
|-----------|---------|------------|
| Failure to issue e-invoice | €15 / invoice | €15,000 |
| Late e-reporting transmission | €250 / transmission | €15,000 |
| Inaccurate e-reporting | €15 / invoice | €15,000 |
