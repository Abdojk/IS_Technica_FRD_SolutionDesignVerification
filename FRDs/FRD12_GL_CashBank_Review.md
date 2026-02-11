# FRD12 - GL / Cash & Bank - Solution Design Verification Report

**Document:** Technica - FRD12-GL-CashBank-V3.0
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Finance & Operations (F&O) - General Ledger, Cash & Bank Management

---

## Executive Summary

This FRD covers the financial backbone of Technica International's D365 F&O implementation: General Ledger setup, Chart of Accounts, Financial Dimensions, Journal processing, Tax, Period Close, Foreign Currency Revaluation, Intercompany Accounting, Consolidation & Elimination, and Cash & Bank Management including both manual and advanced bank reconciliation. The document spans **16 business processes** (SY001-SY003, GL001-GL014, BA001-BA002) with a total of **47 requirements**.

**Overall assessment: Largely standard and well-aligned with D365 OOB capabilities.** Out of 47 requirements, only **1 is a GAP** (GL006-002: VAT exchange rate handling for Lebanese market with mixed fresh USD / 15,000 LBP rates). The solution leverages Poland localization for AP-side VAT, which is a known and proven workaround for multi-rate VAT environments. The AR-side requires customization.

Key concerns center around: (1) the **dual-rate currency environment** unique to Lebanon, (2) **financial dimension design decisions** that may limit future reporting, (3) a **large number of bank accounts (38)** that need careful lifecycle management, and (4) the **"Technica ND" fictitious entity** for non-declared transactions, which raises compliance flags.

---

## Process-by-Process Review

### SY001 - Legal Entities

**What was proposed:**
- 2 active legal entities: Technica International SAL (main operations, employees, inventory) and Technica Offshore SAL (project-based, trading, tax purposes)
- 1 holding company (minimal activity, accounting transactions only)
- 1 fictitious company "Technica ND" for non-declared transactions
- Other entities with transactions but no stock

**Assessment: Functional but carries compliance risk**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Main entity structure (TI SAL + Offshore) | Good | Clear separation of operational and trading entities |
| Holding company | Acceptable | Low activity; standard consolidation target |
| Technica ND (fictitious entity) | Risk Flag | Non-declared transactions in an ERP raise audit and compliance concerns |
| Other entities | Unclear | "Having operations with no stock" needs clearer definition |

**Recommendations:**

1. **Technica ND entity is a significant red flag.** Having a "fictitious company used for non-declared transactions" inside D365 F&O creates an auditable trail of transactions that are by definition not declared. If Technica proceeds, this entity should have extremely restricted access, separate number sequences, and the accounting team must understand that D365 maintains full audit logs. **Recommendation:** Discuss with Technica's legal/compliance team whether this entity should exist in D365 at all. If it must, ensure proper segregation and access controls.

2. **Entity count affects licensing and consolidation complexity.** Each legal entity requires its own ledger, COA assignment, number sequences, and period close procedures. Ensure the total entity count is finalized before go-live to avoid rework. The FRD mentions entities may be consolidated during migration.

3. **Technica Offshore SAL** is described as having "no inventory for the time being" but billing invoices. This is a classic transfer pricing scenario. Ensure intercompany accounting (GL013) properly handles the transaction flow between TI SAL (where inventory/production exists) and Offshore SAL (where invoicing occurs).

---

### SY002 - Fiscal Calendar

**What was proposed:**
- Calendar year (January 1 - December 31) with 14 periods (Opening + 12 months + Closing)
- Period control: Open / On-Hold / Closed states
- Accounting Manager manages period status
- "On-Hold" preferred over "Closed" until final audit is complete
- One fiscal calendar shared across entities

**Assessment: Standard and correct**

| Aspect | Rating | Comment |
|--------|--------|---------|
| 14-period calendar structure | Standard OOB | Correct approach with opening/closing periods |
| On-Hold vs. Closed strategy | Excellent | Prevents irreversible closure before audit completes |
| Single calendar across entities | Appropriate | Simplifies period management for related entities |
| Segregation of duty | Good | Accounting department controls period status |

**Recommendations:**

1. **Period module-level access control is not mentioned.** D365 allows periods to be open/on-hold per module (GL, AP, AR, Inventory, etc.). Technica should define a period close calendar that specifies the order: Inventory closes first, then AP/AR, then GL closes last. This prevents sub-ledger postings after GL reconciliation.

2. **Consider using the Financial Period Close workspace** in D365 for managing the close checklist. It provides task assignment, due dates, and status tracking, which is superior to managing close activities informally.

---

### SY003 - Currencies & Currency Exchange Rates

**What was proposed:**
- Base currency: LBP (Lebanese Pound) with 3 decimal precision
- Reporting currency: USD with 3 decimal precision
- Maximum penny difference: 0.100
- Custom exchange rate type named "Tech"
- Manual exchange rate entry by accounting team (daily adjustable)
- Future enhancement: automated rate feed from international API
- Multiple currencies in use (USD, LBP, EUR, GBP, and others)

**Assessment: Technically correct but operationally challenging**

| Aspect | Rating | Comment |
|--------|--------|---------|
| LBP as base / USD as reporting | Appropriate | Reflects legal requirement to report in LBP |
| 3 decimal precision | Standard | Adequate for LBP and USD |
| Custom exchange rate type "Tech" | Acceptable | Allows Technica-specific rates separate from system default |
| Manual rate entry | Risky | Error-prone for daily operations in volatile currency market |
| Penny difference at 0.100 | Needs Review | May be too tight given LBP/USD volatility |

**Recommendations:**

1. **The Lebanese currency environment is exceptionally complex.** Lebanon effectively operates with multiple exchange rates (official BdL rate, Sayrafa/platform rate, parallel/black-market rate). The FRD acknowledges this ("black market rate," "Sayrafa rate," "fresh dollar") but does not clearly define which rate type is used for which transaction type. **Recommendation:** Create an explicit exchange rate type matrix:

   | Transaction Type | Rate Type | Source |
   |-----------------|-----------|--------|
   | Fresh USD invoices | Black market rate | Manual / API |
   | LBP transactions | 1:1 | N/A |
   | VAT calculation | Official 15,000 LBP | Fixed parameter |
   | Reporting currency translation | Daily market rate | Manual / API |

2. **Automate exchange rate feeds as early as possible** rather than treating it as a "future enhancement." In a multi-rate environment with 38 bank accounts in 4 currencies, manual rate entry is a significant operational bottleneck and error source. D365 supports exchange rate providers (ECB, CBRF, etc.) OOB. For the Lebanese parallel rate, a Power Automate flow pulling from a trusted source and pushing to D365 via Data Entity is straightforward.

3. **Penny difference of 0.100 may cause posting failures** when LBP amounts are involved. With LBP base currency, rounding differences on large transactions can easily exceed 0.100. Monitor this during UAT and be prepared to increase it. For context, many Lebanese implementations use penny differences of 1.000 or higher.

4. **Reporting currency (USD) implications are significant.** Every posted transaction generates both LBP and USD accounting entries. This is correct for dual-reporting, but Technica must understand that the reporting currency rate applies at the time of posting and cannot be retroactively changed without reversal. If the market rate fluctuates significantly intraday, the posting time matters.

---

### GL001 - Create a Ledger Account Number (COA)

**What was proposed:**
- Main Account with Type, Account Type, Reporting Type, Account Category
- Foreign currency revaluation flag per account
- Consolidation account unique per main account (to be provided during migration)
- Suspended accounts for inactive COA entries
- Financial Dimensions: Department, Business Unit, Project, Customers (as mandatory on AR control accounts)

**Assessment: Well-structured with one design concern**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Main account setup attributes | Standard OOB | All standard fields utilized |
| Consolidation account mapping | Good | Essential for GL014 consolidation process |
| Suspending inactive accounts | Good practice | Prevents erroneous postings |
| Customer as financial dimension on AR | Creative but risky | See recommendation below |
| Financial dimension from table (auto) | Standard OOB | Workers, Customers, Vendors auto-populated |
| Custom dimensions (manual) | Standard OOB | Department, Business Unit maintained manually |

**Recommendations:**

1. **Customer as a mandatory financial dimension on AR control accounts is unconventional and problematic.** The stated goal is to extract trial balance by account by project by customer. However:
   - D365 already tracks customers at the sub-ledger level on AR transactions. The Customer is always available on the AR transaction line.
   - Adding Customer as a financial dimension on the control account creates a **dimensional explosion** — every new customer creates a new dimension value, and every AR posting must include the customer dimension.
   - This makes the trial balance excessively granular and complicates reconciliation.
   - **Better alternative:** Use the standard **Customer transaction list** and **Aged receivables** reports, which natively break down by customer without needing a financial dimension. For combined project-customer analysis, use **Power BI** connecting project invoices to customer data.
   - If Technica absolutely needs customer on the trial balance, consider making it a **derived financial dimension** that auto-populates from the sub-ledger rather than a manually-entered dimension.

2. **Financial dimensions confirmed as "optional only" (not mandatory on any account).** While this reduces posting friction, it also means dimension values can be omitted, leading to "blank" dimension entries that make reporting incomplete. **Recommendation:** At minimum, make Department and Business Unit mandatory on P&L accounts (expenses and revenue). This ensures cost center reporting is always complete. Use default financial dimension templates on journal names to reduce manual entry burden.

3. **The COA numbering scheme uses 7-digit account numbers** (e.g., 1122010, 5120030, 6262000). This is adequate. Ensure the numbering has sufficient gaps between ranges for future accounts. The structure appears to follow:
   - 1xxxxxx = Assets
   - 2xxxxxx = Liabilities
   - 3xxxxxx = Equity
   - 4xxxxxx = Revenue/Suppliers
   - 5xxxxxx = Cash/Bank
   - 6xxxxxx = Expenses

   **Note:** Bank accounts in the 51xxxxx range is unusual — typically banks are in the 1xxxxxx (Asset) range. Verify this is intentional and aligns with Technica's legacy COA structure.

---

### GL002 - Create/Update Main Account Structure

**What was proposed:**
- Multiple account structures:
  - Main Account only
  - Main Account + Project
  - Main Account + Project + Department
  - Projects
- Advanced Rules for exceptions (e.g., account 40110002 gets Projects dimension in addition to Department + Legal Entity)
- Dimension restrictions discussed but Technica confirmed no mandatory dimensions
- Access restricted to Finance Manager and Chief Accountant

**Assessment: Reasonable but overly complex structure design**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Multiple account structures | Concern | 4+ structures add validation complexity |
| Advanced Rules for exceptions | Acceptable | Needed for edge cases |
| Access control on structure form | Excellent | Only senior finance staff can modify |
| No mandatory dimensions | Risk | See GL001 recommendations |

**Recommendations:**

1. **Simplify account structures.** Having 4+ account structures means the system must validate each transaction against the correct structure, and users must understand which structure applies to which account range. In practice, **a single account structure with optional dimensions works better** for most implementations. Design:
   - One structure: Main Account + Business Unit + Department + Project (all optional)
   - Use Advanced Rules only for the truly exceptional cases (e.g., 40110002 needing Legal Entity dimension)
   - This simplifies maintenance and reduces user confusion

2. **The account structure examples are inconsistent with the dimension list.** The FRD lists dimensions as Department, Business Unit, Project, and Customers, but the structure examples reference "1* accounts with Business Unit" and "2* accounts with Department & BU." Ensure the final structure design is documented in a single, clear matrix before migration.

3. **Advanced Rules should be minimized.** Each Advanced Rule is an additional validation layer that impacts posting performance and confuses users who get unexpected dimension prompts. Only use Advanced Rules where the business requirement is absolute.

---

### GL003 - Create GL Journal and Adjustments

**What was proposed:**
- Multiple journal names for different transaction types (Daily, Adjustment, Accrual, etc.)
- Workflow approval per journal name
- Attachment feature on journals
- Three currencies visible per journal line (local, reporting, transaction)
- Manual rate adjustment available for foreign transactions
- No posting restrictions or journal controls initially (but awareness documented)
- Multiple voucher scenarios documented (balanced lines, single offset, separate vouchers)

**Assessment: Standard and well-documented**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Journal name segregation | Good | Clear separation by purpose |
| Workflow approval per journal | Excellent | Proper control framework |
| Attachment capability | Standard OOB | Already available |
| Three-currency visibility | Standard OOB | Critical for Lebanese operations |
| Manual rate override | Available OOB | Use with caution |
| No posting restrictions initially | Acceptable | Can enable later as controls mature |
| Voucher handling scenarios | Well-documented | Clears up common confusion |

**Recommendations:**

1. **Reconsider the decision to skip posting restrictions and journal controls.** While the FRD acknowledges these features exist, Technica chose not to use them. For a company with multiple entities and intercompany transactions, journal controls provide critical guardrails:
   - AP journals should not allow customer accounts
   - AR journals should not allow vendor accounts
   - Bank journals should only allow bank main accounts as offset
   - **Recommendation:** Implement basic journal controls from go-live. The effort is minimal (configuration only), and it prevents mis-postings that are painful to reverse.

2. **"Blocking for user groups" should be enabled for intercompany journals.** The FRD mentions this as a future possibility, but given that Technica has multiple legal entities and an intercompany journal type, restricting who can post intercompany transactions is a day-one control requirement.

3. **Number sequences** will be provided during migration. Ensure number sequences are:
   - Continuous (no gaps) for legal compliance in Lebanon
   - Scoped per legal entity (not shared across entities)
   - Pre-allocated to avoid performance bottlenecks on high-volume posting

4. **Foreign currency rate adjustment on journal lines** should be governed by policy. If any user can override the exchange rate on a journal entry, this is a control weakness. **Recommendation:** Document when manual rate override is acceptable (e.g., contractual rates) and configure workflow to flag journals where the rate deviates from the system rate by more than a threshold (e.g., 2%).

---

### GL004 - Create GL Accruals

**What was proposed:**
- Ledger Accrual schemes to redistribute costs/revenues over periods
- Used for fees accrued over the fiscal year (balance sheet → expense allocation)
- Debit and credit accounts set to same balance sheet prepaid account
- Voucher: "New voucher for each transaction date"
- Integration with vendor invoice journal (Functions → Ledger Accruals)
- Future transactions auto-posted on schedule

**Assessment: Standard and correctly configured**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Accrual scheme setup | Standard OOB | Proper use of built-in functionality |
| Prepaid → Expense allocation | Correct | Standard prepaid expense treatment |
| New voucher per transaction date | Good practice | Clean audit trail per period |
| Integration with vendor invoice journal | Standard OOB | Seamless workflow |
| Auto-posting of future transactions | Standard OOB | Reduces manual work |

**Recommendations:**

1. **Accrual schemes should include a review step before period-end.** While auto-posting is convenient, someone should review the upcoming period's accrual entries before they post. **Recommendation:** Use the accrual scheme with "Proposal only" option for the first 2-3 months to validate entries, then switch to auto-posting once the team is confident.

2. **Accrual reversal handling is not addressed.** What happens when an accrued invoice is cancelled or credited? D365 does not automatically reverse accrual scheme entries when the source invoice is reversed. **Recommendation:** Document the manual process for accrual reversal and include it in the month-end close checklist.

3. **Consider subscription billing or prepayment functionality** for recurring fee accruals if the volume is significant. The ledger accrual scheme is best for ad-hoc prepayments; for systematic recurring accruals, Revenue Recognition features in D365 may be more appropriate.

---

### GL005 - Allocation

**What was proposed:**
- Move from manual Excel-based allocation to D365 allocation functionality
- Fixed percentage allocation method
- Allocate shared department costs and shared legal entity costs across departments and legal entities
- Allocation journal with "Proposal only" option for review before posting

**Assessment: Good upgrade from manual process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Ledger allocation rules | Standard OOB | Proper configuration |
| Fixed percentage method | Appropriate | Technica's requirement |
| Cross-department allocation | Standard OOB | Core use case |
| Cross-legal-entity allocation | Standard OOB | Supports intercompany cost sharing |
| Proposal option before posting | Good practice | Allows review |

**Recommendations:**

1. **Allocation percentages need a governance process.** The FRD describes the setup but not who approves allocation percentage changes, or how often they're reviewed. **Recommendation:** Establish a quarterly review of allocation percentages with Finance Manager sign-off. Changes to allocation rules should go through a change management process.

2. **Consider the Cost Accounting module for more sophisticated allocations.** D365's Cost Accounting module provides:
   - Multi-level allocation (primary + secondary cost distribution)
   - Statistical bases (headcount, square footage, machine hours) instead of fixed percentages
   - Overhead calculation sheets
   - If Technica's allocation needs grow beyond simple fixed percentages, Cost Accounting is the natural evolution. Worth keeping in mind for Phase 2.

3. **Allocation timing matters for reporting.** Ensure allocations run **after** all source transactions are posted for the period but **before** financial statements are generated. Add allocation execution to the month-end close checklist with a specific sequence number.

---

### GL006 - Tax

**What was proposed:**
- Lebanon VAT at 11% on 15,000 LBP rate
- Different VAT rates for different entities/countries (Europe, Canada with 2 taxes)
- Poland localization activated for AP-side VAT handling (calculating VAT at 15,000 LBP rate)
- AR-side customization required for B2B USD invoices:
  - Revenue lines at black-market rate
  - VAT lines at 15,000 LBP rate
  - Receivable line = sum of above
  - Settlement handles rate differences with exchange difference entries
- Customer statement updated to show initial amount and VAT amount in separate columns

**Assessment: The most complex area of this FRD**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Standard VAT setup (tax codes, groups) | Standard OOB | No issues |
| Poland localization for AP VAT | Creative workaround | Known solution for multi-rate VAT |
| AR-side customization for B2B invoices | GAP - Complex | Dual-rate calculation on single invoice |
| Settlement with exchange differences | Technically correct | But generates high volume of exchange entries |
| Customer statement modification | GAP - Medium | Report customization |

**GAP Analysis: GL006-002 - VAT Exchange Rate Issue**

| Attribute | Assessment |
|-----------|------------|
| **Complexity** | High |
| **Risk** | Medium-High (affects every B2B USD sales invoice) |
| **Volume Impact** | Every B2B USD invoice generates dual-rate accounting |
| **Testing Effort** | Extensive - must cover invoicing, settlement, partial payment, credit notes |

**Recommendations:**

1. **The Poland localization for AP is a proven workaround** used across Middle Eastern and African D365 implementations where official vs. market rates diverge. This is an acceptable approach. However, **document extensively** that Poland localization is enabled and why, so future support teams understand the configuration rationale.

2. **The AR-side customization is the highest-risk item in this FRD.** The customization must intercept the sales invoice posting routine to split the accounting entry into two rate calculations on the same voucher. Specific concerns:
   - **Credit note handling:** When a B2B USD sales order is credited, the customization must reverse using the original rates (not current rates). Ensure this is in scope.
   - **Partial payments:** If a customer pays 50% of an invoice, the system must correctly split the settlement between the revenue portion (at black-market rate) and the VAT portion (at 15,000 LBP). This is non-trivial.
   - **Reporting:** The customization changes how accounting entries look on the voucher. Ensure all downstream reports (trial balance, AR aging, tax reports) still work correctly.

3. **Alternative approach worth evaluating:** Instead of customizing the sales invoice posting, consider using **two separate invoice lines** — one for the goods/services (in USD at market rate) and one for VAT (in LBP at 15,000 rate). This is less elegant from a user perspective but avoids deep customization of the posting engine. The downside is that it changes how invoices look to customers.

4. **Tax settlement process is not detailed.** How does Technica file VAT returns? If the government requires VAT declared at 15,000 LBP, the tax settlement report must output amounts at that rate, not the black-market rate. Verify the Poland localization handles this correctly for the tax authority reporting requirement.

5. **Withholding tax** is mentioned for European countries but not detailed. If Technica Offshore or other entities operate in jurisdictions with withholding tax, ensure WHT setup is included in migration scope.

---

### GL007 - Month End Close

**What was proposed:**
- Comprehensive close checklist including:
  - Post all open journal vouchers
  - Subledger-to-ledger reconciliation (FA, AR, AP, Bank, Inventory)
  - Open prepayment report in LBP
  - Foreign currency revaluation (AP, AR, Bank, other BS accounts)
  - Process payment jobs
  - Customer reimbursement (Cr. Customer → Db. Vendor for payment)
  - Complete accruals
  - Post FA depreciation
  - Generate financial statements
  - Set period to "Hold"

**Assessment: Thorough and well-sequenced**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Close checklist completeness | Excellent | Covers all major areas |
| Subledger reconciliation reports | Standard OOB | All mentioned reports are available |
| Currency revaluation sequence | Correct | AP/AR before GL revaluation |
| Customer reimbursement process | Standard OOB | Proper Cr. Customer/Db. Vendor linkage |
| Period hold at end | Correct | Prevents post-close entries |

**Recommendations:**

1. **Use the Financial Period Close workspace** to formalize this checklist. D365 provides a dedicated workspace where each close task can be:
   - Assigned to a specific person
   - Given a due date (relative to period end: +1 day, +3 days, etc.)
   - Tracked with dependencies (Task B cannot start until Task A completes)
   - Marked complete with timestamp and user
   This converts the informal checklist into an auditable, trackable process.

2. **The close sequence needs explicit ordering.** The FRD lists tasks but does not number them in execution order. Recommended sequence:
   1. Post all pending sub-ledger transactions (AP invoices, AR invoices, inventory movements)
   2. Run inventory close/recalculation
   3. Post fixed asset depreciation
   4. Complete accrual journals
   5. Run allocation journals
   6. Run foreign currency revaluation (AP → AR → Bank → GL)
   7. Run subledger-to-ledger reconciliation reports
   8. Post any adjustment journals
   9. Generate financial statements
   10. Set period to "Hold"

3. **Inventory close is missing from the checklist.** The FRD mentions "Potential conflicts - inventory and general ledger report" but does not explicitly list running inventory close/recalculation. For a manufacturing company like Technica, inventory close is critical for accurate COGS. **Add inventory close as a mandatory step before GL close.**

4. **"Accrued purchases excluding sales tax report"** for received-not-invoiced is essential. Ensure this is reconciled every month, as large GRNI balances are a common audit finding.

5. **Customer reimbursement** requires a linked vendor record per customer. Ensure this linkage is part of the master data migration, not an afterthought.

---

### GL008 - Year End Close

**What was proposed:**
- Pre-requisites: complete all module closes, reconcile, print reports, set periods on hold
- Year-end process: close fiscal year, transfer opening balances to new year
- Revalue foreign currency balances at latest rate (optional)
- Create new fiscal year, put previous year on hold

**Assessment: Standard and correct**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Pre-requisite checklist | Complete | All standard items covered |
| Opening balance transfer | Standard OOB | Fiscal year close function |
| Period hold (not close) strategy | Excellent | Allows post-audit adjustments |
| Foreign currency revaluation before close | Standard OOB | Important for year-end statements |

**Recommendations:**

1. **Year-end close creates opening balance entries on the retained earnings account.** Ensure the retained earnings account is defined in the GL parameters before the first year-end close. The FRD does not mention this specific setup.

2. **Multiple year-end close runs may be needed.** Auditors may request adjustments after the initial close. D365 supports re-running the year-end close, which reverses previous opening balances and recalculates. Document this capability for the accounting team.

3. **Dimension handling on year-end close is critical.** D365 allows configuring which dimensions are carried forward to the retained earnings account. Typically:
   - Balance Sheet accounts: carry forward all dimensions
   - P&L accounts: close to retained earnings with or without dimensions
   - **Recommendation:** Discuss with Technica whether they want P&L closing entries to retain Business Unit / Department dimensions on retained earnings (for comparative analysis) or close to a single retained earnings line.

---

### GL009 - Financial Statements / Reporting / Inquiries

**What was proposed:**
- Management Reporter (Financial Reporting) for all financial statements
- Income Statement, Balance Sheet, Cash Flow
- Users to be trained to design their own reports
- Trial balance by currency managed through Management Reporter (3 currencies: transaction, reporting, accounting)
- Financial statement mapping to be provided during migration

**Assessment: Standard approach, well-planned**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Management Reporter deployment | Standard OOB | Correct tool for financial statements |
| User self-service report design | Good | Reduces dependency on consultants |
| Three-currency trial balance | Good workaround | Uses Management Reporter column definitions |
| Deferred mapping to migration phase | Acceptable | But should start early |

**Recommendations:**

1. **Start financial statement design early** rather than deferring entirely to migration. The row definitions (account mappings) depend on the final COA, but column definitions (period comparisons, currency display, variance calculations) can be designed now. Common column sets:
   - Actual vs. Budget vs. Variance
   - Current Month vs. YTD vs. Prior Year
   - LBP Actual vs. USD Reporting vs. USD Transaction

2. **Management Reporter has known limitations** with very large account structures or high dimension cardinality. With Technica adding "Customers" as a financial dimension (potentially hundreds of values), report generation time may be impacted. **Test report performance during UAT with realistic data volumes.**

3. **Consider supplementing Management Reporter with Power BI** for operational financial analysis (drill-through to transactions, trend analysis, dashboard views). Management Reporter is best for formal financial statements; Power BI excels at interactive analysis.

4. **The trial balance currency display issue** (Technica wanted currency code in header) was resolved via Management Reporter. This is the correct solution. Document the specific column definition setup so it can be replicated for other reports.

---

### GL010 - Accounts for Automatic Transactions

**What was proposed:**
- Configure automatic transaction accounts for penny differences
- Other automatic posting types to be configured via "Create default types"

**Assessment: Standard, minimal risk**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Penny difference accounts | Standard OOB | Simple configuration |
| Default posting types | Standard OOB | One-time setup |

**Recommendations:**

1. **Review all automatic transaction posting types**, not just penny differences. Key ones for Technica:
   - Penny difference (rounding)
   - Currency exchange rate difference (realized/unrealized gain/loss)
   - Year-end result (retained earnings)
   - Allocation
   - Intercompany (due-to/due-from)
   Ensure all are mapped to appropriate GL accounts before go-live.

2. **Penny difference accounts should be monitored monthly.** Large balances in penny difference accounts indicate systematic rounding issues that may need investigation (e.g., penny difference of 0.100 may be too tight, generating excessive penny postings).

---

### GL011 - Foreign Currency Revaluation

**What was proposed:**
- Revalue GL accounts with foreign currency revaluation checkbox enabled
- Unrealized gain/loss posted as system-generated entries
- Separate entries for accounting currency and reporting currency
- Bank accounts excluded from GL revaluation (handled by Cash & Bank module to avoid duplication)
- Can run daily or monthly

**Assessment: Correctly designed with proper duplication prevention**

| Aspect | Rating | Comment |
|--------|--------|---------|
| GL-level revaluation | Standard OOB | Correct process |
| Dual entry (accounting + reporting) | Standard OOB | Required for dual-currency setup |
| Bank account exclusion from GL reval | Excellent | Prevents double revaluation |
| Frequency flexibility | Good | Monthly sufficient for most accounts |

**Recommendations:**

1. **Define the revaluation run sequence clearly:**
   - First: AP foreign currency revaluation (adjusts vendor balances)
   - Second: AR foreign currency revaluation (adjusts customer balances)
   - Third: Bank foreign currency revaluation (via Cash & Bank module)
   - Fourth: GL foreign currency revaluation (catches remaining balance sheet accounts)
   Running in wrong order can cause cascading adjustments.

2. **Revaluation generates reversal entries on the first day of the next period.** Ensure the accounting team understands that opening balances of a new period will show these reversals, and the net effect is zero (reversal + new revaluation at month-end).

3. **For the Lebanese multi-rate environment, verify which exchange rate type is used for revaluation.** The "Tech" custom rate type should be used, and it must be updated to reflect the actual market rate as of period end, not an average or official rate (unless policy dictates otherwise).

---

### GL012 - Cash Flow Based on Milestone

**What was proposed:**
- Project-level cash flow statement from Project Management module
- Shows all transaction types or filtered to milestones only
- Drill-down to transaction details available

**Assessment: Standard OOB, well-suited for project-based company**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Project cash flow report | Standard OOB | Built into Project module |
| Milestone filtering | Standard OOB | Parameterized report |
| Transaction drill-down | Standard OOB | Good for analysis |

**Recommendations:**

1. **This report is per-project.** For company-wide cash flow forecasting across all projects, Technica should also configure the **Cash Flow Forecasting** feature in D365 (General Ledger → Cash Flow Forecasting). This aggregates:
   - Customer payment predictions (AI-based)
   - Vendor payment schedules
   - Project milestones
   - Budget forecasts

2. **Link this to the broader cash management strategy.** With 38 bank accounts in 4 currencies, Technica needs visibility into which bank account will receive which milestone payment. The project cash flow report does not provide this bank-level detail. Consider a **Power BI cash flow dashboard** that combines project milestones with bank account balances.

---

### GL013 - Intercompany Transactions

**What was proposed:**
- OOB intercompany accounting with daily journals, vendor invoice journals, payment journals
- Balance sheet accounts (Due-to / Due-from) for intercompany postings
- Intercompany journal names configured per company
- Bidirectional posting relationships (both companies must have intercompany accounts for the other)
- Auto-posting in destination company when source company posts

**Assessment: Standard and correctly configured**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Due-to / Due-from accounts | Standard OOB | Proper balance sheet treatment |
| Bidirectional setup | Correct | Both sides configured |
| Journal auto-posting in destination | Standard OOB | Eliminates manual entry in second company |
| Multiple journal types supported | Good | Covers various transaction types |

**Recommendations:**

1. **Intercompany elimination is the counterpart to this process.** Ensure GL014 (Consolidation & Elimination) is configured to eliminate the Due-to / Due-from balances and any intercompany revenue/expense. The FRD covers both but they must be tested as an end-to-end flow.

2. **Intercompany reconciliation report should be run monthly.** D365 does not have a built-in intercompany reconciliation report. **Recommendation:** Build a Power BI report or use Management Reporter to compare Due-to balances in Company A against Due-from balances in Company B. Any mismatches indicate posting failures or timing differences.

3. **Workflow on intercompany journals is critical.** When Company A posts an intercompany journal, Company B gets an auto-generated journal. Ensure the auto-generated journal in Company B either:
   - Auto-posts immediately (faster, less control), or
   - Goes through approval workflow in Company B (slower, more control)
   The FRD does not specify this. For audit purposes, **auto-post with notification to Company B's accountant** is recommended.

4. **Intercompany transactions between Technica International SAL and Technica Offshore SAL** likely include transfer pricing scenarios (Offshore bills customers, International holds inventory). Ensure the intercompany pricing is consistent and documented for tax compliance.

---

### GL014 - Consolidation & Elimination Process

**What was proposed:**
- New consolidation company to be created (separate legal entity)
- Two methods: Consolidation Online + Financial Reporting consolidation via Management Reporter
- Elimination rules configured with net change or fixed method
- Consolidation includes: account mapping, financial dimension mapping, legal entity selection, currency translation
- Elimination journal with proposal option before posting
- Complete process: Consolidate Online → Process Elimination → Run Consolidation → Verify

**Assessment: Well-structured for first-time consolidation users**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Dedicated consolidation entity | Standard OOB | Correct approach |
| Online consolidation | Standard OOB | Simpler than import method |
| Elimination rules | Standard OOB | Properly configured |
| Financial dimension mapping | Standard OOB | Preserves dimensional reporting |
| Currency translation | Standard OOB | Critical for multi-currency entities |
| Proposal before posting | Good practice | Allows review of elimination entries |

**Recommendations:**

1. **If all entities share the same COA (same account numbers), consolidation is significantly simpler.** The FRD mentions that consolidation accounts are "unique per main account, not necessarily the same." If different entities use different COA numbers, the consolidation account mapping must be complete and accurate. **Recommendation:** Use the same COA across all entities to simplify consolidation. If this is not possible, use Consolidation Account Groups for mapping.

2. **Elimination rules need to cover all intercompany account pairs.** Common eliminations for Technica:
   - Due-to / Due-from (balance sheet)
   - Intercompany sales / Intercompany COGS (P&L)
   - Intercompany service fees (if any)
   - Unrealized profit in intercompany inventory transfers (if applicable between TI SAL and Offshore)

3. **The "rebuild balance during consolidation" is set to No** with a recommendation to run it as a separate batch. This is correct for performance but ensure the batch job is scheduled and monitored. Stale balances in the consolidation company will produce incorrect financial statements.

4. **Management Reporter consolidation** (mentioned as method #2) provides real-time consolidated reporting without needing to run the consolidation process. This is excellent for ad-hoc analysis between formal consolidation runs. **Recommendation:** Use Online Consolidation for formal period-end reporting and Management Reporter consolidation for interim analysis.

5. **Penny difference on elimination journals:** The FRD notes that out-of-balance elimination journals require increasing the penny difference threshold. For a multi-currency consolidation, this is common. **Define a specific penny difference threshold for the consolidation company** that accounts for translation rounding, separate from the operating companies' threshold.

---

### BA001 - Bank Management and Reconciliation

**What was proposed:**
- 38 bank accounts across multiple banks (Audi, BLOM, SGBL, Bemo, BLF, Bank of Beirut, Byblos, ABS, Banorient France)
- 4 bank groups: BankUSD, BankLBP, BankEUR, BankGBP
- Both manual and automatic (advanced) bank reconciliation
- MT940 format for automatic reconciliation (bank must provide)
- Matching rules for auto-reconciliation
- "Mark as new" feature for bank-originated transactions (fees, charges)
- Bank reconciliation approval workflow: Accounting Supervisor → Finance Manager → CFO
- Check collection and check payment processes
- Fixed check numbering (pre-printed checks)
- Check reversal with audit trail
- Bank/Letter of Guarantee functionality
- Petty cash accounts included

**Assessment: Comprehensive, well-documented bank management**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Bank account setup (38 accounts) | Complete | All accounts listed with numbers |
| Bank groups by currency | Good | Simplifies reporting and revaluation |
| Manual bank reconciliation | Standard OOB | For banks without MT940 |
| Advanced bank reconciliation (MT940) | Standard OOB | Preferred approach |
| Matching rules | Standard OOB | Key to automation |
| Mark as new for unrecorded items | Standard OOB | Handles bank charges, interest |
| Reconciliation approval workflow | Excellent | Three-level approval |
| Check management | Standard OOB | Fixed numbering with reversal |
| Letter of guarantee | Standard OOB | Project-based guarantee tracking |

**Recommendations:**

1. **38 bank accounts is a large number to reconcile.** Prioritize which banks will use MT940 (advanced reconciliation) and which will remain manual. **Recommendation:** Focus MT940 implementation on the highest-volume banks first (likely Audi, BLOM, SGBL, Bemo). The Banorient France accounts may already support MT940 natively (SEPA region).

2. **The bank account numbering scheme (51xxxxx) should be verified.** As noted in GL001, bank accounts in the 5-series is unusual. In D365, the main account on the bank account record defines where bank transactions post. Ensure these accounts:
   - Have the correct main account type (Balance Sheet - Asset for regular banks, or Liability for overdraft accounts like "BLOM USD O/D Account")
   - Are flagged for foreign currency revaluation (for non-LBP accounts)
   - Are NOT flagged for GL foreign currency revaluation (to avoid duplication, as noted in GL011)

3. **MT940 dependency on banks is a go-live risk.** The FRD correctly notes that "Technica team should collaborate with their banks to get MT940 format." Lebanese banks vary in their electronic banking capabilities. **Recommendation:** Start MT940 testing with banks immediately — do not wait for migration phase. If a bank cannot provide MT940, consider:
   - BAI2 format (some banks support this)
   - CSV import with custom format mapping (D365 supports configurable bank statement formats via Electronic Reporting)
   - Continue with manual reconciliation for that bank

4. **Matching rule design is critical for auto-reconciliation success.** The FRD shows matching based on: document number, reference number, bank transaction code, date, and amount. **Recommendation:** Design matching rules in tiers:
   - **Rule 1 (strict):** Match on amount + document number (exact match)
   - **Rule 2 (medium):** Match on amount + date range (+/- 3 days)
   - **Rule 3 (loose):** Match on amount only
   - Run rules in order; unmatched items remain for manual review

5. **"Mark as new" feature needs governance.** When a bank statement line is marked as "new," D365 creates an automatic journal entry using the bank transaction code → GL account mapping. This is powerful but risky if:
   - The GL account mapping is wrong (bank charge posted to wrong expense account)
   - The amount is material (should go through approval, not auto-post)
   - **Recommendation:** Set a threshold amount above which "Mark as new" items require manual journal entry instead of auto-posting.

6. **Reconciliation workflow (Supervisor → FM → CFO) for all 38 accounts is burdensome.** Consider tiered approval:
   - Petty cash and low-balance accounts: Supervisor only
   - Main operating accounts: Supervisor + Finance Manager
   - High-value accounts: Full three-level approval

7. **Check management:** Fixed (pre-printed) check numbering requires generating check numbers in advance. Ensure the starting check numbers are correctly entered during migration for each bank account that uses checks. The FRD does not mention **post-dated checks**, which are extremely common in Lebanon. **Verify whether Technica uses post-dated checks** — D365 supports them but requires specific configuration.

8. **Letter of Guarantee tracking** is linked to projects. Ensure the guarantee facility setup includes:
   - Guarantee type (performance, advance payment, bid bond)
   - Expiry date tracking with alerts
   - Commission/fee posting
   - Linkage to project (for cost allocation)

---

### BA002 - Bank Transfer Printout Process

**What was proposed:**
- Print bank transfer format from D365
- Technica to provide template for feasibility comparison with OOB report

**Assessment: Simple report customization**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Bank transfer printout | Standard OOB | May need format adjustment |
| Custom template support | Low effort | Standard SSRS report modification |

**Recommendations:**

1. **Use Electronic Reporting (ER) instead of SSRS for bank transfer formats.** ER configurations are easier to maintain, version, and modify without developer involvement. D365 provides ISO 20022 payment formats OOB, which cover most international bank transfer requirements.

2. **If Technica transfers between their own 38 accounts frequently**, ensure internal bank transfer journals are set up with the correct offset bank account and that the transfer is recorded as a single journal (not two separate entries).

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | SY001 | Evaluate compliance implications of "Technica ND" fictitious entity in D365 | Critical | Legal/Audit risk |
| 2 | SY003 | Automate exchange rate feeds now, not as future enhancement | High | Reduces daily manual effort and error risk |
| 3 | SY003 | Create explicit exchange rate type matrix (which rate for which transaction) | High | Prevents inconsistent rate application |
| 4 | SY003 | Increase penny difference threshold beyond 0.100 for LBP environment | Medium | Prevents posting failures |
| 5 | GL001 | Remove Customer as financial dimension on AR control accounts; use sub-ledger reporting instead | High | Reduces dimensional complexity |
| 6 | GL001 | Make Department and Business Unit mandatory on P&L accounts | Medium | Ensures complete cost center reporting |
| 7 | GL002 | Simplify to a single account structure with optional dimensions | Medium | Reduces maintenance and user confusion |
| 8 | GL003 | Enable basic journal controls from go-live (account type restrictions per journal) | Medium | Prevents mis-postings |
| 9 | GL003 | Restrict exchange rate override on journals with threshold-based workflow | Medium | Control improvement |
| 10 | GL006 | Thoroughly test AR-side VAT customization for credit notes and partial payments | Critical | Highest-risk customization |
| 11 | GL006 | Document Poland localization usage rationale for future support teams | Medium | Knowledge continuity |
| 12 | GL007 | Use Financial Period Close workspace for structured close checklist | Medium | Formalizes close process |
| 13 | GL007 | Add inventory close as mandatory step in month-end checklist | High | Manufacturing company requirement |
| 14 | GL008 | Define dimension handling on year-end close (carry forward vs. collapse) | Medium | Affects comparative reporting |
| 15 | GL013 | Build intercompany reconciliation report (Power BI or Management Reporter) | Medium | No OOB report available |
| 16 | GL014 | Use same COA across all entities to simplify consolidation | High | Reduces mapping complexity |
| 17 | BA001 | Prioritize MT940 for high-volume banks; start bank testing immediately | High | Go-live risk mitigation |
| 18 | BA001 | Design tiered matching rules for auto-reconciliation | Medium | Improves match rate |
| 19 | BA001 | Implement tiered reconciliation approval based on account materiality | Low | Reduces CFO approval burden |
| 20 | BA001 | Verify post-dated check requirement and configure if needed | Medium | Common in Lebanese market |

---

## Risk Flags

1. **GL006-002 (VAT dual-rate customization) is the highest-risk item.** It affects the core sales invoice posting routine for B2B USD invoices. Every edge case (credit notes, partial payments, prepayments, intercompany sales) must be tested. This customization touches the heart of the financial posting engine, and bugs will directly impact financial statements and tax reporting. **Allocate significant UAT time for this GAP.**

2. **Technica ND (fictitious entity for non-declared transactions) creates legal exposure.** D365 maintains immutable audit logs. Any transactions in this entity are permanently traceable. Technica's management should make an informed decision about whether this entity belongs in the auditable ERP system.

3. **Lebanese multi-rate currency environment is inherently fragile.** The system relies on correct, timely exchange rate entry for every transaction. One wrong rate on a high-value transaction creates cascading errors through revaluation, settlement, and reporting. The lack of automated rate feeds compounds this risk.

4. **38 bank accounts across 8+ banks with mixed reconciliation methods (manual + MT940) creates operational overhead.** If MT940 is not available from all banks by go-live, the reconciliation team faces a dual-process workload that may cause delays in month-end close.

5. **Cross-FRD dependencies are significant:**
   - GL006 VAT customization affects AR (FRD12-AR), Sales (FRD01), and Project Invoicing (FRD04)
   - GL013 Intercompany affects all entities across all modules
   - GL014 Consolidation depends on consistent COA and dimension setup across all entities
   - BA001 Bank accounts link to AP payment processing (FRD07-AP) and AR collection (FRD12-AR)
   - GL005 Allocation percentages may need data from Production (FRD05) for activity-based costing

6. **Financial dimensions design (Customer as dimension + optional-only policy + multiple account structures)** may result in inconsistent transaction tagging. This will surface as reporting gaps 3-6 months after go-live when management requests dimension-based analysis and finds incomplete data.

---

## Efficiency Verdict

The GL and Cash & Bank solution design is **fundamentally sound and heavily reliant on OOB D365 functionality**, which is the correct approach. With only 1 out of 47 requirements being a GAP, the fit rate is exceptional at **97.9%**. The solution covers all essential financial processes: COA, dimensions, journals, accruals, allocations, tax, period close, year-end close, financial reporting, foreign currency revaluation, intercompany, consolidation, and bank management.

**Where the design excels:**
- Comprehensive month-end and year-end close procedures
- Proper use of On-Hold (vs. Closed) period strategy
- Dual reconciliation approach (manual + MT940 advanced) based on bank capability
- Three-level bank reconciliation approval workflow
- Intercompany accounting with auto-posting to destination company
- Consolidation & elimination as a new capability properly designed
- Accrual schemes replacing manual Excel processes
- Allocation rules replacing manual Excel calculations

**Where the design needs strengthening:**
- **Currency management** is the weakest link. The multi-rate Lebanese environment requires more rigorous rate governance, automated feeds, and explicit rate-type-to-transaction-type mapping than what is documented.
- **Financial dimension strategy** needs refinement. The Customer-as-dimension decision should be reversed, optional dimensions should be reconsidered for P&L accounts, and account structures should be simplified.
- **The single GAP (VAT dual-rate) is high-impact and high-risk.** It requires careful design, extensive testing, and clear documentation. This alone could delay go-live if not managed proactively.
- **Bank management at scale** (38 accounts) needs operational planning beyond just the technical setup. MT940 availability, matching rule design, and approval workflow efficiency all need attention.

**Overall:** The solution is **efficient and appropriate** for Technica's requirements. The recommendations above are refinements and risk mitigations, not fundamental redesigns. The biggest value-adds would be: (1) automating exchange rates, (2) simplifying the dimension/structure design, and (3) investing heavily in testing the VAT customization.

---

*This review is based on D365 F&O General Ledger and Cash & Bank Management best practices as of 2026. Specific recommendations for the Lebanese multi-rate currency environment reflect known patterns from regional D365 implementations.*
