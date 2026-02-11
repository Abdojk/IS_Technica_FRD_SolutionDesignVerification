# FRD01 - Sales - Solution Design Verification Report

**Document:** FRD01 - Sales - V2.0
**Prepared by:** Nermine Boulos | Contributor: Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Sales (CRM) with F&O Integration

---

## Executive Summary

The FRD covers Technica's sales cycle across three core processes: **Trips & Missions (SP001)**, **Leads (SP002)**, and **Opportunities (SP003)**, plus supporting areas (Accounts, Contacts, Competitors, Reporting, Duplicate Detection, Outlook Sync). The solution leverages D365 Sales (CE) as the front-end with integrations to D365 F&O for quotations and expense management.

Overall, the solution design is **reasonable and functional**, but there are areas where efficiency, maintainability, and cost can be improved.

---

## Process-by-Process Review

### SP001 - Trips & Missions (GAP - Custom Entity)

**What was proposed:**
- Custom entity in D365 CE for trip/mission requests
- Approval workflow via email (approve/reject buttons)
- On approval: HR notification + integration to F&O as a Travel Requisition draft
- On rejection: cancel all related tasks/appointments, allow resubmission

**Assessment: Functional but could be simplified**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Custom entity approach | Acceptable | No OOB entity exists for this, so customization is justified |
| Email-based approval | Good | Power Automate Approvals handles this well |
| F&O integration as Travel Requisition | Needs Review | See recommendation below |
| Activity tracking (tasks/appointments) | Good | Standard D365 CE activity model is appropriate |

**Recommendations:**

1. **Consider Power Apps + Power Automate instead of a full CE custom entity.** If trips & missions are primarily an internal approval process (not deeply tied to sales pipeline data), a Model-Driven Power App or even a Canvas App with Dataverse would be lighter, cheaper, and faster to build. This avoids cluttering the Sales app with non-sales entities.

2. **F&O Travel Requisition integration may be over-engineered.** The FRD states that upon CRM approval, a draft Travel Requisition is created in F&O, and the user must then go to F&O to fill in expense categories, dates, and amounts. This creates a **two-system data entry problem**. A more efficient approach:
   - **Option A:** Skip CRM entirely for the financial side. Use F&O Travel Requisition with its own workflow from the start. CRM only tracks the sales activity/visit aspect.
   - **Option B:** If CRM must be the entry point, populate the F&O Travel Requisition with as much data as possible (expense categories, estimated amounts) from CRM to minimize duplicate entry.

3. **The "cancel all tasks/appointments on rejection" logic** should be carefully implemented. Consider setting them to "Cancelled" status rather than deleting, to preserve audit trail.

---

### SP002 - Leads (GAP - Approval for Qualification)

**What was proposed:**
- Standard lead creation (manual or import)
- Custom approval cycle before lead qualification
- Sales Manager approves/rejects via email notification
- Approved = qualify into opportunity; Rejected = disqualify
- Restrict qualify/disqualify buttons based on approval status

**Assessment: Works, but adds friction to a standard process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Lead creation & import | Standard OOB | No issues |
| Custom approval for qualification | Functional | But adds overhead - see below |
| Email notification to manager | Good | Power Automate handles this well |
| Button restriction (qualify/disqualify) | Acceptable | Requires JavaScript or Business Rule customization |

**Recommendations:**

1. **Question the business need for qualification approval.** In most D365 Sales implementations, lead qualification is left to the sales rep's judgment with managerial oversight via dashboards/reports. Adding a mandatory approval step:
   - Slows down the sales cycle (the 9-18 month cycle mentioned in the doc)
   - Creates a bottleneck if the manager is unavailable
   - May frustrate sales engineers

   **Better alternative:** Use a **Business Process Flow (BPF) with a gate/stage** that requires manager sign-off as a recommended step rather than a hard block. Combine with **Sales Insights** or **Assistant cards** to flag leads that need attention. This gives oversight without blocking the process.

2. **If approval is truly mandatory**, consider using **Power Automate Approvals** with the built-in Approval entity rather than custom status fields. This provides:
   - Native approval history tracking
   - Mobile approval capability (Teams, Outlook)
   - Approval delegation when managers are unavailable
   - Audit trail out of the box

3. **Lead scoring** (if available in the license) could automate some of this. High-scoring leads could auto-qualify, while low-scoring ones require manual approval. This is a more modern, data-driven approach.

---

### SP003 - Opportunities (GAP - DFI Entity + F&O Integration)

**What was proposed:**
- Standard opportunity creation from qualified leads
- Opportunity stages tracked through pipeline
- Custom DFI (Data Fulfillment Information) entity linked to opportunity
- DFI supports multiple versions (duplicate & modify approach)
- DFI approval triggers opportunity integration to F&O
- F&O generates the quotation (not CRM)

**Assessment: Most complex area - DFI design needs attention**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Opportunity management | Standard OOB | Well-designed |
| Product catalog integration | Good | Standard approach |
| Forecasting & pipeline | Good | OOB capabilities used appropriately |
| DFI custom entity | Needs Improvement | Significant customization with alternatives available |
| DFI versioning via duplication | Risky | See below |
| F&O integration | Needs Architecture | Integration method not specified |

**Recommendations:**

1. **DFI versioning by record duplication is fragile.** Duplicating records and incrementing version numbers introduces risks:
   - Data integrity issues if duplication logic misses fields
   - Grid becomes cluttered with many versions
   - No native "diff" between versions

   **Better alternatives:**
   - **Option A: Use a single DFI record with an audit history** (enable Auditing on the entity). Users modify the same record, and the full change history is tracked automatically. This is simpler and leverages OOB auditing.
   - **Option B: Use SharePoint document versioning** if the DFI is primarily a document. Store the DFI as a Word/Excel template in SharePoint (integrated with D365), and use SharePoint's native version control.
   - **Option C: If versions must be separate records**, use a parent-child model: one "DFI Header" record per opportunity with multiple "DFI Version" child records. Add a "Current Version" flag for clarity.

2. **DFI as a Power Platform form instead of a CE entity.** Since the DFI is described as a "requirement gathering form" with 4 sections (Strategic, Customer Requirements, Technical Data, Handlift), consider:
   - A **Canvas App embedded in the Opportunity form** for DFI data entry
   - Or a **custom page** (modern approach in D365 CE) for a richer form experience
   - This keeps the DFI tightly coupled with the opportunity without a full custom entity

3. **CRM-to-F&O integration architecture is undefined.** The FRD mentions integration multiple times but never specifies the mechanism. This is critical to define early:
   - **Dual-Write** (real-time sync, best for tightly coupled data like Accounts/Contacts)
   - **Virtual Entities** (read F&O data in CRM without copying it)
   - **Power Automate / Logic Apps** (event-driven, good for approval-triggered actions like DFI → F&O)
   - **Custom Integration via Data Integrator** (batch scenarios)

   **Recommendation:** For the DFI-approval-triggers-F&O-integration scenario, **Power Automate** is the most appropriate. For Account/Contact sync, **Dual-Write** is standard. Define this architecture explicitly.

---

## Supporting Areas

### Accounts & Contacts
- **Standard OOB** - No issues. Parent-child company hierarchy is mentioned and addressed.
- **Note:** The approval comments mention that billing goes to parent company while delivery goes to child company. Ensure this is mapped properly in F&O (customer hierarchy / invoice accounts).

### Competitors
- **Standard OOB** - Correctly identified as not a GAP. Multi-competitor per opportunity is supported natively.

### Reporting, Charts & Dashboards
- **OOB with customization** - Acceptable approach.
- **Recommendation:** Define specific report requirements upfront rather than leaving it to "Technica will specify later." Undefined reporting scope often leads to scope creep.

### Duplicate Detection
- **Standard OOB** - Rules should be configured for all key entities (Accounts, Contacts, Leads) during setup.

### Outlook Synchronization
- **Standard OOB** - Server-side sync. No issues.
- **Tip:** Ensure mailbox configuration and approval is part of the deployment plan, as it requires admin setup per user.

---

## Summary of Recommendations

| # | Area | Recommendation | Impact |
|---|------|---------------|--------|
| 1 | SP001 | Consider Power Apps for Trips & Missions instead of full CE custom entity | Reduces cost & complexity |
| 2 | SP001 | Avoid two-system data entry for Travel Requisition; pre-populate F&O from CRM or keep financial entry in F&O only | Improves user experience |
| 3 | SP002 | Replace mandatory approval gate with BPF stage + oversight dashboards | Speeds up sales cycle |
| 4 | SP002 | If approval is kept, use Power Automate Approvals (native) instead of custom status fields | Better audit trail, mobile support |
| 5 | SP003 | Replace DFI version duplication with audit history or SharePoint versioning | Reduces data integrity risk |
| 6 | SP003 | Define CRM↔F&O integration architecture explicitly (Dual-Write, Power Automate, etc.) | Critical for project success |
| 7 | Reporting | Define specific reports in scope now, not later | Prevents scope creep |

---

## Risk Flags

1. **Integration architecture is the biggest risk.** Multiple processes depend on CRM↔F&O integration (Trips→Travel Requisition, Opportunity→F&O Project/Quotation, Won status sync back). Without a defined integration pattern, each developer may implement differently, leading to maintenance issues.

2. **Customization volume is high for Sales module.** Three GAPs (Trips entity, Lead approval, DFI entity) plus integrations means significant development. Ensure this is reflected in the project timeline and budget.

3. **The DFI entity is the most complex customization.** Versioning, approval, and integration trigger logic make this a high-risk deliverable. Recommend a technical spike/POC early in the project.

---

*This review is based on D365 Sales (CE) and F&O best practices as of 2026. Recommendations aim to reduce customization, improve user experience, and leverage platform capabilities.*
