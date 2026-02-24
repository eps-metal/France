# France E-Invoicing — Gap Fill Iteration 2 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add 18 missing content sections across intermediate and advanced files to make the france-einvoicing series complete and production-ready.

**Architecture:** Each task edits one or two existing Markdown files by inserting exact content at a specified location. No new files are created. All content is pre-written in this plan — the implementer copies it exactly. Slide convention: H1 = deck title, H2 = section divider, H3 = individual slide, body = 3–6 bullets or one table. Never exceed 6 bullets per H3.

**Tech Stack:** Markdown, git

---

## Context: Gap Map

| Gap | Description | Files Affected |
|---|---|---|
| 1 | Credit notes / corrective invoices (BT codes, flow, VAT) | advanced/inbound.md, advanced/outbound.md |
| 2 | Self-billing (auto-facturation) | advanced/inbound.md |
| 3 | Multi-entity / group structures | advanced/inbound.md |
| 4 | Testing and pre-production environments | advanced/inbound.md |
| 5 | Attachments: BG-24 supporting documents | advanced/inbound.md |
| 6 | Transition period complexity (2026–2027 dual intake) | intermediate/inbound.md, intermediate/outbound.md |
| 7 | Factur-X profile levels (EN 16931 minimum) | advanced/outbound.md |
| 8 | Annuaire registration process | intermediate/inbound.md |
| 9 | Status lifecycle timing and SLAs | advanced/inbound.md |
| 10 | Invoice deduplication | advanced/inbound.md |
| 11 | REST API technical detail | advanced/inbound.md |
| 12 | Peppol 4-corner model mapping to Y-model | advanced/inbound.md |
| 13 | EN 16931 BT codes for CIUS-FR fields | advanced/outbound.md |
| 14 | Schematron rule specifics (BR-FR-CTC naming, tooling) | advanced/outbound.md |
| 15 | Archiving technical standards (NF Z42-013) | advanced/inbound.md |
| 16 | E-reporting submission mechanics | advanced/inbound.md, advanced/outbound.md |
| 17 | PA certification, decertification, and migration | intermediate/inbound.md, intermediate/outbound.md |
| 18 | Invoice amendment after submission | advanced/inbound.md |

---

### Task 1: intermediate/inbound.md — Annuaire Registration + Transition Period + PA Decertification (Gaps 6, 8, 17)

**Files:**
- Modify: `france-einvoicing/intermediate/inbound.md`

**Step 1: Read the current file**

```bash
cat france-einvoicing/intermediate/inbound.md
```

Identify the line where `## From Spec to Product` appears (currently last section). All three new sections go BEFORE that line.

Also identify the existing `### The Y-Model` or Annuaire section — the Annuaire registration section should be inserted directly after the existing Annuaire content.

**Step 2: Insert Annuaire registration section**

Find the line:
```
### How the Annuaire Works
```
(or whichever H3 currently covers the Annuaire). Insert the following as a new H3 immediately after that section's last bullet:

```markdown
### Registering in the Annuaire

- Registration is performed by your PA on your behalf — you do not register directly with DGFiP or PPF; your PA submits your SIREN/SIRET and connectivity details to the Annuaire
- Your PA will require: your SIREN (company-level identifier) and every SIRET (establishment-level identifier) that will receive invoices, plus any routing preferences if multiple SIRETs share the PA
- Once registered, your entry is visible to all other PAs on the network — any PA routing an invoice to your SIREN/SIRET will find your PA automatically and route accordingly
- For group structures with multiple SIRETs, each establishment must have its own Annuaire entry even if all establishments are served by the same PA
- If you change PA, the old PA must deregister your SIREN/SIRET and the new PA must register it — there is a brief window during migration when routing may fail; schedule migrations carefully

```

**Step 3: Insert transition period section**

Find the last H3 in the existing content (before `## From Spec to Product` or the bridge slide). Insert before `## From Spec to Product`:

```markdown
## Transition Period

### September 2026 to September 2027: Dual-Mode Operation

| Period | Who Must Send | Who Must Receive |
|---|---|---|
| Before September 1, 2026 | No one (mandate not yet active) | No one mandated |
| September 1, 2026 onwards | GE + ETI | All businesses — GE, ETI, SME, micro |
| September 1, 2027 onwards | All businesses | All businesses (already required) |

- **Critical asymmetry**: all businesses must receive e-invoices from September 1, 2026 even if their own sending deadline is September 1, 2027 — SMEs and micro-enterprises cannot defer their PA receipt configuration
- An SME receiving invoices from a large supplier must be PA-connected from September 1, 2026; if they are not, they are non-compliant — not the sender
- During the transition window (Sept 2026 – Sept 2027), large enterprise senders must be prepared to handle routing for SME customers who may not yet be in the Annuaire — this edge case has no formal DGFiP fallback channel; monitor PA guidance

### PA Decertification and Migration

- DGFiP has authority under PLF 2026 to revoke PA certification — a decertified PA cannot legally transmit invoices or e-reports; all in-flight transmissions halt
- You bear legal responsibility for ensuring your PA remains certified — verify against the official DGFiP PA registry periodically; the list changes as new PAs are added or existing ones are reviewed
- Migration to a new PA requires: (1) sign with new PA, (2) new PA registers your SIREN/SIRET in the Annuaire, (3) old PA removes the entry, (4) update your API/EDI connection — coordinate steps 2 and 3 tightly to avoid a routing gap
- Ensure your PA contract specifies notice obligations in the event of decertification and provides for assisted migration — this is a key procurement point during PA selection, not an afterthought

```

**Step 4: Verify**

- Confirm no H3 has more than 6 bullets
- Confirm the Annuaire registration H3 follows directly after the existing Annuaire H3
- Confirm the transition period and PA migration sections appear before the existing bridge/handoff section

**Step 5: Commit**

```bash
git add france-einvoicing/intermediate/inbound.md
git commit -m "content: add annuaire registration, transition period, PA migration to intermediate/inbound"
```

---

### Task 2: intermediate/outbound.md — Transition Period + PA Decertification (Gaps 6, 17)

**Files:**
- Modify: `france-einvoicing/intermediate/outbound.md`

**Step 1: Read the current file**

```bash
cat france-einvoicing/intermediate/outbound.md
```

Identify the end of the existing content — these two sections go at the end before any existing bridge/handoff slide.

**Step 2: Insert transition period section (sender perspective)**

Append before the final H2 (if there is one) or at the end of the file:

```markdown
## Transition Period

### What Senders Must Plan For: 2026 to 2027

| Date | Obligation Active For |
|---|---|
| September 1, 2026 | GE and ETI must send e-invoices via PA; all businesses must receive |
| September 1, 2027 | SMEs and micro-enterprises must send e-invoices via PA |

- From September 1, 2026, GE and ETI must route all domestic B2B invoices via their PA — there is no grace period for individual customers or transaction types
- E-reporting obligations are active from your mandatory sending date — payment data reporting, B2C transaction reporting, and cross-border B2B reporting all begin on the same date as your sending mandate; they are not deferred
- During 2026–2027, SME/micro senders may not yet be PA-connected — if you receive invoices from them, you may still receive paper or PDF during that window; your PA receipt configuration must handle both e-invoice and legacy formats in parallel

### PA Decertification: Sender Risk

- DGFiP can revoke a PA's certification at any time — a decertified PA cannot legally send invoices or submit e-reports; affected senders must migrate immediately to remain compliant
- Monitor the official DGFiP PA registry as part of your compliance programme — certification status is not automatically communicated to PA customers
- Ensure your PA contract includes decertification notice requirements and assisted migration provisions; a PA migration without notice could cause invoice delivery failures with legal and commercial consequences
- If your PA is decertified mid-period, invoices cannot be held and sent later — the legal clock runs from the invoice date; delays caused by PA failure do not suspend penalty exposure

```

**Step 3: Verify**

- Confirm no H3 exceeds 6 bullets
- Confirm transition table has exactly 2 data rows (2026 and 2027)

**Step 4: Commit**

```bash
git add france-einvoicing/intermediate/outbound.md
git commit -m "content: add transition period and PA decertification risk to intermediate/outbound"
```

---

### Task 3: advanced/inbound.md — Credit Notes, Corrective Invoices, Amendment Flow (Gaps 1, 18)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

```bash
cat france-einvoicing/advanced/inbound.md
```

Identify `## Error Handling and Disputes` — insert the new `## Credit Notes and Corrective Invoices` section immediately before it.

**Step 2: Insert credit notes section**

Insert before `## Error Handling and Disputes`:

```markdown
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
- The only compliant amendment path is: (1) sender issues Annulée status (cancellation by mutual agreement), then (2) sender reissues a new corrective invoice (BT-3 = 384) or credit note (381) with a new BT-1 invoice number
- Annulée requires agreement from both parties — unilateral cancellation by the sender is not sufficient; both PAs must confirm the status
- If the error is discovered after Approuvée status, the correct flow is credit note (reversal) + new corrected invoice — not cancellation; Annulée is only available before Approuvée

```

**Step 3: Verify**

- Confirm BT-3 table has 5 rows (380, 381, 383, 384, 389)
- Confirm "Amendment" H3 contains 4 bullets
- Confirm `BT-25` is formatted as inline code

**Step 4: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add credit notes BT codes and amendment flow to advanced/inbound"
```

---

### Task 4: advanced/inbound.md — Self-Billing + Multi-Entity Structures (Gaps 2, 3)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

After Task 3, the file now has the credit notes section before `## Error Handling and Disputes`. Insert the new sections after `## Error Handling and Disputes` (after the last H3 in that section, before `## Connectivity`).

**Step 2: Insert self-billing section**

Insert after the last H3 in `## Error Handling and Disputes`, before `## Connectivity`:

```markdown
## Special Invoice Scenarios

### Self-Billing (Auto-Facturation)

- In a self-billing arrangement, the buyer (AP function) generates the invoice on behalf of the supplier — the buyer acts as the sender in the PA network, not the receiver
- BT-3 code for self-billed invoices: **389** — this is the only code acceptable for self-billing; submitting a self-billed invoice with BT-3 = 380 will fail CIUS-FR validation
- CIUS-FR still requires SIREN/SIRET for both parties, but the positions are reversed: the buyer's SIREN/SIRET appears in the seller fields (BT-29/BT-30) and the supplier's SIREN/SIRET appears in the buyer fields (BT-46/BT-49)
- The self-billing agreement must be legally in place before invoices under it can be transmitted — DGFiP does not validate the existence of the agreement, but it must be available for audit
- From your AP perspective as the buyer-sender: your PA handles the outbound transmission, but payment data reporting for these invoices remains your obligation

### Multi-Entity and Group Structures

- A PA can service multiple SIRENs/SIRETs under a single technical connection — a corporate group connects once and receives invoices for all French legal entities through one PA integration
- Each legal entity (SIRET) must be individually registered in the Annuaire — the Annuaire entry maps each SIRET to the group's PA; the PA must route by recipient SIRET, not just by SIREN
- Inter-company invoices within a group (entity A invoices entity B) follow the same Y-model path as external supplier invoices — there is no internal shortcut; both SIRETs must be in the Annuaire
- If different group entities use different PAs, inter-company invoices must transit both PAs via the interoperability layer — validate this flow during pre-production testing, as it exercises the full PA-to-PA handoff

```

**Step 3: Verify**

- Confirm `## Special Invoice Scenarios` is a new H2 section
- Confirm self-billing H3 has exactly 5 bullets
- Confirm multi-entity H3 has exactly 4 bullets
- Confirm BT-3 code 389 is bolded in the self-billing section

**Step 4: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add self-billing and multi-entity structures to advanced/inbound"
```

---

### Task 5: advanced/inbound.md — Testing Environments + Attachments BG-24 (Gaps 4, 5)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

Identify `## Connectivity` — insert the testing section before it. The attachments section goes into `## Invoice Validation`, after the existing `### What a Validation Failure Looks Like` table.

**Step 2: Insert attachments section into Invoice Validation**

Find the line after `### What a Validation Failure Looks Like` table (after the last `| Business rule violation |` row). Insert:

```markdown
### Attachments: BG-24 Additional Supporting Documents

- EN 16931 defines `BG-24` (Additional Supporting Documents) as the mechanism for attaching or referencing supporting files alongside an invoice
- Three permitted attachment modes in CIUS-FR: embedded binary (Base64-encoded within the XML), external URI reference, and hash + reference pair for externally stored documents
- Your AP system must handle all three modes: extract embedded attachments, follow URI references, and retain hash metadata — BG-24 content is part of the archived invoice package
- PA validation covers BG-24 structural conformance (correct XML element usage); it does not validate the content of attachments or the accessibility of external URIs — broken URIs will not cause rejection at the PA but will cause processing failures downstream
- Common attachment types: delivery notes, purchase orders, contracts, proof of acceptance — attachment type is at the sender's discretion

```

**Step 3: Insert testing environments section before `## Connectivity`**

Insert before `## Connectivity`:

```markdown
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

```

**Step 4: Verify**

- Confirm BG-24 section is inside `## Invoice Validation`
- Confirm testing section is its own `## Testing and Pre-Production`
- Confirm the test scenario table has 5 rows
- Confirm `BG-24` and `BT-25` are formatted as inline code

**Step 5: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add BG-24 attachments and testing environments to advanced/inbound"
```

---

### Task 6: advanced/inbound.md — Status Lifecycle Timing + Deduplication (Gaps 9, 10)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

Find `### Status Flow: The Happy Path and Exception Paths` — insert the timing section immediately after it. Find `### Handling a Rejected Inbound Invoice — Step by Step` — insert the deduplication section immediately after it.

**Step 2: Insert status timing section**

After `### Status Flow: The Happy Path and Exception Paths` (after its last bullet), insert:

```markdown
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

```

**Step 3: Insert deduplication section**

After `### Handling a Rejected Inbound Invoice — Step by Step` (after step 5), insert:

```markdown
### Invoice Deduplication

- Every invoice submitted to a PA is fingerprinted by sender SIREN + invoice number (BT-1); the PA rejects resubmissions with an identical fingerprint from the same sender
- Deduplication is PA-level, not DGFiP-level — the exact deduplication key (SIREN + BT-1, or SIRET + BT-1) varies by PA; confirm the key with your PA during integration testing
- A corrected resubmission (after Rejetée) may require a new invoice number if the PA registered the original number in its deduplication registry — verify with your PA whether Rejetée submissions are registered or discarded
- For idempotent retry scenarios (network failure after submission, before acknowledgement received), do not automatically resubmit — query the PA for the status of the previous submission first to avoid a duplicate rejection

```

**Step 4: Verify**

- Confirm status timing table has 4 rows
- Confirm deduplication section has exactly 4 bullets
- Confirm `BT-1` is formatted as inline code
- Confirm MDN is spelled out on first use

**Step 5: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add status lifecycle timing and deduplication to advanced/inbound"
```

---

### Task 7: advanced/inbound.md — REST API Detail + Peppol 4-Corner Mapping (Gaps 11, 12)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

Find `## Connectivity`. Find `### AS2/AS4 vs. REST API` — the REST API detail section goes immediately after that table. Find `### Peppol Interoperability` — the 4-corner mapping section goes immediately after it.

**Step 2: Insert REST API detail section**

After the `### AS2/AS4 vs. REST API` table (after the `| REST API |` row), insert:

```markdown
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

```

**Step 3: Insert Peppol 4-corner mapping section**

After `### Peppol Interoperability` and its existing bullets, insert:

```markdown
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

```

**Step 4: Verify**

- Confirm REST API table has 6 rows
- Confirm Peppol 4-corner table has 4 rows matching Corner 1–4
- Confirm `Authorization: Bearer <token>` is formatted as inline code
- Confirm `expires_in` is formatted as inline code

**Step 5: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add REST API technical detail and Peppol 4-corner mapping to advanced/inbound"
```

---

### Task 8: advanced/inbound.md — Archiving Standards + Payment Data Reporting Mechanics (Gaps 15, 16)

**Files:**
- Modify: `france-einvoicing/advanced/inbound.md`

**Step 1: Read the current file**

Find `## Archiving` and `### Legal Archiving Requirements`. The archiving standards section goes immediately after the existing legal requirements bullets. The e-reporting section is new and goes between `## Archiving` and `## From Spec to Product` (or at the end of the file before the bridge slide).

**Step 2: Insert archiving standards section**

After the last bullet in `### Legal Archiving Requirements`, insert:

```markdown
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

```

**Step 3: Insert e-reporting / payment data section**

After `## Archiving` (after all archiving H3s), insert before `## From Spec to Product`:

```markdown
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

```

**Step 4: Verify**

- Confirm archiving standards table has 4 rows
- Confirm payment data table has 4 rows
- Confirm `BT-1` is formatted as inline code
- Confirm no H3 exceeds 6 bullets

**Step 5: Commit**

```bash
git add france-einvoicing/advanced/inbound.md
git commit -m "content: add archiving standards and payment data reporting mechanics to advanced/inbound"
```

---

### Task 9: advanced/outbound.md — Credit Notes, Factur-X Profiles, BT Codes, Schematron (Gaps 1, 7, 13, 14)

**Files:**
- Modify: `france-einvoicing/advanced/outbound.md`

**Step 1: Read the current file**

```bash
cat france-einvoicing/advanced/outbound.md
```

Identify these insertion points:
- After `### CIUS-FR: Mandatory Fields for Outbound Invoices` table — insert expanded BT-code table (Gap 13) as a replacement
- After the CIUS-FR section — insert the credit notes section (Gap 1) as a new H2
- After `## Outbound Validation` section — insert Factur-X profiles (Gap 7) as a new H2
- Within `## Outbound Validation`, after `### Common Outbound Validation Failures` — insert schematron detail (Gap 14)

**Step 2: Replace the CIUS-FR mandatory fields table with BT-annotated version**

Find:
```
| Field | Description | Source in Typical ERP |
|---|---|---|
| Seller SIREN/SIRET | 9-digit (SIREN) or 14-digit (SIRET) French business identifier for the seller | Company master data |
| Buyer SIREN/SIRET | Same identifier for the buyer (required for domestic B2B) | Customer master data |
| Invoice type code (BT-3) | Coded type identifying the transaction: commercial invoice, credit note, debit note, etc. | Invoice transaction type mapping |
| VAT breakdown by rate | Tax amount, taxable amount, and applicable rate for each VAT category present on the invoice | Invoice line VAT calculation engine |
| Payment terms | Due date, payment method, and early payment discount if applicable | Payment terms master |
| French legal mentions | Mandatory text elements required by French law, including the late payment penalty clause and any applicable auto-entrepreneur or regime-specific mentions | Configuration / static text template |
```

Replace with:

```markdown
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
```

**Step 3: Insert credit notes outbound section**

After the last bullet in `### CIUS-FR: Mandatory Fields for Outbound Invoices` (after the table), insert as a new section directly after the CIUS-FR H3:

```markdown
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

```

**Step 4: Insert schematron detail section**

After `### Common Outbound Validation Failures` table (after the last `| Malformed XML |` row), insert:

```markdown
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

```

**Step 5: Insert Factur-X profiles section**

After `## Outbound Validation` section (after all its H3s and before `## E-Reporting Data Requirements`), insert:

```markdown
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

```

**Step 6: Verify**

- Confirm CIUS-FR BT table has 8 rows and 4 columns
- Confirm credit notes table has 4 rows (381, 383, 384, 389)
- Confirm schematron table has 5 rows
- Confirm Factur-X table has 5 rows matching the 5 profiles
- Confirm `BR-FR-CTC-NNN` is formatted as inline code in the schematron section
- Confirm **EN 16931** is bolded in the Factur-X section

**Step 7: Commit**

```bash
git add france-einvoicing/advanced/outbound.md
git commit -m "content: add BT codes, credit notes, schematron detail, and Factur-X profiles to advanced/outbound"
```

---

### Task 10: advanced/outbound.md — E-Reporting Submission Mechanics (Gap 16)

**Files:**
- Modify: `france-einvoicing/advanced/outbound.md`

**Step 1: Read the current file**

Find `## E-Reporting Data Requirements`. The existing section has the field tables for B2C, cross-border B2B, and payment data. The submission mechanics section goes after the payment data table, before `## Legal Framework`.

**Step 2: Insert e-reporting submission mechanics section**

After `### Payment Data Reporting: Fields and Timing` section (after the payment timing bullet), insert:

```markdown
### E-Reporting Submission Mechanics

| Reporting Stream | Trigger | Submission Frequency |
|---|---|---|
| Domestic B2B | Automatic during invoice transmission — PA extracts and relays data to DGFiP | Per invoice, real-time |
| B2C transactions | Your system pushes transaction data to your PA | Periodic — per DGFiP implementing decree; typically aligned with VAT return period |
| Cross-border B2B | Your system pushes transaction data to your PA | Same as B2C |
| Payment data | Your ERP/treasury signals payment to your PA | Periodic — same window as B2C |

- Domestic B2B e-reporting requires no separate action from you — DGFiP data is extracted by your PA as part of the transmission process; this is the only reporting stream that is fully automatic
- B2C and cross-border B2B e-reporting require your billing or POS system to push transaction data to your PA — the integration between your system and the PA for this data stream must be configured and tested separately from the invoice transmission integration
- Submission uses the same PA API or EDI channel as invoice submission; your PA documentation defines the specific endpoint and payload schema for e-reporting data — the DGFiP spec v3.1 defines the required data elements but the submission protocol is PA-implemented
- Verify the current reporting window schedule with your PA and legal counsel before go-live — DGFiP implementing decrees govern the exact timing and the schedule may be updated

```

**Step 3: Verify**

- Confirm submission mechanics table has 4 rows (B2B, B2C, cross-border, payment)
- Confirm the domestic B2B row clearly states it is automatic
- Confirm no H3 exceeds 6 bullets

**Step 4: Commit**

```bash
git add france-einvoicing/advanced/outbound.md
git commit -m "content: add e-reporting submission mechanics to advanced/outbound"
```

---

## Verification Checklist (Final)

After all 10 tasks complete, verify:

```bash
# Confirm all 4 target files have changed
git log --oneline france-einvoicing/intermediate/inbound.md
git log --oneline france-einvoicing/intermediate/outbound.md
git log --oneline france-einvoicing/advanced/inbound.md
git log --oneline france-einvoicing/advanced/outbound.md
```

Per-gap coverage audit:

| Gap | Task | Status |
|---|---|---|
| 1 — Credit notes | Tasks 3, 9 | ☐ |
| 2 — Self-billing | Task 4 | ☐ |
| 3 — Multi-entity | Task 4 | ☐ |
| 4 — Testing environments | Task 5 | ☐ |
| 5 — BG-24 attachments | Task 5 | ☐ |
| 6 — Transition period | Tasks 1, 2 | ☐ |
| 7 — Factur-X profiles | Task 9 | ☐ |
| 8 — Annuaire registration | Task 1 | ☐ |
| 9 — Status timing/SLAs | Task 6 | ☐ |
| 10 — Deduplication | Task 6 | ☐ |
| 11 — REST API detail | Task 7 | ☐ |
| 12 — Peppol 4-corner | Task 7 | ☐ |
| 13 — BT codes for CIUS-FR | Task 9 | ☐ |
| 14 — Schematron specifics | Task 9 | ☐ |
| 15 — NF Z42-013 archiving | Task 8 | ☐ |
| 16 — E-reporting mechanics | Tasks 8, 10 | ☐ |
| 17 — PA decertification | Tasks 1, 2 | ☐ |
| 18 — Amendment flow | Task 3 | ☐ |
