# France E-Invoicing — Node Reference

How the Tungsten stack fulfills the French e-invoicing mandate, described as a set of functional nodes and the flows between them.

---

## The Nodes

### Node 1 — SAP SD / FI (Sending) and MM / FI (Receiving)

**What it is.** The ERP billing and procurement engine. SD (Sales & Distribution) and FI (Financial Accounting) originate outbound invoices. MM (Materials Management) and FI receive and post inbound invoices.

**Role in France e-invoicing.**

On the **sending (AR) side**, SAP SD and/or FI are the origin of every outbound invoice. SD handles sales billing runs that generate billing documents; FI handles direct financial postings for non-SD scenarios. The moment a billing document is saved, a Business Transaction Event (BTE) fires. This BTE is the trigger that starts the entire outbound e-invoicing chain. SAP itself does not know about France, CIUS-FR, or the Y-model — it simply creates a billing event and makes the document data available.

On the **receiving (AP) side**, SAP MM and/or FI are the final destination. Once an inbound invoice has been validated, matched, and approved through the upstream nodes, it is posted into SAP as an MM document (for PO-related invoices via MIRO) or an FI document (for non-PO invoices). SAP then manages the payment run, which is the trigger for the payment data e-reporting obligation (Flow 10.2).

**Master data obligations — SIREN/SIRET storage in SAP.**

SAP does provide places to maintain SIREN and SIRET, but it is not a single, obvious field — the picture differs between ECC and S/4HANA, and between your own entities and your business partners:

- **Your own entities (company code level):** SIREN can be maintained in OBY6 (company code additional data). SIRET — which includes the SIREN plus a 5-digit NIC identifying the establishment — can be maintained at plant level. SAP HR also carries a dedicated SIRET data element (P06_SIRET) for French payroll, but this is not the relevant source for e-invoicing.

- **Business partners (customers and vendors):** S/4HANA has dedicated functionality for maintaining SIREN and SIRET on Business Partners — SAP Help documents titled "Maintaining SIREN or SIRET Numbers for Business Partners" exist for both S/4HANA on-premise and S/4HANA Cloud. In older ECC systems, there was no dedicated SIREN/SIRET field on the customer or vendor master, and implementations typically stored SIRET in the "Tax Number 1" field as a workaround. The French VAT number already embeds the SIREN (format: FR + 2 check digits + 9-digit SIREN), so SIREN is derivable from the VAT registration, but SIRET (with the establishment-level NIC suffix) requires its own field.

- **Open question for OIC enrichment:** During the OIC enrichment step, the cockpit pulls master data from SAP to populate the CIUS-FR mandatory fields (BT-29 seller SIREN, BT-30 seller SIRET, BT-46 buyer SIREN, BT-49 buyer SIRET). The exact SAP fields that OIC reads from — and whether this requires S/4HANA BP fields or can work with the ECC Tax Number workaround — needs confirmation from the product team. This is registered as an open item.

**Other master data.** Payment terms, VAT codes, and French legal mentions must also be maintained in SAP master data — these are pulled during enrichment by the OIC node.

**What SAP does NOT do.** SAP does not generate structured e-invoice formats (Factur-X, UBL, CII). It does not communicate with the PA, the PPF, or DGFiP. It does not manage lifecycle statuses (Déposée, Reçue, Approuvée). It does not handle Annuaire lookups or delivery channel routing. All of that is handled by the downstream nodes.

---

### Node 2 — Process Director OIC (Outbound Invoice Cockpit)

**What it is.** An SAP-certified add-on built on Process Director that provides a centralized cockpit inside SAP for managing outbound billing documents. OIC is the bridge between SAP and the InvoiceAgility cloud platform.

**Role in France e-invoicing.**

OIC is the **sending-side orchestrator**. It performs three functions:

1. **Capture.** OIC listens for BTEs fired by SAP billing events. When a billing document is created in SD or FI, the OIC BTE implementation catches the event in real time, identifies the transaction type, and creates a cockpit document for process management. No custom ABAP is required — OIC hooks into standard SAP BTEs.

2. **Enrichment.** OIC pulls master data from SAP and attaches it to the cockpit document. For France, this means: seller SIREN/SIRET (from company master), buyer SIREN/SIRET (from customer master / business partner), VAT registration details, payment terms, destination email addresses, and electronic delivery identifiers (IBAN, GLN, VAT/Tax ID). This enrichment step is what populates the CIUS-FR mandatory fields that the PA will later validate.

3. **Routing.** OIC's routing engine determines the correct delivery channel based on a hierarchical configuration at the country, company code, and buyer level. For France, the routing logic determines whether the document goes via TeN, via TeC/Peppol, or via PDF email as a fallback. The routing decision is automatic — operators do not manually select channels. **[OPEN: The France-specific routing configuration needs to be detailed. It is possible that for France all outbound documents are routed to the TeC API as the single submission point, with TeC/IvA handling the downstream channel selection (TeN for domestic clearance, Peppol for cross-border). This routing design decision needs confirmation and will be expanded in a subsequent step.]**

**Status synchronization.** Delivery statuses from InvoiceAgility flow back into the OIC cockpit automatically. Operators see per-document status tracking: Ready → Transmitted → Delivered → Error. For France, this includes both buyer delivery status and tax authority reporting status. Error logs are surfaced directly in the cockpit, and email notifications can be sent to designated groups. **[OPEN: Status synchronization for France needs expansion — the specific PAe lifecycle statuses (Déposée, Reçue, Approuvée, Rejetée, Refusée) and how they map to OIC cockpit statuses, the Invoice Status Service (ISS) mechanics, and the latency/polling behaviour need to be documented. See open items file.]**

**What OIC does NOT do.** OIC does not generate the compliant e-invoice format (Factur-X, UBL, CII) — it transmits a standard XML format (ESXML) to InvoiceAgility, which handles the format conversion. OIC does not validate against CIUS-FR schematron rules — that validation happens at the PA level. OIC does not communicate directly with the PPF or DGFiP.

---

### Node 3 — Process Director Inbound Monitor

**What it is.** An SAP add-on that receives incoming invoices as PDF files or XML attachments, extracts structured data, and routes them to Process Director Accounts Payable for processing.

**Role in France e-invoicing.**

Inbound Monitor is the **receiving-side entry point** — the first Tungsten component that touches an inbound e-invoice after it has been delivered by the PA.

**Data reception.** Inbound Monitor can receive data in four ways: from emails sent to an SAP mailbox, from bulk file uploads via a file share, from manual user uploads, or via the Process Director generic API (`/EBY/PDBO_DATA_RECEIVE`). For France e-invoicing, the primary inbound channel will be the API or file share integration with the PA/IvA platform, which delivers structured XML invoices (UBL 2.1, CII) or hybrid Factur-X PDFs after they have transited the Y-model.

**XML extraction and parsing.** When a Factur-X PDF arrives, Inbound Monitor's ZUGFeRD Extraction Service (an external Windows service registered as an RFC destination in SAP) analyses the PDF to determine whether XML invoice data is embedded. If XML is found, it extracts it and passes it back to Inbound Monitor for archiving. If no XML is found (a non-structured PDF), the document is routed to a file share or mailbox for OCR processing — though under the French mandate, all invoices arriving via the PA network will be structured, so OCR fallback should be rare.

For pure XML invoices (UBL 2.1 or CII arriving without a PDF wrapper), Inbound Monitor parses the XML directly using XSLT transformations. The XSLT mapping files define how invoice data fields (seller SIREN, buyer SIRET, line items, VAT breakdown, payment terms) map to Process Director's internal field structure. Different XSLT stylesheets handle different formats — ZUGFeRD/Factur-X, XRechnung, FatturaPA, etc. For France, the relevant stylesheets must map the CIUS-FR data elements correctly.

**Handoff to AP.** Once XML is extracted and mapped, Inbound Monitor automatically transfers the structured data to Process Director Accounts Payable (or a different Process Director document type, configurable via the Object Type field). The handoff creates a new AP document pre-populated with all extracted invoice data.

**Exception handling.** Inbound Monitor provides operator functions for cases where automatic processing fails: manual XML extraction, manual attachment splitting (when one email contains multiple invoices), manual handoff to AP, and document rejection for invalid or duplicate submissions. Each document tracks its processing status: Unprocessed → XML Extracted → Sent to AP → Posted → Paid.

**What Inbound Monitor does NOT do.** It does not validate invoices against CIUS-FR rules — that was already done by the PA before delivery. It does not perform PO matching or AP approval workflows — that happens downstream in Process Director AP. It does not send lifecycle status messages (Reçue, Approuvée, Refusée) back to the sender's PA — that responsibility sits with the PA/IvA layer.

---

### Node 4 — Process Director Accounts Payable

**What it is.** The SAP-based AP automation cockpit that handles invoice validation, matching, approval workflows, and posting to SAP.

**Role in France e-invoicing.**

Process Director AP is the **receiving-side processing engine**. Once Inbound Monitor has extracted and transferred the structured invoice data, AP takes over for business processing.

**Validation and matching.** AP runs standard SAP checks plus configurable Process Director checks on every document. For PO-related invoices (MM), AP performs line item matching against purchase orders — comparing quantities, prices, and delivery references. For non-PO invoices (FI), AP validates account assignments and G/L coding. The system identifies errors (price mismatches, quantity discrepancies, missing goods receipts) and flags them for operator resolution.

For France specifically, AP must handle the full range of inbound use cases: multi-order/multi-delivery invoices (UC 1), down-payment sequences (UC 20/21), partial collection tracking (UC 34), mixed invoices with different VAT treatments (UC 31), self-billed invoices with reversed SIREN/SIRET positions (UC 19b), and credit notes with BT-25 preceding invoice references.

**Approval workflows.** AP supports configurable approval workflows via Work Cycle. Documents can be routed for review and approval based on amount thresholds, cost centers, document types, or custom business rules. Workflow statuses (Sent, In Workflow, Released, Rejected) track the approval progress. For France, the approval/rejection decision is significant because it determines whether the lifecycle status sent back via the PA is Approuvée (accepted) or Refusée (business refusal).

**Posting.** Once validated and approved, AP posts the document to SAP — either as an FI document (financial posting) or an MM document (logistics invoice via MIRO). The SAP document number is recorded, and the AP document status changes to Posted. Subsequent payment by SAP's payment run changes the status to Paid.

**Payment event as e-reporting trigger.** When a posted invoice is paid in SAP, this payment event is the trigger for the Flow 10.2 e-reporting obligation. The payment data (invoice reference, payment date, amount, payment method) must be reported to DGFiP via the PA. AP's status tracking (Posted → Paid) provides the signal that the payment has occurred; the actual e-reporting submission is handled by the PA/IvA layer.

**What AP does NOT do.** AP does not receive invoices from the PA network — Inbound Monitor handles reception. AP does not generate lifecycle status messages — that is the PA's responsibility. AP does not archive invoices in NF Z42-013 compliant format — archiving is handled by the PA's certified archiving service or a separate archive integration.

---

### Node 5 — InvoiceAgility (TeN + TeC) as PAe / PAr

**What it is.** Tungsten's cloud platform that combines two engines: Tungsten e-Network (TeN) for tax authority network delivery and domestic e-invoicing, and Tungsten e-Connect (TeC) for Peppol connectivity and cross-border delivery. Together they form InvoiceAgility (IvA), which is the platform behind Tungsten's certified PA (PA #0072, registered with DGFiP). The PA runs on white-labelled Sovos infrastructure in a SecNumCloud-certified environment.

**Role in France e-invoicing.**

InvoiceAgility is the **PA node** — the legally required intermediary between sender and receiver in the Y-model. It operates in two modes:

**As PAe (Plateforme d'émission — sending PA):**

1. **Receives ESXML from OIC.** OIC transmits the enriched invoice in Tungsten's standard XML format via a configured RFC connection.

2. **Format conversion.** IvA converts the ESXML into the compliant French format — Factur-X (EN 16931 profile or Extended), UBL 2.1, or CII — based on the recipient's capability and configuration. The conversion applies all CIUS-FR mandatory fields, French legal mentions, and VAT breakdown structure.

3. **Annuaire lookup.** IvA queries the Annuaire (maintained by the PPF) using the buyer's SIREN to identify which PA serves the recipient.

4. **PA-to-PA transmission.** IvA delivers the invoice directly to the recipient's PA. Currently, PA-to-PA exchange uses Peppol BIS 3 UBL over Peppol transport; non-Peppol PA-to-PA channels are expected Q3-2026.

5. **E-reporting (Flow 1).** As a side-effect of every domestic B2B invoice transmission, IvA extracts the required e-reporting data and relays it to DGFiP via the PPF. This is automatic — zero effort from the sender.

6. **Lifecycle status management (PAe side).** IvA generates and transmits PAe lifecycle statuses: Déposée (submitted), delivered (CDAR 200). When the recipient's PA responds with acceptance or rejection, those statuses flow back through IvA into the TeN portal and the OIC cockpit via the Invoice Status Service.

**As PAr (Plateforme de réception — receiving PA):**

1. **Receives invoices from sender PAs.** IvA accepts invoices from any certified PA on the network, validates format and CIUS-FR compliance.

2. **Format normalization.** Regardless of the source format (Factur-X, UBL, CII), IvA normalizes the invoice to the target format expected by the customer's AP system.

3. **Delivery to AP system.** IvA makes the normalized invoice available for retrieval by Inbound Monitor (via API, file share, or email delivery).

4. **Lifecycle status management (PAr side).** IvA generates mandatory PAr statuses: Reçue (received, CDAR 200 fires automatically on receipt), Approuvée (when the AP system confirms acceptance), Refusée (when the AP system signals a business refusal, CDAR 210), or Rejetée (if technical validation fails, CDAR 213). These statuses flow back to the sender's PA.

5. **Payment data reporting (Flow 10.2).** When the customer's ERP signals payment (the AP status changes to Paid), IvA submits the payment event data to DGFiP: invoice reference, payment date, amount paid, payment method.

**Additional e-reporting.** Beyond per-invoice domestic B2B reporting, IvA handles aggregated scheduled reporting via Sovos Scheduler for three additional scopes: international invoices sent and received (Flow 10.1), B2C transaction data (Flow 10.3), and payment data for all flows (Flow 10.2). These require the customer's billing/POS system to push transaction data to IvA.

**Channel selection.** IvA automatically selects the right delivery channel per transaction: Peppol via TeC for cross-border or Peppol-registered recipients, domestic tax network via TeN for mandate countries, or PDF email fallback for non-mandate scenarios.

**What IvA does NOT do.** IvA does not perform AP-side business matching (PO matching, goods receipt validation) — that is Process Director AP's job. IvA does not maintain the Annuaire — the PPF does. IvA does not process payment runs — SAP does.

---

### Node 6 — French PPF (Portail Public de Facturation)

**What it is.** The government portal operated by DGFiP. Despite its name, the PPF is NOT in the invoice delivery path.

**Role in France e-invoicing.**

The PPF performs two functions:

1. **Annuaire maintenance.** The PPF maintains the national directory (Annuaire) of all French businesses and their designated PAs. When a business onboards with a PA, that PA registers the business's SIREN/SIRET in the Annuaire, pointing to itself as the routing destination. Every PA queries the Annuaire before transmitting an invoice to discover the correct recipient PA.

2. **E-reporting data relay.** The PPF receives structural relay copies of e-reporting data from PAs and forwards them to DGFiP for VAT monitoring and compliance enforcement. This includes domestic B2B invoice data (relayed automatically by PAs during transmission), lifecycle status data (relayed by PAs as statuses change), and aggregated reporting data (B2C, cross-border, payment data).

**What the PPF does NOT do.** The PPF does not route, validate, or deliver invoices. It does not store invoices. It does not interact with your ERP or AP system. It is not a PA. In the Y-model diagram, the PPF sits off to the side — the two PA legs converge at the Annuaire lookup point, but the invoice travels PA-to-PA directly, never through the PPF.

---

## The Flows

### Flow A — Sending a Domestic B2B Invoice (AR / Outbound)

This is the primary outbound flow for France: your SAP system invoices a French buyer.

```
Step 1         Step 2          Step 3              Step 4           Step 5
SAP SD/FI  →  PD OIC →    InvoiceAgility    →  Recipient's  →  Recipient's
               (capture,    (PAe)                  PA (PAr)        AP system
                enrich,     - Convert to            │
                route)        Factur-X/UBL/CII      │
                            - Annuaire lookup       │
                            - PA-to-PA delivery     │
                            - e-Report (Flow 1)     │
                                │                   │
                                └───── PPF ─────────┘
                                       │          (relay e-reporting
                                       ↓           data only)
                                     DGFiP
```

**Step 1 — SAP SD/FI.** Billing document created in SD (sales billing) or FI (financial posting). BTE fires.

**Step 2 — OIC.** Captures BTE event. Enriches with master data (SIREN, SIRET, VAT, payment terms). Routing engine identifies France/domestic/clearance scenario. Transmits ESXML to IvA on scheduled job via RFC destination (SM59).

**Step 3 — IvA (PAe).** Converts ESXML to compliant format (Factur-X EN 16931 profile, UBL 2.1, or CII per recipient capability). Validates against EN 16931 + CIUS-FR schematron rules (~50+ business rules). Queries Annuaire for buyer's SIREN → discovers recipient PA. Delivers invoice PA-to-PA. Extracts e-reporting data and relays to PPF (Flow 1, automatic). Generates PAe lifecycle statuses: Déposée on submission, delivery confirmation on CDAR 200.

**Step 4 — Recipient PA.** Receives, validates, makes available to recipient AP system. Sends PAr statuses (Reçue, Approuvée/Refusée) back through the network.

**Step 5 — Status return.** Delivery statuses (submitted → delivered → accepted/refused) flow back through IvA into the OIC cockpit. Operators see the full lifecycle in SAP.

---

### Flow B — Receiving a Domestic B2B Invoice (AP / Inbound)

This is the primary inbound flow: a French supplier invoices your SAP system.

```
Step 1          Step 2              Step 3           Step 4          Step 5
Sender's   →  InvoiceAgility  →  PD Inbound   →   PD Accounts  →  SAP MM/FI
PA (PAe)       (PAr)               Monitor          Payable         (post &
               - Receive           - Extract XML    - PO match       pay)
               - Validate          - XSLT parse     - Validate       │
               - Normalize         - Handoff to AP  - Approve        │
               - PAr statuses                       - Post           │
                   │                                                 │
                   │←──────────── Payment signal (Paid status) ──────┘
                   │
                   └── e-Report payment data (Flow 10.2) → PPF → DGFiP
```

**Step 1 — Sender PA.** Supplier's PA transmits invoice PA-to-PA to IvA (your PAr).

**Step 2 — IvA (PAr).** Receives the invoice, validates format and CIUS-FR compliance. Normalizes to target format. Fires PAr status Reçue (CDAR 200) automatically. Makes invoice available for retrieval by Inbound Monitor.

**Step 3 — Inbound Monitor.** Receives structured invoice (Factur-X PDF with embedded XML, or pure UBL/CII XML). ZUGFeRD Extraction Service extracts XML from Factur-X PDFs if applicable. XSLT transformations parse extracted XML into Process Director's internal field structure, mapping CIUS-FR fields (SIREN, SIRET, BT-3 type code, VAT breakdown, payment terms). Archives source documents. Automatically hands over extracted data to Process Director AP. Status: Unprocessed → XML Extracted → Sent to AP.

**Step 4 — Process Director AP.** Receives pre-populated invoice document from Inbound Monitor. For MM documents: performs PO line item matching against purchase order, validates quantities and prices, checks for goods receipts. For FI documents: validates account assignments and G/L coding. Runs SAP standard checks plus Process Director checks. Routes through approval workflow (Work Cycle) if configured. On approval: posts to SAP as MM or FI document. Status: Errors → Unposted → In Workflow → Released → Posted.

**Step 5 — SAP MM/FI.** Invoice is posted. Payment run executes. When paid, the Paid status propagates back through AP and triggers Flow 10.2 payment data reporting: IvA submits payment event (invoice reference, date, amount, method) to DGFiP via PPF.

**Lifecycle status feedback.** The AP approval/rejection decision drives the PAr lifecycle status sent back to the sender:
- AP approves and posts → IvA sends Approuvée to sender's PA
- AP operator rejects on business grounds → IvA sends Refusée (CDAR 210) to sender's PA
- Technical validation failure → IvA sends Rejetée (CDAR 213) to sender's PA

---

### Flow C — Credit Notes and Corrective Invoices

Credit notes (BT-3 = 381), debit notes (383), and corrective invoices (384) follow the same node-to-node path as commercial invoices. The key differences are operational, not architectural:

**Outbound (AR).** SAP creates the credit/debit note. OIC captures and enriches it. IvA converts to the correct format with BT-25 (Preceding Invoice Reference) populated — referencing the original invoice's BT-1 number. The PA validates that BT-25 is present (mandatory for 381/383/384 — omission fails CIUS-FR validation). Delivery and e-reporting follow standard paths.

**Inbound (AP).** Inbound Monitor receives and parses the credit note. AP must validate that the referenced original invoice (BT-25) exists in SAP records before posting. A corrective invoice (384) supersedes the original — AP must mark the original as replaced. VAT on credit notes reverses the original VAT — AP must process the reversal correctly, not as a new negative charge.

---

### Flow D — Self-Billing (UC 19b)

Self-billing reverses the normal sender/receiver relationship. The buyer generates the invoice on behalf of the supplier.

**Your company as buyer-sender.** SAP creates the self-billed invoice (BT-3 = 389). OIC captures it. IvA transmits it to the supplier's PAr with reversed SIREN/SIRET positions: buyer's SIREN/SIRET in seller fields (BT-29/BT-30), supplier's SIREN/SIRET in buyer fields (BT-46/BT-49). Payment data reporting remains your obligation.

**Your company as supplier-receiver.** A self-billed invoice arrives at IvA (your PAr) with BT-3 = 389. Inbound Monitor parses it. AP must recognize the reversed SIREN/SIRET positions and process accordingly. You can accept or reject via standard lifecycle statuses.

---

### Flow E — E-Reporting (Beyond Domestic B2B)

Domestic B2B e-reporting (Flow 1) is automatic — IvA extracts data during invoice transmission. The following streams require explicit data feeds from your systems to IvA:

| Stream | Flow Code | Data Source | Trigger |
|---|---|---|---|
| B2C transactions | 10.3 | POS / billing system → IvA | Periodic (aligned with VAT return period) |
| Cross-border B2B | 10.1 | AR system → IvA | Periodic |
| Payment data (all flows) | 10.2 | SAP payment run → AP → IvA | Per payment event |

IvA handles submission to DGFiP via PPF for all streams. The integration between your POS/billing systems and IvA must be configured and tested separately from the invoice transmission integration.

---

## Node Interaction Matrix

| From → To | SAP SD/FI/MM | PD OIC | PD Inbound Monitor | PD Accounts Payable | IvA (TeN+TeC) | PPF |
|---|---|---|---|---|---|---|
| **SAP SD/FI** | — | BTE fires on billing event | — | — | — | — |
| **PD OIC** | Status display (link to original doc) | — | — | — | ESXML transmission via RFC (SM59) | — |
| **PD Inbound Monitor** | — | — | — | Handoff of parsed XML data | — | — |
| **PD Accounts Payable** | Posts MM/FI document; reads PO/GR data | — | Status sync (Posted, Paid) | — | — | — |
| **IvA (TeN+TeC)** | — | Delivery + e-reporting statuses returned to cockpit | Delivers normalized invoice (API/file/email) | — | PA-to-PA invoice exchange | Annuaire queries; e-reporting data relay |
| **PPF** | — | — | — | — | Annuaire responses; e-reporting relay confirmations | — |

---

## Configuration Checklist per Node

### SAP SD / FI / MM
- [ ] SIREN populated in company master data for all French legal entities (OBY6 additional data)
- [ ] SIRET populated for each French establishment (plant level or BP field in S/4HANA)
- [ ] SIREN/SIRET populated in customer master (for AR) and vendor master (for AP) — confirm field location: S/4HANA BP fields vs. ECC Tax Number workaround
- [ ] VAT codes and payment terms configured for French transactions
- [ ] French legal mentions available as configuration/static text templates

### PD OIC
- [ ] RFC destination configured (SM59) for SAP → InvoiceAgility connectivity
- [ ] BTE implementation activated for relevant billing document types (SD, FI)
- [ ] Country-level configuration for France: clearance type, mapping rules
- [ ] Company code-level scope activation with validity dates
- [ ] Buyer-level overrides for specific routing agreements
- [ ] Routing configuration for France confirmed (direct to TeC API vs. TeN split) — see open items
- [ ] Scheduled jobs configured for document transmission
- [ ] Error notification email groups configured

### PD Inbound Monitor
- [ ] ZUGFeRD Extraction Service installed and RFC destination configured (SM59)
- [ ] XSLT mapping files uploaded for Factur-X, UBL 2.1, CII formats
- [ ] Field mapping configured from XSLT-extracted data to PD AP fields (CIUS-FR fields mapped)
- [ ] Archiving configured (document types for PDF, XML, XMLS in OAC2/OAC3)
- [ ] Data reception channel configured (API endpoint, mailbox, or file share for PA delivery)
- [ ] Worklist configured with status categories (Unprocessed, XML Extracted, Sent to AP, Posted, Paid)

### PD Accounts Payable
- [ ] PO matching configuration for MM documents
- [ ] Account assignment rules for FI documents
- [ ] Tax code determination configured for French VAT rates
- [ ] Approval workflows configured in Work Cycle
- [ ] Rejection/refusal workflow configured (triggers PAr Refusée status via IvA)
- [ ] BTE implementation for status synchronization back to Inbound Monitor

### InvoiceAgility (PA #0072)
- [ ] Entity hierarchy set up in OBi with correct SIREN/SIRET structure
- [ ] e-Address registered in Annuaire (EDEN, SIREN, SIRET, UPEN, UPEM, UPF)
- [ ] KYC (Know Your Business) completed for PA registration
- [ ] LIVE/TEST toggle verified in TeN portal for UAT vs. production
- [ ] Format conversion validated (ESXML → Factur-X/UBL/CII)
- [ ] All 44 French use cases tested that are in scope for the customer
- [ ] Payment data reporting integration configured (ERP payment event → IvA → DGFiP)
- [ ] B2C and cross-border e-reporting feeds configured (if applicable)

### PPF (No customer configuration — government-operated)
- [ ] Verify Annuaire registration is complete and visible to other PAs
- [ ] Confirm routing tests succeed (sender PA can discover your PA via Annuaire lookup)

---

## Timeline Reference

| Milestone | Date | Impact on Nodes |
|---|---|---|
| Grand Pilot | Q1 2026 | PA infrastructure validated (IvA / Sovos) |
| UAT / Pilot | April–May 2026 | All nodes tested end-to-end against Sovos sandbox; pilot = LIVE production |
| **Go-live: receive** | **Sept 1, 2026** | Inbound Monitor, AP, IvA (PAr) must be operational for ALL businesses |
| **Go-live: send (GE+ETI)** | **Sept 1, 2026** | OIC, IvA (PAe) must be operational for large enterprises |
| Go-live: send (SME+micro) | Sept 1, 2027 | OIC, IvA (PAe) must be operational for all remaining businesses |
