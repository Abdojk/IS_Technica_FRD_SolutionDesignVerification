# FRD04 - Project Accounting - Solution Design Verification Report

**Document:** FRD04 - Project Accounting - V2.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Project Management & Accounting

---

## Executive Summary

This is the **most comprehensive FRD** in the set, covering the full project lifecycle from opportunity to revenue recognition. It spans four major processes: Project Planning (PJ001), Project Execution (PJ002), Milestone Invoicing (PJ003), and Revenue Estimation (PJ004). The solution makes **excellent use of D365 Project Accounting standard capabilities** — WBS templates, budget control, milestone billing, resource allocation, forecast models, and percentage-of-completion revenue recognition.

Overall assessment: **Strong, well-detailed solution with mostly FIT requirements.** The few GAPs are manageable. One significant custom requirement (cost reallocation to assemblies) needs careful design.

---

## Process-by-Process Review

### PJ001 - Project Planning

**What was proposed:**
- CRM Opportunity → F&O Opportunity → Project Quotation with WBS templates
- Resource allocation with role-based cost/price matrices
- Budget creation from published WBS
- Item requirements populated from WBS
- Case management for change logs, issues, risks
- Sub-projects for pre-win tracking and cost segregation
- MS Project integration for scheduling
- Installation management as WBS section

**Assessment: Comprehensive and well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Opportunity → Quotation flow | Good | Standard with CRM integration |
| WBS template import | Excellent | Reusable, time-saving |
| Resource allocation & pricing | Excellent | Multi-dimensional cost/price setup |
| Budget from WBS | Standard OOB | Proper workflow approval |
| Item requirements from WBS | Standard OOB | With master planning trigger |
| Case management | Creative | Good use of cases for project governance |
| Sub-projects | Well-designed | Pre-win tracking, cost segregation |
| MS Project integration | Standard OOB | Check-in/check-out model |
| Project stages | Standard OOB | With action restrictions per stage |

**Recommendations:**

1. **Opportunity stages in F&O (Sales Process template):** The setup of stages (Qualify, Feasibility, Capex, Comparison, Negotiation) as activities is workable but somewhat manual. Since Technica is already managing opportunities in CRM (FRD01), consider whether this F&O opportunity staging is truly needed or if it creates duplicate tracking. **Recommendation:** Let CRM own the opportunity pipeline stages. F&O should only receive the opportunity once it reaches the quotation stage, reducing duplication.

2. **Resource notification via Custom Alerts** is a good approach but has limitations — custom alerts are basic. For richer notifications (with project details, links, action buttons), consider **Power Automate flows triggered on resource assignment**. This provides better notification content and supports Teams/mobile notifications.

3. **"Confirmed received day" GAP (PJ001-020):** This is a simple custom field on the item requirement form. Low complexity, but ensure it's clear whether this date comes from:
   - PO confirmed delivery date (auto-populated)
   - Voyage estimated arrival (from Transportation Management)
   - Manual entry

   **Recommendation:** Auto-populate from PO confirmed delivery date where a PO exists. This reduces manual entry.

4. **Sub-project for pre-win work is a smart approach.** However, ensure:
   - The pre-win sub-project uses project type **"Internal"** or **"Cost"** (no billing)
   - Clear naming convention distinguishes pre-win sub-projects (e.g., prefix "PRE-")
   - Budget controls prevent excessive pre-win time booking

5. **Case management for project governance** (change log, issue log, risk register, lessons learned) is a creative use of D365 Case Management. However, consider whether a **dedicated project governance tool** (e.g., Power Apps board, SharePoint lists, or Azure DevOps) might provide a better user experience for these specific needs. Cases work, but they're designed for customer service, not project governance.

6. **Financial dimensions on projects:** Mentioned as FIT (PJ001-018). Ensure the dimension structure is defined early — typical setup: Project + Cost Center + Department. This affects all downstream reporting.

---

### PJ002 - Project Execution

**What was proposed:**
- Budget revision workflow (Original → Current Budget)
- Forecast model lifecycle (T_Forecast → NT_Forecast)
- Hour journals, expense journals, timesheets
- PO processing against projects with auto-consumption
- WBS cost and effort tracking views
- Custom reports (Project Planning, Project List, Master Planning)

**Assessment: Solid execution process with good budget control**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Budget revision workflow | Standard OOB | Proper approval chain |
| Forecast model management | Good | Original vs. Current separation |
| Hour/expense journals | Standard OOB | No issues |
| Timesheets with workflow | Standard OOB | Pre-win tracking included |
| PO against projects | Standard OOB | Auto-consumption on receipt |
| Cost tracking view | Standard OOB | With variance analysis |
| Effort tracking view | Standard OOB | With remaining effort calc |

**GAP: Cost Reallocation to Assemblies**

This is the most significant custom requirement in the document:
- Common activity costs (paint, cleaning, etc.) tracked at project level
- At project close, reallocated to assemblies proportionally based on assembly cost %
- Example: 24,000 distributed as 42%/33%/25% across 3 assemblies

**Recommendations:**

1. **Cost reallocation is not OOB in D365 Project Accounting.** Options:
   - **Option A (Recommended): Project adjustment transactions.** Use project adjustment journals to move costs from the common activity to assembly activities based on calculated percentages. This keeps everything within standard project accounting and maintains full audit trail.
   - **Option B: Indirect cost allocation rules** in Cost Accounting module. More sophisticated but adds complexity.
   - **Option C: Custom X++ batch job** that calculates percentages and creates adjustment journals automatically. Best if this happens frequently.

   **Recommendation:** Start with Option A (manual adjustment journals) and automate with Option C if volume warrants it.

2. **Forecast model lifecycle is well thought out** but complex. Create clear documentation/SOPs for the project team on when to create which forecast model. The T_Forecast → NT_Forecast naming suggests only 2 iterations — ensure the naming convention supports multiple revisions (e.g., Forecast_Rev01, Forecast_Rev02).

3. **WBS Tracking Views — additional columns (Original Budget, Current Budget):** These are GAP items (PJ002-009/010 tracking views are FIT, but additional columns are GAP). Recommendation: Use **Power BI embedded in workspace** for richer budget vs. actual views rather than customizing the WBS form. This is easier to maintain and provides drill-through capabilities.

4. **Report GAPs (PJ002-011/012/013):**
   - **Project Planning Report:** Build in Power BI using project WBS data
   - **Project List - Ongoing:** Build in Power BI with filters for project stage
   - **Master Planning Report:** This is the most complex — it needs to consolidate milestones across all projects with notifications. **Recommend Power BI with data-driven alerts** rather than a custom SSRS report. Power BI can send email alerts when milestones approach due dates.

5. **Timesheet from CRM pushed to F&O project:** This is mentioned briefly but is a significant integration point. Define the mechanism (Dual-Write, Power Automate, custom). Ensure timesheet approval workflow still applies to CRM-originated timesheets.

---

### PJ003 - Milestone Invoice Proposal

**What was proposed:**
- Milestones from project contract billing rules
- HOD marks milestone complete → notifies Finance
- Finance creates invoice proposal → posts invoice
- Retention percentage applied per milestone
- Commercial invoice separate from real invoice
- Cash flow reports (Project Cash Flow, Cash Flow Statement, Forecast)

**Assessment: Standard milestone billing — well-implemented**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Milestone creation from contract | Standard OOB | With retention % |
| Mark milestone complete | Standard OOB | Triggers billing |
| Invoice proposal creation | Standard OOB | Standard workflow |
| Cash flow reports | Mostly OOB | Good use of built-in reports |
| Retention handling | Standard OOB | Configurable % per milestone |

**Recommendations:**

1. **On-account invoice schedule modification (PJ003-005):** Adding commercial invoice number and payment status is a reasonable customization. However, instead of modifying the standard report, **create a new custom report** that combines real invoice data with commercial invoice references. This preserves the standard report for future upgrades.

2. **Cash Flow Aging Report (PJ003-009 - GAP):** The Technica Excel format should be replicated as a **Power BI paginated report**. This provides:
   - Automated data refresh
   - Export to Excel/PDF
   - Consistent formatting matching Technica's existing format
   - Drill-through to project details

3. **The commercial invoice concept** (mentioned as handled in Logistics FRD) creates a cross-FRD dependency. Ensure the real invoice ↔ commercial invoice ↔ packing slip linkage is clearly designed end-to-end.

4. **Milestone value updates:** The FRD notes that "contract values do not update automatically" and project team must update manually. This is a risk for revenue recognition accuracy. **Recommendation:** Add a validation check (Power Automate or periodic batch job) that compares milestone totals to contract value and alerts the project team if they diverge.

---

### PJ004 - Project Estimates / Accrued Revenue

**What was proposed:**
- Percentage of Completion (POC%) input — manual or automatic
- Accrued revenue = Contract Value x POC%
- Gross margin = Accrued Revenue - Actual Cost
- Estimate journal created per period (monthly)
- Post estimates to GL
- GAP: Column for commercial vs. real invoice split

**Assessment: Correct POC/WIP approach for fixed-price projects**

| Aspect | Rating | Comment |
|--------|--------|---------|
| POC% calculation | Standard OOB | Manual or auto (cost-based) |
| Accrued revenue calculation | Standard OOB | Contract x POC% |
| Monthly estimate journals | Standard OOB | Period-based |
| Estimate posting | Standard OOB | WIP/accrual entries |

**Recommendations:**

1. **Manual vs. Automatic POC%:** The FRD allows both. **Recommendation:** Default to automatic (actual cost / total forecast cost) and only allow manual override with approval. Manual POC% input is a common source of revenue manipulation risk. If Technica's auditors require manual input, add an approval step.

2. **Estimate form modification (PJ004-003):** Adding commercial vs. real invoice columns is a display-level customization. Consider implementing this as a **Power BI visual embedded in the estimate form** rather than modifying the form directly. This keeps the standard form intact.

3. **Revenue recognition compliance:** Ensure the POC% method complies with IFRS 15 / ASC 606 requirements. D365 supports both **completed contract** and **percentage of completion** methods. Verify with Technica's auditors which method is required.

4. **Multiple forecast models:** The lifecycle (Original Budget → Current Estimate → Current Budget) with different forecast model names is correct but complex. Create a **reference guide** for the project team mapping each forecast model to its purpose and when it's created/updated.

---

## Setup Review

### Project Groups
- Single "Fixed Price" project group — appropriate if all projects are fixed price
- **Recommendation:** Consider adding a second group for "Internal/R&D" projects (from FRD03) with different posting profiles (no revenue accounts)

### Project Categories
- 4 category groups: Labour, Item, Expenses, Fee — standard and appropriate
- Categories to be collected during migration — ensure this is scheduled

### Project Stages
- Created → On-Hold → In Process → Finished — appropriate lifecycle
- Stage rules restricting actions — well-configured
- **Recommendation:** Consider adding a "Closing" stage between In Process and Finished for final checks, cost reallocation, and lessons learned before formal closure

---

## Summary of Recommendations

| # | Area | Recommendation | Impact |
|---|------|---------------|--------|
| 1 | PJ001 | Let CRM own opportunity stages; F&O receives at quotation stage only | Reduces duplication |
| 2 | PJ001 | Auto-populate "Confirmed received day" from PO delivery date | Reduces manual entry |
| 3 | PJ001 | Add "Closing" project stage for orderly project wrap-up | Better governance |
| 4 | PJ002 | Use adjustment journals for cost reallocation; automate if volume warrants | Solves biggest GAP |
| 5 | PJ002 | Build custom reports in Power BI, not SSRS | Easier maintenance |
| 6 | PJ003 | Add milestone value validation check (total vs. contract) | Prevents revenue errors |
| 7 | PJ004 | Default to automatic POC% with manual override approval | Reduces audit risk |
| 8 | Setup | Add "Internal/R&D" project group for non-billable projects | Proper cost segregation |

---

## Risk Flags

1. **Cost reallocation to assemblies** is the most complex customization. It requires either manual adjustment journals or a custom batch process. Test thoroughly with real project data.

2. **Cross-FRD dependencies are heavy:**
   - CRM Opportunity (FRD01) → F&O Opportunity (FRD04)
   - DFI approval (FRD01) → Quotation (FRD02/04)
   - CAD integration (FRD03) → WBS items (FRD04)
   - Timesheet from CRM → F&O project cost
   - Commercial invoice (Logistics FRD06) → Project invoice report

   These integrations must be designed as a cohesive architecture, not individually.

3. **Forecast model complexity.** The Original Budget → Current Estimate → Current Budget lifecycle with multiple forecast models is powerful but confusing for end users. Invest in training and SOPs.

4. **Report GAPs (3 custom reports + 2 report modifications):** This is a significant reporting effort. Power BI is strongly recommended over SSRS to reduce long-term maintenance.

---

## Efficiency Verdict

The Project Accounting solution is **the most mature and well-designed FRD** in the set. It demonstrates deep understanding of D365 Project Management & Accounting and makes excellent use of:
- WBS templates with cost details
- Resource allocation with multi-dimensional pricing
- Budget control with workflow
- Milestone billing with retention
- POC-based revenue recognition
- MS Project integration
- Sub-project structure for pre-win cost tracking

The main areas for improvement are: **using Power BI for reporting** (instead of SSRS customizations), **designing the cost reallocation mechanism** properly, and **streamlining the opportunity management** between CRM and F&O to avoid duplication.

---

*This review is based on D365 F&O Project Management & Accounting best practices as of 2026.*
