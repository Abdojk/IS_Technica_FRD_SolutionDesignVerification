# Training Manual - Accounts Receivable - Solution Design Verification Report

**Document:** Technica - User Training Manual - Accounts Receivable V2.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Accounts Receivable

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Project Advance Payments | No training coverage of prepayment posting profile setup -- users may post advances directly to AR instead of a liability account, misstating financial statements | [Project Advance Payments](#project-advance-payments) |
| 2 | **CRITICAL** | Cross-FRD / Project Integration | FRD identified project milestone invoicing as the primary revenue channel, but training manual lacks end-to-end milestone billing lifecycle instructions | [Cross-Reference with FRD: Project Milestone Invoicing](#cross-reference-with-frd) |
| 3 | **CRITICAL** | Black Market Rate Invoicing | FRD's highest-complexity GAP (AR003-003 dual exchange rate invoicing) is completely absent from training -- users have no guidance on Lebanon-specific dual-rate VAT scenario | [Cross-Reference with FRD: Black Market Rate Invoicing](#cross-reference-with-frd) |
| 4 | **HIGH** | Customer Creation | Customer creation workflow (GAP 001-004) from FRD is not reflected -- training shows direct creation with no approval routing | [Customer Creation](#customer-creation) |
| 5 | **HIGH** | Customer Creation | No training on Cross-Company Data Sharing for multi-entity customer records despite FRD requirement | [Customer Creation](#customer-creation) |
| 6 | **HIGH** | Customer Payments | No mention of Advanced Bank Reconciliation for automated payment matching -- a critical FRD recommendation | [Customer Payments](#customer-payments) |
| 7 | **HIGH** | Customer Payments | Payment journal workflow approval referenced in FRD is absent from training steps | [Customer Payments](#customer-payments) |
| 8 | **HIGH** | Configuration - Posting Profile | Training teaches single posting profile setup but FRD flagged need for separate intercompany and prepayment posting profiles | [Customer Posting Profile](#accounts-receivable-configuration) |
| 9 | **HIGH** | Free Text Invoicing | No training on FTI approval workflow -- users are instructed to post directly without any approval gate | [Free Text Invoicing](#free-text-invoicing) |
| 10 | **HIGH** | Intercompany Trade | Training covers only setup and order viewing -- missing intercompany invoice posting, payment settlement, and elimination procedures | [Intercompany Trade Operations](#intercompany-trade-operations) |
| 11 | **MEDIUM** | Configuration - Terms of Payment | Training only shows Net and COD methods; FRD requires advance payment terms, milestone-based terms, and letter of credit terms for foreign customers | [Terms of Payment](#accounts-receivable-configuration) |
| 12 | **MEDIUM** | Configuration - Method of Payment | Training covers only generic setup; no guidance on electronic payment methods (SEPA, wire templates) flagged in FRD | [Method of Payment](#accounts-receivable-configuration) |
| 13 | **MEDIUM** | Configuration - Customer Group | Training is generic; does not specify the actual customer groups Technica should configure (FRD recommended expanding beyond Local/Foreign) | [Customer Group](#accounts-receivable-configuration) |
| 14 | **MEDIUM** | Customer Creation | No training on trade agreement setup for customer-specific or group-specific pricing despite detailed FRD coverage | [Customer Creation](#customer-creation) |
| 15 | **MEDIUM** | Customer Creation | Credit hold / CRM integration (GAP 001-013) not mentioned -- users not trained on how credit hold status syncs with CRM | [Customer Creation](#customer-creation) |
| 16 | **MEDIUM** | Foreign Currency Revaluation | No guidance on exchange rate import/maintenance -- training assumes rates already exist but FRD flagged manual entry as error-prone | [Foreign Currency Revaluation](#foreign-currency-revaluation) |
| 17 | **MEDIUM** | Reports | No training on how to filter or display Project ID on AR reports -- FRD flagged this as a 4-GAP requirement (AR006-001 through AR006-004) | [Accounts Receivable Reports](#accounts-receivable-reports) |
| 18 | **MEDIUM** | Reports | Customer Invoice Transaction Report from FRD is missing from training manual | [Accounts Receivable Reports](#accounts-receivable-reports) |
| 19 | **LOW** | Settlement | Undo settlement training lacks warnings about period-close restrictions and audit implications | [Settlement and Undo Settlement](#settlement-and-undo-settlement-for-customer-transactions) |
| 20 | **LOW** | Reverse Transactions | Missing navigation path -- training jumps directly to "Click on Transactions" without specifying starting point | [Reverse Customer Transactions](#reverse-customers-transactions) |
| 21 | **LOW** | Reports | Cash application / unapplied advance payments report recommended in FRD is absent from training | [Accounts Receivable Reports](#accounts-receivable-reports) |
| 22 | **LOW** | Document Quality | Multiple typographical errors, inconsistent formatting, and spacing issues throughout the manual | [Document Quality Observations](#document-quality-observations) |

**Totals:** 3 CRITICAL | 7 HIGH | 8 MEDIUM | 4 LOW

---

## Executive Summary

This Training Manual covers the Accounts Receivable module in D365 F&O for Technica, organized into four main sections: Configuration (Terms of Payment, Method of Payment, Customer Group, Customer Posting Profile), Processes (Customer Creation, Project Advance Payments, Packing Slip Invoicing, Free Text Invoicing, Customer Payments, Settlement/Undo Settlement, Transaction Reversal, Foreign Currency Revaluation, Intercompany Trade), and Reports (Customer Account Statement, Customer Aging, Aged Balances, Customer to Ledger Reconciliation). The document includes 51 screenshots providing visual guidance for users.

**Overall Assessment: The training manual provides a functional walkthrough of basic AR operations but has significant gaps when compared to the FRD design.** The manual covers standard D365 transactional steps adequately -- a user could follow the instructions to create a customer, post a free text invoice, or process a payment. However, the manual fails to address several critical design decisions and GAPs identified in FRD12, most notably:

1. **The project milestone invoicing integration** -- identified in the FRD as the primary revenue channel for Technica's make-to-order business -- receives only partial treatment through the "Project Advance Payments" and "Invoicing a Packing Slip" sections, but lacks the complete lifecycle from milestone completion through collection.

2. **The black market rate / dual exchange rate invoicing scenario** (GAP AR003-003) -- the single highest-complexity GAP in the entire FRD -- is completely absent from the training manual.

3. **Multiple FRD-recommended configurations and processes** (Advanced Bank Reconciliation, electronic payment methods, customer creation workflow, credit hold CRM sync, prepayment posting profile) are not taught.

The training manual reads as a generic D365 AR guide with Technica-specific project advance payment procedures overlaid. It needs to be elevated to a Technica-specific training resource that reflects the actual configured solution, including all GAPs and customizations.

**Net assessment: The manual covers approximately 55-60% of what users need to know to operate the Technica AR solution as designed in the FRD. The remaining 40% -- particularly around project integration, advance payment accounting, dual-rate invoicing, and intercompany operations -- must be added before the manual is fit for user training.**

---

## Process-by-Process Review

### Accounts Receivable Configuration

#### Terms of Payment

**What the training covers:**
- Navigation path: Accounts receivable > Payment setup > Terms of payment
- Creating a new term of payment with code and description
- Setup FastTab with payment method selection
- Net days and COD options

**Assessment: Basic but incomplete for Technica's needs**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Creation steps | Correct | Standard OOB procedure |
| Net/COD methods | Correct | Two basic methods shown |
| Advance payment terms | **>> NEEDS REVIEW <<** | Not covered -- critical for project advance payments |
| Milestone-based terms | **>> NEEDS REVIEW <<** | Not covered -- needed for project billing |
| Letter of credit terms | **>> NEEDS REVIEW <<** | Not covered -- needed for foreign customers |

**>> ATTENTION AREA: Training only shows two payment calculation methods (Net and COD)**

> The FRD identified that Technica's payment terms are still pending definition but recommended advance payment terms, milestone-based terms, Net 30/60/90, and letter of credit terms for foreign customers. The training manual must include examples of these Technica-specific terms once they are finalized, not just the generic creation procedure.

**>> ENHANCEMENT:** Add specific examples of Technica payment terms including: (1) advance payment terms for project deposits, (2) milestone-based terms aligned to FRD04 project milestones, (3) standard Net terms (30/60/90), and (4) LC terms for foreign customers. Each should include the calculation method, days, and typical use case.

---

#### Method of Payment

**What the training covers:**
- Navigation path: Accounts receivable > Payment setup > Method of payment
- Creating a new method with name and description
- General FastTab: Account type (Ledger or Bank) and payment account selection

**Assessment: Generic procedure -- lacks Technica-specific configuration**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Creation steps | Correct | Basic procedure |
| Account type selection | Correct | Ledger or Bank offset |
| Electronic payment formats | **>> NEEDS REVIEW <<** | Not mentioned at all |
| Payment file export setup | **>> NEEDS REVIEW <<** | Not covered |

**>> ENHANCEMENT:** The FRD flagged that electronic payment methods are undefined for foreign customers. The training should include guidance on configuring SEPA credit transfer, wire transfer templates, and letter of credit payment methods. For a manufacturer with international customers, generic "Cash and Bank Transfer" methods are insufficient.

---

#### Customer Group

**What the training covers:**
- Navigation path: Accounts receivable > Setup > Customer group
- Creating a new group with short name and description

**Assessment: Procedure is correct but lacks Technica-specific content**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Creation steps | Correct | Basic procedure |
| Specific groups to create | **>> NEEDS REVIEW <<** | No Technica-specific groups listed |
| Purpose of each group | Not covered | No explanation of how groups drive posting, pricing, or reporting |

**>> ENHANCEMENT:** The FRD defined Local and Foreign customer groups, and the FRD review recommended expanding to include Distributor-Local, Distributor-Foreign, End Customer-Local, End Customer-Foreign, and Intercompany. The training manual should list the exact customer groups to be configured at Technica with explanations of their purpose and how they affect posting profiles, trade agreements, and reporting.

---

#### Customer Posting Profile

**What the training covers:**
- Navigation path: Accounts receivable > Setup > Customer posting profile
- Selecting a posting profile
- Adding setup records by Group
- Assigning summary (control) accounts per customer group

**Assessment: Conceptually correct but critical profiles missing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Group-level setup | Correct | Appropriate granularity |
| Summary account concept | Correct | Good explanation of control accounts |
| Prepayment posting profile | **>> NEEDS REVIEW <<** | Not mentioned -- critical for advance payments |
| Intercompany posting profile | **>> NEEDS REVIEW <<** | Not mentioned -- FRD flagged risk of single profile |
| Shipment posting profile | Partially referenced | Referenced in "Invoicing a packing slip" section but not configured here |

**>> ATTENTION AREA: Training does not cover prepayment or shipment posting profiles**

> The "Project Advance Payments" section later references a "Prepayment" posting profile, and the "Invoicing a Packing Slip" section references a "Shipment" posting profile. However, neither profile is taught in the configuration section. Users are told to select these profiles during transaction processing without understanding how to set them up or what ledger accounts they map to.

**>> ENHANCEMENT:** Add explicit configuration steps for all Technica posting profiles:
- **General / ALL** -- standard AR control account for regular invoices
- **Prepayment** -- customer advance liability account (not AR) for advance payments
- **Shipment** -- for packing slip invoicing scenarios
- **Intercompany** -- separate control account for intercompany receivables (per FRD recommendation)

This is a **CRITICAL** training gap because incorrect posting profile selection directly causes financial misstatement.

---

### Customer Creation

**What the training covers:**
- Navigation path: Sales and marketing > Customers > All customers > New
- Record type selection (Person/Organization)
- Name, customer group, currency, terms of payment, sales tax group, country
- Addresses FastTab: multiple addresses with purposes
- Contact Information FastTab
- Miscellaneous Details FastTab: country/region, direct delivery
- Credit & Collections FastTab: credit rating, credit limit, hold status
- Payment Defaults FastTab: terms of payment, method of payment, bank account
- Invoice and Delivery FastTab: invoice account, delivery terms, sales tax group

**Assessment: Comprehensive field-level guidance but missing workflow and advanced features**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard path via Sales and marketing module |
| Basic fields | Complete | Good coverage of key setup fields |
| Address management | Good | Multiple addresses with purpose explained |
| Contact information | Adequate | Basic guidance provided |
| Credit & collections setup | Good | Credit limit and hold status covered |
| Payment defaults | Good | Key payment fields covered |
| Invoice and delivery | Good | Invoice account and delivery terms |
| Customer creation workflow | **>> NEEDS REVIEW <<** | Not mentioned -- FRD GAP 001-004 |
| Proposed change workflow | **>> NEEDS REVIEW <<** | Not mentioned -- FRD GAP 001-005 |
| Cross-Company Data Sharing | **>> NEEDS REVIEW <<** | Not covered at all |
| Trade agreement setup | **>> NEEDS REVIEW <<** | Not covered -- FRD has detailed procedure |
| CRM credit hold integration | **>> NEEDS REVIEW <<** | Not covered -- FRD GAP 001-013 |

**>> ATTENTION AREA: Customer creation workflow is absent from training**

> FRD GAP 001-004 requires that customer creation follow an approval workflow (Sales department creates, Sales Manager approves). The training manual instructs users to create customers directly with no mention of any approval requirement. If the workflow is implemented as designed, users following this training will not understand why their newly created customers are in a pending/draft state or how to route them for approval.

**>> ENHANCEMENT:** Add a section after customer creation that covers:
1. The customer creation workflow -- what happens after clicking Save (record routes to Sales Manager for approval)
2. How to check approval status
3. What to do if the customer is rejected
4. The proposed change workflow for subsequent customer amendments (GAP 001-005)

**>> ATTENTION AREA: Cross-Company Data Sharing not trained**

> The FRD specifies that customer records are shared across legal entities via Cross-Company Data Sharing. Users need to understand that creating or modifying a customer in one entity affects all entities. This behavioral awareness is critical for a multi-entity environment and is completely absent from the training.

**>> ENHANCEMENT:** Add a training note explaining: "Customer records at Technica are shared across legal entities. When you create or modify a customer in one company (e.g., TINT), the change is automatically reflected in other companies (e.g., TMEN). Be aware that certain fields are shared and cannot differ between entities."

**>> ENHANCEMENT:** Add trade agreement setup training. The FRD includes detailed procedures for customer-specific and group-specific pricing via trade agreement journals. Users involved in pricing decisions need to know how to create, maintain, and apply trade agreements.

---

### Project Advance Payments

**What the training covers:**
- Navigation: Project management and accounting > Projects > All projects
- Selecting a project and navigating to on-account transactions
- Checking "mark as complete" field before posting invoice proposal
- RCO (Revised Contract Order) handling with quotation transfer
- Creating invoice proposals for on-account transactions
- Clearing end date to display transactions
- Selecting correct terms of payment and posting profile
- Posting subsequent advance payments
- Clearing the checkbox under advances and deductions
- Specifying the "Prepayment" posting profile for advance payments
- Checking financial dimensions on invoice proposal header
- PAID checkbox behavior after customer payment processing

**Assessment: This is the most Technica-specific section -- covers a custom process well but has accounting gaps**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation to project | Correct | Standard project module path |
| On-account transaction check | Good | "Mark as complete" validation is important |
| RCO handling | Good | Technica-specific business logic for quotation transfers |
| Invoice proposal creation | Correct | Standard D365 project invoice proposal flow |
| End date clearing | Good | Practical tip for displaying all transactions |
| Posting profile selection | **Partially correct** | Mentions "Prepayment" profile but does not explain the accounting impact |
| Financial dimensions check | Good | Important control point |
| Advance deduction clearing | Good | Critical step to avoid double-counting |
| Prepayment accounting treatment | **>> NEEDS REVIEW <<** | No explanation of where advances post (liability vs AR) |
| Payment linkage | Mentioned | References customer payment section |

**>> ATTENTION AREA: Advance payment accounting treatment is not explained**

> The FRD review identified (HIGH severity) that the advance payment accounting treatment is under-specified. The training manual compounds this problem by instructing users to select the "Prepayment" posting profile without explaining what this means financially. Users need to understand:
> - Advance payments post to a **Customer Advances (liability)** account, not to AR
> - This is critical for correct balance sheet classification
> - When the final invoice is posted and the advance is deducted, the liability is reclassified to AR

**>> ENHANCEMENT:** Add a brief accounting explanation before the procedural steps:

*"When Technica receives an advance payment from a customer before work begins, this is recorded as a Customer Advance (a liability) rather than as revenue. The 'Prepayment' posting profile ensures the advance is posted to the correct liability account. When the project milestone invoice is later issued and the advance is deducted, the system reclassifies the advance from the liability account to the AR control account. This ensures Technica's balance sheet correctly reflects the obligation to deliver goods/services against the advance."*

**>> ENHANCEMENT:** The RCO (Revised Contract Order) handling is Technica-specific business logic that appears to involve custom validation of quotation amounts versus on-account invoices. This should be explicitly labeled as a **custom feature** so users understand it is not standard D365 behavior. Add a note: "The RCO validation and automatic quotation amount update is a Technica customization. Standard D365 does not perform this cross-validation automatically."

---

### Invoicing a Packing Slip

**What the training covers:**
- Navigation: Project management and accounting > Projects > All projects
- Creating invoice proposal from project
- Selecting item requirement transaction origin under project transactions
- Deducting advance payments under "Advances and Deductions" tab
- Using "Split amount" to deduct the required value
- Changing posting profile to "Shipment" and specifying terms of payment
- Posting the invoice

**Assessment: Concise and procedurally correct but lacks context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Correct | Standard project module path |
| Invoice proposal creation | Correct | Standard D365 flow |
| Item requirement selection | Correct | Correct transaction origin filter |
| Advance deduction | Good | Split amount feature is an important practical detail |
| Posting profile change | Correct | "Shipment" profile for packing slip invoicing |
| Business context | **>> NEEDS REVIEW <<** | No explanation of when/why packing slip invoicing occurs |

**>> ENHANCEMENT:** Add introductory context explaining when packing slip invoicing applies at Technica. For example: "Packing slip invoicing is used when goods have been shipped to the customer under a project item requirement. This process converts the delivered items into an invoice while deducting any previously received advance payments."

**>> ENHANCEMENT:** The training should clarify the relationship between this process and the advance payments process. Currently, a user must mentally connect the "Project Advance Payments" section with the "Invoicing a Packing Slip" section. Add a process flow note: "Typical Technica project billing sequence: (1) Receive advance payment -> (2) Ship goods (packing slip) -> (3) Invoice the packing slip and deduct the advance -> (4) Collect remaining balance."

---

### Free Text Invoicing

**What the training covers:**
- Navigation: Accounts receivable > Invoices > All free text invoices
- Creating new FTI
- Entering customer account, method of payment, terms of payment
- Specifying main account on invoice lines
- Entering description, sales tax group, quantity, unit price
- Fixed asset number if applicable
- Posting the invoice

**Assessment: Basic procedure covered but missing workflow and limitations**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Correct | Standard D365 path |
| Header fields | Correct | Customer, payment method, terms |
| Line entry | Correct | Main account, description, tax, quantity, price |
| Fixed asset reference | Good | Includes fixed asset disposal use case |
| FTI workflow approval | **>> NEEDS REVIEW <<** | Not mentioned at all |
| Project allocation limitation | **>> NEEDS REVIEW <<** | Not mentioned |
| Invoice printing/layout | Not covered | No reference to the report layout GAP (AR001-002) |

**>> ATTENTION AREA: Free text invoice workflow is absent**

> The FRD states that FTI workflow approval is to be configured. The training instructs users to click "Post" directly after entering the invoice. If a workflow is configured, the Post button behavior changes -- the user must submit the invoice for approval first. This discrepancy will confuse users.

**>> ENHANCEMENT:** Add workflow steps: "After entering invoice details, click Submit for approval (if workflow is configured). The invoice will route to the designated approver. Once approved, the Post button becomes available. Direct posting without approval may be restricted based on Technica's workflow configuration."

**>> ENHANCEMENT:** Add a note about the project allocation limitation: "Free text invoices can only be allocated to Time & Material projects, not Fixed Price projects. For Technica's fixed-price project billing, use the Project Invoice Proposal process instead."

---

### Customer Payments

**What the training covers:**
- Navigation: Accounts receivable > Payments > Customer payment journal
- Creating new payment journal and selecting journal name
- Opening journal lines
- Entering: date, customer account, description, credit amount, currency, method of payment
- Financial dimensions entry for both account and offset account
- Settling transactions (linking payments to invoices)
- Marking invoices and updating amount for partial payment
- Posting the journal

**Assessment: Well-structured step-by-step procedure but missing automation and workflow**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Correct | Standard D365 path |
| Journal creation | Correct | Name selection tied to payment method |
| Line entry fields | Complete | All key fields covered |
| Financial dimensions | Good | Both account and offset dimensions covered |
| Settlement process | Good | Includes partial payment scenario |
| Note about later settlement | Good | Practical tip that settlement can be done later |
| Payment journal workflow | **>> NEEDS REVIEW <<** | Not mentioned -- FRD requires approval workflow |
| Bank reconciliation | **>> NEEDS REVIEW <<** | Not mentioned -- FRD flagged as critical |
| Electronic payments | Not covered | No guidance on electronic payment processing |
| Advance payment specifics | **>> NEEDS REVIEW <<** | No differentiation between regular and advance payments |

**>> ATTENTION AREA: Payment journal workflow approval is missing**

> The FRD specifies workflow approval for payment journals. The training shows direct posting without any approval submission step. If workflow is enabled, users will encounter a Submit button instead of being able to post directly.

**>> ENHANCEMENT:** Add workflow approval step: "Depending on Technica's configuration, payment journals may require approval before posting. Click Submit to route the journal for approval. Once approved, click Post."

**>> ATTENTION AREA: Advanced Bank Reconciliation not trained**

> The FRD review identified (HIGH severity) that Advanced Bank Reconciliation is critical for automated payment matching but is not mentioned in the FRD. The training manual also omits this entirely. For a company receiving bank transfers as the primary payment method, this is a major operational gap.

**>> ENHANCEMENT:** Add a section or sub-section on bank reconciliation: how to import bank statements, how to use matching rules, and how to handle unmatched transactions. Even if Advanced Bank Reconciliation is implemented post-go-live, the training manual should reference it as an upcoming capability.

**>> ENHANCEMENT:** Add specific guidance for processing advance payments versus regular payments. The current generic payment procedure does not explain how to handle an advance payment (which requires a different posting profile and posts to a liability account rather than settling against an invoice). Cross-reference with the "Project Advance Payments" section.

---

### Settlement and Undo Settlement for Customer Transactions

**What the training covers:**
- Navigation: Accounts receivable > Customer > All customers
- Settle transactions: linking posted payments (prepayment) to invoices
- Marking payment line and invoice line
- Posting the settlement
- Undo settlement: reversing settlement between payment and invoice
- Marking the line and clicking Reverse

**Assessment: Concise and correct but lacks safeguards and context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Correct | Via customer form |
| Settlement marking | Correct | Mark both payment and invoice |
| Undo settlement | Correct | Reverse function |
| Period-close restrictions | **>> NEEDS REVIEW <<** | No warning about closed periods |
| Audit implications | **>> NEEDS REVIEW <<** | No guidance on when undo is appropriate |
| Cross-project settlement | Not covered | FRD mentions one payment covering multiple project invoices |

**>> ENHANCEMENT:** Add warnings:
- "Undo settlement can only be performed if the period is still open. If the fiscal period has been closed, you must reopen it before reversing the settlement."
- "Undo settlement should only be used to correct errors. All settlement reversals are tracked in the audit trail. Consult your supervisor before undoing a settlement."

**>> ENHANCEMENT:** Add guidance for cross-project settlement scenarios: "When a single customer payment covers invoices from multiple projects, you can mark multiple invoices for settlement against one payment. Ensure the financial dimensions on the payment line correctly reflect the allocation across projects."

---

### Reverse Customer's Transactions

**What the training covers:**
- Click on Transactions
- Select the transaction to reverse
- Click Reverse transaction
- Enter reversal date and click OK

**Assessment: Incomplete -- missing navigation path and business rules**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Starting navigation | **>> NEEDS REVIEW <<** | Missing -- does not specify where to find the Transactions button |
| Reversal procedure | Correct | Standard D365 reversal steps |
| Reversal date guidance | Incomplete | No guidance on which date to use |
| Settled transaction reversal | Not covered | Cannot reverse a settled transaction without undo settlement first |

**>> ATTENTION AREA: Navigation path is missing**

> The section starts with "Click on Transactions" without specifying where the user should be. The correct navigation is: Accounts receivable > Customers > All customers > Select customer > Transactions (on the Action Pane under Customer tab). Without this, users cannot find the starting point.

**>> ENHANCEMENT:** Add:
1. Full navigation path
2. Note: "You cannot reverse a transaction that has been settled. If the transaction is settled, you must first undo the settlement (see Settlement and Undo Settlement section above) before reversing."
3. Guidance on reversal date: "The reversal date should be in an open fiscal period. If the original transaction date is in a closed period, use the first date of the current open period."

---

### Foreign Currency Revaluation

**What the training covers:**
- Navigation: Accounts receivable > Periodic task > Foreign currency revaluation
- Simulation step before actual processing
- Entering "Considered date" (posting date) and "Date of rate" (exchange rate date)
- Reviewing simulation report
- Running actual revaluation
- Reviewing posted voucher
- Marking transaction as "Reviewed"

**Assessment: Good procedural coverage with simulation step -- but missing exchange rate setup**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Correct | Standard D365 path |
| Simulation before processing | Excellent | Best practice -- review before commit |
| Date fields explanation | Good | Clear distinction between posting date and rate date |
| Voucher review | Good | Important verification step |
| Reviewed marking | Good | Control feature for management sign-off |
| Exchange rate maintenance | **>> NEEDS REVIEW <<** | No guidance on how exchange rates are entered/imported |
| Revaluation frequency | Not specified | No guidance on when to run (monthly per FRD) |
| "TECH" exchange rate type | Not mentioned | FRD specifies a custom exchange rate type |

**>> ATTENTION AREA: Exchange rate maintenance is not covered**

> The revaluation process assumes exchange rates already exist in the system. Users are not taught how rates are entered or maintained. The FRD review recommended automating exchange rate import instead of manual daily entry (MEDIUM severity). The training should at minimum explain how to manually enter rates and ideally reference the automated import if configured.

**>> ENHANCEMENT:** Add a prerequisite section: "Before running foreign currency revaluation, ensure exchange rates are up to date. Navigate to General ledger > Currencies > Exchange rates to verify or enter the current rates for the 'TECH' exchange rate type. Technica uses a custom exchange rate type called 'TECH' for all currency conversions."

**>> ENHANCEMENT:** Add frequency guidance: "Foreign currency revaluation should be run at minimum monthly as part of the month-end close process. During periods of high currency volatility, consider running weekly for more accurate interim financial statements."

---

### Intercompany Trade Operations

**What the training covers:**
- Navigation: Accounts receivable > Customers > All customers
- Selecting a customer and navigating to Intercompany setup
- Activating intercompany relationship
- Specifying vendor company and vendor account in the other company
- Sales order and purchase order policies
- Ensuring product availability across entities (TINT and TMEN)
- Viewing related orders from a sales order

**Assessment: Covers setup but is severely incomplete on operational procedures**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation to setup | Correct | Standard D365 intercompany setup path |
| Activation and vendor mapping | Correct | Core intercompany configuration |
| Policy configuration | Mentioned | References sales order and purchase order policies |
| Product availability note | Good | Important cross-entity requirement |
| Related order viewing | Good | Useful traceability feature |
| Intercompany SO creation | Not covered | No training on creating intercompany sales orders |
| Intercompany invoicing | **>> NEEDS REVIEW <<** | Not covered at all |
| Intercompany payment/settlement | **>> NEEDS REVIEW <<** | Not covered at all |
| Intercompany elimination | **>> NEEDS REVIEW <<** | Not covered -- FRD flagged this as missing |
| Transfer pricing | Not covered | FRD flagged compliance requirement |

**>> ATTENTION AREA: Intercompany trade training is setup-only -- operational procedures are missing**

> The FRD covers intercompany trade as a complete cycle: setup, automatic SO creation from PO, document number synchronization, pricing, and policy automation. The training manual only covers the initial setup and how to view related orders. Users are not trained on:
> 1. How to create an intercompany sales order (or how the automatic SO is triggered from a PO in the other entity)
> 2. How to invoice intercompany transactions
> 3. How to settle intercompany payments
> 4. How intercompany elimination works for consolidation

**>> ENHANCEMENT:** Expand this section significantly to include:
1. **Creating an intercompany transaction** -- demonstrate the full flow from PO in the buying company to automatic SO in the selling company
2. **Intercompany invoicing** -- how posting an invoice in one entity automatically creates a corresponding entry in the other
3. **Intercompany settlement** -- how to settle intercompany balances
4. **Entity-specific notes** -- clarify which steps are performed in TINT vs TMEN

The FRD review also recommended keeping intercompany automation conservative at go-live. The training should reflect the actual automation level configured and clearly state which steps are automatic versus manual.

---

### Accounts Receivable Reports

#### Customer Account Statement Report

**What the training covers:**
- Navigation: Accounts receivable > Inquiries and reports > Customer reports > Customer account statement report

**Assessment: Navigation only -- no explanation of parameters or output**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Report parameters | Not covered | No guidance on date interval, customer selection, etc. |
| Report interpretation | Not covered | No explanation of what the output shows |
| Project ID display | **>> NEEDS REVIEW <<** | FRD GAP AR006-001 not addressed |

---

#### Customer Aging Report

**What the training covers:**
- Brief description: classifies outstanding balances by aging periods (0-30, 31-60, etc.)
- Navigation: Credit and collections > Inquiries and reports > Customer report > Customer aging report

**Assessment: Minimal -- description is helpful but no operational guidance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Concept explanation | Good | Clear description of aging purpose |
| Navigation path | Correct | Standard D365 path |
| Aging period configuration | Not covered | No guidance on Technica's extended aging brackets (30/60/90/180/1y/1.5y/2y/3y) |
| Report parameters | Not covered | No guidance on running the report |
| Project ID display | **>> NEEDS REVIEW <<** | FRD GAP AR006-002 not addressed |

---

#### Aged Balances

**What the training covers:**
- Description: displays outstanding balances segmented by aging periods
- Allows real-time or snapshot-based analysis
- Navigation: Credit and collection Module > Collection > Aged Balances

**Assessment: Good description but overlaps with Customer Aging Report without differentiation**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Concept explanation | Good | Clear description of purpose |
| Real-time vs snapshot distinction | Good | Important operational concept |
| Navigation path | Correct | Standard D365 path |
| Differentiation from Aging Report | **>> NEEDS REVIEW <<** | Users may be confused about when to use which report |

**>> ENHANCEMENT:** Add a note explaining the difference between the Customer Aging Report (a printable SSRS report) and the Aged Balances inquiry (an interactive screen within the Collections workspace). Users need to understand when to use each.

---

#### Customer to Ledger Reconciliation

**What the training covers:**
- Description: vital month-end process ensuring sub-ledger matches GL control account
- Validates all customer transactions are accurately recorded
- Navigation: Accounts receivable > Inquiries and reports > Customer reports > Customer to Ledger Reconciliation

**Assessment: Excellent conceptual explanation -- best-described report in the manual**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Concept explanation | Excellent | Clear explanation of purpose, timing, and importance |
| Navigation path | Correct | Standard D365 path |
| Reconciliation procedure | Not covered | No step-by-step for investigating discrepancies |
| Frequency guidance | Implied | "Month-end" mentioned |

**>> ENHANCEMENT:** Add guidance on what to do when discrepancies are found: "If the sub-ledger total does not match the GL control account, investigate: (1) Unposted journals, (2) Direct GL postings to the AR control account (which should be restricted), (3) Timing differences in period cutoff."

---

#### Missing Reports from FRD

**>> ATTENTION AREA: Reports coverage does not match FRD scope**

> The FRD defines four reports under AR-006, plus a recommendation for a cash application report. The training manual covers four reports but the set does not fully align:

| FRD Report | Training Manual Coverage |
|------------|------------------------|
| Customer Account Statement | Covered (navigation only) |
| Customer Invoice Transaction Report | **NOT COVERED** |
| Customer Transactions Inquiry | **NOT COVERED** (closest is Customer to Ledger Reconciliation, which is different) |
| Customer Aging Report | Covered (description + navigation) |
| Cash Application / Unapplied Advances Report (recommended) | **NOT COVERED** |

The training manual includes **Aged Balances** and **Customer to Ledger Reconciliation** which are not in the original FRD report list but are valid and useful additions.

**>> ENHANCEMENT:** Add the Customer Invoice Transaction Report and Customer Transactions Inquiry to the training manual to match FRD scope. Also add the recommended Cash Application report if it is configured.

**>> ENHANCEMENT:** For all reports, add:
1. Parameter explanation (what each filter does)
2. A sample output interpretation (what key columns mean)
3. When to use each report (daily, weekly, monthly, ad-hoc)
4. Project ID filtering/display guidance once GAPs AR006-001 through AR006-004 are implemented

---

## Cross-Reference with FRD

This section compares the training manual content against the findings and recommendations in FRD12 (Accounts Receivable) review to identify gaps between what was designed and what is being taught.

### Critical FRD Findings vs Training Coverage

| FRD Finding | Severity in FRD | Training Manual Status | Gap Assessment |
|-------------|-----------------|----------------------|----------------|
| Project milestone invoicing integration absent -- AR-Project end-to-end flow must be designed | **CRITICAL** | **Partially covered** -- Project Advance Payments and Packing Slip sections exist but do not cover the full milestone lifecycle (milestone completion -> invoice proposal -> AR posting -> advance settlement -> collection) | **CRITICAL GAP** -- Training has project-related steps but not the complete milestone billing flow that the FRD flagged as the primary revenue channel |
| Black market rate invoicing (GAP AR003-003) -- highest-complexity GAP requiring cross-FRD design | **CRITICAL** | **NOT COVERED** -- No mention of dual exchange rate invoicing, mixed VAT invoices, or Lebanon-specific rate scenarios | **CRITICAL GAP** -- Users will have zero guidance on this complex scenario |
| Customer creation workflow (GAP 001-004) -- verify OOB or build custom | **HIGH** | **NOT COVERED** -- Training shows direct customer creation with no workflow | **HIGH GAP** -- Users will not understand the approval process |
| Credit hold CRM integration (GAP 001-013) -- mechanism undefined | **HIGH** | **NOT COVERED** -- Credit & Collections FastTab is mentioned but no CRM sync behavior explained | **MEDIUM GAP** -- Impacts CRM users more than F&O users |
| Payment terms undefined -- pending from Technica | **HIGH** | **PARTIALLY COVERED** -- Generic creation procedure shown but no Technica-specific terms | **Consistent gap** -- Both FRD and training lack specific terms |
| Advance payment accounting -- prepayment posting profile needed | **HIGH** | **PARTIALLY COVERED** -- "Prepayment" profile mentioned in steps but accounting treatment not explained | **HIGH GAP** -- Users told to select the profile but not why |
| Advanced Bank Reconciliation -- critical for automated payment matching | **HIGH** | **NOT COVERED** | **HIGH GAP** -- Major operational efficiency feature omitted |
| Project ID on reports (GAPs AR006-001 through AR006-004) | **HIGH** | **NOT COVERED** -- Reports section shows navigation paths only, no Project ID filtering | **MEDIUM GAP** -- Depends on whether the GAPs have been implemented |
| Customer groups too simple (Local/Foreign) | **MEDIUM** | **NOT COVERED** -- Generic group creation shown | **LOW GAP** -- Configuration issue, not training issue |
| Single posting profile risky for intercompany | **MEDIUM** | **NOT COVERED** -- Only one posting profile concept shown | **MEDIUM GAP** -- Multiple profiles referenced in process sections but not in config section |
| ER framework for invoice reports instead of SSRS | **MEDIUM** | **NOT COVERED** -- No report customization training | **LOW GAP** -- Developer concern, not end-user training |
| Electronic payment methods for foreign customers | **MEDIUM** | **NOT COVERED** | **MEDIUM GAP** -- Users handling foreign payments need this |
| Automate exchange rate import | **MEDIUM** | **NOT COVERED** -- Exchange rate maintenance entirely absent | **MEDIUM GAP** -- Users need to know how rates get into the system |
| Conservative intercompany automation at go-live | **MEDIUM** | **NOT APPLICABLE** -- Training only covers setup, not operations | **Indeterminate** -- Cannot assess automation level from training |
| Power BI for AR reporting | **MEDIUM** | **NOT COVERED** | **LOW GAP** -- May be a post-go-live initiative |
| Cross-Company Data Sharing field planning | **LOW** | **NOT COVERED** | **MEDIUM GAP** -- Users need behavioral awareness in multi-entity environment |
| Cash application / unapplied advance payments report | **LOW** | **NOT COVERED** | **LOW GAP** -- Depends on implementation |

### FRD Processes vs Training Manual Processes

| FRD Process | Training Manual Section | Alignment |
|-------------|------------------------|-----------|
| Customer Master Setup (detailed) | Customer Creation + Configuration sections | **Partial** -- Training covers creation fields but misses workflow, data sharing, trade agreements |
| AR-001: Free Text Invoicing | Free Text Invoicing | **Partial** -- Basic procedure covered, workflow and project limitation missing |
| AR-002: Customer Payments | Customer Payments + Settlement sections | **Partial** -- Manual payment covered, bank reconciliation and workflow missing |
| AR-003: Sales Order Processing | **NOT COVERED** | **MAJOR GAP** -- No sales order processing training at all. FRD defines the standard SO cycle (Create > Confirm > Packing Slip > Invoice > Payment) but training manual does not include it |
| AR-004: Foreign Currency Revaluation | Foreign Currency Revaluation | **Good** -- Process steps well-covered, exchange rate maintenance missing |
| AR-005: Intercompany Trade Operations | Intercompany Trade Operations | **Partial** -- Setup covered, operational procedures missing |
| AR-006: Inquiries & Reports | Accounts Receivable Reports | **Partial** -- Different report set, minimal operational guidance |
| Project Milestone Invoicing (cross-FRD) | Project Advance Payments + Invoicing a Packing Slip | **Partial** -- Project-specific steps exist but no complete lifecycle |

**>> ATTENTION AREA: Sales Order Processing (AR-003) is entirely absent from the training manual**

> The FRD defines AR-003 as the standard sales order cycle: Create SO > Confirm > Packing Slip (partial delivery) > Invoice > Payment. This is a core AR process for non-project sales (spare parts, non-project equipment). The training manual does not cover sales order invoicing at all. Users handling non-project sales will have no training guidance.

**>> ENHANCEMENT:** Add a "Sales Order Invoicing" section covering: (1) How sales order invoices are generated from the sales order, (2) Partial delivery and partial invoicing scenarios, (3) The relationship between the sales order invoice and the customer payment/settlement process.

---

## Document Quality Observations

The following quality issues were identified in the training manual text:

| Issue | Location | Detail |
|-------|----------|--------|
| Spacing in navigation path | Customer Posting Profile | "Accounts receivable > S etup > Customer posting profile" -- extra space in "Setup" |
| Heading spacing | Accounts receivable processes | "Accounts receivable pr ocesses" -- broken word |
| Heading spacing | Customer creation | "C ustomer creation" -- broken word |
| FastTab spelling | Customer creation section | "fas t t ab" appears multiple times instead of "FastTab" |
| FastTab capitalization | Customer creation section | "p ayment d efaults f as t t ab" -- broken words and lowercase |
| Double comma | Aged Balances description | "assess, credit risk" has a double comma nearby ("overdue invoices,, helping") |
| Inconsistent formatting | Throughout | Some sections use bullet lists (List Paragraph style), others use normal paragraphs for step-by-step instructions |
| Missing section numbers | Throughout | No numbered sections or consistent hierarchy -- makes cross-referencing difficult |
| No version history | Document properties | Author is Jana Sweid, created 2026-01-26, but no change log or version history visible in the document |
| Inconsistent heading style | Aged Balances | "Aged Balances:" has a colon while other headings do not |

**>> ENHANCEMENT:** Perform a formatting and proofreading pass across the entire document to correct typographical errors, standardize FastTab terminology, add section numbers, and ensure consistent use of bullet lists versus numbered steps.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | Project Advance Payments | Add accounting explanation for prepayment posting profile -- where advances post and why | Critical | Prevents financial misstatement from incorrect posting profile selection |
| 2 | Cross-FRD | Add complete project milestone billing lifecycle training (milestone completion through collection) | Critical | Core revenue cycle for Technica's make-to-order business |
| 3 | Black Market Rate | Add dual exchange rate / mixed VAT invoicing procedure once GAP AR003-003 is developed | Critical | Lebanon-specific regulatory requirement with no user guidance |
| 4 | Customer Creation | Add customer creation workflow training (approval routing, status checking, rejection handling) | High | Users will not understand workflow behavior without training |
| 5 | Customer Creation | Add Cross-Company Data Sharing awareness training for multi-entity behavior | High | Prevents unintended changes propagating across entities |
| 6 | Customer Payments | Add Advanced Bank Reconciliation training (bank statement import, matching rules) | High | Major operational efficiency feature |
| 7 | Customer Payments | Add payment journal workflow approval steps | High | Training conflicts with configured workflow behavior |
| 8 | Configuration | Add prepayment, shipment, and intercompany posting profile configuration training | High | Profiles referenced in process sections but never taught |
| 9 | Free Text Invoicing | Add FTI workflow approval steps and project allocation limitation note | High | Training shows direct posting which may be restricted |
| 10 | Intercompany Trade | Expand to full operational training: SO creation, invoicing, settlement, elimination | High | Current coverage is setup-only |
| 11 | Sales Order Processing | Add entire AR-003 sales order invoicing section (missing from manual) | High | FRD process with zero training coverage |
| 12 | Configuration | Add Technica-specific payment terms examples (advance, milestone, Net 30/60/90, LC) | Medium | Generic procedure without business context |
| 13 | Configuration | Add electronic payment method configuration (SEPA, wire, LC) | Medium | Supports international customer operations |
| 14 | Configuration | Add specific customer groups with business purpose explanations | Medium | Generic procedure without Technica context |
| 15 | Customer Creation | Add trade agreement setup training for customer pricing | Medium | Detailed FRD content with no training coverage |
| 16 | Foreign Currency Reval. | Add exchange rate maintenance prerequisite (manual entry or automated import) | Medium | Users assume rates exist but need to know how they get there |
| 17 | Reports | Add Project ID filtering guidance once GAPs AR006-001 through AR006-004 are implemented | Medium | Key FRD requirement for project-based AR reporting |
| 18 | Reports | Add Customer Invoice Transaction Report and Customer Transactions Inquiry | Medium | FRD-defined reports missing from training |
| 19 | Reports | Add parameter explanations, output interpretation, and usage frequency for all reports | Medium | Current coverage is navigation-only |
| 20 | Settlement | Add period-close warnings and audit implications for undo settlement | Low | Prevents operational errors in closed periods |
| 21 | Reverse Transactions | Add full navigation path and business rules (settled transaction restriction) | Low | Section is currently unusable without starting point |
| 22 | Reports | Add cash application / unapplied advance payments report | Low | FRD-recommended report for advance payment tracking |
| 23 | Document Quality | Perform formatting, proofreading, and structural standardization pass | Low | Multiple typographical errors and inconsistent formatting |

---

## Risk Flags

1. **>> ATTENTION AREA: Training manual does not reflect the designed solution for project milestone invoicing.**

   > The FRD review identified project milestone invoicing integration as the single most critical missing coverage area in the FRD itself. The training manual partially addresses this through "Project Advance Payments" and "Invoicing a Packing Slip" sections, but these do not constitute a complete end-to-end training for Technica's primary revenue cycle. Users processing milestone invoices will need to piece together instructions from multiple sections and rely on tribal knowledge for gaps. This creates a high risk of processing errors during and after go-live.

2. **>> ATTENTION AREA: Black market rate invoicing (highest-complexity GAP) has zero training coverage.**

   > GAP AR003-003 was flagged in the FRD review as the single highest-complexity item requiring cross-FRD technical design across GL, Tax, AR, and Sales modules. The training manual contains no reference to dual exchange rate scenarios, mixed VAT invoices, or any Lebanon-specific invoicing procedures. Once this customization is developed, comprehensive training must be created -- this is not a feature users can intuit.

3. **>> ATTENTION AREA: Sales order processing (AR-003) is entirely absent from the training manual.**

   > The FRD defines a complete sales order cycle (Create SO > Confirm > Packing Slip > Invoice > Payment) as process AR-003. The training manual does not include any sales order invoicing training. While project invoicing may account for the majority of Technica's revenue, non-project sales (spare parts, one-off equipment) still flow through sales orders and users need to be trained.

4. **>> ATTENTION AREA: Multiple workflow-dependent processes are taught without workflow steps.**

   > The FRD specifies workflow approval for customer creation, free text invoices, and payment journals. The training manual instructs users to perform these actions directly (create customer, post FTI, post payment) with no mention of approval routing. When workflows are enabled, the actual user experience will differ significantly from what the training teaches. This creates a high risk of user confusion and support calls during go-live.

5. **>> ATTENTION AREA: Posting profile training is fragmented and incomplete.**

   > The configuration section teaches one generic posting profile setup. The process sections reference three distinct profiles (General, Prepayment, Shipment) without explaining their configuration or accounting impact. Users are told to "make sure to select the correct posting profile" without understanding what "correct" means or what the consequences of a wrong selection are. For a make-to-order manufacturer with advance payments, this is a financial control risk.

6. **>> ATTENTION AREA: Intercompany training covers only 30% of the required knowledge.**

   > The intercompany section covers initial setup and order viewing but omits the operational cycle: creating intercompany transactions, invoicing, payment settlement, and elimination. For a multi-entity manufacturer (TINT and TMEN), incomplete intercompany training risks processing errors that affect both entities and complicate consolidation.

7. **>> ATTENTION AREA: Report training provides navigation paths but no operational guidance.**

   > All four report sections provide only the menu path to reach the report. None explain the report parameters, how to interpret the output, or when to run the report. Users will reach the report screen but not know how to configure it for their specific needs. The FRD's requirement for Project ID on reports (4 GAPs) is also not addressed.

---

*This review is based on D365 F&O Accounts Receivable, Credit & Collections, and Project Accounting best practices as of 2026. The training manual was assessed against both D365 standard functionality and the specific requirements documented in FRD12 - Accounts Receivable. Recommendations aim to close gaps between the designed solution and user training, ensure completeness for go-live readiness, and reduce operational risk from undertrained users.*
