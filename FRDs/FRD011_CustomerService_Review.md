# FRD011 - Customer Service - Solution Design Verification Report

**Document:** FRD011 - Customer Service - V3.0
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Customer Service (CE/CRM) with F&O Integration

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | CS001 - Claims | F&O auto quotation on case resolution (CS001-011) is the most complex GAP — requires clear data mapping and project reference | [CS001 - Claims Registration](#cs001---claims-registration-5-gaps) |
| 2 | **CRITICAL** | CS001/CS002/AMC | Timesheet CRM-to-F&O integration is undefined — affects Claims, Snags, and AMC; must be designed once and reused | [Risk Flags](#risk-flags) |
| 3 | **HIGH** | CS001/CS002 | Shared timesheet entity needed in Dataverse — build once, reuse across Claims and Snags | [CS001 - Claims Registration](#cs001---claims-registration-5-gaps) |
| 4 | **HIGH** | CS002 - Snags | Claims and Snags should use the same Case entity with different Case Types — not separate entities | [CS002 - Snag Registration](#cs002---snag-registration-2-gaps) |
| 5 | **HIGH** | CS002/FRD05 | SNAG tracking exists in both CRM (cases) and F&O (WBS) with no linkage — define which system is master | [CS002 - Snag Registration](#cs002---snag-registration-2-gaps) |
| 6 | **HIGH** | AMC | AMC process is split between CRM and F&O — cross-system handoff needs clear definition | [Entitlements / AMC](#entitlements--amc-annual-maintenance-contracts) |
| 7 | **HIGH** | Mobile | Mobile access for on-site engineers is mentioned but not architecturally defined | [Risk Flags](#risk-flags) |
| 8 | **MEDIUM** | CS001 | Warranty tracking (CS001-010) — use OOB Entitlements instead of building custom warranty entity | [CS001 - Claims Registration](#cs001---claims-registration-5-gaps) |
| 9 | **MEDIUM** | CS001 | Simplify F&O quotation creation to notification + manual instead of full auto-creation | [CS001 - Claims Registration](#cs001---claims-registration-5-gaps) |
| 10 | **MEDIUM** | CS002 | Snag category distinction needed — pre-shipping vs. installation vs. commissioning | [CS002 - Snag Registration](#cs002---snag-registration-2-gaps) |
| 11 | **MEDIUM** | AMC | AMC project structure in F&O needs definition — project group, type, billing rules | [Entitlements / AMC](#entitlements--amc-annual-maintenance-contracts) |
| 12 | **LOW** | CS001/CS002 | Task assignment email notifications — standard Power Automate, low effort | [CS001 - Claims Registration](#cs001---claims-registration-5-gaps) |
| 13 | **LOW** | CS003 - Surveys | Automated survey sending — use Customer Voice with Power Automate | [CS003 - Surveys](#cs003---surveys-1-gap) |
| 14 | **LOW** | Knowledge Articles | Enable Knowledge Search in the case form for agent productivity | [Knowledge Articles](#knowledge-articles-all-oob) |

**Totals:** 2 CRITICAL | 5 HIGH | 4 MEDIUM | 3 LOW

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
| Timesheet registration | **>> NEEDS REVIEW <<** | Custom entity required — see GAP CS001-008 |
| F&O quotation on resolution | **>> NEEDS ARCHITECTURE <<** | Most complex integration — see GAP CS001-011 |

**GAP Analysis:**

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|-----------|----------------|
| CS001-005 | Email notification on task assignment | Low | **Power Automate** flow triggered when a task is created/assigned. This is a very common automation — 1-2 hours to build. Could also use **D365 Workflow** for case assignment notifications. |
| CS001-008 | Register timesheets in CRM | Medium | Two options: **(A)** Custom entity "Timesheet Entry" in Dataverse linked to task/case/project. Simple form with date, hours, overtime flag. **(B)** Use **Power Apps** component model for a timesheet entry experience embedded in the case form. Option A is faster. |
| CS001-009 | Overtime calculation and setup | Medium | Add "Overtime Hours" field to the timesheet entity. Calculation logic (e.g., hours > 8 = overtime) can be handled via **Business Rule** or **Power Automate**. Roll-up fields on the case to show total hours and overtime totals. |
| CS001-010 | Warranty registration and validation | Medium | Use **Entitlements** (already mentioned in the FRD) with SLA terms for warranty periods. When a case is created, auto-check if the customer/product has an active entitlement. If expired → flag the case. This is partially OOB — entitlements support this pattern. |
| CS001-011 | Auto quotation in F&O on case resolution | High | This is the most complex GAP. On case resolution → trigger **Power Automate** flow → create Project Quotation in F&O via OData API with parts and time cost from the case. Requires: (1) clear data mapping between CRM case items and F&O quotation lines, (2) project reference from the case to F&O project. |

**>> ATTENTION AREA: F&O auto quotation integration is the most complex GAP (CS001-011)**

> On case resolution, a Project Quotation must be created in F&O with parts and time cost data from the CRM case. This requires clear data mapping between CRM case items and F&O quotation lines, plus a project reference. The integration mechanism must be explicitly defined before development.

**>> ATTENTION AREA: Timesheet entity design impacts multiple processes (CS001-008/CS001-009)**

> The timesheet entity is needed by both Claims (CS001) and Snags (CS002), plus AMC. It must be designed as a shared, reusable component from the start — not duplicated per process.

**Recommendations:**

1. **>> ENHANCEMENT:** CS001-010 (Warranty) — Don't build a custom warranty entity. Use D365 Customer Service **Entitlements** with term-based tracking. Configure entitlements per customer/product with start/end dates. The entitlement check on case creation is standard behavior when "Decrease Remaining On" is configured.

2. **>> ENHANCEMENT:** CS001-011 (Auto quotation in F&O) — This is a significant integration. Design it carefully:
   - The case must reference a F&O project ID (already stated — projects integrated from F&O)
   - On resolution: collect all task timesheets (hours + cost) and parts used
   - Create quotation in F&O with these as quotation lines
   - **Alternative:** Instead of auto-creating, send a summary notification to Sales Support with a link to create the quotation manually. This reduces integration complexity and gives Sales Support control over pricing.

3. **>> ENHANCEMENT:** Timesheet CRM → F&O push is referenced in both CS001 and CS002. This is a shared integration pattern — build it once and reuse. The timesheet integration should map CRM timesheet entries to F&O project timesheet lines with: Worker, Project ID, Activity, Hours, Date, Overtime flag.

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
| Timesheet | **>> NEEDS REVIEW <<** | GAP (same as CS001) — reuse CS001 solution |

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| CS002-005 | Email notification on assignment | Same as CS001-005 — reuse the same Power Automate flow |
| CS002-008 | Register timesheets | Same as CS001-008 — reuse the same timesheet entity |

**>> ATTENTION AREA: Claims and Snags should share the same Case entity**

> Using separate entities for Claims and Snags would lead to duplication of configuration, flows, and reporting. The standard D365 approach is a single Case entity with different Case Types, enabling unified reporting across both processes.

**>> ATTENTION AREA: SNAG tracking exists in two systems without linkage**

> Customer Service SNAGs are tracked as cases in CRM, while Production SNAGs (FRD05) are tracked in the WBS in F&O. These are different tracking mechanisms for the same concept. Without a clear bridge, the same snag could be tracked in two systems with no linkage.

**Recommendations:**

1. **>> ENHANCEMENT:** Claims vs. Snags should be the same Case entity with different Case Types (not separate entities). This is the standard D365 approach and enables unified reporting across both. The BPF can be different per case type if needed, but the underlying entity should be shared.

2. **>> ENHANCEMENT:** Snags before shipping = revisions (mentioned in the FRD). Ensure this category is distinct from post-installation snags for reporting. Use a "Snag Category" field (Pre-shipping/Installation/Commissioning) for filtering.

3. **>> ENHANCEMENT:** Cross-FRD link with FRD05 (Production) — Production SNAGs are tracked in the WBS with SNAG categories. Customer Service SNAGs are tracked as cases in CRM. Consider whether production SNAGs should also create cases in CRM for unified visibility, or if they remain WBS-only. Define which system is the master for customer-facing snags vs. internal production snags.

---

### CS003 - Surveys (1 GAP)

**What was proposed:**
- Customer satisfaction surveys using Voice of the Customer / Customer Voice
- Automated sending upon case resolution

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| CS003-002 | Automated survey sending | **Power Automate** flow triggered on case resolution → send Dynamics 365 Customer Voice survey. This is a standard pattern — Microsoft documents this integration. ~2 hours to build. |

**>> ENHANCEMENT:** Use **Dynamics 365 Customer Voice** (replacement for Voice of the Customer) which has built-in Power Automate connectors for automated sending. This is the modern approach and is included in the Customer Service license.

---

### Knowledge Articles (All OOB)

**Assessment: Standard D365 Customer Service feature — no issues.**

The BPF for knowledge articles (New → Author → Review → Publish) is OOB and appropriate. This enables the CS team to build a self-service knowledge base over time.

**>> ENHANCEMENT:** Consider enabling **Knowledge Search** in the case form so agents can search for relevant articles while working on a case. This is an OOB feature that improves resolution time.

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
| AMC as F&O project | **>> NEEDS ARCHITECTURE <<** | Cross-system approach needs design |

**>> ATTENTION AREA: AMC process is split between CRM and F&O**

> AMC scheduling and case management happens in CRM, while project accounting and billing happens in F&O. The handoff between systems needs clear definition — who creates what, when, and how they stay in sync.

**Recommendations:**

1. **>> ENHANCEMENT:** AMC as "mini projects" in F&O is a practical approach for accounting. Ensure:
   - AMC projects use a specific project group (e.g., "Service/AMC")
   - Project type is **Time & Material** (for billing actual time/parts)
   - The CRM entitlement is linked to the F&O project via a reference field

2. **>> ENHANCEMENT:** Scheduled AMC activities — The FRD mentions activities scheduled ahead of time. Use **Recurring Activities** or **Power Automate recurrence** to auto-create scheduled service visits based on the AMC contract terms. This ensures proactive scheduling rather than reactive.

3. **>> ATTENTION AREA: Mobile timesheet entry for on-site engineers is undefined**

> The reviewer comment about site leader timesheets (DSR) is important. Daily Site Reports need a clear process. If site leaders are on-site and filling timesheets on mobile, the architecture must be decided.

**>> ENHANCEMENT:** For mobile timesheet entry, decide between: **D365 Mobile App**, **Canvas App** for mobile timesheet entry, or custom mobile solution. The "travel day" flag on timesheets is critical for payroll integration — ensure it's included in the CRM→F&O timesheet push.

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

1. **>> ATTENTION AREA: Timesheet CRM-to-F&O integration is the most critical integration**

> This integration affects Claims, Snags, and AMC. Without a single, well-designed integration pattern, each process may implement its own version, leading to maintenance issues.

**>> ENHANCEMENT:** Design the timesheet integration once with a clean data model (Worker, Project, Activity, Hours, Date, Overtime, Travel flag) and reuse everywhere.

2. **>> ATTENTION AREA: SNAG tracking exists in both CRM (cases) and F&O (WBS activities from FRD05)**

> Without a clear bridge, the same snag could be tracked in two different systems with no linkage.

**>> ENHANCEMENT:** Define which system is the master for customer-facing snags vs. internal production snags.

3. **>> ATTENTION AREA: AMC process is split between CRM (cases, scheduling) and F&O (project, billing)**

> The handoff between systems needs clear definition — who creates what, when, and how they stay in sync.

4. **>> ATTENTION AREA: Mobile access for on-site engineers is not architecturally defined**

> Mobile timesheets are mentioned but the implementation approach is undefined. This affects usability for field engineers.

**>> ENHANCEMENT:** Decide between: D365 Mobile App, Power Apps Canvas App, or custom mobile solution. This decision should be made early as it affects the timesheet entity design.

---

## Efficiency Verdict

The Customer Service solution is **practical and uses D365 Customer Service appropriately**. The case management model for both Claims and Snags is standard and efficient. The main improvement opportunities are:

- **Reuse, not duplicate:** Build one timesheet entity, one notification flow, one case entity with different types
- **Entitlements for warranties:** Don't reinvent — the OOB entitlement system handles this
- **Simplify the F&O quotation auto-creation** — a notification-based approach is safer and faster to implement than full integration

---

*This review is based on D365 Customer Service and Customer Voice best practices as of 2026.*
