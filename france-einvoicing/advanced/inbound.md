---
title: Receiving E-Invoices in
created: 2026-05-09
modified: 2026-05-09
topics:
  - france
  - compliance
  - e-invoicing
doc_type: guide
status: raw
summary: >
  title: Receiving E-Invoices in France — Advanced
keywords:
  - ai
  - ap
  - ar
  - compliance
  - france
  - fraud
  - inbound
  - invoice
---


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

### Attachments: BG-24 Additional Supporting Documents

- EN 16931 defines `BG-24` (Additional Supporting Documents) as the mechanism for attaching or referencing supporting files alongside an invoice
- Three permitted attachment modes in CIUS-FR: embedded binary (Base64-encoded within the XML), external URI reference, and hash + reference pair for externally stored documents
- Your AP system must handle all three modes: extract embedded attachments, follow URI references, and retain hash metadata — BG-24 content is part of the archived invoice package
- PA validation covers BG-24 structural conformance (correct XML element usage); it does not validate the content of attachments or the accessibility of external URIs — broken URIs will not cause rejection at the PA but will cause processing failures downstream
- Common attachment types: delivery notes, purchase orders, contracts, proof of acceptance — attachment type is at the sender's discretion

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

### CDAR Message Codes

| Code | Status | Meaning | Direction |
|------|--------|---------|-----------|
| 200 | Delivered | Invoice successfully transmitted to recipient PAr | PAe → PPF |
| 210 | Refused | Business refusal by buyer (e.g., duplicate, wrong PO) | PAr → PPF |
| 212 | Paid | Payment confirmed; full settlement reported to DGFiP | PAe → PPF |
| 213 | Rejected | Technical validation failure at receiving PA | PAr → PPF |

- CDAR codes are the machine-readable status messages exchanged between PAs and relayed to DGFiP via PPF — they are the transport-level encoding of the lifecycle statuses described above
- As a receiver, codes 210 and 213 are the ones you or your system trigger: 213 when your PA rejects an invoice technically; 210 when you raise a business refusal
- Code 200 fires automatically when your PA accepts delivery — no action required from you
- Code 212 is triggered by a payment event — typically from your AP/treasury system signaling to your PA that payment has been made

### Status Lifecycle: Timing and SLA Expectations

| Transition | Indicative Timing | Governed by SLA? |
|---|---|---|
| Déposée → Reçue (PA validates and routes) | Minutes to 1 hour under normal conditions | PA service contract, not DGFiP spec |
| Reçue → Approuvée (recipient acknowledges) | Recipient-defined; typically 24–72 hours for automated systems | No regulatory deadline |
| Rejetée returned to sender | Near-synchronous in most PA implementations | Per PA implementation |
| Status relay back to sender ERP | Synchronous for REST; queued for AS2/AS4 | Protocol-dependent |

- Status timing is not standardised by DGFiP at the invoice level — transmission SLAs are between you and your PA, established in your service agreement
- For AS2/AS4 flows, the MDN (Message Disposition Notification) confirms transport delivery; invoice status is a separate, subsequent event — do not confuse transport acknowledgement with invoice status
- The legally relevant timestamp is the Déposée timestamp in the PA's audit log, not when your ERP received the status update — status delivery latency does not shift your compliance clock

## Credit Notes and Corrective Invoices

### BT-3 Invoice Type Codes

| BT-3 Code | Document Type | Use Case |
|---|---|---|
| 380 | Commercial Invoice | Standard supply of goods or services |
| 381 | Credit Note | Full or partial reversal of a prior invoice; reduces the amount owed |
| 383 | Debit Note | Upward adjustment to a prior invoice; increases the amount owed |
| 384 | Corrective Invoice | Error correction that replaces the original invoice |
| 389 | Self-Billed Invoice | Buyer generates the invoice on behalf of the supplier |

### Receiving a Credit Note or Corrective Invoice

- Credit notes (381), debit notes (383), and corrective invoices (384) follow the same Y-model path and PA validation rules as commercial invoices — no special routing or separate channel
- All three types must reference the original invoice via `BT-25` (Preceding Invoice Reference) — your AP system should validate that the referenced invoice exists in your records before posting
- A corrective invoice (384) is not a cancellation — it replaces the original and must be treated as the authoritative version for that transaction; mark the original as superseded
- VAT on a credit note reverses the VAT from the original invoice — process the reversal correctly in your AP system, not as a new negative charge

### Invoice Amendment: No In-Flight Modification

- Once an invoice has status Déposée, it cannot be modified — PA transmission is immutable
- The only compliant amendment path is: (1) sender issues Annulée status (cancellation by mutual agreement), then (2) sender reissues a new corrective invoice (BT-3 = 384) or credit note (381) with a new `BT-1` invoice number
- Annulée requires agreement from both parties — unilateral cancellation by the sender is not sufficient; both PAs must confirm the status
- If the error is discovered after Approuvée status, the correct flow is credit note (reversal) + new corrected invoice — not cancellation; Annulée is only available before Approuvée

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

### Invoice Deduplication

- Every invoice submitted to a PA is fingerprinted by sender SIREN + invoice number (`BT-1`); the PA rejects resubmissions with an identical fingerprint from the same sender
- Deduplication is PA-level, not DGFiP-level — the exact deduplication key (SIREN + `BT-1`, or SIRET + `BT-1`) varies by PA; confirm the key with your PA during integration testing
- A corrected resubmission (after Rejetée) may require a new invoice number if the PA registered the original number in its deduplication registry — verify with your PA whether Rejetée submissions are registered or discarded
- For idempotent retry scenarios (network failure after submission, before acknowledgement received), do not automatically resubmit — query the PA for the status of the previous submission first to avoid a duplicate rejection

## Special Invoice Scenarios

### Self-Billing (Auto-Facturation)

- In a self-billing arrangement, the buyer (AP function) generates the invoice on behalf of the supplier — the buyer acts as the sender in the PA network, not the receiver
- BT-3 code for self-billed invoices: **389** — this is the only code acceptable for self-billing; submitting a self-billed invoice with BT-3 = 380 will fail CIUS-FR validation
- CIUS-FR still requires SIREN/SIRET for both parties, but the positions are reversed: the buyer's SIREN/SIRET appears in the seller fields (`BT-29/BT-30`) and the supplier's SIREN/SIRET appears in the buyer fields (`BT-46/BT-49`)
- The self-billing agreement must be legally in place before invoices under it can be transmitted — DGFiP does not validate the existence of the agreement, but it must be available for audit
- From your AP perspective as the buyer-sender: your PA handles the outbound transmission, but payment data reporting for these invoices remains your obligation

### Multi-Entity and Group Structures

- A PA can service multiple SIRENs/SIRETs under a single technical connection — a corporate group connects once and receives invoices for all French legal entities through one PA integration
- Each legal entity (SIRET) must be individually registered in the Annuaire — the Annuaire entry maps each SIRET to the group's PA; the PA must route by recipient SIRET, not just by SIREN
- Inter-company invoices within a group (entity A invoices entity B) follow the same Y-model path as external supplier invoices — there is no internal shortcut; both SIRETs must be in the Annuaire
- If different group entities use different PAs, inter-company invoices must transit both PAs via the interoperability layer — validate this flow during pre-production testing, as it exercises the full PA-to-PA handoff

## Testing and Pre-Production

### Integration Testing: What to Validate

- DGFiP opened a formal interoperability testing environment in October 2025 for PA-to-PA connectivity validation — the formal DGFiP-managed window closed January 14, 2026; your PA's own sandbox environment is now the primary testing channel
- Request sandbox access from your PA during onboarding — sandbox credentials, test SIREN/SIRET values, and test PA identifiers are PA-specific; do not use live production SIRENs in testing
- For format validation without a PA sandbox, use the FNFE-MPE online validator — publicly available, validates UBL 2.1, CII, and Factur-X against EN 16931 schematron rules and CIUS-FR extensions; returns the same BR-FR-CTC error codes your PA will return
- End-to-end test scenarios to cover before go-live:

| Scenario | What It Tests |
|---|---|
| Valid invoice → PA acceptance | Happy path: format, CIUS-FR fields, Annuaire routing |
| Missing SIREN → structured rejection | CIUS-FR validation error handling in your ERP |
| Business refusal by recipient | Refusée status flow back to sender system |
| Credit note with BT-25 reference | Credit note round-trip and AP matching |
| Status messages back to ERP | Déposée, Reçue, Approuvée returned correctly |

## Connectivity

### AS2/AS4 vs. REST API

| Protocol | Type | Best For | Notes |
|---|---|---|---|
| AS2 | EDI, async | High-volume batch; existing AS2 infrastructure | Established standard; widely supported by EDI middleware |
| AS4 | EDI, async | Same as AS2 with WS-Security layer | Preferred for new EDI implementations requiring message-level security |
| REST API | HTTP, sync/event | Modern integrations; event-driven architectures | DGFiP spec v3.1 defines exact endpoints; OAuth 2.0 (client credentials flow) for authentication |

### REST API: Technical Detail

| Aspect | Specification |
|---|---|
| Authentication | OAuth 2.0 client credentials flow — request a bearer token from the PA's token endpoint; include as `Authorization: Bearer <token>` on every API call |
| Token lifetime | Typically 3,600 seconds (1 hour); read `expires_in` from the token response — do not hard-code; refresh before expiry |
| Invoice submission | `POST` to the PA's invoice endpoint with the invoice payload as `multipart/form-data` or `application/xml`; exact path defined per PA, not standardised in DGFiP spec |
| Status query | `GET /invoices/{id}/status` (or PA-equivalent); poll periodically — coordinate polling interval with your PA to avoid rate-limit errors |
| Webhook / push events | Some PAs support push notification callbacks as an alternative to polling — not mandated by DGFiP spec; PA-specific; reduces latency and polling overhead |
| Error responses | HTTP 4xx for validation failures; PA returns a structured error body containing the DGFiP error code and the specific BR-FR-CTC rule that failed; HTTP 5xx for PA-side faults |

- Implement token refresh as a background process, not inline with invoice submission — a token expiry mid-batch should not cause invoice loss
- Treat `POST` submission as non-idempotent unless your PA explicitly supports idempotency keys — on network failure, query for invoice status before deciding whether to resubmit

### Peppol Interoperability

- AIFE (Agence pour l'Informatique Financière de l'État, via Chorus Pro) is the designated French Peppol Authority — the national access point connecting France to the European Peppol network; DGFiP governs the mandate but AIFE holds the Peppol authority role
- Cross-border invoices arriving from EU Peppol senders can reach French recipients via their PA; the PA acts as the bridge between the Peppol network and the domestic Y-model
- Peppol BIS 3.0, which is built on UBL 2.1, is the compatible format for cross-border Peppol invoices entering the French network
- For multinational supply chains: a French recipient connected to a single PA can receive invoices from both domestic French senders and EU Peppol-enabled senders without separate onboarding per country

### Peppol 4-Corner Model: Mapping to the Y-Model

| Peppol Concept | Y-Model Equivalent | Notes |
|---|---|---|
| Corner 1 — sender's Access Point | Sender's PA | French sender's PA acts as the Peppol Access Point for outgoing cross-border invoices |
| Corner 2 — receiver's Access Point | Recipient's PA | Recipient's PA receives Peppol-delivered invoices and injects them into the domestic Y-model flow |
| Corner 3 — SMP (Service Metadata Publisher) | Annuaire | Peppol SMP lookup identifies the recipient's Access Point; Annuaire performs this function for France-domestic routing |
| Corner 4 — SML (Service Metadata Locator) | Not directly applicable | SML is the Peppol-wide root directory; within France, the Annuaire is the equivalent |

- A French recipient registered in the Annuaire with a Peppol-capable PA automatically receives invoices from EU Peppol senders — no separate Peppol registration is required beyond your PA onboarding
- Peppol cross-border invoices use BIS 3.0 (UBL 2.1 profile) for the international segment; CIUS-FR extensions apply only to the domestic French leg of the transaction
- Not all certified PAs have activated Peppol connectivity — confirm Peppol capability explicitly with your PA during selection if cross-border receipt is in scope

## Archiving

### Legal Archiving Requirements

- French commercial law (`Code de commerce L.123-22`) requires accounting records to be retained for 10 years; French tax law (`Livre des procédures fiscales`) sets a minimum of 6 years for tax documents including invoices — retain for 10 years to satisfy both obligations simultaneously
- The archive must preserve the original structured file (e.g., UBL 2.1 or CII XML) AND the human-readable representation (e.g., the PDF layer in Factur-X) — both are legally required
- Altering, converting, or re-rendering the archived format voids its legal evidentiary value; what was received must be what is stored
- DGFiP auditors can request access to the archive at any time during the retention period; PA-based certified archiving services typically provide a compliant audit export capability that satisfies this obligation

### Archiving Technical Standards

| Standard | Scope |
|---|---|
| NF Z42-013 | French standard for trustworthy archival of electronic documents — defines integrity protection (hash + signature), chain-of-custody metadata, and audit log requirements |
| NF Z42-020 | Extends NF Z42-013 for digital archiving systems; referenced in PA certification requirements |
| EN 16931 XML payload | The original structured file (UBL 2.1 or CII XML) must be archived unmodified — format conversion or re-rendering voids evidentiary value |
| PDF/A-3 (Factur-X) | The complete PDF/A-3 container with embedded XML must be archived as received; do not strip the XML attachment |

- NF Z42-013 certification is typically held by your PA or a dedicated archiving service provider — you do not implement it yourself unless you operate your own archive infrastructure
- To satisfy a DGFiP audit request, the archive must produce: the original file, its hash as recorded at time of archival, and the chain-of-custody log — confirm your PA or archive provider can generate this export
- Cloud-based certified archiving is available as a managed service from most certified PAs — confirm during PA selection whether archiving is included in the service or priced separately

## Payment Data Reporting

### Payment Reporting Mechanics for Received Invoices

| Field | Source | Notes |
|---|---|---|
| Invoice reference | BT-1 from the received invoice | Must match the invoice identifier exactly as submitted |
| Payment date | ERP settlement date | Actual settlement date, not the due date |
| Payment amount | Amount settled | Report each instalment separately for partial payments |
| Payment method | ERP / treasury system payment record | SEPA transfer, card, cash, cheque — per DGFiP code list |

- Payment data reporting for received B2B invoices is an AP-side obligation — your system must signal payment completion to your PA, which submits the event to DGFiP
- Reporting is periodic, not real-time — DGFiP implementing decrees define the reporting window; typical alignment is with VAT return periods (monthly or quarterly depending on your VAT regime)
- Submission uses the same PA API or EDI channel as invoice status updates — your PA documentation defines the specific endpoint and payload schema for payment event data
- The obligation is active from your mandatory receipt date (September 1, 2026) — payment reporting is not deferred to your sending deadline

### E-Reporting Flow Type Codes (Receiver Perspective)

| Flow | Scope | Your Role as Receiver |
|------|-------|-----------------------|
| 1 | Domestic B2B invoice data | Sender's obligation — automatic during their transmission |
| 8 | Domestic B2B invoice data relay (PA → PPF) | Automatic — your PA relays on receipt |
| 9 | Domestic B2B lifecycle/status data relay (PA → PPF) | Automatic — your PA relays your status responses |
| 10.2 | Payment data for all invoice flows | **Your obligation** — triggered when you pay a received invoice |

- As a receiver of domestic B2B invoices, Flow 10.2 is your primary e-reporting obligation — you must report payment events for invoices you receive
- Flows 1, 8, and 9 are handled by the sender's PA or automatically by the PA infrastructure — no action required from you
- Flow 10.2 data: invoice reference, payment date, amount paid, payment method — submitted via your PA when your ERP/treasury system confirms payment

## Use Cases: Inbound Scenarios

### UC 0 — Receiving a Standard B2B Invoice (Baseline)

- **Volume share:** 40–55%
- Single order, single delivery, direct payment from you to the supplier
- Standard inbound lifecycle: Reçue → Approuvée → Encaissée
- Your AP system matches to PO, validates, posts, and triggers payment
- CDAR: your PA fires 200 on receipt; you trigger approval; your system triggers 212 on payment
- E-reporting: Flow 10.2 when you pay
- **This must work flawlessly before addressing any other use case**

### UC 1 — Receiving a Multi-Order / Multi-Delivery Invoice

- **Volume share:** 25–35%
- Invoice references multiple POs and/or deliveries from the same supplier
- AP matching complexity: your system must match one invoice against multiple PO/GR combinations
- No special e-reporting treatment; standard Flow 10.2 on payment

### Receiving Intercompany Invoices

- **Volume share:** 15–25%
- Invoice from a sister entity within the same group — treated as normal domestic B2B if both are French-established
- Each legal entity's SIRET is independently registered in the Annuaire
- Your PA routes by SIRET, not by group affiliation — each entity's AP system processes independently
- Cross-border intercompany from a non-French entity arrives as a standard inbound (if via Peppol) or outside e-invoicing scope entirely

### UC 34 — Receiving Invoices with Partial Collection

- **Volume share:** 8–15%
- You make partial payments over time; each is a distinct collection event
- Your AP/treasury system must report each partial payment to your PA (Flow 10.2) individually
- Reversals: if a prior payment is cancelled, report the reversal as a separate event — do not overwrite
- CDAR 212 sent only at full settlement
- Critical for services where VAT is due on collection

### UC 20/21 — Down-Payment and Final Invoice Sequences

- **Volume share:** 5–10% each
- UC 20: you receive an advance invoice — process and pay it; report via Flow 10.2
- UC 21: you receive the final invoice offsetting prior advances — your AP must reconcile the advance(s) against the final balance
- Avoid double-posting the VAT already reported on the advance invoice

### UC 2 — Receiving an Already-Paid Invoice

- **Volume share:** 3–6%
- Invoice arrives after you've already paid (e.g., prepaid orders)
- Your AP system must recognize the already-paid state and reconcile without re-triggering payment
- Payment data (Flow 10.2) timing: payment event may have occurred before the invoice was received

### UC 31 — Receiving a Mixed Invoice

- **Volume share:** 3–6%
- Invoice contains principal and accessory operations with different tax treatments
- Your AP system must handle line-level VAT correctly — the main operation's VAT category governs accessories
- Standard receipt flow; complexity is in the posting and tax recovery logic

### UC 32 — Monthly/Recurring Invoices

- **Volume share:** 5–10%
- Recurring invoices (energy, telecom, SaaS, leasing)
- Each period's invoice is a separate document in the PA network
- Payment data reporting per actual collection date — critical for VAT on collection scenarios

### UC 19b — Receiving a Self-Billed Invoice

- **Volume share:** 2–5%
- In this scenario, YOU (the buyer) issued the invoice on behalf of the supplier — but the supplier receives it via their PAr
- If you are the supplier in a self-billing arrangement: the invoice arrives at your PAr with BT-3 = 389
- Your AP system must recognize the reversed SIREN/SIRET positions and process accordingly
- You can accept or reject it via standard lifecycle statuses

### UC 22a / 22b — Receiving Discounted Invoices

- **Volume share:** 1–3% combined
- UC 22a: early payment discount on services with VAT due on collection — discount affects VAT timing
- UC 22b: early payment discount on goods/accrual VAT — discount reduces taxable base at invoice time
- Your AP system must apply the correct discount and VAT treatment based on the invoice terms

### Other Inbound Use Cases

| UC | Name | Volume | Key Receiver Consideration |
|---|---|---|---|
| 38 | Sub-lines and line grouping | 10–20% | AP system must parse hierarchical line structures |
| 3 | Third-party payer identified | 8–12% | You may not be the payer despite being the buyer |
| 8 | Payable to designated third party | 8–12% | Payment goes to factor/treasury center, not supplier |
| 5 | Employee expenses with invoice | 5–8% | Invoice in company name; reimbursement is internal |
| 11 | Invoice sent to third party for buyer | 5–8% | You may receive via a third-party processor |
| 10 | Beneficiary unknown | 3–5% | Payee details may arrive later via lifecycle update |
| 36 | Professional secrecy | 2–5% | Sensitive data handling required |
| 7 | Lodge/purchasing card payment | 2–4% | Reconcile against card statements |
| 12 | Transparent intermediary | 2–4% | Intermediary processes on your behalf |
| 26 | Contractual reservation | 1–2% | 5% retention; split payment lifecycle |
| 33 | Margin VAT scheme | 0.5–1.5% | No VAT shown; cannot recover VAT |
| 29 | VAT group | Volume reducer | Intra-group out of scope |

## From Spec to Product

### What the Product Level Covers

- You now have the full technical picture: spec reference, validation rules, status lifecycle, error handling, connectivity options, Peppol interoperability, and archiving obligations
- The product level applies this knowledge to Tungsten's specific AP and AR products — showing precisely where each product sits in the Y-model flow and how it implements these requirements
- Topics covered: PA connectivity and integration architecture, format handling within Tungsten, ERP handoff mechanisms, operator workflows for exceptions, and e-reporting automation
