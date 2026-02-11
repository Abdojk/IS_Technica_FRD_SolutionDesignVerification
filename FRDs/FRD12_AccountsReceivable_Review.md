# FRD12 - Accounts Receivable - Solution Design Verification Report

**Document:** Technica - FRD12 - Accounts Receivable - V2.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - Accounts Receivable, Credit & Collections

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Cross-FRD | Project milestone invoicing integration is absent — AR-Project end-to-end flow must be explicitly designed | [Critical Missing Coverage: Project Milestone Invoicing Integration](#critical-missing-coverage-project-milestone-invoicing-integration) |
| 2 | **CRITICAL** | AR-003 - Sales Order Processing | Black market rate invoicing is highest-complexity GAP — requires dedicated cross-FRD technical design (GL + AR + Tax) | [AR-003: Sales Order Processing](#ar-003-sales-order-processing) |
| 3 | **HIGH** | Customer Setup | Verify if OOB customer creation workflow exists in target D365 build before custom development (GAP 001-004) | [Customer Master Setup](#customer-master-setup) |
| 4 | **HIGH** | Customer Setup | Credit hold CRM integration mechanism undefined — use Dual-Write or Power Automate (GAP 001-013) | [Customer Master Setup](#customer-master-setup) |
| 5 | **HIGH** | Customer Setup | Payment terms are undefined — must be resolved before UAT | [Customer Master Setup](#customer-master-setup) |
| 6 | **HIGH** | AR-002 - Customer Payments | Advance payment accounting treatment is under-specified — define prepayment posting profile | [AR-002: Customer Payments](#ar-002-customer-payments) |
| 7 | **HIGH** | AR-002 - Customer Payments | Advanced Bank Reconciliation not mentioned — critical for automated payment matching | [AR-002: Customer Payments](#ar-002-customer-payments) |
| 8 | **HIGH** | AR-006 - Inquiries & Reports | Project ID on reports — test financial dimension approach before building custom reports (4 GAPs) | [AR-006: Inquiries & Reports](#ar-006-inquiries--reports) |
| 9 | **MEDIUM** | Customer Setup | Customer groups too simple (Local/Foreign) — expand to include Distributor and Intercompany segments | [Customer Master Setup](#customer-master-setup) |
| 10 | **MEDIUM** | Customer Setup | Single posting profile is risky for intercompany — add separate intercompany posting profile | [Customer Master Setup](#customer-master-setup) |
| 11 | **MEDIUM** | AR-001 - Free Text Invoicing | Use ER framework for invoice report customization instead of SSRS modification | [AR-001: Free Text Invoicing](#ar-001-free-text-invoicing) |
| 12 | **MEDIUM** | AR-002 - Customer Payments | Electronic payment methods undefined for foreign customers (SEPA, wire templates, LC) | [AR-002: Customer Payments](#ar-002-customer-payments) |
| 13 | **MEDIUM** | AR-004 - Foreign Currency Reval. | Automate exchange rate import instead of manual daily entry | [AR-004: Foreign Currency Revaluation](#ar-004-foreign-currency-revaluation) |
| 14 | **MEDIUM** | AR-005 - Intercompany Trade | Keep intercompany automation conservative at go-live; expand post-stabilization | [AR-005: Intercompany Trade Operations](#ar-005-intercompany-trade-operations-poso) |
| 15 | **MEDIUM** | AR-006 - Inquiries & Reports | Build unified Finance Power BI workspace for AR reporting (shared with GL, AP, Project) | [AR-006: Inquiries & Reports](#ar-006-inquiries--reports) |
| 16 | **LOW** | Customer Setup | Cross-Company Data Sharing field planning — shared fields cannot differ between entities | [Customer Master Setup](#customer-master-setup) |
| 17 | **LOW** | AR-006 - Inquiries & Reports | Missing report: Cash application / unapplied advance payments report | [AR-006: Inquiries & Reports](#ar-006-inquiries--reports) |

**Totals:** 2 CRITICAL | 6 HIGH | 6 MEDIUM | 2 LOW

---

## Executive Summary

This FRD covers Technica's Accounts Receivable function across six processes: Free Text Invoicing (AR-001), Customer Payments (AR-002), Sales Order Processing (AR-003), Foreign Currency Revaluation (AR-004), Intercompany Trade Operations (AR-005), and Inquiries & Reports (AR-006). The document also defines the full customer master setup including customer groups, workflows, trade agreements, posting profiles, payment methods/terms, aging brackets, collections, and credit hold rules.

The overall solution **leans heavily on standard D365 F&O capabilities**, which is appropriate. Most requirements are FIT. The GAPs identified are relatively minor: a customer creation workflow (001-004), a CRM credit hold integration (001-013), report layout modifications (AR001-002, AR003-002), a black market rate invoicing scenario (AR003-003), and four report modifications to include Project ID (AR006-001 through AR006-004).

However, the FRD has **notable gaps in coverage** rather than in fit/gap analysis. The document does not explicitly address the integration between AR and the milestone-based project invoicing described in FRD04 (Project Accounting). The Document Approvals section even contains a reviewer comment asking: *"What about the invoices issuing coming from the project milestone?"* -- with the response pointing to FRD04 Section PJ003. This cross-FRD dependency is the single biggest design concern, as the majority of Technica's revenue flows through project milestone invoicing, not free text invoices or standard sales orders.

Overall assessment: **Solid standard AR setup with minor customizations needed. The critical gap is the lack of explicit integration design between AR and Project Accounting milestone billing.**

---

## Process-by-Process Review

### Customer Master Setup

**What was proposed:**
- Customer data imported via Excel templates with automatic number sequence
- Customer status controls: on-hold for invoicing, accounting entry, or all
- Cross-Company Data Sharing for multi-entity customer records
- Two customer groups: Local and Foreign
- Customer creation workflow (GAP - requires customization)
- Proposed change workflow for customer amendments (partial FIT)
- Trade agreement journals for customer-specific and group-specific pricing
- Single posting profile ("ALL" / "General") for all customer groups
- Payment methods: Cash and Bank Transfer
- Payment terms: pending from Technica
- Aging brackets: 30, 60, 90, 180, 1y, 1.5y, 2y, 3y, and over
- Collections centralized screen with aging ranges
- Credit hold rules with CRM integration (GAP)

**Assessment: Well-structured master data design with appropriate standard features**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Customer import via templates | Standard OOB | Data Management Framework handles this |
| Automatic number sequence | Standard OOB | No issues |
| Customer on-hold controls | Standard OOB | Invoicing / accounting / all blocking |
| Cross-Company Data Sharing | Standard OOB | Good use of shared service model |
| Customer groups (Local/Foreign) | **>> NEEDS REVIEW <<** | Simple grouping — may be insufficient for distributors and intercompany |
| Customer creation workflow | **>> NEEDS REVIEW <<** | D365 does not have OOB customer creation workflow — verify target build |
| Proposed change workflow | Standard OOB | Built-in feature for field-level change approval |
| Trade agreement journals | Standard OOB | Well-documented setup procedure |
| Posting profiles | **>> NEEDS REVIEW <<** | Single profile — risky for intercompany scenarios |
| Payment methods | Standard OOB | Cash + Bank Transfer |
| Payment terms | **>> NEEDS REVIEW <<** | Undefined — pending from Technica |
| Aging brackets | Standard OOB | Extended brackets beyond standard 30/60/90 |
| Collections | Standard OOB | Centralized collection agent workspace |
| Credit hold with CRM sync | **>> NEEDS ARCHITECTURE <<** | Integration mechanism not specified |

#### GAP 001-004: Customer Creation Workflow

**>> ATTENTION AREA: OOB feature may exist in target D365 build**

> D365 F&O versions 10.0.32+ introduced a Customer creation workflow as a preview/standard feature. Before investing in custom development, verify whether the current D365 version deployed at Technica includes this feature natively.

| Attribute | Assessment |
|-----------|------------|
| **Description** | Sales department creates customer, Sales Manager approves |
| **Complexity** | Medium |
| **FRD Proposed Solution** | Custom workflow for customer creation |
| **D365 Reality Check** | As of recent D365 F&O versions (10.0.32+), Microsoft introduced a **Customer creation workflow** as a preview/standard feature. Before building custom, verify whether the current D365 version deployed at Technica includes this feature natively. If not available, this requires X++ workflow development. |
| **Recommended Approach** | **>> ENHANCEMENT:** **Option A (Preferred):** Check if the OOB customer creation workflow is available in the target D365 build. If yes, configure it directly -- no customization needed. **Option B:** If not available OOB, implement using the standard D365 workflow framework with a custom workflow type for the CustTable. This is a well-documented pattern -- estimated effort 16-24 hours. **Option C (Lightest):** Use the existing "Proposed change workflow" even for new records by creating the customer in a "Draft/Pending" status and routing through the proposed change workflow for activation. This avoids custom workflow development entirely. |
| **Effort Estimate** | Option A: 2-4 hours (configuration). Option B: 16-24 hours (development). Option C: 4-8 hours (configuration). |

#### GAP 001-005: Customer Amendment Workflow with On-Hold Field

| Attribute | Assessment |
|-----------|------------|
| **Description** | Proposed change workflow extended to include the "On-Hold" field |
| **Complexity** | Low |
| **D365 Reality Check** | The proposed change workflow is OOB. Adding the "On-Hold" field to the tracked fields list is a configuration task. The FRD correctly lists the fields subject to proposed change approval (Name, Credit limit, Sales tax group, Method of payment, etc.). |
| **Recommended Approach** | Standard configuration. Add the "On-Hold" field to the proposed change tracking fields. No development required. Ensure the workflow is configured with appropriate escalation paths so that on-hold changes are not stuck waiting for approval. |
| **Effort Estimate** | 1-2 hours (configuration) |

#### GAP 001-013: Credit Hold Integration Back to CRM

**>> ATTENTION AREA: Integration mechanism not specified**

> The FRD requires credit hold status sync from D365 F&O back to CRM but does not specify the integration mechanism. This must be defined before development starts.

| Attribute | Assessment |
|-----------|------------|
| **Description** | When a customer is placed on credit hold in D365 F&O, the status should be synced back to CRM as "On Hold" |
| **Complexity** | Medium |
| **FRD Proposed Solution** | Integration (mechanism not specified) |
| **Recommended Approach** | **>> ENHANCEMENT:** **Option A (Preferred): Dual-Write** -- If Technica is using Dual-Write for Account/Contact synchronization (as implied by FRD01), add the credit hold/on-hold status to the Dual-Write mapping. This provides real-time bidirectional sync with minimal custom development. **Option B: Power Automate** -- Trigger a flow when the customer hold status changes in D365 F&O (using the Finance & Operations connector or Dataverse virtual entity) and update the CRM Account record. This is event-driven and easier to maintain. **Option C: Custom integration via Data Integrator** -- Batch sync of credit status. Not recommended for this scenario as near-real-time is important for credit enforcement. |
| **Effort Estimate** | Option A: 4-8 hours (Dual-Write map extension). Option B: 8-12 hours (Power Automate development + testing). |

**Additional Customer Master Recommendations:**

1. **>> ENHANCEMENT:** **Customer groups are too simple.** Two groups (Local/Foreign) may be insufficient for a manufacturing company with distributors and end customers. The FRD itself mentions "some of them are considered distributors, others end customers." Consider adding: **Distributor-Local**, **Distributor-Foreign**, **End Customer-Local**, **End Customer-Foreign**, **Intercompany**. This enables more granular trade agreements, posting profiles, and reporting.

2. **>> ENHANCEMENT:** **Single posting profile is risky.** Using one "ALL" posting profile means all customer types post to the same AR control account. For a multi-entity manufacturer with intercompany trade, consider at minimum two posting profiles: one for external customers and one for intercompany customers. This provides cleaner balance sheet segregation and simplifies intercompany reconciliation.

3. **>> ATTENTION AREA: Payment terms are undefined**

   > The FRD states "Will be waiting from Technica team list of payment terms." This is a migration dependency that must be resolved before UAT. Payment terms directly affect aging calculations, cash flow forecasting, collection strategies, and credit management.

   **>> ENHANCEMENT:** For a make-to-order manufacturer, typical terms include: advance payment (before production), milestone-based (aligned to FRD04 project milestones), Net 30/60/90, and letter of credit terms for foreign customers. Define these during the design phase to avoid rework.

4. **The trade agreement setup is well-documented** with clear screenshots and instructions. The decision to exclude spare parts from trade agreements is noted. Ensure spare parts pricing is handled through item-level default pricing or a separate price list mechanism.

---

### AR-001: Free Text Invoicing

**What was proposed:**
- Free text invoice for non-item sales and fixed asset disposals
- Check if customer exists, create if not, then enter/print/post invoice
- Ledger account-based invoicing (no item numbers)
- Can allocate to project (Time & Material type only)
- Report layout modification required (GAP)
- Workflow to be defined during migration

**Assessment: Standard functionality with one report GAP**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Free text invoice creation | Standard OOB | Ledger account-based, no items |
| Fixed asset disposal via FTI | Standard OOB | Correct use case |
| Project allocation on FTI | **>> NEEDS REVIEW <<** | Limited to T&M projects only — significant for fixed-price business |
| FTI workflow | Standard OOB | Configurable approval chain |
| Report layout modification | GAP | SSRS report customization |

#### GAP AR001-002: Free Text Invoice Report Layout Modification

| Attribute | Assessment |
|-----------|------------|
| **Description** | Modify the FTI print layout to match Technica's requirements |
| **Complexity** | Low-Medium |
| **FRD Proposed Solution** | Modify D365 report layout |
| **Recommended Approach** | **>> ENHANCEMENT:** **Option A (Preferred): Use D365 Report Designer** to modify the standard FreeTextInvoice SSRS report. Clone the standard design, apply Technica branding and field layout. This is standard practice and preserves upgradeability if done as an extension. **Option B: Use ER (Electronic Reporting)** framework if Technica needs the invoice in a specific electronic format (e.g., for e-invoicing compliance). ER is the modern approach for document formatting in D365 F&O and supports Excel/Word/PDF templates that business users can modify without developer involvement. |
| **Effort Estimate** | 8-16 hours depending on layout complexity |

**Recommendations:**

1. **>> ATTENTION AREA: Project allocation limitation (T&M only) is significant**

   > The FRD correctly notes that free text invoices can only be allocated to Time & Material projects, not Fixed Price projects. Since Technica uses Fixed Price milestone billing (FRD04 PJ003), free text invoices cannot be used for project-related billing adjustments on fixed price projects. Ensure the business team understands this limitation.

   **>> ENHANCEMENT:** Any adjustments to fixed price project invoices must go through **project credit notes** or **invoice proposals**, not free text invoices.

2. **Fixed asset disposal via FTI** is a valid approach. However, ensure the Fixed Asset module (FRD12 - Fixed Assets) is configured to support disposal-by-sale with automatic gain/loss calculation. The FTI must reference the fixed asset for proper accounting.

3. **FTI workflow approval** is deferred to migration phase. Define this early -- at minimum, a threshold-based workflow (e.g., FTI > $10,000 requires manager approval) prevents unauthorized invoice creation.

---

### AR-002: Customer Payments

**What was proposed:**
- Customer payment journal creation
- Settlement function for matching payments to invoices
- Support for advance payments before invoice issuance
- One payment journal can include invoices from different projects
- Workflow approval for payment journals

**Assessment: Standard payment processing -- all FIT**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Payment journal creation | Standard OOB | Manual entry |
| Invoice settlement | Standard OOB | Function > Settlement |
| Advance payments | **>> NEEDS REVIEW <<** | Accounting treatment under-specified |
| Multi-project payments | Standard OOB | Settlement handles multiple invoices |
| Payment journal workflow | Standard OOB | Configurable approval |

**Recommendations:**

1. **>> ATTENTION AREA: Advance payment handling needs more detail**

   > The FRD mentions "Technica have advanced payment before doing the settlement" but does not specify the accounting treatment. Without a dedicated prepayment posting profile, advances could be incorrectly posted directly to AR (overstating receivables) instead of a customer advance liability account. This has financial statement implications.

   **>> ENHANCEMENT:** For a make-to-order manufacturer, advance payments are common (deposit before production). The proper D365 approach is:
   - Create a **customer prepayment journal** using a dedicated prepayment posting profile
   - This posts to a **Customer Advances (liability)** account, not directly to AR
   - When the project milestone invoice is posted, the advance is **settled** against the invoice
   - This ensures proper balance sheet classification (advances as liabilities, not AR)

   **Action required:** Define a prepayment posting profile and the advance payment ledger account. This is configuration, not customization, but must be designed explicitly.

2. **>> ATTENTION AREA: Bank reconciliation is not mentioned**

   > For a company receiving bank transfers (the primary payment method), automated bank reconciliation is critical. D365 F&O provides Advanced Bank Reconciliation with import of bank statements (MT940, BAI2, CAMT.053 formats). This is a major efficiency gain for payment processing.

   **>> ENHANCEMENT:** Add Advanced Bank Reconciliation to scope for automated payment matching. This dramatically reduces manual payment matching effort.

3. **The cross-project payment scenario** (one payment journal covering invoices from multiple projects) is standard but requires attention to the **financial dimension** assignment on the payment line. Ensure the payment offset correctly distributes across projects when settling multiple project invoices.

4. **>> ENHANCEMENT:** **No mention of electronic payment methods.** For foreign customers, consider configuring **SEPA credit transfer**, **wire transfer templates**, or **letter of credit** payment methods. The current "Cash & Bank Transfer" is too generic for international manufacturing operations.

---

### AR-003: Sales Order Processing

**What was proposed:**
- Standard sales order cycle: Create SO > Confirm > Packing Slip (partial delivery) > Invoice > Payment
- Sales invoice layout modification (GAP)
- Black market rate invoicing with mixed VAT invoices (GAP - refers to GL FRD)

**Assessment: Standard process with two GAPs requiring attention**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Sales order processing | Standard OOB | Create > Confirm > Ship > Invoice > Pay |
| Partial delivery (packing slip) | Standard OOB | Standard functionality |
| Sales invoice generation | Standard OOB | From sales order |
| Invoice layout modification | GAP | SSRS report customization |
| Black market rate invoicing | **>> RISKY <<** | Complex — multi-currency/rate scenario, highest-complexity GAP |

#### GAP AR003-002: Sales Invoice Report Layout Modification

| Attribute | Assessment |
|-----------|------------|
| **Description** | Modify the sales invoice printout to match Technica's format |
| **Complexity** | Low-Medium |
| **Recommended Approach** | **>> ENHANCEMENT:** Same approach as AR001-002. Clone the standard SalesInvoice report design and apply Technica layout. Use ER framework if electronic format is needed. Consolidate this effort with the FTI report modification -- both should follow the same branding template. |
| **Effort Estimate** | 8-16 hours |

#### GAP AR003-003: Black Market Rate Invoicing

**>> ATTENTION AREA: Highest-complexity GAP in the entire FRD**

> This is a Lebanon-specific regulatory/economic requirement arising from the dual exchange rate environment. The solution must handle invoicing where the base amount uses one exchange rate and VAT uses a different rate (the official rate of 15,000 LBP/USD). This is not a standard D365 scenario and cuts across GL, Tax, AR, and Sales modules.

| Attribute | Assessment |
|-----------|------------|
| **Description** | Mixed invoices with VAT calculated at a "black market" exchange rate (likely the parallel market rate in Lebanon, where official and market exchange rates diverge significantly) |
| **Complexity** | High |
| **FRD Proposed Solution** | Detailed in GL FRD (VAT section) |
| **Risk Assessment** | This is a **Lebanon-specific regulatory/economic requirement** arising from the dual exchange rate environment. The solution must handle invoicing where the base amount uses one exchange rate and VAT uses a different rate (the official rate of 15,000 LBP/USD). This is not a standard D365 scenario. |
| **Recommended Approach** | **>> ENHANCEMENT:** This GAP is cross-FRD (GL + AR + Sales). The solution must be designed holistically. Typical approaches: **Option A: Custom exchange rate type** -- Create a separate exchange rate type (e.g., "VAT Rate") and configure the tax calculation engine to use this rate instead of the default transaction rate. This requires X++ customization in the tax calculation pipeline. **Option B: Manual tax override** -- Allow users to manually adjust the calculated tax amount on the invoice. This is simpler but error-prone and does not scale. **Option C: Tax calculation service** -- Use the D365 Tax Calculation microservice with custom logic for rate differentiation. This is the most modern approach but adds architectural complexity. |
| **Effort Estimate** | 40-80 hours depending on approach (significant custom development) |

**Recommendations:**

1. **Sales order processing overlaps with FRD01 (Sales) and FRD04 (Project Accounting).** For Technica's make-to-order business, most "sales" are actually project contracts billed through milestones, not standard sales orders. The standard sales order process in AR-003 likely applies to a **smaller volume** -- spare parts sales, non-project items, or one-off equipment sales. Clarify the volume split between project-invoiced revenue and SO-invoiced revenue to properly size the effort.

2. **>> ENHANCEMENT:** **The black market rate GAP is the single highest-complexity item** in this FRD. It must be resolved at the GL/Tax level and then verified end-to-end through AR. Ensure this is tracked as a cross-FRD design item with its own technical design document.

---

### AR-004: Foreign Currency Revaluation

**What was proposed:**
- Daily exchange rate updates ("TECH" exchange rate type)
- Invoices and payments primarily in EUR and USD
- Monthly foreign currency revaluation process
- Unrealized gain/loss posted to GL

**Assessment: Fully standard -- all FIT**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Exchange rate daily update | **>> NEEDS REVIEW <<** | Manual entry is error-prone — automate import |
| Custom "TECH" exchange rate type | Standard OOB | Allows company-specific rates |
| Monthly revaluation | Standard OOB | Periodic batch job |
| Unrealized gain/loss posting | Standard OOB | Automatic journal entries |

**Recommendations:**

1. **>> ENHANCEMENT:** **Automate exchange rate import.** D365 F&O supports automatic exchange rate import from providers (ECB, CBR, HMRC, etc.). For "TECH" custom rates, set up a **periodic import job** from an Excel file or API if Technica uses internal rates. Manual daily entry is error-prone and time-consuming.

2. **>> ATTENTION AREA: Lebanon dual-rate environment affects exchange rate type definition**

   > The "TECH" exchange rate type must be clearly defined — is it the official rate, the market rate, or a company-determined rate? This directly impacts the black market rate GAP (AR003-003) and the revaluation gain/loss calculations.

   **>> ENHANCEMENT:** Ensure the "TECH" exchange rate type definition is aligned with the dual-rate invoicing solution designed for GAP AR003-003.

3. **Consider running revaluation more frequently than monthly** during periods of high currency volatility. D365 supports any frequency -- weekly revaluation may provide more accurate interim financial statements, which is particularly relevant in the volatile LBP/USD/EUR environment.

4. **Revaluation of advance payments** deserves attention. If customers make advance payments in foreign currency, these balances must also be revalued. Ensure the prepayment posting profile is included in the revaluation parameters.

---

### AR-005: Intercompany Trade Operations (PO/SO)

**What was proposed:**
- Intercompany customer/vendor setup across legal entities
- Automatic SO creation in selling company when PO is created in buying company
- Trade agreement pricing for intercompany transactions
- Automatic copying of packing slip, product receipt, and invoice numbers
- Policy-based automation (picking, packing, invoicing, payment posting)

**Assessment: Standard intercompany trade -- all FIT**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Intercompany customer/vendor setup | Standard OOB | Bidirectional relationship |
| Auto SO from PO | Standard OOB | Core intercompany feature |
| Price setup via trade agreements | Standard OOB | Consistent with AR trade agreement setup |
| Document number synchronization | Standard OOB | Packing slip, receipt, invoice |
| Policy automation | **>> NEEDS REVIEW <<** | Conservative approach recommended at go-live |

**Recommendations:**

1. **>> ENHANCEMENT:** **Transfer pricing compliance.** Intercompany prices must comply with transfer pricing regulations. The FRD mentions using trade agreements for intercompany pricing, which is correct. Ensure transfer pricing documentation is maintained and that the pricing method (cost-plus, comparable, etc.) is approved by Technica's tax advisors. D365 can enforce this through trade agreement validation.

2. **>> ATTENTION AREA: Intercompany elimination not mentioned**

   > The FRD does not mention intercompany elimination entries for consolidated financial statements. Ensure the GL setup (FRD - GL/Cash/Bank) includes elimination rules for intercompany AR/AP balances during consolidation.

3. **Number sequence design** for intercompany orders is a detail that matters operationally. The FRD shows a combined number sequence option (company code + PO number). This is the recommended approach -- it provides immediate traceability of which entity originated the order.

4. **>> ENHANCEMENT:** **Automation level should be conservative initially.** The FRD mentions the ability to automate packing slip posting, invoicing, and payment posting. For go-live, configure only the auto-SO creation and let users manually process the remaining steps. Automate additional steps post-stabilization once the team is comfortable.

---

### AR-006: Inquiries & Reports

**What was proposed:**
- Customer Account Statement (with date interval, credit limit, due dates)
- Customer Invoice Transaction Report (open/paid/both, date range)
- Customer Transactions Inquiry (detailed customer data)
- Customer Aging Report (standard aging)
- All four reports require modification to include Project ID (FIT/GAP)

**Assessment: Standard reports with consistent modification requirement**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Customer Account Statement | Standard OOB | With date filters |
| Invoice Transaction Report | Standard OOB | Open/paid filter |
| Customer Transactions Inquiry | Standard OOB | Detailed inquiry form |
| Customer Aging Report | Standard OOB | With aging buckets |
| Project ID on all reports | **>> NEEDS REVIEW <<** | Requires report modification or Power BI — test financial dimension approach first |

#### GAP AR006-001 through AR006-004: Project ID on AR Reports

**>> ATTENTION AREA: Four reports need Project ID — test financial dimension approach before custom development**

> Project ID is stored on customer invoice transactions (via project invoice proposals). The data is available — the challenge is surfacing it on standard AR reports that were not designed with project dimensions. If Project is a financial dimension, many standard inquiries already support dimension filtering, potentially resolving the requirement with zero customization.

| Attribute | Assessment |
|-----------|------------|
| **Description** | Add Project ID column to all four standard AR reports |
| **Complexity** | Medium (aggregated across 4 reports) |
| **FRD Assessment** | Marked as "FIT/GAP" with note "Modify report to include Project ID if possible" |
| **D365 Reality Check** | Project ID is stored on customer invoice transactions (via project invoice proposals). The data is available -- the challenge is surfacing it on standard AR reports that were not designed with project dimensions. |
| **Recommended Approach** | **>> ENHANCEMENT:** **Option A (Preferred): Power BI report pack.** Build a single "AR Analytics" Power BI report that includes all four report types as separate pages/tabs with Project ID as a standard dimension. This is the most maintainable approach: one Power BI model serves all four needs, users get drill-through from aging to transactions to invoices, and Project ID filtering is native. **Option B: Modify standard SSRS reports** to include a Project ID column. This requires modifying the underlying data provider (DP class) for each report to join project transaction data. Estimated effort: 12-20 hours per report (48-80 hours total). **Option C: Financial dimension approach.** If Project is configured as a financial dimension (referenced in FRD04 PJ001-018), the dimension value will appear on transactions and can be included in standard reports via dimension filters. This may partially address the need without custom development -- test this first. |
| **Effort Estimate** | Option A: 24-40 hours (Power BI). Option B: 48-80 hours (SSRS). Option C: 4-8 hours (configuration + testing). |

**Recommendations:**

1. **>> ENHANCEMENT:** **Test the financial dimension approach first (Option C).** If Project is a financial dimension on customer transactions, many standard inquiries and reports already support dimension filtering and display. This could resolve the requirement with zero customization.

2. **>> ENHANCEMENT:** **If Power BI is chosen (Option A),** align this effort with the Power BI reporting recommended in FRD04 (Project Accounting) and FRD07 (Procurement). A unified Power BI workspace for Finance (covering AR, AP, GL, Project) is more efficient than building reports per module.

3. **The aging report is the most critical.** For a project-based manufacturer with long payment cycles, aging by project provides actionable collection data. Ensure the aging report can show aging per project contract, not just per customer.

4. **>> ENHANCEMENT:** **Missing report: Cash application report.** The FRD does not mention a report for unapplied cash / advance payments. For a business that takes advances before invoicing, tracking unapplied customer prepayments is essential. Add this to the reporting scope.

---

## Critical Missing Coverage: Project Milestone Invoicing Integration

**>> ATTENTION AREA: Primary revenue channel is not designed in this FRD**

> The most significant omission in this FRD is the lack of explicit design for how project milestone invoices (FRD04, PJ003) flow through Accounts Receivable. For Technica — a make-to-order manufacturer where revenue flows through project contracts — the AR module's primary function is managing receivables from project milestone invoices. This was flagged by a reviewer in the Document Approvals section but remains unresolved.

**What should be addressed:**

| Topic | Expected Coverage | Status in FRD |
|-------|-------------------|---------------|
| Milestone invoice posting to AR | How does PJ003 invoice proposal create AR transactions? | Not covered (deferred to FRD04) |
| Retention invoicing | How is retained amount invoiced upon project completion? | Not covered |
| Advance payment application to milestone invoices | How are customer prepayments offset against milestone invoices? | Mentioned briefly in AR-002 but no process detail |
| Revenue recognition ↔ AR | How does POC% accrued revenue relate to AR balances? | Not covered |
| Project-based dunning/collections | Should collections prioritize by project contract value? | Not covered |
| Customer statements with project detail | How do statements show milestone invoice detail? | Partially covered (AR006 Project ID GAP) |

**>> ENHANCEMENT:** Create an **AR-Project Integration Design Addendum** that explicitly maps the end-to-end flow from project milestone completion through invoice proposal, AR posting, advance payment settlement, and collection. This is the core revenue cycle for Technica and must not remain implicit across two separate FRDs.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | Customer Setup | Verify if OOB customer creation workflow exists in target D365 build before custom development | High | Avoids unnecessary customization |
| 2 | Customer Setup | Expand customer groups beyond Local/Foreign to include Distributor and Intercompany segments | Medium | Better trade agreements, reporting, and posting control |
| 3 | Customer Setup | Add a second posting profile for intercompany customers | Medium | Cleaner balance sheet and easier intercompany reconciliation |
| 4 | Customer Setup | Use Dual-Write or Power Automate for credit hold sync to CRM (GAP 001-013) | High | Real-time credit enforcement across CRM and F&O |
| 5 | AR-001 | Use ER framework for invoice report customization instead of SSRS modification | Medium | Business-user-maintainable templates |
| 6 | AR-002 | Define prepayment posting profile and advance payment accounting treatment explicitly | High | Correct balance sheet classification of customer advances |
| 7 | AR-002 | Add Advanced Bank Reconciliation to scope for automated payment matching | High | Major efficiency gain for payment processing |
| 8 | AR-003 | Track black market rate invoicing as a cross-FRD design item with dedicated technical design | Critical | Highest complexity GAP -- affects AR, GL, and Tax |
| 9 | AR-004 | Automate exchange rate import instead of manual daily entry | Medium | Reduces errors and manual effort |
| 10 | AR-005 | Keep intercompany automation conservative at go-live; expand post-stabilization | Medium | Reduces go-live risk |
| 11 | AR-006 | Test financial dimension approach for Project ID on reports before building custom reports | High | Could resolve 4 GAPs with zero development |
| 12 | AR-006 | Build unified Finance Power BI workspace for AR reporting (shared with GL, AP, Project) | Medium | Single source of truth for financial analytics |
| 13 | Cross-FRD | Create AR-Project Integration Design Addendum for milestone invoicing end-to-end flow | Critical | Core revenue cycle must be explicitly designed |
| 14 | AR-002 | Define electronic payment methods for foreign customers (SEPA, wire templates, LC) | Medium | Supports international operations |

---

## Risk Flags

1. **>> ATTENTION AREA: Project milestone invoicing is the primary revenue channel but is not designed in this FRD.**

   > The AR document focuses on free text invoices, sales orders, and intercompany trade — but for a make-to-order manufacturer using fixed-price project contracts, the vast majority of AR transactions originate from project invoice proposals. The absence of this integration design creates a risk that AR configuration (posting profiles, aging, collections, reporting) will not properly support the project invoicing flow. This must be resolved before configuration begins.

2. **>> ATTENTION AREA: Black market rate invoicing (AR003-003) is the highest-complexity GAP**

   > This cuts across GL, Tax, AR, and Sales modules. The FRD defers to the GL FRD for the solution, but the AR implications (customer statements, aging, revaluation, payment application) must be explicitly tested. A misconfigured dual-rate tax calculation can cause audit failures and regulatory non-compliance.

3. **>> ATTENTION AREA: Payment terms are undefined**

   > The FRD states Technica is "working on updating their existing ones." Payment terms directly affect aging calculations, cash flow forecasting, collection strategies, and credit management. This cannot remain open past the design phase.

4. **>> ATTENTION AREA: Advance payment accounting is under-specified**

   > The FRD mentions advance payments but does not define the posting treatment. Without a dedicated prepayment posting profile, advances could be incorrectly posted directly to AR (overstating receivables) instead of a customer advance liability account. This has financial statement implications.

5. **Report modification scope may grow.** Four reports need Project ID columns, and two reports need layout changes -- totaling six report customizations. If done as SSRS modifications, this is a significant development effort (potentially 80+ hours). The financial dimension approach or Power BI alternative could dramatically reduce this.

6. **Cross-Company Data Sharing for customers** is powerful but requires careful planning. Sharing customer records across entities means changes in one entity propagate to others. Ensure Technica understands that shared fields cannot differ between entities. For fields that must differ (e.g., payment terms by entity, credit limits by entity), exclude them from the sharing policy.

---

## Efficiency Verdict

The Accounts Receivable FRD provides a **competent, standard-leaning solution** that correctly leverages D365 F&O OOB functionality for the majority of requirements. The customer master setup is thorough (trade agreements, posting profiles, aging brackets, collections, credit hold), and the intercompany trade process is well-documented with clear screenshots.

However, the FRD has **three structural weaknesses** that must be addressed:

1. **The project milestone invoicing integration is conspicuously absent.** For Technica -- a make-to-order manufacturer where revenue flows through project contracts -- the AR module's primary function is managing receivables from project milestone invoices. This must be explicitly designed, not left as a cross-reference to FRD04.

2. **The black market rate invoicing scenario is the single hardest technical challenge** in the AR scope and is under-designed. A dedicated technical design document is warranted.

3. **Several operational decisions are deferred to migration phase** (payment terms, credit hold rules, workflow details, report templates). While some deferral is normal, these items affect AR configuration fundamentally and should be resolved during the design phase to avoid rework.

The areas where the FRD excels: trade agreement setup (detailed procedure with screenshots), intercompany trade operations (comprehensive walkthrough), aging bracket design (extended to 3 years -- appropriate for long project cycles), and collection management (centralized agent workspace).

**Net assessment: 70% of the AR solution is solid OOB configuration. The remaining 30% requires targeted design work -- primarily around project integration, advance payments, and the dual-rate tax scenario -- to bring this FRD to implementation-ready status.**

---

*This review is based on D365 F&O Accounts Receivable, Credit & Collections, and Project Accounting best practices as of 2026. Recommendations aim to reduce customization, ensure proper project-AR integration, and leverage platform capabilities including Power BI and Electronic Reporting.*
