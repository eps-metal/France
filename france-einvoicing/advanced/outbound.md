# Sending E-Invoices in France — Advanced

## Technical Requirements for Senders

### What DGFiP External Specifications v3.1 Requires of Senders

- Invoice format must conform to one of the three accepted syntaxes — Factur-X, UBL 2.1, or UN/CEFACT CII — each implementing EN 16931 plus the CIUS-FR extension; no proprietary formats are accepted
- CIUS-FR mandatory fields must be populated in full — these go beyond the European base standard and include French-specific identifiers, legal mentions, and VAT breakdown detail
- Transmission must occur exclusively via a certified PA (Plateforme Agréée) — direct submission to DGFiP is not permitted under any circumstance
- E-reporting data derived from the invoice must also be submitted to DGFiP by your PA — this is a parallel obligation, not an alternative to e-invoicing

### CIUS-FR: Mandatory Fields for Outbound Invoices

| Field | BT Reference | Description | Source in Typical ERP |
|---|---|---|---|
| Seller SIREN | BT-29, scheme 0009 | 9-digit French company identifier for the seller | Company master data |
| Seller SIRET | BT-30, scheme 0010 | 14-digit establishment identifier for the seller | Company master data |
| Buyer SIREN | BT-46, scheme 0009 | 9-digit identifier for the buyer (domestic B2B) | Customer master data |
| Buyer SIRET | BT-49, scheme 0010 | 14-digit buyer establishment identifier | Customer master data |
| Invoice type code | BT-3 | 380 = commercial invoice; 381 = credit note; 383 = debit note; 384 = corrective; 389 = self-billed | Invoice transaction type mapping |
| VAT breakdown | BG-23 group: BT-116 (taxable), BT-117 (VAT amount), BT-119 (rate) | Tax amount, taxable amount, and rate for each VAT category | Invoice line VAT calculation engine |
| Payment terms | BT-20 (terms text), BT-9 (due date) | Due date, payment method, early payment discount if applicable | Payment terms master |
| French legal mentions | BT-22 (invoice note) | Mandatory text under French law: late payment penalty clause, regime-specific mentions | Configuration / static text template |

### Sending Credit Notes and Corrective Invoices

| Document Type | BT-3 Code | Key Requirement |
|---|---|---|
| Credit note | 381 | Must reference original invoice via BT-25 (Preceding Invoice Reference) |
| Debit note | 383 | Upward adjustment; must reference original invoice via BT-25 |
| Corrective invoice | 384 | Replaces the original; must carry a new BT-1 (Invoice Number) |
| Self-billed invoice | 389 | Buyer generates on behalf of supplier; buyer's SIREN in seller fields |

- BT-25 (Preceding Invoice Reference) is mandatory for BT-3 codes 381, 383, and 384 — omitting it fails CIUS-FR validation; populate with the BT-1 value from the original invoice
- Credit notes and debit notes follow the same outbound Y-model path as commercial invoices — same PA validation, same Annuaire routing, same status lifecycle
- VAT on credit notes: the VAT breakdown must mirror the original invoice's VAT rates; a credit note cannot introduce a new VAT rate not present on the original
- E-reporting for credit notes: they are reportable transactions with signed amounts (negative for credit notes); confirm your PA correctly signs the VAT amounts before submission

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

### Schematron Rules: Technical Reference

| Aspect | Detail |
|---|---|
| Rule naming | `BR-FR-CTC-NNN` (e.g., `BR-FR-CTC-001`) — all CIUS-FR schematron rules use this prefix |
| Current version | 1.3.0 — published by FNFE-MPE (Fédération Nationale de la Facture Électronique et des Marchés Publics Électroniques) |
| Authoritative source | FNFE-MPE GitHub repository and DGFiP External Specifications v3.1 |
| Validation tooling | Saxon (XSLT-based); ph-schematron (open-source); FNFE-MPE online validator (no install required; accepts UBL 2.1, CII, Factur-X) |
| Failure behaviour | Any single failing rule causes full invoice rejection — there are no partial passes |

- Run Schematron locally before PA submission — building it into your ERP export pipeline as a pre-flight check eliminates preventable PA rejections
- The FNFE-MPE online validator returns the same `BR-FR-CTC-NNN` error codes your PA will return — use it during development to confirm a fix is complete before resubmitting to the PA
- The ~50+ rules cover: SIREN/SIRET format (9 digits, numeric only), BT-3 code list membership, VAT calculation accuracy (amount = rate × taxable base), and BT-25 presence for credit notes

## Format Selection

### Factur-X Profiles: Compliance Requirements

| Profile | EN 16931 Compliant? | CIUS-FR Compliant? | Mandate Acceptable? |
|---|---|---|---|
| Minimum | No | No | No — insufficient data |
| Basic WL | No | No | No — no line items |
| Basic | Partial | No | No — missing mandatory fields |
| EN 16931 | Yes | Yes (with CIUS-FR extensions) | Yes — minimum acceptable profile |
| Extended | Yes | Yes | Yes — recommended for complex supply chains |

- The minimum Factur-X profile acceptable for the French e-invoicing mandate is **EN 16931** — Minimum, Basic WL, and Basic profiles do not carry the mandatory CIUS-FR data elements and will be rejected by your PA
- AFNOR XP Z12-012 (in force January 15, 2026) is the French standard defining the CIUS-FR mapping onto Factur-X — use this as the authoritative technical reference for Factur-X implementations, not generic Factur-X documentation
- The Extended profile is backward-compatible with EN 16931 — any PA that accepts EN 16931 also accepts Extended; use Extended if your supply chain requires additional data elements beyond the EN 16931 baseline

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

### E-Reporting Submission Mechanics

| Reporting Stream | Trigger | Submission Frequency |
|---|---|---|
| Domestic B2B | Automatic during invoice transmission — PA extracts and relays data to DGFiP | Per invoice, real-time |
| B2C transactions | Your system pushes transaction data to your PA | Periodic — per DGFiP implementing decree; typically aligned with VAT return period |
| Cross-border B2B | Your system pushes transaction data to your PA | Same as B2C |
| Payment data | Your ERP/treasury signals payment to your PA | Periodic — same window as B2C |

- Domestic B2B e-reporting requires no separate action from you — DGFiP data is extracted by your PA as part of the transmission process; this is the only reporting stream that is fully automatic
- B2C and cross-border B2B e-reporting require your billing or POS system to push transaction data to your PA — this integration must be configured and tested separately from the invoice transmission integration
- Submission uses the same PA API or EDI channel as invoice submission; your PA documentation defines the specific endpoint and payload schema for e-reporting data
- Verify the current reporting window schedule with your PA and legal counsel before go-live — DGFiP implementing decrees govern the exact timing and the schedule may be updated

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
