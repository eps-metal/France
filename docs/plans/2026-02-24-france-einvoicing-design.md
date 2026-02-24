# France E-Invoicing Mandate — Content Design

**Date:** 2026-02-24
**Project:** `/home/ethen/projects/france`
**Status:** Approved

---

## Purpose

Produce educational and operational content explaining how inbound (AP) and outbound (AR) e-invoicing will work under France's mandatory B2B e-invoicing regime. Content will be used by both internal Tungsten Automation staff and customers/prospects. Format is Markdown structured to support presentation delivery (e.g., PowerPoint).

---

## Audience and Levels

Four progressive levels. The first three are education; the fourth is implementation/operation.

| Level | Name | Audience assumption | Focus |
|---|---|---|---|
| 1 | Intro | No prior knowledge | What, why, and when |
| 2 | Intermediate | Aware of the mandate | How the ecosystem works |
| 3 | Advanced | Familiar with the ecosystem | Technical and regulatory depth |
| 4 | Product | Tungsten users / implementers | Tungsten products in action |

The Product level will be stub-heavy on first pass, with content areas flagged for subsequent iterations.

---

## Regulatory Foundation

All content must reflect the **current (2026)** state of the mandate. Much published content is outdated. Key facts:

| Element | Current (2026) |
|---|---|
| Platform name | PA (Plateforme Agréée) — replaces the former term PDP |
| Exchange architecture | **Y-model**: PA-to-PA direct exchange; PPF is directory + DGFiP data concentrator only |
| National directory (Annuaire) | Maintained by PPF; businesses register their PA; drives invoice routing |
| PPF role | Directory + data relay to DGFiP; not an invoice exchange hub |
| B2G invoicing | Chorus Pro (unchanged) |
| Supported formats | Factur-X, UBL 2.1, CII — all conforming to EN 16931 + CIUS-FR extension |
| E-reporting scope | B2C transactions, cross-border B2B, **and payment data reporting** for B2B/B2C/B2G |
| DGFiP | French Peppol authority |
| Approved PAs | 101 PAs listed as of January 17, 2026 |
| Legal anchor | PLF 2026 (Finance Bill) adopted February 2, 2026 |
| Timeline certainty | Postponement rejected by National Assembly in April 2025; September 2026 is locked |

**Key dates:**

| Date | Obligation |
|---|---|
| September 1, 2026 | All businesses must be able to **receive** e-invoices |
| September 1, 2026 | **GE** (5,000+ employees; >€1.5B turnover or >€2B balance sheet) and **ETI** (250–5,000 employees; <€1.5B turnover or <€2B balance sheet) must **send** e-invoices and comply with e-reporting |
| September 1, 2027 | **SMEs** and **micro-enterprises** must **send** e-invoices and comply with e-reporting |

---

## File Structure

One file per topic per level. Inbound (AP) and outbound (AR) are the primary organizing axis within each level.

```
france-einvoicing/
│
├── intro/
│   ├── inbound.md
│   └── outbound.md
│
├── intermediate/
│   ├── inbound.md
│   └── outbound.md
│
├── advanced/
│   ├── inbound.md
│   └── outbound.md
│
└── product/
    ├── inbound.md
    └── outbound.md
```

---

## Content Map

### intro/inbound.md
- What is e-invoicing?
- Why is France mandating it? (VAT fraud prevention, modernisation)
- What does "receiving an e-invoice" mean in practice?
- Who is affected and when?
- What does a business need to do at a high level?

### intro/outbound.md
- What does "sending an e-invoice" mean?
- Who must send, and by when? (GE + ETI: Sept 2026; SME: Sept 2027)
- What is a PA and why do I need one?
- High-level steps to get ready

### intermediate/inbound.md
- The ecosystem: PA, PPF/Annuaire, DGFiP, Y-model
- How an inbound e-invoice travels from sender's PA to recipient's PA
- The Annuaire: how routing works
- E-reporting obligations on the receiving side
- Format overview: Factur-X, UBL 2.1, CII
- Difference between e-invoicing and e-reporting

### intermediate/outbound.md
- Choosing and connecting to a PA (101 listed PAs)
- Full outbound Y-model flow: ERP → PA → Annuaire lookup → recipient PA → recipient
- E-reporting for B2C, cross-border B2B, and payment data
- Format requirements: when to use Factur-X vs UBL vs CII
- CIUS-FR: what French-specific fields are required

### advanced/inbound.md
- DGFiP External Specifications v3.1 as the technical reference
- Invoice status lifecycle (acknowledgement, validation, rejection states)
- Validation rules (50+ mandatory business rules, schematron)
- Error handling and dispute flows
- Peppol interoperability (DGFiP as French Peppol authority)
- Connectivity: AS2/AS4 and REST API
- Archiving obligations

### advanced/outbound.md
- Outbound technical specifications
- Mandatory data fields under CIUS-FR extension
- Schematron validation on outbound
- E-reporting data structure and submission mechanics
- Payment data reporting: scope, timing, format
- PLF 2026 legal detail and implications
- Penalties and compliance risk

### product/inbound.md *(stub-heavy)*
- [STUB] Overview: where Tungsten AP products sit in the inbound flow
- [STUB] PA connectivity: how Tungsten connects to / acts as a PA
- [STUB] Inbound invoice receipt and format handling
- [STUB] Validation and enrichment before ERP handoff
- [STUB] ERP integration points
- [STUB] Exception handling and operator workflow
- [STUB] E-reporting obligations handled by Tungsten

### product/outbound.md *(stub-heavy)*
- [STUB] Overview: where Tungsten AR products sit in the outbound flow
- [STUB] PA connectivity for outbound
- [STUB] Format generation: Factur-X / UBL / CII from ERP data
- [STUB] CIUS-FR field mapping from ERP source fields
- [STUB] E-reporting: how Tungsten handles B2C, cross-border, payment data
- [STUB] ERP integration points for outbound
- [STUB] Status tracking and acknowledgement back to ERP

---

## Structural Principles

- Each file maps to a **slide section or chapter** in a presentation
- Headings within files map to **individual slides**
- Content is written for the level's assumed knowledge — no backward references required
- Product-level stubs use a consistent `[STUB: topic]` marker so gaps are visually obvious
- All content is Markdown; no HTML, no inline styles

---

## Out of Scope (First Pass)

- B2G (Chorus Pro) beyond introductory mention
- Non-French EU e-invoicing mandates
- Specific PA vendor comparisons
- Detailed ERP-specific configuration guides (deferred to product stubs)
