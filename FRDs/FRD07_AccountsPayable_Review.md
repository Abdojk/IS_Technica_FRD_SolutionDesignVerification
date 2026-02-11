# FRD07 - Accounts Payable - Solution Design Verification Report

**Document:** Technica - FRD07 - Accounts Payable - V2.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Accounts Payable, Vendor Payments, Invoice Matching
**Cross-FRD Dependencies:** Procurement & Sourcing (FRD07-Procurement), Project Accounting (FRD04), Logistics (FRD06), GL & Cash/Bank (FRD12-GL)

---

## Executive Summary

This FRD covers four core Accounts Payable processes: PO-based Invoice Processing (AP001), Non-PO Invoice Processing (AP002), Vendor Payment Processing (AP003), and Down Payment / Advance Payment handling (AP004). The document also covers reporting requirements and setup/configuration parameters.

The solution makes **good use of D365 standard AP capabilities** -- invoice matching (2-way and 3-way), vendor invoice workflows, payment journals with approval, and invoice registers for non-PO invoices. However, there are several areas where the design introduces **unnecessary complexity or misses standard D365 features** that would simplify the process. Notably, the handling of advance payments uses a manual journal approach where D365's standard **Vendor Prepayment** functionality would be cleaner, and several notification GAPs could be eliminated with standard D365 Alerts or Print Management.

The document identifies approximately **12 GAP items** across all processes. Upon review, **3-4 of these may already be achievable OOB** or with minimal configuration (not customization). The remaining GAPs are low-to-medium complexity, with no single GAP requiring significant custom development.

Overall assessment: **Solid AP foundation using mostly standard features, but with room to simplify advance payment handling and eliminate unnecessary GAPs by leveraging OOB functionality more aggressively.**

---

## Process-by-Process Review

### AP001 - PO Invoice Processing (3 GAPs)

**What was proposed:**
- Vendor invoice received and matched against PO (2-way or 3-way matching depending on item model group)
- Invoice matching validation: price tolerance, quantity tolerance, charge matching
- Match variance resolution workflow -- discrepancies routed to Procurement for review
- Posting invoice only after successful match
- Workflow approval for vendor invoices above threshold
- Automatic charge allocation from PO to invoice
- Financial dimensions defaulting from PO lines
- Project-related PO invoices post cost to project
- Credit note processing (negative invoice against original)
- Intercompany invoice handling for shared services

**Assessment: Strong standard implementation with minor gaps**

| Aspect | Rating | Comment |
|--------|--------|---------|
| 2-way / 3-way matching | Standard OOB | Correct use of item model group to drive matching policy |
| Price tolerance setup | Standard OOB | Percentage and amount-based tolerances |
| Invoice matching workflow | Standard OOB | Route discrepancies for approval |
| Charge matching | Standard OOB | PO charges flow to invoice |
| Financial dimensions from PO | Standard OOB | Proper dimension inheritance |
| Project cost posting | Standard OOB | Via PO project reference (links to FRD04) |
| Credit note processing | Standard OOB | Negative invoice with original reference |
| Vendor invoice workflow | Standard OOB | Amount-based or vendor-group-based routing |

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|------------|----------------|
| AP001-GAP01 | Email notification to requester when invoice is posted against their PO | Low | **Likely OOB via D365 Alerts.** Set up a change-based alert on invoice posting filtered by PO originator. If more detail is needed in the email body, use **Power Automate** triggered on invoice journal posting. Do NOT build custom X++ for this. |
| AP001-GAP02 | Custom fields on vendor invoice form (commercial invoice number, customs declaration reference) | Low | Standard form extension via **custom fields (no-code)** in D365. Use the "Add columns" feature in the Vendor Invoice form. If these fields need to print on reports, a minor SSRS modification is needed. Estimate: 4-8 hours. |
| AP001-GAP03 | Automatic matching validation notification -- alert Procurement when match fails | Low | **Standard OOB.** Invoice matching workflow already routes match failures to a review queue. Configure the workflow to send email notifications to the Procurement group on assignment. No customization needed -- this is a workflow configuration, not a GAP. **Reclassify as FIT.** |

**Recommendations:**

1. **AP001-GAP03 should be reclassified as FIT.** The D365 vendor invoice workflow has built-in work item assignment notifications. When a match variance is detected, the workflow routes the invoice to a review queue and can send email/Teams notifications to the assigned reviewer. This is configuration, not customization.

2. **3-way matching policy alignment with Procurement FRD:** The Procurement FRD (FRD07-Procurement, PR001) already specifies 3-way matching. Ensure the matching policy is set at the **Vendor level or Item Model Group level** (not per-PO), so it applies consistently. The AP and Procurement teams must agree on a single matching policy matrix.

3. **Project PO invoice posting** correctly flows costs to the project (linking to FRD04 PJ002). Ensure the **auto-consumption on receipt** setting from the Procurement FRD aligns with when AP posts the invoice. If auto-consumption is enabled, the item is consumed on product receipt, and the invoice only updates the financial voucher. If disabled, consumption happens at invoice posting. This must be consistent.

4. **Credit note processing** should use the standard "Create credit note" function from the original invoice, not a manual negative invoice. This preserves the linkage and simplifies settlement.

---

### AP002 - Non-PO Invoice Processing (3 GAPs)

**What was proposed:**
- Invoice register for initial capture (date, vendor, amount, description)
- Invoice approval journal for departmental review and coding
- Workflow approval based on amount thresholds and expense type
- Financial dimension assignment by approver
- Recurring invoices for fixed monthly charges (rent, utilities, retainers)
- Prepayment invoices without PO
- Intercompany cost allocation

**Assessment: Standard non-PO invoice handling with some design concerns**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Invoice register | Standard OOB | Initial capture journal |
| Invoice approval journal | Standard OOB | Proper 2-step process |
| Workflow approval | Standard OOB | Configurable thresholds |
| Financial dimension coding | Standard OOB | At approval stage |
| Recurring invoices | Standard OOB | Periodic journal with recurrence |
| Intercompany allocation | Standard OOB | Via intercompany accounting |

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|------------|----------------|
| AP002-GAP01 | Department head notification when non-PO invoice exceeds budget | Low | **Potentially OOB.** D365 Budget Control can be configured to check non-PO invoices at the approval journal stage. If budget is exceeded, it triggers a warning or hard stop. The notification aspect can be handled by the **workflow approval routing** -- route over-budget invoices to the department head for explicit approval. This is configuration, not customization. **Reclassify as FIT with configuration.** |
| AP002-GAP02 | Attachment of scanned invoice PDF to the journal voucher | Low | **Standard OOB.** D365 Document Management allows attaching files to any transaction record including invoice journals. The "Attach" button is standard on all journal forms. If the requirement is for **automatic OCR capture**, that requires **AI Builder** or a third-party solution -- but simple attachment is OOB. Clarify the actual requirement. |
| AP002-GAP03 | Vendor invoice automation -- auto-create invoice journal from emailed PDF invoices | Medium | This is a legitimate GAP. D365 **Vendor Invoice Automation** (preview/GA feature) with **AI Builder** for invoice capture can automate this. Alternatively, use a third-party OCR solution (Kofax, ABBYY) integrated via Power Automate. **Recommendation:** If volume justifies it (>50 non-PO invoices/month), implement D365 Vendor Invoice Capture with AI Builder. If volume is low, manual entry via invoice register is sufficient. |

**Recommendations:**

1. **AP002-GAP02 (PDF attachment) is almost certainly OOB.** D365 Document Management supports file attachments on all journal forms. If the requirement is simply "attach the scanned invoice to the journal line," this is standard functionality. The only custom scenario would be automatic OCR-to-journal creation (which is AP002-GAP03). Recommend **removing AP002-GAP02 as a GAP entirely** and marking it FIT.

2. **The 2-step process (Invoice Register then Invoice Approval Journal) is correct** for organizations where AP captures invoices and departments approve/code them. However, if Technica has a small AP team and departments submit their own invoices, consider using **Vendor Invoice Journal** directly with workflow approval instead. This eliminates the 2-step handoff and is simpler. Discuss with Technica which scenario applies.

3. **Recurring invoices:** Ensure the periodic journal template includes proper financial dimensions and that the recurrence end dates are maintained. Stale recurring journals that continue posting after a contract ends are a common issue.

4. **Budget checking on non-PO invoices** is a critical control. The FRD should explicitly state whether budget control is **warning-only or hard-stop** for non-PO invoices. The Procurement FRD specifies warning-only for PRs -- AP should have a stricter policy (hard-stop or requires override approval) since non-PO invoices bypass the PR/PO approval chain.

---

### AP003 - Vendor Payment Processing (3 GAPs)

**What was proposed:**
- Payment proposal generation (by due date, vendor group, or manual selection)
- Payment journal with approval workflow
- Payment methods: check, wire transfer (bank transfer), LC (Letter of Credit)
- Check printing and management (void, reprint, reissue)
- Electronic payment file generation (bank format)
- Vendor settlement (automatic or manual)
- Payment calendar for payment scheduling
- Cash discount handling (take/lose based on payment date)
- Centralized payment for intercompany
- Bank reconciliation after payment posting

**Assessment: Comprehensive standard payment process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Payment proposal | Standard OOB | Flexible selection criteria |
| Payment journal workflow | Standard OOB | Amount-based approval |
| Check management | Standard OOB | Print, void, reissue |
| Wire transfer / electronic payment | Standard OOB | Bank file export formats |
| Letter of Credit (import) | Standard OOB | D365 has Import LC module |
| Vendor settlement | Standard OOB | Auto or manual |
| Cash discount | Standard OOB | Terms-based calculation |
| Centralized payment | Standard OOB | Intercompany |
| Bank reconciliation | Standard OOB | Standard or Advanced |

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|------------|----------------|
| AP003-GAP01 | Payment approval by amount -- different approvers for different thresholds (e.g., CFO approves >$50K) | Low | **Standard OOB.** Vendor payment journal workflow supports **conditional approval routing based on transaction amount**. Configure workflow conditions: Amount < 50K routes to Finance Manager, Amount >= 50K routes to CFO. This is pure workflow configuration. **Reclassify as FIT.** |
| AP003-GAP02 | Post-dated check (PDC) handling -- print check with future date, do not clear bank until maturity | Low-Medium | D365 has **OOB Post-dated check** functionality in Cash and Bank Management. Enable post-dated checks in the Cash and Bank parameters. Checks are registered with a maturity date and only clear the bank account when the maturity journal is posted. **Reclassify as FIT** if Technica enables this standard feature. |
| AP003-GAP03 | Notification to vendor when payment is processed (remittance advice via email) | Low | **Standard OOB via Print Management.** Configure the **Vendor Payment Advice** report with email delivery in Print Management. This automatically emails the remittance advice to the vendor's email on file when the payment is posted. No customization needed. **Reclassify as FIT.** |

**Recommendations:**

1. **All three AP003 GAPs are potentially FIT.** This is the most significant finding in this review:
   - **AP003-GAP01** (payment approval thresholds): Standard workflow conditional routing. Zero customization.
   - **AP003-GAP02** (post-dated checks): OOB feature in Cash & Bank Management. Needs to be enabled in parameters. Common in Middle East / North Africa implementations.
   - **AP003-GAP03** (remittance advice email): Print Management with email destination. Standard functionality.

   **Recommend reclassifying all three as FIT**, which would make AP003 a fully standard process with zero customization.

2. **Letter of Credit (Import LC):** D365 has a dedicated **Import Letter of Credit** module under Cash and Bank Management. Ensure the FRD explicitly maps to this module. The process is: Create LC facility > Issue LC against vendor invoice > Ship documents > Amend LC > Pay LC at maturity. If Technica uses import LCs frequently (likely, given they are a make-to-order manufacturer importing materials), this module should be properly configured. **Cross-reference with FRD12-GL (Cash & Bank) for LC bank facility setup.**

3. **Electronic payment formats:** Ensure the correct **payment file format** is configured for Technica's banks. D365 supports multiple formats (ISO 20022, NACHA, country-specific). For a Lebanon/MENA-based company, verify whether the bank supports SWIFT MT101 or a proprietary format. The Electronic Reporting (ER) framework allows configuring custom formats without X++ development.

4. **Centralized payment for intercompany** is mentioned but not detailed. If Technica has multiple legal entities, the centralized payment setup (one entity pays on behalf of others) requires careful intercompany settlement configuration. This links to the GL/Cash Bank FRD.

---

### AP004 - Down Payment / Advance Payment (3 GAPs)

**What was proposed:**
- Down payment request from project/procurement team
- Advance payment journal posting (prepayment to vendor before invoice)
- Application of prepayment against final vendor invoice
- Down payment as percentage of PO value
- Workflow approval for advance payment requests
- Tracking of advance payment balances per vendor
- Reversal of advance payment if PO is cancelled

**Assessment: Functional but uses a more manual approach than necessary**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Advance payment journal | Standard OOB | General journal with vendor posting |
| Apply prepayment to invoice | Standard OOB | Settlement/offset function |
| Workflow approval for advance | Standard OOB | Journal workflow |
| Reversal of prepayment | Standard OOB | Reverse transaction |

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|------------|----------------|
| AP004-GAP01 | Prepayment linked to PO -- track which PO the advance is against and auto-apply on invoice | Medium | **This is OOB in D365 via Vendor Prepayment on Purchase Order.** D365 has a standard feature: PO header > Invoice tab > Prepayment. You can define a prepayment value (amount or %) on the PO. When the prepayment invoice is created, it links to the PO. When the final invoice is posted, the prepayment is automatically applied. **Reclassify as FIT.** This is the most impactful finding -- it eliminates the need for manual journal entries and manual tracking. |
| AP004-GAP02 | Advance payment balance report per vendor showing outstanding prepayments | Low | **Partially OOB.** Vendor transactions filtered by "Prepayment" transaction type show outstanding balances. For a formatted report, use **Power BI** with vendor transaction data filtered to prepayment posting types. Alternatively, the **Vendor balance list** report with prepayment filter provides this. |
| AP004-GAP03 | Notification to project manager when advance payment is posted against their project PO | Low | **Power Automate** trigger on payment journal posting where PO has a project reference. Send Teams/email notification to the project manager from the project record. Alternatively, use **D365 Alerts** on vendor transaction posting with project filter. |

**Recommendations:**

1. **AP004-GAP01 is the most important finding in this review.** The FRD describes a manual process for advance payments (create general journal, manually track against PO, manually apply to final invoice). D365 has a **standard Vendor Prepayment feature directly on the Purchase Order**:
   - On the PO header, define the prepayment amount or percentage
   - Generate a prepayment invoice directly from the PO
   - The prepayment creates a vendor transaction linked to the PO
   - When the final vendor invoice is posted against the PO, the prepayment is **automatically applied/settled**
   - Full tracking and reporting of prepayment balances per PO

   This eliminates the manual journal approach entirely and provides better audit trail, automatic settlement, and PO-level tracking. **Strongly recommend redesigning AP004 to use the standard PO Prepayment feature.**

2. **For project-linked advance payments** (common at Technica given their project-based business model -- see FRD04): When the PO is linked to a project, the prepayment posting can also post to the project as a cost. Ensure the project posting profile is configured to handle prepayment transactions correctly (post to a WIP/advance account, not directly to project cost, until the final invoice is posted).

3. **The advance payment approval workflow** should be integrated into the PO workflow, not a separate workflow. When the PO includes a prepayment, the PO approval workflow should route to Finance for prepayment approval as part of the PO approval chain. This avoids two separate approval processes.

4. **Tracking advance payment balances:** Instead of a custom report, configure the **Vendor aging report** to include prepayment transactions. This provides a standard view of all outstanding vendor balances including prepayments. For project-specific advance tracking, use the **Project committed costs** inquiry which shows PO prepayments as committed costs.

---

## Reports Review

The FRD identifies several reporting requirements. Assessment:

| Report | OOB? | Recommendation |
|--------|------|----------------|
| Vendor invoice journal list | Yes | Standard journal report |
| Vendor transaction report (by date range, vendor) | Yes | Vendor transactions inquiry + export |
| Vendor aging report (30/60/90/120 days) | Yes | Standard Vendor Aging report with configurable periods |
| Vendor balance report by financial dimension | Yes | Vendor balance list with dimension filters |
| Payment history report | Yes | Vendor transactions filtered by payment type |
| Outstanding invoice report (unpaid) | Yes | Vendor aging or Open transactions |
| Check register / payment register | Yes | Standard Bank transaction / Check report |
| Vendor prepayment balance report | Partial | Vendor transactions with prepayment filter; for project-level view, use Power BI |
| Vendor withholding tax report | Yes | Standard withholding tax report |
| Vendor 1099 report (if applicable) | Yes | Standard 1099 reporting (US entities only) |

**Report Recommendations:**

1. **Most reports are fully OOB.** No custom SSRS development is needed for the majority of these reports.

2. **For enhanced analytics** (trend analysis, vendor spend dashboards, payment cycle time), use **Power BI Vendor Analytics** workspace which is available OOB in D365 and provides embedded dashboards.

3. **Vendor prepayment balance report** is the only report that may need a custom view. Recommend building this as a **Power BI paginated report** that pulls vendor transactions by posting type = Prepayment, grouped by PO and project.

---

## Setup and Configuration Review

| Configuration | Assessment | Comment |
|--------------|------------|---------|
| Vendor posting profiles | Standard OOB | By vendor group -- proper approach |
| Invoice matching policy | Standard OOB | 2-way / 3-way by item model group |
| Payment terms | Standard OOB | Net 30/60/90, EOM, etc. |
| Cash discount terms | Standard OOB | Early payment discounts |
| Payment methods | Standard OOB | Check, wire, LC |
| Number sequences | Standard OOB | For invoices, payments, journals |
| Financial dimension defaults | Standard OOB | From vendor master, PO |
| Withholding tax setup | Standard OOB | If applicable by jurisdiction |
| Vendor groups | Standard OOB | By type (material, service, intercompany) |
| AP parameters | Standard OOB | Matching, workflow, posting defaults |

**Configuration Recommendations:**

1. **Invoice matching tolerances** should be defined with input from Procurement (FRD07-Procurement). Price tolerance of 0% means every price variance triggers review -- this is strict but common in manufacturing. Consider allowing a small tolerance (e.g., 1-2%) for freight/currency rounding to avoid excessive match exceptions.

2. **Vendor groups** should align with the Procurement FRD vendor classification (domestic, international, intercompany, subcontractor). Ensure the same grouping is used for both procurement policies and AP posting profiles.

3. **Three-way matching** is already defined in the Procurement FRD for stocked items. Ensure the item model group "Matching policy" field is consistently set. Service items should use 2-way matching (no product receipt for services).

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | AP001 | Reclassify GAP03 (match failure notification) as FIT -- use invoice workflow notifications | Medium | Eliminates 1 GAP |
| 2 | AP002 | Reclassify GAP02 (PDF attachment) as FIT -- standard Document Management | Low | Eliminates 1 GAP |
| 3 | AP002 | Evaluate D365 Vendor Invoice Capture with AI Builder for non-PO invoice automation | Medium | Process efficiency |
| 4 | AP002 | Configure budget control hard-stop (not just warning) for non-PO invoices | High | Financial control |
| 5 | AP003 | Reclassify all 3 GAPs as FIT: payment thresholds (workflow), PDC (OOB feature), remittance email (Print Management) | High | Eliminates 3 GAPs |
| 6 | AP003 | Enable Post-Dated Check feature in Cash & Bank parameters | Medium | Standard MENA requirement |
| 7 | AP003 | Verify electronic payment format for Technica's banks (SWIFT MT101 / ISO 20022) | Medium | Payment processing |
| 8 | AP004 | **Redesign advance payment process to use standard PO Prepayment feature** | **Critical** | Eliminates manual journals, auto-settles, provides PO-level tracking |
| 9 | AP004 | Integrate advance payment approval into PO workflow (not separate) | Medium | Process simplification |
| 10 | AP004 | Configure project posting profile for prepayment WIP handling | Medium | Correct project cost treatment |
| 11 | Reports | Use Power BI for vendor prepayment balance report | Low | Single custom report |
| 12 | Setup | Define matching tolerances with Procurement input; consider 1-2% price tolerance | Medium | Reduces match exceptions |
| 13 | Setup | Align vendor groups between AP and Procurement FRDs | Medium | Consistency |

---

## Risk Flags

1. **Advance Payment design is suboptimal.** The current FRD describes a manual general journal approach for vendor advance payments. D365 has a **standard PO Prepayment feature** that automates linking, tracking, and settlement. Using the manual approach creates risk of:
   - Prepayments not being applied to the correct invoice
   - No automatic PO-level tracking of advance balances
   - Manual reconciliation errors
   - Project cost misallocation for project-linked advances

   **This is the single most important redesign recommendation in this review.**

2. **Cross-FRD matching policy inconsistency risk.** The matching policy (2-way vs. 3-way) is defined in the Procurement FRD at the PO level and in the AP FRD at the invoice validation level. These must be **identical** -- if Procurement sets 3-way matching but AP is configured differently, match exceptions will occur on every invoice. Establish a single matching policy matrix owned by one team.

3. **Post-dated checks (PDC) require bank module configuration.** If Technica uses PDCs heavily (common in Lebanon/MENA region), the Cash & Bank Management module must be configured with PDC clearing journals, maturity tracking, and potentially a PDC register. This has a dependency on **FRD12-GL (Cash & Bank)**. Ensure the GL/Cash Bank FRD includes PDC setup.

4. **Non-PO invoice budget control gap.** PO-based invoices inherit budget control from the PR/PO approval chain (FRD07-Procurement). Non-PO invoices bypass this chain entirely. Without explicit budget control on non-PO invoice approval journals, there is a risk of uncontrolled spending outside the PO process. **Recommend hard-stop budget control on non-PO invoice approval journals.**

5. **Letter of Credit process is mentioned but not detailed.** For a manufacturing company importing materials, LC management is a significant process. The AP FRD should either detail the LC process fully or reference FRD12-GL (Cash & Bank) where it should be covered. Currently this appears to be a gap in coverage between FRDs.

6. **Vendor withholding tax** is listed in setup but not detailed in any process. If Technica operates in jurisdictions requiring withholding tax on vendor payments (common in MENA), the calculation, reporting, and remittance process needs explicit process documentation.

---

## Cross-FRD Dependency Map

| Dependency | From FRD | To FRD | Integration Point | Status |
|-----------|----------|--------|-------------------|--------|
| PO-to-Invoice matching | Procurement (FRD07-PR) | AP (FRD07-AP) | Matching policy, PO reference on invoice | Must align matching policy |
| Project PO invoice cost posting | AP (FRD07-AP) | Project Accounting (FRD04) | Project ID on PO/invoice, cost posting profile | Must align auto-consumption setting |
| Advance payment on project PO | AP (FRD07-AP) | Project Accounting (FRD04) | Prepayment WIP posting, committed cost tracking | Needs project posting profile for prepayments |
| Landed cost invoice | Logistics (FRD06) | AP (FRD07-AP) | Voyage cost invoices, charge allocation to inventory | Standard Landed Cost to AP flow |
| Post-dated checks | AP (FRD07-AP) | GL/Cash Bank (FRD12-GL) | PDC clearing, bank reconciliation, maturity tracking | Must enable PDC in Cash & Bank |
| Letter of Credit payments | AP (FRD07-AP) | GL/Cash Bank (FRD12-GL) | LC facility, LC issuance, LC payment at maturity | Process ownership unclear between FRDs |
| Budget control on non-PO invoices | AP (FRD07-AP) | Budgeting (FRD12-Budget) | Budget check at approval journal posting | Must configure budget control rules |
| Vendor master alignment | Procurement (FRD07-PR) | AP (FRD07-AP) | Vendor groups, payment terms, posting profiles | Single vendor master shared |

---

## Efficiency Verdict

The Accounts Payable FRD is **fundamentally sound** and makes correct use of D365's core AP features -- invoice matching, vendor invoice workflows, payment journals, and settlement. The PO invoice process (AP001) and payment process (AP003) are well-designed and largely standard.

However, **the solution leaves significant efficiency on the table in two areas:**

1. **Advance Payment (AP004):** The manual journal approach ignores D365's standard PO Prepayment feature, which provides automated linking, tracking, settlement, and project cost handling. Redesigning this process to use the OOB feature would eliminate manual work, reduce errors, and provide better visibility. This is the **highest-impact recommendation** in this review.

2. **GAP over-reporting:** Of the approximately 12 GAPs identified, **at least 5-6 are achievable with standard D365 configuration** (workflow routing, Print Management email, Document Management attachments, post-dated checks, budget control). Reclassifying these as FIT would significantly reduce the customization scope and cost of the AP implementation.

The AP module is a **dependency hub** connecting Procurement, Project Accounting, Logistics, and Cash/Bank Management. The cross-FRD alignment (especially matching policy, auto-consumption settings, and PDC handling) requires a joint design session across module teams before configuration begins.

**Estimated GAP reduction:** From ~12 identified GAPs to ~5-6 actual GAPs requiring customization, with the most complex being Vendor Invoice Automation (AP002-GAP03) at medium complexity.

---

*This review is based on D365 F&O Accounts Payable best practices as of 2026. All recommendations should be validated against Technica's specific D365 version and licensed modules.*
