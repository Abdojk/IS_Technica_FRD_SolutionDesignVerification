# FRD12 - Budgeting - Solution Design Verification Report

**Document:** FRD12 - Budgeting - V3.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Budget Management & Budget Control

---

## Executive Summary

This FRD covers Technica International's budgeting strategy across 7 setup areas (BUD001-BUD007) and 3 business processes: Project Forecast to Budget (BUDP01), Budget Transfer (BUDP02), and Budget Register Entry (BUDP03). The solution design takes a **project-centric budgeting approach** where budget originates from the Project module (WBS forecasts) and is pushed to General Ledger budget control. This is a **sound architectural choice** for a make-to-order manufacturer where the majority of expenditure is project-driven.

The document is **shorter and less detailed than other FRDs in the set** (e.g., FRD04 Project Accounting, FRD07 Procurement). Several critical configuration decisions are marked as "pending" or "to be confirmed during migration," which introduces implementation risk. All identified requirements are marked FIT (no formal GAPs declared), though the document leaves several areas insufficiently defined that could surface as GAPs during build.

Overall assessment: **Architecturally correct but under-specified. The project-to-budget pipeline is well-conceived; the non-project (departmental) budgeting and budget control details need significantly more definition before build can begin.**

---

## Process-by-Process Review

### BUD001-BUD007 - Basic Budget Setup

**What was proposed:**
- Budget cycle set to **Quarterly** within yearly periods
- Financial dimensions for budgeting: **Department** and **Business Unit** (pending final confirmation)
- Additional dimension **"Budget Type"** for reference-only sub-classification (e.g., Ink, Business Cards under IT Consumables)
- Single budget model
- Three budget codes: Original Budget, Budget Transfer, Revision
- No budget transfer rules (all transfers go through workflow approval)
- Budget control on account structure: Main Account + Department + Business Unit
- Over-budget permission granted to specific budget controller role
- Budget funds available formula includes: Original Budget + Revision + Transfer minus Actual Expenditure, Unposted Actuals, Encumbrances (PO), Pre-Encumbrances (PR), and their unconfirmed variants
- All journals subject to budget control **except** Payroll, Fixed Assets, and Allocation journals
- Expense reports controlled by project category

**Assessment: Solid foundation but several decisions are deferred**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Budget cycle (Quarterly) | Good | Appropriate for project-based business with quarterly reviews |
| Financial dimensions (Dept + BU) | Pending | Final confirmation still required from Technica -- this is a blocking dependency |
| "Budget Type" reference dimension | Creative but Risky | Non-controlling dimension for sub-classification; see recommendation below |
| Single budget model | Appropriate | Correct for a company without scenario planning needs |
| Budget codes (3 types) | Standard OOB | Original, Transfer, Revision -- clean and sufficient |
| No transfer rules | Acceptable | Workflow-based transfer approval is the safer choice |
| Budget control formula | Excellent | Comprehensive formula including all encumbrance levels |
| Over-budget permission | Standard OOB | Role-based override -- correct approach |
| Document scope (journals, expense) | Well-defined | Proper exclusions for Payroll, FA, Allocation |
| Budget reporting (BUD007) | Under-specified | Only one sentence -- needs far more detail |

**Recommendations:**

1. **"Budget Type" dimension requires careful design.** Adding a financial dimension for reference-only classification (no budget control) is technically feasible but adds complexity to every transaction entry. Users must select this dimension on every journal line, expense report line, and PO line, even though it has no control effect. **Alternative approach:** Use **Main Account sub-segments** or **Financial Tags** (available in recent D365 releases) for this sub-classification. Financial Tags provide reference categorization without adding to the dimension string and without impacting budget control. If the dimension approach is retained, ensure:
   - The dimension is clearly marked as "not controlled" in all training materials
   - Default values are configured wherever possible to reduce data entry burden
   - The dimension is excluded from the budget control account structure

2. **Financial dimension confirmation is a critical-path blocker.** The dimensions (Department, Business Unit) affect every setup downstream: budget control configuration, budget register entries, financial reporting, and cross-module integration. **Recommendation:** Escalate this decision immediately. No budget build should begin until dimensions are finalized. Any change after configuration will require full rebuild of budget control rules.

3. **Budget cycle says "Yearly" in one place and "Quarterly" in another.** BUD001 states "Technica will control its budget during yearly periods" and then "Technica budget cycle will be set to Quarterly." This needs clarification. In D365, the Budget Cycle Time Span defines the period granularity for budget amounts and control. **Recommendation:** For a make-to-order manufacturer, **annual budget with quarterly period allocation** is typical. Set the Budget Cycle Time Span to the fiscal year, and distribute amounts across quarterly fiscal periods. This allows budget entry at annual level with quarterly monitoring.

4. **Over-budget permission scope is vague.** The FRD says "certain budget controller" but does not define the role, the threshold, or whether over-budget should be warning-only or hard-stop for other users. **Recommendation:** Define explicitly:
   - Default behavior for all users: **Hard stop** (budget check fails, transaction cannot proceed)
   - Budget controller role: **Allow with warning** (can override)
   - Finance Manager: **Allow with warning**
   - Define whether an audit log of over-budget overrides is required (it should be)

5. **Payroll and Fixed Assets exclusion is correct** but needs validation. If Technica wants to track total departmental spend (including salaries and depreciation) against budget, these exclusions will create blind spots. **Recommendation:** Confirm with Technica Finance whether they need:
   - Option A: Budget control on operational expenses only (current design) -- simpler
   - Option B: Full budget control including payroll and FA -- requires budget entries for salary and depreciation projections

6. **BUD007 - Budget Reporting is critically under-specified.** A single sentence about "tracking project category expense across multiple projects" is insufficient for a reporting requirement. **Recommendation:** Expand BUD007 into a proper reporting specification covering:
   - Budget vs. Actual by Department by Period (standard D365 budget report)
   - Budget vs. Actual by Main Account by Department (Power BI recommended)
   - Budget consumption % by Business Unit (Power BI dashboard)
   - Project budget vs. actual cross-project view (from FRD04 project accounting data)
   - Budget variance alerts (Power Automate when consumption exceeds threshold)

---

### BUDP01 - Project Forecast to Budget (All FIT - 5 requirements)

**What was proposed:**
- Create WBS in Project module with cost forecasts
- Push project forecast to General Ledger budget using "Copy Project Forecast to Budget" function
- Budget register entry created automatically with budget type "Project"
- Workflow approval on budget register entry
- After approval, "Update budget balance" activates budget control
- 1 project per year per department with specific activity categories (e.g., "IT Equipment Exhibition 2023")
- Budget controlled at activity level within department (no fungibility between categories)
- For journals and expense reports: account linked to project category that maps back to budget
- For procurement: procurement category linked to expense category linked to project module
- Exceptional/non-project budget handled via manual register entries under "Original," "Revision," or "Transfer" codes

**Assessment: This is the core budgeting mechanism and it is well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| WBS creation and forecast | Standard OOB | Leverages FRD04 project structure |
| Copy forecast to budget | Standard OOB | Built-in D365 function |
| Automatic budget register entry | Standard OOB | Proper linkage to budget module |
| Workflow approval | Standard OOB | Single workflow for all budget types |
| Budget activation | Standard OOB | Update budget balance step |
| Per-department project per year | Creative | Effective departmental budget ownership |
| Activity-level control (no fungibility) | Good | Prevents cross-category spending |
| Procurement category mapping | Needs Verification | Mapping chain must be validated |
| Exceptional manual entries | Standard OOB | For non-project departmental budget |

**Recommendations:**

1. **The "1 project per year per department" model is creative but has scaling concerns.** If Technica has 15 departments, that is 15 projects per year just for budget management. Over 5 years, that is 75 projects cluttering the project list. **Recommendations:**
   - Use a clear naming convention with prefix: `BUD-[DEPT]-[YEAR]` (e.g., `BUD-IT-2024`, `BUD-FIN-2024`)
   - Use a dedicated **Project Group** (e.g., "Budget Control") to segregate these from real customer projects
   - Set project type to **"Internal"** or **"Cost"** (never "Fixed price" or "Time and material" which are for customer billing)
   - Implement a year-end closure process for previous-year budget projects
   - Consider using **Project Contracts** to group department budget projects by year for reporting

2. **Category-level budget rigidity needs validation with business users.** The FRD explicitly states: "Technica budget strategy will not accept taking a budget from training and use it on laptop even if it doesn't exceed the overall budget." This is very rigid. While Finance Manager confirmed this, department heads may resist during UAT when they cannot reallocate within their own budget. **Recommendation:** Implement the rigid control as designed, but ensure the Budget Transfer process (BUDP02) is fast and lightweight enough to handle frequent reallocation requests. A slow approval process combined with rigid control will frustrate operations.

3. **The procurement category-to-project category mapping chain is critical and complex.** The flow is: Procurement Category --> Category Expense --> Project Category --> Budget Account. Each link must be correctly configured. **Recommendation:**
   - Create a complete mapping matrix before configuration
   - Test every procurement category with a test PR and PO to verify budget check fires
   - Document the mapping as a reference for ongoing maintenance
   - This is a direct integration point with **FRD07 (Procurement)** -- coordinate with procurement team

4. **The "Copy Project Forecast to Budget" function overwrites previous entries by default.** If Technica revises the WBS forecast mid-year and re-pushes to budget, the previous budget register entries need to be handled. **Recommendation:** Define the re-push process:
   - Option A: Delete old budget register entries and re-push (clean but loses history)
   - Option B: Push revision as a separate budget code "Revision" (preserves history, requires manual reconciliation)
   - Option C: Use forecast models (as described in FRD04) to manage versions, and always push from the "Current" model
   - **Recommend Option C** as it aligns with the forecast model lifecycle already designed in FRD04

5. **Integration with FRD04 is tight and well-aligned.** The project forecast-to-budget pipeline is essentially the same mechanism described in FRD04's PJ002 (Project Execution). Ensure:
   - The forecast model used for the budget push is clearly identified (T_Forecast or NT_Forecast from FRD04)
   - Budget register entries from project forecast carry the correct financial dimensions
   - The workflow for budget approval is consistent between FRD04 and FRD12

---

### BUDP02 - Budget Transfer (Pending Workflow Confirmation)

**What was proposed:**
- Once original budget is posted, transfers between departments/accounts are possible
- Budget transfer workflow approval (pending confirmation from Technica)
- No transfer rules defined (per BUD005)

**Assessment: Minimal definition -- needs expansion**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Budget transfer capability | Standard OOB | Core D365 function |
| Workflow approval | Pending | Technica has not confirmed workflow requirement |
| Transfer rules | Not Used | All transfers subject to approval (safe approach) |
| Transfer scope/limits | Not Defined | No guidance on what transfers are allowed |
| Process flow | Not Defined | No flowchart or step-by-step description provided |

**Recommendations:**

1. **Workflow for budget transfers should be mandatory, not optional.** Budget transfers without approval represent a control weakness. **Recommendation:** Implement a two-level approval workflow:
   - Level 1: Department Head of the source budget (approves releasing funds)
   - Level 2: Finance Controller (approves the transfer overall)
   - For inter-department transfers, add approval from the receiving Department Head as well

2. **Define transfer boundaries explicitly.** The FRD does not specify:
   - Can budget be transferred between departments? (likely yes, but confirm)
   - Can budget be transferred between business units? (may have legal entity implications)
   - Can budget be transferred between main accounts? (e.g., from Travel to Equipment)
   - Can budget be transferred between fiscal periods/quarters?
   - **Recommendation:** Document allowed and prohibited transfer combinations. D365 Budget Transfer Rules can enforce these boundaries automatically, even though the FRD says they won't use them. Reconsider using transfer rules for at least the prohibited combinations.

3. **Budget transfer integration with FRD04 project budgets.** If the original budget came from project forecast (BUDP01), a budget transfer at the GL level does **not** automatically update the project budget. This creates a disconnect: the project shows one budget, the GL shows another. **Recommendation:** Define whether budget transfers should also trigger a project budget revision in FRD04. If so, the transfer process needs an additional step to update the project forecast.

---

### BUDP03 - Budget Register Entry (Manual Process)

**What was proposed:**
- Finance requests departmental budgets via email
- Departments create budget register entries using Excel Open-in-Excel functionality
- Departments fill in their budget figures in Excel
- Submit for workflow approval
- Finance approves
- Budget takes effect after approval

**Assessment: Practical approach for non-project departmental budgets**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Email-based budget request | Process Step | Not a system requirement |
| Excel-based entry | Standard OOB | Open-in-Excel is built-in |
| Departmental entry | Good | Empowers department ownership |
| Workflow approval | Standard OOB | Finance approval gate |
| Budget activation | Standard OOB | Standard post-approval step |

**Recommendations:**

1. **The Excel-based entry approach is practical but has data quality risks.** Departments may enter incorrect main accounts, wrong dimensions, or misformatted amounts. **Recommendation:**
   - Create a **budget register entry template** with pre-populated main accounts and dimensions per department
   - Lock the account/dimension columns so departments can only fill in amounts
   - Use D365 **Budget Register Entry validation** to catch errors on import
   - Consider using a **Budget Preparation workspace** (available in D365) for a more guided experience

2. **No budget calendar or timeline is defined.** The FRD does not specify when annual budget preparation begins, submission deadlines, review cycles, or final approval dates. **Recommendation:** Define a budget calendar:
   - Month 1: Finance sends templates to departments
   - Month 2: Departments prepare and submit
   - Month 3: Finance reviews, consolidates, requests revisions
   - Month 4: Final approval and activation
   - This should align with Technica's fiscal year start

3. **Relationship between BUDP01 (project forecast push) and BUDP03 (manual entry) needs clarity.** If project-related expenses are budgeted via BUDP01 and departmental/overhead expenses via BUDP03, there must be a clear demarcation. Specifically:
   - Which main accounts are populated by project forecast?
   - Which main accounts are populated by manual entry?
   - Is there any overlap (same account budgeted by both methods)?
   - **Recommendation:** Create an account allocation matrix showing: Account X = Project Budget (BUDP01), Account Y = Departmental Budget (BUDP03), with no overlap. This prevents double-budgeting.

4. **No mention of budget amendments or mid-year revisions.** The FRD describes initial budget entry but not how mid-year changes are handled. **Recommendation:** Use the "Revision" budget code for mid-year amendments, with the same workflow approval. Document the triggers for revision (e.g., new project won, organizational changes, emergency spend).

---

## GAP Assessment

The FRD declares **all requirements as FIT** across all three processes. However, several items are functionally dependent on decisions not yet made, and the following **implicit gaps** should be tracked:

| Implicit Gap | Description | Complexity | Recommendation |
|-------------|-------------|-----------|----------------|
| Budget Type dimension | New financial dimension for reference classification | Medium | Consider **Financial Tags** instead of a full financial dimension. Tags provide categorization without impacting budget control structure or transaction entry burden. If dimension is retained, ensure default values are configured per department. |
| Budget reporting (BUD007) | "Track project category expense across multiple projects" -- single sentence, no specification | Medium | Build a **Power BI report suite** with: (1) Budget vs. Actual by Department, (2) Budget consumption dashboard, (3) Cross-project category spend analysis. Use D365 Budget vs. Actuals data entity as the source. |
| Procurement budget control mapping | Procurement category to expense category to project category chain for budget checking on PRs/POs | Medium | Create and validate the complete **category mapping matrix** before go-live. Test each procurement category with end-to-end PR-to-PO-to-Budget-Check scenarios. This is the bridge between FRD07 and FRD12. |
| Budget transfer workflow | Workflow for budget transfers not yet confirmed | Low | Implement workflow as **mandatory**. Define transfer boundaries and approval levels. |
| Budget re-push from project | What happens when WBS forecast is revised and re-pushed to budget | Medium | Define versioning strategy. Recommend using forecast model revisions (aligned with FRD04) and pushing from the "Current" forecast model only. |
| Budget calendar/timeline | No annual budget preparation timeline defined | Low | Define and document a budget preparation calendar as an SOP, not a system configuration. |

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | BUD002 | Finalize financial dimensions immediately -- this is a critical-path blocker | **Critical** | Blocks all budget setup and configuration |
| 2 | BUD002 | Replace "Budget Type" dimension with D365 Financial Tags for reference classification | High | Reduces transaction entry complexity |
| 3 | BUD001 | Clarify budget cycle: annual budget with quarterly period allocation | Medium | Prevents configuration confusion |
| 4 | BUD006 | Define explicit over-budget permission levels per role (hard stop vs. warning) | High | Strengthens budget control governance |
| 5 | BUD006 | Validate Payroll/FA exclusion with Finance -- confirm they don't need full cost visibility | Medium | Prevents reporting blind spots |
| 6 | BUD007 | Expand budget reporting to a full specification; build in Power BI | High | Currently a single sentence -- insufficient for build |
| 7 | BUDP01 | Establish naming convention and project group for budget projects (e.g., BUD-IT-2024) | Medium | Prevents project list clutter |
| 8 | BUDP01 | Validate procurement-category-to-budget mapping chain end-to-end with FRD07 | **Critical** | Budget control will not work on procurement without correct mapping |
| 9 | BUDP01 | Define WBS forecast re-push strategy using FRD04 forecast model versioning | High | Prevents budget overwrite issues |
| 10 | BUDP02 | Implement mandatory two-level workflow for budget transfers | High | Closes control gap |
| 11 | BUDP02 | Define allowed/prohibited transfer boundaries | Medium | Prevents unauthorized budget movement |
| 12 | BUDP03 | Create pre-populated Excel templates per department with locked account columns | Medium | Reduces data entry errors |
| 13 | BUDP03 | Define budget preparation calendar (SOP) | Medium | Ensures timely budget cycle completion |
| 14 | BUDP03 | Create account allocation matrix (project budget vs. departmental budget) | High | Prevents double-budgeting |

---

## Risk Flags

1. **Unconfirmed financial dimensions are the single biggest risk.** The FRD explicitly states "Still waiting final decision from Technica team as they might change them." Dimensions affect budget control configuration, budget register entries, account structure, and every downstream process. Any change after build begins will cascade across the entire budget module and likely into GL, Project Accounting, and Procurement. **Action:** Escalate to project steering committee for immediate decision.

2. **Cross-FRD integration complexity is high and under-documented.** The budgeting solution depends on tight integration with:
   - **FRD04 (Project Accounting):** WBS forecast is the primary budget source; forecast model lifecycle must be synchronized
   - **FRD07 (Procurement):** Budget control on PRs and POs requires correct category mapping chain (Procurement Category --> Expense Category --> Project Category --> Main Account)
   - **Expense Management (referenced but no FRD number):** Expense reports controlled by project category against budget

   These integration points are mentioned but not designed as end-to-end flows. **Risk:** Each FRD team builds their component in isolation, and integration fails during SIT/UAT.

3. **Budget control at activity level (no fungibility) will generate frequent budget transfer requests.** The design explicitly prevents spending Training budget on Laptops even within the same department's overall allocation. While this is what Finance requested, it will create operational friction. If the Budget Transfer process (BUDP02) is slow or bureaucratic, departments will escalate. **Mitigation:** Ensure the transfer workflow has a target SLA (e.g., 24-hour approval) and consider a self-service transfer capability for small amounts below a defined threshold.

4. **The "Budget Type" reference dimension adds transaction entry overhead.** Every journal line, expense report line, and PO line will require this additional dimension selection. If defaults are not properly configured, users will either select incorrect values (degrading data quality) or resist entering the value (creating incomplete records). **Mitigation:** Either replace with Financial Tags or invest heavily in default value configuration and user training.

5. **Budget reporting is essentially undefined.** BUD007 contains a single sentence. Budget reporting is one of the most visible outputs of the budgeting process -- Finance leadership, department heads, and executive management will all expect tailored views. An undefined reporting scope risks: (a) scope creep during build, (b) last-minute report requests during UAT, or (c) go-live with no usable budget reports. **Mitigation:** Conduct a dedicated reporting requirements workshop before build phase.

6. **No mention of budget carry-forward or year-end process.** The FRD does not address what happens at fiscal year-end:
   - Do unspent budgets carry forward to next year?
   - How are new-year budget projects created?
   - What is the transition process from old to new budget?
   - **Mitigation:** Define year-end budget procedures as part of the SOP, and configure D365 Budget Carry-Forward if needed.

---

## Efficiency Verdict

The FRD12 Budgeting solution is **architecturally sound but operationally thin.** The core design decision -- driving budget from project forecasts (BUDP01) and supplementing with manual departmental entries (BUDP03) -- is the correct approach for a make-to-order manufacturer where project costs dominate. The budget control configuration (BUD006) is properly specified with a comprehensive funds-available formula, appropriate journal scope, and encumbrance/pre-encumbrance tracking.

However, the document is the **least mature FRD** compared to the detailed specifications in FRD04 (Project Accounting) and FRD07 (Procurement). Several critical decisions are deferred ("still waiting," "to be confirmed during migration"), the budget transfer process lacks a flowchart or detailed steps, and budget reporting is reduced to a single sentence. For a module that touches every department's spending authority, this level of specification is insufficient.

**Key strengths:**
- Project forecast-to-budget pipeline leverages D365 standard capability
- Budget control formula is comprehensive and includes all encumbrance levels
- Activity-level control enforces Technica's strict departmental budget discipline
- Workflow-based approval for all budget operations
- Proper exclusion of Payroll and Fixed Assets from budget control

**Key gaps to close before build:**
- Finalize financial dimensions (Department, Business Unit -- or alternatives)
- Resolve "Budget Type" dimension vs. Financial Tags decision
- Expand BUD007 into a full reporting specification
- Design end-to-end integration flows with FRD04 and FRD07
- Define budget transfer boundaries and mandatory workflow
- Create procurement category-to-budget mapping matrix
- Document budget preparation calendar and year-end procedures

The solution will work well once these gaps are addressed. The risk is not in the design itself but in the **missing details that could derail configuration and testing.**

---

*This review is based on D365 F&O Budget Management & Budget Control best practices as of 2026.*
