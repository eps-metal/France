# French E-Invoicing Use Cases: Priority & Volume Analysis

**Prepared:** 2026-03-18
**Scope:** All French industries (cross-market, not sector-specific)
**Baseline:** ~2–2.5 billion B2B invoices per year across ~4 million VAT-registered entities
**Sources:** DGFiP specifications, AFNOR XP Z12-014, Symtrax, FNFE-MPE, INSEE, ASF (factoring), PwC, EY, KPMG France, ITESOFT, sector associations

---

## Methodology Notes

- Volume shares are **not mutually exclusive** — a single invoice can exhibit multiple use cases (e.g., a multi-order invoice (UC1) that is also payable to a factor (UC8))
- No authoritative source publishes a use-case-level volume breakdown; all estimates are derived from market data and first-principles reasoning
- "Standard" B2B (single order, single delivery, direct buyer-to-seller payment, no third parties) likely accounts for **40–55%** of total volume — all other percentages represent invoices that exhibit the listed characteristic
- Priority reflects customer implementation urgency, not regulatory importance per se
- Volume confidence levels: **High** = data-backed, **Medium** = reasoned from sector data, **Low** = informed estimate

---

## Full Use Case Table

| # | Name | Priority | Est. Volume Share | Volume Confidence | Key Drivers |
|---|------|----------|-------------------|-------------------|-------------|
| **1** | Multi-order / Multi-delivery | **High** | 25–35% | Medium-High | LME monthly consolidated billing; manufacturing, wholesale, retail supply chains; EN 16931 base standard lacked support — France extended for it |
| **Interco** | Intercompany invoices *(non-official)* | **High** | 15–25% | Medium | Every GE/ETI (Wave 1) has multiple legal entities; group structures, SSCs, management fees, cost allocations; ~74% of French turnover in corporate groups |
| **34** | Partial collection / cancellation of collection | **High** | 8–15% | Medium | Default VAT regime for ALL services is cash-accounting (TVA sur encaissements); services = ~75% of French GDP; lifecycle status reporting mandatory (CDAR codes) |
| **38** | Sub-lines and line grouping | **Medium-High** | 10–20% | Medium | Data-structure issue affecting manufacturing, utilities, telecom, IT bundles, construction; ERP flattening makes mapping non-trivial |
| **3** | Third-party payer (known) | **High** | 8–12% | Medium | Group treasury centralization standard for ETI/GE; INSEE: groups control 71% of French added value; also covers insurance co-payments and construction direct payment |
| **8** | Payable to designated third party (factor/treasury) | **High** | 8–12% | Medium-High | France is #1 factoring market in Europe — €425.9B factored in 2023 (~16% GDP); ~31K–33K companies use factoring; factor must be registered in Annuaire as payee |
| **20** | Advance payment invoice (acompte) | **High** | 5–10% | Medium | Legally mandatory when deposit received (CGI art. 289-I-1-c); standard in BTP (20–30% deposits), professional services, capital goods, software projects |
| **21** | Final invoice after advance payment | **High** | 5–10% | Medium | Mandatory complement to UC20; 1:1 pairing; offset mechanics require linking advance and final invoices to avoid double VAT reporting |
| **32** | Monthly / recurring payments | **High** | 5–10% | Medium-High | Energy (3.77M professional sites), telecom (every business), SaaS, leasing, maintenance contracts; virtually every business sends and receives recurring invoices |
| **5** | Employee-paid expenses with invoice | **High** | 5–8% | Medium | Universal applicability; invoices >€150 HT in company's name enter e-invoicing circuit; 36% of tax audit adjustments relate to expense documentation |
| **11** | Invoice sent to third party on behalf of buyer | **Medium-High** | 5–8% | Medium | Billing agents, property management (syndics, ~750K copropriétés), corporate SSCs, outsourced AP; Annuaire must support buyer ≠ routing recipient |
| **31** | Mixed invoice (main + ancillary transaction) | **High** | 3–6% | Medium | Extremely common — equipment + installation, software + services, goods + transport; VAT classification by dominant operation; broad cross-industry applicability |
| **10** | Beneficiary unknown at invoice creation (subrogation) | **Medium** | 3–5% | Medium | Confidential/post-hoc factoring; supplier assigns receivable to factor after invoice issued; meaningful share of France's massive factoring market |
| **2** | Invoice already paid by buyer | **Medium** | 3–6% | Medium | B2B e-commerce (pay-then-invoice), SaaS/digital subscriptions, COD equivalents; growing but still minority vs. traditional order-invoice-pay |
| **19b** | Self-billing (autofacturation) | **High** *(sector)* / **Medium** *(cross-market)* | 2–5% | Medium | Deeply entrenched in automotive OEM (VMI), grande distribution (scan-based trading), agriculture cooperatives, energy; must carry "Autofacturation" mention |
| **36** | Professional secrecy / sensitive data | **Medium** | 2–5% | Medium | 900K+ regulated liberal professionals (lawyers, doctors, notaries, accountants); data-handling constraint (generic Flow 1 descriptions) rather than structural difference; secrecy violations carry criminal penalties |
| **12** | Transparent intermediary on buyer side | **Medium** | 2–4% | Medium | Property management (gestionnaires immobiliers), corporate SSCs, outsourced AP; requires dedicated Annuaire address for intermediary |
| **7** | Purchase paid with lodge/purchasing card | **Medium** | 2–4% | Low-Medium | Lodge cards standard in GE/ETI for corporate travel (SNCF, airlines, hotels); lower p-card penetration than UK/US; reconciliation complexity between invoice data and card settlement |
| **13** | Subcontracting with direct payment | **Medium-High** *(BTP)* / **Low-Medium** *(cross-market)* | 1–3% | Low | Mandatory in public procurement when subcontract >€600 TTC (Loi 75-1334); BTP ~€157B sector; ~30% of construction turnover involves subcontracting |
| **22b** | Early payment discount – goods / accrual VAT | **Medium** | 1–3% | Low | Goods + services under option pour les débits; growing via supply chain finance programs; credit note required; more common than 22a |
| **40** | Netting / offsetting | **Medium** | 1–3% | Low-Medium | Legally automatic in France (Art. 1347 Code civil); standard in corporate groups and bilateral trading (automotive OEMs ↔ tier-1 suppliers); two invoices still issued — only settlement method changes |
| **17a** | Payment intermediary (marketplace) | **Medium-High** | 1–3% | Low | B2B marketplaces growing ~15%/year; Amazon Business, Manutan, sector platforms; DGFiP priority for tax transparency; current volume modest but steep growth trajectory |
| **15** | Third-party order/payment (central purchasing) | **Medium** | 1–2% | Low | UGAP, RESAH, group purchasing bodies; three-party routing adds complexity; mandate relationship must be documented |
| **19a** | Invoice under billing mandate | **Medium** | 1–2% | Low | Outsourced billing providers, SSCs for large groups; mandate must be established in writing prior to invoice; implicit mandate allowed for <10 invoices/year |
| **4** | Invoice partially covered by third party | **Low-Medium** | 1–3% | Low | Insurance-backed B2B, government subsidies, co-payments; less common than full third-party payment (UC3); split settlement and allocation complexity |
| **9** | Distributor/depositary model | **Low-Medium** | 1–3% | Low | Pharma distribution (OCP, Alliance Healthcare), book distribution (Hachette, Interforum), wine cooperatives; sector-specific; technically treated like UC8 |
| **26** | Contractual reservation clause (retenue de garantie) | **Medium-High** *(BTP)* | 1–2% | Medium | Standard 5% retention in construction contracts for 12–13 months; invoice for full amount, payment split; lifecycle status management (95% paid, 5% pending/released); 381K+ BTP companies |
| **33** | Margin VAT scheme | **Medium** | 0.5–1.5% | Medium | Used vehicles (~2.5M professional B2B sales/year), travel agencies, art/antiques; must NOT show VAT amount separately — PA must support format variant |
| **25** | Vouchers and gift cards | **Medium** | 0.5–1.5% | Medium | €8B+ French gift card market (2024); 68% B2B (HR, CSE, employee benefits); only single-purpose (BUU) trigger e-invoicing at transfer; most B2B gifts are multi-purpose (BUM) = out of scope |
| **17b** | Payment intermediary + billing mandate | **Medium** | 0.5–1.5% | Low | Subset of 17a where platform also issues invoice under mandate (e.g., Amazon Marketplace configurations); fewer platforms use this combined model |
| **22a** | Early payment discount – cash accounting VAT (services) | **Medium** | 1–2% | Low | Services with TVA sur encaissements + early payment discount; VAT base reduced by discount; credit note required; narrower than 22b due to cash-VAT specificity |
| **30** | VAT already collected / invoice issued afterwards | **Medium** | 0.5–1% | Low | B2C transaction (e-reported) + later request for formal B2B invoice; data reconciliation challenge; hospitality, retail, professional services at point of sale |
| **39** | Multi-seller | **Low-Medium** | 0.5–2% | Low | Water bills (syndicats des eaux), toll subscriptions, travel agencies bundling services; per-seller tax treatment must be preserved in composite invoice |
| **27** | Toll tickets | **Medium** | 0.2–0.5% | Medium | 7M+ Ulys badge users; B2B leg = corporate fleet télépéage accounts generating monthly invoices; cash/card toll = B2C e-reporting (not B2B invoice) |
| **28** | Restaurant receipts | **Medium** | 0.3–0.8% | Medium | #1 expense note category (42% of expense reports); e-invoicing only when buyer requests formal invoice OR amount >€150 HT; real-time SIREN capture at table = operational pain |
| **16** | Disbursements (débours) | **Low** | <0.5% | Medium | Lawyers, notaries, architects advancing costs on behalf of clients; excluded from VAT base when conditions met; niche by nature |
| **14** | B2B co-contracting / joint contracting | **Low** | <0.5% | Low | Groupement momentané d'entreprises (GME) in BTP/public procurement; project-based, low transaction volume; each co-contractor invoices their portion |
| **18** | Debit note | **Low** | <0.5% | Medium | **Partially out of scope**: DGFiP confirmed true debit notes excluded from e-invoicing; only debit notes constituting VAT invoices fall within scope |
| **29** | VAT group / single taxable person (assujetti unique) | **Medium-High** *(large groups)* | Volume **reducer** (removes 2–5%) | Low | Art. 256 C regime (since Jan 2023); intra-AU transactions outside VAT scope = outside e-invoicing; banking/insurance primary adopters; few AUs created but each removes millions of intra-group invoices |
| **23** | Self-billing (individual/professional) | **Low** | <0.1% | Medium | Narrow scope: individual selling solar electricity to EDF, agricultural individual sales; most solar installations <3 kWc = excluded |
| **37** | Unincorporated joint venture (société en participation) | **Low** | <0.1% | Medium | No legal personality, not RCS-registered, no SIREN; SEP cannot invoice — members invoice individually; very rare legal form |
| **42** | Tax refund / duty-free (détaxe) | **Low** | <0.1% B2B | High | Primarily B2C retail mechanism; handled by Global Blue, Planet; B2B touchpoint is corrective invoicing only after customs validation |
| **35** | Author's statements (notes de droits d'auteur) | **Low** | <0.05% | Medium | ~300K registered artist-authors; few documents each/year; specific VAT rates (5.5%/10%); publishing, music, audiovisual only |
| **41** | Barter | **Low** | <0.05% | Medium | French market explicitly underdeveloped; ~1,500 companies, ~100 exchanges/month nationally; advertising, luxury, events; still generates two invoices each |
| **6** | Employee-paid expenses without invoice | **Low** | ~0% (out of scope) | High | Only a receipt, no formal invoice; falls outside e-invoicing perimeter entirely; may trigger B2C e-reporting from vendor side |
| **24** | Earnest money / arrhes | **Low** | ~0% (out of scope) | High | No VAT due on arrhes (not consideration) = outside e-invoicing AND e-reporting scope; use case exists to prevent misclassification as acompte (UC20) |

---

## Priority Tiers Summary

### Tier 1 — Must-Have at Go-Live (Sept 2026)
These affect the largest share of invoices and/or are legally mandatory paths:

| # | Name | Est. Volume |
|---|------|------------|
| 1 | Multi-order / Multi-delivery | 25–35% |
| Interco | Intercompany invoices | 15–25% |
| 34 | Partial collection / payment lifecycle | 8–15% |
| 3 | Third-party payer (known) | 8–12% |
| 8 | Factoring / designated third-party payee | 8–12% |
| 20 | Advance payment invoice | 5–10% |
| 21 | Final invoice after advance | 5–10% |
| 32 | Monthly / recurring billing | 5–10% |
| 5 | Employee expenses with invoice | 5–8% |
| 31 | Mixed invoice | 3–6% |

### Tier 2 — High Impact in Specific Contexts
High priority within their sectors; meaningful cross-market volume:

| # | Name | Est. Volume |
|---|------|------------|
| 38 | Sub-lines and line grouping | 10–20% |
| 11 | Invoice routed to third party for buyer | 5–8% |
| 19b | Self-billing | 2–5% |
| 36 | Professional secrecy data handling | 2–5% |
| 13 | Subcontracting with direct payment (BTP) | 1–3% |
| 17a | Payment intermediary / marketplace | 1–3% |
| 26 | Reservation clause / retenue de garantie (BTP) | 1–2% |
| 10 | Beneficiary unknown / confidential factoring | 3–5% |

### Tier 3 — Standard Support Required
Moderate frequency; well-defined scenarios:

| # | Name | Est. Volume |
|---|------|------------|
| 2 | Invoice already paid | 3–6% |
| 7 | Lodge/purchasing card | 2–4% |
| 12 | Transparent intermediary (buyer) | 2–4% |
| 22b | Early payment discount (goods/accrual) | 1–3% |
| 40 | Netting / offsetting | 1–3% |
| 33 | Margin VAT scheme | 0.5–1.5% |
| 25 | Vouchers/gift cards (BUU only) | 0.5–1.5% |
| 29 | VAT group / assujetti unique | Volume reducer |

### Tier 4 — Niche / Low Volume
Address when a specific customer segment requires it:

| # | Name | Est. Volume |
|---|------|------------|
| 4 | Partial third-party coverage | 1–3% |
| 9 | Distributor / depositary model | 1–3% |
| 15 | Central purchasing / third-party order | 1–2% |
| 19a | Billing mandate | 1–2% |
| 22a | Early payment discount (cash VAT) | 1–2% |
| 17b | Payment intermediary + mandate | 0.5–1.5% |
| 28 | Restaurant receipts | 0.3–0.8% |
| 30 | VAT already collected, invoice later | 0.5–1% |
| 27 | Toll tickets | 0.2–0.5% |
| 39 | Multi-seller | 0.5–2% |

### Tier 5 — Monitor / Educate (Effectively Out of Scope or Negligible)
| # | Name | Notes |
|---|------|-------|
| 6 | Expenses without invoice | Out of scope for e-invoicing |
| 24 | Earnest money (arrhes) | Out of scope — no VAT, no invoice |
| 18 | Debit note | Mostly out of scope (DGFiP confirmed) |
| 16 | Disbursements | Niche; liberal professions only |
| 14 | Co-contracting / GME | Low volume; BTP/public only |
| 23 | Self-billing (individual/professional) | <0.1%; solar, agriculture |
| 37 | Unincorporated joint venture | <0.1%; no SIREN, rare form |
| 42 | Tax refund / détaxe | B2C-primary; B2B touchpoint minimal |
| 35 | Author's statements | <0.05%; publishing/arts only |
| 41 | Barter | <0.05%; France market underdeveloped |

---

## Key Observations

1. **UC1 is the dominant baseline complexity** — a quarter to a third of all B2B invoices cover multiple orders/deliveries. Any PA or ERP integration must handle this by default.

2. **France's factoring market is enormous** — UC8 + UC10 together likely represent 10–15% of invoices. The Annuaire designation of third-party payees is operationally critical.

3. **Interco is the sleeper issue** — not an official DGFiP case but arguably the highest-impact scenario for Wave 1 customers (GE/ETI), who by definition are multi-entity.

4. **Service sector = UC34 everywhere** — TVA sur encaissements is the default for services (75% of GDP). Partial payment lifecycle status is not optional.

5. **UC20/21 are legally mandatory, not optional** — advance invoicing is required by CGI whenever a deposit is received. Every BTP, consulting, and capital goods customer hits this.

6. **UC38 is a data problem, not a business problem** — sub-line and grouping support is an ERP output issue; its effective volume impact (10–20%) is often underestimated.

7. **Arrhes (UC24) ≠ acompte (UC20)** — the use case exists almost entirely to prevent misclassification. Zero invoice volume of its own.
