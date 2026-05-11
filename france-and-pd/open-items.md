---
title: France E-Invoicing — Open Items
created: 2026-03-05
modified: 2026-03-05
topics:
  - france-compliance
  - e-invoicing
  - open-issues
  - architecture-refinement
doc_type: scratch
status: draft
summary: Tracked open items and subsequent investigation tasks related to French e-invoicing node architecture, including Process Director OIC routing, mandate requirement mapping, and pending technical clarifications.
keywords:
  - France
  - e-invoicing
  - open items
  - Process Director
  - OIC
  - routing
  - configuration
  - mandate
  - architecture refinement
aliases:
  - France Open Items
  - France Followups
---

# France E-Invoicing — Open Items

Tracked items for subsequent investigation and expansion related to `france-einvoicing-nodes.md`.

---

## Open Item 1 — OIC Routing Configuration for France

**Status:** Open
**Node:** Process Director OIC (Node 2)
**Priority:** High — architectural decision

**Context.** The node reference currently describes OIC's routing engine as selecting between TeN, TeC/Peppol, and PDF email based on country/company code/buyer configuration. For France specifically, it may be that all outbound documents are routed to the TeC API as the single submission point, with TeC/IvA then handling the downstream channel selection (TeN for domestic clearance, Peppol for cross-border, etc.).

**To resolve:**
- Confirm whether OIC sends everything to TeC for France, or whether OIC itself distinguishes between TeC and TeN routes
- If TeC is the single entry point, document how TeC determines the onward path (domestic → TeN → PA network vs. cross-border → Peppol)
- Map this to the OIC integration scenarios (Scenarios 1–5 in the OIC documentation) and identify which scenario(s) apply to France
- Update the node reference and flow diagrams accordingly

---

## Open Item 2 — OIC Status Synchronization for France Lifecycle Statuses

**Status:** Open
**Node:** Process Director OIC (Node 2) ↔ InvoiceAgility (Node 5)
**Priority:** High — operator experience

**Context.** The node reference states that delivery statuses flow back from IvA into the OIC cockpit, but the France-specific lifecycle statuses (Déposée, Reçue, Approuvée, Rejetée, Refusée, Encaissée) are richer than the generic OIC status model (Ready, Transmitted, Delivered, Error).

**To expand:**
- How do the French PAe/PAr lifecycle statuses (as defined in DGFiP External Specifications v3.1) map to the OIC cockpit status fields?
- What is the Invoice Status Service (ISS) and how does it relay statuses from IvA back to SAP?
- What is the latency model — does OIC poll for statuses, or does IvA push them? What is the typical delay between a PA status change and its visibility in the OIC cockpit?
- How are CDAR message codes (200, 210, 212, 213) surfaced to the operator — as raw codes, mapped descriptions, or both?
- What does the operator see and what can they do when a Refusée (business refusal) or Rejetée (technical rejection) status arrives?
- How is the Encaissée (payment reported) status triggered — does it flow from SAP payment run → AP → IvA → PPF, and is it visible in OIC?

---

## Open Item 3 — SIREN/SIRET Field Mapping in OIC Enrichment

**Status:** Open
**Node:** SAP SD/FI/MM (Node 1) → Process Director OIC (Node 2)
**Priority:** Medium — implementation detail

**Context.** OIC enriches outbound invoices with master data from SAP. For France, this includes populating BT-29 (seller SIREN), BT-30 (seller SIRET), BT-46 (buyer SIREN), and BT-49 (buyer SIRET). SAP S/4HANA has dedicated BP fields for SIREN/SIRET; older ECC systems typically use "Tax Number 1" as a workaround.

**To resolve:**
- Which SAP fields does OIC read from during enrichment to populate the CIUS-FR SIREN/SIRET fields?
- Does this differ between S/4HANA (with native BP SIREN/SIRET fields) and ECC (with Tax Number workaround)?
- Is any configuration required in OIC to map the correct SAP fields to the ESXML output for France?
- Can SIREN be derived from the VAT number (FR + 2 check digits + 9-digit SIREN), or must it be explicitly maintained as a separate field?

---

## Open Item 4 — Inbound: IvA-to-Inbound-Monitor Integration Mechanism

**Status:** Open (from product-level stub documents)
**Node:** InvoiceAgility (Node 5) → PD Inbound Monitor (Node 3)
**Priority:** High — integration design

**Context.** The product-level inbound documentation (`france-einvoicing/product/inbound.md`) is almost entirely stubs. The connection between IvA as PAr and Inbound Monitor — the mechanism by which a received invoice is made available to the SAP-side processing chain — is not documented.

**To resolve:**
- Does IvA push invoices to Inbound Monitor (via API call, RFC, email), or does Inbound Monitor poll IvA?
- Which Inbound Monitor data reception method is used: mailbox, file share, API (`/EBY/PDBO_DATA_RECEIVE`), or a combination?
- In what format does the invoice arrive at Inbound Monitor — original format from the sender PA, or already normalized by IvA?
- How are attachments (BG-24 additional supporting documents) handled in the handoff?

---

## Open Item 5 — AP Approval → PAr Lifecycle Status Feedback Loop

**Status:** Open (from product-level stub documents)
**Node:** PD Accounts Payable (Node 4) → InvoiceAgility (Node 5)
**Priority:** High — compliance requirement

**Context.** When AP approves or rejects an invoice, this decision must trigger the corresponding PAr lifecycle status (Approuvée or Refusée) back to the sender via IvA. The mechanism for this feedback loop is not documented.

**To resolve:**
- How does AP signal approval/rejection to IvA? Is it an API call, a status update in Inbound Monitor that propagates, or a direct integration?
- Is this automated or does it require operator action?
- What is the timing — does the status fire immediately on posting, or on a scheduled batch?
- How does the Rejetée (technical rejection) path work — is this triggered at the IvA/PA level before the invoice even reaches Inbound Monitor?

---

*Last updated: 2026-03-25*
