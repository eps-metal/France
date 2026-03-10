# France Mandate – Technical Enablement Update (March 2026)

**Source:** CaaS deck, `pipeline/a1-source/product/caas/FRANCE enablement_Mar 26/`
**Presenters:** James Sealey (Director Product), James Hopton (Product Owner)

---

## Key Product Terminology

| Term | Meaning |
|------|---------|
| **TeN** | Tungsten Network (the e-invoicing platform) |
| **TeC** | Tungsten e-Connect (integration/connector product) |
| **Sovos** | Tungsten's PA partner for France — officially confirmed as PA by French tax authority |
| **OBD** | On-boarding environment (Customer UAT) |
| **PAe / PAr** | PA émetteur (sender) / PA récepteur (receiver) |
| **KYC/B** | Know Your Customer/Business onboarding |
| **PD** | Partner Directory (production & UAT) |

---

## TeN Roadmap (aligned with Sovos)

**Q4-25:** Tungsten officially confirmed as Plateforme Agréée by French tax authority

**Q1-26 — Phase 1: Pilot Initiation**
- Integration with FR Public Portal & Central Directory (PS supported)
- Inbound & Outbound e-Invoicing (basic use cases + volumes)
- Invoice Lifecycle: mandatory status between PAs
- e-Reporting: basic pass-through, limited volume
- KYC: Manual — status UNDEFINED
- PA-to-PA: Peppol only

**Q2-26 — Phase 2: Incremental Enhancements (April/May release)**
- FR Public Portal & Central Directory integration (self-serve UAT)
- +13 additional e-invoicing use cases
- Enhanced invoice lifecycle status
- e-Reporting consolidation — flagged as RISK

**Q3-26 — Phase 3: Production Ready**
- ALL remaining e-invoicing use cases
- Automated off-boarding
- PA-to-PA out of Peppol (non-Peppol transports)
- High volume capacity
- KYC Automated — flagged as RISK

**Go Live: Sept 1, 2026** — Large & Medium must send; all must receive

---

## Use Cases

- **44 total** use cases exist in France (some industry-specific)
- **7 launch scope** for initial Sovos testing (March/April):
  - 0: Standard invoice
  - 2: Invoice already paid by Buyer or Third Party
  - 19b: Self-Billing
  - 20: Down-payment invoice
  - 21: Final invoice after down-payment
  - 22a: Discounted service invoice with VAT due on payment
  - 22b: Discounted invoice with debit-basis VAT (goods/services)
- Tungsten can already support **all 44** use cases internally (mappings exist), but only 7 are testable end-to-end via Sovos initially

---

## Pilot is LIVE — Critical Point

> The pilot operates in a **live production environment** — actual business transactions are exchanged. It is **not** a test/sandbox environment with dummy data.

All customers using Tungsten PA are strongly encouraged to complete a thorough UAT round before entering pilot.

---

## What's Supported by Phase

| Area | March/April (Pilot init) | May (UAT) | Pilot |
|------|--------------------------|-----------|-------|
| **Annuaire** | Simple SIRET/SIREN reg + modification | + Test Mode in TeN portal (self-serve UI) | Same |
| **KYC** | Manual onboarding, PA Consent docs | N/A for UAT | TBD |
| **Invoice Processing** | AR payload creation, AP payload reception | + more end-to-end coordination via PAs | Same |
| **Lifecycle** | Mandatory PAe/PAr messages | Automated codes (submitted/received); optional codes (paid, refused) NOT YET | Same |
| **E-reporting** | Pass-through (complete files only), file replacement | Same; optional lifecycle codes NOT YET | N/A |

---

## Environments

- **TeN Production** ↔ Sovos Production ↔ Other PAs Production + French Gov Production
- **TeN OBD** (Customer UAT) ↔ Sovos Sandbox ↔ Other PAs Sandbox + French Gov Test
- **TeN Test0** ↔ TeC Dev/Test (internal)
- TeC Partner (Sign Up live) → TeC Production → TeN Production
- TeN portal has a **LIVE/TEST toggle** — switches session to test environment; warns unsaved work may be lost

### Test Account Setup (OBD)
- Create test AAA accounts on OBD for simple SIREN/SIRET
- Use Postman to manually call Sovos API to create records in PA environment
- Access Sovos PA environment to verify record creation

---

## PS Consultation Guide

What Professional Services should address per phase:

| Topic | UAT | Pilot |
|-------|-----|-------|
| Test accounts (OBD) | Create/utilise test accounts for simple SIREN/SIRET | Create/review existing AAAs against FR entities; review hierarchy |
| KYC | N/A for UAT; engage early so client considers Pilot | Consider who will need to complete for Pilot phase |
| Invoicing scenarios | Which use cases apply; align to UAT phases; payload requirements | Same |
| Lifecycle | Which lifecycle statuses to submit; mandatory only or also optional? | Same |
| e-reporting | (blank — TBD) | (blank — TBD) |

---

## TeC Dependencies on TeN

- **On-boarding:** Interop API must be updated to support SIREN/SIRET setting in TeN

## TeC Documentation Pending

Three mapping docs in progress:
1. Invoice Mapping
2. LifeCycle Mapping
3. E-Reporting Mapping
