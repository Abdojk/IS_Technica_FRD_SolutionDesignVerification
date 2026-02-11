# FRD011 - Customer Service - Solution Design Verification Report

**Document:** FRD011 - Customer Service - V3.0
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Customer Service (CE/CRM) with F&O Integration

---

## Executive Summary

This FRD covers three Customer Service processes: Claims Registration (CS001), Snag Registration (CS002), and Survey Creation (CS003), plus supporting areas (Knowledge Articles, Entitlements/AMC). The solution uses **D365 Customer Service** (CE) with case management, business process flows, and integration to F&O for timesheets and quotations. Most core case management is OOB, with GAPs mainly around timesheet registration, notification automation, warranty tracking, and F&O quotation creation.

Overall assessment: **Good use of D365 Customer Service with practical GAPs that are solvable via Power Automate and standard integrations.**

---

## Process-by-Process Review

### CS001 - Claims Registration (5 GAPs)

**What was proposed:**
- Case creation (manual or auto from email via Automatic Record Creation rules)
- Business Process Flow: Identify → Assessment → Resolution
- Tasks assigned to engineers with email notifications
- Timesheet registration against tasks (pushed to F&O)
- Overtime hours tracking
- Case resolution triggers quotation creation in F&O
- Case can be re-opened if needed
- Integration from Outlook for case creation

**Assessment: Solid case management — GAPs are integration-focused**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Manual case creation | Standard OOB | No issues |
| Auto case from email | Standard OOB | Automatic Record Creation rules |
| Outlook case creation | Standard OOB | D365 App for Outlook |
| Business Process Flow | Standard OOB | 3-stage flow is appropriate |
| Task assignment | Standard OOB | Activity management |
| Timeline control | Standard OOB | Full interaction history |
| Case resolution | Standard OOB | Standard resolve dialog |
| Case re-opening | Standard OOB | Standard reactivation |

**GAP Analysis:**

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|-----------|----------------|
| CS001-005 | Email notification on task assignment | Low | **Power Automate** flow triggered when a task is created/assigned. This is a very common automation — 1-2 hours to build. Could also use **D365 Workflow** for case assignment notifications. |
| CS001-008 | Register timesheets in CRM | Medium | Two options: **(A)** Custom entity "Timesheet Entry" in Dataverse linked to task/case/project. Simple form with date, hours, overtime flag. **(B)** Use **Power Apps** component model for a timesheet entry experience embedded in the case form. Option A is faster. |
| CS001-009 | Overtime calculation and setup | Medium | Add "Overtime Hours" field to the timesheet entity. Calculation logic (e.g., hours > 8 = overtime) can be handled via **Business Rule** or **Power Automate**. Roll-up fields on the case to show total hours and overtime totals. |
| CS001-010 | Warranty registration and validation | Medium | Use **Entitlements** (already mentioned in the FRD) with SLA terms for warranty periods. When a case is created, auto-check if the customer/product has an active entitlement. If expired → flag the case. This is partially OOB — entitlements support this pattern. |
| CS001-011 | Auto quotation in F&O on case resolution | High | This is the most complex GAP. On case resolution → trigger **Power Automate** flow → create Project Quotation in F&O via OData API with parts and time cost from the case. Requires: (1) clear data mapping between CRM case items and F&O quotation lines, (2) project reference from the case to F&O project. |

**Recommendations:**

1. **CS001-010 (Warranty):** Don't build a custom warranty entity. Use D365 Customer Service **Entitlements** with term-based tracking. Configure entitlements per customer/product with start/end dates. The entitlement check on case creation is standard behavior when "Decrease Remaining On" is configured.

2. **CS001-011 (Auto quotation in F&O):** This is a significant integration. Design it carefully:
   - The case must reference a F&O project ID (already stated — projects integrated from F&O)
   - On resolution: collect all task timesheets (hours + cost) and parts used
   - Create quotation in F&O with these as quotation lines
   - **Alternative:** Instead of auto-creating, send a summary notification to Sales Support with a link to create the quotation manually. This reduces integration complexity and gives Sales Support control over pricing.

3. **Timesheet CRM → F&O push** is referenced in both CS001 and CS002. This is a shared integration pattern — build it once and reuse. The timesheet integration should map CRM timesheet entries to F&O project timesheet lines with: Worker, Project ID, Activity, Hours, Date, Overtime flag.

---

### CS002 - Snag Registration (2 GAPs)

**What was proposed:**
- Snags created manually or imported from Excel
- Created by Site Manager during installation/commissioning
- Linked to customer, project, and specific assembly
- Business Process Flow: Identification → Assessment → Resolution
- Project Manager assigned to manage snag
- Tasks created and assigned to engineers
- Timesheet registration (same as claims)
- Case can be reactivated

**Assessment: Same pattern as Claims — reuse the same configuration**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Snag case creation | Standard OOB | Different case type |
| Excel import | Standard OOB | Import from Excel feature |
| BPF (3 stages) | Standard OOB | Same pattern as Claims |
| Task management | Standard OOB | Same as Claims |
| Timesheet | GAP (same as CS001) | Reuse CS001 solution |

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| CS002-005 | Email notification on assignment | Same as CS001-005 — reuse the same Power Automate flow |
| CS002-008 | Register timesheets | Same as CS001-008 — reuse the same timesheet entity |

**Recommendations:**

1. **Claims vs. Snags should be the same Case entity** with different Case Types (not separate entities). This is the standard D365 approach and enables unified reporting across both. The BPF can be different per case type if needed, but the underlying entity should be shared.

2. **Snags before shipping = revisions** (mentioned in the FRD). Ensure this category is distinct from post-installation snags for reporting. Use a "Snag Category" field (Pre-shipping/Installation/Commissioning) for filtering.

3. **Cross-FRD link with FRD05 (Production):** Production SNAGs are tracked in the WBS with SNAG categories. Customer Service SNAGs are tracked as cases in CRM. These are **different tracking mechanisms for the same concept**. Consider whether production SNAGs should also create cases in CRM for unified visibility, or if they remain WBS-only.

---

### CS003 - Surveys (1 GAP)

**What was proposed:**
- Customer satisfaction surveys using Voice of the Customer / Customer Voice
- Automated sending upon case resolution

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| CS003-002 | Automated survey sending | **Power Automate** flow triggered on case resolution → send Dynamics 365 Customer Voice survey. This is a standard pattern — Microsoft documents this integration. ~2 hours to build. |

**Recommendation:** Use **Dynamics 365 Customer Voice** (replacement for Voice of the Customer) which has built-in Power Automate connectors for automated sending. This is the modern approach and is included in the Customer Service license.

---

### Knowledge Articles (All OOB)

**Assessment: Standard D365 Customer Service feature — no issues.**

The BPF for knowledge articles (New → Author → Review → Publish) is OOB and appropriate. This enables the CS team to build a self-service knowledge base over time.

**Recommendation:** Consider enabling **Knowledge Search** in the case form so agents can search for relevant articles while working on a case. This is an OOB feature that improves resolution time.

---

### Entitlements / AMC (Annual Maintenance Contracts)

**What was proposed:**
- Entitlements as maintenance contracts
- Set number of visits/cases per contract
- Activate, deactivate, renew
- Default entitlement per customer
- AMC treated as mini projects in F&O

**Assessment: Good use of Entitlements — the AMC approach needs clarity**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Entitlement creation | Standard OOB | With term-based decrement |
| Default entitlement | Standard OOB | Auto-link to new cases |
| Activate/deactivate/renew | Standard OOB | Lifecycle management |
| AMC as F&O project | Needs Design | Cross-system approach |

**Recommendations:**

1. **AMC as "mini projects" in F&O** is a practical approach for accounting. Ensure:
   - AMC projects use a specific project group (e.g., "Service/AMC")
   - Project type is **Time & Material** (for billing actual time/parts)
   - The CRM entitlement is linked to the F&O project via a reference field

2. **Scheduled AMC activities:** The FRD mentions activities scheduled ahead of time. Use **Recurring Activities** or **Power Automate recurrence** to auto-create scheduled service visits based on the AMC contract terms. This ensures proactive scheduling rather than reactive.

3. **The reviewer comment about site leader timesheets (DSR)** is important. Daily Site Reports need a clear process:
   - If site leaders are on-site and filling timesheets on mobile, use **D365 Mobile** or a **Canvas App** for mobile timesheet entry
   - The "travel day" flag on timesheets is critical for payroll integration — ensure it's included in the CRM→F&O timesheet push

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | CS001/002 | Build Power Automate notification template for task assignment | Low |
| 2 | CS001/002 | Create shared timesheet entity in Dataverse — reuse across Claims and Snags | High |
| 3 | CS001 | Use Entitlements for warranty tracking — don't build custom | Medium |
| 4 | CS001 | Simplify F&O quotation creation to notification + manual, not full auto | Medium |
| 5 | CS002 | Use same Case entity for Claims and Snags (different Case Types) | High |
| 6 | CS003 | Use Customer Voice with Power Automate for survey automation | Low |
| 7 | AMC | Define AMC project structure in F&O (project group, type, billing rules) | Medium |
| 8 | Mobile | Provide mobile timesheet entry for on-site engineers | High |

---

## Risk Flags

1. **Timesheet CRM → F&O integration** is the most critical integration in this FRD. It affects Claims, Snags, and AMC. Design it once with a clean data model (Worker, Project, Activity, Hours, Date, Overtime, Travel flag) and reuse everywhere.

2. **SNAG tracking exists in both CRM (cases) and F&O (WBS activities from FRD05).** Without a clear bridge, the same snag could be tracked in two different systems with no linkage. Define which system is the master for customer-facing snags vs. internal production snags.

3. **AMC process is split between CRM (cases, scheduling) and F&O (project, billing).** The handoff between systems needs clear definition — who creates what, when, and how they stay in sync.

4. **Mobile access for on-site engineers** is mentioned (timesheets from mobile) but not architecturally defined. Decide between: D365 Mobile App, Power Apps Canvas App, or custom mobile solution.

---

## Efficiency Verdict

The Customer Service solution is **practical and uses D365 Customer Service appropriately**. The case management model for both Claims and Snags is standard and efficient. The main improvement opportunities are:

- **Reuse, not duplicate:** Build one timesheet entity, one notification flow, one case entity with different types
- **Entitlements for warranties:** Don't reinvent — the OOB entitlement system handles this
- **Simplify the F&O quotation auto-creation** — a notification-based approach is safer and faster to implement than full integration

---

*This review is based on D365 Customer Service and Customer Voice best practices as of 2026.*
