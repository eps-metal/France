---
title: Sending E-Invoices in Fra
created: 2026-05-09
modified: 2026-05-09
topics:
  - france
  - compliance
  - e-invoicing
doc_type: guide
status: raw
summary: >
  title: Sending E-Invoices in France — Advanced
keywords:
  - ai
  - ar
  - compliance
  - france
  - fraud
  - invoice
  - outbound
---


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

### E-Reporting Flow Type Codes

| Flow | Scope | Aggregation |
|------|-------|-------------|
| 1 | Domestic B2B invoice data | Per-invoice, automatic — PA extracts during transmission |
| 8 | Domestic B2B invoice data relay (PA → PPF) | Automatic, extracted during transmission |
| 9 | Domestic B2B lifecycle/status data relay (PA → PPF) | Automatic, per status change |
| 10 | Non-domestic and supplementary reporting | Aggregated on schedule by PA |
| 10.1 | International B2B + B2C invoicing data not in flows 8/9 | Aggregated on schedule |
| 10.2 | Payment data for all invoice flows (8/9/10.1) | Aggregated, triggered by ERP payment events |
| 10.3 | B2C transaction data (no invoice issued) | Aggregated on schedule |
| 10.4 | B2C transaction payment data (invoiceless) | Aggregated on schedule |

- Flow 1 fires as an async side-effect of every domestic B2B invoice transmission — zero supplier effort required
- Flows 8 and 9 are PA-to-PPF relay flows — your PA handles these transparently; they are not sender-initiated
- Flow 10 is the umbrella for all non-domestic reporting; sub-flows 10.1–10.4 are scheduled by your PA
- "No action on your side" is true for the reporting mechanism itself; ERP data feeds for cross-border, B2C, and payment events must still be configured upfront

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

## CDAR Message Codes

### CDAR: Invoice Lifecycle Message Codes

| Code | Status | Meaning | Direction |
|------|--------|---------|-----------|
| 200 | Delivered | Invoice successfully transmitted to recipient PAr | PAe → PPF |
| 210 | Refused | Business refusal by buyer (e.g., duplicate, wrong PO) | PAr → PPF |
| 212 | Paid | Payment confirmed by supplier; full settlement reported to DGFiP | PAe → PPF |
| 213 | Rejected | Technical validation failure at receiving PA | PAr → PPF |

- CDAR codes are the machine-readable status messages exchanged between PAs and relayed to DGFiP via PPF
- Code 200 (Delivered) fires automatically on successful PA-to-PA transmission — no action required from the sender
- Code 212 (Paid) is separate from the CDAR lifecycle — it is triggered by a payment event in the sender's ERP/treasury system, not by invoice acceptance
- Codes 210 and 213 both indicate failure but at different levels: 213 is a technical validation failure (format/schema); 210 is a business-level refusal by the buyer after the invoice was technically accepted

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

## Use Cases: Outbound Scenarios

### UC 0 — Standard B2B Invoice (Baseline)

- **Volume share:** 40–55% — the most common invoice type by far
- Single order, single delivery, direct buyer-to-seller payment, no third parties, no special VAT regime, no advance payments
- The default against which all other use cases are defined as deviations
- BT-3 code: 380 (commercial invoice)
- Standard lifecycle: Déposée → Reçue → Approuvée → Encaissée
- CDAR: 200 on delivery; 212 on payment
- E-reporting: Flow 1 (automatic, per-invoice)
- **This must work flawlessly before addressing any other use case**

### UC 1 — Multi-Order / Multi-Delivery

- **Volume share:** 25–35%
- One invoice covers several POs and/or several deliveries
- Invoice content must support many-to-many links between invoice, orders, and deliveries while preserving document traceability
- Transmission path is identical to UC 0; the complexity is in the ERP data extraction and reference fields
- E-reporting: Flow 1 (standard domestic B2B)

### Intercompany Invoices

- **Volume share:** 15–25%
- Invoices between separate legal entities within the same corporate group
- If both entities are distinct French-established taxable persons with separate SIRETs, treated as normal domestic B2B e-invoicing — no shortcut
- Each legal entity's SIRET must be individually registered in the Annuaire
- Same PD OIC → InvoiceAgility → PAe flow; InvoiceAgility routes based on legal entity SIRET, not business unit
- Cross-border intercompany (one entity outside France) shifts to e-reporting (Flow 10.1) rather than domestic e-invoicing
- Key setup: distinguish legal entity from business unit in master data

### UC 34 — Partial Collection & Payment Reversals

- **Volume share:** 8–15%
- Tracking partial payments and reversal/cancellation of payments
- Event-based payment model: each partial payment is a distinct collection event with its own date, amount, and method
- Reversals preserved — cancellation of a prior collection is recorded as a separate event, not an override
- Payment data reporting (Flow 10.2) reflects actual collection history: partials, reversals, and net collected position
- CDAR 212 (paid) sent only at full settlement — not on partial payments
- Matters most when VAT is due on collection (TVA sur encaissements) — default for the entire service economy

### UC 20 — Advance Payment (Down-Payment) Invoice

- **Volume share:** 5–10%
- An advance/deposit invoice is issued before the final invoice
- Must be a compliant structured invoice with its own BT-1 and lifecycle
- VAT may be due at the advance stage depending on the nature of the supply
- E-reporting: Flow 1 per advance invoice; payment data (Flow 10.2) per advance collection

### UC 21 — Final Invoice After Advance Payment

- **Volume share:** 5–10%
- Final invoice settles or offsets one or more prior advance invoices
- Must reference prior advance invoice(s) and show the balance due after offset
- Reporting must avoid duplicate turnover/tax representation — the advance VAT was already reported
- Key: offset mechanics and references to prior advances in the structured XML

### UC 2 — Invoice Already Paid by Buyer

- **Volume share:** 3–6%
- Invoice issued after the buyer has already paid at time of issuance
- Must be issued in compliant structured format with a workflow representing an already-paid state
- May require payment status transmission or lifecycle status showing invoice is settled at issuance
- CDAR 212 timing: payment event may precede or coincide with invoice transmission

### UC 31 — Mixed Invoice

- **Volume share:** 3–6%
- One invoice contains a main (principal) transaction and an ancillary (accessory) transaction
- VAT category of the main operation governs the accessory operations
- Complexity is in the payload: line-level tax logic must correctly classify principal vs. accessory
- Transmission path is standard; the challenge is ERP data extraction and InvoiceAgility mapping

### UC 32 — Monthly Payments / Recurring Billing

- **Volume share:** 5–10%
- Recurring billing arrangements (energy, telecom, SaaS, leasing, maintenance)
- Each billing period generates its own compliant invoice through the standard flow
- Invoice timing and payment timing may be decoupled; estimated payments may generate e-reporting until annual reconciliation invoice
- Payment data reporting (Flow 10.2) reflects actual collection dates — matters when VAT is due on collection

### UC 19b — Self-Billing

- **Volume share:** 2–5%
- The buyer issues the invoice on behalf of the supplier
- BT-3 code: 389 (only acceptable code for self-billing)
- SIREN/SIRET positions are reversed: buyer's identifiers in seller fields
- The invoice is delivered to the supplier via their PAr — they receive it and respond with acceptance or rejection
- InvoiceAgility correctly attributes the supplier's legal identity and VAT responsibility, and flags it as self-billed in the XML payload
- Self-billing agreement must be legally in place before transmission

### UC 22a — Early Payment Discount (Cash Accounting VAT)

- **Volume share:** 1–2%
- Early payment discount applied to services where VAT is due on collection (TVA sur encaissements)
- Invoice logic must handle discount effects in a cash-accounting VAT context
- Payment and collection dates become important for VAT reporting
- Configurable discount logic tied to VAT exigibility at collection

### UC 22b — Early Payment Discount (Accrual/Debit-Basis VAT)

- **Volume share:** 1–3%
- Early payment discount applied to goods, or services under accrual VAT / debits option
- VAT timing differs from cash-accounting cases — discount reduces the taxable base at invoice time
- Payment reporting less central than in cash-accounting scenarios, but reconciliation remains important
- Tax engine must distinguish cash vs. accrual VAT treatment

### Other Outbound Use Cases

| UC | Name | Volume | Key Difference from UC 0 |
|---|---|---|---|
| 38 | Sub-lines and line grouping | 10–20% | Hierarchical line structure (GROUP/DETAIL/INFORMATION types) |
| 3 | Third-party payer known at invoicing | 8–12% | Must identify paying third party in addition to buyer |
| 8 | Invoice payable to designated third party | 8–12% | Beneficiary differs from supplier (e.g., factor) |
| 5 | Employee-paid expenses with invoice | 5–8% | Invoice in company's name but reimbursement flow |
| 11 | Invoice sent to third party on behalf of buyer | 5–8% | Routing recipient differs from legal buyer |
| 10 | Beneficiary unknown at creation | 3–5% | Payee details added later; lifecycle updates |
| 36 | Professional secrecy / sensitive data | 2–5% | Generic descriptions in Flow 1; detail only in invoice |
| 7 | Lodge card / purchasing card | 2–4% | Payment instrument differs from normal AP |
| 12 | Transparent intermediary (buyer side) | 2–4% | Intermediary processes without replacing buyer |
| 4 | Partially covered by third party | 1–3% | Split-payment responsibility |
| 9 | Third party managing order/reception/invoicing | 1–3% | Complex supply chain role mapping |
| 13 | Subcontracting with direct payment | 1–3% | Direct payment and delegation roles |
| 17a | Payment intermediary | 1–3% | Marketplace/platform model |
| 40 | Netting / offsetting | 1–3% | Settlement by offset; two separate invoices required |
| 15 | Third party orders/pays on behalf of buyer | 1–2% | Ordering party differs from buyer |
| 19a | Invoice under billing mandate | 1–2% | Third party issues invoice on supplier's behalf |
| 26 | Contractual reservation clause | 1–2% | 5% retention withheld; split payment lifecycle |
| 39 | Multi-seller | 0.5–2% | Multiple sellers grouped on one invoice |
| 17b | Payment intermediary + billing mandate | 0.5–1.5% | Combined marketplace + mandated invoicing |
| 25 | Vouchers and gift cards | 0.5–1.5% | BUU (single-purpose) in scope; BUM (multi-purpose) out of scope |
| 33 | Margin VAT scheme | 0.5–1.5% | VAT amount NOT shown separately |
| 30 | VAT already collected | 0.5–1% | Must avoid double reporting |
| 28 | Restaurant receipts | 0.3–0.8% | B2B only if >€150 HT or buyer requests invoice |
| 27 | Toll tickets | 0.2–0.5% | Corporate fleet = B2B; cash/card = B2C e-reporting |
| 14 | B2B co-contracting | <0.5% | Multiple contractors on same arrangement |
| 16 | Disbursements | <0.5% | Reimbursement, not resale |
| 18 | Debit note | <0.5% | True debit notes excluded from scope; only VAT invoices in scope |
| 29 | VAT group (assujetti unique) | Volume reducer | Intra-group = out of scope; external uses shared SIREN |
| 23 | Self-billing (individual + professional) | <0.1% | Non-standard party setup |
| 37 | Unincorporated joint venture (SEP) | <0.1% | No SIREN; members invoice individually |
| 42 | Tax refund / duty-free | <0.1% | B2B = corrective invoicing only |
| 35 | Author's statements | <0.05% | Non-standard document type |
| 41 | Barter | <0.05% | Two invoices required; VAT due immediately |
| 6 | Employee expenses without invoice | ~0% | Out of scope (no formal invoice) |
| 24 | Earnest money (arrhes) | ~0% | Out of scope (not consideration for supply) |

## Peppol for Outbound

### Sending to Cross-Border Recipients via Peppol

- DGFiP is the French Peppol authority — certified French PAs can connect to the European Peppol network, enabling delivery to any Peppol-registered recipient in the EU and beyond
- If your PA supports Peppol, you can reach EU Peppol-registered recipients using UBL 2.1 formatted to Peppol BIS 3.0; your PA handles the network routing transparently
- E-reporting to DGFiP is still required for all cross-border transactions, even those delivered via Peppol — Peppol delivery does not satisfy the e-reporting obligation
- Verify Peppol capability explicitly with your PA before assuming cross-border delivery is supported — not all certified PAs have activated Peppol connectivity
