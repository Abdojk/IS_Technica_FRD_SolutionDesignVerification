# Training Manual - Inventory Management - Solution Design Verification Report

**Document:** Technica - User Training Manual - Inventory Management V4.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Inventory Management, Warehouse Management (WMS)

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Product Creation | Product creation process conflates Product Masters with Distinct Products -- training instructs users to create Released Products directly, skipping the shared product creation step, which will fail in multi-entity environments | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 2 | **CRITICAL** | BOM Journal | BOM Journal section trains users on raw material splitting via negative/positive quantities with manual cost entry -- FRD review recommended lightweight production orders (cutting BOMs) instead; training must align to final design decision | [Section 3.2.4 - BOM Journal](#24-bom-journal-section-324) |
| 3 | **CRITICAL** | FRD Coverage Gaps | Multiple FRD-defined processes are completely absent from training: Transfer Orders (TO0001-TO0004), Item Creation Workflow/Approval (IN04-002), WMS Mobile Receiving (W001-W003), Quality Check on Receiving, Item Reservation for Projects (IN02-001) | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 4 | **HIGH** | Product Creation | Storage dimension group hardcoded as "SiteWHLoc" -- FRD review identified two groups needed (with/without Location); training must document both and guide users on selection criteria | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 5 | **HIGH** | Product Creation | Tracking dimension group stated as "None" contradicts FRD requirement for serial number tracking on high-value components -- training must document when serial tracking applies | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 6 | **HIGH** | Product Creation | Item Model Group hardcoded as "Weighted Average" -- FRD specifies Standard Cost for finished products; training must differentiate costing methods by item type | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 7 | **HIGH** | Product Creation | Product Lifecycle State not mentioned -- FRD review recommended Draft/Active lifecycle states to prevent unapproved items from being used in transactions | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 8 | **HIGH** | Inbound Orders | Arrival overview process is incomplete -- does not cover WMS mobile device receiving which is the primary receiving method defined in the FRD (W001-W003) | [Section 3.1 - Inbound Orders](#20-inbound-orders-section-31) |
| 9 | **HIGH** | Master Planning | Master Planning section is superficial -- missing navigation to run MRP, firming process, planned order conversion to PO, and distinction between Planning Optimization vs. deprecated MRP | [Section 4.0 - Master Planning](#40-master-planning-section-40) |
| 10 | **HIGH** | Transfer Journal | Transfer journal described as transferring between warehouses, but this is incorrect for WMS-enabled warehouses -- WMS warehouses require Transfer Orders, not transfer journals | [Section 3.2.5 - Transfer Journal](#25-transfer-journal-section-325) |
| 11 | **MEDIUM** | Product Creation | Reservation hierarchy specified as "Standard" without explanation of what this means or when to use alternative hierarchies | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 12 | **MEDIUM** | Adjustment Journal | Adjustment journal section does not mention journal approval workflow -- FRD requires approval on inventory journals for governance | [Section 3.2.2 - Adjustment Journal](#23-adjustment-journal-section-322) |
| 13 | **MEDIUM** | Counting Journal | Counting journal section does not mention Cycle Counting Plans for systematic spot checks -- FRD review recommended this over ad-hoc journals | [Section 3.2.3 - Counting Journal](#30-counting-journal-and-mobile-counting-sections-323-and-35) |
| 14 | **MEDIUM** | Production Process | WMS raw material picking section uses manual "Complete Work" in back office instead of training on mobile device execution -- contradicts WMS operational model | [Section 3.3 - Production Process](#33-production-process-warehouse-work-for-raw-material-picking) |
| 15 | **MEDIUM** | Reservation Management | Unreserve process instructs users to "change reservation to 0" without explaining implications -- removing project reservations can break project cost tracking and planned order linkage | [Section 5.0 - Reserved and Unreserved Item Check](#50-reserved-and-unreserved-item-check-section-50) |
| 16 | **MEDIUM** | Inventory Reports | Report section is purely descriptive with no step-by-step instructions -- inconsistent with the procedural approach used in all other sections | [Section 6.0 - Inventory Reports](#60-inventory-reports-section-60) |
| 17 | **MEDIUM** | Document Structure | Section numbering is inconsistent -- Section 3.0 header says "Inventory Processes" but subsections jump to 3.1.1 without a 3.0 or 3.1 parent; some headings have numbers, others do not | [Document Structure and Quality](#document-structure-and-quality) |
| 18 | **LOW** | Product Creation | Item groups table lists 10 groups (CRM, SFP, FIP, FAC, OEM, INS, PAF, SER, DIG, SUB) but does not explain selection criteria -- users need guidance on which group to select for each item type | [Section 2.0 - Product Creation](#10-product-creation-section-20) |
| 19 | **LOW** | Movement Journal | Movement journal does not mention Site dimension which is mandatory for all inventory transactions -- instruction jumps from item number to warehouse, skipping site selection | [Section 3.2.1 - Movement Journal](#21-movement-journal-section-321) |
| 20 | **LOW** | Counting Journal | "Quantity" field description is misleading -- states it "will show the differences" but in D365 the difference is displayed in the "Quantity" column only after the "Counted" field is entered and the system calculates the variance | [Section 3.2.3 - Counting Journal](#30-counting-journal-and-mobile-counting-sections-323-and-35) |
| 21 | **LOW** | Project Item Consumption | Project item journal section is too brief (5 lines) for a core inventory-to-project consumption process -- missing project category selection, cost price validation, financial dimension defaulting | [Section 3.4 - Consume Items on Project](#34-consume-items-on-the-project-directly-section-34) |
| 22 | **LOW** | Overall Completeness | No section on Inventory Closing/Recalculation -- critical month-end process for weighted average costing that directly affects financial accuracy | [Cross-Reference with FRD](#cross-reference-with-frd) |

**Totals:** 3 CRITICAL | 7 HIGH | 7 MEDIUM | 5 LOW

---

## Executive Summary

This training manual covers Technica's Inventory Management operations in D365 F&O across six main sections: Product Creation, Inventory Processes (Inbound Orders, Journal Entries, Production Picking, Project Consumption, Mobile Counting), Master Planning, Reservation Management, and Inventory Reports. The document is a V4.0 release, indicating multiple revision cycles, and includes 77 screenshots to support step-by-step instructions.

**Overall assessment: The training manual provides a reasonable starting point for basic inventory operations but has significant gaps when measured against the FRD requirements and D365 best practices.**

Key findings:

1. **The Product Creation section is the most problematic area.** It conflates Product Masters with Distinct Products, hardcodes configuration values that the FRD explicitly identified as needing multiple options (storage dimension groups, item model groups, tracking dimensions), and omits the Product Lifecycle State governance mechanism recommended in the FRD review.

2. **Multiple FRD-defined processes are completely absent from the training.** Transfer Orders (the FRD's most detailed WMS process with 4 requirements), WMS mobile device receiving (3 requirements), item creation approval workflow, quality check on receiving, and the item reservation workflow are all missing. This represents approximately 40% of the FRD scope that has no training coverage.

3. **The BOM Journal section trains users on the exact approach that the FRD review flagged as suboptimal.** The FRD review recommended lightweight production orders (cutting BOMs) for raw material splitting instead of BOM journals with manual cost entry. The training must align with whichever approach is ultimately selected.

4. **Journal entries are well-documented procedurally** but lack governance context -- journal approval workflows, reason codes, and posting validation are not covered.

5. **The document has structural inconsistencies** -- section numbering is erratic (some sections numbered, some not), the Inventory Processes section (3.0) lacks a proper parent heading, and the Reports section switches from procedural to descriptive format without explanation.

6. **The manual is well-supported by screenshots** (77 images across the document), which is a positive aspect for user training. However, the screenshots appear to be from a specific environment configuration and may need updating if the FRD review recommendations are adopted.

---

## Process-by-Process Review

### 1.0 Product Creation (Section 2.0)

**What the training covers:**
- Navigation to Released Products form
- Step-by-step field entry for new product creation
- Item group selection from a predefined table (10 groups)
- Storage dimension group, tracking dimension, reservation hierarchy
- Coverage group and min/max setup
- Cost group assignment
- Procurement categories and attributes
- Product dimensions (Configuration) and variants
- Default order settings

**Assessment: Contains multiple D365 accuracy issues and FRD misalignments**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Acceptable | Released products path is correct |
| Product type/subtype guidance | **>> NEEDS REVIEW <<** | Only covers "Item/Product" -- does not address Product Masters |
| Item model group | **>> NEEDS REVIEW <<** | Hardcoded as "Weighted Average" -- FRD requires two costing methods |
| Storage dimension group | **>> NEEDS REVIEW <<** | Hardcoded as "SiteWHLoc" -- FRD needs two groups |
| Tracking dimension group | **>> NEEDS REVIEW <<** | States "None" -- FRD requires serial tracking for some items |
| Item groups table | Good | Clear mapping of 10 item groups |
| Coverage group setup | Good | Min/max and requirement-based coverage explained |
| Cost group mapping | Good | Alignment between project category and cost group is useful |
| Product dimensions/variants | Acceptable | Configuration variant creation is documented |
| Product Lifecycle State | **MISSING** | Not mentioned anywhere -- FRD review recommended this |
| Default order settings | Good | Default site/warehouse guidance is practical |

**>> ATTENTION AREA: Product creation process conflates Product Masters and Distinct Products**

> The training instructs users to navigate directly to "Released products" and click "New." In D365 F&O, creating a Released Product directly (without first creating a Shared Product in the global product catalog) works only for single-entity environments. The FRD references "different legal entities" in the product creation context. If Technica operates or plans to operate multiple legal entities, users must first create the product in Product Information Management > Products > Products (shared catalog), then release it to the legal entity. The current training path will fail or create orphaned records in multi-entity scenarios.

**>> ATTENTION AREA: Storage Dimension Group hardcoded to single value**

> The training states "Storage dimension group: Select SiteWHLoc." The FRD review (Enhancement #6) explicitly identified that **two Storage Dimension Groups** are needed -- one with Location tracking (for WMS-enabled warehouses) and one without (for basic warehouses). Training users to always select "SiteWHLoc" will cause problems for items that should not have location-level tracking. Storage dimension groups cannot be changed after transactions are posted.

**>> ATTENTION AREA: Item Model Group hardcoded to Weighted Average**

> The training states "Select an item model group (Weighted average)." The FRD specifies **Weighted Average for stock items** and **Standard Cost for finished products**. This is a critical distinction -- using Weighted Average for finished products in a make-to-order environment with BOMs will not support proper BOM cost rollup and standard cost revaluation. The training must document both costing methods and provide clear guidance on when each applies.

**>> ATTENTION AREA: Tracking Dimension Group set to "None" contradicts FRD**

> The training states "Tracking dimension group: None; Serial might be used in a later stage." The FRD (IN01-003) explicitly requires serial number tracking for high-value components. "Might be used in a later stage" is not an acceptable training instruction -- either serial tracking is configured for specific item categories or it is not. The tracking dimension group, like the storage dimension group, cannot be changed after transactions are posted.

**>> ENHANCEMENT:** Rewrite the Product Creation section to include:
1. Decision tree: When to create a Product Master (with variants) vs. a Distinct Product
2. Decision matrix for Storage Dimension Group selection (SiteWHLoc vs. SiteWH)
3. Decision matrix for Item Model Group selection (Weighted Average vs. Standard Cost) based on item type
4. Explicit guidance on when to enable serial number tracking (Tracking Dimension Group: Serial vs. None)
5. Product Lifecycle State assignment (Draft for new items, Active after approval)
6. Clear indication that dimension groups are permanent after first transaction

**>> ENHANCEMENT:** Add a "Product Creation Decision Guide" table:

| Item Type | Item Group | Item Model Group | Storage Dim Group | Tracking Dim Group | Lifecycle State |
|-----------|-----------|-----------------|-------------------|-------------------|----------------|
| Component/Raw Material | CRM | Weighted Average | SiteWHLoc | None or Serial | Draft |
| Semi-Finished Product | SFP | Standard Cost | SiteWHLoc | None | Draft |
| Finished Product | FIP | Standard Cost | SiteWHLoc | Serial | Draft |
| Consumables | FAC | Weighted Average | SiteWH | None | Draft |

**>> ENHANCEMENT:** Document the Product Lifecycle State process:
- New items are created in "Draft" state (cannot be used in transactions)
- After review/approval, lifecycle state is changed to "Active"
- Discontinued items are set to "Discontinued" (blocked from new transactions)
- This was identified as the recommended approach for GAP IN04-002 in the FRD review

---

### 2.0 Inbound Orders (Section 3.1)

**What the training covers:**
- Two methods of receiving: Arrival Overview and direct PO receiving
- Step-by-step for Arrival Overview (filter, select, start arrival, post journal, product receipt)
- Step-by-step for PO Product Receipt posting (select PO, generate product receipt, specify receipt number)

**Assessment: Procedurally sound for back-office receiving but missing WMS mobile receiving**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Arrival overview process | Good | Correctly described with proper navigation |
| Days back/forward filtering | Good | Useful practical guidance |
| Item arrival journal posting | Good | Step sequence is correct |
| Product receipt from arrival journal | Good | Functions > Product receipt path is correct |
| Direct PO product receipt | Good | Standard process, well-documented |
| PO status filtering (Confirmed, Open Order) | Good | Practical guidance for users |
| Close for receipt checkbox | Good | Important partial receipt handling |
| WMS mobile device receiving | **MISSING** | FRD defines W001-W003 for mobile receiving |
| Quality check on receiving | **MISSING** | FRD defines pass/fail quality gate |
| Landed Cost / voyage receiving | **MISSING** | FRD cross-reference to FRD06 Logistics |

**>> ATTENTION AREA: WMS mobile device receiving is not documented**

> The FRD (W001-W003) defines WMS mobile device receiving as the primary method for warehouse-managed items, with back-office receiving reserved for trade items. The training manual only covers back-office receiving methods (Arrival Overview and direct PO Product Receipt). For a WMS-enabled environment, the mobile device receiving flow (scan PO, scan item, scan LP, confirm receipt, put-away work generation) is the daily operational method and must be included.

**>> ATTENTION AREA: No quality check process on receiving**

> The FRD defines a quality check step during receiving where items are inspected (pass/fail) and failed items are routed to a non-conformance location. The training manual makes no mention of any quality verification during the receiving process. Even if the full Quality Management module is not used (per FRD decision), the Inventory Status-based quality gate (Available vs. QC Hold) should be documented as part of the receiving workflow.

**>> ENHANCEMENT:** Add a subsection "3.1.3 - Receive Items via WMS Mobile Device" covering:
1. Login to Warehouse Management mobile app
2. Select "PO Item Receiving" menu item
3. Scan/enter PO number
4. Scan/enter item number and quantity
5. License plate generation
6. Quality check step (pass = Available status, fail = QC Hold status)
7. Put-away work generation and execution
8. Confirmation and completion

**>> ENHANCEMENT:** Add a subsection "3.1.4 - Quality Check on Receiving" covering:
1. How to set Inventory Status to "QC Hold" for items pending inspection
2. How to change status to "Available" after passing inspection
3. How to route failed items to the non-conformance location
4. When this process applies (which item types, which receiving methods)

---

### 2.1 Movement Journal (Section 3.2.1)

**What the training covers:**
- Purpose: adjust inventory levels with specific offset ledger account
- Navigation to movement journal
- Line entry: item number, warehouse, quantity, cost price, offset account
- Posting the journal

**Assessment: Technically correct but missing Site dimension and use-case guidance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear distinction from adjustment journal |
| Navigation path | Correct | Inventory management > Journal entries > Movement |
| Field-by-field instructions | Acceptable | Basic fields covered |
| Site dimension | **MISSING** | Site is mandatory but not mentioned |
| Journal name selection | **MISSING** | Users need to know which journal name to select |
| Reason codes | **MISSING** | Not mentioned -- FRD governance requires reason codes |
| Use-case examples | **MISSING** | No guidance on when to use movement vs. adjustment |

**>> ENHANCEMENT:** Add Site field to the step-by-step instructions (Site is a mandatory inventory dimension). Add guidance on journal name selection and include at least two practical use-case examples (e.g., "Use movement journal when you need to charge a specific GL account for scrap disposal" vs. "Use adjustment journal when the default posting profile should determine the GL account").

---

### 2.2 Adjustment Journal (Section 3.2.2)

**What the training covers:**
- Purpose: reconcile discrepancies between physical stock and system records
- Key difference from movement journal: cost posts to default GL account via item group posting profile
- Navigation and line entry (same fields as movement journal minus offset account)
- Posting

**Assessment: Correct but missing journal approval workflow**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear distinction from movement journal |
| Posting profile explanation | Good | Important for users to understand auto-posting |
| Journal approval workflow | **MISSING** | FRD requires approval on inventory journals |
| Reason codes | **MISSING** | No guidance on categorizing adjustments |
| Threshold guidance | **MISSING** | No guidance on when adjustments require escalation |

**>> ATTENTION AREA: Journal approval workflow not documented**

> The FRD review identified that inventory journal approval workflow is enabled as a governance control. The training manual does not mention the approval process at all. Users need to understand that after creating and validating journal lines, the journal must be submitted for approval before it can be posted. This is a critical governance step that prevents unauthorized inventory adjustments.

**>> ENHANCEMENT:** Add a subsection on the journal approval process:
1. After entering journal lines, click "Submit for approval" (not "Post")
2. Journal routes to the designated approver based on journal type and amount
3. Approver reviews and approves/rejects
4. Only approved journals can be posted
5. Include guidance on reason code selection for each adjustment type (scrap, damage, count variance, etc.)

---

### 2.3 Adjustment Journal (Section 3.2.2)

*Covered above -- no separate subsection needed.*

---

### 2.4 BOM Journal (Section 3.2.4)

**What the training covers:**
- Purpose: raw material distribution -- stock out one material, stock in others as if related through a BOM
- Example: sheet metal cut into parts
- Navigation to BOM journal
- Add sheet metal with negative quantity, parts with positive quantity
- Manually update cost of positive quantities
- Post the journal

**Assessment: This is the most contentious section -- directly conflicts with FRD review recommendation**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Acceptable | The use case is clear |
| BOM journal as splitting mechanism | **>> NEEDS REVIEW <<** | FRD review recommended production orders instead |
| Manual cost entry on positive lines | **>> NEEDS REVIEW <<** | Error-prone and lacks audit trail |
| Navigation path | **>> NEEDS REVIEW <<** | Path described is non-standard -- see below |
| Scrap handling | **MISSING** | No guidance on how to handle scrap pieces |

**>> ATTENTION AREA: BOM Journal approach conflicts with FRD review recommendation**

> The FRD review (Enhancement #2, HIGH priority) explicitly recommended **lightweight production orders (cutting BOMs)** instead of BOM journals or adjustment journals for raw material splitting. The rationale was that production orders maintain automatic cost flow, create clear audit trails, and handle scrap naturally -- while BOM journals require manual cost entry ("Update the cost of the positive quantities as per the usage") which is error-prone and creates artificial gain/loss when costs are entered incorrectly.
>
> The training manual should align with the final design decision. If the cutting BOM / production order approach is adopted, this entire section needs to be rewritten. If the BOM journal approach is retained despite the FRD review recommendation, the manual cost entry process needs much more detailed guidance including cost calculation methodology and validation steps.

**>> ATTENTION AREA: BOM Journal navigation path is non-standard**

> The training states: "Navigate to Inventory management, Journal entries, Items, Select Bill of Material." The standard D365 navigation path is **Inventory management > Journal entries > Bills of materials**. The described path suggests navigating to an "Items" submenu which does not exist as a standard menu item under Journal entries. This may reflect a custom menu configuration or an error in the training document.

**>> ENHANCEMENT:** Based on the final design decision:

**Option A (if cutting production orders are adopted):** Replace the BOM Journal section entirely with a new section on "Raw Material Cutting Process" that covers:
1. Creating a cutting production order from a pre-defined cutting BOM
2. Starting the production order
3. Reporting input material consumption
4. Reporting output pieces (including scrap quantity)
5. Ending the production order (cost automatically flows from input to output)

**Option B (if BOM journal approach is retained):** Significantly expand the section to include:
1. How to calculate the correct cost for positive quantity lines (cost allocation methodology)
2. How to handle scrap (negative net quantity)
3. Validation steps before posting (total debit must equal total credit)
4. Correct navigation path
5. Examples with actual numbers showing cost flow

---

### 2.5 Transfer Journal (Section 3.2.5)

**What the training covers:**
- Purpose: transfer items between stocking locations, batches, or product variants
- Key point: no cost implications, no in-transit tracking
- Distinction from transfer orders (use transfer orders for in-transit tracking)
- Navigation and line entry: item, quantity, from/to site, from/to warehouse
- Posting creates two transactions (issue and receipt)

**Assessment: Conceptually correct but contains a critical WMS limitation that is not documented**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear purpose statement |
| Transfer order vs. transfer journal distinction | Good | Important for users |
| Two-transaction explanation | Good | Helps users understand what happens on posting |
| WMS warehouse limitation | **>> NEEDS REVIEW <<** | Transfer journals cannot be used for WMS-enabled warehouses |
| From/to location fields | **MISSING** | Location is part of the dimension group but not in the steps |

**>> ATTENTION AREA: Transfer journals cannot be used between WMS-enabled warehouses**

> The training states "you can transfer items from one warehouse to another warehouse within the same company." While this is true for basic (non-WMS) warehouses, **transfer journals are not supported for warehouses with Warehouse Management enabled**. For WMS-enabled warehouses, users must use **Transfer Orders** (which generate warehouse work for picking and receiving). Since the FRD defines WMS-enabled warehouses, this section needs a clear warning about when transfer journals can and cannot be used. Using a transfer journal for WMS warehouses will result in an error or inventory discrepancies.

**>> ENHANCEMENT:** Add a decision matrix:

| Scenario | Use Transfer Journal | Use Transfer Order |
|----------|---------------------|--------------------|
| Between locations in same basic warehouse | Yes | No |
| Between basic warehouses (same site) | Yes | Yes |
| Between WMS-enabled warehouses | **No** | **Yes (required)** |
| Between sites | No | Yes |
| Need in-transit tracking | No | Yes |

---

### 3.3 Production Process (Warehouse Work for Raw Material Picking)

**What the training covers:**
- Purpose: provide inventory items to production team
- Work generated during release of production order (for reserved quantities)
- Navigation to All Work screen
- Filtering work by type or order number
- Completing work via back-office Complete Work function
- Important note on residual quantities requiring transfer journal

**Assessment: Good conceptual coverage but uses back-office work completion instead of mobile device**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear link between production release and work generation |
| Reservation prerequisite | Good | Correctly states work is for reserved quantities only |
| All Work navigation | Correct | Warehouse management > Work > All work |
| Back-office work completion | **>> NEEDS REVIEW <<** | Should train on mobile device execution |
| Residual quantity handling | Good | Practical guidance on transfer journal for remainders |
| Picking list journal relationship | **MISSING** | Does not explain how WMS work relates to the picking list journal |

**>> ATTENTION AREA: WMS work should be completed on mobile device, not in back office**

> The training instructs users to complete warehouse work from the back office ("Complete work > Select the user ID > Validate the work > Complete work"). In a WMS-enabled environment, warehouse work is primarily executed on the **Warehouse Management mobile device** -- the worker scans the pick location, confirms the item and quantity, scans the put location, and the system completes the work. Back-office work completion is an administrative override, not the standard operational method. Training users on back-office completion as the primary method undermines the purpose of WMS implementation.

**>> ENHANCEMENT:** Rewrite this section to train on mobile device execution:
1. Worker receives work assignment on mobile device (or selects from work list)
2. Scan/confirm pick location (raw materials warehouse location)
3. Scan/confirm item number and quantity
4. Scan/confirm put location (factory/production input location)
5. System completes the work and updates inventory dimensions
6. Add a note that back-office completion is available as an administrative fallback

---

### 3.4 Consume Items on the Project Directly (Section 3.4)

**What the training covers:**
- Project item journal removes quantity from inventory and records cost on project
- Navigation: Project management and accounting > Projects > All projects
- Select project, navigate to Journals > Item
- Create new journal, add lines, post

**Assessment: Too brief for a core process -- missing critical field guidance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear purpose in two sentences |
| Navigation path | Correct | Standard project journal navigation |
| Step-by-step | **Insufficient** | Only 5 lines for a process with multiple critical fields |
| Project category | **MISSING** | Must be specified on each line -- determines revenue recognition |
| Cost price validation | **MISSING** | Users need to verify cost price matches inventory |
| Financial dimensions | **MISSING** | Project, department, cost center defaults |
| Site/warehouse selection | **MISSING** | Not mentioned but mandatory |
| Inventory reservation | **MISSING** | How does this relate to project reservation (Section 5.0)? |

**>> ENHANCEMENT:** Expand this section to include:
1. Project category selection (and reference to the ProjItem/Purchased/Raw material mapping from Section 2.0)
2. Site and warehouse specification
3. Cost price validation (system defaults from inventory; verify before posting)
4. Relationship to project reservation (posting the item journal removes the reservation)
5. Financial dimension defaulting from the project setup
6. At minimum, double the length of this section with practical field-by-field guidance

---

### 3.0 Counting Journal and Mobile Counting (Sections 3.2.3 and 3.5)

**What the training covers:**
- Counting journal: create, generate lines from on-hand, enter counted quantity, system calculates difference, post
- Mobile counting: sign in, select menu, enter location/item/quantity, finish

**Assessment: Both sections are functionally correct but lack governance and planning context**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Counting journal creation | Good | Standard process correctly described |
| Create lines from on-hand | Good | Practical approach for full counts |
| Counted vs. system quantity | Acceptable | Concept is right but field description is slightly inaccurate |
| Mobile counting steps | Good | Simple, clear instructions |
| Cycle counting plans | **MISSING** | FRD review recommended systematic cycle counting |
| Counting frequency guidance | **MISSING** | No guidance on biweekly spot checks vs. annual counts |
| Approval on count adjustments | **MISSING** | Large variances should require approval |
| Tag counting | **MISSING** | FRD review recommended for large warehouses |

**>> ATTENTION AREA: Counting journal "Quantity" field description is inaccurate**

> The training states: "In the Quantity field, will show the differences between the system counted and physically counted quantities." In D365, the standard counting journal shows the **current on-hand quantity** in the "Quantity" column when lines are generated. After the user enters the "Counted" quantity, the system calculates and displays the **variance** in a separate field. The current description conflates these two states and may confuse users.

**>> ENHANCEMENT:** Add a subsection on "Counting Strategy" that explains:
1. Biweekly spot checks: Use Cycle Counting Plans to automatically select items/locations (FRD review recommendation #10)
2. Annual full count: Use counting journal with batch line generation for all items
3. Threshold-based approval: Count adjustments above a defined threshold require manager approval before posting
4. Reconcile the counting journal section with the mobile counting section -- explain when to use each method

---

### 4.0 Master Planning (Section 4.0)

**What the training covers:**
- Conceptual overview of master planning (demand vs. supply calculation)
- Min/max replenishment explanation
- Two types of master plans (requirement-based vs. min/max)
- Navigation to planned orders
- Check and approve planned orders

**Assessment: Superficial -- missing critical operational steps**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Conceptual explanation | Good | Clear purpose statement |
| Min/max coverage explanation | Good | Well-linked to item setup in Section 2.0 |
| Two plan types | Good | Useful distinction for users |
| How to run master planning | **MISSING** | No step-by-step for executing MRP |
| Planned order firming | **MISSING** | Approve mentioned but firm/convert not documented |
| Planning Optimization vs. deprecated MRP | **MISSING** | Critical -- deprecated engine will be removed |
| Net requirements view | **MISSING** | Essential for understanding why an order was planned |
| Planned production orders | **MISSING** | Only planned purchase orders mentioned |

**>> ATTENTION AREA: Master Planning section is missing the core operational steps**

> The training tells users that planned orders "will show under master planning > master planning > planned orders" but does not explain **how to run master planning** in the first place. Users need to know: (a) the navigation to run master planning, (b) which master plan to select, (c) whether to run regenerative or net change, (d) the scope parameters (item/coverage group filters), and (e) how to firm planned orders into actual purchase orders or production orders.

**>> ATTENTION AREA: No mention of Planning Optimization**

> Microsoft has deprecated the legacy MRP engine in D365 F&O and is replacing it with **Planning Optimization**. The training should reference Planning Optimization as the current/future engine and note any differences in the user experience. If Technica is going live on Planning Optimization (which is now the default), some navigation paths and options differ from the legacy engine.

**>> ENHANCEMENT:** Expand this section to include:
1. How to run master planning (navigation, plan selection, run parameters)
2. How to review planned orders (filters, net requirements view)
3. How to firm planned orders into purchase orders or production orders
4. Distinction between planned purchase orders and planned production orders
5. How to handle planned orders that should not be firmed (alternatives in stock)
6. Reference to Planning Optimization as the current engine

---

### 5.0 Reserved and Unreserved Item Check (Section 5.0)

**What the training covers:**
- Viewing reserved items via Inventory transactions
- Filtering by project ID to see project-reserved items
- Unreserving items by navigating to Reservation form and changing quantity to 0

**Assessment: Functionally correct but lacks impact awareness and governance**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation to transactions | Correct | Inventory management > Inquiries and reports > Transactions |
| Project filter | Good | Practical guidance |
| Unreserve process | **>> NEEDS REVIEW <<** | No guidance on implications |
| Re-reservation to different project | **MISSING** | FRD covers reservation switching |
| Authorization | **MISSING** | Who can unreserve? No role guidance |

**>> ATTENTION AREA: Unreserving items without impact explanation is risky**

> The training instructs users to "change the reservation to 0 and select reserve lot" without explaining the downstream impact. Removing a project reservation can: (a) make the item available for other projects or sales orders (potentially causing allocation conflicts), (b) break the linkage between the item and the project cost forecast, (c) affect master planning calculations (the demand may regenerate a new planned order for the project), and (d) impact the project budget tracking. Users must understand these implications before unreserving items.

**>> ENHANCEMENT:** Add:
1. When it is appropriate to unreserve (item no longer needed for project, alternative item used, project cancelled)
2. Impact on project cost tracking and master planning
3. How to re-reserve to a different project (reservation switching per FRD)
4. Authorization guidance (who should approve unreservation decisions)

---

### 6.0 Inventory Reports (Section 6.0)

**What the training covers:**
- On-hand list: filter by dimensions, display breakdowns
- Inventory transactions: view transaction details with reference and status
- Critical on-hand inventory: items below minimum levels
- Inventory value report: physical and financial quantities, COGS, WIP, reconciliation

**Assessment: Descriptive overview only -- no step-by-step instructions**

| Aspect | Rating | Comment |
|--------|--------|---------|
| On-hand list description | Acceptable | Conceptually correct |
| Inventory transactions | Acceptable | Mentions key statuses |
| Critical on-hand | Good | Links to item coverage setup |
| Inventory value report | Good | Mentions key use cases (reconciliation, COGS) |
| Navigation paths | **MISSING** | No navigation provided for any report |
| Step-by-step instructions | **MISSING** | Inconsistent with all other sections |
| Report parameters | **MISSING** | No guidance on setting date ranges, filters |
| Inventory aging report | **MISSING** | FRD mentions aging and slow-moving reports |
| On-hand field definitions table | Good | Table 2 provides clear field definitions |

**>> ATTENTION AREA: Report section format inconsistent with rest of document**

> All other sections in the training manual follow a procedural step-by-step format with screenshots. The Reports section switches to a purely descriptive format with no navigation paths, no step-by-step instructions, and no screenshots (the on-hand list section appears to have one screenshot but no instructions). This inconsistency suggests the section was not completed to the same standard as the rest of the document.

**>> ENHANCEMENT:** For each report, add:
1. Navigation path
2. Key parameters/filters to set
3. How to interpret the output
4. When to use each report (monthly reconciliation, daily stock check, etc.)
5. Add the missing Inventory Aging report and Slow-Moving Items report (referenced in FRD)

---

## Document Structure and Quality

**>> ATTENTION AREA: Section numbering is inconsistent**

> The document uses the following numbering scheme:
> - 1.0 Introduction (numbered)
> - 2.0 Product Creation (numbered)
> - "Inventory Processes" (Section 3 header -- **unnumbered**)
> - 3.1.1 Arrival overview (sub-subsection number without parent 3.1)
> - "Movement Journal" (heading -- **unnumbered**)
> - "Adjustment Journal" (heading -- **unnumbered**)
> - 3.2.3 Counting journal (numbered -- but 3.2.1 and 3.2.2 are unnumbered)
> - 3.2.4 BOM journal (numbered)
> - 3.2.5 Transfer journal (numbered)
> - 3.4 Consume items on the project directly (numbered)
> - 3.5 Counting by mobile application (numbered)
> - 4.0 Master Planning (numbered)
> - 5.0 Reserved and unreserved item check (numbered)
> - 6.0 Inventory Reports (numbered)
>
> This inconsistency makes it difficult to reference specific sections and suggests sections were added incrementally without structural review.

**>> ENHANCEMENT:** Standardize all section numbering. Suggested structure:
- 1.0 Introduction
- 2.0 Product Creation
- 3.0 Inventory Processes
  - 3.1 Inbound Orders
    - 3.1.1 Arrival Overview
    - 3.1.2 Receive from PO Directly
    - 3.1.3 WMS Mobile Receiving (NEW)
  - 3.2 Journal Entries
    - 3.2.1 Movement Journal
    - 3.2.2 Adjustment Journal
    - 3.2.3 Counting Journal
    - 3.2.4 BOM Journal (or Cutting Process)
    - 3.2.5 Transfer Journal
  - 3.3 Production Picking (WMS Work)
  - 3.4 Project Item Consumption
  - 3.5 Mobile Counting
- 4.0 Master Planning
- 5.0 Reservation Management
- 6.0 Inventory Reports
- 7.0 Transfer Orders (NEW -- per FRD)
- 8.0 Inventory Closing and Recalculation (NEW)

---

## Cross-Reference with FRD

This section maps FRD009 requirements and FRD review findings to their coverage in the training manual.

### FRD Requirements Coverage Matrix

| FRD Requirement | FRD Section | Training Coverage | Gap Assessment |
|----------------|-------------|-------------------|----------------|
| Product Setup (IN01-001 to IN01-005) | Section 1 | **Partially covered** in Section 2.0 | Storage dim groups, tracking dims, and costing methods are oversimplified |
| Site/Warehouse/Location Setup | Section 2 | **Not covered** | No training on warehouse structure, location setup, or WMS configuration |
| Item Reservation for Projects (IN02-001) | Section 3 | **Not covered** | FRD GAP -- reservation from PR/Item Requirements not trained |
| Quality Check on Receiving | Section 3 | **Not covered** | Pass/fail and non-conformance routing not trained |
| Material Delivery Form (IN03-001) | Section 4 | **Not covered** | FRD GAP -- not trained regardless of solution approach |
| Raw Material Splitting (IN03-009) | Section 4 | **Partially covered** in BOM Journal (3.2.4) | Covers BOM journal approach but FRD review recommended production orders |
| Item Creation Approval (IN04-002) | Section 5 | **Not covered** | FRD GAP -- Product Lifecycle State workflow not trained |
| WMS Mobile Receiving (W001-W003) | Section 6 | **Not covered** | Primary receiving method in FRD is completely absent from training |
| Transfer Orders (TO0001-TO0004) | Section 7 | **Not covered** | Most detailed WMS process in FRD has zero training coverage |
| Transfer Order Workflow (TO0002) | Section 7 | **Not covered** | FRD GAP -- approval workflow not trained |
| WMS Item Movement | Section 8 | **Not covered** | Mobile device movement not trained (transfer journal partially covers this) |
| Stock Counting (Spot + Annual) | Section 9 | **Partially covered** in Sections 3.2.3 and 3.5 | Missing cycle counting plans, counting strategy |
| Inventory Adjustment with Approval | Section 10 | **Partially covered** in Section 3.2.2 | Adjustment journal covered but approval workflow missing |
| Master Planning | Cross-section | **Partially covered** in Section 4.0 | Conceptual only -- no operational steps |
| Project Item Consumption | Cross-section | **Partially covered** in Section 3.4 | Too brief -- missing critical fields |
| Inventory Reports | Cross-section | **Partially covered** in Section 6.0 | Descriptive only -- no procedures |
| Inventory Closing/Recalculation | Implied by WAM | **Not covered** | Critical month-end process completely absent |

**Coverage Summary:** Of 17 major FRD requirement areas, **7 are not covered at all**, **8 are partially covered**, and **2 are adequately covered**. This represents approximately **40% complete FRD coverage**.

### FRD Review Enhancement Alignment

| FRD Review Enhancement | Priority | Training Alignment | Status |
|----------------------|----------|-------------------|--------|
| #1: Replace PR reservation with Project Item Requirements | HIGH | Not addressed -- no reservation training at all | **>> NEEDS REVIEW <<** |
| #2: Use cutting production orders instead of BOM journals | HIGH | **Training teaches the opposite approach** (BOM journals) | **>> NEEDS REVIEW <<** |
| #3: Use Product Lifecycle State for item approval | HIGH | Not mentioned in training | **>> NEEDS REVIEW <<** |
| #4: Power Automate for transfer order approval | MEDIUM | Transfer orders not covered in training | **>> NEEDS REVIEW <<** |
| #5: Use Production Picking List as Material Delivery Form | MEDIUM | Not covered in training | **>> NEEDS REVIEW <<** |
| #6: Define two Storage Dimension Groups | MEDIUM | **Training hardcodes single group** | **>> NEEDS REVIEW <<** |
| #7: Close product dimension debate (Config + Size only) | LOW | Training mentions Configuration only; Size not covered | Partial |
| #8: Clarify WMS-enabled vs. basic warehouses | MEDIUM | Not addressed -- no warehouse setup training | **>> NEEDS REVIEW <<** |
| #9: System-directed put-away for Components | LOW | Not covered | N/A |
| #10: Cycle Counting Plans for spot checks | LOW | Not covered -- ad-hoc counting only | **>> NEEDS REVIEW <<** |
| #11: Reconsider Quality Management exclusion | HIGH | No quality process in training at all | **>> NEEDS REVIEW <<** |
| #12: Auto-approve transfer orders from Master Planning | MEDIUM | Transfer orders not covered | N/A |
| #13: Map transfer order flow to Production FRD05 warehouses | HIGH | No cross-FRD warehouse references | **>> NEEDS REVIEW <<** |
| #14: Align receiving with Logistics FRD06 | MEDIUM | No voyage/landed cost receiving covered | **>> NEEDS REVIEW <<** |

**Alignment Summary:** Of 14 FRD review enhancements, **the training manual does not align with any of them**. Enhancement #2 is actively contradicted (training teaches BOM journals; FRD review recommends production orders). Enhancement #6 is actively contradicted (training hardcodes one storage dimension group; FRD review says two are needed).

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | Product Creation | Rewrite to include decision matrices for dimension groups, costing methods, and tracking dimensions -- stop hardcoding single values | **CRITICAL** |
| 2 | BOM Journal / Cutting | Align with final FRD design decision -- rewrite as cutting production order process OR significantly expand BOM journal guidance with cost calculation methodology | **CRITICAL** |
| 3 | Missing Processes | Add training sections for: Transfer Orders, WMS Mobile Receiving, Item Creation Workflow, Quality Check on Receiving | **CRITICAL** |
| 4 | Product Lifecycle State | Add lifecycle state assignment to product creation and document the Draft-to-Active approval process | **HIGH** |
| 5 | WMS Mobile Device | Replace back-office work completion instructions with mobile device execution as primary method for production picking | **HIGH** |
| 6 | Master Planning | Expand from conceptual overview to full operational guide including run steps, firming, and planned order management | **HIGH** |
| 7 | Transfer Journal | Add WMS limitation warning -- transfer journals cannot be used for WMS-enabled warehouses; provide decision matrix | **HIGH** |
| 8 | Inbound Orders | Add WMS mobile receiving section and quality check process | **HIGH** |
| 9 | Journal Approval | Add approval workflow documentation to all journal types (adjustment, counting, movement) | **MEDIUM** |
| 10 | Counting Strategy | Add Cycle Counting Plans for systematic spot checks; document counting frequency guidance | **MEDIUM** |
| 11 | Reservation Management | Add impact explanation and authorization guidance for unreservation; document reservation switching | **MEDIUM** |
| 12 | Reports | Convert from descriptive to procedural format with navigation paths, parameters, and interpretation guidance | **MEDIUM** |
| 13 | Document Structure | Standardize section numbering throughout the document | **MEDIUM** |
| 14 | Project Item Consumption | Expand with project category, cost price, financial dimension guidance | **LOW** |
| 15 | Inventory Closing | Add new section on Inventory Closing and Recalculation (month-end process) | **LOW** |
| 16 | Movement Journal | Add Site dimension and use-case examples | **LOW** |
| 17 | Item Group Selection | Add selection criteria guidance for the 10 item groups | **LOW** |

---

## Risk Flags

### 1. FRD-to-Training Misalignment on Raw Material Splitting -- CRITICAL RISK

**>> ATTENTION AREA: Training teaches a process the FRD review recommended against**

> The BOM Journal section (3.2.4) trains users on the exact approach -- manual negative/positive quantity entries with manual cost allocation -- that the FRD review (Enhancement #2, HIGH priority) identified as suboptimal. The FRD review recommended lightweight production orders (cutting BOMs) that maintain automatic cost flow and audit trail. If the FRD review recommendation is adopted during implementation but the training manual is not updated, users will be trained on an incorrect process. Conversely, if the BOM journal approach is retained, the FRD review recommendation was rejected without documentation. **A design decision must be finalized and the training manual aligned accordingly before go-live training begins.**

### 2. 40% FRD Coverage Gap -- HIGH RISK

**>> ATTENTION AREA: Nearly half of FRD-defined processes have no training coverage**

> The training manual covers approximately 60% of the inventory processes defined in FRD009. The missing 40% includes some of the most operationally critical processes: Transfer Orders (the primary inter-warehouse movement method), WMS Mobile Receiving (the primary goods receipt method), and the Item Creation Approval Workflow (a governance control). Deploying with these gaps means users will either: (a) not know how to perform these processes, (b) learn through trial and error (high error risk), or (c) require separate informal training that is not documented. All three outcomes are unacceptable for a go-live scenario.

### 3. Hardcoded Configuration Values in Product Creation -- HIGH RISK

**>> ATTENTION AREA: Training hardcodes values that need to vary by item type**

> The product creation section instructs users to always select "Weighted Average" for item model group, "SiteWHLoc" for storage dimension group, "None" for tracking dimension group, and "Standard" for reservation hierarchy. The FRD explicitly requires different values for different item types (Standard Cost for finished products, serial tracking for high-value components, two storage dimension groups). If users follow these training instructions literally, items will be created with incorrect configurations that **cannot be changed after the first transaction is posted**. This is the highest data-integrity risk in the entire training manual because dimension group errors are permanent.

### 4. Missing Inventory Closing Process -- MEDIUM RISK

The training manual does not include Inventory Closing and Recalculation, which is the critical month-end process for Weighted Average costing. Without proper inventory closing, the system will not settle inventory transactions to the weighted average, leading to incorrect cost of goods sold, incorrect inventory valuation, and GL-to-subledger discrepancies. This is typically a finance-team responsibility but inventory users need to understand its impact on their day-to-day cost visibility.

### 5. Back-Office Orientation vs. WMS Operational Reality -- MEDIUM RISK

The training manual is predominantly back-office oriented (forms, journals, list pages), while the FRD defines a WMS-enabled environment where most operational activities (receiving, picking, movement, counting) are executed on the **Warehouse Management mobile application**. The training manual only includes one mobile section (counting in Section 3.5). This creates a disconnect between how users are trained and how they will actually work day-to-day. The mobile application should be the primary training focus for warehouse operations staff, with back-office functions presented as administrative/supervisory tools.

### 6. No Error Handling or Troubleshooting Guidance -- LOW RISK

The training manual assumes every process will complete successfully. There is no guidance on: (a) what to do when a journal fails validation, (b) how to handle partial receipts or over-receipts, (c) what to do when inventory is blocked or on hold, (d) how to reverse a posted journal, or (e) common error messages and their resolutions. While this is typical for a first-pass training manual, adding a "Common Issues" subsection to each process area would significantly reduce support tickets post go-live.

---

*This review is based on D365 F&O Inventory Management, Warehouse Management, and Product Information Management best practices as of 2026. The training manual was evaluated against both general D365 accuracy standards and the specific requirements documented in FRD009 - Inventory Management V2.0 and its associated FRD review.*
