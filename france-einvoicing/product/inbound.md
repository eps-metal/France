# Receiving E-Invoices — Tungsten Products

## Where Tungsten Sits in the Inbound Flow

### Tungsten AP and the Y-Model

Tungsten AP products participate in the inbound flow at the point where structured invoices delivered by the PA must be processed, validated, and routed into an ERP. In the Y-model, Tungsten operates downstream of the PA — receiving invoices that have already transited the interoperability layer — and handles the business logic steps (matching, enrichment, exception handling) before ERP handoff.

> **[STUB: Diagram and narrative showing the precise position of Tungsten AP products in the Y-model inbound flow — which component connects to the PA, via which protocol, and where the handoff to ERP occurs]** — content to be added in a subsequent iteration.

### PA Role: Does Tungsten Act as a Certified PA?

> **[STUB: Clarify whether Tungsten operates as a certified PA (Plateforme Agréée), connects as a downstream system to a customer-selected PA, or both — and what this means for how customers configure their France e-invoicing setup]** — content to be added in a subsequent iteration.

## Receiving the Invoice

### How an Inbound E-Invoice Enters Tungsten

> **[STUB: Describe the technical integration point between the PA and Tungsten AP — whether Tungsten polls the PA, receives push notifications, or uses an event-driven API; include the relevant connection protocols (REST, AS2/AS4) and authentication approach]** — content to be added in a subsequent iteration.

### Supported Inbound Formats

> **[STUB: List the inbound structured formats Tungsten AP processes natively (Factur-X, UBL 2.1, CII) and identify whether format normalization or conversion occurs within Tungsten or is expected to be handled by the PA before delivery]** — content to be added in a subsequent iteration.

## Processing the Invoice

### Validation and Business Matching

> **[STUB: Describe the validation and matching steps Tungsten performs beyond what the PA has already validated — e.g., PO matching, goods receipt matching, supplier master validation, duplicate detection, CIUS-FR data enrichment; and which of these are configurable vs. fixed]** — content to be added in a subsequent iteration.

### Routing and ERP Handoff

> **[STUB: Describe how Tungsten routes validated invoices to the correct ERP entity or cost centre; list the ERP integrations supported for France e-invoicing; describe the handoff mechanism (API call, file drop, message queue)]** — content to be added in a subsequent iteration.

## Exception Handling

### Operator Workflow for Invoice Exceptions

> **[STUB: Describe what an AP operator sees when an invoice fails matching or validation — the exception queue, available actions (approve, reject, query sender, escalate), and how Tungsten surfaces the relevant data to support the decision]** — content to be added in a subsequent iteration.

### Handling Rejections and Refusals Back to Sender

> **[STUB: Describe how Tungsten initiates a technical rejection (Rejetée) or business refusal (Refusée) via the PA status mechanism — what the operator action is in Tungsten, how that triggers the status update at the PA level, and what the sender receives]** — content to be added in a subsequent iteration.

## E-Reporting Support

### Payment Data Reporting: Tungsten's Role

> **[STUB: Describe how Tungsten supports the payment data reporting obligation for received B2B invoices — whether Tungsten detects or receives the payment event from the ERP, how it triggers the e-reporting submission to the PA/DGFiP, and whether this is automated or requires operator action]** — content to be added in a subsequent iteration.

## Archiving

### Invoice Archiving for Compliance

> **[STUB: Describe Tungsten's approach to the 10-year archiving obligation — whether Tungsten provides certified archiving natively, delegates to the PA's archiving service, or integrates with a third-party archive; and what formats are preserved (structured XML, PDF/human-readable)]** — content to be added in a subsequent iteration.
