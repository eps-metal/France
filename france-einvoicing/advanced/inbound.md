# Receiving E-Invoices in France — Advanced

## Technical Reference

### DGFiP External Specifications v3.1

- The authoritative technical reference for France's e-invoicing mandate — defines API endpoints, status codes, data schemas, and business rules that every PA must implement
- Current version is v3.1; this is the authoritative version for the September 2026 go-live
- PA vendors are required to implement the spec; vendor-specific documentation is secondary — when in doubt, the DGFiP spec governs
- Available directly from DGFiP; any integration work should be validated against v3.1, not against PA vendor summaries alone

## Invoice Validation

### Schematron Validation: The Business Rules Layer

- Schematron is an XML rule-based validation language; the DGFiP spec defines a set of Schematron rules that are applied to every inbound invoice at the PA before acceptance or routing
- Validation runs at the PA level — the invoice never reaches the recipient if it fails
- A substantial set of mandatory business rules (per DGFiP External Specifications v3.1) covers: mandatory field presence, SIREN format correctness (9 digits, no letters), VAT calculation accuracy (amount must match rate × taxable base), and line-item totals matching the invoice header total
- A validation failure returns a structured XML error response containing the error code, a description, and the specific rule that was violated — not a generic rejection

### What a Validation Failure Looks Like

| Error Category | Example Rule | PA Response |
|---|---|---|
| Missing mandatory field | SIREN absent on seller | Reject with code + rule reference |
| Format error | SIREN not 9 digits | Reject with code + rule reference |
| Calculation mismatch | VAT amount doesn't match rate × base | Reject with code + rule reference |
| Business rule violation | Payment terms exceed legal maximum | Reject with code + rule reference |

## Invoice Status Lifecycle

### Status States: Full Technical View

| French Name | English | Trigger | Who Sets It | Action Required |
|---|---|---|---|---|
| Déposée | Deposited | Invoice submitted to sending PA | Sender's PA | None — awaiting routing |
| Reçue | Received | Delivered to recipient PA | Recipient's PA | Process or acknowledge |
| Approuvée | Approved | Recipient system confirms receipt | Recipient (via PA) | None — invoice in process |
| Rejetée | Rejected | Failed technical/format validation | PA (either) | Sender must correct and resubmit |
| Refusée | Refused | Recipient disputes on business grounds | Recipient (via PA) | Dispute resolution process |
| En Anomalie | Under Query | Issue raised needing clarification | Recipient (via PA) | Respond to query |
| Annulée | Cancelled | Invoice withdrawn by mutual agreement | Sender (via PA) | None |
| Encaissée | Payment Reported | Payment event reported to DGFiP | Recipient (via PA) | None |

### Status Flow: The Happy Path and Exception Paths

- **Happy path**: Déposée → Reçue → Approuvée → Encaissée — invoice submitted, delivered, approved, and payment reported to DGFiP
- **Rejection path**: Déposée → Rejetée — technical or format validation fails at the PA; sender must correct the source invoice and resubmit as a new submission
- **Refusal path**: Déposée → Reçue → Refusée — invoice is technically valid and legally exists, but the recipient raises a business-level dispute; requires resolution between buyer and seller outside the PA flow
- **En Anomalie path**: Déposée → Reçue → En Anomalie → (clarification exchanged) → Approuvée — recipient raises a query rather than an outright refusal; invoice remains live while the issue is resolved

## Error Handling and Disputes

### Rejection vs. Refusal — A Critical Distinction

| Dimension | Rejection (Rejetée) | Refusal (Refusée) |
|---|---|---|
| Nature | Technical failure | Business dispute |
| Invoice status | Never legally existed | Valid invoice, recipient contests |
| Who can trigger | PA (automatically) | Recipient (deliberate action) |
| Resolution | Sender corrects and resubmits | Dispute resolution between parties |
| DGFiP notified | Yes (via PA) | Yes (via PA) |

### Handling a Rejected Inbound Invoice — Step by Step

1. PA notifies your system with a structured XML error containing the error code and the specific failed rule
2. Your AP system or operator identifies the exact validation failure from the error detail
3. Notify the sending supplier with the full error detail — not just "rejected"
4. Supplier corrects the invoice in their ERP and resubmits via their PA
5. The corrected submission is treated as a fresh invoice — a new status lifecycle begins from Déposée

## Connectivity

### AS2/AS4 vs. REST API

| Protocol | Type | Best For | Notes |
|---|---|---|---|
| AS2 | EDI, async | High-volume batch; existing AS2 infrastructure | Established standard; widely supported by EDI middleware |
| AS4 | EDI, async | Same as AS2 with WS-Security layer | Preferred for new EDI implementations requiring message-level security |
| REST API | HTTP, sync/event | Modern integrations; event-driven architectures | DGFiP spec v3.1 defines exact endpoints; OAuth 2.0 (client credentials flow) for authentication |

### Peppol Interoperability

- AIFE (Agence pour l'Informatique Financière de l'État, via Chorus Pro) is the designated French Peppol Authority — the national access point connecting France to the European Peppol network; DGFiP governs the mandate but AIFE holds the Peppol authority role
- Cross-border invoices arriving from EU Peppol senders can reach French recipients via their PA; the PA acts as the bridge between the Peppol network and the domestic Y-model
- Peppol BIS 3.0, which is built on UBL 2.1, is the compatible format for cross-border Peppol invoices entering the French network
- For multinational supply chains: a French recipient connected to a single PA can receive invoices from both domestic French senders and EU Peppol-enabled senders without separate onboarding per country

## Archiving

### Legal Archiving Requirements

- French commercial law (`Code de commerce L.123-22`) requires accounting records to be retained for 10 years; French tax law (`Livre des procédures fiscales`) sets a minimum of 6 years for tax documents including invoices — retain for 10 years to satisfy both obligations simultaneously
- The archive must preserve the original structured file (e.g., UBL 2.1 or CII XML) AND the human-readable representation (e.g., the PDF layer in Factur-X) — both are legally required
- Altering, converting, or re-rendering the archived format voids its legal evidentiary value; what was received must be what is stored
- DGFiP auditors can request access to the archive at any time during the retention period; PA-based certified archiving services typically provide a compliant audit export capability that satisfies this obligation

## From Spec to Product

### What the Product Level Covers

- You now have the full technical picture: spec reference, validation rules, status lifecycle, error handling, connectivity options, Peppol interoperability, and archiving obligations
- The product level applies this knowledge to Tungsten's specific AP and AR products — showing precisely where each product sits in the Y-model flow and how it implements these requirements
- Topics covered: PA connectivity and integration architecture, format handling within Tungsten, ERP handoff mechanisms, operator workflows for exceptions, and e-reporting automation
