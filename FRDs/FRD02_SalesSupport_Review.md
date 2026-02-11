# FRD02 - Sales Support - Solution Design Verification Report

**Document:** FRD02 - Sales Support - V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O)

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | SS007 - Quotation Preparation | CAD↔D365 integration is mentioned but not architected — must be defined before development | [SS007 - Quotation Preparation](#ss007---quotation-preparation-3-gaps) |
| 2 | **HIGH** | SS002 - DFI | Cross-system DFI hand-off mechanism undefined — clarify whether full data or reference link is needed | [SS002 - DFI](#ss002---dfi-gap---customization) |
| 3 | **HIGH** | SS005 - Cost & Selling Price | Costing sheet design is foundational — must be finalized before any cost calculations; schedule dedicated workshop | [SS005 - Assembly Cost & Selling Price](#ss005---assembly-cost--selling-price-calculation-all-fit) |
| 4 | **HIGH** | SS000 - Setup | Costing sheet and cost groups must be finalized before go-live — treat as high-priority setup item | [SS000 - Setup](#ss000---setup) |
| 5 | **HIGH** | Cross-FRD Dependency | DFI flows from CRM (FRD01) to F&O (FRD02) — integration mechanism must be consistent across both FRDs | [Risk Flags](#risk-flags) |
| 6 | **MEDIUM** | SS007 - Quotation Preparation | Printout GAPs — use Power BI paginated reports instead of multiple SSRS customizations for easier maintenance | [SS007 - Quotation Preparation](#ss007---quotation-preparation-3-gaps) |
| 7 | **MEDIUM** | SS008 - Resource Time & Efforts | Pre-win time tracking needs internal project structure — create pre-sales project with proper project type | [SS008 - Resource Time & Efforts](#ss008---resource-time--efforts) |
| 8 | **LOW** | SS003 - Engineering Job Order | Deadline GAP is trivial — add custom date field + Power Automate alert | [SS003 - Engineering Job Order](#ss003---engineering-job-order-mostly-fit) |
| 9 | **LOW** | SS005 - Cost & Selling Price | Consider multiple costing versions for what-if scenario comparison | [SS005 - Assembly Cost & Selling Price](#ss005---assembly-cost--selling-price-calculation-all-fit) |
| 10 | **LOW** | SS007 - Quotation Preparation | Justification field GAP — simple custom text field on quotation line | [SS007 - Quotation Preparation](#ss007---quotation-preparation-3-gaps) |

**Totals:** 1 CRITICAL | 4 HIGH | 2 MEDIUM | 3 LOW

---

## Executive Summary

The FRD covers the Sales Support team's role in transforming customer requirements (via DFI from CRM) into costed, quoted solutions using D365 F&O. It spans 8 processes: DFI tracking, Engineering Job Orders, BOM building, cost/price calculation, quotation preparation with WBS, and resource time tracking. The solution makes **strong use of standard D365 F&O capabilities** — particularly Engineering Change Management, BOM costing, and Project Quotations with WBS templates.

Overall assessment: **Well-designed with good use of standard functionality.** A few areas need attention.

---

## Process-by-Process Review

### SS002 - DFI (GAP - Customization)

**What was proposed:**
- Sales Support receives DFI from CRM and builds solutions based on it
- GAP: Ability to track DFI in F&O

**Assessment: Cross-system dependency needs clarity**

| Aspect | Rating | Comment |
|--------|--------|---------|
| DFI tracking in F&O | **>> NEEDS ARCHITECTURE <<** | How will DFI data surface in F&O? |

**>> ATTENTION AREA: DFI hand-off mechanism is undefined**

> The FRD says Sales Support needs to "track DFI" in F&O, but it does not specify whether the full DFI data needs to replicate to F&O or whether Sales Support just needs a reference/link. This cross-system dependency (CRM → F&O) must be clarified before development.

**Recommendations:**

1. **>> ENHANCEMENT:** Clarify the DFI hand-off mechanism. FRD01 (Sales) defines DFI as a custom CRM entity with versions and approvals. This FRD says Sales Support needs to "track DFI" in F&O. The question is: does the full DFI data need to replicate to F&O, or does Sales Support just need a reference/link?
   - **If reference only:** Use a URL field on the Project Quotation linking back to the CRM DFI record. No customization needed.
   - **If full data needed:** Use Dual-Write or Virtual Entities to surface CRM DFI data in F&O without duplicating it.
   - **Avoid:** Recreating the DFI entity in F&O — this creates data duplication and sync issues.

---

### SS003 - Engineering Job Order (Mostly FIT)

**What was proposed:**
- Sales Support creates Engineering Change Requests for new product designs
- R&D reviews and approves via workflow
- Uses D365 Engineering Change Management module

**Assessment: Excellent choice — proper use of standard module**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Engineering Change Request | Standard OOB | Correct module choice |
| Workflow approval by HOD | Standard OOB | Well-designed |
| Stage tracking | Standard OOB | Built-in lifecycle states |
| Deadline on change request (GAP) | **>> NEEDS REVIEW <<** | Simple custom field — minor GAP |

**Recommendations:**

1. **>> ENHANCEMENT:** The deadline GAP (SS003-05) is trivial. Add a custom date field "Deadline" to the Engineering Change Request header. Consider adding an alert/notification via Power Automate when the deadline approaches. This is a 1-hour task, not a significant customization.

2. **This is the right approach.** Engineering Change Management was purpose-built for this exact scenario. No alternative needed.

---

### SS004 - Building BOM (All FIT)

**What was proposed:**
- Items maintain multiple BOM versions for different configurations
- BOM can be updated on production orders
- Track where items are used (Where-Used inquiry)
- Engineering Change Orders for BOM modifications

**Assessment: Standard D365 BOM management — no issues**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Multiple BOM versions | Standard OOB | Well-supported |
| BOM update on production order | Standard OOB | Built-in capability |
| Where-Used tracking | Standard OOB | Standard inquiry |
| Engineering Change Orders | Standard OOB | Part of ECM module |

**Recommendations:**

1. **No changes needed.** This is textbook D365 BOM management. The multi-version approach with activation dates is correct and allows for proper cost tracking over time.

2. **Consider configuring BOM approval workflows** if not already planned. This prevents unauthorized BOM changes that could affect costing.

---

### SS005 - Assembly Cost & Selling Price Calculation (All FIT)

**What was proposed:**
- Dedicated costing version for Sales Support
- Items assigned to cost groups
- Cost calculation routine for assembly cost/selling price
- Cost sheet defines cost structure
- Margin applied per cost group (profit-setting)
- Missing component notifications during calculation

**Assessment: Sophisticated and correct use of D365 costing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Costing version per department | Excellent | Proper isolation of cost scenarios |
| Cost groups for categorization | Excellent | Enables granular margin control |
| Costing sheet structure | Excellent | Defines full cost/price hierarchy |
| Profit-setting per cost group | Excellent | Solves the margin-at-top-level problem elegantly |
| Calculation warnings | Standard OOB | Missing BOM, route, cost alerts |
| Costing sheet design | **>> NEEDS REVIEW <<** | Foundational — must be finalized before go-live |

**>> ATTENTION AREA: Costing sheet design is foundational and must be finalized upfront**

> The costing sheet is the backbone of all price calculations. A poorly designed costing sheet will cascade errors into every quotation. Changes later are disruptive. This must be treated as a high-priority setup item.

**Recommendations:**

1. **This is one of the strongest parts of the solution.** The use of cost groups with profit-settings to apply margins at every BOM level (not just top-level) is the correct D365 approach and is more accurate than manual top-level margin adjustments.

2. **>> ENHANCEMENT:** Consider creating multiple costing versions beyond just "Sales Support" — e.g., "What-If" scenarios for different margin strategies. D365 supports this natively and it allows Sales Support to compare scenarios without affecting the active costing.

3. **>> ENHANCEMENT:** Ensure the costing sheet is designed carefully upfront. It's the backbone of all price calculations. Changes later are disruptive. Recommend a dedicated workshop to finalize the costing sheet structure before go-live.

---

### SS006 - Using Products from Library (FIT)

**What was proposed:**
- If item already exists with cost/price, skip to quotation directly

**Assessment: Logical shortcut — no issues.** Standard product catalog usage.

---

### SS007 - Quotation Preparation (3 GAPs)

**What was proposed:**
- Project Quotation created from CRM Opportunity (without Project ID)
- Import WBS from pre-built templates
- WBS activities contain items, hours, expenses with role assignments
- CAD system integration: WBS sent to CAD → items returned to D365
- Quotation lines generated from WBS
- Workflow approval before sending to customer
- Printout customization (3 GAPs)
- Opportunity Won → Quotation auto-confirmed → Project created

**Assessment: Core process is solid; GAPs are manageable**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Project Quotation from Opportunity | Good | Standard CRM→F&O flow |
| WBS template import | Excellent | Saves significant time, reusable |
| Cost detail in WBS activities | Standard OOB | Items, hours, expenses per activity |
| CAD integration | **>> NEEDS ARCHITECTURE <<** | Not detailed enough — critical integration undefined |
| Generate quotation lines from WBS | Standard OOB | Built-in function |
| Approval workflow | Standard OOB | Standard workflow framework |
| Won/Lost status sync | Good | CRM↔F&O coordination |

**>> ATTENTION AREA: CAD↔D365 integration is mentioned but not architected**

> The FRD says "WBS will be sent to CAD with the related activity number then the CAD will fill the related item and send it back to D365." This is a critical integration for the quotation process and has no architecture specified. The CAD system, data format, and integration mechanism are all undefined.

**GAP Analysis:**

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|-----------|----------------|
| SS005-06 | Justification field on quotation line | Low | Add a custom text field to `ProjQuotationLine`. Simple extension — 2-3 hours of work. |
| SS005-07 | Rearrange labor cost as separate line items on printout | Medium | SSRS report customization. However, consider using **Power BI paginated reports** instead — they're easier to maintain and modify than SSRS, and Technica can adjust layouts without developer help. |
| SS005-08 | Show/Hide installation on printout checkbox | Low | Add a boolean field + conditional section in the report. Straightforward. |

**Recommendations:**

1. **>> ENHANCEMENT (CRITICAL):** CAD↔D365 integration needs a proper integration design:
   - What is the CAD system? (AutoCAD, SolidWorks, etc.)
   - What format? (API, file-based, middleware?)
   - Recommendation: Use **Azure Logic Apps** or **Power Automate** with a shared staging area (e.g., SharePoint, Dataverse table) for the handoff. Define the data contract explicitly.

2. **WBS templates are a great approach** — ensure Technica prepares and migrates all templates during the migration phase as stated. Recommend building a template naming convention and governance process.

3. **The Opportunity Won → Quotation Confirmed → Project Created pipeline is well-designed.** This is the standard D365 Project Operations flow and is efficient.

4. **>> ENHANCEMENT:** For the printout GAPs: Instead of multiple SSRS customizations, consider a single **Power BI paginated report** that handles all printout variations (with/without installation, labor breakdown). This is more future-proof and easier for Technica to maintain.

---

### SS008 - Resource Time & Efforts

**What was proposed:**
- Sales Support records time on timesheets linked to projects/WBS
- Time tracking starts before project is won (during sales phase)

**Assessment: Correct approach, but needs project structure planning**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Timesheet tracking | Standard OOB | No issues |
| Pre-win time tracking | **>> NEEDS REVIEW <<** | Requires internal project structure |

**>> ATTENTION AREA: Pre-win time tracking requires internal project structure**

> Time tracking starts before a customer project exists (during the sales phase). Without a dedicated internal project structure, there is no valid project to charge time against, and pre-sales costs could accidentally flow to customer billing.

**Recommendations:**

1. **>> ENHANCEMENT:** Create a "Pre-Sales" or "Business Development" internal project with dedicated WBS activities for sales support effort. This allows time tracking before a customer project exists. When the opportunity is won, the actual project is created separately.

2. **>> ENHANCEMENT:** Set the project type correctly:
   - Pre-sales project: **Internal** or **Cost** project type (no revenue recognition)
   - Customer project: **Time & Material** or **Fixed-price** project type
   - This ensures pre-sales costs don't accidentally flow to customer billing

3. **Consider project budget controls** on the pre-sales project to prevent unlimited time being charged to sales activities.

---

### SS000 - Setup

**What was proposed:**
- Costing sheet definition
- Cost group configuration
- Engineering product categories
- Product owner groups

**Assessment: All standard setup — no issues**

**>> ATTENTION AREA: Foundational setup items must be completed before dependent processes**

> Costing sheet and cost groups are the foundation for all cost/price calculations (SS005) and quotation preparation (SS007). If these are not finalized before those processes are configured, rework will be required.

**Recommendations:**

1. **>> ENHANCEMENT:** Costing sheet and cost groups must be finalized before any cost calculations can run. These are foundational. Schedule a dedicated setup workshop.

2. **Product owner groups** tied to Engineering Change Management approval is a smart access control mechanism. Ensure the groups are defined to match Technica's organizational structure.

---

## Summary of Recommendations

| # | Area | Recommendation | Impact |
|---|------|---------------|--------|
| 1 | SS002 | Use Virtual Entities or URL link for DFI in F&O instead of data duplication | Eliminates sync issues |
| 2 | SS003 | Deadline GAP is trivial — custom date field + alert | 1-hour task |
| 3 | SS005 | Consider multiple costing versions for scenario comparison | Better pricing decisions |
| 4 | SS007 | Define CAD↔D365 integration architecture | Unblocks development |
| 5 | SS007 | Use Power BI paginated reports instead of SSRS for quotation printouts | Easier maintenance |
| 6 | SS008 | Create internal pre-sales project structure for time tracking before win | Proper cost separation |
| 7 | SS000 | Prioritize costing sheet + cost groups setup workshop | Foundation for all pricing |

---

## Risk Flags

1. **>> ATTENTION AREA: CAD integration is undefined**

> This is a critical integration for the quotation process and has no architecture specified. Development cannot proceed without a defined integration pattern, data contract, and middleware selection.

2. **>> ATTENTION AREA: Cross-FRD dependency on DFI**

> DFI flows from CRM (FRD01) to F&O (FRD02). The integration mechanism must be consistent across both FRDs. If FRD01 and FRD02 teams choose different approaches, maintenance and data integrity will suffer.

3. **>> ATTENTION AREA: Costing sheet design is critical**

> A poorly designed costing sheet will cascade errors into every quotation. This must be treated as a high-priority setup item with a dedicated workshop before go-live.

---

## Efficiency Verdict

The Sales Support solution is **well-designed and makes excellent use of D365 standard capabilities**, particularly:
- Engineering Change Management for new product requests
- BOM costing with cost groups and profit-settings for margin control
- Project Quotation with WBS templates for repeatable quoting

The main gaps are **minor printout customizations** (manageable) and **undefined integrations** (CAD, DFI) that need architecture decisions.

---

*This review is based on D365 F&O and Engineering Change Management best practices as of 2026.*
