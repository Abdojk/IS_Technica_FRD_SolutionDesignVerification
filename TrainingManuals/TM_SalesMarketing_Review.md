# Training Manual - Sales and Marketing - Solution Design Verification Report

**Document:** Technica - User Training Manual - Sales and Marketing V2.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 Sales (CRM) / D365 F&O - Sales and Marketing

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Sales Process Coverage | Training manual covers only the Project-sales pipeline (Prospect > Lead > Opportunity > Project Quotation) but omits the standard Sales Order lifecycle (Sales Quotation > Sales Order > Picking/Packing > Invoice) despite the Purpose section promising coverage of "sales quotations, orders, and returns" | [3.1 - Missing Sales Order Lifecycle](#31-missing-standard-sales-order-lifecycle-quotation--order--delivery--invoice) |
| 2 | **CRITICAL** | Sales Returns | Sales return order process is entirely absent -- the Purpose section explicitly promises coverage of "returns" but no content exists | [3.2 - Missing Sales Returns](#32-missing-sales-return-order-process) |
| 3 | **CRITICAL** | FRD-to-Training Gap: Trips & Missions (SP001) | FRD01 defines a custom Trips & Missions entity with approval workflow and F&O Travel Requisition integration -- training manual contains zero coverage of this process | [4.1 - Trips & Missions](#41-sp001---trips--missions--completely-absent) |
| 4 | **CRITICAL** | FRD-to-Training Gap: DFI Entity (SP003) | FRD01 defines a custom DFI (Data Fulfillment Information) entity with versioning, approval, and F&O integration trigger -- training manual contains zero coverage | [4.3 - DFI Entity](#43-sp003---dfi-entity--completely-absent) |
| 5 | **HIGH** | FRD-to-Training Gap: Lead Approval | FRD01 specifies a mandatory approval workflow before lead qualification with button restrictions -- training manual shows only basic lead creation with no mention of approval | [4.2 - Lead Approval](#42-sp002---lead-approval-workflow--absent) |
| 6 | **HIGH** | Opportunity Stage Customization | Training manual references custom fields (Offer Deadline, Offer Type, Opportunity Type, Buyers tab with 4-row limit, Win/Lost Reason, Closing Feedback) but provides no explanation that these are Technica-specific customizations requiring special handling | [3.5 - Opportunity Customizations](#35-opportunity-management--custom-fields-undocumented-as-customizations) |
| 7 | **HIGH** | Customer Creation -- Number Sequence | Training manual instructs users to "Click New" but does not explain whether the customer account number is auto-generated or manually entered; number sequence configuration is not addressed | [3.3 - Customer Creation](#33-customer-creation--missing-critical-setup-context) |
| 8 | **HIGH** | Project Quotation Process | Training manual covers creating a project quotation from the opportunity but stops at "notification sent to tendering team" -- does not cover quotation line entry, WBS templates, approval workflow, send-to-customer, or confirm/cancel | [3.6 - Project Quotation](#36-project-quotation--incomplete-end-to-end-coverage) |
| 9 | **MEDIUM** | Prospect Entity Accuracy | Training manual uses F&O navigation path "Sales and marketing > Relationships > Prospects > All prospects" which is valid, but the Prospect entity in D365 F&O is limited and rarely used in modern implementations -- FRD01 uses D365 CE (CRM) for the sales pipeline, creating a platform mismatch | [3.4 - Prospects](#34-prospect-management--platform-mismatch-with-frd) |
| 10 | **MEDIUM** | Lead Creation Navigation | Navigation "From the prospect screen > Sell > New > Lead" follows F&O Sales and Marketing module conventions, but FRD01 places lead management in D365 CE -- manual should clarify which system is the system of record | [3.4 - Prospects](#34-prospect-management--platform-mismatch-with-frd) |
| 11 | **MEDIUM** | Reporting Section Completeness | Reports section lists standard OOB reports but does not reference any custom reports, dashboards, or Power BI content that would reflect Technica-specific KPIs identified in the FRD | [3.7 - Reports](#37-sales-and-marketing-reports--incomplete-and-lacks-custom-content) |
| 12 | **MEDIUM** | Sales Tax Group Configuration | Training manual instructs users to select a "sales tax group" during customer creation without explaining what determines the correct group or referencing the tax configuration -- users may select incorrectly | [3.3 - Customer Creation](#33-customer-creation--missing-critical-setup-context) |
| 13 | **MEDIUM** | Credit & Collections | "Specify the credit rating & credit limit of the customer of it exists" -- training manual does not explain what happens when the credit limit is reached (order hold, workflow escalation) or who is authorized to set/change credit limits | [3.3 - Customer Creation](#33-customer-creation--missing-critical-setup-context) |
| 14 | **LOW** | Document Structure | No version history table, no glossary of terms, no role-based guidance (which sections apply to which user roles), no prerequisites section | [3.8 - Document Quality](#38-document-structure-and-quality) |
| 15 | **LOW** | Error Handling / Troubleshooting | Training manual provides only "happy path" instructions -- no guidance on common errors, validation failures, or what to do when a step fails | [3.8 - Document Quality](#38-document-structure-and-quality) |
| 16 | **LOW** | Security Roles | No mention of security roles or access permissions required for each process -- users will not know what to do if they encounter "Access Denied" | [3.8 - Document Quality](#38-document-structure-and-quality) |
| 17 | **LOW** | Typo / Quality | Paragraph 72: "of it exists" should be "if it exists"; Paragraph 167: "After ending one or multiple contacts" should be "After adding one or multiple contacts" | [3.8 - Document Quality](#38-document-structure-and-quality) |

**Totals:** 4 CRITICAL | 4 HIGH | 5 MEDIUM | 4 LOW

---

## Executive Summary

The Technica User Training Manual for Sales and Marketing V2.0 covers three broad functional areas: (1) Customer master data creation in D365 F&O, (2) the project-oriented sales pipeline (Prospect > Lead > Opportunity > Project Quotation) using the F&O Sales and Marketing module, and (3) standard sales and marketing reports. The document contains 44 embedded screenshots and follows a step-by-step instructional format.

**Overall assessment: The manual has significant structural gaps that must be addressed before it is suitable for end-user training.**

The most critical finding is a **scope mismatch**: the Purpose section explicitly states the manual covers "sales quotations, orders, and returns," yet the document contains **no coverage of the standard Sales Order lifecycle** (Sales Quotation creation, Sales Order processing, Picking, Packing Slip, Invoice posting) and **no coverage of Sales Returns** (Return Material Authorization, return order creation, item receipt, credit note). These are core D365 F&O Sales and Marketing processes that users will need to execute daily.

The second critical finding is a **substantial gap between the FRD and the training manual**. FRD01 (Sales) defines three custom GAP processes -- Trips & Missions (SP001), Lead Qualification Approval (SP002), and the DFI Entity with F&O Integration (SP003). The training manual documents **none** of these customizations. Since these are Technica-specific processes that do not exist out-of-the-box, users cannot learn them from any other source. The training manual is the only place these custom workflows should be documented for end users.

The content that is present (Customer creation, Prospect/Lead/Opportunity pipeline, Reports) is generally accurate in terms of D365 F&O navigation paths and field references, though several areas lack sufficient context for a training document (e.g., no explanation of when to use specific values, no error handling guidance, no security role prerequisites).

---

## Process-by-Process Review

### 3.1 Missing Standard Sales Order Lifecycle (Quotation > Order > Delivery > Invoice)

**>> ATTENTION AREA: The core transactional sales process is entirely absent**

> The Purpose section states: *"It explains how to create and manage sales quotations, orders, and returns."* However, the training manual contains no content on:
>
> - **Sales Quotation creation** (Sales and marketing > Sales quotations > All quotations)
> - **Sales Quotation confirmation/sending** to customer
> - **Sales Order creation** (from quotation, manual, or intercompany)
> - **Sales Order line entry** (item selection, pricing, delivery dates, warehousing)
> - **Order Confirmation** posting
> - **Picking List** generation and processing
> - **Packing Slip** posting (delivery confirmation)
> - **Sales Invoice** posting
> - **Sales Order holds** (credit hold, manual hold)

This is the most frequently executed process in the Sales and Marketing module. Without this training content, users will be unable to process day-to-day sales transactions.

**>> ENHANCEMENT:** Add a complete "Sales Order Processing" section covering the end-to-end lifecycle:
1. Creating a sales quotation (manual and from opportunity)
2. Quotation approval and sending
3. Converting quotation to sales order (or creating sales order directly)
4. Sales order line configuration (items, quantities, prices, delivery schedule, warehouses)
5. Posting the order confirmation
6. Picking list / picking route processing
7. Packing slip posting
8. Invoice posting
9. Credit note scenarios
10. Sales order inquiry and status tracking

**>> NEEDS REVIEW <<** -- This gap must be resolved before the training manual can be considered complete.

---

### 3.2 Missing Sales Return Order Process

**>> ATTENTION AREA: Sales returns are promised in scope but not documented**

> The Purpose section explicitly includes "returns" in scope. The D365 F&O Sales Return Order process includes:
>
> - **Return Material Authorization (RMA)** number generation
> - **Return order creation** (Sales and marketing > Sales orders > Return orders)
> - **Return reason codes** configuration and selection
> - **Item arrival and receipt** processing
> - **Disposition codes** (Credit, Replace, Scrap)
> - **Credit note** generation
> - **Return-to-vendor** scenarios (if applicable)

**>> ENHANCEMENT:** Add a dedicated "Sales Returns" section covering:
1. When and why to create a return order
2. RMA number assignment
3. Return reason code selection
4. Receiving returned goods (item arrival journal or direct receipt)
5. Disposition code assignment and its impact (credit only, replace, scrap)
6. Posting the credit note / return invoice
7. Linking returns to original sales orders

**>> NEEDS REVIEW <<** -- Must be added to fulfill the documented scope commitment.

---

### 3.3 Customer Creation -- Missing Critical Setup Context

**What is documented:**
The training manual provides a reasonable step-by-step guide for customer creation in D365 F&O, covering: record type, name, customer group, currency, terms of payment, sales tax group, country, contacts, bank accounts, addresses (with purpose-driven addressing), contact information, miscellaneous details, credit & collections, sales demographics, payment defaults, and invoice/delivery defaults.

**Assessment: Procedurally adequate but lacks business context and guardrails**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | `Sales and marketing > Customers > All customers > Click new` is valid |
| Field coverage | Good | Most mandatory and important fields are covered |
| Address management | Good | Purpose-driven addressing is correctly explained |
| Number sequence handling | **>> NEEDS REVIEW <<** | Not mentioned whether account numbers are auto or manual |
| Customer group guidance | **>> NEEDS REVIEW <<** | No guidance on which group to select or what groups mean |
| Tax group guidance | **>> NEEDS REVIEW <<** | No guidance on correct tax group selection |
| Credit limit handling | **>> NEEDS REVIEW <<** | No explanation of downstream impact |

**>> ENHANCEMENT:** Add the following context to the Customer Creation section:
1. **Number sequence**: Explain whether the customer account number is automatically generated or manually entered. If manual, specify the naming convention Technica uses.
2. **Customer group**: Provide a reference table of Technica's customer groups and when to use each one (e.g., "Domestic", "Export", "Intercompany"). Customer group drives default posting profiles and financial dimensions.
3. **Sales tax group**: Explain the logic for selecting the correct tax group. Incorrect tax group selection will result in wrong tax calculations on all transactions for that customer.
4. **Credit limit**: Explain what happens operationally when a customer's credit limit is exceeded (e.g., sales order hold, escalation workflow). Identify who has authority to modify credit limits.
5. **Invoice account**: The manual mentions "Customer's invoice account" but does not explain the parent/child billing relationship. The FRD01 review notes this is a Technica requirement (billing to parent company, delivery to child company). This must be explicitly documented.

**>> ATTENTION AREA: The FRD01 review specifically flags that "billing goes to parent company while delivery goes to child company" -- this parent-child invoice account relationship is not explained in the training manual.**

---

### 3.4 Prospect Management -- Platform Mismatch with FRD

**What is documented:**
The training manual covers prospect creation in D365 F&O at `Sales and marketing > Relationships > Prospects > All prospects`, including name, type, currency, country, account source, nature, status, activities, contacts, and industry demographics.

**Assessment: Valid F&O content but potentially conflicting with FRD architecture**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct for F&O | Valid F&O Sales and Marketing path |
| Field coverage | Adequate | Key fields covered |
| Platform alignment with FRD | **>> NEEDS REVIEW <<** | FRD01 uses D365 CE (CRM) for the sales pipeline |

**>> ATTENTION AREA: Platform mismatch between training manual and FRD**

> FRD01 (Sales) specifies that the sales pipeline (Prospects, Leads, Opportunities) is managed in **D365 Sales (CE/CRM)**, with integration to F&O for quotations and financial processes. However, this training manual documents the Prospect/Lead/Opportunity flow using **D365 F&O Sales and Marketing module** navigation paths.
>
> This creates a fundamental question: **Which system is the system of record for the sales pipeline?**
>
> - If D365 CE is the system of record (as FRD01 states), the training manual should document CRM screens, not F&O screens
> - If D365 F&O is the system of record (as the training manual implies), the FRD architecture needs to be reconciled
> - If both systems are used (CE for some users, F&O for others), this must be explicitly clarified

**>> ENHANCEMENT:** Resolve the platform alignment:
1. Confirm with the project team whether the Prospect > Lead > Opportunity flow is executed in D365 CE or D365 F&O
2. If D365 CE: Rewrite sections 2.2 through 2.5 to reference CRM navigation, forms, and terminology
3. If D365 F&O: Update the FRD01 to reflect that F&O (not CE) is the system of record for the sales pipeline
4. If both: Add a clear "System Landscape" section explaining which users work in which system and how data flows between them

**>> NEEDS REVIEW <<** -- This architectural ambiguity must be resolved to avoid user confusion during training.

---

### 3.5 Opportunity Management -- Custom Fields Undocumented as Customizations

**What is documented:**
The training manual covers opportunity creation from a lead, with fields: Subject, Estimated closing date, Sales process, Offer deadline, Offer type, Opportunity type, Primary contact, Buyers (with 4-row limit), Competitors, Win/Lost Reason, Closing Feedback, Activities, Attachments, Stage management, and Status changes.

**Assessment: Good coverage of the process flow, but critical context is missing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Opportunity creation from lead | Correct | Standard conversion path |
| Standard fields (Subject, Est. Close Date) | Correct | OOB fields |
| Custom fields (Offer Deadline, Offer Type, Opportunity Type) | **>> NEEDS REVIEW <<** | Not identified as custom |
| Buyers tab (4-row limit) | **>> NEEDS REVIEW <<** | Custom entity/subgrid -- not standard OOB |
| Competitors tab | Partially standard | OOB supports competitors, but the specific fields listed are custom |
| Win/Lost Reason tab | **>> NEEDS REVIEW <<** | Custom fields beyond standard OOB |
| Closing Feedback tab | **>> NEEDS REVIEW <<** | Entirely custom |
| Stage management with mandatory tabs | **>> NEEDS REVIEW <<** | Custom business rule |

**>> ATTENTION AREA: Users need to understand which fields are Technica customizations**

> Several fields and tabs documented in the Opportunity section are **not standard D365 out-of-the-box fields**. These include:
> - **Offer Deadline, Offer Type, Opportunity Type** -- custom fields on the opportunity header
> - **Buyers tab** with structured columns (buyer influence, type, support to offer, buyer profile) and a **4-row limit** -- this is a custom subgrid, not OOB
> - **Win/Lost Reason tab** with detailed competitive analysis fields (delivery, analysis of buyers, technical solution rating, quality of equipment, lead time, price level compared to Technica, technology evaluation) -- custom section
> - **Closing Feedback tab** (case study flag, project awarded to, feedback, turning point, improvement areas) -- custom section
> - **Mandatory Buyers and Competitors tabs** before stage progression -- custom business rule/plugin
>
> If users encounter issues with these custom components (e.g., validation errors, missing fields after an update), they need to know these are customizations that require support escalation, not standard Microsoft functionality.

**>> ENHANCEMENT:** Add a note or callout box identifying Technica-specific customizations, and for each custom field, provide:
1. What the field means in Technica's business context
2. What values are expected (dropdowns, free text, numeric ranges)
3. Who is responsible for maintaining the lookup values (e.g., Offer Types)
4. What happens if the field is left blank (is it mandatory? does it block stage progression?)

---

### 3.6 Project Quotation -- Incomplete End-to-End Coverage

**What is documented:**
The training manual covers: (1) Creating a project quotation from the opportunity via `Opportunity > New > Project Quotation`, and (2) noting that a notification is sent to the tendering team.

**Assessment: Critically incomplete -- only the trigger step is documented**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Creation from opportunity | Correct | Valid navigation |
| Notification to tendering team | Noted | Custom notification -- not OOB |
| Quotation header entry | **Missing** | Not documented |
| WBS template import | **Missing** | FRD02 identifies this as a key capability |
| Quotation line entry | **Missing** | Not documented |
| Cost/Price calculation | **Missing** | FRD02 identifies costing version usage |
| Approval workflow | **Missing** | FRD02 identifies approval before sending |
| Send to customer | **Missing** | Not documented |
| Confirm/Cancel/Lost | **Missing** | Not documented |
| Won > Project creation | **Missing** | FRD02 identifies auto-confirmation pipeline |

**>> ATTENTION AREA: The project quotation process is a critical Sales Support workflow documented extensively in FRD02 but barely touched in the training manual**

> FRD02 (Sales Support) describes a comprehensive project quotation process including WBS template import, CAD system integration, cost calculation using dedicated costing versions, approval workflow, and the Won > Confirm > Create Project pipeline. The training manual provides only two sentences on this entire process.

**>> ENHANCEMENT:** Expand the Project Quotation section to include:
1. Quotation header fields (customer, project group, dates, financial dimensions)
2. WBS template import and modification
3. Adding items, hours, and expenses to WBS activities
4. Running cost/price calculation
5. Reviewing quotation totals
6. Submitting for approval (workflow steps)
7. Sending quotation to customer
8. Handling customer responses (confirm, revise, cancel, lost)
9. Won scenario: auto-confirmation and project creation flow

**>> NEEDS REVIEW <<** -- The tendering team and sales engineers need detailed quotation processing guidance.

---

### 3.7 Sales and Marketing Reports -- Incomplete and Lacks Custom Content

**What is documented:**
The training manual lists six report categories with navigation paths:
1. **Customers** -- `Sales and marketing > Inquiries and reports > Customers > Customer report`
2. **Customer base data** -- `Sales and marketing > Inquiries and reports > Customers > Customer base data`
3. **Sales orders per customer** -- `Sales and marketing > Inquiries and reports > Sales order reports > sales orders per customer`
4. **Open sales order lines by ship date** -- `Sales and marketing > Inquiries and reports > Sales order reports > Sales order lines`
5. **Sales and probability performance** -- BI reports (Revenue by customer/product/period/location, Customer profitability, Profitability analysis)
6. **Customer/Item statistics** -- `Sales and marketing > Inquiries and reports > Customer statistics > Customer/item statistics`

**Assessment: Standard OOB reports are correctly listed, but coverage is incomplete**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation paths | Correct | Valid F&O paths |
| Report descriptions | Adequate | Brief but informative |
| Sales order reports | **>> NEEDS REVIEW <<** | Lists reports for sales orders, but the manual doesn't teach sales order creation |
| BI reports | Noted | No navigation path or access instructions provided |
| Custom/Technica-specific reports | **Missing** | No custom reports documented |
| Opportunity/Lead pipeline reports | **Missing** | No pipeline reporting documented |
| Report parameters/filters | **Missing** | No guidance on how to filter or customize report output |

**>> ENHANCEMENT:** Improve the Reports section:
1. Add instructions on how to set report parameters and filters (date ranges, customer groups, etc.)
2. Add export-to-Excel guidance for ad-hoc analysis
3. Document how to access the BI reports (Power BI workspace navigation, embedding in D365)
4. Add opportunity/lead pipeline reports (critical for sales management)
5. Add any Technica-specific custom reports or workspaces
6. Remove or clarify the sales order reports -- currently the manual references sales order reports but does not teach users how to create sales orders

**>> ATTENTION AREA: The FRD01 review flagged that "Technica will specify later" for reporting scope. The training manual should document whatever has been agreed upon since the FRD was written.**

---

### 3.8 Document Structure and Quality

**Assessment: The document lacks several structural elements expected in an enterprise training manual**

| Element | Status | Comment |
|---------|--------|---------|
| Purpose and Scope | Present | But scope statement does not match actual content |
| Table of Contents | Present | Referenced but appears to be a placeholder (no page numbers visible in extracted content) |
| Version history | **Missing** | No revision log showing what changed between V1.0 and V2.0 |
| Glossary of terms | **Missing** | Technica-specific terms (DFI, Offer Type, Prospect Nature) are undefined |
| Role-based guidance | **Missing** | No indication of which roles use which sections |
| Prerequisites | **Missing** | No section on system access, security roles, or setup required before following the manual |
| Screenshots | Present (44 images) | Significant visual support included |
| Error handling | **Missing** | No troubleshooting guidance for common errors |
| Cross-references | **Missing** | No links to related training manuals or reference documents |
| Typographical errors | Present | "of it exists" (should be "if it exists"), "After ending" (should be "After adding") |

**>> ENHANCEMENT:** Add the following structural elements:
1. **Version history table** with date, author, and change summary for each version
2. **Glossary** defining Technica-specific terms and D365 terminology
3. **Role matrix** mapping each section to the security roles that need it (e.g., Sales Manager, Sales Engineer, Sales Administrator)
4. **Prerequisites section** listing required system access, security roles, and any setup that must be completed before users can follow the procedures
5. **Troubleshooting appendix** with common errors and resolutions
6. **Cross-reference section** linking to related manuals (e.g., Accounts Receivable training manual for invoice follow-up, Project Accounting for quotation costing details)

---

## Cross-Reference with FRD

This section compares the training manual content against the findings and requirements documented in FRD01 (Sales) and FRD02 (Sales Support) to identify gaps where the training material does not cover designed functionality.

### 4.1 SP001 - Trips & Missions -- COMPLETELY ABSENT

| FRD Requirement | Training Manual Coverage | Gap Severity |
|----------------|------------------------|-------------|
| Custom Trips & Missions entity in D365 CE | Not mentioned | **CRITICAL** |
| Trip request creation with details | Not mentioned | **CRITICAL** |
| Approval workflow via email (approve/reject) | Not mentioned | **CRITICAL** |
| HR notification on approval | Not mentioned | **CRITICAL** |
| F&O Travel Requisition draft creation on approval | Not mentioned | **CRITICAL** |
| Rejection handling (cancel tasks/appointments) | Not mentioned | **CRITICAL** |
| Resubmission after rejection | Not mentioned | **CRITICAL** |

**>> ATTENTION AREA: The Trips & Missions process is a custom entity defined in FRD01 with no OOB equivalent. Users cannot learn this from any other source. This MUST be documented in the training manual.**

> The FRD01 review identified this as a GAP requiring custom development. The review also recommended considering Power Apps as an alternative to a full CE custom entity, and flagged the two-system data entry problem (CRM approval + F&O Travel Requisition). Regardless of which implementation approach is chosen, the training manual must document the end-user experience for creating, submitting, and tracking trip/mission requests.

**>> ENHANCEMENT:** Add a dedicated "Trips & Missions" training section covering:
1. When and why to create a trip/mission request
2. How to access the Trips & Missions form (whether in CE, Power Apps, or F&O)
3. Required fields and how to fill them
4. How to submit for approval
5. What happens after approval (HR notification, Travel Requisition in F&O)
6. What happens after rejection (how to view rejection reason, how to resubmit)
7. How to track trip status
8. How to complete Travel Requisition details in F&O (if applicable)

**>> NEEDS REVIEW <<**

---

### 4.2 SP002 - Lead Approval Workflow -- ABSENT

| FRD Requirement | Training Manual Coverage | Gap Severity |
|----------------|------------------------|-------------|
| Custom approval cycle before lead qualification | Not mentioned | **HIGH** |
| Sales Manager approval via email notification | Not mentioned | **HIGH** |
| Approved = qualify into opportunity | Partially covered (lead-to-opportunity conversion is shown) | **MEDIUM** |
| Rejected = disqualify | Not mentioned | **HIGH** |
| Restricted qualify/disqualify buttons based on approval status | Not mentioned | **HIGH** |

**>> ATTENTION AREA: The training manual shows a simple lead-to-opportunity conversion path that does not reflect the actual configured process**

> FRD01 mandates that leads cannot be qualified without Sales Manager approval. The training manual shows `From the lead > generate > opportunity` with no mention of an approval gate. If this approval workflow has been implemented, users following the training manual will encounter restricted buttons and approval requirements they were not trained on. If the approval workflow was descoped, this should be documented.

> The FRD01 review recommended replacing the mandatory approval with a BPF stage + oversight dashboards, or at minimum using Power Automate Approvals instead of custom status fields. Whatever approach was implemented, it must be reflected in the training.

**>> ENHANCEMENT:** Update the Lead section to include:
1. The approval requirement before qualification (if implemented)
2. How to submit a lead for approval
3. How the Sales Manager approves/rejects (email, Teams, or in-app)
4. What happens visually on the lead form during the approval process
5. How to handle a rejected lead (resubmission or disqualification)
6. Which buttons are available at each approval stage

---

### 4.3 SP003 - DFI Entity -- COMPLETELY ABSENT

| FRD Requirement | Training Manual Coverage | Gap Severity |
|----------------|------------------------|-------------|
| Custom DFI entity linked to opportunity | Not mentioned | **CRITICAL** |
| DFI sections (Strategic, Customer Requirements, Technical Data, Handlift) | Not mentioned | **CRITICAL** |
| DFI versioning (duplicate & modify approach) | Not mentioned | **CRITICAL** |
| DFI approval workflow | Not mentioned | **CRITICAL** |
| DFI approval triggers F&O integration | Not mentioned | **CRITICAL** |
| F&O quotation generation from approved DFI | Not mentioned | **CRITICAL** |

**>> ATTENTION AREA: The DFI is the most complex customization in the Sales module and has zero training coverage**

> The FRD01 review identified the DFI entity as the "most complex area" with concerns about versioning fragility and undefined integration architecture. FRD02 (Sales Support) also references the DFI as the trigger for the entire quotation preparation process. This entity bridges the gap between CRM sales activities and F&O project quotation creation. Without DFI training, users will not be able to:
> - Capture customer requirements in structured format
> - Create and manage DFI versions
> - Submit DFI for approval
> - Trigger the F&O integration pipeline
>
> The FRD01 review recommended replacing version duplication with audit history or SharePoint versioning, and defining the CRM-to-F&O integration architecture. Whatever approach was implemented must be thoroughly documented.

**>> ENHANCEMENT:** Add a comprehensive "DFI (Data Fulfillment Information)" training section covering:
1. What a DFI is and its role in the sales process
2. How to create a DFI from an opportunity
3. How to fill in each section (Strategic, Customer Requirements, Technical Data, Handlift)
4. How versioning works (creating new versions, comparing versions)
5. How to submit for approval
6. What happens on approval (F&O integration trigger)
7. How to track DFI status
8. Common DFI scenarios and examples

**>> NEEDS REVIEW <<**

---

### 4.4 Cross-FRD Dependencies Not Addressed

| Dependency | FRD Source | Training Manual Status |
|-----------|-----------|----------------------|
| CRM-to-F&O Account/Contact sync | FRD01 / FRD02 | Not mentioned -- users don't know if customer data syncs automatically |
| DFI flow from CRM to F&O | FRD01 > FRD02 | Not mentioned |
| Opportunity Won > Quotation Confirmed > Project Created | FRD02 | Partially mentioned (quotation creation) but pipeline not explained |
| CAD system integration with WBS | FRD02 | Not mentioned |
| Pre-sales time tracking on internal project | FRD02 | Not mentioned |

**>> ENHANCEMENT:** Add a "System Landscape" or "How the Systems Work Together" introduction section that explains:
1. Which processes happen in D365 CE (CRM) vs. D365 F&O
2. How data flows between the two systems (automatic sync, manual handoff, or triggered integration)
3. What the user needs to do in each system and when
4. Key integration points (DFI approval > F&O quotation, Opportunity Won > Project creation)

---

## Summary of Recommendations

| # | Priority | Area | Recommendation | Impact |
|---|----------|------|---------------|--------|
| 1 | **CRITICAL** | Sales Orders | Add complete Sales Order lifecycle training (Quotation > Order > Pick > Pack > Invoice) | Users cannot process daily sales transactions without this |
| 2 | **CRITICAL** | Sales Returns | Add Sales Return Order process training (RMA, receipt, disposition, credit note) | Scope commitment is unmet; users cannot process returns |
| 3 | **CRITICAL** | Trips & Missions | Add SP001 Trips & Missions training section aligned with FRD01 design | Custom process with no other documentation source |
| 4 | **CRITICAL** | DFI Entity | Add SP003 DFI entity training section aligned with FRD01/FRD02 design | Most complex customization has zero user guidance |
| 5 | **HIGH** | Lead Approval | Update Lead section to include SP002 approval workflow per FRD01 | Users will encounter restricted buttons they were not trained on |
| 6 | **HIGH** | Opportunity Customizations | Document all Technica-specific custom fields and business rules on the opportunity form | Users need to understand custom vs. standard behavior |
| 7 | **HIGH** | Customer Number Sequence | Add number sequence explanation and customer group selection guidance | Prevents incorrect customer setup |
| 8 | **HIGH** | Project Quotation | Expand to cover full quotation lifecycle (WBS, costing, approval, send, confirm) per FRD02 | Tendering team and Sales Support have no process guidance |
| 9 | **MEDIUM** | Platform Alignment | Resolve whether training should reference D365 CE or F&O for the sales pipeline | Eliminates user confusion about which system to use |
| 10 | **MEDIUM** | Reporting | Expand reports section with parameters, BI access, custom reports, and pipeline reports | Users need actionable reporting guidance |
| 11 | **MEDIUM** | Tax Configuration | Add guidance on correct sales tax group selection during customer creation | Prevents tax calculation errors |
| 12 | **MEDIUM** | Credit Management | Document credit limit handling and downstream impacts | Prevents confusion when orders are held |
| 13 | **MEDIUM** | System Landscape | Add a section explaining CRM vs. F&O usage and data flow between systems | Essential context for dual-system environment |
| 14 | **LOW** | Document Structure | Add version history, glossary, role matrix, prerequisites, troubleshooting | Standard training manual quality requirements |
| 15 | **LOW** | Quality | Fix typos ("of it exists" > "if it exists", "After ending" > "After adding") | Professional document quality |
| 16 | **LOW** | Security | Add security role prerequisites for each process area | Users need to know required access levels |

---

## Risk Flags

### Risk 1: Users Will Be Unable to Process Core Sales Transactions

**>> ATTENTION AREA: The training manual does not cover the standard sales order lifecycle**

> This is the highest operational risk. If users are trained only on prospect/lead/opportunity management and customer creation, they will lack the knowledge to execute the most common daily transaction: processing a sales order from creation through invoicing. This will result in:
> - Immediate support escalation volume at go-live
> - Delayed order processing and revenue recognition
> - Potential data quality issues from untrained users experimenting with the system

### Risk 2: Custom Processes Have Zero Training Coverage

**>> ATTENTION AREA: Three custom GAP processes from FRD01 are not documented**

> Trips & Missions (SP001), Lead Approval (SP002), and DFI (SP003) are all customizations that do not exist in standard D365. There are no Microsoft Learn articles, no community guides, and no other documentation sources for these processes. The training manual is the **only** vehicle for teaching users how to use these custom features. Without training content, these customizations -- which represent significant development investment -- will be underutilized or misused.

### Risk 3: Platform Confusion Between CRM and F&O

**>> ATTENTION AREA: The training manual documents F&O navigation for processes that the FRD places in D365 CE**

> The FRD01 architecture positions D365 Sales (CE/CRM) as the system of record for the sales pipeline, with F&O handling financial processes. The training manual documents the Prospect > Lead > Opportunity flow using F&O Sales and Marketing module paths. This inconsistency will cause confusion during training delivery:
> - Trainers will not know which screens to demonstrate
> - Users will not know which application to log into
> - Screenshots in the manual may not match what users see
>
> This must be resolved before training sessions are conducted.

### Risk 4: Scope Statement Creates False Expectations

**>> ATTENTION AREA: The Purpose section promises more than the manual delivers**

> The stated scope includes "sales quotations, orders, and returns" but the manual does not cover standard sales quotation processing, sales orders, or returns. Users (and auditors) will expect these topics to be covered based on the scope statement. Either the content must be added or the scope statement must be corrected to reflect actual coverage.

### Risk 5: FRD Review Findings Not Reflected in Training

**>> ATTENTION AREA: Architectural decisions recommended in the FRD reviews have not been incorporated**

> The FRD01 review raised critical findings about DFI versioning (recommending audit history over record duplication), integration architecture (undefined CRM-to-F&O mechanism), and lead approval friction. The FRD02 review raised findings about the DFI hand-off mechanism, CAD integration, and pre-sales time tracking. None of these findings or their resolutions are reflected in the training manual. If design decisions were made in response to these findings, the training material must be updated to reflect the final implemented design.

---

*This review is based on D365 F&O Sales and Marketing module capabilities and D365 Sales (CE) best practices as of 2026. Cross-references are made against FRD01 (Sales) V2.0 and FRD02 (Sales Support) V2.0 review findings. Recommendations aim to close training gaps, ensure FRD-to-training alignment, and prepare end users for successful system adoption.*
