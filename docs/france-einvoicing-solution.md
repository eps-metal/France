---
title: Tungsten Automation — France E-Invoicing Solution
doc_type: enablement
audience: customer-facing
created: 2026-03-10
status: draft
---

# Tungsten Automation — France E-Invoicing Solution

## The Tungsten France Stack

- **Tungsten is registered PA #0072**, officially confirmed by DGFiP — authorized to send, receive, and report e-invoices under the French mandate
- The PA runs on white-labelled **Sovos infrastructure** in a **SecNumCloud-certified environment** (EU-only access); all PA functions are surfaced through Tungsten's own UI — customers never interact with Sovos directly
- **Invoice Agility (IvA)** combines two engines: **Tungsten e-Network (TeN)** for tax-authority network delivery and **Tungsten e-Connect (TeC)** for Peppol connectivity — IvA automatically selects the right channel per transaction
- **SAP Outbound Invoice Cockpit (OIC)**, built on Process Director, captures billing events via standard BTEs (non-invasive), enriches documents with master data, and routes them to IvA for compliant delivery
- **44 French use cases** are supported end-to-end; 7 are available today for Sovos testing, with additional use cases rolling out through 2026
- Supported invoice formats: **Factur-X** (hybrid PDF+XML), **UBL 2.1**, and **CII** — all EN 16931 + CIUS-FR compliant

## How the Architecture Works

France operates a **Y-model** — invoices travel PA-to-PA directly. The PPF (Portail Public de Facturation) is never in the invoice path; it maintains the **Annuaire** (national directory) and relays reporting data to DGFiP.

### Invoice flow

```
Outbound:  ERP → OIC → IvA (creates UBL) → Tungsten PAe → Recipient PAr → Recipient system
                                                  │                              │
                                                  ├── Annuaire lookup (routing) ─┘
                                                  └── PPF (e-reporting / Flow 1)
                                                            │
                                                            └── DGFiP

Inbound:   Sender PAe → Tungsten PAr → IvA → ERP / AP system
                              │
                              └── PPF (lifecycle status updates)
```

- Tungsten acts as **PAe** (sending PA) on outbound flows and **PAr** (receiving PA) on inbound flows — two distinct roles within the same certified PA infrastructure
- **IvA constructs the UBL payload** from ERP source data before handing off to the PAe for validation and transmission — format conversion happens inside IvA, not at the PA layer
- **Tungsten PAe** validates against EN 16931 + CIUS-FR, queries the Annuaire for the recipient's registered PAr, and transmits PA-to-PA
- PA-to-PA exchange currently uses **Peppol BIS 3 UBL** over Peppol transport; non-Peppol PA-to-PA channels arrive Q3-2026
- Lifecycle status messages flow back through the same path into the Invoice Status Service (ISS), TeN portal, and SAP cockpit

## Sending Invoices (AR / Outbound)

### Flow: ERP → customer's recipient PA

| Step | Component | What happens |
|------|-----------|-------------|
| 1 | **SAP** | Billing document created; BTE fires |
| 2 | **OIC** | Captures event, enriches with master data (VAT, SIRET, payment terms), applies business rules |
| 3 | **IvA** | Converts to compliant format (Factur-X, UBL, or CII per recipient capability), selects delivery channel |
| 4 | **Tungsten PAe** | Validates against EN 16931 + CIUS-FR, looks up recipient PAr in Annuaire, transmits PA-to-PA; sends CDAR 200 (delivered) to PPF |
| 5 | **Recipient PAr** | Receives invoice; sends CDAR 213 (technical rejection) or CDAR 210 (business refusal) if not accepted |
| 6 | **OIC** | Delivery statuses surface in SAP cockpit via Invoice Status Service; CDAR 212 (paid) sent to PPF when supplier confirms payment |

- OIC captures SAP billing events through standard BTEs — no custom ABAP, no invasive modifications
- IvA handles format conversion and channel routing without manual intervention
- All 44 French use cases covered, including self-billing, down-payment sequences, and discount scenarios

## Receiving Invoices (AP / Inbound)

### Flow: sender PA → your ERP

| Step | Component | What happens |
|------|-----------|-------------|
| 1 | **Sender PAe** | Transmits invoice PA-to-PA to Tungsten PAr |
| 2 | **Tungsten PAr** | Receives, performs technical validation; sends CDAR 213 (rejected) to PPF if validation fails |
| 3 | **IvA (TeN)** | Normalizes to your target format, applies enrichment and matching rules |
| 4 | **ERP / AP system** | Invoice lands in your workflow — ready for PO matching, coding, approval |
| 5 | **Tungsten PAr** | Sends CDAR 210 (refused) to PPF on business refusal; CDAR 212 (paid) to PPF when payment confirmed |

- Inbound invoices arrive in any of the three mandate formats (Factur-X, UBL, CII) — IvA normalizes regardless of source format
- Exception handling and matching logic run inside IvA before delivery to ERP, reducing manual touchpoints
- PAr statuses (mandatory: received, processing, accepted; optional: refused, paid) are managed automatically by the PA and visible in TeN portal

## Lifecycle & E-Reporting

### Lifecycle statuses (CDAR format)

| Code | Message | Triggered by | Meaning |
|------|---------|-------------|---------|
| **CDAR 200** | Delivered | Tungsten PAe | Invoice successfully transmitted to recipient PAr |
| **CDAR 213** | Rejected | Tungsten PAr | Technical validation failure at receiving PA |
| **CDAR 210** | Refused | Tungsten PAr | Business refusal by buyer (e.g., duplicate, wrong PO) |
| **CDAR 212** | Paid | Tungsten PAe | Payment confirmed by supplier |

- All CDAR messages are sent to PPF automatically — no customer action required
- Rejection (213) and refusal (210) statuses flow back through IvA to the Invoice Status Service (ISS) and surface in the SAP cockpit
- All lifecycle messages visible in **TeN portal** and pushed back to ERP/SAP cockpit via ISS

### E-reporting (data to PPF → DGFiP)

| Flow | Scope | Method |
|------|-------|--------|
| **Flow 1** | Domestic B2B invoices | Per-invoice, not aggregated — Sovos extracts and submits automatically |
| **Flow 10** | International B2B, B2C, and payment data | Aggregated on schedule via Sovos Scheduler |

- **Flow 1** fires on every domestic invoice transaction — no customer action required
- **Flow 10** covers: cross-border B2B invoices sent/received, B2C transaction data, payment data for all invoice categories
- Payment data reporting is separate from CDAR lifecycle updates — triggered by actual payment events, not invoice status changes

## Getting Live

### Onboarding steps

- **Entity hierarchy**: set up in OBi (Tungsten's platform) with correct **SIREN/SIRET structure** before go-live — head office entity + establishment children (max 1 level deep); PS team assists with initial configuration
- **Company registration**: each entity needs SIRET as company registration ID; SIREN embedded in VAT number (format: FR + 2 check digits + 9-digit SIREN)
- **e-Address registration**: supported formats include EDEN, SIREN, SIRET, UPEN, UPEM, and UPF — registered in the Annuaire for inbound routing
- **KYC (Know Your Business)**: required for PA registration under the regulatory framework; Tungsten facilitates the process

### Timeline to go-live

| Milestone | Timing | What it means |
|-----------|--------|---------------|
| Grand Pilot | Q1 2026 | Sovos + PA infrastructure validated |
| UAT / Pilot | April–May 2026 | Customer testing against Sovos sandbox (TeN portal LIVE/TEST toggle); **pilot = LIVE production with real transactions** |
| **Go-live** | **Sept 1, 2026** | All businesses must receive; GE + ETI must send |
| Full mandate | Sept 1, 2027 | SMEs and micro-enterprises must send |

- Tungsten strongly recommends completing full UAT before entering pilot — pilot is not a sandbox, it processes real invoices in production
- TeN portal provides a **LIVE/TEST toggle** for UAT testing against the Sovos sandbox environment
