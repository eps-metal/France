# Sending E-Invoices in France — Advanced

## Technical Requirements for Senders

### What DGFiP External Specifications v3.1 Requires of Senders

- Invoice format must conform to one of the three accepted syntaxes — Factur-X, UBL 2.1, or UN/CEFACT CII — each implementing EN 16931 plus the CIUS-FR extension; no proprietary formats are accepted
- CIUS-FR mandatory fields must be populated in full — these go beyond the European base standard and include French-specific identifiers, legal mentions, and VAT breakdown detail
- Transmission must occur exclusively via a certified PA (Plateforme Agréée) — direct submission to DGFiP is not permitted under any circumstance
- E-reporting data derived from the invoice must also be submitted to DGFiP by your PA — this is a parallel obligation, not an alternative to e-invoicing

### CIUS-FR: Mandatory Fields for Outbound Invoices

| Field | Description | Source in Typical ERP |
|---|---|---|
| Seller SIREN/SIRET | 9-digit (SIREN) or 14-digit (SIRET) French business identifier for the seller | Company master data |
| Buyer SIREN/SIRET | Same identifier for the buyer (required for domestic B2B) | Customer master data |
| Invoice type code (BT-3) | Coded type identifying the transaction: commercial invoice, credit note, debit note, etc. | Invoice transaction type mapping |
| VAT breakdown by rate | Tax amount, taxable amount, and applicable rate for each VAT category present on the invoice | Invoice line VAT calculation engine |
| Payment terms | Due date, payment method, and early payment discount if applicable | Payment terms master |
| French legal mentions | Mandatory text elements required by French law, including the late payment penalty clause and any applicable auto-entrepreneur or regime-specific mentions | Configuration / static text template |

## Outbound Validation

### Validation Layers: What Your PA Checks

| Layer | What Is Checked | Error Type |
|---|---|---|
| Structural | XML is well-formed; PDF+XML attachment is valid for Factur-X; file encoding is correct | Format error — invoice rejected before parsing |
| Semantic (EN 16931) | Required data elements are present; monetary amounts are consistent; VAT logic is internally coherent | Semantic error — invoice rejected |
| CIUS-FR | French-specific mandatory fields are present and correctly formatted per CIUS-FR rules | CIUS error — invoice rejected |
| Business rules (schematron) | ~50+ cross-field checks: calculation verification, code list validation, date logic, conditional field dependencies | Business rule violation — invoice rejected |

### Common Outbound Validation Failures

| Error Type | Likely Cause | Fix |
|---|---|---|
| Missing SIREN | Customer master lacks SIREN/SIRET for the buyer | Populate SIREN/SIRET in customer master before first submission; validate completeness as part of onboarding |
| VAT calculation mismatch | Rounding difference between line-level totals and header-level VAT summary | Align rounding logic to EN 16931 specification; apply rounding at header level, not line level |
| Incorrect invoice type code | Wrong BT-3 code used — e.g., credit note submitted with commercial invoice code | Map all ERP transaction types to the correct BT-3 code list; test each transaction type in pre-production |
| Missing payment terms | Payment terms not exported from ERP into the invoice XML | Confirm payment terms are included in the XML output template, not just the PDF layer |
| Malformed XML | ERP export encoding error, missing namespace declaration, or illegal characters | Validate XML against schema before submission; PA rejects malformed documents immediately without parsing |

## E-Reporting Data Requirements

### B2C E-Reporting: Required Fields

| Field | Description | Mandatory? |
|---|---|---|
| Seller SIREN | 9-digit seller identifier | Yes |
| Transaction date | Date of supply or invoice date | Yes |
| Amount excl. VAT | Total taxable amount before VAT | Yes |
| VAT amount by rate | VAT amount and corresponding rate for each VAT category on the transaction | Yes |
| Transaction type code | Identifies the nature of the supply (goods, services, etc.) | Yes |
| Payment method | Cash, card, bank transfer, etc. | Yes |

### Cross-Border B2B E-Reporting: Required Fields

| Field | Description | Difference from B2C |
|---|---|---|
| Seller SIREN | 9-digit seller identifier | Same as B2C |
| Transaction date | Date of supply or invoice date | Same as B2C |
| Amount excl. VAT | Total taxable amount before VAT | Same as B2C |
| VAT amount by rate | VAT amount and rate for each VAT category | Same as B2C |
| Transaction type code | Nature of the supply | Same as B2C |
| Payment method | Cash, card, transfer, etc. | Same as B2C |
| Buyer country code | ISO country code of the foreign buyer | Required — replaces SIREN |
| Buyer SIREN | Not required | Foreign entity — omit; use buyer country code instead |

### Payment Data Reporting: Fields and Timing

| Field | Description |
|---|---|
| Invoice reference | The invoice identifier this payment settles |
| Payment date | Date payment was received or made |
| Payment amount | Amount paid — may be partial for installment arrangements |
| Payment method | How payment was made (transfer, card, cash, etc.) |

- Payment data must be reported within the period defined in DGFiP implementation orders; timing windows are subject to implementing decree — verify the current schedule with your PA and legal counsel before go-live

## Legal Framework

### PLF 2026: Key Provisions for Senders

- Confirms the mandatory sending timeline: September 1, 2026 for GE and ETI; September 1, 2027 for SMEs and microenterprises — note the asymmetry: all businesses must be able to **receive** from September 1, 2026, even if their sending deadline is later
- Establishes the Annuaire (the DGFiP routing directory) as a legally binding mechanism — all B2B invoice routing must use the Annuaire via a certified PA; bypassing it is not legally compliant
- Defines PA certification requirements and grants DGFiP the authority to revoke PA certification — senders bear responsibility for ensuring their PA remains certified
- Confirms that e-reporting scope covers B2C transactions, cross-border B2B transactions, and payment data — all three streams are legally required, not optional

### Penalties for Non-Compliance

| Violation | Proposed Penalty (PLF 2026) | Annual Cap |
|---|---|---|
| Failure to issue e-invoice | €15 per invoice | €15,000/year |
| Failure to e-report or late e-reporting | €250 per transmission | €15,000/year |
| Inaccurate e-reporting | €15 per invoice | €15,000/year |

- Penalties above are per PLF 2026 provisions (adopted February 2, 2026) — amounts are subject to implementing decree and may be adjusted; verify the current penalty schedule before finalizing your compliance planning

## Peppol for Outbound

### Sending to Cross-Border Recipients via Peppol

- DGFiP is the French Peppol authority — certified French PAs can connect to the European Peppol network, enabling delivery to any Peppol-registered recipient in the EU and beyond
- If your PA supports Peppol, you can reach EU Peppol-registered recipients using UBL 2.1 formatted to Peppol BIS 3.0; your PA handles the network routing transparently
- E-reporting to DGFiP is still required for all cross-border transactions, even those delivered via Peppol — Peppol delivery does not satisfy the e-reporting obligation
- Verify Peppol capability explicitly with your PA before assuming cross-border delivery is supported — not all certified PAs have activated Peppol connectivity
