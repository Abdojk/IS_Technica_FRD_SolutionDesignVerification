# Training Manual - GL, Cash & Bank - Solution Design Verification Report

**Document:** Technica - User Training Manual - GL, C&B V2.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - General Ledger, Cash & Bank Management

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Section 7 - Foreign Currency Revaluation | Title reads "Foreign Currency Revolution" instead of "Revaluation" -- indicates possible copy-editing failure across the section; must verify all terminology | [Section 7 - Foreign Currency Revaluation](#7-foreign-currency-revaluation) |
| 2 | **CRITICAL** | FRD Coverage Gap - Tax (GL006) | VAT setup, dual-rate VAT (Poland localization for AP, AR-side customization), tax settlement -- the FRD's highest-risk item -- is completely absent from the training manual | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 3 | **CRITICAL** | FRD Coverage Gap - COA & Dimensions (GL001/GL002) | Chart of Accounts creation, financial dimension setup, account structures, and dimension entry rules are not covered -- users will not know how to maintain master data | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 4 | **CRITICAL** | FRD Coverage Gap - Financial Reporting (GL009) | Management Reporter / Financial Reporting is listed only as a report name in the reports table -- no training on designing or running financial statements | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 5 | **HIGH** | Section 2 - GL Journal | No mention of workflow approval, attachment handling, or financial dimension entry -- these are FRD-confirmed requirements (GL003) | [Section 2 - GL Journal](#2-gl-journal) |
| 6 | **HIGH** | Section 2 - GL Journal | No guidance on the three-currency display (transaction, accounting, reporting) or manual exchange rate override -- critical for Lebanese multi-rate environment | [Section 2 - GL Journal](#2-gl-journal) |
| 7 | **HIGH** | Section 5 - Month End Close | Inventory close is missing from the month-end checklist, consistent with the same gap identified in FRD12 -- critical for a manufacturing company | [Section 5 - Month End Close](#5-month-end-close) |
| 8 | **HIGH** | Section 5 - Month End Close | Foreign currency revaluation sequence (AP, AR, Bank, GL) is not documented -- users need to know the correct execution order | [Section 5 - Month End Close](#5-month-end-close) |
| 9 | **HIGH** | Section 6 - Year End Close | No mention of retained earnings account configuration, dimension carry-forward decisions, or the ability to re-run year-end close after audit adjustments | [Section 6 - Year End Close](#6-year-end-close) |
| 10 | **HIGH** | Section 8 - Intercompany | No training on intercompany setup (due-to/due-from account configuration, bidirectional relationship setup) -- users only see how to post a journal, not how to configure the framework | [Section 8 - Intercompany Transactions](#8-intercompany-transactions) |
| 11 | **HIGH** | Section 13 - Bank Reconciliation | Only manual reconciliation is covered -- Advanced Bank Reconciliation (MT940/electronic statement import, matching rules) is completely missing despite being a key FRD requirement (BA001) | [Section 13 - Bank Reconciliation](#13-bank-reconciliation) |
| 12 | **HIGH** | FRD Coverage Gap - Consolidation & Elimination (GL014) | Consolidation Online, elimination rules, and Management Reporter consolidation are not covered in the training manual despite being a new capability for Technica | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 13 | **MEDIUM** | Section 3 - GL Accruals | No mention of accrual reversal process when source invoices are cancelled or credited | [Section 3 - GL Accruals](#3-gl-accruals) |
| 14 | **MEDIUM** | Section 4 - Periodic & Allocation Journals | Allocation percentage governance and review process not documented; no guidance on who can modify allocation rules | [Section 4 - Periodic--allocation-journals](#4-periodic--allocation-journals) |
| 15 | **MEDIUM** | Section 7 - Foreign Currency Revaluation | No mention of the bank account exclusion from GL revaluation (to avoid double revaluation) -- this was a key design decision in the FRD | [Section 7 - Foreign Currency Revaluation](#7-foreign-currency-revaluation) |
| 16 | **MEDIUM** | Section 7 - Foreign Currency Revaluation | No guidance on which exchange rate type ("Tech" custom type) to use for revaluation or how to verify the rate before running the process | [Section 7 - Foreign Currency Revaluation](#7-foreign-currency-revaluation) |
| 17 | **MEDIUM** | Section 9 - Bank Account Creation | No reference to the 38 bank accounts or 4 bank groups (BankUSD, BankLBP, BankEUR, BankGBP) defined in the FRD; no guidance on financial dimension assignment on bank accounts | [Section 9 - Bank Account Creation](#9-bank-account-creation) |
| 18 | **MEDIUM** | Section 10 - Deposit Slip | Process is correct but lacks context on when to use deposit slips vs. other payment recording methods | [Section 10 - Deposit Slip](#10-deposit-slip) |
| 19 | **MEDIUM** | Section 11 - Checks | No mention of post-dated checks, which are extremely common in Lebanon and flagged in FRD review | [Section 11 - Checks](#11-checks) |
| 20 | **MEDIUM** | Section 11 - Checks | Check reversal process is not covered -- FRD explicitly mentions check reversal with audit trail | [Section 11 - Checks](#11-checks) |
| 21 | **MEDIUM** | Section 14 - Reports | Reports listed as a flat table with no training on how to run, parameterize, or interpret any of the reports | [Section 14 - Reports](#14-reports) |
| 22 | **MEDIUM** | FRD Coverage Gap - Cash Flow (GL012) | Project-level cash flow by milestone is not covered in training | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 23 | **MEDIUM** | FRD Coverage Gap - Automatic Transactions (GL010) | Configuration of accounts for automatic transactions (penny differences, exchange differences, year-end result) not covered | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 24 | **LOW** | Section 1 - Introduction | Acronym table is incomplete -- only 3 acronyms defined (System, C&B, GL); missing FA, AP, AR, COA, BPF, MT940, IBAN, SWIFT, VAT, etc. | [Section 1 - Introduction](#1-introduction) |
| 25 | **LOW** | Section 4 - Periodic Journals | Copy vs. Move retrieval option not explained with sufficient context on the consequences of each choice | [Section 4 - Periodic--allocation-journals](#4-periodic--allocation-journals) |
| 26 | **LOW** | Section 11 - Checks | Navigation path for creating checks references "Payment reversals > Checks" which is an unusual navigation path -- verify correctness | [Section 11 - Checks](#11-checks) |
| 27 | **LOW** | General | 86 screenshots are included but the manual lacks a Table of Contents and consistent step numbering across sections | [General Observations](#general-observations) |

**Totals:** 4 CRITICAL | 8 HIGH | 11 MEDIUM | 4 LOW

---

## Executive Summary

This Training Manual covers the user-facing processes for D365 F&O General Ledger and Cash & Bank Management modules. The document is structured across **14 sections** spanning: GL Journals, Accruals, Periodic & Allocation Journals, Month End Close, Year End Close, Foreign Currency Revaluation, Intercompany Transactions, Bank Account Creation, Deposit Slips, Checks, Bank Transfers, Bank Reconciliation, and Reports. It includes **86 screenshots** and **2 tables** (acronyms and reports listing).

**Overall assessment: The manual provides basic step-by-step navigation instructions but has significant gaps in coverage, contextual depth, and alignment with the FRD design decisions.**

The document reads as a generic D365 click-through guide rather than a Technica-specific training manual. It does not reference Technica's specific configuration choices (custom exchange rate type "Tech," specific journal names, bank account groupings, the Poland localization for VAT, the multi-rate currency environment, or the 38 bank accounts). Users following this manual will know how to click buttons but will not understand:

1. **Why** specific processes exist (business context)
2. **Which** specific values to select (Technica-specific configuration)
3. **What** to do when exceptions occur (error handling, edge cases)
4. **How** processes connect to each other (end-to-end flow awareness)

Most critically, several FRD processes that are new capabilities for Technica -- Tax/VAT handling (GL006), Chart of Accounts maintenance (GL001), Financial Reporting via Management Reporter (GL009), Consolidation & Elimination (GL014), and Advanced Bank Reconciliation with MT940 (BA001) -- are either completely absent or reduced to a single line in a table. These are precisely the areas where users need the most training.

The manual also propagates some gaps identified in the FRD review (e.g., missing inventory close in month-end, missing post-dated check coverage) rather than correcting them, suggesting the training material was developed without incorporating FRD review feedback.

---

## Process-by-Process Review

### 1. Introduction

**What is covered:**
- Purpose statement describing the manual scope (GL and Cash & Bank modules)
- Acronym table with 3 entries: System (Dynamics 365), C&B (Cash & Bank), GL (General Ledger)

**Assessment: Adequate purpose statement but incomplete acronyms**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Purpose clarity | Good | Clearly states the manual covers GL and C&B modules |
| Learning objectives | Generic | "Comprehensive understanding" is vague; should state specific competencies |
| Acronym table | **>> NEEDS REVIEW <<** | Only 3 acronyms; missing COA, FA, AP, AR, VAT, MT940, IBAN, SWIFT, LBP, USD, BPF, SSRS, ER |
| Prerequisites | Missing | No mention of required user security roles, prior training, or system access |

**>> ENHANCEMENT (LOW):** The acronym table should be expanded to include all abbreviations used throughout the document. Additionally, a prerequisites section should be added specifying: (a) required D365 security roles for each process, (b) assumed prior knowledge, and (c) reference to completed training on D365 navigation basics.

---

### 2. GL Journal

**What is covered:**
- Navigation: General ledger > Journal entries > General journals
- Steps: Create new journal, select journal name, open lines, enter date/company/account type/account/description/debit-credit
- Intercompany offset company field explanation
- Offset account type and account selection
- Currency selection
- Validate and Post

**Assessment: Functional but critically incomplete for Technica's requirements**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Basic journal creation steps | Correct | Standard OOB navigation and field entry |
| Intercompany offset explanation | Good | Correctly explains the offset company field purpose |
| Currency field mention | Present | But no multi-rate context |
| Workflow approval | **Missing** | FRD GL003 confirms workflow approval per journal name |
| Financial dimensions | **Missing** | Not mentioned at all; FRD GL001 confirms dimensions are required |
| Three-currency display | **Missing** | FRD GL003 explicitly covers three-currency visibility per line |
| Exchange rate override | **Missing** | FRD GL003 documents manual rate adjustment capability |
| Attachment feature | **Missing** | FRD GL003 lists attachment on journals as a requirement |
| Voucher handling (balanced/offset/separate) | **Missing** | FRD GL003 documents multiple voucher scenarios |
| Journal controls/restrictions | **Missing** | FRD GL003 discusses posting restrictions |

**>> ATTENTION AREA: GL Journal training omits critical Technica-specific functionality**

> The GL Journal section covers only the most basic click-path. It does not train users on: (1) how to enter financial dimensions on journal lines, (2) how to read and verify the three-currency amounts (transaction, accounting, reporting), (3) how to handle workflow approval when it is required, (4) when and how to override exchange rates, or (5) how to attach supporting documents. All of these are confirmed requirements in FRD GL003.

**Recommendations:**

1. **>> ENHANCEMENT (HIGH):** Add a sub-section on financial dimension entry during journal posting. Users must understand which dimensions are available (Department, Business Unit, Project, Customer), when each is relevant, and how default dimensions work. This is fundamental to all GL postings and directly impacts the quality of financial reporting.

2. **>> ENHANCEMENT (HIGH):** Add training on the three-currency display. For every journal line, users should understand:
   - Transaction currency (the currency of the original transaction)
   - Accounting currency (LBP -- Technica's base currency)
   - Reporting currency (USD -- Technica's reporting currency)
   - How the exchange rate is applied and where to verify it
   - When manual rate override is permitted (per Technica policy)

3. **>> ENHANCEMENT:** Add workflow approval steps. When a journal requires approval, the user should know: how to submit, how to check approval status, what happens if rejected, and how to resubmit.

4. **>> ENHANCEMENT:** Add guidance on journal name selection. The FRD confirms multiple journal names exist for different purposes (Daily, Adjustment, Accrual, Intercompany, etc.). Users should understand which journal name to use for which transaction type.

---

### 3. GL Accruals

**What is covered:**
- Section 3.1: Accrual scheme creation (setup of accrual identification, debit/credit accounts, voucher method, number sequence, calendar basis, frequency, occurrence count, posting timing, spread method)
- Section 3.2: Vendor invoice creation through AP Invoice Journal (as the source transaction for accruals)
- Section 3.3: Accrual amount distribution via GL journal using Functions > Ledger Accrual

**Assessment: Well-documented setup and execution, missing reversal/exception handling**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Accrual scheme setup steps | Excellent | Detailed field-by-field guidance with explanation |
| Vendor invoice as source | Correct | Proper linkage to AP invoice journal |
| Accrual distribution via GL journal | Correct | Functions > Ledger Accrual is the right path |
| Spread method explanation (Evenly vs. Number of days) | Good | Clear differentiation |
| Posting timing options (Beginning/Middle/End) | Good | Well-explained |
| Accrual reversal process | **Missing** | What happens when source invoice is cancelled? |
| Proposal-only option | **Missing** | FRD GL004 recommends proposal review before posting |
| Auto-posting of future accrual entries | **Partially covered** | Mentioned but not explicitly trained |

**>> ATTENTION AREA: Accrual reversal and exception handling not covered**

> The FRD review (GL004) specifically flagged that D365 does not automatically reverse accrual scheme entries when the source invoice is reversed. This manual should include guidance on how to manually reverse accruals when an invoice is cancelled or credited.

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Add a sub-section on accrual reversal. Document the manual process: (a) identify the accrual scheme entries that were auto-posted, (b) create a reversing journal entry for future periods' accruals, (c) verify the net effect is zero. Include this in the month-end close checklist training (Section 5).

2. **>> ENHANCEMENT:** Add guidance on using the "Proposal only" option during the first months of go-live, as recommended in the FRD review. This allows users to review accrual entries before they auto-post.

3. The step-by-step for accrual scheme creation is one of the better sections in the manual. Extend this level of detail to other sections.

---

### 4. Periodic & Allocation Journals

**What is covered:**
- Section 4.1: Periodic journal creation (name, description, journal lines with dates, accounts, recurring frequency, last date)
- Section 4.2: Periodic journal retrieval (create new GL journal, Period journal > Retrieve journal, To date, Copy vs. Move)
- Section 4.3: General process overview (brief explanation of the concept)
- Section 4.4: Allocation rule setup (name, description, effective dates, fixed percentage method, source account/dimensions, destination accounts/percentages)
- Section 4.5: Allocation journal process (Process allocation request, rule selection, date, proposal only option)

**Assessment: Good coverage of mechanics, weak on governance and context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Periodic journal creation | Good | Clear step-by-step with field explanations |
| Date handling (Empty date field option) | Good | Important nuance documented |
| Copy vs. Move retrieval | Present | But consequences not fully explained |
| Allocation rule setup | Good | Fixed percentage method correctly documented |
| Source and destination configuration | Good | Multi-destination with percentages explained |
| Proposal only option | Present | Mentioned in allocation process |
| Allocation percentage governance | **Missing** | Who approves changes to allocation percentages? |
| Allocation timing in close process | **Missing** | When should allocations run relative to other close tasks? |
| Cross-legal-entity allocation | **Missing** | FRD GL005 mentions shared legal entity cost allocation |

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Add governance guidance for allocation rules. Per the FRD review (GL005), allocation percentages should be reviewed quarterly with Finance Manager sign-off. The training manual should tell users: (a) who is authorized to create/modify allocation rules, (b) the approval process for changes, and (c) how to document the basis for allocation percentages.

2. **>> ENHANCEMENT (LOW):** Expand the Copy vs. Move explanation. "Copy" preserves the periodic journal for future retrieval; "Move" is a one-time use that removes the source. Users need to understand that "Move" is irreversible and should only be used for truly one-time transactions.

3. **>> ENHANCEMENT:** Add a note on allocation timing: allocations should run after all source transactions are posted for the period but before financial statements are generated. Reference the month-end close sequence in Section 5.

---

### 5. Month End Close

**What is covered:**
- Section 5.1: Financial Period Close Configuration (Closing Roles, Resources, Task Areas, Calendars, Templates with relative due dates, Closing Schedules)
- Section 5.2: Financial Period Close Workspace (Summary Tiles, Tasks and Status, Helpful Links and Updates)
- Section 5.3: Period Close Process (navigate to workspace, verify all tasks completed, change period status to "On hold," avoid "Permanently closed")

**Assessment: Excellent workspace configuration training, but missing operational close checklist content**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Financial Period Close Configuration | Excellent | Detailed setup of roles, resources, areas, calendars, templates, schedules |
| Task template with relative due dates | Excellent | Best practice for structured close |
| Workspace navigation and usage | Good | Three sections clearly explained |
| "On hold" vs. "Permanently closed" warning | Excellent | Critical advice correctly delivered |
| Locked schedule concept | Good | Explains how completed schedules are archived |
| Inventory close step | **Missing** | Not mentioned -- flagged as CRITICAL in FRD review |
| Sub-ledger reconciliation steps | **Missing** | FRD GL007 lists FA, AR, AP, Bank, Inventory reconciliation |
| Foreign currency revaluation sequence | **Missing** | FRD GL007 specifies AP > AR > Bank > GL revaluation order |
| Accrual completion step | **Missing** | No mention of completing accrual journals |
| FA depreciation posting step | **Missing** | FRD GL007 includes this in checklist |
| Financial statement generation step | **Missing** | Should be final step before period hold |
| Customer reimbursement process | **Missing** | FRD GL007 covers Cr. Customer > Db. Vendor |

**>> ATTENTION AREA: The manual teaches the workspace tool but not the actual close process**

> Section 5 excels at teaching users how to configure and navigate the Financial Period Close workspace. However, it does not document the actual month-end close checklist -- the specific tasks, their sequence, and their dependencies. The FRD (GL007) provides a comprehensive checklist that should be translated into training content.

**Recommendations:**

1. **>> ENHANCEMENT (HIGH):** Add a complete month-end close checklist with execution sequence. Based on FRD GL007 and the FRD review recommendations, the manual should include:
   1. Post all pending sub-ledger transactions (AP invoices, AR invoices, inventory movements)
   2. Run inventory close/recalculation (** missing from both FRD and this manual**)
   3. Post fixed asset depreciation
   4. Complete accrual journals
   5. Run allocation journals
   6. Run foreign currency revaluation (AP > AR > Bank > GL)
   7. Run sub-ledger-to-ledger reconciliation reports
   8. Post any adjustment journals
   9. Generate financial statements
   10. Set period to "On hold"

2. **>> ENHANCEMENT (HIGH):** Add the foreign currency revaluation sequence. The order matters: AP first, then AR, then Bank (via Cash & Bank module), then GL. Running in the wrong order causes cascading adjustments. This was a specific design decision documented in the FRD.

3. **>> ENHANCEMENT (HIGH):** Add inventory close as a mandatory step. The FRD review (GL007) flagged this as a HIGH priority item. For a manufacturing company like Technica, inventory close is essential for accurate COGS calculation. Without it, financial statements will be inaccurate.

---

### 6. Year End Close

**What is covered:**
- Section 6.1: Year-End Close Template Setup (group name, fiscal calendar, legal entity selection, transfer balance sheet dimensions toggle, P&L dimension handling -- Close All vs. Close Single)
- Section 6.2: Year-End Close Process (navigate to year-end close, run for template, select legal entities, enter voucher number with fiscal year, batch processing recommendation)

**Assessment: Correctly structured but missing critical configuration and re-run guidance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Template setup with legal entity grouping | Good | Correct process |
| Balance sheet dimension carry-forward | Documented | "Best practice to set to Yes" is correct advice |
| P&L Close All vs. Close Single | Documented | Correctly explains the options |
| Voucher naming with fiscal year | Good practice | Correctly recommended |
| Batch processing recommendation | Correct | Standard for large data volumes |
| Retained earnings account setup | **Missing** | Must be configured in GL parameters before first year-end close |
| Re-running year-end close | **Missing** | Auditors may require adjustments; D365 supports re-run |
| Pre-requisite checklist | **Missing** | FRD GL008 specifies: complete all module closes, reconcile, print reports, set periods on hold |
| Dimension carry-forward decision guidance | **Incomplete** | Options described but no Technica-specific recommendation |

**>> ATTENTION AREA: Retained earnings account is a prerequisite not mentioned in training**

> If the retained earnings account is not defined in GL parameters before the first year-end close, the process will fail. This is a one-time setup but must be trained.

**Recommendations:**

1. **>> ENHANCEMENT (HIGH):** Add a pre-requisites section before the year-end close process:
   - Verify retained earnings main account is defined in General Ledger Parameters
   - Complete all monthly closes for the fiscal year
   - Reconcile all sub-ledger modules to GL
   - Print and archive final financial statements
   - Set all 12 monthly periods to "On hold"

2. **>> ENHANCEMENT (HIGH):** Add guidance on re-running year-end close. D365 supports re-running the process, which reverses previous opening balances and recalculates. This is essential when auditors request post-close adjustments. Users should understand this capability and the correct procedure.

3. **>> ENHANCEMENT:** Provide Technica-specific guidance on the P&L dimension handling decision. Per the FRD review (GL008), Technica should decide whether P&L closing entries retain Business Unit/Department dimensions on retained earnings (for comparative analysis) or close to a single retained earnings line. This decision should be documented in the training manual as a finalized policy.

---

### 7. Foreign Currency Revaluation

**What is covered:**
- Explanation of the GL foreign currency revaluation purpose (adjust foreign currency balances for rate changes)
- Navigation: General ledger > Periodic Tasks > Foreign Currency Revaluation
- Parameters: From date, To date, Date of rate, Main accounts to revalue (All/Balance Sheet/P&L), Legal entity, Currencies
- Preview before posting option
- Exclude reporting currency adjustments option
- Filter for specific main accounts
- Batch processing option

**Assessment: Technically accurate but marred by a critical naming error and missing Technica-specific context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Section title | **>> CRITICAL ERROR <<** | "Foreign Currency Revolution" instead of "Revaluation" |
| Navigation path | Correct | Periodic Tasks > Foreign Currency Revaluation |
| Date parameter explanation | Good | Explains P&L vs. BS date handling difference |
| Main account filter (All/BS/P&L) | Correct | Standard parameter |
| Multi-legal-entity capability | Documented | Can run for multiple entities at once |
| Preview before posting | Documented | Good practice |
| Bank account exclusion guidance | **Missing** | FRD GL011 specifies bank accounts should be excluded from GL reval to avoid duplication |
| Exchange rate type selection | **Missing** | No mention of "Tech" custom rate type or which rate to use |
| Revaluation reversal entries | **Missing** | Users should understand auto-reversal on day 1 of next period |
| Revaluation run sequence | **Missing** | Must run after AP/AR/Bank revaluation |

**>> ATTENTION AREA: Section title contains a significant error**

> The heading reads **"7. Foreign Currency Revolution"** instead of **"Foreign Currency Revaluation."** While this may appear to be a simple typo, in a training manual, such errors erode user confidence in the document's accuracy. More importantly, users searching the document for "revaluation" will not find this section. This must be corrected and the entire section reviewed for additional terminology errors.

**Recommendations:**

1. **>> ENHANCEMENT (CRITICAL):** Correct the title from "Foreign Currency Revolution" to "Foreign Currency Revaluation." Review the entire section for any other terminology errors that may have resulted from a find-and-replace or autocorrect issue.

2. **>> ENHANCEMENT (MEDIUM):** Add guidance on bank account exclusion. The FRD (GL011) documents a specific design decision: bank accounts are excluded from GL foreign currency revaluation because they are handled by the Cash & Bank module separately. Users must understand that if bank accounts are included in GL revaluation, it will cause double-counting. Add a note or warning box explaining this.

3. **>> ENHANCEMENT (MEDIUM):** Add exchange rate type guidance. Technica uses a custom rate type called "Tech." Users need to know: (a) which rate type to select for revaluation, (b) how to verify the rate is current before running the process, and (c) who is responsible for updating the rate.

4. **>> ENHANCEMENT:** Explain the auto-reversal behavior. D365 automatically reverses revaluation entries on the first day of the next period. Users seeing these reversal entries for the first time may be confused or attempt to reverse them manually (creating a double reversal). Training should explain this is expected system behavior.

---

### 8. Intercompany Transactions

**What is covered:**
- Conceptual overview of intercompany transactions (facilitate financial activities between legal entities)
- Intercompany journal entry creation with example (C1 transferring funds to C2)
- Step-by-step: select company, offset company, account type (Bank), specify bank accounts, enter amount, validate, post
- System-generated entries explanation (originating: DR IC Receivable / CR Bank; destination: DR Bank / CR IC Payable)
- Key points: automatic posting, reconciliation, currency handling

**Assessment: Good conceptual example but limited to a single scenario with no setup training**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear purpose statement |
| Bank-to-bank transfer example | Good | Concrete, easy-to-follow scenario |
| System-generated entries | Excellent | Shows both sides of the intercompany entry |
| Automatic posting explanation | Good | Users understand the dual-company effect |
| Intercompany setup configuration | **Missing** | Due-to/Due-from account setup, bidirectional relationship configuration |
| Multiple transaction types | **Missing** | FRD GL013 covers daily journals, vendor invoice journals, payment journals |
| Intercompany reconciliation | **Mentioned** | But no actual steps or report references |
| Workflow on intercompany journals | **Missing** | FRD GL013 discusses auto-post vs. approval in destination company |
| Currency conversion details | **Mentioned** | "System handles currency conversion" is vague |
| Technica entity-specific guidance | **Missing** | No mention of TI SAL, Offshore SAL, or specific intercompany flows |

**>> ATTENTION AREA: Setup and configuration training is absent**

> The FRD (GL013) specifies that intercompany accounting requires: (a) balance sheet accounts for due-to/due-from, (b) bidirectional posting relationships configured in both companies, and (c) intercompany journal names per company. None of this setup is covered in the training. Users know how to create one type of intercompany journal entry, but not how the framework is configured or maintained.

**Recommendations:**

1. **>> ENHANCEMENT (HIGH):** Add intercompany setup training. Include:
   - How to configure due-to/due-from accounts per entity pair
   - How to set up the bidirectional intercompany relationship (both companies must reference each other)
   - How to configure intercompany journal names
   - Who is responsible for this configuration (typically Finance Manager or Chief Accountant)

2. **>> ENHANCEMENT:** Expand the example to cover multiple intercompany scenarios relevant to Technica:
   - Cash transfer between entities (currently covered)
   - Intercompany expense allocation (e.g., shared services from TI SAL to Offshore)
   - Intercompany sales/invoicing (Offshore invoices customers, TI SAL holds inventory)
   - Each scenario should show the accounting entries on both sides

3. **>> ENHANCEMENT:** Add an intercompany reconciliation procedure. The FRD review (GL013) recommends building a reconciliation report to compare due-to balances in one entity against due-from balances in another. Training should include how to run this report and what to do when mismatches are found.

---

### 9. Bank Account Creation

**What is covered:**
- Navigation: Cash and bank management > Bank accounts > Bank accounts
- Field-by-field creation: routing number type, bank account ID, routing number, bank account number, main account, name, bank group, currency, credit limit, SWIFT code, IBAN
- Address and Contact information FastTabs
- Financial Dimensions FastTab mention
- Save

**Assessment: Correct and reasonably complete for generic bank account setup**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard OOB |
| Core field coverage | Good | All essential fields documented |
| SWIFT and IBAN | Documented | Important for international payments |
| Credit limit field | Documented | Useful for overdraft accounts |
| Financial Dimensions FastTab | Mentioned | But no guidance on which dimensions to enter |
| Bank groups reference | Generic | "Select a group" -- no Technica-specific guidance |
| Main account linkage explanation | Good | Explains auto-posting behavior |
| Technica-specific bank groupings | **Missing** | FRD defines 4 groups: BankUSD, BankLBP, BankEUR, BankGBP |
| Foreign currency revaluation flag | **Missing** | Must be enabled for non-LBP bank accounts |
| Bank statement format configuration | **Missing** | MT940/electronic banking setup |
| Letter of guarantee setup | **Missing** | FRD BA001 covers this functionality |

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Add Technica-specific bank group reference. The FRD defines four bank groups by currency (BankUSD, BankLBP, BankEUR, BankGBP). The training should instruct users to select the appropriate group based on the bank account currency, and explain the significance (reporting, revaluation grouping).

2. **>> ENHANCEMENT (MEDIUM):** Add guidance on the financial dimensions FastTab. Users need to know which dimensions to assign to bank accounts and why. At minimum, the Business Unit dimension should be assigned to identify which entity/division uses the account.

3. **>> ENHANCEMENT:** Add a note about the foreign currency revaluation checkbox. For non-LBP bank accounts (USD, EUR, GBP), the revaluation flag must be enabled on the main account to ensure month-end currency adjustments are processed correctly.

---

### 10. Deposit Slip

**What is covered:**
- Navigation: General ledger > Journal entries > General journals
- Create payment lines with bank account as offset
- Payment tab: "Use a deposit slip" checkbox, bank transaction type for deposits, payment reference
- Validate and Post
- Functions > Deposit slip to print
- Date selection for printout
- Dialog for each bank account with print destination options
- Deposit slip number auto-populated after printing

**Assessment: Correct and complete for the deposit slip process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation and creation steps | Correct | Standard OOB process |
| "Use a deposit slip" checkbox | Correctly identified | Key step for grouping payments |
| Bank transaction type selection | Correct | Links to bank deposit type |
| Print functionality | Correct | Functions > Deposit slip |
| Deposit slip number tracking | Good | Explains auto-population |
| Context on when to use | **Missing** | No guidance on deposit slip vs. other payment recording |
| Cross-reference to GL Journal section | Present | References section 1.1 (should be section 2) |

**>> NEEDS REVIEW <<** The document references "section 1.1" for GL journal details, but GL Journal is Section 2. This internal cross-reference error should be corrected.

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Add contextual guidance on when deposit slips are used. Explain that deposit slips are used when multiple customer payments are deposited together at the bank, and the bank requires a deposit slip that matches the total deposit amount.

2. **>> ENHANCEMENT:** Correct the cross-reference from "section 1.1" to "Section 2 - GL Journal."

---

### 11. Checks

**What is covered:**
- Section 11.1: Check layout configuration (Free vs. Fixed check numbering, check form, prefix, paper length, check start position, slip copies, company logo, test printout)
- Section 11.2: Check payment via vendor payment journal (navigate to AP > Payments > Vendor payment journal, create journal, settle transactions, generate payments with check method, validate and post)

**Assessment: Good coverage of check creation and payment, missing reversal and post-dated checks**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Check layout configuration | Good | Detailed field-by-field instructions |
| Free vs. Fixed numbering explanation | Good | Clear differentiation |
| Check number creation (Fixed method) | Correct | Create checks before use |
| Vendor payment journal for check payment | Correct | Standard process |
| Settle transactions process | Good | Mark and settle invoices |
| Generate payments functionality | Correct | Method of payment = Check |
| Check reversal | **Missing** | FRD BA001 explicitly mentions check reversal with audit trail |
| Post-dated checks | **Missing** | FRD review flagged as MEDIUM priority; extremely common in Lebanon |
| Check number navigation path | **>> NEEDS REVIEW <<** | "Payment reversals > Checks" is unusual |
| Void check process | **Missing** | What happens when a check is damaged or needs cancellation? |

**>> ATTENTION AREA: Check reversal and post-dated checks are missing**

> The FRD (BA001) explicitly documents check reversal with audit trail, and the FRD review flags post-dated checks as a critical requirement for the Lebanese market. Neither is covered in this training manual.

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Add check reversal training. Include:
   - Navigation to check reversal
   - Reason codes for reversal
   - Impact on vendor transaction (re-opens the settled invoice)
   - Audit trail generated by the reversal
   - When to use reversal vs. void

2. **>> ENHANCEMENT (MEDIUM):** Add post-dated check handling if applicable to Technica. Post-dated checks require specific D365 configuration (bridging accounts, maturity date tracking, automatic posting on maturity). If Technica issues or receives post-dated checks (common in Lebanon), this is a significant training gap.

3. **>> ENHANCEMENT (LOW):** Verify the navigation path "Cash and bank management > Payment reversals > Checks" for creating check numbers. The standard path is typically within the bank account form or Cash and bank management > Bank accounts > Checks. Ensure the documented path is correct for Technica's D365 version.

---

### 12. Bank Transfer

**What is covered:**
- Navigation: Cash and bank management > Bank accounts > Bank account transfers OR General ledger > Journal entries > General journals
- Create journal, select bank journal name, open lines
- Select company, account type = Bank, specify source bank account
- Description, debit/credit amount
- Intercompany offset company (for cross-entity transfers)
- Offset account type = Bank, specify destination bank account
- Currency selection
- Validate and Post

**Assessment: Correct and concise**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Dual navigation paths | Good | Shows both entry points |
| Bank-to-bank transfer process | Correct | Standard OOB |
| Intercompany transfer mention | Good | Covers cross-entity scenario |
| Currency field | Present | Important for multi-currency transfers |
| Exchange rate handling on transfer | **Missing** | When transferring between different currency accounts, what rate applies? |
| Internal transfer journal recommendation | **Missing** | FRD BA002 mentions single journal for internal transfers |

**Recommendations:**

1. **>> ENHANCEMENT:** Add guidance on exchange rate handling for cross-currency bank transfers. When transferring from a USD bank account to an LBP bank account (or vice versa), users need to understand: (a) which exchange rate is applied, (b) whether they can override the rate, and (c) where the exchange rate gain/loss is posted. This is particularly important in Technica's multi-rate environment.

---

### 13. Bank Reconciliation

**What is covered:**
- Navigation: Cash and bank management > Bank accounts > Bank account
- Select bank > Reconcile > Account Reconciliation
- New reconciliation: enter statement date, reference, currency, ending balance
- Save, then Transactions button
- Mark accurate lines as cleared
- "Create New" for bank-originated transactions (define bank transaction type, amount, main account)
- Ensure Unreconciled is zero
- Reconcile account

**Assessment: Covers only manual reconciliation; Advanced Bank Reconciliation is completely absent**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Manual reconciliation process | Correct | Standard OOB steps |
| Statement header creation | Correct | Date, reference, currency, ending balance |
| Mark as cleared | Correct | Core reconciliation action |
| Create New for bank charges | Correct | Handles bank-originated items |
| Unreconciled = zero check | Good | Ensures completeness |
| Advanced Bank Reconciliation (MT940) | **Missing** | FRD BA001 specifies MT940 import for high-volume banks |
| Matching rules | **Missing** | FRD BA001 defines automatic matching rules |
| Electronic bank statement import | **Missing** | Core functionality for automation |
| Reconciliation approval workflow | **Missing** | FRD BA001 specifies 3-level approval: Supervisor > FM > CFO |
| Mark as new governance | **Missing** | FRD review recommends threshold for auto-posting |
| Bank reconciliation report/printout | **Missing** | Users need to verify and archive reconciliation results |

**>> ATTENTION AREA: Advanced Bank Reconciliation is the primary reconciliation method for high-volume banks**

> The FRD (BA001) specifies that Technica will use both manual and Advanced Bank Reconciliation (with MT940 electronic statements). The Advanced method is intended for the highest-volume banks (Audi, BLOM, SGBL, Bemo). This training manual only covers manual reconciliation, which is the fallback method for banks that cannot provide electronic statements.

**Recommendations:**

1. **>> ENHANCEMENT (HIGH):** Add a complete section on Advanced Bank Reconciliation. Include:
   - How to import electronic bank statements (MT940 format)
   - How matching rules work (auto-matching by amount, document number, date range)
   - How to review and approve auto-matched transactions
   - How to handle unmatched items (manual matching or "Mark as new")
   - How to run and complete the reconciliation
   - How to submit the reconciliation for workflow approval

2. **>> ENHANCEMENT (HIGH):** Add the reconciliation approval workflow. The FRD specifies a three-level approval (Accounting Supervisor > Finance Manager > CFO). Users need to know how to submit for approval, check approval status, and handle rejections.

3. **>> ENHANCEMENT:** Add guidance on the "Create New" / "Mark as new" feature governance. Per the FRD review, there should be a threshold amount above which new items require manual journal entry instead of auto-posting. Users should understand this policy.

---

### 14. Reports

**What is covered:**
- A table listing reports by module:
  - **General Ledger (21 reports):** Voucher transactions, Audit trail, Posted sales tax, Posted withholding tax, Chart of accounts, Management Reporter, Sales tax payments, Unposted journal entries, Posted transactions by journal, Print journal, Bank, Open ledger transactions, Transaction origin, Account statement by currency, Transaction list by date, Sales tax list, Sales tax transactions, Detailed trial balance, Summary trial balance, Withholding tax transactions
  - **Cash and Banks (17 reports):** Bank transactions, Balance control, Bank remittance, Letters of guarantee inquiry, Transactions, Bank statement, Payment summary by date, Deposit summary by date, Bank payment summary by vendor, Deposit summary by customer, Bank account balance, Payment advice, Deposit slip, Letters of credit/guarantee, Letter of guarantee expiration dates, Reconciliation, Unreconciled bank transactions, Print statement

**Assessment: Report catalog only, no training content**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Report listing completeness | Good | Comprehensive list for both modules |
| Report navigation paths | **Missing** | No menu paths provided |
| Report parameters and filters | **Missing** | No guidance on how to run reports |
| Report output interpretation | **Missing** | No explanation of what each report shows |
| Management Reporter training | **Missing** | Listed as a report name, no design/build/run training |
| Audit trail duplicate | **>> NEEDS REVIEW <<** | "Audit trail" appears twice in GL reports list |

**>> ATTENTION AREA: Reports section is a catalog, not training material**

> Listing 38 report names in a table does not constitute training. Users need to know: (a) where to find each report, (b) what parameters to set, (c) what the output means, and (d) when to use each report. At minimum, the key reports used in daily operations and month-end close should have step-by-step training.

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** For the top 5-10 most-used reports, add sub-sections with:
   - Navigation path
   - Key parameters and filter options
   - Sample output explanation
   - When to use (daily operations, month-end, audit, etc.)

   Priority reports for training:
   - Trial Balance (Detailed and Summary)
   - Voucher Transactions
   - Bank Account Balance
   - Bank Reconciliation Report
   - Transaction List by Date

2. **>> ENHANCEMENT (MEDIUM):** Add a dedicated Management Reporter section. The FRD (GL009) confirms that all financial statements (Income Statement, Balance Sheet, Cash Flow) will be built in Management Reporter. Users are supposed to be trained to design their own reports. This is a significant training gap.

3. **>> ENHANCEMENT:** Remove the duplicate "Audit trail" entry from the GL reports list.

---

## General Observations

**Document Structure and Quality:**

| Aspect | Assessment |
|--------|------------|
| Overall structure | 14 sections covering major GL and C&B processes |
| Screenshots | 86 screenshots included -- good visual support |
| Table of Contents | **Missing** -- users cannot quickly navigate to relevant sections |
| Step numbering | **Inconsistent** -- some sections use numbered steps, others use bullets, some use no numbering |
| Cross-references | **Inaccurate** -- Section 10 references "section 1.1" which does not exist (should be Section 2) |
| Technica-specific content | **Minimal** -- reads as a generic D365 manual, not Technica-specific |
| Error handling / troubleshooting | **Missing** -- no guidance on what to do when errors occur |
| Security role guidance | **Missing** -- no mention of which user roles can perform each process |
| Terminology accuracy | **Flawed** -- "Foreign Currency Revolution" in Section 7 title |

**>> ENHANCEMENT (LOW):** Add a Table of Contents at the beginning of the document. With 14 main sections and multiple sub-sections, users need a navigation aid. D365 training manuals are reference documents used during daily work, not read cover-to-cover.

**>> ENHANCEMENT:** Standardize step numbering across all sections. Use numbered steps (Step 1, Step 2, etc.) consistently. Numbered steps allow trainers and support staff to reference specific steps ("Go back to Step 5 in Section 2").

---

## Cross-Reference with FRD

The following table maps FRD processes to training manual coverage, identifying gaps:

| FRD Process | FRD Ref | Training Manual Section | Coverage | Gap Assessment |
|------------|---------|------------------------|----------|----------------|
| Legal Entities | SY001 | Not covered | **NONE** | LOW -- setup/configuration, not daily user process |
| Fiscal Calendar | SY002 | Section 5 (partial) | **PARTIAL** | MEDIUM -- period status changes covered; calendar setup/maintenance not covered |
| Currencies & Exchange Rates | SY003 | Section 7 (partial) | **PARTIAL** | HIGH -- revaluation covered; daily rate entry, rate type matrix, automated feeds not covered |
| Chart of Accounts | GL001 | Not covered | **NONE** | **CRITICAL** -- COA maintenance is a routine process; users need to create/modify/suspend accounts |
| Account Structures | GL002 | Not covered | **NONE** | **CRITICAL** -- impacts every posting; users need to understand structure/dimension rules |
| GL Journals | GL003 | Section 2 | **PARTIAL** | HIGH -- basic posting covered; workflow, dimensions, multi-currency, controls missing |
| Accruals | GL004 | Section 3 | **GOOD** | LOW -- well-covered; add reversal handling |
| Allocations | GL005 | Section 4 | **GOOD** | LOW -- well-covered; add governance |
| Tax / VAT | GL006 | Not covered | **NONE** | **CRITICAL** -- highest-risk FRD item (dual-rate VAT customization) has zero training coverage |
| Month End Close | GL007 | Section 5 | **PARTIAL** | HIGH -- workspace trained; actual close checklist missing |
| Year End Close | GL008 | Section 6 | **GOOD** | MEDIUM -- process covered; prerequisites and re-run capability missing |
| Financial Reporting (MR) | GL009 | Section 14 (name only) | **NONE** | **CRITICAL** -- Management Reporter is listed but has no training content |
| Automatic Transactions | GL010 | Not covered | **NONE** | MEDIUM -- configuration process, not covered |
| Foreign Currency Reval | GL011 | Section 7 | **GOOD** | MEDIUM -- process covered; bank exclusion and rate type guidance missing |
| Cash Flow by Milestone | GL012 | Not covered | **NONE** | MEDIUM -- project-level reporting not covered |
| Intercompany | GL013 | Section 8 | **PARTIAL** | HIGH -- one scenario covered; setup, reconciliation, multi-scenario missing |
| Consolidation & Elimination | GL014 | Not covered | **NONE** | **HIGH** -- new capability for Technica; no training at all |
| Bank Management | BA001 | Sections 9, 12, 13 | **PARTIAL** | HIGH -- basic operations covered; Advanced Recon (MT940), matching rules, approval workflow missing |
| Bank Transfer Printout | BA002 | Section 12 | **PARTIAL** | LOW -- transfer process covered; printout format not addressed |

**Summary of FRD-to-Training Alignment:**
- **GOOD coverage (4 processes):** GL004 Accruals, GL005 Allocations, GL008 Year End Close, GL011 Foreign Currency Revaluation
- **PARTIAL coverage (6 processes):** SY002, SY003, GL003, GL007, GL013, BA001
- **NO coverage (7 processes):** GL001, GL002, GL006, GL009, GL010, GL012, GL014
- **Not expected in training (1 process):** SY001

**>> NEEDS REVIEW <<** The training manual covers only 10 of 17 FRD processes, and only 4 of those have good coverage. Seven FRD processes have zero training content, including the highest-risk item (GL006 - Tax/VAT).

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | Section 7 Title | Correct "Foreign Currency Revolution" to "Foreign Currency Revaluation" and review entire section for terminology errors | Critical | Document credibility; user confusion |
| 2 | GL006 - Tax/VAT | Create complete training section for VAT setup, dual-rate VAT handling (Poland localization AP-side, AR-side customization), tax settlement | Critical | Highest-risk FRD item has zero training |
| 3 | GL001/GL002 - COA & Structures | Create training sections for Chart of Accounts maintenance, financial dimension setup, and account structure rules | Critical | Foundation of all GL postings |
| 4 | GL009 - Financial Reporting | Create dedicated Management Reporter training: design row/column definitions, run reports, export, schedule | Critical | Primary financial statement delivery tool |
| 5 | Section 2 - GL Journal | Add financial dimension entry, three-currency display, workflow approval, exchange rate handling, attachments | High | Most-used daily process |
| 6 | Section 5 - Month End Close | Add complete close checklist with sequenced steps including inventory close and revaluation order | High | Operational accuracy of financial statements |
| 7 | Section 6 - Year End Close | Add retained earnings prerequisite, re-run guidance, dimension carry-forward decision | High | Annual critical process |
| 8 | Section 8 - Intercompany | Add intercompany setup, multiple scenarios, reconciliation procedure | High | Multi-entity operations |
| 9 | Section 13 - Bank Reconciliation | Add Advanced Bank Reconciliation (MT940 import, matching rules, workflow approval) | High | Primary reconciliation method for high-volume banks |
| 10 | GL014 - Consolidation | Create training section for Consolidation Online, elimination rules, Management Reporter consolidation | High | New capability with no training content |
| 11 | Section 3 - Accruals | Add accrual reversal process and proposal-only option training | Medium | Exception handling |
| 12 | Section 4 - Allocations | Add governance guidance (who approves percentage changes, quarterly review) | Medium | Internal control |
| 13 | Section 7 - Reval | Add bank exclusion guidance, exchange rate type selection, reversal entry explanation | Medium | Avoid double revaluation; rate accuracy |
| 14 | Section 9 - Bank Account | Add Technica-specific bank groups, financial dimension guidance, revaluation flag | Medium | Correct bank account configuration |
| 15 | Section 11 - Checks | Add check reversal, post-dated check handling, void check process | Medium | Complete check lifecycle |
| 16 | Section 14 - Reports | Add step-by-step training for top 10 reports with navigation, parameters, and interpretation | Medium | Reports are useless without run instructions |
| 17 | GL010/GL012 | Add Automatic Transaction accounts setup and Cash Flow by Milestone training | Medium | Complete process coverage |
| 18 | General - Document | Add Table of Contents, standardize step numbering, fix cross-references | Low | Usability and navigation |
| 19 | Section 1 - Introduction | Expand acronym table, add prerequisites and security role requirements | Low | Completeness |
| 20 | Section 4 - Periodic Journals | Expand Copy vs. Move explanation with consequences | Low | User understanding |

---

## Risk Flags

1. **GL006 Tax/VAT training gap is the single highest risk.** The FRD identifies the dual-rate VAT customization (Poland localization for AP, AR-side custom code) as the most complex and highest-risk item in the entire GL/C&B scope. It affects every B2B USD sales invoice. Yet the training manual contains absolutely no content on tax setup, VAT calculation, tax groups, or the dual-rate handling. If users go live without understanding this process, they will not be able to correctly create sales invoices with VAT, file VAT returns, or handle credit notes involving the dual-rate split. **This gap must be closed before go-live.**

2. **Financial Reporting (Management Reporter) training absence undermines the entire implementation value.** The purpose of implementing D365 is better financial management and reporting. The FRD (GL009) states users will be trained to design their own reports. The training manual lists "Management Reporter" as a single line in a table. Without proper MR training, Technica will remain dependent on consultants for every financial statement change, negating a key benefit of the system.

3. **Advanced Bank Reconciliation training gap threatens month-end close timeline.** With 38 bank accounts, manual reconciliation for all of them is not sustainable. The FRD designed MT940-based Advanced Reconciliation for high-volume banks precisely to address this. If users are only trained on manual reconciliation, the monthly close process will be significantly slower than designed, and the investment in MT940 integration will be wasted.

4. **Chart of Accounts and Financial Dimension training gap will cause data quality issues.** Every GL posting requires correct account and dimension selection. Without training on the COA structure (7-digit accounts, account types, account categories) and the dimension framework (Department, Business Unit, Project, Customer), users will post to wrong accounts and omit dimensions. These errors are costly to correct after the fact and will render financial reports unreliable.

5. **Consolidation & Elimination training absence is a go-live risk for group reporting.** The FRD (GL014) documents consolidation as a new capability that Technica is implementing for the first time. Users have no prior experience with this process. The absence of any consolidation training means the finance team will be unable to produce consolidated financial statements without consultant support.

6. **The "Foreign Currency Revolution" title error raises a broader quality concern.** If the most technically precise section of the manual contains a title error this significant, it suggests the document may not have undergone technical review. Other sections should be carefully verified for accuracy of navigation paths, field names, and process descriptions against the actual D365 environment.

7. **Cross-FRD impact: Training gaps compound across modules.** The GL/C&B training manual gaps do not exist in isolation:
   - Tax training gap affects AR invoicing (covered in a separate FRD)
   - Intercompany training gap affects AP/AR processing across all entities
   - Bank reconciliation training gap affects AP payment processing and AR collection tracking
   - Financial dimension training gap affects every module that posts to GL

   If other training manuals have similar gap patterns, the cumulative training deficit at go-live could be substantial.

8. **Manual reads as generic D365 documentation, not Technica-specific training.** The document does not reference any Technica-specific configurations, journal names, account numbers, bank groups, exchange rate types, or business policies. Users will be unable to bridge the gap between generic D365 instructions and their actual work environment without additional verbal training or supplementary materials. This creates a risk that the manual will not be used as a reference after go-live, reducing its value to zero.

---

## Appendix: Document Statistics

| Metric | Value |
|--------|-------|
| Total sections | 14 (plus introduction) |
| Total sub-sections | 17 |
| Total screenshots | 86 |
| Total tables | 2 (acronyms + reports) |
| Total paragraphs with content | 397 |
| FRD processes covered | 10 of 17 (59%) |
| FRD processes with GOOD coverage | 4 of 17 (24%) |
| FRD processes with NO coverage | 7 of 17 (41%) |
| Enhancement items identified | 27 total |
| Critical items | 4 |
| High items | 8 |
| Medium items | 11 |
| Low items | 4 |

---

*This review is based on D365 F&O General Ledger and Cash & Bank Management best practices as of 2026, cross-referenced against FRD12-GL-CashBank-V3.0 and its Solution Design Verification Report. Specific recommendations for the Lebanese multi-rate currency environment reflect known patterns from regional D365 implementations. The training manual should be revised to address the identified gaps before user acceptance testing (UAT) begins, as users who are not properly trained cannot effectively validate the system during UAT.*
