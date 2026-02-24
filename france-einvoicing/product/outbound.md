# Sending E-Invoices — Tungsten Products

## Where Tungsten Sits in the Outbound Flow

### Tungsten AR and the Y-Model

Tungsten AR products sit between the ERP — which generates invoice source data — and the PA (Plateforme Agréée), which transmits the structured invoice directly to the recipient's PA via the Y-model. Tungsten is responsible for transforming ERP data into a compliant structured format (Factur-X, UBL 2.1, or CII) and handing it off to the PA for validation, routing, and onward delivery.

> **[STUB: Diagram and narrative showing the precise position of Tungsten AR products in the Y-model outbound flow — which component generates the structured invoice, which connects to the PA, via which protocol, and how the outbound journey is triggered]** — content to be added in a subsequent iteration.

### PA Role for Outbound

> **[STUB: Clarify how Tungsten AR connects to the outbound PA — whether the same PA handles both inbound and outbound, whether PA selection is customer-configurable, and what Tungsten manages vs. what the PA manages in the outbound flow]** — content to be added in a subsequent iteration.

## Generating the E-Invoice

### Supported Outbound Formats

> **[STUB: List which outbound structured formats Tungsten AR can generate (Factur-X, UBL 2.1, CII); describe how format selection is determined (customer configuration, recipient PA capability, or negotiated with recipient); and whether format conversion is performed within Tungsten or delegated to the PA]** — content to be added in a subsequent iteration.

### CIUS-FR Field Mapping

> **[STUB: Describe how Tungsten AR maps ERP source fields to the CIUS-FR mandatory fields (Seller/Buyer SIREN-SIRET, BT-3 invoice type code, VAT breakdown by rate, payment terms, French legal mentions); which fields require customer-specific configuration vs. are auto-populated; and how SIREN/SIRET data is sourced from ERP master data]** — content to be added in a subsequent iteration.

## Connecting to the PA

### Outbound PA Integration

> **[STUB: Describe the technical connection from Tungsten AR to the outbound PA — whether REST API, AS2/AS4, or both are supported; the authentication mechanism; how submission acknowledgements and validation errors from the PA are surfaced back to Tungsten and the ERP]** — content to be added in a subsequent iteration.

## E-Reporting

### B2C E-Reporting from Tungsten

> **[STUB: Describe how Tungsten AR handles the B2C e-reporting obligation — whether B2C transaction data is automatically extracted from AR transactions and submitted via the PA, or whether a separate configuration or process is required; what data fields Tungsten sources from the ERP for B2C reports]** — content to be added in a subsequent iteration.

### Cross-Border B2B E-Reporting

> **[STUB: Describe how Tungsten AR identifies cross-border B2B transactions and generates the required e-report (seller SIREN, amounts, VAT, buyer country code); whether this is automatic or requires operator configuration per customer; and how the submission is routed to the PA]** — content to be added in a subsequent iteration.

### Payment Data Reporting (Outbound)

> **[STUB: Describe how Tungsten AR handles payment data reporting for outbound invoices — whether Tungsten receives payment confirmation signals from the ERP or a downstream treasury system, how that triggers PA submission of payment data (invoice reference, payment date, amount, method), and whether this is automated or requires operator action]** — content to be added in a subsequent iteration.

## Closing the Loop

### Status Tracking and ERP Acknowledgement

> **[STUB: Describe how delivery and acceptance statuses returned by the recipient's PA are received by Tungsten AR and surfaced back to the ERP and AR operator; how failed deliveries (Rejetée, Refusée) are escalated; what the operator sees and can do in Tungsten when a status exception occurs]** — content to be added in a subsequent iteration.

### Operator Visibility and Audit Trail

> **[STUB: Describe what visibility Tungsten AR provides to outbound operators — per-invoice status tracking, exception queues, dashboards; and the audit trail Tungsten maintains for compliance purposes (submission timestamps, PA acknowledgements, status history) in the context of the 10-year archiving obligation]** — content to be added in a subsequent iteration.
