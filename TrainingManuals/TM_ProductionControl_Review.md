# Training Manual - Production Control - Solution Design Verification Report

**Document:** Technica - User Training Manual - Production Control V5.0
**Prepared by:** Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Production Control, Shop Floor Execution

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | Shop Floor Execution - Time Justification | Custom justification prompt when job exceeds estimated time is undocumented as a customization -- no FRD GAP reference, no technical spec, and it alters standard Shop Floor Execution behavior | [Production Floor Execution](#4a---firming-and-processing-production-orders-shop-floor-execution) |
| 2 | **CRITICAL** | FRD Cross-Reference - Assembly Process | FRD defines full assembly lifecycle (PRD014), test strategy meetings, and POC% tracking -- Training Manual contains zero assembly-specific training | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 3 | **CRITICAL** | FRD Cross-Reference - Non-Conformities | FRD defines non-conformity process (PRD013) with scrap cost tracking and rework via WBS -- Training Manual covers only basic error quantity reporting with no structured non-conformance workflow | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 4 | **HIGH** | FRD Cross-Reference - SNAG Management | FRD defines complete SNAG process (PRD015) with WBS activities, SNAG categories, case management, and ECR linkage -- Training Manual has no SNAG coverage whatsoever | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 5 | **HIGH** | FRD Cross-Reference - Notifications | FRD defines 5 notification GAPs (Power Automate) for drawing release, rework, job time exceedance, and cross-order alerts -- Training Manual documents none of these notification workflows | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 6 | **HIGH** | Transfer Journal - From/To Reversal | Steps 11-12 instruct user to transfer FROM Prod-In TO BULK but labels say "location to = Prod-In" and "location from = BULK" -- reversed from the stated intent of returning material to inventory | [Returning Unconsumed Raw Material](#4i---returning-unconsumed-raw-material-transfer-journal) |
| 7 | **HIGH** | Production Pools | Manual production order references pool types (Non-conformity, customer request, snags from site, factory rework) but no guidance on when to use each pool or how pools drive reporting and filtering | [Manual Production Order](#4b---create-a-manual-production-order) |
| 8 | **HIGH** | Reservation Strategy | FRD CRITICAL finding flags "Require Full Reservation = TRUE" as conflicting with partial assembly start -- Training Manual does not address reservation parameters, leaving users unaware of this constraint | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 9 | **MEDIUM** | Product Creation - Tracking Dimensions | Training states "Tracking dimension group: None; Serial might be used in a later stage" -- this defers a decision that affects inventory architecture and cannot be easily changed post-go-live | [Product Creation](#3e---product-creation) |
| 10 | **MEDIUM** | BOM Line Type | Training instructs "line type needs to be pegged supply" for all BOM lines -- this is incorrect for stocked raw materials; Pegged Supply is only appropriate for make-to-order components | [BOMs](#3g---boms) |
| 11 | **MEDIUM** | Dismantling / BOM Journal | BOM journal dismantling process uses negative Report-as-Finished which is a non-standard approach -- no mention of D365 formula/disassembly BOM or standard reverse BOM functionality | [BOM Journal for Dismantling](#4g---bom-journal-for-dismantling) |
| 12 | **MEDIUM** | Revision Cases - Strikethrough Text | Section 5 contains strikethrough text indicating deleted guidance about firming sequence -- final training manual should not contain draft markup | [Revision Cases](#5---revision-cases) |
| 13 | **MEDIUM** | FRD Cross-Reference - CAD Integration | FRD identifies CAD integration as the single biggest production risk affecting BOM population, drawing URLs, and rework triggers -- Training Manual mentions CAD/Inventor integration only in passing with no user procedures | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 14 | **MEDIUM** | FRD Cross-Reference - Quality Management | FRD recommends using D365 Quality Management module for non-conformances -- Training Manual does not reference Quality Management at all | [Cross-Reference with FRD](#cross-reference-with-frd) |
| 15 | **MEDIUM** | Reporting Section | Reports section (Section 6) describes report purposes but provides no step-by-step instructions for generating, filtering, or exporting any report | [Reports and Workspaces](#6---production-control-reports-and-workspaces) |
| 16 | **MEDIUM** | Resource Setup - Capability and Skill | Resource setup covers machines, human resources, and vendors but omits resource capabilities and skills configuration which are needed for automatic resource selection during scheduling | [Resources](#3b---resources) |
| 17 | **LOW** | Navigation Paths | Several navigation paths use legacy menu names (e.g., "production information management" instead of "Product information management") -- inconsistent casing and naming | [General](#general-observations) |
| 18 | **LOW** | Cost Group Setup | Product creation references "specify a cost group" but no guidance on cost group selection criteria or how cost groups roll up in cost calculations | [Product Creation](#3e---product-creation) |
| 19 | **LOW** | Warehouse Structure | Training references Main warehouse and Prod-In location but does not map to the FRD's 5-warehouse structure (1000, 1001, 1002, 2001, 2002) -- terminology mismatch risk | [General](#general-observations) |
| 20 | **LOW** | Security and Role-Based Access | No mention of security roles, duty assignments, or role-based access restrictions for any production process -- users have no guidance on who can perform which actions | [General](#general-observations) |

**Totals:** 3 CRITICAL | 5 HIGH | 8 MEDIUM | 4 LOW

---

## Executive Summary

This Training Manual (V5.0) covers Technica's Production Control module across six major sections: configuration (resources, calendars, products, routes, BOMs), production processes (MRP firming through production floor execution), revision handling, dismantling, packing/containerization, and reporting. The manual is **operationally detailed** for the processes it covers, with step-by-step screenshots and clear navigation paths for day-to-day production operations.

However, the manual has **significant gaps when compared to the FRD05 Production design**. The FRD defines 15 production processes (PRD001 through PRD015) including assembly, non-conformities, SNAGs, and notification workflows. The Training Manual effectively covers only the manufacturing production lifecycle (PRD003-PRD012) and omits assembly-specific processes, SNAG management, structured non-conformance handling, and all Power Automate notification workflows. This means **approximately 40% of the FRD-defined production scope has no corresponding training material**.

The manual is strongest in its coverage of the core production order lifecycle -- from MRP firming through shop floor execution to report-as-finished and order ending. The Gantt chart scheduling, cost variance analysis, and revision case handling are practical and well-documented. The manual is weakest in its coverage of custom/GAP functionality, cross-module integration points, and exception handling scenarios.

Overall assessment: **Solid training foundation for core manufacturing operations, but requires substantial expansion to cover the full FRD scope, custom processes, and exception workflows before it is ready for user training.**

---

## Process-by-Process Review

### 3a - Resource Group

**What is documented:**
- Navigation to Production Control > Setup > Resources > Resource Groups
- Creating a new resource group with name, description, site
- Linking to production unit
- Defaulting setup and run time cost categories
- Specifying calendar and adding resources with expiration dates
- Ledger postings note (to be provided by finance)

**Assessment: Adequate for basic setup**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Navigation path | Correct | Standard D365 path |
| Field-level guidance | Good | Covers key fields including cost categories and calendar |
| Resource expiration | Good | Important detail for capacity planning accuracy |
| Ledger postings | Incomplete | States "need to be provided by finance" with no further guidance |

The resource group setup is well-covered. The note about ledger postings being provided by finance is appropriate for a training manual -- this is typically a one-time configuration activity.

---

### 3b - Resources

**What is documented:**
- Navigation to Production Control > Setup > Resources > Resources
- Resource types: Machine, Human Resources, Vendor
- Operation fast tab: run time cost category, efficiency, scheduling percentage, finite capacity
- Calendar assignment
- Resource group linking
- Ledger postings and financial dimensions

**Assessment: Good foundation but missing capability configuration**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Resource types | Correct | Machine, Human Resources, Vendor are standard types |
| Operation tab fields | Good | Covers key scheduling parameters |
| Finite capacity checkbox | Important | Correctly highlighted -- critical for scheduling accuracy |
| Calendar assignment | Good | Properly explained |
| Resource capabilities/skills | **Missing** | Not documented |

**>> ATTENTION AREA: Resource capabilities and skills are not documented**

> D365 supports resource capabilities and skills that enable automatic resource selection during job scheduling. If Technica uses multiple machines with different capabilities (e.g., different bending capacities, CNC types), the capability setup is essential for the scheduler to select the correct resource. Without this, manual resource assignment on every route is required.

**>> ENHANCEMENT:** Add documentation for resource capabilities (Production Control > Setup > Resources > Resource capabilities) if Technica plans to use automatic resource selection. If all resource assignment is manual via route specification, document that decision explicitly so users understand why.

---

### 3c/3d - Calendar Management

**What is documented:**
- General calendar: Organization Administration > Setup > Calendars
- Updating working times and closing dates
- Resource-specific calendar deviation: Production Control > Setup > Resources > Resources > Calendar Deviation

**Assessment: Well-documented**

The distinction between general calendar updates and resource-specific deviations is clearly explained. This is a common source of user confusion, and the manual handles it well. The FRD (PRD007) references calendar configuration as critical for scheduling accuracy, and the training manual addresses this appropriately.

---

### 3e - Product Creation

**What is documented:**
- Full released product creation workflow (28 steps)
- Product type, subtype, item model group (Weighted Average)
- Item group codes table (CRM, SFP, FIP, FAC, OEM, INS, PAF, SER, DIG, SUB)
- Storage dimension group (SiteWHLoc)
- Tracking dimension group (None -- serial may be used later)
- Coverage group setup (Requirement vs. Min/Max)
- Cost group, product categories, attributes
- Product dimensions and variants
- Default order settings

**Assessment: Comprehensive product setup but with notable concerns**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Step-by-step creation | Excellent | Very detailed 28-step process |
| Item group table | Excellent | Clear mapping of codes to groups |
| Item model group | Correct | Weighted Average is appropriate |
| Storage dimension group | Correct | SiteWHLoc for warehouse-managed items |
| Tracking dimensions | **>> NEEDS REVIEW <<** | "None" with serial deferred |
| Coverage group | Good | Requirement vs. Min/Max well explained |
| Attribute setup | Good | Includes procurement category linkage |

**>> ATTENTION AREA: Tracking dimension group deferred decision**

> The manual states "Tracking dimension group: None; Serial might be used in a later stage." This is a significant architectural decision that is being deferred. Changing the tracking dimension group after items have transactions is extremely disruptive in D365 -- it requires item migration or new item creation. If Technica needs serial tracking for quality traceability, warranty management, or regulatory compliance, this must be decided before go-live, not deferred.

**>> ENHANCEMENT:** Resolve the serial tracking decision before training rollout. If serial tracking is required for finished goods (likely for manufactured equipment), configure the tracking dimension group as "Serial" with "Active in inventory process" or "Active in sales process" depending on the traceability requirement. Document the decision rationale in the training manual so users understand why serial is or is not required.

**>> ATTENTION AREA: Cost group lacks selection guidance**

> Step 11 says "specify a cost group" but provides no guidance on which cost group to select for different item types. Cost groups drive cost rollup calculations and variance analysis. Incorrect cost group assignment will produce incorrect cost reports.

**>> ENHANCEMENT:** Add a mapping table showing which cost group corresponds to which item group (e.g., CRM items use "Material" cost group, SFP items use "Semi-Finished" cost group, etc.).

---

### 3f - Routes

**What is documented:**
- Route creation and numbering
- Operation sequencing with Next operation
- Route group assignment (Discrete Manufacturing)
- Cost categories (setup and run)
- Time entry (setup time and run time)
- Resource requirements (resource group, specific resource)
- Simultaneous/parallel operations (secondary priority)
- Route approval and version approval/activation

**Assessment: Excellent route documentation**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Operation sequencing | Excellent | Next operation field clearly explained |
| Parallel operations | Excellent | Secondary priority mechanism well documented |
| Route group | Good | Single route group (Discrete Manufacturing) is appropriate for Technica |
| Approval workflow | Excellent | Two-level approval (route + version) clearly distinguished |
| Resource requirements | Good | Resource group as "most used type" is correct guidance |
| Bottleneck guidance | Excellent | "Put bottleneck operation as primary" is a best practice |

The route documentation is one of the strongest sections. The explanation of the two-level approval process (route approval for correctness, version approval for item assignment) is particularly well done. The guidance on parallel operations and bottleneck scheduling is a genuine best practice that many training manuals miss.

**>> ATTENTION AREA: FRD route selection GAP not reflected in training**

> The FRD (PRD004) identifies a GAP for route selection based on product family (PRD004-02). The training manual documents only manual route creation and version assignment. If the standard route version selection rules (by date, quantity, configuration) are configured per the FRD recommendation, users need to understand how the system automatically selects the correct route version.

**>> ENHANCEMENT:** Add a section explaining route version selection logic -- how D365 selects the active route version based on date validity, quantity ranges, and configuration. This is critical for users to understand why the system may select a different route than expected.

---

### 3g - BOMs

**What is documented:**
- BOM creation with automatic numbering
- BOM version assignment to specific item
- BOM line entry (components, raw materials, semi-finished products)
- Line type: Pegged Supply
- Two-level BOM approval (BOM approval + version approval/activation)
- Copy BOM function
- BOM Tree / BOM Designer
- Where Used inquiry
- Item BOM vs. Production BOM distinction

**Assessment: Good BOM coverage with one significant inaccuracy**

| Aspect | Rating | Comment |
|--------|--------|---------|
| BOM creation workflow | Good | Clear step-by-step |
| Two-level approval | Excellent | Clearly distinguishes BOM approval from version activation |
| MRP dependency on activation | Excellent | Critical point well explained |
| BOM Designer / Where Used | Good | Useful inquiry tools documented |
| Item BOM vs. Production BOM | Excellent | Common confusion point well addressed |
| Line type guidance | **Incorrect** | See below |

**>> ATTENTION AREA: BOM line type "Pegged Supply" should not be the default for all lines**

> The manual states: "The line type needs to be pegged supply. Pegged supply is used when a component is not normally stocked and must be procured or produced only when needed." While the definition is correct, instructing users to use Pegged Supply for ALL BOM lines is incorrect for a manufacturing environment where some raw materials are stocked (Min/Max coverage). Pegged Supply creates a 1:1 linkage between the parent order and the component supply, which is correct for make-to-order sub-assemblies but NOT correct for stocked raw materials.

**>> ENHANCEMENT:** Clarify BOM line type selection:
- **Item (default):** Use for stocked raw materials that are replenished via Min/Max or standard MRP coverage
- **Pegged Supply:** Use for make-to-order sub-assemblies and components that should only be procured/produced when the parent order requires them
- **Vendor:** Use for subcontracted operations

This distinction is critical for MRP behavior. Using Pegged Supply on stocked materials will generate unnecessary planned orders tied to individual production orders instead of consolidated replenishment.

---

### 4a - Firming and Processing Production Orders (Shop Floor Execution)

**What is documented:**
- Firming planned production orders from MRP (filtered by project ID)
- Multi-level pegging and explosion views (detailed example with Main Assembly, Subassembly 1, Subassembly 2)
- Requirement quantity vs. covered quantity interpretation
- Cost calculation details
- Operations scheduling then Job scheduling
- Job scheduling options (forward/backward, finite capacity/material, references)
- Gantt chart usage (drag operations, per-resource view, multi-order view)
- Release to warehouse and work creation
- Production Floor Execution login and job management
- Start job, report progress, report scrap
- **Custom: justification reason when job exceeds estimated time**
- Picking list journal posting (with actual consumption adjustment)
- Report as Finished (good quantity and error quantity)
- End production order
- Cost variance analysis (estimated vs. realized)

**Assessment: This is the strongest section of the entire manual**

The 31-step production order lifecycle is thoroughly documented with an excellent worked example using Main Assembly 1, Subassembly 1, and Subassembly 2. The multi-level pegging walkthrough (Level 0, Level 1, Level 2) with requirement vs. covered quantity analysis is an outstanding training technique that will help users understand MRP logic.

| Aspect | Rating | Comment |
|--------|--------|---------|
| MRP firming | Excellent | Project ID filtering, pegging views |
| Multi-level BOM example | Outstanding | Best section of the manual |
| Scheduling options | Excellent | Forward/backward, finite capacity, reference sync |
| Gantt chart | Excellent | Drag-and-drop, per-resource view, multi-order |
| Warehouse release | Good | Work creation and printout |
| Shop Floor Execution | Good | Start/report/complete flow |
| Picking list posting | Good | Actual consumption adjustment |
| RAF and End | Good | Error quantity and MRP re-run |
| Cost variance | Good | Estimated vs. realized comparison |

**>> ATTENTION AREA: Custom justification prompt is undocumented as a customization**

> Step 23 states: "When the job is completed click the checkmark. If time spent on the operation exceeds the estimated time, an error will appear asking user to please fill justification reason." This is NOT standard D365 behavior. The standard Shop Floor Execution app does not prompt for a justification reason when time is exceeded. This is a **custom development** (likely related to FRD GAP PRD010-02) that is presented in the training manual as if it were standard functionality.

**>> ENHANCEMENT (CRITICAL):** This custom justification prompt must be:
1. **Cross-referenced to the FRD GAP** (PRD010-02: Notification when job exceeds estimated time) to ensure the training material and the technical specification are aligned
2. **Documented as a customization** so that future system upgrades are aware this is non-standard behavior
3. **Tested thoroughly** since it modifies the production floor execution interface which is a Microsoft first-party app with limited extension points

**>> ATTENTION AREA: Reference number linkage explanation is valuable but needs emphasis**

> The explanation that sub-assemblies have reference type "Production Line" and main assemblies have reference type "Sales Order" is an important traceability concept. This is correctly documented and should be highlighted more prominently as it is key to understanding how the entire production structure links back to the project/sales order.

---

### 4b - Create a Manual Production Order

**What is documented:**
- Manual production order creation for standard and rework items
- Item number, warehouse, quantity, delivery date
- BOM and route selection (manual or automatic)
- Production pool selection (Non-conformity, customer request, snags from site, factory rework)
- Project ID linkage
- Estimation, scheduling, and release

**Assessment: Good workflow but pool guidance is missing**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Creation steps | Good | Clear and complete |
| Pool types | Listed | But no selection guidance |
| Project linkage | Good | Critical for Technica's project-centric model |
| Rework mention | Brief | "The same applies when creating a production order for items that are reported as rework" |

**>> ATTENTION AREA: Production pool selection lacks decision criteria**

> The manual lists four pool types (Non-conformity, customer request, snags from site, factory rework) without explaining when each should be used, who decides, or how the pool selection affects downstream reporting and filtering. The FRD (PRD015) defines SNAG categories with specific manufacturing vs. design distinctions that should map to these pools.

**>> ENHANCEMENT:** Add a decision matrix:
- **Non-conformity:** Use when a quality issue is detected during manufacturing (maps to PRD013)
- **Customer request:** Use when the customer requests a change after production has started
- **Snags from site:** Use for issues discovered during on-site installation/assembly (maps to PRD015 post-execution SNAGs)
- **Factory rework:** Use for internal rework of manufacturing defects (maps to PRD015 pre-execution SNAGs)

Include guidance on how these pools appear in production reports and how management uses them for quality KPI tracking.

---

### 4c/4d/4e - BOM and Route Modifications on Production Orders

**What is documented:**
- Adding/removing BOM lines on a production order
- Re-estimation and rescheduling requirements after BOM changes
- BOM changes must occur before warehouse release
- Copying BOM from another item (requires Created status, reset status if needed)
- Adding/modifying route operations on a production order
- Copying route from another item (requires Created status)

**Assessment: Practical and well-structured**

The guidance on modifying BOMs and routes on production orders is excellent. Key points are correctly emphasized:
- Re-estimation after BOM changes (to recalculate material requirements)
- Rescheduling after route changes (to recalculate resource bookings)
- Status requirements for copy operations (Created status, with reset status option)

This section aligns well with the FRD's revision case requirements and provides the practical "how-to" that users need.

---

### 4f - Removing an Already Produced Item (Stock Adjustment)

**What is documented:**
- Inventory adjustment journal for removing subassemblies
- Navigation: Inventory Management > Journal Entries > Inventory Adjustment
- Journal line entry: item number, warehouse, quantity, cost price
- Posting the journal

**Assessment: Correct but incomplete**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Journal type selection | Correct | Inventory Adjustment is appropriate |
| Navigation | Correct | Standard path |
| Steps | Adequate | Basic journal entry flow |
| Financial impact explanation | Good | Cost posting to GL based on item group profile |
| Approval workflow | Missing | No mention of journal approval before posting |

**>> ENHANCEMENT:** Add guidance on journal name selection and whether journal approval is required before posting. In most D365 implementations, inventory adjustment journals require supervisor approval to prevent unauthorized stock changes.

---

### 4g - BOM Journal for Dismantling

**What is documented:**
- Creating products for dismantled parts
- Creating a new BOM for the main assembly containing dismantled items
- Setting selling prices to sum to total original price
- Negative Report-as-Finished to stock out the main product
- Auto-created BOM journal for inventory adjustment
- Posting the BOM journal to remove main assembly and add dismantled items

**Assessment: Functional but uses a non-standard approach**

**>> ATTENTION AREA: Negative RAF for dismantling is unconventional**

> The described process uses a negative quantity Report-as-Finished to reverse the main assembly out of inventory, then relies on the auto-created BOM journal to add the dismantled components. While this achieves the financial result, it is not the standard D365 approach for disassembly. D365 supports **BOM journal type** specifically for disassembly (report-as-finished with negative quantity on the BOM journal), but the more standard approach for a full disassembly process would be to use a **production order with a disassembly BOM** (where the finished good is the input and the components are the outputs).

**>> ENHANCEMENT:** Document the reason for choosing this approach over a standard disassembly production order. If Technica specifically requires the negative RAF method due to cost allocation needs (selling price distribution across dismantled items), document that business justification. This helps future consultants understand why the non-standard approach was chosen.

---

### 4h - Release Sales Order, Packing and Containerization

**What is documented:**
- BOM line explosion on sales order (for dismantled items)
- Release to warehouse
- Shipment ID retrieval (from sales order line or Active Shipments screen)
- Packing and containerization process (new containers, item packing, container closing, release)
- Weight specification on container close

**Assessment: Well-documented cross-module process**

| Aspect | Rating | Comment |
|--------|--------|---------|
| BOM explosion | Good | Clear distinction of when it applies |
| Warehouse release | Good | Standard process |
| Shipment ID location | Excellent | Two methods documented (SO line, Active Shipments) |
| Containerization | Good | Step-by-step with open lines, container creation, packing |
| Container close/release | Good | Weight capture included |

This section bridges Production Control and Warehouse Management effectively. The packing and containerization workflow is typically documented in a Warehouse Management training manual, so its inclusion here provides valuable end-to-end process context for production users.

**>> ATTENTION AREA: This section may belong in a Warehouse Management training manual**

> Including sales order release, packing, and containerization in the Production Control manual may create confusion about process ownership. Consider cross-referencing to a Warehouse Management training manual rather than duplicating content.

---

### 4i - Returning Unconsumed Raw Material (Transfer Journal)

**What is documented:**
- Transfer journal for moving unused materials from production area to main warehouse
- Navigation: Inventory Management > Journal Entries > Transfer
- Journal line entry with from/to site and warehouse
- Mention of a "Remaining items report" that needs to be developed

**Assessment: Contains a critical from/to reversal error**

**>> ATTENTION AREA: Transfer journal from/to fields are reversed in the instructions**

> The narrative correctly states the intent: "moving from the Prod-In location, which is the production area to BULK location, which is the main warehouse." However, steps 11-12 state:
> - Step 11: "In the location to, select the production floor (Prod-In)"
> - Step 12: "In the location from, select BULK"
>
> This is **reversed**. If the user follows these steps literally, they will transfer material FROM the main warehouse TO the production floor -- the opposite of the intended action. The correct instructions should be:
> - "In the location from, select the production floor (Prod-In)"
> - "In the location to, select BULK"

**>> ENHANCEMENT (HIGH):** Correct the from/to field assignments in steps 11-12 to match the stated intent. Also add a verification step: after posting the transfer journal, instruct users to check on-hand inventory at both locations to confirm the transfer moved material in the correct direction.

**>> ATTENTION AREA: "Remaining items report" is flagged as needing development**

> Step 2 states: "Remaining items report should be developed to reflect the on-hand remaining quantities of a specific project." This is a custom report requirement embedded in a training manual. It should be tracked as a development item, not buried in training documentation.

**>> ENHANCEMENT:** Cross-reference this report requirement to the FRD or a development backlog. In the interim, document how users can use the standard D365 "On-hand inventory" inquiry filtered by warehouse/location to achieve similar results.

---

### 5 - Revision Cases

**What is documented:**
- Case A: BOM revision before firming (re-run MRP)
- Case B: New BOM revision before ending production order (two options: new picking list journal OR add to production BOM + MRP)
- Case C: Fault during production (error quantity on RAF + re-run MRP)
- Case D: New BOM revision after ending (new production order with rework pool)
- Case E: Removing an item from BOM (new BOM version)

**Assessment: Excellent exception handling documentation**

This is one of the most valuable sections of the training manual. Revision cases are where users make the most mistakes, and this section provides clear decision trees for five distinct scenarios.

| Scenario | Rating | Comment |
|----------|--------|---------|
| Pre-firming revision | Excellent | Simple and correct -- re-run MRP |
| Mid-production addition | Excellent | Two clear options with different trade-offs |
| Fault/error quantity | Good | RAF error quantity + MRP re-run is correct |
| Post-end revision | Good | New production order with rework pool |
| BOM line removal | Good | Clear and concise |

**>> ATTENTION AREA: Strikethrough text remains in the document**

> The section contains strikethrough text: "~~Ensure to select the production orders of the lowest subassemblies with ready BOMs and don't firm the main assembly production order since its BOM is not ready yet.~~" This is draft markup that should be removed from a final training document. Its presence suggests the document has not undergone final editorial review.

**>> ENHANCEMENT:** Remove all strikethrough text from the final training manual. If the deleted guidance is still relevant as a consideration, rephrase it as a "Note" or "Important" callout rather than leaving crossed-out text.

**>> ATTENTION AREA: Case D (post-end rework) lacks consumed item handling detail**

> Case D mentions consuming the damaged item "by adding it to the new BOM or a stock adjustment is required" but does not provide step-by-step instructions for either approach. Given that this is an exception scenario where users are most likely to make errors, more detailed guidance is needed.

---

### 6 - Production Control Reports and Workspaces

**What is documented:**
- Production Floor Management workspace (job status, deviation calculation)
- Productions Started inquiry
- Current Production Jobs inquiry
- Delayed Production Orders inquiry
- Cost Estimates and Costing report
- Overview Costing report
- Capacity Reservation screen and report
- Production Overview report

**Assessment: Good coverage of available reports but lacks operational guidance**

| Report/Workspace | Rating | Comment |
|-----------------|--------|---------|
| Production Floor Management | Good | Deviation calculation formula included |
| Productions Started | Basic | Navigation only |
| Current Production Jobs | Basic | Navigation only |
| Delayed Production Orders | Basic | Navigation only |
| Cost Estimates and Costing | Good | Purpose well explained |
| Overview Costing | Good | Brief but adequate |
| Capacity Reservation | Good | Purpose well explained |
| Production Overview | Good | Status descriptions included |

**>> ATTENTION AREA: Reports section lacks step-by-step generation instructions**

> The reports section describes what each report shows and its business purpose, but provides no step-by-step instructions for generating, filtering, exporting, or scheduling any of the reports. For a training manual, users need to know HOW to run the report, not just WHAT it contains.

**>> ENHANCEMENT:** For each report, add:
1. Step-by-step instructions to generate the report
2. Key filter parameters and recommended filter values
3. How to export to Excel (standard D365 export functionality)
4. Recommended frequency (daily for production floor management, weekly for cost reports, etc.)

The deviation calculation formula (|Estimated - Actual| / Estimated) is a valuable inclusion and should be kept.

---

## General Observations

### Navigation Path Inconsistencies

Several navigation paths contain casing or naming inconsistencies:
- "production information management" should be "Product information management" (Step in Where Used section)
- "production control > setup > resources> resources" -- inconsistent spacing around ">"
- Some paths use ">" while the narrative uses different separators

**>> ENHANCEMENT:** Standardize all navigation paths to use consistent casing matching the D365 interface (e.g., "Production control > Setup > Resources > Resources") and consistent spacing.

### Warehouse Structure Terminology

The training manual references "Main warehouse," "Prod-In," and "BULK" locations. The FRD defines a 5-warehouse structure (1000 Main, 1001 Mfg In, 1002 Mfg Out, 2001 Assembly In, 2002 Assembly Out). The terminology should be aligned to prevent confusion during training.

### Security Roles Not Addressed

The training manual does not mention security roles or access restrictions for any process. Users should understand which D365 security role grants access to each function (e.g., Production Manager, Production Clerk, Shop Floor Operator) and what actions each role can perform.

**>> ENHANCEMENT:** Add a role-permission matrix at the beginning of the manual or at the start of each major section, identifying which security role(s) can perform the documented actions.

---

## Cross-Reference with FRD

The following table maps each FRD05 process to its coverage in the Training Manual:

| FRD Process | FRD ID | Training Manual Coverage | Gap Assessment |
|-------------|--------|--------------------------|---------------|
| Production Lifecycle & 3D Drawing | PRD001-002 | **Not covered** | **CRITICAL GAP** -- FRD defines 5 GAPs including CAD integration, drawing release notifications, rework hold, and rework counter. None documented in TM. |
| Bill of Material | PRD003 | **Well covered** (Section 3g) | Good alignment. BOM creation, approval, activation, tree view, where-used all documented. |
| Routes | PRD004 | **Well covered** (Section 3f) | Good alignment. Route creation, approval, version activation documented. PRD004-02 (product family route selection) not addressed. |
| PO Tracking & Order Generation | PRD005-006 | **Partially covered** (Section 4a) | MRP firming and planned order processing covered. Open PO tracking for inbound material visibility not documented. Cross-order notification (mfg complete to assembly) not covered. |
| Schedule Production Orders | PRD007 | **Well covered** (Section 4a, steps 6-12) | Excellent. Operations scheduling, job scheduling, Gantt chart, forward/backward scheduling all documented. |
| Release Production Order | PRD008 | **Covered** (Section 4a, step 14) | Release process documented. Component availability check (checkmark system) from FRD not explicitly shown. Production Floor Management dashboard referenced in Section 6. |
| Resource Consumption | PRD009 | **Partially covered** (Section 4a, steps 15-16) | Warehouse work creation and picking documented. Transfer order flow between warehouses (Main to Mfg) not documented as a separate process. |
| Splitting Production Order | PRD010 | **Not covered** | FRD recommends updating quantity rather than splitting. Manual does not address this scenario. |
| Shop Floor Execution | PRD011 | **Well covered** (Section 4a, steps 17-25) | Good alignment. Job start, progress reporting, scrap reporting documented. Custom justification prompt included but not flagged as customization. |
| Report As Finished | PRD012 | **Covered** (Section 4a, steps 27-29) | RAF with good and error quantity documented. License plate entry documented. |
| Non-Conformities | PRD013 | **Minimally covered** | FRD defines structured non-conformance process with scrap cost tracking, rework via WBS activities, and CAD re-integration. TM only covers error quantity on RAF. No Quality Management module coverage. |
| Assembly | PRD014 | **Not covered** | **CRITICAL GAP** -- FRD defines assembly-specific processes including test strategy meetings, manual route selection, POC% tracking, and assembly start before all parts ready. None in TM. |
| SNAGs | PRD015 | **Not covered** | **HIGH GAP** -- FRD defines SNAG management via WBS activities, SNAG categories, case management, and ECR linkage. No SNAG coverage in TM. |
| Notifications (5 GAPs across FRD) | Various | **Not covered** | **HIGH GAP** -- FRD defines Power Automate notifications for drawing release, rework, job time exceedance, manufacturing completion, and cross-order alerts. None documented. |
| Setup - Full Reservation | Setup | **Not covered** | **HIGH GAP** -- FRD flags "Require Full Reservation = TRUE" as CRITICAL risk conflicting with assembly start. TM does not address reservation parameters. |
| Setup - Warehouse Structure | Setup | **Partially covered** | TM references Main warehouse and Prod-In location. FRD defines 5-warehouse structure. Terminology not aligned. |

### Key Cross-Reference Findings

**>> ATTENTION AREA: Assembly process (PRD014) has zero training coverage**

> The FRD defines assembly as a distinct process with different characteristics from manufacturing: test strategy meetings determine assembly routes, assembly managers manually select routes, assembly can start before all manufactured parts are complete, and POC% tracking is required. The Training Manual does not distinguish between manufacturing and assembly production orders at all. Users performing assembly operations will have no training material.

**>> ENHANCEMENT (CRITICAL):** Create a dedicated assembly section in the Training Manual covering:
1. How assembly production orders differ from manufacturing orders
2. Test strategy meeting output and route assignment
3. Starting assembly with partial component availability
4. POC% tracking (whether via partial RAF or custom mechanism per FRD review)
5. Assembly-specific reporting

**>> ATTENTION AREA: SNAG management (PRD015) has zero training coverage**

> SNAGs are tracked via WBS activities with specific categories (manufacturing vs. design), case management for workflow, and linkage to Engineering Change Requests. The production pool types listed in the manual (snags from site, factory rework) suggest SNAG functionality exists in the system, but no process documentation is provided.

**>> ENHANCEMENT (HIGH):** Create a SNAG management section covering:
1. When and how to create a SNAG production order (pre-execution vs. post-execution)
2. SNAG category selection (manufacturing vs. design)
3. Case management workflow for SNAG approval
4. Linking SNAGs to WBS activities for cost tracking
5. ECR linkage for design-related SNAGs

**>> ATTENTION AREA: CAD integration user procedures are missing**

> The FRD identifies CAD integration (from Inventor) as the single biggest production risk, affecting BOM population, drawing URL storage, and rework triggers. The Training Manual mentions in passing that "The BOM and BOM versions will be integrated into D365 from Inventor" but provides no user procedures for:
> - Triggering or verifying a CAD import
> - Validating imported BOM data
> - Handling import errors
> - Understanding when manual BOM entry is required vs. when CAD integration handles it

**>> ENHANCEMENT:** Once CAD integration is developed, add a dedicated section covering the user's role in the CAD-to-D365 data flow. Even if the integration is automated, users need to understand how to verify imported data and what to do when the integration fails.

**>> ATTENTION AREA: Quality Management module not referenced**

> The FRD review recommends using D365 Quality Management for non-conformance tracking instead of the informal approach described. The Training Manual does not reference Quality Management at all -- no quality orders, no non-conformance records, no corrective actions. If Quality Management is implemented per the FRD recommendation, training material is needed.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | Shop Floor Execution | Document the custom justification prompt as a customization with FRD GAP cross-reference (PRD010-02); ensure technical specification exists | CRITICAL |
| 2 | Assembly Process | Create complete assembly section covering test strategy meetings, manual route selection, POC% tracking, and partial component start | CRITICAL |
| 3 | Non-Conformities | Expand non-conformance coverage beyond basic error quantity to include structured workflow, Quality Management module, and rework process | CRITICAL |
| 4 | SNAG Management | Create SNAG section covering pre/post-execution SNAGs, categories, case management, WBS linkage, and ECR linkage | HIGH |
| 5 | Notification Workflows | Document all 5 Power Automate notification workflows once developed (drawing release, rework, time exceedance, mfg completion, cross-order) | HIGH |
| 6 | Transfer Journal | Correct the from/to field reversal in steps 11-12 of the transfer journal section | HIGH |
| 7 | Production Pools | Add decision matrix for pool selection with downstream reporting impact | HIGH |
| 8 | Reservation Parameters | Document the reservation strategy and its impact on production order release, aligned with FRD resolution of the Full Reservation conflict | HIGH |
| 9 | Tracking Dimensions | Resolve serial tracking decision before training rollout; document rationale | MEDIUM |
| 10 | BOM Line Types | Correct the blanket "Pegged Supply" guidance; add line type selection criteria for different item types | MEDIUM |
| 11 | Dismantling Process | Document business justification for negative RAF approach vs. standard disassembly production order | MEDIUM |
| 12 | Strikethrough Text | Remove all draft markup from the final training document | MEDIUM |
| 13 | CAD Integration | Add user procedures for CAD import verification, error handling, and manual fallback once integration is developed | MEDIUM |
| 14 | Quality Management | If QM module is implemented per FRD recommendation, create training material for quality orders and non-conformance records | MEDIUM |
| 15 | Reports Section | Add step-by-step report generation instructions, filter guidance, and recommended run frequency for each report | MEDIUM |
| 16 | Resource Capabilities | Document resource capabilities/skills setup if automatic resource selection is used | MEDIUM |
| 17 | Navigation Paths | Standardize all navigation paths to match D365 interface casing and formatting | LOW |
| 18 | Cost Group Guidance | Add cost group selection mapping table by item type | LOW |
| 19 | Warehouse Terminology | Align warehouse naming between TM and FRD (Main/Prod-In vs. 1000/1001/1002/2001/2002) | LOW |
| 20 | Security Roles | Add role-permission matrix identifying which security roles can perform each documented action | LOW |

---

## Risk Flags

1. **Assembly training gap is the highest risk.** The FRD defines assembly as a fundamentally different process from manufacturing (different route selection, partial component start, POC% tracking), yet the Training Manual makes no distinction. Users performing assembly operations will apply manufacturing procedures, leading to process errors and scheduling conflicts. This directly compounds the FRD's CRITICAL finding about the Full Reservation parameter conflicting with partial assembly start.

2. **Custom Shop Floor Execution behavior is undocumented as a customization.** The justification prompt when job time exceeds estimates modifies Microsoft's first-party Shop Floor Execution app. If this customization is not properly documented, it will be lost during D365 upgrades and the training material will become inaccurate. This is a supportability risk.

3. **Transfer journal from/to reversal will cause incorrect inventory movements.** If users follow the literal steps 11-12, they will move material in the wrong direction (from main warehouse to production floor instead of the reverse). This is a data integrity risk that should be corrected immediately.

4. **40% of FRD scope has no training material.** Assembly (PRD014), SNAGs (PRD015), non-conformity workflow (PRD013), CAD integration (PRD001-002), and notification workflows are all defined in the FRD but absent from the Training Manual. Users will not be trained on these processes unless additional training material is created.

5. **Tracking dimension decision deferral creates post-go-live risk.** If serial tracking is needed and not configured before go-live, retroactively adding it requires item migration -- one of the most disruptive changes in D365. This architectural decision must be finalized before training rollout.

6. **Strikethrough text and draft markup indicate the document has not undergone final editorial review.** A V5.0 document should not contain tracked-change artifacts. This raises concerns about whether other content has been reviewed and approved.

7. **"Remaining items report" development requirement is buried in training material.** Custom report requirements should be tracked in the development backlog, not embedded in user documentation. There is a risk this requirement will be missed during development planning.

---

*This review is based on D365 F&O Production Control, Manufacturing Execution, and Shop Floor Execution best practices as of 2026. Recommendations aim to close FRD-to-training gaps, correct procedural inaccuracies, and ensure comprehensive user training coverage.*
