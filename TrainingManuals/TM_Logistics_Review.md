# Training Manual - Logistics - Solution Design Verification Report

**Document:** Technica - User Training Manual - Logistics V3.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Logistics (Landed Cost, Transportation Management)

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Transportation Management | TM module is referenced in scope but NO training content exists for transportation tenders, load planning, route assignment, or carrier selection -- entire FRD process LOG003 is untrained | [5.0 RFQ for Export vs. FRD LOG003](#50-request-for-quotation-for-export-processes) |
| 2 | **CRITICAL** | Packing (LOG004) | FRD defines a packing process (BOM explode, box assignment, packing slip per box) -- zero coverage in the training manual | [Missing Content - Packing](#missing-process---log004-packing) |
| 3 | **CRITICAL** | Direct Delivery (LOG002) | FRD defines direct-to-customer delivery via Landed Cost (Option 1 & Option 2) -- zero coverage in the training manual | [Missing Content - Direct Delivery](#missing-process---log002-direct-delivery) |
| 4 | **HIGH** | 3.1 Voyages | Goods-in-transit warehouse concept is not explained -- users will not understand why inventory moves to a transit warehouse before physical receipt | [3.1 Voyages](#31-voyages) |
| 5 | **HIGH** | 3.1 Voyages | Matching policy prerequisite is mentioned ("turned off for vendor") but no explanation of why, what it controls, or how to configure it | [3.1 Voyages](#31-voyages) |
| 6 | **HIGH** | 3.2 Invoice Journal | Variance handling between estimated and actual costs is not explained -- users will not know what happens when actual cost differs from estimate or how clearing accounts reconcile | [3.2 Invoice Journal](#32-invoice-journal) |
| 7 | **HIGH** | 5.0 RFQ for Export | RFQ process is documented via Project Management module (Item Tasks) instead of Transportation Management tenders as specified in FRD LOG003 -- fundamental module mismatch | [5.0 RFQ for Export](#50-request-for-quotation-for-export-processes) |
| 8 | **HIGH** | 5.0 RFQ for Export | Purchase Pool field mentioned for classifying export vs. import on auto-created PO but no guidance on what values to use, where Purchase Pools are configured, or reporting implications | [5.0 RFQ for Export](#50-request-for-quotation-for-export-processes) |
| 9 | **MEDIUM** | 2.7 Cost Type Codes | Detailed field-level table is provided (17 fields) but no worked example showing a realistic cost type code setup for Technica's actual cost categories (freight, insurance, customs duty, handling) | [2.7 Cost Type Codes](#27-cost-type-codes) |
| 10 | **MEDIUM** | 2.8 Auto Cost | Apportionment methods are listed but no guidance on which method Technica should use for which cost type -- users left to guess | [2.8 Auto Cost](#28-auto-cost) |
| 11 | **MEDIUM** | 2.11 Tracking Control Center | Three create types (Blank, Lead Time, Status Update) are described at a technical level but no end-to-end example shows how they work together during a real voyage | [2.11 Tracking Control Center](#211-tracking-control-center) |
| 12 | **MEDIUM** | 2.5 Legs | Instruction states "in case the leg is an interim step other than sailing between the ports, define the ports as an" -- sentence is incomplete and meaning is lost | [2.5 Legs](#25-legs) |
| 13 | **MEDIUM** | 4.0 Reports | Reports section is extremely thin -- three reports listed with one-line descriptions and no screenshots, filter guidance, or interpretation instructions | [4.0 Landed Cost Reports](#40-landed-cost-reports) |
| 14 | **MEDIUM** | General | No security role or access permission guidance anywhere in the document -- users do not know which roles grant access to which screens | [General Observations](#general-observations) |
| 15 | **LOW** | 2.4 Vessels | Navigation path is incorrect: "Landed cost > Shipping information setup > Vessels" -- correct path is "Landed cost > Setup > Delivery information setup > Vessels" | [2.4 Vessels](#24-vessels-optional) |
| 16 | **LOW** | 2.10 Journey Template | Instruction says "Press on new, to create a new BOM" -- this is a copy-paste error; BOM is unrelated to journey templates | [2.10 Journey Template](#210-journey-template) |
| 17 | **LOW** | General | No version history, change log, or document control section despite being V3.0 -- users cannot identify what changed from V2.0 | [General Observations](#general-observations) |
| 18 | **LOW** | General | Heading numbering is inconsistent -- "Introduction" has no number, "Purpose" has no number, then "1.2 Scope" implies a missing "1.1", and sections jump to "2.0" | [General Observations](#general-observations) |

**Totals:** 3 CRITICAL | 5 HIGH | 6 MEDIUM | 4 LOW

---

## Executive Summary

This training manual covers Technica's logistics processes using the **Landed Cost** module and a portion of the **RFQ process** for export shipments. The document provides reasonable step-by-step guidance for Landed Cost configuration (Sections 2.1-2.11), voyage creation and invoicing (Sections 3.1-3.2), basic reporting (Section 4), and an RFQ process for export vendor selection (Section 5).

However, the training manual has **critical coverage gaps** when cross-referenced against the FRD06 Logistics requirements. Three entire FRD processes -- **LOG002 (Direct Delivery)**, **LOG003 (Outbound Shipments via Transportation Management)**, and **LOG004 (Packing)** -- have no training content whatsoever. The RFQ process in Section 5 is documented through the Procurement/Project Management module rather than through the Transportation Management tenders specified in the FRD, representing a fundamental module mismatch that could confuse users during go-live.

The Landed Cost configuration sections (2.1-2.11) are generally accurate in their D365 navigation paths and field descriptions, but lack practical worked examples tied to Technica's specific business scenarios. The operational process sections (3.1-3.2) cover the basics of voyage creation and cost accrual but omit critical concepts such as goods-in-transit warehousing, cost variance handling, and the clearing account reconciliation lifecycle.

Overall assessment: **The manual is a solid starting point for Landed Cost basics but is significantly incomplete relative to the FRD scope. It requires substantial additions before it is ready for user training.**

---

## Process-by-Process Review

### Section 1 - Introduction

**What is covered:**
- Purpose statement identifying Landed Cost and Transportation Management as the two module areas
- Scope statement referencing key functionalities

**Assessment: Adequate but incomplete**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Purpose clarity | Good | Clearly states the two module areas |
| Scope definition | **>> NEEDS REVIEW <<** | Claims to cover Transportation Management but Section 5 uses Procurement RFQ instead |
| Audience identification | Missing | No mention of target user roles |
| Prerequisites | Missing | No mention of required D365 knowledge or prior training |

**>> ATTENTION AREA: Scope statement claims Transportation Management coverage that does not exist**

> The Purpose section states the manual covers "how to do transportation tender for outbound shipments" via the "Transportation management module." However, the actual content in Section 5 uses the Procurement RFQ process through Project Management, not the Transportation Management tender process. This mismatch between stated scope and actual content will confuse users.

**>> ENHANCEMENT:** Add a "Target Audience" subsection identifying the logistics roles this manual is intended for (e.g., Logistics Coordinator, Shipping Clerk, Logistics Manager). Add a "Prerequisites" subsection listing any prior training required (e.g., D365 navigation basics, Procurement training manual).

**>> ENHANCEMENT:** The heading numbering is inconsistent. "Introduction" has no section number, "Purpose" has no number, then "1.2 Scope" implies a missing "1.1 Purpose." Standardize numbering across the entire document (e.g., 1.0 Introduction, 1.1 Purpose, 1.2 Scope).

---

### Section 2 - Landed Cost Configuration

This is the most comprehensive section of the manual, covering 11 configuration areas. Each subsection follows a consistent pattern: brief description, navigation path, and step-by-step field instructions.

#### 2.1 Shipping Container Types

**Assessment: Adequate**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Setup > Containers setup > Shipping container types |
| Field descriptions | Basic but sufficient | Code, description, volume, weight |
| Screenshots | Present (referenced by document images) | Supports visual learners |

No major issues. This is a simple setup form and the instructions are proportionate.

---

#### 2.2 Shipping Containers (Optional)

**Assessment: Adequate**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Setup > Containers setup > Shipping containers |
| Field descriptions | Sufficient | Code, description, container type link |
| "Optional" designation | Helpful | Correctly marked as optional since containers can be created during voyage |

---

#### 2.3 Shipping Ports

**Assessment: Adequate**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Setup > Delivery information setup > Shipping ports |
| Field descriptions | Sufficient | Code and name |

**>> ENHANCEMENT:** Consider adding a note about port naming conventions (e.g., "Use ISO port codes like AEJEA for Jebel Ali") to ensure consistency across users.

---

#### 2.4 Vessels (Optional)

**Assessment: Minor error detected**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | **>> NEEDS REVIEW <<** | Path shown as "Landed cost > Shipping information setup > Vessels" |
| Field descriptions | Sufficient | Code, description, mode of delivery |
| "Optional" designation | Correct | Vessels can be created during voyage |

**>> ATTENTION AREA: Incorrect navigation path**

> The manual states the navigation path is "Landed cost > Shipping information setup > Vessels." The correct D365 path is **"Landed cost > Setup > Delivery information setup > Vessels."** The "Setup" level is missing and "Shipping information" should be "Delivery information." This will prevent users from finding the screen.

---

#### 2.5 Legs

**Assessment: Contains incomplete instruction**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Multi-leg journeys setup > Legs |
| Field descriptions | **>> NEEDS REVIEW <<** | Incomplete sentence |
| From/To port explanation | Adequate | Basic field mapping |

**>> ATTENTION AREA: Incomplete sentence in leg setup instructions**

> The manual states: "In case the leg is an interim step other than sailing between the ports, define the ports as an." The sentence ends abruptly. The intended instruction likely should read: "...define the ports as **an activity port** (or **the same port for both from and to**)." This incomplete instruction will leave users unable to configure interim legs correctly.

**>> ENHANCEMENT:** Add guidance on how legs connect sequentially -- the "to port" of one leg should match the "from port" of the next leg in a journey template. This is a common configuration mistake.

---

#### 2.6 Activities

**Assessment: Adequate but brief**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Multi-leg journeys setup > Activities |
| Field descriptions | Minimal | Code, name, optional shipping company link |

**>> ENHANCEMENT:** Activities are a critical component of voyage tracking (they drive the Tracking Control Center). The manual should list the recommended activities for Technica's business (e.g., "Departed," "In Transit," "Customs Clearance," "Arrived at Port," "Delivered to Warehouse") rather than leaving this entirely to user interpretation.

---

#### 2.7 Cost Type Codes

**Assessment: Thorough field reference but lacks practical context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Costing setup > Cost type codes |
| Field descriptions | Excellent | 17-field detailed table with clear descriptions |
| Practical examples | Missing | No Technica-specific cost type code examples |
| Standard vs. Moving Average variance | Well-explained | Detailed variance scenarios included |

**>> ENHANCEMENT:** The 17-field table is comprehensive as a reference but overwhelming for initial training. Add a "Quick Start" example showing the setup of 2-3 cost type codes typical for Technica (e.g., Ocean Freight with Item debit/Vendor credit, Customs Duty with Item debit/Ledger credit, Insurance with Item debit/Vendor credit). This gives users a concrete pattern to follow.

**>> ATTENTION AREA: Clearing account concept needs more context**

> The clearing account field is described as "Select the clearing account for the cost type. Specify a separate clearing account per cost type to help with reconciliation." While technically correct, users need to understand the lifecycle: estimated costs post to the clearing account at voyage creation, and actual costs clear against it when the vendor invoice is matched. Without this context, the clearing account purpose is opaque to non-accountants.

---

#### 2.8 Auto Cost

**Assessment: Complete field listing but lacks decision guidance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Costing setup > Auto cost |
| Cost area options | Listed | Voyage, item, shipping container, folio, PO |
| Account table filter | Well-explained | Table, Group, All options |
| Apportionment methods | All 7 listed | Percent, Quantity, Amount, Volume, Weight, Volumetric, Measurement |
| Category options | All 4 listed | Fixed, Pieces, Percent, Rate |

**>> ENHANCEMENT:** The six apportionment methods and four category options are listed but without guidance on when to use each. For a training manual, add a decision table:

> | Cost Type | Recommended Apportionment | Recommended Category | Rationale |
> |-----------|--------------------------|---------------------|-----------|
> | Ocean Freight | Volume or Weight | Fixed | Freight typically varies by container size/weight |
> | Insurance | Percent | Percent | Insurance is typically a % of goods value |
> | Customs Duty | Amount | Percent | Duty is typically a % of goods value |
> | Handling | Quantity | Pieces | Handling charges are often per-piece |

This is the kind of practical guidance that differentiates a training manual from a help article.

**>> ATTENTION AREA: Auto cost is a key FRD recommendation that is under-explained**

> The FRD06 review specifically flagged auto-costing rules as an enhancement to reduce manual charge entry on voyages (Enhancement #2, MEDIUM). The training manual covers the auto cost setup screen but does not explain the operational benefit -- that properly configured auto costs will **automatically populate estimated charges when a voyage is created**, eliminating the need for users to manually enter estimated costs for each voyage. This operational context is essential for user adoption.

---

#### 2.9 Voyage Statuses

**Assessment: Adequate**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Setup > Voyage statuses |
| Status actions | Well-covered | Modify, Delete, Mandatory, Parent hierarchy |
| Cost area dimension | Noted | Voyage, item, container, folio, PO |

**>> ENHANCEMENT:** List Technica's planned voyage statuses (e.g., Created, Confirmed, In Transit, At Port, Customs Clearance, Delivered, Closed) with their intended Modify/Delete/Mandatory settings. A blank setup screen provides no training value -- users need to see the target state.

---

#### 2.10 Journey Template

**Assessment: Contains a copy-paste error**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Multi-leg journeys setup > Journey templates |
| Template structure | Well-explained | Legs, sequence, port mapping |
| Journey from/to port checkboxes | Helpful | Important detail included |

**>> ATTENTION AREA: Copy-paste error -- "BOM" reference**

> The manual states: "Press on new, to create a new BOM, & provide it with a description." This is clearly a copy-paste error from a manufacturing/BOM context. It should read: "Press on new, to create a new **journey template**, & provide it with a description." This error appears directly before the correct instruction ("Press on new, to create a new journey template"), suggesting a duplicated/unedited line.

---

#### 2.11 Tracking Control Center

**Assessment: Technically accurate but conceptually dense without examples**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Landed cost > Setup > Delivery information setup > Tracking control center |
| Three create types | All covered | Blank, Lead Time, Status Update |
| Source/Target field mapping | Explained | For Blank create type |
| Lead time specification | Explained | For Lead Time create type |
| Status update triggers | Explained | For Status Update create type |

**>> ENHANCEMENT:** The Tracking Control Center is one of the most powerful but confusing configurations in Landed Cost. The manual describes the three create types correctly but does not show how they interact. Add an end-to-end example:

> **Example:** For a Shanghai-to-Jebel Ali voyage with legs "Shanghai Port" -> "Ocean Transit" -> "Jebel Ali Port":
> - **Lead Time rule:** Leg = Ocean Transit, Activity = Sailing, Lead Time = 21 days
> - **Status Update rule:** When Leg = Jebel Ali Port, Activity = Arrived, update voyage status to "At Destination Port"
> - **Blank rule:** When Leg = Jebel Ali Port, Activity = Arrived, update PO confirmed delivery date from voyage end date

This single example would make the abstract field descriptions concrete and actionable.

---

### 3.1 Voyages

**What is covered:**
- Voyage creation from a purchase order
- Prerequisites (matching policy, delivery terms)
- Voyage editor and purchase order selection
- Staging list and container assignment
- Cost inquiry screen
- Tracking inquiry and status updates
- Multi-PO voyage creation from All Voyages
- Invoice posting from voyage

**Assessment: Core flow is covered but critical concepts are missing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Voyage creation from PO | Good | Step-by-step with screenshots |
| Voyage editor workflow | Good | Filter, select, stage, containerize |
| Multi-PO voyage | Covered | Alternative path via All Voyages |
| Cost inquiry | Covered | Estimated vs. actual display |
| Tracking inquiry | Covered | Lead times and status updates |
| Goods-in-transit warehouse | **Missing** | Not mentioned at all |
| Matching policy prerequisite | **>> NEEDS REVIEW <<** | Mentioned but not explained |
| Delivery terms prerequisite | Partially covered | Goods-in-transit checkbox noted |
| PO invoice posting | **>> NEEDS REVIEW <<** | Mentioned but not contextualized |

**>> ATTENTION AREA: Goods-in-transit warehouse is completely missing**

> The FRD06 review identifies goods-in-transit warehousing as a standard OOB feature being used by Technica (LOG001 -- "In-Transit warehouse transfer when goods ship"). This is a fundamental concept in Landed Cost: when a voyage is confirmed, inventory ownership transfers to Technica and goods move to a virtual "in-transit" warehouse. The training manual never mentions this concept, the in-transit warehouse setup, or how users should expect inventory to appear in this warehouse. Users will be confused when they see inventory in a warehouse they did not physically receive goods at.

**>> ATTENTION AREA: Matching policy instruction is unexplained**

> The manual states: "Before creating a voyage from the purchase order make sure of the following: Matching policy is turned off for the vendor." This is presented as a blind prerequisite with no explanation. Users need to understand:
> 1. **What** the matching policy controls (3-way match between PO, receipt, and invoice)
> 2. **Why** it must be turned off for Landed Cost vendors (because costs are allocated via the voyage, not matched line-by-line)
> 3. **Where** to configure it (Vendor master > Invoice and delivery tab, or Accounts payable parameters)
> 4. **What happens** if they forget (invoices may be held in workflow)

**>> ENHANCEMENT:** The PO invoice posting step ("Click on post invoice from the purchase order section on the manage fast tab") appears after containerization but before the goods have been received. The manual should clarify the business scenario: in Landed Cost with goods-in-transit, the PO invoice can be posted while goods are in transit because ownership has already transferred. Without this context, users from a traditional procurement background will question why they are posting invoices for goods they have not physically received.

**>> ENHANCEMENT:** Add a section on voyage receipt -- the physical receipt of goods at the destination warehouse. The manual covers voyage creation and tracking but stops before the actual receipt process (Landed cost > Voyages > All voyages > select voyage > Receive). This is a critical operational step.

---

### 3.2 Invoice Journal

**What is covered:**
- Vendor invoice journal creation
- Vendor account selection (shipping company)
- Linking voyage costs via Functions > Voyage costs
- Allocating actual costs per cost type code
- Posting the invoice journal

**Assessment: Basic flow covered but critical financial concepts missing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Journal creation | Good | Step-by-step |
| Vendor account guidance | Helpful | "Should be the shipping company vendor account" |
| Voyage cost linkage | Covered | Functions > Voyage costs dropdown |
| Cost allocation | Basic | Estimated vs. actual per cost type |
| Variance handling | **Missing** | No explanation of what happens when actual != estimated |
| Clearing account reconciliation | **Missing** | No explanation of how clearing accounts settle |
| Partial cost accrual | **Missing** | No guidance on handling when not all costs are known |

**>> ATTENTION AREA: Cost variance lifecycle is not explained**

> The FRD06 review notes that the cost type code setup includes clearing accounts, standard cost variance accounts, and moving average variance accounts. The training manual covers the setup fields (Section 2.7) and the invoice journal process (Section 3.2) but never connects them. Users need to understand:
> 1. At voyage creation, **estimated costs** post to the clearing account
> 2. When the vendor invoice journal is posted with **actual costs**, the clearing account is relieved
> 3. Any **difference** (variance) between estimated and actual posts to the variance account
> 4. The variance account must be reviewed and reconciled periodically
>
> Without this lifecycle explanation, users will post invoice journals without understanding the financial impact, and the finance team will face unexplained clearing account balances.

**>> ENHANCEMENT:** Add guidance on partial invoicing scenarios. In reality, not all voyage costs arrive simultaneously -- freight may be invoiced before customs duty. The manual should explain how to handle multiple invoice journals for the same voyage and how to track which cost types have been accrued vs. which remain as estimates.

---

### 4.0 Landed Cost Reports

**What is covered:**
- Voyage Lines report (inquiry)
- Voyage Costing by Individual Cost (report)
- Voyage Costing by Reporting Category (report)

**Assessment: Severely underdeveloped section**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Report identification | Three reports listed | Correct navigation paths |
| Report descriptions | One-line each | Too brief for training |
| Screenshot examples | Not present for reports | Users cannot see expected output |
| Filter guidance | Missing | No guidance on date ranges, voyage selection, etc. |
| Interpretation guidance | Missing | No explanation of what to look for in the reports |
| Export/print instructions | Missing | No guidance on output options |

**>> ATTENTION AREA: Reports section provides no actionable training value**

> The reports section lists three reports with minimal descriptions. For each report, users need:
> 1. When to use this report (business scenario)
> 2. How to filter it (which parameters matter)
> 3. How to interpret the results (what columns mean, what to look for)
> 4. Sample output (screenshot of a realistic report)
>
> The FRD06 review specifically recommended verifying that standard "Voyage costing" reports meet Technica's needs (Enhancement in LOG001). The training manual should demonstrate these reports with Technica-relevant data to confirm they meet reporting requirements.

**>> ENHANCEMENT:** Consider adding the "All shipping containers" inquiry view as a fourth report -- it is mentioned in the FRD as a key tracking tool for container-level visibility.

---

### 5.0 Request for Quotation for Export Processes

**What is covered:**
- RFQ creation from Project Management module (Item Tasks)
- Vendor invitation and bid submission
- Vendor reply management and comparison
- Bid acceptance and auto-PO creation
- Purchase Pool classification for export/import

**Assessment: Functional process but fundamentally misaligned with FRD**

| Aspect | Rating | Comment |
|--------|--------|---------|
| RFQ creation from project | Step-by-step | Clear instructions with screenshots |
| Vendor bid management | Well-covered | Register, compare, accept workflow |
| Bid comparison screen | Covered | Visibility benefit explained |
| Auto-PO creation | Noted | With purchase pool classification |
| Module alignment with FRD | **>> NEEDS REVIEW <<** | FRD specifies TM tenders, manual uses Procurement RFQ |

**>> ATTENTION AREA: Fundamental module mismatch with FRD LOG003**

> **This is the most significant finding in this review.** The FRD06 (LOG003 - Outbound Shipments) specifies that outbound shipping should use **Transportation Management (TM) tenders** for carrier rate comparison. The TM tender process is a purpose-built feature within the Transportation Management module that:
> - Creates rate requests tied to loads and routes
> - Compares carrier rates within the TM rate engine
> - Integrates with load planning and shipment execution
> - Provides freight audit and reconciliation
>
> Instead, the training manual documents an **RFQ process through the Procurement module** accessed via Project Management (Item Tasks). While this achieves a similar outcome (comparing vendor bids), it:
> - Does not integrate with TM loads, routes, or shipments
> - Does not leverage the TM rate engine or freight billing
> - Creates a purchase order rather than a freight order
> - Cannot be used for freight audit reconciliation
>
> **This discrepancy must be resolved.** Either:
> 1. The FRD should be updated to reflect that Technica will use Procurement RFQs instead of TM tenders (and the reasons documented), OR
> 2. The training manual should be rewritten to use the TM tender process as specified in the FRD
>
> If Option 1 is chosen, the scope statement in Section 1 must be corrected to remove the reference to "Transportation management module."

**>> ENHANCEMENT:** Regardless of which module is used, the manual should document:
- How the RFQ/tender result connects to the actual shipment execution
- How freight costs from the winning bid flow into financial reporting
- How to handle scenarios where the actual freight cost differs from the bid

**>> ATTENTION AREA: Purchase Pool classification is undocumented**

> The manual states: "Make sure to fill in the purchase pool in the purchase order header to classify if the purchase order is for export or import." This instruction references a classification mechanism without providing:
> - What Purchase Pool values are available (e.g., "EXPORT," "IMPORT")
> - Where Purchase Pools are configured (Procurement > Setup > Purchase pools)
> - What downstream reporting or processing depends on this classification
> - What happens if the field is left blank

---

## Missing Process - LOG002 Direct Delivery

**>> NEEDS REVIEW <<**

The FRD06 defines a **Direct Delivery** process (LOG002) with two options:
- **Option 1:** Sales order (Item Requirement) with Direct Delivery flag, linked to PO and voyage, with auto-delivery on PO receipt
- **Option 2:** Customer site as a separate warehouse, with PO and voyage receiving at the site warehouse

The FRD review recommended Option 1 as the preferred approach and flagged warehouse proliferation risks with Option 2.

**The training manual contains zero content for this process.** Users involved in direct-to-customer deliveries will have no guidance on:
- How to set up a direct delivery link between sales order and purchase order
- How to create a voyage for direct delivery shipments
- How inventory flows differ from standard inbound shipments
- How project consumption is triggered on receipt

**>> ENHANCEMENT:** A new section (e.g., "3.3 Direct Delivery Voyages") must be added covering the chosen option with step-by-step instructions, clearly noting how it differs from the standard voyage process in Section 3.1.

---

## Missing Process - LOG004 Packing

**>> NEEDS REVIEW <<**

The FRD06 defines a **Packing** process (LOG004) involving:
- BOM explosion to view all parts requiring packing
- Box number/name assignment to BOM lines
- Packing slip generation showing contents per box

The FRD review noted this could be addressed using the standard D365 **WMS packing station** feature and flagged the risk of building a custom solution that duplicates OOB capability (Enhancement #5, MEDIUM).

**The training manual contains zero content for this process.** Users responsible for packing outbound shipments will have no guidance on:
- How to access the packing station or equivalent feature
- How to assign items to boxes/containers
- How to generate box-level packing slips
- How to handle partial packing or repacking scenarios

**>> ENHANCEMENT:** A new section (e.g., "3.4 Packing and Container Assignment") must be added, ideally using the standard WMS packing station feature as recommended in the FRD review.

---

## Missing Process - Outbound Shipment Execution (LOG003)

**>> NEEDS REVIEW <<**

Beyond the RFQ/tender module mismatch, the FRD06 defines a complete outbound shipment execution flow (LOG003) that includes:
- **Load creation** from item requirements (project WBS)
- **Route assignment** on load (country/port-based)
- **Load release** and carrier selection
- **Commercial invoice** (no accounting impact) for shipping documents
- **Packing slip / delivery note** posting

None of these steps are covered in the training manual. Section 5 covers only the vendor selection (RFQ) portion, not the actual shipment execution.

**>> ENHANCEMENT:** Even if the TM tender process is replaced by a Procurement RFQ (per Section 5), the remaining outbound shipment steps (load creation, commercial invoice, packing slip posting) still need training documentation. These are core operational activities that logistics users will perform daily.

---

## Cross-Reference with FRD

The table below maps each FRD06 process to its training manual coverage:

| FRD Process | FRD Description | TM Section | Coverage Assessment |
|-------------|----------------|------------|-------------------|
| **LOG000 - Setup** | Journey templates, legs, lead times, tracking control center, modes of delivery | Sections 2.1-2.11 | **Covered** -- Good coverage of all configuration entities. Missing recommended activity and status value lists. |
| **LOG001 - Inbound Shipments** | Voyage creation, container tracking, charge allocation, in-transit warehouse, delivery date tracking, goods receipt | Sections 3.1-3.2 | **Partially Covered** -- Voyage creation and cost accrual are covered. Goods-in-transit warehouse, physical receipt, and variance handling are missing. |
| **LOG002 - Direct Delivery** | Direct-to-customer delivery via PO-SO link or site warehouse | None | **NOT COVERED** -- Entire process is absent from the training manual. |
| **LOG003 - Outbound Shipments** | Load planning, route assignment, TM tenders, carrier selection, commercial invoice, packing slip | Section 5 (partial, wrong module) | **Critically Misaligned** -- RFQ is documented via Procurement module instead of TM tenders. Load planning, route assignment, commercial invoice, and packing slip are not covered. |
| **LOG004 - Packing** | BOM explode, box assignment, packing slip per box | None | **NOT COVERED** -- Entire process is absent from the training manual. |

### FRD Enhancement Cross-Reference

The FRD06 review identified 7 enhancements. The table below shows whether the training manual accounts for them:

| FRD Enhancement | Severity | TM Alignment |
|----------------|----------|-------------|
| #1 - Commercial invoice must link to project accounting (FRD04) | HIGH | **Not addressed** -- Commercial invoice is not mentioned in the training manual |
| #2 - Auto-costing rules to reduce manual charge entry | MEDIUM | **Partially addressed** -- Auto cost setup is documented (Section 2.8) but the operational benefit of auto-population on voyage creation is not explained |
| #3 - Prefer Option 1 for direct delivery | MEDIUM | **Not addressed** -- Direct delivery is not in the training manual |
| #4 - BOM version/phantom line instead of manual BOM removal | MEDIUM | **Not addressed** -- BOM handling for direct delivery is not in the training manual |
| #5 - Verify WMS packing station before custom solution | MEDIUM | **Not addressed** -- Packing is not in the training manual |
| #6 - Power Automate for document received notification | LOW | **Not addressed** -- No mention of email notifications for document receipt |
| #7 - Power BI for average lead time calculation | LOW | **Not addressed** -- No mention of lead time analytics |

---

## General Observations

### Strengths

1. **Consistent structure for configuration sections (2.1-2.11):** Each configuration entity follows a predictable pattern -- description, navigation path, step-by-step instructions. This is good training material design.

2. **Abundant screenshots (61 images):** The document contains 61 embedded images, which is excellent for a visual step-by-step guide. Screenshots significantly reduce user confusion during training.

3. **Correct navigation paths (with one exception):** Nearly all D365 navigation paths are accurate and current, indicating the manual was written by someone with hands-on system access.

4. **Cost type code field table is excellent:** The 17-field table in Section 2.7 is the most detailed section in the document and provides genuine reference value, including variance account scenarios with worked calculations.

5. **RFQ process is well-documented end-to-end:** Despite the module mismatch concern, the RFQ process in Section 5 is the best-documented operational process in the manual, with clear step-by-step instructions from creation through bid comparison to PO generation.

### Weaknesses

1. **No security/role guidance:** The manual never mentions which D365 security roles are required to access the described screens. Users with insufficient permissions will encounter errors that the manual does not prepare them for.

2. **No error handling or troubleshooting:** The manual assumes every step succeeds. There is no guidance on common errors (e.g., "Why can't I create a voyage from this PO?" "Why are my auto costs not populating?") or how to resolve them.

3. **No glossary of terms:** Landed Cost introduces specialized terminology (folio, voyage, leg, activity, apportionment, clearing account) that may be unfamiliar to users. A glossary would significantly improve accessibility.

4. **No document version control:** Despite being labeled "V3.0," there is no change log or version history table showing what changed from V1.0 or V2.0. This makes it impossible for users or reviewers to identify what was updated.

5. **No end-to-end scenario:** The manual covers individual setup screens and individual process steps but never walks through a complete business scenario from PO creation through voyage, tracking, receipt, and cost reconciliation. An end-to-end walkthrough would tie all sections together.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | LOG003 / Section 5 | Resolve module mismatch: either rewrite Section 5 to use TM tenders or formally document the decision to use Procurement RFQs and update the FRD | Critical |
| 2 | LOG002 | Add complete training section for Direct Delivery process as specified in FRD | Critical |
| 3 | LOG004 | Add complete training section for Packing process as specified in FRD | Critical |
| 4 | LOG003 | Add training content for outbound shipment execution (load creation, route assignment, commercial invoice, packing slip posting) | Critical |
| 5 | Section 3.1 | Add goods-in-transit warehouse explanation -- what it is, how inventory moves there, and how physical receipt works | High |
| 6 | Section 3.2 | Add cost variance lifecycle explanation -- estimated vs. actual, clearing account reconciliation, variance posting | High |
| 7 | Section 3.1 | Explain matching policy prerequisite (what, why, where, consequence of not doing it) | High |
| 8 | Section 5 | Document Purchase Pool configuration and classification values | High |
| 9 | Section 3.1 | Add voyage receipt process (physical goods receipt at destination warehouse) | High |
| 10 | Section 2.8 | Add decision table for apportionment method and category selection by cost type | Medium |
| 11 | Section 2.11 | Add end-to-end Tracking Control Center example with all three create types | Medium |
| 12 | Section 2.7 | Add quick-start example with 2-3 realistic Technica cost type codes | Medium |
| 13 | Section 4.0 | Expand reports section with filter guidance, interpretation notes, and sample output | Medium |
| 14 | General | Add security role requirements for each major process | Medium |
| 15 | General | Add end-to-end business scenario walkthrough (PO to cost reconciliation) | Medium |
| 16 | Section 2.4 | Correct navigation path from "Shipping information setup" to "Delivery information setup" | Low |
| 17 | Section 2.10 | Remove copy-paste error referencing "BOM" in journey template instructions | Low |
| 18 | General | Add version history / change log for V3.0 | Low |
| 19 | General | Standardize heading numbering (1.0, 1.1, 1.2, etc.) | Low |
| 20 | General | Add glossary of Landed Cost terminology | Low |

---

## Risk Flags

1. **Training Readiness Risk (CRITICAL):** Three of five FRD processes (LOG002, LOG003, LOG004) have no training content. If user training is delivered using this manual as-is, users responsible for direct delivery, outbound shipments, and packing will have zero guidance. This is a **go-live readiness risk** that must be addressed before UAT.

2. **Module Mismatch Risk (CRITICAL):** The RFQ process documented in Section 5 uses a different D365 module than what the FRD specifies. If the system is configured for TM tenders but users are trained on Procurement RFQs (or vice versa), training will not match the production system. This mismatch must be resolved at the design level, not the training level.

3. **Financial Understanding Risk (HIGH):** The manual does not explain the financial impact of Landed Cost operations -- goods-in-transit accounting, clearing account mechanics, or cost variance handling. Users who follow the manual steps without understanding the financial context may post transactions incorrectly, leading to inventory valuation errors and general ledger discrepancies that are difficult to unwind.

4. **Cross-FRD Dependency (HIGH):** The FRD06 review flagged that the commercial invoice must link to project accounting (FRD04). The training manual does not mention commercial invoices at all. If this cross-module process is implemented, training material must be created that spans both the logistics and project accounting manuals.

5. **Incomplete Prerequisite Documentation (MEDIUM):** The matching policy and delivery terms prerequisites in Section 3.1 are stated as requirements but not explained. Users who encounter system blocks due to incorrect prerequisites will not be able to self-diagnose or resolve the issue, increasing support ticket volume during go-live.

---

*This review is based on D365 F&O Landed Cost and Transportation Management best practices as of 2026. The training manual was cross-referenced against the FRD06 Logistics Solution Design Verification Report to identify coverage gaps and alignment issues.*
