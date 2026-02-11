# Training Manual - Accounts Payable - Solution Design Verification Report

**Document:** Technica - User Training Manual - Accounts Payable V2.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Accounts Payable

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Prepayments on PO | Training teaches standalone prepayment but omits auto-application of prepayment against final PO invoice -- users will not know how to settle prepayments | [Prepayments on Purchase Order](#4-prepayments-on-the-purchase-order) |
| 2 | **CRITICAL** | FRD Cross-Reference | FRD CRITICAL finding (AP004 -- redesign advance payments to use OOB PO Prepayment) is not reflected in training -- training still teaches partial manual approach | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 3 | **CRITICAL** | Non-PO Invoices | Entire non-PO invoice process (Invoice Register, Approval Journal, recurring invoices) from FRD AP002 is missing from the training manual | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 4 | **HIGH** | Vendor Payments | Payment Proposal generation is not documented -- users are only taught manual line-by-line payment entry, missing the primary D365 payment workflow | [Vendor Payments](#5-vendor-payments) |
| 5 | **HIGH** | Vendor Payments | Check printing, void, reissue process is absent -- critical for organizations using check-based vendor payments | [Vendor Payments](#5-vendor-payments) |
| 6 | **HIGH** | Invoice Matching | No training on 2-way/3-way invoice matching validation -- a core AP control in D365 that the FRD explicitly designs for | [PO Invoice Processing](#3-invoicing-a-purchase-order) |
| 7 | **HIGH** | Workflow Approvals | Vendor invoice workflow approval process is not documented anywhere in the manual -- users will not know how to submit/approve/reject invoices | [PO Invoice Processing](#3-invoicing-a-purchase-order) |
| 8 | **HIGH** | Exchange Rate Modification | Payment journal exchange rate instruction to "multiply by 100" is incorrect for standard D365 -- this could cause posting errors of 100x magnitude | [Exchange Rate Modification](#7-exchange-rate-modification) |
| 9 | **HIGH** | FRD Cross-Reference | Letter of Credit (LC) payment process identified in FRD is completely absent from training | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 10 | **MEDIUM** | Vendor Creation | No training on vendor approval workflow -- vendor master changes in D365 typically require approval before activation | [Vendor Creation](#1-vendor-creation) |
| 11 | **MEDIUM** | Vendor Creation | Withholding tax group setup on vendor master is not covered -- required for MENA jurisdictions | [Vendor Creation](#1-vendor-creation) |
| 12 | **MEDIUM** | Vendor Invoice Journal | No guidance on when to use Vendor Invoice Journal vs. PO Invoice vs. Invoice Register -- users may select wrong journal type | [Vendor Invoice Journal](#2-create-vendor-invoice-journal) |
| 13 | **MEDIUM** | Charges Codes | Configuration section documents charges codes setup but no process section teaches users how to apply charges on transactions | [Configuration Review](#accounts-payable-configuration-review) |
| 14 | **MEDIUM** | Settlement | Settlement process is taught but lacks guidance on partial settlement, cash discount handling, and cross-company settlement | [Settlement](#6-settlement-and-undo-settlement) |
| 15 | **MEDIUM** | Foreign Currency Revaluation | No explanation of when or why to run revaluation -- users are taught button clicks without understanding the business purpose or impact | [Foreign Currency Revaluation](#8-foreign-currency-revaluation) |
| 16 | **MEDIUM** | Vendor Posting Profile | Training only shows Group-level posting profile setup -- does not explain Table or All options, or when each is appropriate | [Configuration Review](#accounts-payable-configuration-review) |
| 17 | **MEDIUM** | Reports | Aging report section does not explain how to configure aging period definitions or interpret aging buckets | [Reports Review](#9-accounts-payable-reports) |
| 18 | **MEDIUM** | FRD Cross-Reference | Post-dated check (PDC) handling from FRD AP003 is not covered in training -- common requirement in MENA region | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 19 | **MEDIUM** | FRD Cross-Reference | Budget control on non-PO invoices (FRD HIGH finding) has no training coverage | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 20 | **LOW** | Configuration | Number sequences for AP are not mentioned -- users/admins need to understand the numbering logic | [Configuration Review](#accounts-payable-configuration-review) |
| 21 | **LOW** | Vendor Creation | No mention of vendor hold/block functionality for placing vendors on payment or invoicing hold | [Vendor Creation](#1-vendor-creation) |
| 22 | **LOW** | Reports | No training on Power BI embedded analytics or Vendor Analytics workspace -- modern reporting capabilities are ignored | [Reports Review](#9-accounts-payable-reports) |
| 23 | **LOW** | General | Document lacks a glossary, keyboard shortcuts, troubleshooting tips, or error resolution guidance for common AP issues | [General Document Quality](#general-document-quality) |
| 24 | **LOW** | Reversal | Reversal section is very brief -- does not cover reversal of settled transactions, reversal reasons, or trace number handling | [Transaction Reversal](#reversal-of-vendor-transactions) |

**Totals:** 3 CRITICAL | 6 HIGH | 9 MEDIUM | 5 LOW

---

## Executive Summary

This training manual covers the Accounts Payable module in Dynamics 365 Finance and Operations for Technica. The document is organized into three main sections: (1) AP Configuration (terms of payment, methods of payment, vendor groups, vendor posting profiles, charges codes), (2) AP Processes (vendor creation, vendor invoice journal, PO invoicing, prepayments, vendor payments, settlement/undo settlement, transaction reversal, exchange rate modification, foreign currency revaluation), and (3) AP Reports (vendor balance inquiry, transaction inquiry, invoice inquiry, transaction report, balance list, account statement, invoice transactions, and aging report).

**Overall Assessment: The training manual provides a basic procedural walkthrough of core AP functions but has significant gaps in coverage when compared against both D365 best practices and the FRD solution design.** The manual focuses heavily on step-by-step button clicks with screenshots, which is useful for new users, but lacks the contextual understanding (when to use which process, why certain controls exist, what happens when things go wrong) that enables users to operate independently. Most critically, several processes designed in the FRD (non-PO invoices, payment proposals, invoice matching, workflow approvals, post-dated checks, Letter of Credit) are entirely absent from the training material, meaning users will not be trained on processes they are expected to perform.

The manual is strongest in its coverage of basic vendor creation, PO invoicing mechanics, and reporting inquiries. It is weakest in its treatment of advance payments (incomplete lifecycle), payment processing (manual-only, no payment proposal), and the complete absence of non-PO invoice handling which represents a substantial portion of daily AP work.

---

## Process-by-Process Review

### Accounts Payable Configuration Review

**What was documented:**
- Terms of Payment: Navigation, creation, payment method selection (Net/COD), payment day association
- Method of Payment: Navigation, creation, posting area setup (Ledger or Bank), payment account selection
- Vendor Group: Navigation, creation, code/description, terms of payment, tax group, payment-to-due-date interval
- Vendor Posting Profile: Summary account assignment by vendor group
- Charges Codes: Code creation, debit/credit type setup (Item, Ledger account, Customer/Vendor), posting account assignment

**Assessment: Adequate for basic setup awareness, but lacks depth for administrators**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Terms of Payment | Acceptable | Covers basic setup; misses advanced options (installment plans, payment schedules) |
| Method of Payment | Acceptable | Covers basics; missing file format setup for electronic payments |
| Vendor Group | Acceptable | Basic coverage; no explanation of downstream impact on posting/reporting |
| Vendor Posting Profile | **>> NEEDS REVIEW <<** | Only shows Group-level setup; does not explain Table vs. All vs. Group or multi-profile scenarios |
| Charges Codes | **>> NEEDS REVIEW <<** | Setup is documented but usage in transactions is never covered |
| Number Sequences | Missing | Not mentioned at all |
| AP Parameters | Missing | Global AP parameters not covered |
| Matching Policy Setup | Missing | Critical for invoice processing -- not in configuration section |

**>> ATTENTION AREA: Configuration section covers setup but never explains practical application**

> The charges codes section (Section 5 of configuration) documents how to create charge codes but nowhere in the process sections does the manual teach users how to apply charges to a vendor invoice or purchase order. Users will know how to create a charge code but not how to use one. Similarly, vendor groups are set up but the downstream impact on posting profiles, default payment terms, and tax behavior is not explained.

**>> ATTENTION AREA: Vendor Posting Profile training is incomplete**

> The manual only demonstrates the "Group" option in the Account code field. In D365, posting profiles support three levels: **Table** (specific vendor), **Group** (vendor group), and **All** (default fallback). Users and administrators need to understand the hierarchy and when each level is appropriate. Additionally, the concept of multiple posting profiles (e.g., separate profiles for prepayments vs. standard invoices) is not addressed, which is directly relevant to the prepayment process covered later.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a brief explanation of how each configuration element impacts downstream transactions. For example, after the Terms of Payment section, add a note: "The term of payment assigned to a vendor will default onto all purchase orders and invoices for that vendor, controlling when the payment becomes due."

2. **>> ENHANCEMENT:** Expand the Vendor Posting Profile section to cover Table, Group, and All account code levels, and explain the lookup hierarchy. Include a note about prepayment posting profiles which are essential for the advance payment process documented later in the manual.

3. **>> ENHANCEMENT:** Add a section on AP Parameters (Accounts payable > Setup > Accounts payable parameters) covering at minimum: invoice matching validation settings, default financial dimension behavior, and workflow enablement. These are foundational settings that affect every AP process.

4. **>> ENHANCEMENT:** Add number sequence configuration reference (Accounts payable > Setup > Accounts payable parameters > Number sequences tab) so administrators understand how invoice numbers, payment numbers, and voucher numbers are generated.

---

### 1. Vendor Creation

**What was documented:**
- Navigation to vendor creation form (Procurement and sourcing > Vendors > All Vendors > New)
- Person vs. Organization type selection
- Vendor name, group, language assignment
- Address entry
- Contact information (phone, email, URL, telex, fax, social media)
- Currency, sales tax group, delivery terms, mode of delivery
- Terms of payment, method of payment
- Bank account setup (bank account identifier, name, bank group, account number, IBAN)
- Preferred bank account assignment under Payment tab
- Contact people setup
- Preferred contact under purchasing demographics

**Assessment: Good coverage of basic vendor creation with notable omissions**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Basic vendor fields | Good | Name, group, language, address well covered |
| Contact information | Good | Multiple contact types explained |
| Financial defaults (currency, tax, payment) | Good | Key fields highlighted in bold |
| Bank account setup | Good | Detailed steps with screenshots |
| Contact people | Good | Setup and preferred contact covered |
| Vendor approval workflow | **>> NEEDS REVIEW <<** | Not mentioned at all |
| Vendor hold/block | Missing | No coverage of invoicing/payment hold |
| Withholding tax setup | Missing | Not covered; required for MENA operations |
| One-time vendor | Missing | Scenario not addressed |
| Vendor collaboration portal | Missing | External vendor access not mentioned |

**>> ATTENTION AREA: Vendor approval workflow is absent**

> In a controlled D365 environment, new vendor creation and vendor master changes typically go through a **Proposed Vendor / Vendor Change Request** approval workflow. The FRD identifies vendor creation as a managed process, but the training manual does not mention any approval step. Users may create vendors assuming they are immediately active, when in reality the vendor may be in a "Proposed" or "Hold" state pending approval. This creates confusion and delays if users try to create POs or invoices against unapproved vendors.

**>> ATTENTION AREA: Withholding tax group not covered**

> For companies operating in MENA jurisdictions (which the FRD confirms for Technica), withholding tax on vendor payments is a common requirement. The FRD review (Risk Flag #6) specifically flags that withholding tax is listed in setup but not documented in any process. The training manual perpetuates this gap by not showing users where to assign the withholding tax group on the vendor master (Vendor details > Invoice and delivery tab > Withholding tax group).

**Recommendations:**

1. **>> ENHANCEMENT:** Add a section on vendor approval workflow: after clicking Save on a new vendor, explain that the vendor may enter a "Proposed" state and require workflow approval before it becomes active. Show the workflow submission and approval steps.

2. **>> ENHANCEMENT:** Add coverage of vendor hold functionality (Vendor details > General section > On hold field). Explain the three hold options: No (active), Invoice (block invoicing), All (block all transactions), and Payment (block payments only). This is a critical control that AP users need to understand.

3. **>> ENHANCEMENT:** Include withholding tax group assignment in the vendor creation steps, particularly for service vendors where withholding is commonly applicable. Add after Step 8 (Sales tax group): "Select the withholding tax group if applicable to this vendor (common for service providers in certain jurisdictions)."

---

### 2. Create Vendor Invoice Journal

**What was documented:**
- Navigation to Invoice journal (Accounts payable > Invoices > Invoice journal)
- Journal creation and name selection
- Opening journal lines
- Line entry: date, vendor account, invoice date, invoice number, description, credit amount, currency
- Offset account setup (ledger, fixed asset, or project)
- Sales tax group and item sales tax group selection
- Financial dimensions for both vendor account and offset account
- Validation and posting (with note about workflow submission if applicable)

**Assessment: Good procedural steps but missing critical context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation and creation | Good | Clear step-by-step |
| Line entry fields | Good | All key fields covered |
| Financial dimensions | Good | Both account and offset account dimensions addressed |
| Tax handling | Good | Both tax group fields explained |
| Workflow mention | Minimal | One sentence: "In case there is a workflow defined, submit to workflow" -- no detail |
| When to use this journal | **>> NEEDS REVIEW <<** | No guidance on when Invoice Journal vs. PO Invoice vs. Invoice Register is appropriate |
| Multi-line invoices | Missing | Only single-line scenario shown |
| Default account from journal name | Missing | Not explained |
| Document attachment | Missing | Attaching source invoice document not covered |

**>> ATTENTION AREA: No guidance on journal type selection**

> D365 AP has multiple invoice entry methods: **Vendor Invoice Journal** (for non-PO invoices posted directly), **Invoice Register + Approval Journal** (for 2-step non-PO processing), **PO Invoice** (for purchase-order-linked invoices), and **Pending Vendor Invoices** (for PO invoices saved but not yet posted). The training manual documents the Vendor Invoice Journal and PO Invoice separately but never explains when to use which. A user receiving a vendor invoice has no guidance on which path to follow. This is a fundamental navigation gap that will cause confusion and incorrect postings.

**>> ATTENTION AREA: Workflow handling is insufficiently documented**

> The manual mentions workflow in a single sentence ("In case there is a workflow defined, submit the invoice to the workflow"). For a process that the FRD designs with workflow approval, this is inadequate. Users need to understand: how to submit to workflow, how to view workflow status, how to respond to workflow tasks (approve/reject/delegate/request change), and what happens when a workflow is rejected.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a decision tree or table at the beginning of the AP Processes section explaining when to use each invoice entry method:
   - **Vendor Invoice Journal**: Non-PO invoices that do not require departmental approval (e.g., already approved expenses)
   - **Invoice Register + Approval Journal**: Non-PO invoices requiring departmental review and coding
   - **PO Invoice (from Purchase Order)**: Invoices that match a purchase order
   - **Pending Vendor Invoices**: PO invoices saved as draft for later review and posting

2. **>> ENHANCEMENT:** Expand workflow coverage to include: submitting a journal to workflow, checking workflow status (via the yellow workflow bar), approving/rejecting workflow items from the work items queue, and handling rejected invoices (edit and resubmit).

3. **>> ENHANCEMENT:** Add a note about document attachment: "Use the Attach button (paperclip icon) on the journal header or line to attach the scanned vendor invoice for audit purposes. This is standard D365 Document Management functionality."

---

### 3. Invoicing a Purchase Order

**What was documented:**
- Navigation to purchase orders (Accounts payable > Purchase orders > All purchase orders)
- Opening the Invoice tab and clicking Invoice in Generate group
- Alternative path via Pending vendor invoices (Accounts payable > Invoices > Pending vendor invoices)
- Setting quantity to "Product receipt quantity"
- Updating vendor invoice header data
- Updating line quantity or unit price
- Sales tax group and item sales tax group selection on line details
- Checking totals before posting
- Posting the invoice

**Assessment: Covers basic PO invoicing mechanics but misses critical control features**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation and invoice creation | Good | Both paths documented (from PO and from Pending) |
| Quantity matching to product receipt | Good | Correctly instructs to use "Product receipt quantity" |
| Header and line updates | Good | Key fields addressed |
| Tax handling | Good | Both tax groups on line level |
| Totals review before posting | Good | Important control step included |
| Invoice matching validation | **>> NEEDS REVIEW <<** | Completely absent -- this is a core D365 AP control |
| Match variance handling | Missing | No guidance on what to do when matching fails |
| Invoice workflow approval | Missing | Not documented for PO invoices |
| Credit note processing | Missing | Not covered |
| Charges on PO invoice | Missing | Not covered despite charges codes being in configuration |
| Intercompany invoicing | Missing | Not covered |

**>> ATTENTION AREA: Invoice matching validation is completely absent from training**

> Invoice matching (2-way: PO-to-invoice, and 3-way: PO-to-product-receipt-to-invoice) is one of the most critical AP controls in D365. The FRD (AP001) explicitly designs for both 2-way and 3-way matching with price tolerance, quantity tolerance, and charge matching. The training manual does not mention invoice matching at all. Users will not know:
> - How to view matching results (Invoice matching details button)
> - What the matching status icons mean (green check, yellow warning, red X)
> - What to do when a match fails (route for approval, override, investigate)
> - How matching policies differ by item type (3-way for stocked items, 2-way for services)
>
> This is a fundamental gap. Without matching validation training, users may post invoices with significant price or quantity variances, bypassing a key financial control.

**>> ATTENTION AREA: Credit note processing is not documented**

> The FRD (AP001) includes credit note processing as a standard process. The training manual does not cover how to create a vendor credit note (either via the "Create credit note" function from the original invoice, or as a negative invoice). Users who receive vendor credit notes will not know how to record them.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a dedicated subsection on invoice matching: explain the matching policy (2-way vs. 3-way), show users how to access Invoice matching details (button on the PO invoice form), explain the matching status indicators, and provide guidance on handling match variances (submit to workflow for exception approval, or investigate and correct).

2. **>> ENHANCEMENT:** Add credit note processing: show users how to create a credit note using the "Create credit note" function from the original invoice (Vendor invoice > Functions > Create credit note). Explain that this preserves the link to the original invoice and simplifies settlement.

3. **>> ENHANCEMENT:** Add a section on charges handling on PO invoices: show how charges from the PO flow to the invoice, and how to add or modify charges on the invoice (Invoice > Maintain charges).

4. **>> ENHANCEMENT:** Document the invoice workflow approval process for PO invoices, including how the yellow workflow bar appears, how to submit, and how approvers process their work items.

---

### 4. Prepayments on the Purchase Order

**What was documented:**
- Navigation to prepayment setup on PO (Accounts payable > Purchase orders > All purchase orders > Purchase > Prepay > Prepayment)
- Entering prepayment information (fixed amount or percentage)
- Saving the prepayment
- Generating a prepayment invoice (Invoice > Generate > Prepayment invoice)
- Entering invoice number, reviewing details, and posting

**Assessment: Partially correct but critically incomplete lifecycle**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Prepayment definition on PO | Good | Both amount and percentage options covered |
| Prepayment invoice generation | Good | Correct navigation path |
| Prepayment invoice posting | Good | Basic posting covered |
| Application of prepayment to final invoice | **>> NEEDS REVIEW <<** | **Completely missing** -- the most critical step |
| Prepayment reversal if PO cancelled | Missing | Not covered |
| Prepayment tracking and balance inquiry | Missing | Not covered |
| Prepayment posting profile | Missing | Not explained |
| Project-linked prepayment handling | Missing | Not covered |

**>> ATTENTION AREA: Prepayment application against final invoice is not documented -- CRITICAL GAP**

> The training manual shows users how to create and post a prepayment invoice, but **stops there**. The critical next step -- applying (settling) the prepayment against the final vendor invoice when it arrives -- is completely absent. In D365, when the final PO invoice is posted, the system can automatically apply the prepayment if properly configured, or the user must manually settle the prepayment. Without this training, users will:
> - Post prepayment invoices
> - Later post the final invoice for the full amount
> - Result in the vendor being overpaid (prepayment + full invoice amount)
> - Leave orphaned prepayment balances on the vendor account
>
> This is the **most critical training gap** in the entire document.

**>> ATTENTION AREA: FRD CRITICAL finding not reflected in training**

> The FRD review (Enhancement #1, CRITICAL severity) identified that the advance payment process should be redesigned to fully use the standard D365 PO Prepayment feature rather than a manual journal approach. The training manual does use the OOB PO Prepayment path (which is positive), but it stops short of the complete lifecycle. If the FRD recommendation was accepted, the training should document the full end-to-end flow including automatic prepayment application on final invoice posting.

**Recommendations:**

1. **>> ENHANCEMENT (CRITICAL):** Add the prepayment application step. After the prepayment invoice is posted and the final PO invoice is received, document:
   - Open the final PO invoice (Accounts payable > Invoices > Pending vendor invoices)
   - On the invoice form, click "Apply prepayment" (or navigate to the Prepayment tab)
   - The system displays the prepayment amount available for this PO
   - Mark the prepayment for application
   - The invoice total is reduced by the prepayment amount
   - Post the final invoice -- the prepayment is automatically settled

2. **>> ENHANCEMENT:** Add prepayment balance tracking: show users how to view outstanding prepayment balances per vendor and per PO (Vendor transactions filtered by Prepayment transaction type, or PO header Prepayment tab).

3. **>> ENHANCEMENT:** Add prepayment reversal/cancellation steps for scenarios where the PO is cancelled after the prepayment invoice is posted.

4. **>> ENHANCEMENT:** Explain the prepayment posting profile requirement: a separate vendor posting profile for prepayments must be configured (Accounts payable > Setup > Vendor posting profiles with a "Prepayment" profile) so that prepayment amounts post to a prepayment liability account rather than the standard AP summary account.

---

### 5. Vendor Payments

**What was documented:**
- Navigation to vendor payment journal (Accounts payable > Payments > Vendor payment journal)
- Journal creation and name selection (Cash, Transfer, Check)
- Opening journal lines
- Line entry: date, vendor account, description, debit amount, currency
- Offset account setup (bank or ledger)
- Method of payment selection
- Financial dimensions for vendor account and offset/bank account
- Settle transactions function (linking invoices to payment)
- Marking invoices for settlement and partial payment amount update
- Validation and posting

**Assessment: Covers manual payment entry but misses the primary D365 payment workflow**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Manual payment journal entry | Good | Clear step-by-step with all key fields |
| Settlement during payment | Good | Settle transactions function well explained |
| Partial payment | Mentioned | Brief mention of updating "amount to settle" |
| Payment proposal | **>> NEEDS REVIEW <<** | **Completely absent** -- this is the primary payment method in D365 |
| Check printing | Missing | Not covered at all |
| Electronic payment file generation | Missing | Not covered |
| Post-dated checks | Missing | Not covered (FRD identifies as common MENA requirement) |
| Cash discount handling | Missing | Not explained |
| Payment approval workflow | Missing | Not documented |
| Centralized/intercompany payment | Missing | Not covered |
| Payment reversal/void | Missing | Not covered |

**>> ATTENTION AREA: Payment Proposal is the primary payment method in D365 and is completely absent**

> In standard D365 AP operations, the **Payment Proposal** (Vendor payment journal > Lines > Payment proposal > Create payment proposal) is the primary method for generating vendor payments. It automatically selects invoices due for payment based on criteria such as due date, vendor group, payment method, or specific vendors. The proposal can be reviewed, modified, and then transferred to the payment journal lines. This is far more efficient than manually entering each payment line, which is the only method taught in this training manual.
>
> For an organization processing more than a handful of vendor payments per cycle, the payment proposal is essential. Teaching only manual entry will result in slow, error-prone payment processing and missed payment deadlines.

**>> ATTENTION AREA: Check printing and management is absent**

> The FRD (AP003) explicitly includes check printing, void, and reissue as standard payment methods. For vendors paid by check, the process of printing checks from the payment journal (Generate payments > Check) is not documented. Users will not know how to print checks, void a check, or reissue a replacement.

**>> ATTENTION AREA: Cash discount handling is not explained**

> The FRD designs for cash discount terms as part of payment processing. When a payment is made within the discount period, D365 can automatically calculate and apply the cash discount, reducing the payment amount. The training manual does not explain this, meaning users may overpay vendors by not taking available discounts, or may be confused when the system applies an unexpected discount.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a dedicated section on Payment Proposal: document the full flow from creating a payment proposal (criteria selection), reviewing the proposal results, modifying selected invoices, and transferring to journal lines. This should be presented as the **primary** payment method, with manual entry as the alternative for ad-hoc payments.

2. **>> ENHANCEMENT:** Add check management: printing checks from Generate payments, voiding checks (Payment tab > Check > Void), and reissuing checks. Include the check register inquiry.

3. **>> ENHANCEMENT:** Add electronic payment file generation: using Generate payments with the correct export format, reviewing the payment file, and submitting to the bank.

4. **>> ENHANCEMENT:** Add cash discount handling: explain how cash discounts are calculated based on payment terms, how the discount appears in the settlement form, and how to take or forfeit a discount.

5. **>> ENHANCEMENT:** Add payment approval workflow: if a workflow is configured on the payment journal, document the submission and approval process.

---

### 6. Settlement and Undo Settlement

**What was documented:**
- Navigation to settle transactions from vendor form (Accounts payable > Vendors > All vendors > Settle transactions)
- Marking payment and invoice lines for settlement
- Posting the settlement
- Undo settlement navigation (from vendor form > Undo settlement)
- Marking and reversing the settlement

**Assessment: Basic settlement mechanics covered, but lacks depth**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Open transaction settlement | Good | Clear steps with mark and post |
| Undo settlement | Good | Reverse process documented |
| Partial settlement | Missing | Not explained how to settle partial amounts |
| Cash discount at settlement | Missing | Not covered |
| Cross-company settlement | Missing | Intercompany settlement not addressed |
| Settlement priority | Missing | Oldest-first vs. manual selection not explained |
| Settle remainder | Missing | How to handle remaining open amounts |

**>> ATTENTION AREA: Partial settlement and cash discount handling at settlement are missing**

> In practice, partial settlements (applying a payment to only part of an invoice) and cash discount settlements (where the payment amount plus discount equals the invoice amount) are common scenarios. The manual only shows the basic mark-and-post flow without addressing these variations. Users encountering partial payments or discount scenarios will not have training guidance.

**Recommendations:**

1. **>> ENHANCEMENT:** Add partial settlement scenario: show how to modify the "Amount to settle" field on a marked transaction to settle less than the full amount, and explain that the remaining balance stays open.

2. **>> ENHANCEMENT:** Add cash discount at settlement: when marking a transaction, explain how the "Cash discount" and "Cash discount amount to take" fields work, and what happens when the payment date is within or outside the discount period.

---

### Reversal of Vendor Transactions

**What was documented:**
- Navigation to vendor transactions (from vendor form > Transactions)
- Selecting a transaction for reversal
- Clicking "Reverse transaction"
- Entering reversal date and confirming

**Assessment: Minimal coverage of an important process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Basic reversal steps | Acceptable | Core mechanics covered |
| Reversal of settled transactions | Missing | Must unsettled before reversing -- not explained |
| Reversal reason codes | Missing | Not mentioned |
| Trace number for reversal tracking | Missing | Not covered |
| Reversal date restrictions | Missing | Period close restrictions not explained |
| Reversal of payment journals | Missing | Specific to payment reversal |

**>> ATTENTION AREA: Reversal prerequisites not explained**

> In D365, a settled transaction cannot be reversed directly -- it must first be unsettled (undo settlement) before reversal is possible. The manual documents both undo settlement and reversal as separate sections but does not explain this dependency. A user attempting to reverse a settled invoice will encounter an error with no guidance on how to resolve it.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a prerequisite note at the beginning of the reversal section: "If the transaction has been settled (linked to a payment), you must first undo the settlement before the transaction can be reversed. See the Settlement and Undo Settlement section."

2. **>> ENHANCEMENT:** Mention reversal reason codes if configured, and explain that the reversal creates a new voucher linked to the original for audit trail purposes.

---

### 7. Exchange Rate Modification

**What was documented:**

**Purchase Order Invoice:**
- Navigation to pending vendor invoices
- Switching "Fixed rate" to Yes under Header > Setup > Currency
- Modifying the exchange rate
- Posting the invoice

**Vendor Payment Journal:**
- Instruction to insert "account type" and "Exchange rate" columns
- Method 1: Two-line approach -- vendor line with settlement, bank line with exchange rate (multiply by 100)
- Method 2: Under General tab, modify currency and exchange rate without multiplication, plus modify LBP exchange rate to balance

**Assessment: Partially correct with a significant risk in the payment journal instructions**

| Aspect | Rating | Comment |
|--------|--------|---------|
| PO invoice exchange rate override | Good | Fixed rate toggle correctly documented |
| Payment journal -- two-line method | **>> NEEDS REVIEW <<** | "Multiply by 100" instruction is non-standard and risky |
| Payment journal -- General tab method | Acceptable | More standard approach |
| When to override exchange rates | Missing | No business context for when this is appropriate |
| Exchange rate type explanation | Missing | Default vs. fixed rate not explained |

**>> ATTENTION AREA: "Multiply exchange rate by 100" instruction is potentially incorrect and dangerous**

> The training manual instructs users to "multiply the exchange rate by 100" when entering the exchange rate on the bank line of a payment journal. This is **not standard D365 behavior**. D365 exchange rates are stored in the system based on the Exchange Rate Type configuration, and the display factor (1, 10, 100, etc.) is set at the currency pair level in the exchange rate table. If the system is configured with a display factor of 100 (common for currencies like LBP where rates are large numbers), the system handles this automatically. Manually multiplying by 100 on top of the system's display factor would result in an exchange rate that is 100x too large, causing massive posting errors.
>
> This instruction suggests either: (a) the system is configured with a non-standard display factor and users need specific guidance, or (b) the instruction is incorrect. Either way, this needs immediate verification and clarification. Incorrect exchange rates on payment journals directly impact bank reconciliation and realized gain/loss calculations.

**Recommendations:**

1. **>> ENHANCEMENT:** Remove or verify the "multiply by 100" instruction. If the exchange rate display factor for the currency pair is already set to 100 in the Exchange rate table (General ledger > Currencies > Exchange rates), then users should enter the rate as displayed by the system -- no manual multiplication is needed. Document the correct approach based on Technica's actual currency configuration.

2. **>> ENHANCEMENT:** Add business context: explain when exchange rate overrides are appropriate (e.g., bank confirmation rate differs from system rate, contractual rate, etc.) and who has authority to override rates.

3. **>> ENHANCEMENT:** Recommend using the General tab method (Method 2) as the standard approach rather than the two-line method, as it is cleaner and less error-prone.

---

### 8. Foreign Currency Revaluation

**What was documented:**
- Navigation to foreign currency revaluation (Accounts payable > Periodic task > Foreign currency revaluation)
- Simulation step before actual processing
- Entering posting date ("Considered date") and exchange rate date ("Date of rate")
- Running the simulation and reviewing the report
- Running the actual revaluation posting
- Reviewing the posted voucher and marking as reviewed

**Assessment: Procedurally correct but lacks critical business context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation | Good | Correct path |
| Simulation before posting | Good | Important control step documented |
| Date parameters | Acceptable | Fields documented but purpose not explained |
| Actual posting | Good | Steps are clear |
| Review and mark as reviewed | Good | Post-processing step included |
| Business purpose explanation | **>> NEEDS REVIEW <<** | No explanation of why revaluation is needed |
| Vendor/currency selection criteria | Missing | Not shown (which vendors, which currencies to include) |
| Revaluation method | Missing | Standard vs. mark-to-market not explained |
| Impact on financial statements | Missing | Not explained |
| Frequency guidance | Missing | When to run (month-end, quarter-end) not stated |

**>> ATTENTION AREA: No explanation of business purpose or impact**

> The training teaches users to click buttons without explaining why. Foreign currency revaluation adjusts the carrying value of open vendor balances denominated in foreign currencies to reflect current exchange rates, impacting unrealized gain/loss on the income statement and the AP balance on the balance sheet. Without understanding the purpose, users may run the revaluation at incorrect times, with incorrect date parameters, or fail to run it when needed (e.g., at month-end close), resulting in inaccurate financial statements.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a brief explanation at the top of this section: "Foreign currency revaluation is a month-end (or period-end) process that adjusts open vendor balances in foreign currencies to reflect current exchange rates. This ensures that the accounts payable balance on the balance sheet and the unrealized gain/loss on the income statement are accurate as of the reporting date."

2. **>> ENHANCEMENT:** Add guidance on selection criteria: which currencies to include, whether to use the revaluation for all vendors or specific groups, and what the "Considered date" and "Date of rate" fields actually control (Considered date = posting date for the revaluation voucher; Date of rate = which date's exchange rate to use for revaluation).

3. **>> ENHANCEMENT:** Add frequency guidance: "This process is typically run as part of the month-end or period-end closing process, after exchange rates have been updated for the period-end date."

---

### 9. Accounts Payable Reports

**What was documented:**
- Vendor Balance Inquiry: real-time vendor balance view (from vendor > Balance)
- Vendor Transactions Inquiry: all financial transactions for a vendor (from vendor > Transactions)
- Vendor Invoices Inquiry: all vendor invoices with status (from vendor > Invoices)
- Vendor Transactions Report: summary over selected period (Accounts payable > Inquiries and reports > Vendor transactions report)
- Vendor Balance List Report: vendor balances with amounts due/overdue (Accounts payable > Inquiries and reports > Vendor reports > Vendor balance list report)
- Vendor Account Statement Report: detailed statement per vendor (Accounts payable > Inquiries and reports > Vendor reports > Account statement report)
- Vendor Invoice Transactions Report: invoice transaction details (Accounts payable > Inquiries and reports > Vendor reports > Invoice > Vendor invoice transactions report)
- Vendor Aging Report: outstanding balances by aging period with setup of aging period definitions

**Assessment: Good coverage of standard reports with minor gaps**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Inquiry forms (balance, transactions, invoices) | Good | Correct navigation with descriptions |
| Standard reports (balance list, account statement, invoice transactions) | Good | Key reports covered |
| Vendor aging report | Good | Includes aging period definition setup |
| Report parameter explanation | **>> NEEDS REVIEW <<** | Screenshots show dialog boxes but parameters are not explained |
| Report export/print options | Missing | Not mentioned |
| Power BI / Analytics workspace | Missing | Modern reporting not covered |
| Vendor prepayment balance report | Missing | Identified in FRD as needed |
| Check register / payment register | Missing | Not covered |

**>> ATTENTION AREA: Aging report configuration needs more detail**

> The aging report section shows the aging period definition setup form but does not explain how to configure the aging periods (e.g., 0-30, 31-60, 61-90, 91-120, 120+), how the aging date basis works (due date vs. transaction date vs. document date), or how to interpret the resulting report. The aging report is one of the most frequently used AP reports for cash flow management and vendor relationship management.

**Recommendations:**

1. **>> ENHANCEMENT:** For each report, add a brief explanation of the key parameters (date range, vendor selection, posting layer, etc.) and what the report output contains. Currently, screenshots of dialog boxes are shown but the parameters are not explained.

2. **>> ENHANCEMENT:** Add coverage of report export options: printing to screen, exporting to Excel, exporting to PDF. These are standard D365 functions that users need for daily reporting.

3. **>> ENHANCEMENT:** Add a section on the Power BI Vendor Analytics workspace (available OOB in D365) which provides interactive dashboards for vendor spend analysis, payment cycle time, and aging visualization.

4. **>> ENHANCEMENT:** Add the check/payment register report: Accounts payable > Inquiries and reports > Payment > Check register. This is essential for organizations using check payments.

---

## General Document Quality

| Quality Aspect | Assessment |
|---------------|------------|
| Structure and organization | Good -- logical flow from configuration to processes to reports |
| Screenshots | Present throughout -- but auto-generated alt text ("AI-generated content may be incorrect") suggests these may need verification against the actual Technica environment |
| Step-by-step instructions | Good -- numbered steps with field-level detail |
| Navigation paths | Good -- provided for all processes |
| Business context / "why" | **Weak** -- manual focuses on "how to click" but rarely explains "why" or "when" |
| Error handling / troubleshooting | **Absent** -- no guidance on common errors or how to resolve them |
| Glossary / terminology | **Absent** -- D365 terminology (voucher, settlement, posting profile, financial dimensions) used without definition |
| Keyboard shortcuts | **Absent** -- no efficiency tips for power users |
| Role-based guidance | **Absent** -- no indication of which steps require which security roles |
| Version/environment reference | **Absent** -- no reference to D365 version or Technica-specific configuration |

**>> ATTENTION AREA: Screenshots may not reflect Technica's configured environment**

> Multiple screenshots carry auto-generated alt text noting "AI-generated content may be incorrect." While this is likely a DOCX metadata artifact, it is important to verify that all screenshots reflect Technica's actual D365 environment (correct legal entity, configured fields, company-specific customizations) rather than generic demo data. Training with screenshots from a different environment causes confusion when users see different field layouts in production.

**Recommendations:**

1. **>> ENHANCEMENT:** Add a brief glossary section defining key D365 AP terms: voucher, settlement, posting profile, financial dimensions, subledger, summary account, matching policy, charges, etc.

2. **>> ENHANCEMENT:** Add a troubleshooting section covering common AP errors: "Voucher is not balanced," "Budget exceeded," "Invoice matching validation failed," "Period is closed," "Vendor is on hold," etc. For each error, explain the cause and resolution.

3. **>> ENHANCEMENT:** Verify all screenshots against Technica's configured environment before final publication.

4. **>> ENHANCEMENT:** Add role-based notes: indicate which processes require AP Clerk, AP Manager, or AP Supervisor security roles.

---

## Cross-Reference with FRD

This section compares the training manual content against the FRD07 Accounts Payable review findings to identify gaps between what was designed and what is being trained.

### FRD Process Coverage in Training Manual

| FRD Process | FRD Scope | Training Coverage | Gap Assessment |
|------------|-----------|-------------------|----------------|
| **AP001 - PO Invoice Processing** | 2-way/3-way matching, price/quantity tolerance, match variance workflow, charge matching, financial dimensions, project PO invoices, credit notes, intercompany | Basic PO invoicing without matching validation, no workflow, no credit notes, no intercompany | **SIGNIFICANT GAP** -- Core AP controls (matching, workflow) not trained |
| **AP002 - Non-PO Invoice Processing** | Invoice register, approval journal, workflow, recurring invoices, prepayment invoices without PO, intercompany allocation | **Vendor Invoice Journal only** -- Invoice Register and Approval Journal 2-step process not covered, recurring invoices absent | **CRITICAL GAP** -- Entire non-PO process from FRD is missing |
| **AP003 - Vendor Payment Processing** | Payment proposal, check/wire/LC, check management, electronic payment, settlement, cash discount, centralized payment, bank reconciliation | Manual payment entry and basic settlement only -- payment proposal, check management, electronic payment, LC, cash discount all missing | **SIGNIFICANT GAP** -- Primary payment workflow not trained |
| **AP004 - Down Payment / Advance Payment** | PO-linked prepayment, percentage/amount, approval workflow, application to final invoice, balance tracking, reversal | Prepayment creation and invoice generation only -- application to final invoice, tracking, reversal all missing | **CRITICAL GAP** -- Prepayment lifecycle incomplete |

### FRD GAP Items -- Training Coverage

| FRD GAP | Description | FRD Recommendation | Covered in Training? |
|---------|-------------|-------------------|---------------------|
| AP001-GAP01 | Email notification to requester on invoice posting | OOB via D365 Alerts or Power Automate | No -- alerts/notifications not covered in training |
| AP001-GAP02 | Custom fields on vendor invoice form | No-code custom fields | No -- custom fields not mentioned |
| AP001-GAP03 | Match failure notification | Reclassify as FIT (workflow notification) | No -- matching not covered at all |
| AP002-GAP01 | Budget notification on non-PO invoices | OOB via Budget Control | No -- budget control not mentioned |
| AP002-GAP02 | PDF attachment to journal | OOB Document Management | No -- document attachment not covered |
| AP002-GAP03 | Auto-create invoice from emailed PDF | D365 Vendor Invoice Capture / AI Builder | No -- not covered |
| AP003-GAP01 | Payment approval by amount threshold | Standard workflow routing | No -- payment workflow not covered |
| AP003-GAP02 | Post-dated check handling | OOB PDC feature in Cash & Bank | No -- PDC not covered |
| AP003-GAP03 | Remittance advice email to vendor | OOB Print Management | No -- not covered |
| AP004-GAP01 | Prepayment linked to PO with auto-application | OOB PO Prepayment feature | **Partially** -- creation covered, application missing |
| AP004-GAP02 | Advance payment balance report | Vendor transactions + Power BI | No -- not covered |
| AP004-GAP03 | Notification to PM on advance payment | Power Automate / D365 Alerts | No -- not covered |

### FRD CRITICAL/HIGH Findings -- Training Impact

| FRD Finding | Severity | Training Impact |
|------------|----------|-----------------|
| Redesign advance payment to use OOB PO Prepayment | **CRITICAL** | Training uses the PO Prepayment path (positive) but does not complete the lifecycle (application to final invoice). **Training must be updated to show the full prepayment flow.** |
| Reclassify AP003 GAPs as FIT (payment thresholds, PDC, remittance email) | **HIGH** | None of these standard features are covered in training. If implemented as OOB configuration, training must be added. |
| Budget control hard-stop on non-PO invoices | **HIGH** | Non-PO invoices are not covered in training at all. Budget control is not mentioned anywhere. |
| Cross-FRD matching policy alignment (2-way vs. 3-way) | **HIGH** | Matching is not covered in training. Once the matching policy is defined, training must explain it to AP users. |
| Non-PO invoice budget control gap | **HIGH** | No training coverage. Users processing non-PO invoices will not understand budget check behavior. |

**>> ATTENTION AREA: Training manual has not been updated to reflect FRD solution design decisions**

> The training manual appears to have been written independently of the FRD, or against an earlier version of the FRD. Multiple processes designed in the FRD (non-PO invoices, payment proposals, invoice matching, post-dated checks, Letter of Credit) are completely absent from the training material. This suggests the training manual needs a significant update to align with the approved solution design before it can be used for user training.

**>> ATTENTION AREA: FRD CRITICAL finding on advance payments is partially reflected but incomplete**

> The FRD's most critical finding was to use the standard PO Prepayment feature instead of manual journals. The training manual does teach the PO Prepayment path (which is the right approach), suggesting alignment with the FRD recommendation. However, the training stops at prepayment invoice generation and does not cover application to the final invoice, which is the most important operational step.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Impact |
|---|------|---------------|----------|--------|
| 1 | Prepayments | Add prepayment application to final invoice -- complete the prepayment lifecycle documentation | **Critical** | Prevents vendor overpayment; completes the core prepayment flow |
| 2 | Non-PO Invoices | Add entire non-PO invoice processing section (Invoice Register, Approval Journal, direct Invoice Journal) | **Critical** | Covers a major daily AP process entirely missing from training |
| 3 | Invoice Matching | Add invoice matching validation section (2-way/3-way, tolerance, match status, exception handling) | **Critical** | Core financial control not currently trained |
| 4 | Payment Proposal | Add Payment Proposal section as the primary payment creation method | **High** | Enables efficient batch payment processing |
| 5 | Check Management | Add check printing, void, and reissue documentation | **High** | Required for check-based payment operations |
| 6 | Workflow Approvals | Add invoice and payment workflow submission, approval, and rejection processes | **High** | Users cannot operate in a workflow-controlled environment without this |
| 7 | Exchange Rate | Verify and correct the "multiply by 100" exchange rate instruction | **High** | Prevents potential 100x posting errors |
| 8 | Credit Notes | Add vendor credit note creation and processing | **High** | Common transaction type with no training coverage |
| 9 | Post-Dated Checks | Add PDC handling if applicable to Technica (likely, given MENA region) | **Medium** | FRD-designed feature with no training |
| 10 | Cash Discounts | Add cash discount handling in payments and settlements | **Medium** | Prevents missed discounts and user confusion |
| 11 | Budget Control | Add budget checking behavior on non-PO invoices | **Medium** | Critical financial control with no training |
| 12 | Journal Selection | Add decision guide for selecting the correct invoice entry method | **Medium** | Prevents incorrect journal usage |
| 13 | Vendor Approval | Add vendor approval workflow documentation | **Medium** | Needed for controlled vendor master management |
| 14 | Withholding Tax | Add withholding tax group assignment on vendor master | **Medium** | Required for MENA jurisdictions |
| 15 | Business Context | Add "why" explanations and business context throughout the manual | **Medium** | Transforms click-by-click guide into meaningful training |
| 16 | Foreign Currency Reval | Add business purpose, timing, and parameter explanations | **Medium** | Enables correct period-end execution |
| 17 | Aging Report | Add aging period configuration and report interpretation guidance | **Medium** | Key cash management report needs better coverage |
| 18 | Charges Usage | Add process-level documentation for applying charges on transactions | **Medium** | Configuration is documented but usage is not |
| 19 | Troubleshooting | Add common AP error messages and resolution steps | **Low** | Reduces support tickets and user frustration |
| 20 | Glossary | Add D365 AP terminology definitions | **Low** | Helps new users understand system concepts |
| 21 | Power BI / Analytics | Add Vendor Analytics workspace and Power BI reporting | **Low** | Modern reporting capabilities ignored |
| 22 | Report Parameters | Add parameter explanations for each report dialog | **Low** | Users currently see dialog boxes without guidance |
| 23 | Role-Based Notes | Add security role requirements per process | **Low** | Clarifies access expectations |
| 24 | Screenshot Verification | Verify all screenshots against Technica's configured environment | **Low** | Ensures training matches production |

---

## Risk Flags

**>> ATTENTION AREA: Incomplete prepayment lifecycle creates overpayment risk**

> The training manual teaches users to create and post prepayment invoices but does not teach them to apply prepayments against final invoices. In a production environment, this will result in vendors being paid the prepayment amount AND the full invoice amount (double payment). This is the **highest-priority training gap** and must be addressed before go-live training.

1. **Prepayment overpayment risk.** Users will create prepayment invoices (trained) and then post full final invoices (trained) without applying the prepayment against the final invoice (not trained). The result is overpayment equal to the prepayment amount on every PO with a prepayment. Given that prepayments are common on Technica's project POs, the financial exposure could be significant.

**>> ATTENTION AREA: Non-PO invoice process gap creates operational paralysis**

> The FRD designs a complete non-PO invoice process (AP002) covering invoice registers, approval journals, and workflow. The training manual does not cover this process at all. Users responsible for processing non-PO invoices (utilities, rent, professional services, etc.) will have no training on how to perform their daily work.

2. **Non-PO invoice processing is untrained.** The training manual covers PO invoices and vendor invoice journals but entirely omits the Invoice Register and Approval Journal 2-step process that the FRD designs for departmental review of non-PO invoices. AP clerks processing non-PO invoices will have no training coverage.

**>> ATTENTION AREA: Exchange rate instruction may cause material posting errors**

> The instruction to "multiply the exchange rate by 100" on payment journal bank lines is non-standard and potentially incorrect depending on Technica's exchange rate configuration. If applied incorrectly, this would cause payment postings at 100x the correct exchange rate, resulting in massive unrealized gain/loss entries and bank reconciliation failures.

3. **Exchange rate multiplication instruction is risky.** The "multiply by 100" guidance for payment journal exchange rates does not align with standard D365 behavior where the display factor is configured at the currency pair level. If the system already accounts for the display factor, manual multiplication would create 100x errors on every foreign currency payment.

**>> ATTENTION AREA: Missing invoice matching training undermines AP financial controls**

> Invoice matching (2-way and 3-way) is one of the primary financial controls in D365 Accounts Payable. The FRD explicitly designs for matching with tolerances and exception workflows. The training manual does not mention matching at all. Without matching training, users may override or ignore match failures, or may not understand why the system prevents them from posting an invoice -- both scenarios undermine the financial control framework.

4. **Invoice matching control bypass risk.** Users untrained on invoice matching may not understand match failure indicators, may not know how to resolve match exceptions, and may seek workarounds (such as changing quantities or prices to force a match) that undermine the control. This creates audit risk and potential for incorrect vendor payments.

**>> ATTENTION AREA: Training manual appears misaligned with FRD solution design**

> Multiple FRD-designed processes are absent from the training manual, suggesting the training was written before the FRD was finalized or without reference to the FRD. Before go-live training, the manual must be comprehensively updated to reflect all processes in the approved FRD solution design, including all configuration decisions made during implementation.

5. **Training-FRD alignment gap.** The training manual covers approximately 60% of the processes designed in the FRD. The remaining 40% (non-PO invoices, payment proposals, invoice matching, check management, electronic payments, post-dated checks, Letter of Credit, budget control, recurring invoices) represents a significant training gap that will impact user readiness at go-live. A dedicated training update sprint is recommended before UAT.

---

*This review is based on D365 F&O Accounts Payable best practices as of 2026. All recommendations should be validated against Technica's specific D365 version, licensed modules, and implementation-specific configuration decisions. The training manual should be updated after FRD finalization and configuration completion to ensure full alignment with the deployed solution.*
