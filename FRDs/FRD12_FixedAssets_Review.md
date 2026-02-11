# FRD12 - Fixed Assets - Solution Design Verification Report

**Document:** Technica - FRD12 - Fixed Assets - V2.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Fixed Assets Module

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Cross-FRD Integration | Project capitalization to fixed asset process is completely missing — add FA-001-07 sub-process | [Integration with FRD04](#integration-with-frd04---project-accounting-investment-sub-projects) |
| 2 | **CRITICAL** | FA-001-03 Acquisition | No capitalization threshold or low-value asset policy defined | [FA-001-03: Fixed Asset Acquisition](#fa-001-03-fixed-asset-acquisition) |
| 3 | **HIGH** | FA-001-04 Depreciation | Undefined second depreciation book — resolve before UAT | [FA-001-04: Fixed Asset Depreciation](#fa-001-04-fixed-asset-depreciation) |
| 4 | **HIGH** | FA-001-01 Creation via PO | Procurement category-to-FA group mapping deferred too late — complete during design phase | [FA-001-01: Fixed Asset Creation Automatic via PO Receipt](#fa-001-01-fixed-asset-creation-automatic-via-po-receipt) |
| 5 | **HIGH** | FA-001-02 Manual Creation | Define explicit business rules for inventory-to-asset transfers (triggers, timeframe, approval) | [FA-001-02: Fixed Asset Creation Manually with Stocked Item](#fa-001-02-fixed-asset-creation-manually-with-stocked-item) |
| 6 | **MEDIUM** | FA-001-03 Acquisition | Financial dimension enforcement should be mandatory on FA book, not fail at posting | [FA-001-03: Fixed Asset Acquisition](#fa-001-03-fixed-asset-acquisition) |
| 7 | **MEDIUM** | FA-001-03 Acquisition | Define depreciation start date policy (acquisition date vs. put-in-service date) | [FA-001-03: Fixed Asset Acquisition](#fa-001-03-fixed-asset-acquisition) |
| 8 | **MEDIUM** | FA-001-05 Disposal | Confirm partial disposal requirements | [FA-001-05: Fixed Asset Disposal](#fa-001-05-fixed-asset-disposal-sales--scrap) |
| 9 | **MEDIUM** | FA-002 Reports | Build depreciation forecast and capex vs budget Power BI reports | [FA-002: Inquiries & Reports](#fa-002---inquiries--reports) |
| 10 | **LOW** | Setup | Add Low-Value Assets group with immediate or accelerated write-off | [Fixed Asset Groups](#fixed-asset-groups) |
| 11 | **LOW** | Setup | Clarify Vehicles vs Transportation Equipment groups — document classification criteria | [Fixed Asset Groups](#fixed-asset-groups) |

**Totals:** 2 CRITICAL | 3 HIGH | 4 MEDIUM | 2 LOW

---

## Executive Summary

This FRD covers the Fixed Assets lifecycle for Technica International, spanning two main process areas: **FA-001 (Asset Creation to Acquisition to Disposal)** with six sub-processes, and **FA-002 (Inquiries & Reports)** with seven standard reports. The solution is **almost entirely FIT** -- every single requirement across both process areas maps to standard D365 F&O functionality with zero GAPs identified.

The design is functionally correct and demonstrates solid understanding of D365 Fixed Assets capabilities: automatic asset creation from PO receipt, inventory-to-asset transfers, straight-line depreciation, journal-based disposal (scrap and sale via free text invoice), and financial dimension transfers. Workflow approvals are applied to acquisition, depreciation, and disposal journals, which is appropriate for a manufacturing company with financial governance needs.

However, the document is **notably thin compared to the other FRDs** in this set. It lacks depth in several critical areas: there is no mention of low-value or below-threshold asset handling, no integration with the project capitalization process described in FRD04 (investment sub-projects), no mention of asset impairment, revaluation, or insurance valuation, and the second depreciation book remains undefined. The FRD reads more like a configuration walkthrough than a requirements document. While the "all FIT" status is technically accurate, it masks **significant design gaps that need to be addressed before go-live**.

Overall assessment: **Functionally sound but incomplete. The solution works for basic fixed asset operations but needs supplementation for a manufacturing company of Technica's complexity.**

---

## Process-by-Process Review

### FA-001 - Fixed Asset Creation to Acquisition to Disposal

#### FA-001-01: Fixed Asset Creation Automatic via PO Receipt

**What was proposed:**
- Enable "Create a new fixed asset during product receipt or invoice posting" in FA parameters
- Configure "Business rules for fixed asset determination" linking procurement categories to FA groups
- On PO product receipt, the system auto-creates the asset record
- On PO invoice posting, the asset is auto-acquired (status = "Acquired")
- Quantity handling: Q=3 on one line creates 1 asset; 3 lines of Q=1 creates 3 assets

**Assessment: Standard OOB -- correctly configured**

| Aspect | Rating | Comment |
|--------|--------|---------|
| FA parameter setup | Correct | Standard toggle for auto-creation |
| Procurement category to FA group mapping | **>> NEEDS REVIEW <<** | Data dependency on Technica team — deferred too late |
| Asset creation on receipt | Standard OOB | Asset number generated at receipt |
| Acquisition on invoice | Standard OOB | Proper two-step process |
| Quantity handling explanation | Good | Important user-facing detail documented |
| Purchase Requisition flag behavior | Good | Lock behavior when PR flag is not set is noted |

**>> ATTENTION AREA: Procurement category mapping deferred to migration phase**

> The FRD states the procurement category-to-FA group mapping will be provided "during the migration phase." This is too late. Incorrect or incomplete mappings will cause PO receipts to fail to create assets or to assign them to wrong groups. This mapping is a critical data dependency for the primary acquisition method.

**Recommendations:**

1. **>> ENHANCEMENT:** **Procurement category mapping is a critical data dependency.** The FRD states this will be provided "during the migration phase." This is too late. Incorrect or incomplete mappings will cause PO receipts to fail to create assets or to assign them to wrong groups. **Recommendation:** Complete this mapping during the design/build phase and validate it during Conference Room Pilot (CRP), not migration.

2. **The quantity behavior is a known user confusion point.** The FRD correctly documents that Q=3 on one line creates a single asset record. However, in practice, users frequently make this mistake when ordering multiple identical assets (e.g., 3 laptops). **Recommendation:** Add a validation or at minimum a warning message when a fixed-asset-flagged PO line has quantity > 1. Alternatively, document this clearly in the user training materials with worked examples.

3. **Missing: What happens when the procurement category is not mapped?** The FRD mentions "If the setup was missing some procurement category this will not be enabled automatically" but does not define the fallback process. **Recommendation:** Define an explicit process for handling unmapped categories -- either a manual asset creation step or a periodic review of POs that should have created assets but did not.

---

#### FA-001-02: Fixed Asset Creation Manually with Stocked Item

**What was proposed:**
- Items received into inventory via normal purchase cycle (FA auto-creation disabled)
- After a delay (e.g., 10 days), asset is created manually without acquisition
- Inventory-to-fixed-asset journal transfers stock out and acquires the asset
- Requires setup of inventory journal name with type "Fixed asset"

**Assessment: Correct process for inventory-to-asset conversion**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Inventory journal name setup | Correct | Journal type "Fixed asset" |
| Inventory-to-FA journal process | Standard OOB | Stock reduction + asset acquisition |
| Cost transfer accuracy | Good | Asset acquired at inventory cost |
| Business rules for transfer triggers | **>> NEEDS IMPROVEMENT <<** | Vague criteria — "sometimes" is not a business rule |
| Responsible person assignment | Noted | Post-acquisition assignment mentioned |

**>> ATTENTION AREA: Undefined business rules for inventory-to-asset transfers**

> The FRD language "sometimes they receive a product into the stock then move it as fixed asset" is vague. This process needs clearer business rules: Under what circumstances does this happen? Who decides? What is the approval process? Without clear criteria, items may sit in inventory indefinitely before being capitalized, creating accounting inaccuracies.

**Recommendations:**

1. **>> ENHANCEMENT:** **The "sometimes they receive a product into the stock then move it as fixed asset" language is vague.** This process needs clearer business rules: Under what circumstances does this happen? Who decides? What is the approval process? Without clear criteria, items may sit in inventory indefinitely before being capitalized, creating accounting inaccuracies. **Recommendation:** Define specific triggers (e.g., items above a capitalization threshold, items in specific procurement categories) and a maximum timeframe for the inventory-to-asset transfer.

2. **>> ENHANCEMENT:** **The FRD states "this process always starts by the project module."** This is a critical statement that is not elaborated. If stocked items destined for fixed assets flow through Project Accounting, this is directly related to the FRD04 investment sub-project process (project type "Investment" for asset capitalization). **Recommendation:** Explicitly define the end-to-end flow: Project PO receipt -> Inventory -> Project cost -> Asset capitalization. This cross-FRD linkage must be designed as a single integrated process.

3. **Valuation risk during the inventory holding period.** If an item sits in inventory for 10+ days before becoming a fixed asset, inventory costing adjustments during that period affect the final asset cost. **Recommendation:** Ensure FIFO/standard cost methodology is aligned and that the inventory-to-FA journal captures the correct cost at the time of transfer, not the original PO cost if adjustments have occurred.

---

#### FA-001-03: Fixed Asset Acquisition

**What was proposed:**
- FA acquisition journal with workflow: Accountant submits -> Chief Accountant approves and posts
- Manual journal creation with acquisition transaction type
- Acquisition proposal for bulk assets
- Financial dimensions must be filled in the asset book after acquisition
- Depreciation will not post without financial dimensions on the asset book

**Assessment: Standard OOB with proper workflow**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Acquisition journal workflow | Good | Proper segregation of duties |
| Acquisition proposal (bulk) | Standard OOB | Efficient for mass acquisitions |
| Financial dimension requirement | **>> NEEDS REVIEW <<** | Enforcement is reactive (fails at posting), not proactive |
| Capitalization threshold | **>> NEEDS IMPROVEMENT <<** | Not defined — fundamental accounting policy missing |
| Depreciation start date policy | **>> NEEDS REVIEW <<** | Not addressed — acquisition date vs. put-in-service date |
| Balance inquiry after acquisition | Good | User verification step documented |

**>> ATTENTION AREA: No capitalization threshold or low-value asset policy defined**

> The FRD does not define a minimum value for capitalizing assets. Most jurisdictions and accounting policies have a threshold below which items are expensed directly. Without this, users will inconsistently capitalize or expense similar items, leading to audit findings.

**>> ATTENTION AREA: Financial dimension enforcement is reactive, not proactive**

> The FRD states that if dimensions are not filled, "the depreciation transaction will not get posted." This is a reactive control (fails at posting time) rather than a proactive one. The issue should be caught at data entry, not at depreciation time.

**Recommendations:**

1. **>> ENHANCEMENT:** **Financial dimension enforcement is critical but fragile as described.** The FRD states that if dimensions are not filled, "the depreciation transaction will not get posted." This is a reactive control (fails at posting time) rather than a proactive one. **Recommendation:** Configure the financial dimension framework to make the relevant dimensions (Department, Cost Center) **mandatory on the fixed asset book** form. This prevents the record from being saved without dimensions, catching the issue at data entry rather than at depreciation time.

2. **>> ENHANCEMENT (CRITICAL):** **Missing: Capitalization threshold / low-value asset policy.** The FRD does not define a minimum value for capitalizing assets. Most jurisdictions and accounting policies have a threshold below which items are expensed directly. **Recommendation:** Define Technica's capitalization threshold (e.g., items below USD 500 or equivalent are expensed) and document how low-value assets are handled -- either as expense items or in a separate "Low-Value Assets" group with immediate or accelerated write-off.

3. **>> ENHANCEMENT:** **Missing: Acquisition date vs. put-in-service date.** D365 supports separate acquisition date and depreciation start date. The FRD does not address whether assets start depreciating from the acquisition date, the receipt date, or a separate put-in-service date. For manufacturing equipment, the time between receipt and commissioning can be weeks or months. **Recommendation:** Define the depreciation start date policy and configure the depreciation convention accordingly (mid-month, full-month, actual date).

---

#### FA-001-04: Fixed Asset Depreciation

**What was proposed:**
- Depreciation profile: "Straight" using Straight Line Life Remaining method
- Monthly depreciation run
- Depreciation journal with workflow: Accountant submits -> Chief Accountant approves and posts
- Depreciation proposal auto-generates entries for all eligible assets
- Financial dimensions auto-populate on offset account from asset book
- System suggests depreciation amount; can be manually overridden

**Assessment: Standard OOB -- correctly configured**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Straight Line Life Remaining method | Appropriate | Good for transferred assets with remaining life |
| Monthly depreciation run | Standard | Matches typical accounting periods |
| Depreciation proposal automation | Standard OOB | Efficient batch processing |
| Manual override capability | Standard OOB | Adjusts remaining life amounts |
| Workflow approval | Good | Proper control |
| Financial dimension auto-population | Good | Reduces manual entry errors |
| Second depreciation book | **>> NEEDS REVIEW <<** | Undefined — critical open item with cascading impact |

**>> ATTENTION AREA: Undefined second depreciation book**

> The FRD explicitly states: "The other book is still under checking internally by Technica finance manager and he shall get back to us with the findings." This is a significant open item. If it is a tax book, the posting layer configuration, depreciation profile, service lives, and posting profiles all need separate setup. If Technica's finance manager has not decided by the time configuration begins, it will cause rework.

**Recommendations:**

1. **"Straight Line Life Remaining" is appropriate for migrated assets but consider "Straight Line Service Life" for new assets.** The "Life Remaining" method calculates depreciation based on the net book value divided by the remaining useful life. This is ideal for migrated assets where the original cost, accumulated depreciation, and remaining life are known. However, for newly acquired assets, "Straight Line Service Life" (cost / total useful life) is more conventional and predictable. **Recommendation:** Evaluate whether both methods are needed -- one for migrated assets and one for new acquisitions -- or if "Life Remaining" is intentionally chosen for all assets.

2. **>> ENHANCEMENT:** **The second depreciation book is undefined.** The FRD explicitly states: "The other book is still under checking internally by Technica finance manager and he shall get back to us with the findings." This is a significant open item. Common reasons for a second book include:
   - Tax depreciation (different rates/methods from accounting)
   - IFRS vs. local GAAP differences
   - Management/internal reporting
   - Insurance valuation

   **Recommendation:** This must be resolved before go-live configuration. If it is for tax depreciation, the posting layer (Current vs. Tax) must be configured. If unused, remove the placeholder to avoid confusion.

3. **Missing: Depreciation run calendar and cutoff procedures.** The FRD says "monthly basis" but does not define: Who initiates the monthly run? What is the cutoff date? Is there a period-end checklist? What happens if a depreciation run is missed for a period? **Recommendation:** Document the monthly depreciation process as part of the period-end close procedure, including the responsible role, deadline, and exception handling.

4. **Manual override risk.** The FRD mentions that depreciation amounts can be changed manually and the system will recalculate remaining amounts. This is a legitimate feature but creates audit risk. **Recommendation:** Restrict manual override capability to a senior role (e.g., Chief Accountant only) via security role configuration.

---

#### FA-001-05: Fixed Asset Disposal (Sales & Scrap)

**What was proposed:**
- **Scrap:** Disposal journal with transaction type "Disposal - scrap" + workflow approval
  - Proposal auto-populates eligible assets
  - Offset account from posting profile (disposal section)
  - No value entered in debit/credit for full disposal
  - Status updates to "Scrapped" with disposal date
- **Sale:** Free text invoice with fixed asset reference
  - Customer account + invoice date + unit price of sale
  - Main account auto-populated from FA posting profile
  - Asset must be open and acquired
  - Status updates to "Sold"
  - Full transaction trail: acquisition reversal, accumulated depreciation reversal, gain/loss

**Assessment: Standard OOB -- both disposal methods correctly described**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Scrap via disposal journal | Standard OOB | With workflow approval |
| Sale via free text invoice | Standard OOB | Correct use of FTI for non-trade sales |
| Posting profile for disposal accounts | Correct | Separate disposal P&L setup |
| Full disposal (no partial) | **>> NEEDS REVIEW <<** | Partial disposal not addressed — see recommendation |
| Status tracking (Scrapped/Sold) | Standard OOB | Automatic status update |
| Gain/loss recognition | Standard OOB | Auto-calculated from NBV vs. sale price |

**>> ATTENTION AREA: Partial disposal not addressed**

> The FRD only covers full asset disposal. In manufacturing, partial disposals are common: selling one component of a production line, or scrapping a portion of a building. D365 supports partial disposal through "Disposal - sale" and "Disposal - scrap" transaction types with specific amounts.

**Recommendations:**

1. **>> ENHANCEMENT:** **Partial disposal is not addressed.** The FRD only covers full asset disposal. In manufacturing, partial disposals are common: selling one component of a production line, or scrapping a portion of a building. D365 supports partial disposal through "Disposal - sale" and "Disposal - scrap" transaction types with specific amounts. **Recommendation:** Confirm with Technica whether partial disposals occur and document the process if they do.

2. **Free text invoice for asset sale is correct but has a limitation.** Free text invoices do not support item numbers or sales tax groups from item masters. If Technica needs to charge VAT on asset sales (common in many jurisdictions), the tax group must be manually set on the FTI line. **Recommendation:** Configure a default sales tax group for asset disposal FTI lines or create a specific FTI template for asset sales.

3. **Missing: Disposal approval beyond the journal workflow.** The FRD describes the journal posting workflow (Accountant -> Chief Accountant), but asset disposal typically requires pre-approval from management (e.g., department head, CFO) before the accounting entry is made. **Recommendation:** Add a pre-disposal approval step -- this could be a Power Automate approval or a simple requisition process before the disposal journal is created.

4. **Missing: Asset retirement obligation.** For industrial equipment, there may be decommissioning or environmental cleanup obligations that should be recognized when the asset is disposed. **Recommendation:** Confirm with Technica whether any asset categories carry retirement obligations that need accounting treatment.

---

#### FA-001-06: Fixed Asset Transfer

**What was proposed:**
- Transfer financial dimension values in the asset book from one department to another
- Transfer history tracked with comments
- Historical data preserved in transfer history inquiry

**Assessment: Standard OOB -- simple and effective**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Financial dimension transfer | Standard OOB | Department-to-department |
| Transfer history inquiry | Standard OOB | Full audit trail |
| Transfer comments | Good | Documentation of reason |

**Recommendations:**

1. **Transfer scope is limited to financial dimensions only.** The FRD does not address physical asset transfers (site-to-site, warehouse-to-warehouse) which are common in manufacturing companies with multiple facilities. D365 supports asset transfers across legal entities and across financial dimensions. **Recommendation:** Confirm whether Technica has multiple sites/locations that require physical asset transfers with corresponding accounting entries. If Technica operates from a single site, this is not needed.

2. **Missing: Transfer approval workflow.** The FRD does not mention whether asset transfers require approval. Transferring an asset between departments affects departmental P&L (depreciation charges). **Recommendation:** Add a transfer approval workflow, particularly for high-value assets.

---

### FA-002 - Inquiries & Reports

**What was proposed:**
Seven OOB reports/inquiries:
1. Fixed asset transactions
2. Fixed asset listing report
3. Fixed asset acquisitions
4. Fixed asset disposal
5. Fixed asset movements
6. Fixed asset statement
7. Fixed asset insurance report

**Assessment: All standard OOB reports -- adequate baseline**

| Report | OOB? | Adequacy | Comment |
|--------|------|----------|---------|
| Fixed asset transactions | Yes | Good | Drill-down to voucher level |
| Fixed asset listing | Yes | Good | Master data overview |
| Fixed asset acquisitions | Yes | Good | Period-based acquisition tracking |
| Fixed asset disposal | Yes | Good | Disposal tracking |
| Fixed asset movements | Yes | Good | Addition/disposal/transfer summary |
| Fixed asset statement | Yes | Good | NBV and accumulated depreciation |
| Fixed asset insurance | Yes | Adequate | Requires insurance data maintenance |
| Depreciation forecast report | No | **>> NEEDS IMPROVEMENT <<** | Missing — essential for budgeting |
| Capex vs. budget report | No | **>> NEEDS IMPROVEMENT <<** | Missing — essential for capital planning |

**>> ATTENTION AREA: Missing management reports for a manufacturing company**

> The report scope covers only standard OOB reports. Key management reports for depreciation forecasting, NBV aging, and capital expenditure vs. budget are absent. These are essential for a manufacturing company's budgeting and capital planning processes.

**Recommendations:**

1. **>> ENHANCEMENT:** **Missing reports for a manufacturing company:**
   - **Asset register by location/department:** Critical for physical verification and departmental cost allocation. The standard listing report can be filtered but a dedicated view is more practical.
   - **Depreciation forecast/projection report:** Shows future depreciation expense by period. Essential for budgeting (cross-reference with FRD12 Budgeting).
   - **Asset NBV aging report:** Assets by age band showing NBV, useful for capital planning.
   - **Capital expenditure vs. budget report:** Actual acquisitions vs. planned capex.

   **Recommendation:** Build these as Power BI reports using the Fixed Assets data entities, not as SSRS customizations. This aligns with the approach recommended across other FRDs.

2. **The insurance report is included but insurance valuation is not discussed anywhere in the setup.** If Technica maintains insurance values for assets, a separate insurance value model or book may be needed. **Recommendation:** Clarify whether the insurance report is actually required and, if so, how insurance values are maintained (manual entry, indexed adjustment, etc.).

---

## Setup Review

### Fixed Asset Groups

Nine groups defined: Computer Software, Constructions, Industrial Equipment, Industrial Tools, Vehicles, General Installations, Office and Computer Equipment, Office Furniture, Transportation Equipment.

| Observation | Recommendation |
|-------------|---------------|
| Groups are well-categorized for a manufacturing company | Adequate |
| Each group has a unique number sequence | Good practice for asset identification |
| Service life is defined per group (in years, supports decimals) | Standard approach |
| Missing: "Low-Value Assets" group | **>> NEEDS IMPROVEMENT <<** **Add a group for assets below the capitalization threshold with a 1-year or immediate write-off life** |
| Missing: "Leased Assets" group | If Technica has any operating/finance leases, a separate group is needed (IFRS 16 compliance) |
| "Transportation Equipment" vs. "Vehicles" overlap | **>> NEEDS REVIEW <<** **Clarify the distinction** -- are vehicles for personnel transport and transportation equipment for goods? If so, document the classification criteria. |

### Depreciation Profile

Single profile "Straight" with Straight Line Life Remaining method.

| Observation | Recommendation |
|-------------|---------------|
| Single depreciation method for all asset groups | Appropriate if local regulations permit |
| Straight Line Life Remaining | Good for migration; verify suitability for new assets |
| Monthly depreciation period | Standard |
| Missing: Depreciation conventions | **Define convention (mid-month, full-month, half-year) for the first and last depreciation period** |

### Depreciation Books

One confirmed book (AccDep - Accounting Depreciation). One undefined book pending Technica finance manager review.

| Observation | Recommendation |
|-------------|---------------|
| AccDep book is properly configured | Good |
| Second book is undefined | **>> NEEDS REVIEW <<** **Critical open item -- resolve before UAT** |
| Posting layer not mentioned | Confirm AccDep posts to "Current" layer |
| No tax book defined | If local tax depreciation rates differ from accounting rates, a tax book is required |

### Posting Profile

Single posting profile "ALL" applied across all books and groups.

| Observation | Recommendation |
|-------------|---------------|
| Single posting profile simplifies setup | Appropriate for current scope |
| Account details deferred to migration phase | Ensure chart of accounts supports all required FA posting types |
| Disposal posting (scrap and sale) configured separately | Good practice |

---

## Cross-FRD Integration Analysis

### Integration with FRD04 - Project Accounting (Investment Sub-Projects)

**>> ATTENTION AREA: Project capitalization to fixed asset process is completely missing**

> FRD04 explicitly describes the use of "Investment" project type sub-projects for tracking asset capitalization. This means project costs (labor, materials, expenses) accumulated on an investment sub-project are eventually capitalized as a fixed asset. This is the **single largest gap in the document** and must be addressed before go-live. Without it, project costs intended for capitalization will have no defined path into the fixed asset register.

| Integration Point | Status in FRD12 | Risk |
|-------------------|----------------|------|
| Project cost capitalization to fixed asset | Not mentioned | **High** -- process gap |
| Investment sub-project -> Asset creation | Not mentioned | **High** -- no defined trigger |
| Capitalized cost calculation (which costs qualify) | Not mentioned | **Medium** -- accounting policy gap |
| Asset group assignment for project-capitalized assets | Not mentioned | **Medium** -- classification gap |

**>> ENHANCEMENT (CRITICAL):** Add a new sub-process (FA-001-07) covering "Fixed Asset Acquisition from Project Capitalization" that defines:
- When an investment sub-project is complete, how is the fixed asset created?
- Is it manual creation + acquisition journal, or does D365's "Fixed asset - credit project" estimate mechanism handle it?
- Which project cost categories are capitalizable (labor, materials, overhead)?
- How are capitalized assets assigned to FA groups?
- What is the approval workflow for project capitalization?

### Integration with FRD07 - Procurement and Sourcing

The automatic asset creation via PO receipt (FA-001-01) depends on procurement category-to-FA group mappings maintained in the Procurement module. This integration is documented but the data dependency is deferred.

### Integration with FRD10 - Asset Management & Maintenance (Out of Scope)

FRD10 (Asset Management & Maintenance) is marked as out of scope. This means there is **no link between fixed asset records (financial) and maintenance asset records (operational)**. In D365, the Asset Management module can be linked to Fixed Assets, enabling maintenance cost tracking against the financial asset record.

**Recommendation:** Even though FRD10 is out of scope for this phase, design the fixed asset group structure and numbering to be compatible with a future Asset Management implementation. Use consistent naming and ensure the asset number sequence can be referenced by the Asset Management module later.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | FA-001-01 | Complete procurement category-to-FA group mapping during design phase, not migration | High | Prevents PO receipt failures |
| 2 | FA-001-01 | Add validation or warning when fixed-asset PO line has quantity > 1 | Medium | Prevents user errors |
| 3 | FA-001-02 | Define explicit business rules for when inventory items become fixed assets (triggers, timeframe, approval) | High | Prevents accounting inaccuracies |
| 4 | FA-001-02 | Document end-to-end flow linking FRD04 investment sub-projects to asset capitalization | High | Closes critical process gap |
| 5 | FA-001-03 | Make financial dimensions mandatory on FA book form, not just fail at depreciation posting | Medium | Proactive vs. reactive control |
| 6 | FA-001-03 | Define capitalization threshold and low-value asset policy | High | Fundamental accounting policy |
| 7 | FA-001-03 | Define depreciation start date policy (acquisition date vs. put-in-service date) | Medium | Affects depreciation accuracy |
| 8 | FA-001-04 | Resolve the undefined second depreciation book before UAT | High | Configuration dependency |
| 9 | FA-001-04 | Evaluate whether "Life Remaining" method is appropriate for new assets or if "Service Life" is better | Medium | Depreciation accuracy |
| 10 | FA-001-04 | Restrict manual depreciation override to senior roles via security configuration | Low | Audit control |
| 11 | FA-001-05 | Confirm whether partial disposals are required and document the process | Medium | Process completeness |
| 12 | FA-001-05 | Configure default sales tax group for asset sale free text invoices | Medium | Tax compliance |
| 13 | FA-001-06 | Confirm whether physical asset transfers (site-to-site) are needed | Low | Process completeness |
| 14 | FA-002 | Build depreciation forecast, NBV aging, and capex vs. budget reports in Power BI | Medium | Management reporting |
| 15 | Setup | Add "Low-Value Assets" group with immediate or accelerated write-off | High | Accounting completeness |
| 16 | Setup | Clarify distinction between "Vehicles" and "Transportation Equipment" groups | Low | Classification clarity |
| 17 | Setup | Define depreciation conventions for first/last period | Medium | Depreciation accuracy |
| 18 | Integration | Define FA-001-07 sub-process for project capitalization to fixed asset | High | Closes biggest design gap |

---

## Risk Flags

1. **Project capitalization to fixed asset process is completely missing.** FRD04 describes investment sub-projects for asset capitalization, but FRD12 has no corresponding process for receiving these capitalized assets. This is the single largest gap in the document and must be addressed before go-live. Without it, project costs intended for capitalization will have no defined path into the fixed asset register.

2. **The undefined second depreciation book is a deferred decision with cascading impact.** If it is a tax book, the posting layer configuration, depreciation profile, service lives, and posting profiles all need separate setup. If Technica's finance manager has not decided by the time configuration begins, it will cause rework. Escalate this decision.

3. **No capitalization threshold or low-value asset policy is defined.** This is a fundamental accounting policy that affects every asset acquisition. Without it, users will inconsistently capitalize or expense similar items, leading to audit findings.

4. **Procurement category mapping deferred to migration phase.** If this mapping is incomplete or incorrect, the automatic asset creation from PO receipts (the primary acquisition method) will fail silently -- POs will be received without creating assets, and the error will only be discovered during reconciliation or audit.

5. **The "all FIT / zero GAP" assessment may create a false sense of completeness.** While it is true that every described requirement maps to OOB functionality, the document omits several processes that a manufacturing company typically needs (project capitalization, low-value assets, partial disposal, depreciation forecasting). The absence of GAPs does not mean the solution is complete.

6. **No mention of asset impairment testing.** Under IAS 36, assets must be tested for impairment when indicators exist. For industrial equipment in a make-to-order manufacturer, impairment triggers (technology obsolescence, market decline) are realistic. D365 supports write-down adjustments, but the process and triggers are not defined.

---

## Efficiency Verdict

The Fixed Assets FRD is the **most straightforward and least complex document** in the Technica FRD set. It correctly leverages standard D365 F&O Fixed Assets functionality and avoids unnecessary customization -- every requirement is a FIT, which is commendable. The core asset lifecycle (creation, acquisition, depreciation, disposal, transfer) is properly described with appropriate workflow controls.

However, the document suffers from **significant omissions rather than design flaws.** The three most critical gaps are:

1. **No integration with project capitalization** (FRD04 investment sub-projects), which is likely a primary source of high-value asset creation for a make-to-order manufacturer.
2. **No capitalization threshold or low-value asset policy**, which is a fundamental accounting requirement.
3. **An unresolved second depreciation book**, which may represent tax or regulatory reporting requirements.

The FRD reads as a configuration guide for standard Fixed Assets functionality rather than a comprehensive requirements document for a manufacturing company. It covers the "how" (step-by-step journal entries, form screenshots) but insufficiently addresses the "what" (business policies, decision criteria, edge cases) and the "why" (accounting standards compliance, reporting needs).

**Recommended next steps:**
- Conduct a focused workshop with Technica's finance manager to resolve the three critical gaps listed above
- Add sub-process FA-001-07 for project capitalization
- Define the capitalization threshold and low-value asset policy
- Resolve the second depreciation book decision
- Build Power BI reports for depreciation forecasting and capital expenditure tracking
- Validate the solution end-to-end during CRP with real asset data spanning all acquisition methods (PO, manual, inventory transfer, project capitalization)

---

*This review is based on D365 F&O Fixed Assets module best practices as of 2026. Recommendations aim to close process gaps, ensure accounting policy completeness, and align with the broader Technica implementation (particularly FRD04 Project Accounting).*
