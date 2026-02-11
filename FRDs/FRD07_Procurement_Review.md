# FRD07 - Procurement and Sourcing - Solution Design Verification Report

**Document:** FRD07 - Procurement and Sourcing - V3.0
**Prepared by:** Edward Khoury | Contributors: Abdo Khoury, Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Procurement & Sourcing, Vendor Collaboration Portal

---

## Executive Summary

This FRD covers 6 procurement processes: Purchase Requisitions, Master Planning for materials, PO Processing, Purchase Returns, Vendor Registration, and Vendor Evaluation. The solution makes **strong use of D365 standard procurement capabilities** including Vendor Portal, RFQ processing, purchasing workflows, and procurement policies. Most requirements are FIT with a handful of minor GAPs (mostly notifications and custom fields). One significant GAP is the Vendor Evaluation process which requires full customization.

Overall assessment: **Well-structured procurement solution with mostly standard functionality. Clean and practical.**

---

## Process-by-Process Review

### PR001 - Internal Requisition (1 GAP)

**What was proposed:**
- Purchase Requisition with procurement catalog
- Category-based catalog with department restrictions
- Budget check on submission (warning, not blocking)
- Workflow approval (different flows for project vs. non-project)
- RFQ from requisition with vendor portal integration
- Bid scoring and comparison before awarding
- Preparer can create PR on behalf of requester
- 3-way matching policy
- Financial dimensions default from employee

**Assessment: Comprehensive and standard — excellent**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Purchase requisition workflow | Standard OOB | Flexible approval chains |
| Procurement catalog | Standard OOB | Category-based with restrictions |
| Budget checking | Standard OOB | Warning on over-budget |
| RFQ via Vendor Portal | Standard OOB | With bid scoring |
| Preparer/Requester model | Standard OOB | Delegation supported |
| 3-way matching | Standard OOB | PO-receipt-invoice matching |
| Financial dimensions on employee | Standard OOB | Default department |

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| PR001-027 | Needed date on PR lines | This may actually be FIT in recent D365 versions — the "Requested date" field exists on PR lines. Verify if the standard field meets the requirement before customizing. If a specific "Needed on date" display is required, it's a minor form extension. |

**Recommendations:**

1. **Purchasing policy for PO re-approval** is well-configured. The list of fields that trigger re-approval (supplier, price, discount, currency, etc.) vs. those that don't (delivery date, shipping docs) is a practical and correct setup.

2. **Engineer access to comparison sheets** — ensure security roles allow engineers read-only access to RFQ comparison without giving them procurement permissions.

3. **External item ID per vendor** for technical specs is a standard D365 feature (External Item Description on item-vendor mapping). No customization needed.

4. **The dual workflow** (project-based PR requires PO workflow; non-project PR skips PO workflow since PR was already approved) is a smart efficiency gain. This reduces approval overhead for routine purchases.

---

### PR002 - Master Planning (All FIT)

**What was proposed:**
- Demand Forecast for finished products
- Min/Max for spare parts
- Master Planning engine generates planned POs
- Requesting department firms planned POs
- Procurement processes confirmed POs

**Assessment: Standard Master Planning — no issues**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Demand Forecast | Standard OOB | Year/period based |
| Min/Max replenishment | Standard OOB | For spare parts |
| Master Planning scheduling | Standard OOB | Periodic execution |
| Planned PO generation | Standard OOB | Auto-calculated |
| Firming process | Standard OOB | Manual review + firm |

**Recommendation:** Ensure vendor lead times are maintained accurately in item coverage settings. Master Planning accuracy depends entirely on correct lead times. Consider a periodic review process for lead time data.

---

### PR003 - PO Processing (4 GAPs)

**What was proposed:**
- PO creation with vendor portal submission
- Vendor acceptance/rejection on portal
- PO workflow with dynamic approval
- Proforma confirmation email to supplier
- Change order process with re-approval workflow
- Advance payment request flow
- PO revision (reopen → re-submit workflow)
- Site leader receives cross-shipping POs

**Assessment: Well-designed with standard workflow. GAPs are minor.**

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|-----------|----------------|
| PR003-005 | Email notification on PO proforma confirmation to supplier/requester | Low | **Power Automate** trigger on PO confirmation → email with PDF attachment. Could also use standard D365 print management with email delivery. Check if **Print Management** with email destination already handles this before customizing. |
| PR003-006 | Missing fields on PO (ex-works date, penalty terms) | Low | Simple form extensions. Custom fields added to PO header/lines. Standard X++ extension — 4-6 hours. |
| PR003-007 | Button to email accounting team for PO payment | Low | **Power Automate** button trigger or custom D365 action button. Could also use standard **D365 workflow notification** at PO invoice stage instead of a manual button. |
| PR003-010 | Notify requester when delivery date > needed date | Low | **Power Automate** comparing PO delivery date to PR needed date. Or use standard **D365 Alerts** with custom condition. |
| PR003-011 | PO payment amount on PO list | Low | Add display method or computed column showing payments against PO. Check if **Vendor transactions** filtered by PO already provides this. |

**Recommendations:**

1. **PR003-005 (Email on confirmation):** Before customizing, check if **D365 Print Management** with "Email" destination for PO Confirmation already meets this need. Many implementations use print management email delivery and skip custom notification development.

2. **PR003-007 (Payment request button):** Instead of a custom button, consider using the standard **PO workflow** with a "Payment Requested" status that triggers an email to accounting. This keeps everything in the workflow framework and provides audit trail.

3. **Vendor Portal acceptance** is a great feature. Ensure vendor training is planned for portal usage — vendor adoption is often the biggest challenge, not the technology.

---

### PR004 - Return Purchase Order (All FIT)

**What was proposed:**
- PO with negative quantity for returns
- Mark return PO to original PO (cost accuracy)
- Charges on return PO
- Invoice return PO

**Assessment: Standard D365 purchase return process — no issues**

The marking of return PO to original PO is the correct approach for cost accuracy. The note about extra cost flowing to the project is also correct behavior.

---

### PR005 - Vendor Registration (1 GAP)

**What was proposed:**
- Prospective vendor creation
- Email invitation with registration link
- Vendor self-registration wizard on portal
- Document/certification upload
- Procurement review and approval workflow
- Auto-creation of vendor master record on approval

**Assessment: Standard Vendor Collaboration — well-implemented**

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| PR005-007 | Questionnaire form during onboarding with attachment customization | D365 has OOB **Questionnaire** functionality that can be embedded in vendor registration. The "GAP" may only be the attachment piece. Verify if standard questionnaire + document attachment meets the need before building custom. |

---

### PR006 - Vendor Evaluation (Full GAP)

**What was proposed:**
- Vendor scoring across 3 dimensions:
  - Quality (50%): Based on SNC (non-conformance) POs / total POs
  - Finance (35%): Payment terms-based + manual price scoring
  - Service (15%): Manual communication support rating
- Auto-blacklist vendors scoring < 50/100

**Assessment: Requires full customization — most significant procurement GAP**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Quality score (SNC ratio) | Needs Design | Where are SNC POs tracked? Need to link to Quality Management non-conformances |
| Finance score (payment terms) | Needs Design | How are payment terms scored? Need scoring matrix |
| Service score (manual) | Simple | Manual input form |
| Auto-blacklist | Needs Design | Vendor hold/block mechanism |

**Recommendations:**

1. **Consider Power Apps for Vendor Evaluation** instead of X++ customization. Build a Model-Driven App that:
   - Pulls PO data and non-conformance data from D365 via Dataverse/Virtual Entities
   - Provides a scoring form for manual inputs
   - Calculates composite score
   - Triggers vendor hold in D365 when score < 50

   This is **faster to build, easier to modify**, and doesn't require developer support for scoring formula changes.

2. **Alternatively, use Power BI** for the automated scoring (Quality and Finance) with a manual input form for Service scores. Power BI can calculate, display dashboards, and trigger alerts for low-scoring vendors.

3. **For the auto-blacklist:** Use D365 standard **Vendor Hold** functionality (on-hold for all transactions) rather than deleting or deactivating vendor records. This preserves history while blocking future transactions.

4. **SNC PO tracking:** This requires integration with Quality Management (FRD not in scope?) or a custom "Non-Conformance" flag on PO lines. Define how non-conformances are recorded before building the scoring.

---

### Reports

10 reports requested. Assessment:

| Report | OOB? | Recommendation |
|--------|------|----------------|
| Purchase Confirmation report | Yes | Print Management |
| RFQ report | Yes | Standard |
| Detailed PO report (by status) | Partial | Standard PO list with filters; status breakdown may need Power BI |
| Item PO History (price history) | Yes | Purchase statistics |
| All PRs (assigned/prepared/requested) | Yes | Standard list with filters |
| Top 100 vendors | Partial | Power BI from vendor transaction data |
| Vendor/Item statistics | Yes | Standard |
| Open PO lines by delivery date | Yes | Standard |
| Open PO lines by vendor | Yes | Standard |
| Purchase receiving log | Yes | Standard |

Most reports are OOB or achievable with standard list filters. **Top 100 vendors** and **detailed PO by status** are best served by **Power BI dashboards**.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | PR001 | Verify if "Needed date on PR lines" is already OOB before customizing | Low |
| 2 | PR003 | Use D365 Print Management email delivery for PO confirmation notification | Medium |
| 3 | PR003 | Replace custom payment button with workflow status notification | Low |
| 4 | PR005 | Verify if standard Questionnaire module handles vendor onboarding form | Medium |
| 5 | PR006 | Build vendor evaluation in Power Apps + Power BI instead of X++ | High |
| 6 | PR006 | Use Vendor Hold for blacklisting, not vendor deletion | Medium |
| 7 | Reports | Use Power BI for Top 100 vendors and PO status dashboards | Medium |

---

## Risk Flags

1. **Vendor Evaluation (PR006) is a full custom build.** This is the most significant GAP and needs proper scoping. A Power Apps approach would be faster and more maintainable than X++ customization.

2. **Vendor Portal adoption risk.** The solution relies heavily on vendors using the portal for RFQ responses and PO acceptance. Ensure vendor onboarding and training is part of the project plan. Have a fallback process for vendors who refuse to use the portal.

3. **The many PO fields to add (PR003-006)** need to be designed as a cohesive extension, not piecemeal additions. Plan a single PO extension package.

---

## Efficiency Verdict

The Procurement FRD is **well-designed and makes strong use of D365 standard capabilities**. The Vendor Portal, RFQ process, purchasing policies, and workflow engine are all used appropriately. The dual workflow approach (project vs. non-project) is an efficient design.

The main improvement area is **Vendor Evaluation**, which should be built using Power Platform rather than X++ for faster delivery and easier maintenance.

---

*This review is based on D365 F&O Procurement & Sourcing best practices as of 2026.*
