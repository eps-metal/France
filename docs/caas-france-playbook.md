# French eInvoice Reform Playbook — InvoiceAgility

**Source:** CaaS deck, `pipeline/a1-source/product/caas/French eInvoice Reform Playbook/`
**Product:** InvoiceAgility (IvA)
**Note:** Architecture section marked WIP in deck

---

## SteerCo Team

| Function | Owner |
|----------|-------|
| Product Management | Emily Nash-Walker |
| Country Compliance | Ruud van Hilten |
| R&D | Mattias Reichler & James Sealey |
| CS SVP | Tommy Triplett |
| PS | Chris Williamson |
| CSM | Kristel Beurrios |
| TS | Gareth Jones |
| Cloud | Joakim Billings |

---

## Communication Plan

| Date | Audience | Message | Sent By |
|------|----------|---------|---------|
| March 11 | French Buyers and Suppliers | Overview of French eInvoice Mandate; Tungsten certified as PA | Standard Comms (Kelly Frakes) |
| March 11 | PS Projects rolling out into France | Overview of mandate; PA certification; Launch Planning | PS PM |
| March 23 | TBD | TBD | TBD |
| April 6 | TBD | TBD | TBD |

---

## Architecture and Data Flows

### Transactional Data Flow Overview

Three flow types:
- **E-invoicing flow** (orange) — domestic B2B sales only
- **E-reporting flow** (light blue) — cross-border B2B & B2C sales + international purchases
- **Invoice & payment status flow** (blue/gray)

**Supplier side (PAe):**
- Maps data to core-set or specific format
- Applies controls
- Sends mandatory fields to public portal (PPF)
- Sends lifecycle status to public portal

**Buyer side (PAr):**
- Maps data to buyer format
- Sends lifecycle status back to supplier & public portal

---

### AP Invoice Workflow (Inbound to Buyer)

```
Sending PAe ──┐
              ├──→ Tungsten Automation PAr ──→ Tungsten InvoiceAgility ──→ Buyer
Central Dir ──┘                                                          (invoices,
                                                                          reporting,
                                                                          lifecycle)
```

- Tungsten acts as **PAr** (receiving PA) for AP/inbound flows
- Central Directory and Data Concentrator feeds routing data to Tungsten PAr
- IvA receives payload from PAr and delivers to buyer systems

---

### AR Invoice Workflow (Outbound from Supplier)

```
Supplier ──→ Tungsten InvoiceAgility ──→ Tungsten Automation PAe ──→ Receiving PAr ──┐
                                                                                      ├──→ Buyer
                                                                    Central Dir ──────┘
```

- Tungsten acts as **PAe** (sending PA) for AR/outbound flows
- IvA constructs the payload and hands off to Tungsten PAe for PA-to-PA transmission
- Central Directory and Data Concentrator used for routing on recipient side

---

### Data and Reporting Flow Reference (14 flows)

**E-Invoicing (Flows 1–5):**
1. Scenario A — Mandatory Invoicing Data, Core Set Format
2. Scenario A/B1/B2/C — Full Invoicing Data, Core Set Formats
3. Scenario C — Full Invoicing Data, Other Formats
4. Scenario — Supplier to PPF (backward compatibility)
5. PPF to Buyer (backward compatibility)

**Lifecycle + Payment (Flows 6–7):**
6. Core Set Formats
7. Other Formats (excluding flows to PPF)

**E-Reporting (Flows 8–10):**
8. International B2B (Sales & Purchases) — Core Set Formats
9. B2C Invoices — Core Set Formats
10. Specific e-Reporting formats:
    - 10.1 B2B & B2C invoicing data (when not available in flows 8 & 9)
    - 10.2 Payment data for invoices (flows 8/9), invoicing data (10.1), or e-invoices*
    - 10.3 B2C transaction data (invoiceless transactions)
    - 10.4 B2C transaction payment data (invoiceless transactions)

**Central Directory (Flows 11–14):**
11. Directory consultation request from Supplier for invoice addressing
12. Directory update request from Buyer to Public Portal
13. Directory update by a PA at Buyer request
14. Directory consultation request from a PA for routing invoices to a Buyer

> *When not available through flows 6 or 7

---

### Standard Invoicing Transaction Flow (IVA as OD)

Swimlane sequence — Supplier → IVA → Sovos PAe → Sovos PAr → IVA → Buyer → Government:

1. **Supplier** — Billing data
2. **Tungsten IVA** — Create payload in UBL
3. **Sovos PAe** — Validate invoice → Consult Annuaire → Create legal invoice → Send legal invoice to PAr → (async) Create Flow 1 report and send to PPF
4. **Sovos PAr** — Receive legal invoice → Technical validation → Create buyer payload
5. **Tungsten IVA** — Receive payload from PAr → Create final buyer payload
6. **Buyer** — Receive payload and process
7. **Government** — Receive Flow 1 report and update tax systems

---

### Mandatory Status Messages (CDAR format)

> **All message formats (200, 210, 212, 213) use CDAR**

| Message | Code | Direction | Meaning |
|---------|------|-----------|---------|
| Invoice delivered to PAr | **CDAR 200** | PAe → PPF | Submitted/delivered |
| Invoice rejected (technical) | **CDAR 213** | PAr → PPF | Technical validation failure |
| Invoice refused (business) | **CDAR 210** | PAr → PPF | Business refusal by buyer |
| Invoice paid | **CDAR 212** | PAe → PPF | Payment confirmed |

**Flow:**
- PAe sends CDAR 200 (delivered) to PPF when invoice successfully transmitted to PAr
- PAr sends CDAR 213 (rejected) if technical validation fails; CDAR 210 (refused) for business refusal
- Both rejection and refusal status messages flow back left through PAe → IvA → ISS → Supplier
- Paid status (CDAR 212) originates from supplier side ("declares invoice paid") → PAe → PPF; IvA updates ISS

---

## IVA in France — Integration Scenarios

### Key Principle
> **TeN is the OD (origination/destination) that connects to PAe and PAr — all systems must eventually connect to TeN**

### Supplier Source Systems
- WF (Workflow)
- IS (Invoice Submission)
- OIC / Process Director
- Other (third-party)

### Buyer Target Systems
- APS / RSI / Markview
- Process Director (PDAP)
- ERP
- Non-TA workflow/cockpit

---

### Outbound Scenarios (AR — Supplier Side)

| Supplier Entry | Path to PAe |
|---------------|-------------|
| PD OIC | PD OIC → TeC → TeN → PAe |
| ERP (with TeC) | ERP → TeC → TeN → PAe |
| ERP (direct) | ERP → TeN → PAe |

---

### Full End-to-End Scenarios (Outbound + Inbound)

| # | Outbound | | Inbound |
|---|----------|--|---------|
| 1 | PD OIC → TeC → TeN → PAe | PAr → | Buyer (non-TA) |
| 2 | ERP → TeC → TeN → PAe | PAr → | Buyer (non-TA) |
| 3 | ERP → TeN → PAe | PAr → | Buyer (non-TA) |
| 4 | PD OIC → TeC → TeN → PAe | PAr → TeN → | Non-TA |
| 5 | ERP → TeC → TeN → PAe | PAr → TeN → | Non-TA |
| 6 | ERP → TeN → PAe | PAr → TeN → | Non-TA |
| 7 | PD OIC → TeC → TeN → PAe | PAr → TeN → TeC → APS → | Non-TA |
| 8 | ERP → TeC → TeN → PAe | PAr → TeN → TeC → APS → | Non-TA |
| 9 | ERP → TeN → PAe | PAr → TeN → TeC → APS → | Non-TA |
| 10 | PD OIC → TeC → TeN → PAe | PAr → TeN → TeC → APS → | PDAP |
| 11 | ERP → TeC → TeN → PAe | PAr → TeN → TeC → APS → | PDAP |
| 12 | ERP → TeN → PAe | PAr → TeN → TeC → APS → | PDAP |

---

### Lifecycle Flows (Mandatory Only)

Full chain: `PD OIC → TeC → TeN → PAe → PAr → TeN → TeC → APS → PDAP`

**AR side (outbound/supplier) status messages:**
- CDAR 200 — invoice submitted (PAe → PPF)
- Invoice rejected → flows back: PAe → TeN → TeC → PD OIC
- Invoice refused → flows back: PAe → TeN → TeC → PD OIC
- Invoice paid → supplier declares via ISS; flows: PD OIC → TeC → TeN → PAe → PAr → PPF

**AP side (inbound/buyer) status messages:**
- CDAR 200 — invoice submitted (displayed at PAr)
- CDAR 210 — invoice rejected (PAr → PPF); flows forward: PAr → TeN → TeC → APS → PDAP
- CDAR 213 — invoice refused (PAr → PPF); flows forward: PAr → TeN → TeC → APS → PDAP
- CDAR 212 — invoice paid (PAe → PAr → PPF)

---

### E-Reporting Flows

**Outbound e-reporting (domestic, international, B2C, payment):**
```
ERP / PD OIC → TeC → TeN → PAe → PPF
                                  ↓
                         Reporting status (back to sender)
```
- Reporting file generated at ERP/OIC and PD OIC level
- **Flow 1**: Domestic invoices — not aggregated

**Inbound e-reporting (international purchases):**
```
PPF ← PAr ← TeN ← TeC ← APS ← PDAP ← ERP
 ↓
Reporting status (forward to buyer systems)
```
- **Flow 10**: International, payment, and B2C — aggregated

---

## Use Case Inventory

- Full use case inventory maintained in: **French PA Use Case Inventory.xlsx**
- 44 total French use cases (reference from other decks)
