# Pipeline Source Findings — French E-Invoicing Mandate

> Files found in `~/projects/pipeline` with French e-invoicing content modified within the last four months (after 2025-11-10). Searched 2026-03-10.

---

## Primary Sources

### France Mandate Workshop Action Items
**Path:** `a3-output/prod-mgt/France_Mandate_Workshop_Action_Items_Combined.md`
**Modified:** 2026-02-26

Workshop notes from Feb 24–25, 2026. The most current and operationally detailed source.

Key content:
- PA registration crisis: only 5 of 134 French customers registered (3.7%)
- French Y-model architecture: Peppol UBL 2.1, CII, Factur-X formats
- 42 mandate use cases defined by France; ~30 mapped by Tungsten
- Lifecycle reporting: 4 mandatory statuses (submitted, rejected, refused, paid)
- Business scale: 1.14M transactions, €15B USD value, 9,000 suppliers
- Revenue context: France = 14% of AP/AR ARR
- 59 critical/high priority action items across 9 categories
- Sovos partnership issues documented
- CEDAR status messages coverage

---

### France Enablement Technical Update
**Path:** `a2-processed/prod-mgt/2026.1/france-enablement-2025-11-25.md`
**Modified:** 2026-02-18

Technical enablement document from November 25, 2025. Best source for architecture and integration detail.

Key content:
- 4 reform elements: e-invoicing (domestic B2B/B2G), e-reporting (B2C + cross-border), lifecycle status reporting, payment status declaration
- Transaction data flows and ecosystem diagram
- SecNumCloud certification requirements for PA environment
- Peppol BIS 3 UBL format for PA-to-PA exchange
- IVA (Invoice Agility) component development timelines
- AP/AR workflow diagrams: Tungsten PA integration points
- TeN/TeC environment architecture for CTC (Continuous Transaction Controls)
- Customer onboarding: Tungsten PA opt-in, company hierarchy, portal details

---

### France Compliance Demo Status (Email Thread)
**Path:** `mail-based-knowledge/in-process/2026-02-06; 13-04; van-Hilten; france-compliance-demo-status.md`
**Modified:** 2026-02-18

Email thread from Feb 6, 2026 documenting a gap between product announcement claims and actual demo capability.

Key content:
- PA accreditation achieved; pilot phase registration completed
- Demo gap: IVA UI not yet demoable natively
- Active demos rely on Sovos sandbox video rather than native IVA UI
- Action items: improve native IVA demo capabilities, align announcements with deliverable status

---

### Corrective Invoices — Poland/France Impact (Email Thread)
**Path:** `mail-based-knowledge/in-process/2025-12-03; 21-49; Nguyen; corrective-invoice-poland-france-impact.md`
**Modified:** 2026-02-18

Email thread from Dec 3, 2025 on corrective invoices as a new mandatory document type.

Key content:
- Corrective invoices required under France mandate (alongside Poland KSeF)
- Impact on PDAP 3-way matching logic, workflow routing, reference tracking, variance recalculation
- Open questions: SAP standard handling, PDAP technical requirements, roadmap inclusion

---

## Supporting Sources

### E-Invoicing Compliance Capability Brief
**Path:** `a3-output/iva2/iteration5/Capabilities/CAP-E-Invoicing-Compliance.md`
**Modified:** 2026-03-04

Platform capability brief. Most recently modified file in the set.

Key content:
- France positioned as a reporting-model jurisdiction
- Factur-X format support (hybrid PDF/XML)
- Peppol network integration
- Chorus Pro connectivity (B2G)
- Tax determination and digital signatures

---

### E-Invoicing Standards — OIC Skills Knowledge Base
**Path:** `a0-project/product/pd/oic/.claude/skills/e-invoicing-standards.md`
**Modified:** 2026-02-18

Internal knowledge base used for OIC (Order-to-Invoice Compliance) product context.

Key content:
- Factur-X profiles: Minimum, Basic, EN16931
- SIRET number requirement for France compliance
- EN 16931 core standard + CIUS-FR extension
- Country-specific validation rules

---

### France Slides Forward (Email, Noise)
**Path:** `mail-based-knowledge/processed/noise/2026-01-22; 16-34; Nash-Walker; france-slides-forward.md`
**Modified:** 2026-02-18

Forwarded email from Emily Nash-Walker. Classified as noise. Minimal content; references France-related slide deck.

---

## Summary

| Source | Type | Modified | Value |
|--------|------|----------|-------|
| France Mandate Workshop Action Items | Workshop notes | 2026-02-26 | Highest — most current, operational depth |
| France Enablement Technical Update | Technical doc | 2026-02-18 | High — architecture and integration detail |
| Compliance Demo Status | Email thread | 2026-02-18 | Medium — product gap context |
| Corrective Invoice Impact | Email thread | 2026-02-18 | Medium — corrective invoice requirement |
| E-Invoicing Compliance Capability Brief | Capability brief | 2026-03-04 | Medium — product positioning |
| E-Invoicing Standards (OIC) | Knowledge base | 2026-02-18 | Low-medium — format/standard reference |
| France Slides Forward | Email (noise) | 2026-02-18 | Low — content not retrieved |
