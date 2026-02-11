# FRD12 - Expense Management - Solution Design Verification Report

**Document:** FRD12 - Expense Management - V3.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Expense Management

---

## Executive Summary

This FRD covers a single consolidated process (EM001) encompassing the full employee expense lifecycle: **Travel Requisition, Cash Advance, and Expense Report**. The solution makes **good use of D365 F&O standard Expense Management capabilities** -- per diem setup, shared/expense categories, cash advance processing, expense report reconciliation against travel requisitions, and project cost allocation via expense categories. Of 9 requirements, **7 are FIT and 2 are GAP**.

However, the document reveals several structural concerns:

1. **The CRM-to-F&O Travel Requisition integration (EM001-001) is the most significant item** and directly depends on FRD01's Trips & Missions custom entity. The integration design is superficially described -- fields are listed but the mechanism, error handling, and data mapping are undefined.
2. **Critical setup data is missing from Technica** (workflows, policies, per diem rates, shared categories with accounts, payment methods). The solution design cannot be fully validated without this data.
3. **The "expense charged to customer" GAP (EM001-004)** is described too vaguely. The business need (debit note to customer for reimbursable expenses) is clear, but the proposed solution (a checkbox) is incomplete -- it does not address how the downstream debit note is generated.
4. **The document mislabels itself** as "Accounts Payable requirements" in the introduction, suggesting it was templated from another FRD without proper review.

Overall assessment: **Functional solution leveraging standard D365 capabilities, but incomplete in design details. The two GAPs need sharper definition, and the 8 missing data items from Technica represent a project risk.**

---

## Process-by-Process Review

### EM001 - Expense Management (Travel Requisition, Cash Advance & Expense Report)

This is treated as a single process in the FRD. For clarity, this review breaks it into its three functional sub-processes plus the CRM integration layer.

---

### EM001 (Part A) - CRM Integration: Trip Request to Travel Requisition

**What was proposed:**
- Travel/trip requests created and approved in CRM (D365 CE custom entity from FRD01)
- On CRM approval, data is pushed to D365 F&O as a Travel Requisition draft
- Fields auto-populated: Business purpose, Destination, Travel description, Amount (header)
- A customized header field for controlling header vs. line values
- A validation ensuring header amount equals sum of line details
- CRM sequence number stored as reference; D365 number sequence preserved for direct F&O entries

**Assessment: Integration concept is sound but design is incomplete**

| Aspect | Rating | Comment |
|--------|--------|---------|
| CRM as trip entry point | Acceptable | Justified since sales team lives in CRM |
| Auto-population of TR fields | Good | Reduces duplicate entry vs. FRD01's concern |
| Header-to-line validation | Good | Prevents mismatched totals |
| Dual number sequence (CRM ref + D365) | Good | Maintains traceability without breaking D365 |
| Integration mechanism | Undefined | No mention of Dual-Write, Power Automate, or custom API |
| Error handling | Missing | What happens if CRM push fails? No retry/notification logic |
| Field mapping specification | Incomplete | Only header fields listed; line-level mapping absent |

**Recommendations:**

1. **Define the integration mechanism explicitly.** Given this is an event-driven, approval-triggered action (not continuous sync), **Power Automate with Dataverse trigger + F&O connector** is the most appropriate pattern. Dual-Write is overkill for this scenario. A custom API (X++ Data Entity exposed as OData) for Travel Requisition creation would provide the endpoint. Specifically:
   - Trigger: CRM Trip record status changes to "Approved"
   - Action: Call F&O Travel Requisition OData endpoint to create header + lines
   - Fallback: On failure, set CRM record to "Integration Error" status and notify admin

2. **The "customized field on header" is vague.** Clarify what this field represents. If it is a trip total amount from CRM, map it to the Travel Requisition estimated amount field (standard). If it is something else (e.g., budget limit, trip category), define it precisely. Unnecessary custom fields on standard forms increase upgrade risk.

3. **Line-level data must be addressed.** Currently, only header fields are auto-populated. The FRD states the user must manually fill expense categories, dates, and amounts in F&O after the integration. This creates a **two-system data entry problem** (also flagged in FRD01 review). Better approach:
   - If CRM captures estimated expenses per category (airfare, hotel, meals), push them as Travel Requisition lines with category, estimated date range, and estimated amount.
   - If CRM only captures a lump sum, create a single Travel Requisition line with a generic "Trip Expense" category and the total amount, then let the user refine in F&O.

4. **Cross-reference with FRD01 (SP001 - Trips & Missions):** FRD01's review recommended considering whether the CRM entity is necessary at all, or whether a Power App could serve the purpose. If the CRM entity is retained, ensure the field structure in CRM aligns with the D365 F&O Travel Requisition entity fields. A field mapping document should be produced as a design artifact.

---

### EM001 (Part B) - Travel Requisition Process

**What was proposed:**
- Employee creates Travel Requisition in F&O (or receives one from CRM integration)
- Fills: Business purpose, destination, travel description, expense categories, estimated amounts
- Financial dimensions auto-populated from worker record
- Workflow: Submit -> Manager Approval -> HR Manager Approval -> Approved
- Approved TR can later be linked to Expense Reports for reconciliation

**Assessment: Standard OOB -- well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Travel Requisition creation | Standard OOB | Clean form, proper fields |
| Financial dimension defaulting | Standard OOB | From worker employment record |
| Workflow (Manager -> HR) | Standard OOB | Needs Technica's actual approval matrix |
| TR-to-Expense Report reconciliation | Standard OOB | Partial reconciliation supported |
| Per diem integration on TR | Standard OOB | Location-based per diem rates |

**Recommendations:**

1. **The suggested workflow (Employee -> Manager -> HR Manager) is a placeholder.** The document explicitly states "Technica needs to provide us with the updated workflow." This is a project risk -- workflow design affects security roles, approval hierarchies, and delegation rules. **Recommendation:** Escalate this as a blocking item. Do not proceed to configuration without confirmed workflows. Provide Technica with a workflow questionnaire covering:
   - Approval thresholds (e.g., < $1,000 = manager only, > $1,000 = manager + HR + CFO)
   - Delegation rules when approvers are absent
   - Whether the same workflow applies to all legal entities
   - Auto-approval rules (if any)

2. **Per diem setup is pending Technica's internal policies.** The FRD correctly identifies per diem configuration by country with hotel/meal/other breakdowns, but no rates are defined. **Recommendation:** Provide Technica with the D365 per diem template (Excel export from Per Diem setup form) pre-populated with common countries and standard GSA/HMRC rates as a starting point. This accelerates data collection.

3. **Travel Requisition reconciliation with Expense Reports** is a powerful standard feature that is well-described. The partial reconciliation capability (showing remaining amount to reconcile) is correctly documented. No changes needed here.

---

### EM001 (Part C) - Cash Advance Process

**What was proposed:**
- Employee creates Cash Advance request with amount, currency, location
- Financial dimensions default from worker record
- Workflow approval: same suggested flow (Manager -> HR Manager)
- Once approved, accountant processes payment via "Approved and paid cash advances" screen
- Payment creates a General Ledger journal: Debit Cash Advance Account, Credit Vendor (employee-mapped)
- Cash advance accounts are unique per currency
- Employee financial dimension is mandatory on cash advance accounts
- Employee must be mapped as a Vendor for AP processing
- Journal can be auto-posted or manually posted (configurable)

**Assessment: Standard OOB with proper account structure**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Cash advance request | Standard OOB | Simple, clean process |
| Currency-specific accounts | Good Practice | Proper multi-currency handling |
| Employee financial dimension | Good Practice | Enables cost tracking by employee |
| Employee-to-vendor mapping | Standard OOB | Required for AP settlement |
| Payment processing | Standard OOB | Journal creation + posting |
| Auto-post option | Standard OOB | Correctly left as configurable |

**Recommendations:**

1. **Cash advance account per currency is necessary but creates maintenance overhead.** For a company using "many currencies" (as stated), this means many GL accounts. **Recommendation:** Use a single cash advance control account with the currency as a financial dimension rather than separate accounts per currency -- **but only if Technica's chart of accounts supports this pattern.** If they must use separate accounts (common in Middle Eastern accounting practices), create a clear naming convention (e.g., 1310-USD, 1310-EUR, 1310-LBP) and document all accounts in a setup register.

2. **The employee-as-vendor setup is standard but requires governance.** Every employee with cash advance access must have a corresponding vendor record. **Recommendation:**
   - Create vendors in a dedicated vendor group (e.g., "Employee Vendors") with restricted payment terms
   - Use a number sequence aligned with employee IDs for easy cross-reference
   - Add this to the employee onboarding/offboarding checklist -- new employee = create vendor; departing employee = settle all advances and inactivate vendor
   - Consider using the **D365 Employee Expense Vendor auto-creation** feature (available in recent builds) which auto-creates the vendor when the employee is first mapped

3. **Manual vs. auto-post for cash advance journals.** The FRD correctly leaves this configurable. **Recommendation:** Start with manual posting (auto-post disabled) during go-live to allow the finance team to review entries. Switch to auto-post after 2-3 months of stable operation when the team is confident in the setup.

4. **The "paid" status transition should trigger a notification to the employee.** The current process relies on the employee checking the system. **Recommendation:** Add a Power Automate flow triggered on cash advance status change to "Paid" that sends an email/Teams notification to the employee. This is low-effort and high-value for user experience.

---

### EM001 (Part D) - Expense Report Process

**What was proposed:**
- Employee creates Expense Report after travel, detailing actual expenses with receipts
- Can link to previously approved Travel Requisition for reconciliation
- Expense categories drive field behavior (per diem fields, hotel itemization, airline itemization)
- Workflow approval -> Payable officer posts the expense report
- Posting creates journal: Debit Expense (per category), Credit Vendor (employee)
- Cash advance return: special category "C.A. Returns" allows full/partial return of advance
- Partial returns reduce cash advance balance; remaining balance settable via AP
- Project expense: expense category enabled for Project links expense to project module and P&L

**Assessment: Well-documented standard process with good edge case coverage**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Expense report creation | Standard OOB | Proper category-driven fields |
| Travel Requisition reconciliation | Standard OOB | Partial reconciliation supported |
| Hotel/airline itemization | Standard OOB | Expense type drives sub-categories |
| Cash advance return (full/partial) | Standard OOB | Well-documented with example |
| Project cost allocation | Standard OOB | Category enabled for project + expense |
| Receipt attachment | Standard OOB | Mobile app scanning supported |
| Workflow & posting | Standard OOB | Proper approval -> posting flow |

**Recommendations:**

1. **Mobile expense submission (EM001-003) is FIT and should be prioritized.** The D365 Expense Management mobile app (available on iOS/Android) allows receipt scanning with OCR, automatic category suggestion, and on-the-go expense entry. **Recommendation:** Include mobile app setup and user training in the deployment plan. Specifically:
   - Configure the mobile workspace in F&O
   - Test OCR accuracy with Technica's common receipt types (Arabic/English receipts, multiple currencies)
   - Define whether mobile-submitted expenses still require desktop review before workflow submission

2. **The cash advance return flow is well-designed** but the example only shows partial return. Document the full return scenario as well, and clarify what happens if the employee's expense report total exceeds the cash advance (i.e., the company owes the employee the difference). This is handled by standard AP settlement, but it should be explicitly documented for user training.

3. **Project expense allocation (EM001-009) is correctly described** but lacks detail on:
   - Which expense categories will be project-enabled (this depends on Technica's project cost structure from FRD04)
   - Whether project budget checking applies to expenses (it should, if project budget control is enabled)
   - How project financial dimensions interact with employee financial dimensions on the same transaction

   **Recommendation:** Cross-reference with FRD04 (Project Accounting) to ensure expense categories used in projects align with project category groups. The expense category must exist in both Expense Management shared categories AND Project category configuration.

---

## GAP Analysis

### GAP 1: EM001-001 - CRM Trip Integration to Travel Requisition

| Attribute | Assessment |
|-----------|------------|
| **Complexity** | Medium-High |
| **Effort Estimate** | 3-5 days development + 2 days testing |
| **Risk Level** | Medium (cross-system integration) |
| **Dependencies** | FRD01 (CRM Trips entity), Integration architecture decision |

**What is proposed:** CRM trip approval triggers creation of D365 F&O Travel Requisition.

**Is there a better way?**

| Option | Approach | Pros | Cons |
|--------|----------|------|------|
| **A (Proposed)** | CRM push to F&O Travel Requisition | Single trip entry point for sales team; CRM context preserved | Requires integration development; partial data push means double entry |
| **B (Recommended)** | CRM push with full line-level data | Complete automation; no double entry | Requires CRM entity to capture expense categories and amounts (more CRM customization) |
| **C (Alternative)** | CRM for trip tracking only; F&O for Travel Requisition | Clean separation; no integration needed for TR | Sales team must enter TR separately in F&O; loses traceability |
| **D (Best if feasible)** | Power Apps portal for trip + TR | Single entry point with both CRM context and F&O financial data | Requires Power Apps development; may not suit all users |

**Recommendation:** Go with **Option B** -- enhance the CRM trip entity to capture estimated expense lines (category, amount, dates) and push complete Travel Requisitions to F&O. The incremental CRM customization cost is lower than the ongoing cost of manual re-entry by every traveling employee. If CRM entity changes are too expensive, fall back to **Option A** with a clear SOP for employees to complete the TR in F&O within 24 hours of CRM approval.

---

### GAP 2: EM001-004 - Expense Report "Charged to Customer" Checkbox

| Attribute | Assessment |
|-----------|------------|
| **Complexity** | Low (checkbox) to Medium-High (end-to-end with debit note) |
| **Effort Estimate** | 1 day for checkbox; 3-5 days for debit note automation |
| **Risk Level** | Low (checkbox only); Medium (if debit note automation is in scope) |
| **Dependencies** | AR module (debit note process), Customer master data |

**What is proposed:** Add a checkbox on expense report lines to flag expenses that should be charged to the customer. Technica currently debits these to an expense account and later issues a manual debit note to the customer.

**Is there a better way?**

| Option | Approach | Pros | Cons |
|--------|----------|------|------|
| **A (Proposed)** | Checkbox on expense line + manual debit note later | Simple to implement; familiar to finance team | Manual process; risk of forgetting to invoice customer; no automation |
| **B (Recommended)** | Expense category "Customer-Reimbursable" + Intercompany/Project billing | Uses standard expense categories; can auto-route to specific GL account; integrates with project invoicing if project-linked | Requires discipline in category selection; needs category-level setup |
| **C (Best for project expenses)** | Use Project billing rules for reimbursable expenses | Standard D365 project invoicing creates customer invoice automatically; full audit trail; integrates with FRD04 milestone/T&M billing | Only works for project-linked expenses; not suitable for non-project travel |
| **D (Comprehensive)** | Combination: Project billing (Option C) for project expenses + Custom "Reimbursable" flag with Power Automate notification for non-project expenses | Covers both scenarios; automates where possible | More setup effort |

**Recommendation:** Implement **Option D** as a phased approach:
- **Phase 1 (Go-live):** For project-linked expenses, use the standard D365 Project billing mechanism -- expenses posted to a project can be invoiced to the customer via project invoice proposals. This is entirely standard and already supported by EM001-009.
- **Phase 2 (Go-live):** For non-project reimbursable expenses, add a custom enum field (not just a checkbox) on the expense line with values: "Company Borne" (default), "Customer Reimbursable." Map this field to a specific GL account (e.g., "Customer Reimbursable Expenses - Clearing"). Add a **Power Automate flow** that triggers when an expense report with "Customer Reimbursable" lines is posted, sending a notification to the AR team with customer name, amount, and expense details.
- **Phase 3 (Post go-live):** If volume warrants, build a periodic batch job that collects all "Customer Reimbursable" clearing account balances by customer and auto-creates AR free-text invoice proposals (debit notes) for review and posting.

---

## Setup Review

### Expense Management Parameters
- Ledger daily journal name for cash advance: Correctly identified as configurable
- Post cash advance immediately: Correctly left as optional
- **No mention of:** Expense policy configuration (per diem limits, receipt requirements, spending limits by category). This is a significant omission that depends on Technica's internal policies (listed as missing data item #2).

### Shared Categories / Expense Categories
- Correctly described as mandatory setup
- Expense type drives field behavior (per diem, hotel, airline) -- well-explained with examples
- Payment method per category -- properly linked
- **Missing:** Actual category list, GL account mapping, and sub-ledger posting profiles. These are listed as pending from Technica.

### Per Diem Configuration
- Country-based setup correctly described
- Hotel/meal/other breakdown supported
- **Recommendation:** Configure per diem reduction rules (e.g., if hotel is provided by company, reduce per diem by hotel portion). D365 supports automatic per diem reduction -- ensure this is configured to match Technica's policy.

### Cash Advance Accounts
- Per-currency setup: Properly designed
- Employee financial dimension: Confirmed as mandatory -- good governance decision
- **Recommendation:** Document the full account structure in a setup workbook before configuration begins.

### Worker Setup (Employee Financial Dimension + Vendor Mapping)
- Financial dimension on worker employment record: Required for expense management -- correctly identified
- Vendor mapping on worker: Required for cash advance payment -- correctly documented with step-by-step instructions
- **Recommendation:** Create a checklist for HR/IT to ensure every expense-eligible employee has both the financial dimension AND vendor mapping configured before go-live. Missing either will cause transaction failures.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | CRM Integration | Define integration mechanism (Power Automate + OData endpoint recommended) | **Critical** | Blocks GAP development |
| 2 | CRM Integration | Push line-level data from CRM to eliminate double entry in F&O | High | Improves user efficiency |
| 3 | CRM Integration | Produce a formal field mapping document (CRM Trip -> F&O Travel Requisition) | High | Reduces integration defects |
| 4 | Workflow | Escalate workflow design as blocking item; provide Technica a questionnaire template | **Critical** | Blocks configuration |
| 5 | Customer Reimbursable | Use project billing for project expenses (standard); custom enum + clearing account for non-project | High | Solves GAP properly |
| 6 | Customer Reimbursable | Add Power Automate notification to AR team for reimbursable expenses | Medium | Prevents missed invoicing |
| 7 | Per Diem | Provide Technica with pre-populated per diem template to accelerate data collection | Medium | Accelerates setup |
| 8 | Per Diem | Configure per diem reduction rules for company-provided meals/hotel | Medium | Policy compliance |
| 9 | Cash Advance | Start with manual journal posting; switch to auto-post after stabilization | Medium | Reduces go-live risk |
| 10 | Cash Advance | Add payment notification to employee via Power Automate | Low | Better user experience |
| 11 | Mobile App | Include mobile expense workspace in deployment plan and test OCR with local receipts | Medium | High adoption value |
| 12 | Worker Setup | Create onboarding checklist for financial dimension + vendor mapping per employee | High | Prevents transaction failures |
| 13 | Expense Policy | Define and configure expense policies (spending limits, receipt thresholds) once Technica provides internal policies | High | Compliance and control |

---

## Risk Flags

1. **Eight missing data items from Technica.** The FRD explicitly lists 8 categories of data that Technica has not yet provided: workflows, internal policies, shared categories with accounts, per diem rates, cash advance accounts, payment methods, expense purposes/locations, and employee access lists. **This is the single biggest risk to this workstream.** Without this data, configuration cannot begin, and the design cannot be fully validated. Recommend a formal data collection deadline with escalation path.

2. **CRM integration architecture is undefined across multiple FRDs.** FRD12's CRM Trip -> Travel Requisition integration depends on FRD01's custom Trip entity, which in turn has its own design questions (see FRD01 review). If FRD01's Trip entity design changes, FRD12's integration must change with it. These two FRDs must be designed together, not in isolation.

3. **The "Charged to Customer" GAP has downstream implications not addressed.** The FRD describes the need but not the end-to-end flow. Who creates the debit note? When? How is the customer identified on the expense line? What GL accounts are involved in the clearing? Without answering these questions, the checkbox is cosmetic -- it flags the expense but does not solve the billing problem.

4. **Expense policy configuration is absent.** D365 F&O Expense Management supports robust policy rules:
   - Maximum amount per expense category
   - Receipt required above threshold amount
   - Per diem limits by location
   - Advance booking requirements for flights
   - Non-compliant expense warnings vs. hard blocks

   None of these are discussed in the FRD. This is a governance gap -- expense management without policies is just expense recording.

5. **No mention of intercompany expense handling.** If Technica has multiple legal entities (common in international manufacturing), employees traveling for one entity but employed by another will need intercompany expense allocation. This is standard in D365 but must be configured. Verify if this scenario applies.

6. **Vendor settlement process for employee reimbursement is described but payment method is not.** After the expense report is posted and the vendor (employee) balance is updated, how is the employee actually paid? Options in D365:
   - AP payment journal (manual)
   - AP payment proposal (batch)
   - Integration with payroll (add to next paycheck)

   The FRD stops at "settle transactions" but does not describe the actual disbursement. This should be defined.

---

## Efficiency Verdict

The Expense Management solution is **functionally adequate and makes appropriate use of D365 F&O standard capabilities.** The core processes -- Travel Requisition, Cash Advance, and Expense Report -- are well-understood by the implementer and correctly documented with step-by-step screenshots.

**What works well:**
- Standard D365 Expense Management module is correctly leveraged for all three sub-processes
- Cash advance account structure (per currency, with employee financial dimension) is well-designed
- Cash advance return and partial reconciliation are properly handled
- Project expense integration is correctly identified and described
- Mobile app capability (EM001-003) is acknowledged as a FIT -- high value for traveling employees
- Per diem setup by country with hotel/meal/other breakdown follows best practice

**What needs improvement:**
- **CRM integration design is the weakest part** -- superficial description, undefined mechanism, incomplete field mapping, and no error handling. This must be elevated to a dedicated integration design document.
- **The "Charged to Customer" GAP needs end-to-end design**, not just a checkbox. The debit note generation process must be defined and linked to the AR module.
- **Expense policies are entirely absent.** This is a significant functional gap in the FRD itself -- Technica's internal policies should drive D365 policy configuration, and the FRD should at least outline the policy framework.
- **The 8 missing data items from Technica** suggest the workshops were incomplete. A focused follow-up session is needed to collect this data before configuration begins.

**Comparison with other FRDs in this set:** This FRD is **less mature than FRD04 (Project Accounting)** and **FRD07 (Procurement)**, which are more detailed and complete. It is comparable in maturity to FRD01 (Sales), which also has undefined integration architecture. The FRD reads more as a setup guide than a functional requirements document -- it explains how D365 works rather than specifying what Technica needs. The requirements table at the end (9 items) is thin relative to the process complexity.

**Final assessment:** Proceed with the standard configuration for Travel Requisition, Cash Advance, and Expense Report. **Do not begin GAP development** until: (a) the CRM integration architecture is formally defined across FRD01 and FRD12, (b) Technica provides the 8 missing data items, and (c) the "Charged to Customer" requirement is fully specified end-to-end.

---

*This review is based on D365 F&O Expense Management best practices as of 2026. Recommendations aim to reduce customization, close design gaps, and ensure the solution is production-ready.*
