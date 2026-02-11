# FRD009 - Inventory Management - Solution Design Verification Report

**Document:** FRD009 - Inventory Management - V2.0
**Prepared by:** Edward Khoury | Contributors: Abdo Khoury, Jalal Iskandar
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Inventory Management, Warehouse Management (WMS), Product Information Management

---

## Executive Summary

This FRD covers inventory management across product setup, warehouse structure, component stock keeping, raw materials stock keeping, head of stock operations, WMS mobile device operations, transfer orders, item movements, stock counting, and inventory adjustments. The document addresses a **make-to-order manufacturing environment** where inventory is heavily project-driven -- items are procured, reserved, picked, consumed, and returned against specific projects.

The solution makes **reasonable use of D365 standard inventory features** but has several areas of concern:

1. **Multiple unresolved reviewer comments** (EA, EK) indicate requirements are still being debated -- notably around spare parts scope, ordering process alignment with Procurement FRD07, and product dimension clarity.
2. **Four identified GAPs**, of which **two may not be true GAPs** (IN02-001 and IN04-002 have OOB or near-OOB alternatives).
3. **The raw materials handling process** (linear meter tracking, item splitting, scrap piece tracking) is the most complex area and requires careful design to avoid over-customization.
4. **WMS is enabled** but only for basic receiving and put-away -- this is appropriate for the current maturity level but the document should clarify which warehouses are WMS-enabled vs. basic.
5. **Quality Management module is explicitly excluded** ("Quality management module will not be used to avoid long cycles"), which is a risky decision for a manufacturing company that tracks non-conformances.

Overall assessment: **Functional design with standard D365 inventory foundations, but several GAPs are over-classified, raw material tracking needs tighter design, and the Quality Management exclusion should be reconsidered.**

---

## Process-by-Process Review

### 1. Product Setup (IN01-001 through IN01-005) -- All FIT

**What was proposed:**
- Storage Dimension Groups: Site, Warehouse, Location, Inventory Status
- Tracking Dimension Groups: Serial number tracking (some items), no batch tracking
- Product Dimension Group: Configuration and Size variants used
- Item Model Group: Weighted Average Method (WAM) for stock items; Standard Cost for finished products
- Category Hierarchy: Brand and Category required; full list during migration

**Assessment: Standard Product Information Management -- mostly sound**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Storage dimensions (Site/WH/Location/Status) | Standard OOB | Correct for WMS-enabled warehouses |
| Tracking by serial number | Standard OOB | Appropriate for high-value components |
| No batch tracking | Acceptable | Valid if no expiration/lot control needed |
| Configuration variant for conveyors | Standard OOB | Correct use of product masters |
| Size variant for tubes | Standard OOB | Good approach for tube size tracking |
| WAM costing for stock items | Standard OOB | Common for manufacturing |
| Standard Cost for finished products | Standard OOB | Correct for make-to-order with BOM costing |
| Inventory Status dimension | Standard OOB | Enables quality/hold status tracking without QM module |

**Recommendations:**

1. **Clarify which items need Location tracking and which do not.** The FRD states "some items will be tracked by Site, warehouse and location, and some others would not require Location tracking." This means **two Storage Dimension Groups** are needed -- one with Location active, one without. Ensure the naming convention makes this clear (e.g., "SDG-SiteWHLoc" vs. "SDG-SiteWH"). Storage dimension groups **cannot be changed after transactions are posted**, so getting this right during setup is critical.

2. **The unresolved comment about Color and Style dimensions** (EA1: "Not Clear! Color? Style? Are these considered as Dimension Groups?") should be formally closed. If only Configuration and Size are used, document that Color and Style are explicitly excluded to prevent confusion during item setup.

3. **Inventory Status dimension is a strong choice** -- it can serve as a lightweight quality gate (Available, QC Hold, Blocked, Non-Conformance) without implementing the full Quality Management module. This partially compensates for the QM exclusion. Ensure status values are defined early and mapped to business processes.

4. **Cross-FRD dependency (FRD05 - Production):** The Standard Cost for finished products must align with the BOM costing approach in Production FRD05 (PRD003). Ensure BOM cost rollup is configured to feed standard cost calculations.

---

### 2. Sites/Warehouses/Locations (Setup)

**What was proposed:**
- 1 Site (Lebanon)
- Multiple Warehouses:
  - WH Components/Spare Parts: 10 locations (Mechanical, Motors, Cables, Electrical, Pneumatic, Wood, Gaz, Supplies, Samples, Non-Moving)
  - WH Raw Material: 3 location groups (Mechanical, Supplies, Raw Material with sub-types)
- WMS enabled
- Warehouse Clerk registers goods; Stock Person receives
- Option for direct receiving to final location OR generating put-away work

**Assessment: Reasonable warehouse structure for a single-site operation**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Single site, multiple warehouses | Standard OOB | Simple and appropriate |
| Location-level tracking | Standard OOB | Required for WMS |
| RECV and BULK locations | Standard OOB | Standard WMS receiving pattern |
| Separate Components vs. Raw Material warehouses | Good Design | Clear material segregation |
| Samples and Non-Moving locations | Good Design | Proper disposition tracking |

**Recommendations:**

1. **Clarify which warehouses are WMS-enabled.** The FRD mentions WMS for receiving/put-away but does not specify which warehouses use advanced WMS vs. basic inventory. Recommendation: Enable **WMS only for the Components/Spare Parts warehouse** (high item count, many locations, mobile device usage) and keep **Raw Material warehouse on basic inventory management** (fewer SKUs, simpler flow). This reduces WMS configuration overhead.

2. **Cross-FRD dependency (FRD05 - Production):** Production FRD05 (PRD009) defines Manufacturing warehouses (1001 Mfg In, 1002 Mfg Out, 2001 Assembly In, 2002 Assembly Out). Ensure the transfer order flow from the inventory warehouses defined here to the production warehouses is mapped end-to-end. The FRD does not explicitly reference these production warehouses.

3. **Cross-FRD dependency (FRD06 - Logistics):** Logistics FRD06 uses an In-Transit warehouse for Landed Cost voyages. Ensure goods receiving in this FRD accounts for items arriving via voyage (Landed Cost) vs. local purchase. The receiving process described here (warehouse clerk registers, stock person receives) should specify whether it applies to both local POs and voyage-received POs.

4. **The reviewer comment about removing KSA and Egypt sites** (EA4) should be confirmed as resolved. The FRD only shows Lebanon, so this appears addressed, but ensure no legacy site references remain in configuration.

---

### 3. Component Stock Keeper (IN02-001 through IN02-007) -- 1 GAP

**What was proposed:**
- Anyone creates Purchase Requisition (PR) for materials
- PR approved via workflow, sent to Store Keeper
- Store Keeper checks availability, suggests alternatives
- Items reserved for specific projects from PR
- Goods receiving with quality check (pass/fail on device)
- Failed QC items go to non-conformance location
- Material requests for projects: check reservation, pick, deliver to production
- Master Planning generates planned orders; PM is requester
- Internal Delivery Order (IDO) for picking items from warehouse to production
- Stock out/in transactions, return from factory, reservation switching
- Min/Max reports, consumption reports, non-moving item tracking
- Spot checks (biweekly) and yearly inventory counts

**Assessment: Core inventory operations are well-designed, but the GAP classification needs review**

| Aspect | Rating | Comment |
|--------|--------|---------|
| PR-based procurement workflow | Standard OOB | Aligns with FRD07 Procurement |
| Project reservation from PR | **See GAP analysis** | IN02-001 flagged as GAP |
| Quality check on device (pass/fail) | Standard OOB | WMS quality check step or Inventory Status |
| Non-conformance location routing | Standard OOB | Location directives can route failed items |
| Material requests via item requirements | Standard OOB | Project item requirements |
| Master Planning for planned orders | Standard OOB | Aligns with FRD07 (PR002) |
| IDO / picking for production | Standard OOB | Production picking list / transfer order |
| Min/Max replenishment | Standard OOB | Item coverage settings |
| Reservation switching between projects | Standard OOB | Remove reservation + re-reserve |

#### GAP Analysis: IN02-001 -- Item Reservation from PR

| Attribute | Detail |
|-----------|--------|
| **GAP ID** | IN02-001 |
| **Description** | Reservation of items from Purchase Requisition is mandatory |
| **FRD Classification** | GAP |
| **Assessed Complexity** | Medium |
| **Is it truly a GAP?** | **Partially -- depends on interpretation** |

**Analysis:** In standard D365, reservation is typically done at the Purchase Order level or via Item Requirements on Projects, not directly from the Purchase Requisition. However, there are two important considerations:

1. **If the goal is to reserve existing on-hand stock when a PR is created for a project**, this can be achieved through **Project Item Requirements** instead of Purchase Requisitions. Item Requirements on projects allow reservation of on-hand inventory and generate planned purchase orders for shortages. This is the **standard D365 approach for project-driven material requests** and may eliminate this GAP entirely.

2. **If the goal is to reserve against incoming PO quantities triggered by the PR**, this is not standard. D365 supports marking (PR line to PO line) but not physical reservation against unreceived PO quantity.

**Recommendation:** Replace the PR-based reservation workflow with **Project Item Requirements** (available in Project Management and Accounting module). This provides:
- Automatic reservation of on-hand inventory
- Automatic generation of planned purchase orders for shortages (via Master Planning)
- Direct project cost linkage
- Standard D365 flow without customization

This aligns with the FRD's own statement: "PR can be done manually by the requesters for material items which will be linked to a specific project. Moreover, master planning will be used to automate the process." Item Requirements achieve exactly this.

**Cross-FRD dependency (FRD07 - Procurement):** If Item Requirements replace PRs for project materials, ensure FRD07's PR workflow (PR001) is updated to exclude project-linked material requests from the standard PR approval chain, since Item Requirements follow a different approval path.

---

### 4. Raw Materials Stock Keeper (IN03-001 through IN03-009) -- 2 GAPs

**What was proposed:**
- Receiving drawing material requests (BOM-driven)
- Picking raw materials with special handling for lengths (raw material may be longer than needed; remainder returned)
- Tracking consumption by Length x Width (square meters)
- Tracking linear meters of cut pieces (e.g., 6-meter pipe cut into smaller pieces)
- Tracking scrap pieces
- Checking alternative items from BOM
- Material Delivery Form for recording delivered quantities
- Item splitting without inventory adjustment

**Assessment: This is the most complex area of the FRD and needs the most design attention**

| Aspect | Rating | Comment |
|--------|--------|---------|
| UOM for square meters | Standard OOB | Unit conversion setup |
| Item movement between locations | Standard OOB | Movement journal or WMS movement |
| Transfer order between warehouses | Standard OOB | Standard |
| Alternative items in BOM | Standard OOB | BOM line substitution feature |
| Negative adjustment for scrap | Standard OOB | Inventory adjustment journal |
| Scrap items to specific warehouse | Standard OOB | Transfer to scrap warehouse/location |
| Min/Max report | Standard OOB | Standard coverage reporting |

#### GAP Analysis: IN03-001 -- Material Delivery Form

| Attribute | Detail |
|-----------|--------|
| **GAP ID** | IN03-001 |
| **Description** | Material delivery form to record quantities delivered to production per project |
| **FRD Classification** | GAP |
| **Assessed Complexity** | Low-Medium |
| **Is it truly a GAP?** | **Likely not a true GAP -- standard functionality may cover this** |

**Analysis:** The "Material Delivery Form" is described as a paper-based process where the warehouse records raw material usage per project (Length x Width in square meters). In D365, this function is served by:

1. **Production Picking List Journal** -- records what was picked from inventory for a production order (linked to project via the production order).
2. **Project Item Consumption Journal** -- records material consumption directly against a project.
3. **Transfer Order with project reference** -- moves inventory from warehouse to production with project linkage.

All three maintain a record of what was delivered, when, and for which project.

**Recommendation:** Use the **Production Picking List** as the system equivalent of the Material Delivery Form. If additional fields are needed (e.g., Length x Width calculation, cut piece tracking), add them as custom fields on the picking list journal line. This is a minor form extension (Low complexity), not a full custom form build.

Alternatively, if the Material Delivery Form needs to be a standalone document for warehouse internal use, create a simple **Power Apps canvas app** that reads from the picking list journal and displays it in the warehouse-friendly format.

#### GAP Analysis: IN03-009 -- Item Splitting Without Inventory Adjustment

| Attribute | Detail |
|-----------|--------|
| **GAP ID** | IN03-009 |
| **Description** | Split one inventory record into multiple records (e.g., 1x 6m pipe into 3x 2m pipes) without using inventory adjustment |
| **FRD Classification** | GAP |
| **Assessed Complexity** | Medium-High |
| **Is it truly a GAP?** | **Yes -- true GAP, but the proposed approach (avoiding adjustment journals) may be incorrect** |

**Analysis:** The scenario is: a 6-meter pipe is cut into smaller pieces for different projects. The warehouse needs to track each piece separately. The FRD mentions using "inventory adjustment journals" for this but also flags wanting to do it "without inventory adjustment."

This is a genuine process challenge. D365 does not have a native "split inventory" function. The standard approaches are:

1. **Inventory Adjustment Journal (current proposal in FRD text):** Negative adjustment for the original item, positive adjustments for the resulting pieces. The FRD notes "for the positive quantity, the correct cost amount must be specified." This works but creates artificial gain/loss postings.

2. **Product Variants (Size dimension):** If the pipe item uses Size as a product dimension (which the FRD already proposes for tubes), then cutting a 6m pipe into 2m pieces is a **variant conversion**. However, D365 does not support variant-to-variant conversion natively.

3. **BOM Journal / Production Order:** Create a simple "cutting" BOM where the input is 1x 6m pipe and the output is 3x 2m pipes. Process this as a mini production order. This maintains full cost traceability without artificial adjustments.

**Recommendation:** Use a **lightweight production order (cutting order)** with a single-level BOM:
- Input: 1x Item "Pipe SS" (Size: 6m)
- Output: 3x Item "Pipe SS" (Size: 2m) + scrap if applicable

This approach:
- Maintains cost flow without manual cost entry
- Uses standard D365 production costing (input cost flows to output)
- Creates a clear audit trail
- Handles scrap naturally (report scrap quantity on the production order)
- Avoids the need for manual cost specification on positive adjustments
- Integrates with the existing Production module (FRD05)

**Cross-FRD dependency (FRD05 - Production):** The cutting order approach requires defining cutting BOMs and potentially a dedicated "Cutting" resource/route. Coordinate with Production to include this in the route/BOM setup.

**Complexity assessment:** Medium if using the production order approach (BOM setup + simple route). The original approach (adjustment journal) is actually harder to manage correctly because of manual cost entry requirements.

---

### 5. Head of Stock Operations (IN04-001 through IN04-007) -- 1 GAP

**What was proposed:**
- Receives and forwards material requests
- Checks item availability and books on projects
- Quality and quantity check on receiving (without Quality Management module)
- Approves stock purchase requisitions
- Approves newly created items (workflow required)
- Stock level, consumption, aging, and slow-moving item reports
- Scrap management
- Daily follow-up on all stock transactions
- Task management for daily/monthly operations
- Item creation by Engineers, approval by designated person
- Attributes on items for search facilitation
- New items cannot be used until approved

**Assessment: Supervisory and governance layer -- mostly standard with one debatable GAP**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Bulk receiving from multiple POs | Standard OOB | PO arrival overview / Load receiving |
| On-hand list report | Standard OOB | On-hand inventory report |
| Inventory aging report | Standard OOB | Standard report |
| Slow-moving items report | Standard OOB | Inventory value report with date filters |
| Negative adjustment for scrap | Standard OOB | Adjustment journal with reason codes |
| Task management for operations | Standard OOB | Can use D365 Activity/Task or Planner integration |
| Item search by attributes | Standard OOB | Product attributes + attribute-based search |

#### GAP Analysis: IN04-002 -- Workflow Approval on Item Creation

| Attribute | Detail |
|-----------|--------|
| **GAP ID** | IN04-002 |
| **Description** | Workflow approval for newly created items -- item cannot be used until approved |
| **FRD Classification** | GAP |
| **Assessed Complexity** | Medium |
| **Is it truly a GAP?** | **Partially -- D365 has a product release workflow, but not a full item creation approval workflow** |

**Analysis:** The requirement is:
- Engineers create items
- A designated person reviews and approves (checks for duplicates, validates attributes)
- Item cannot be used in transactions until approved

D365 has the following relevant standard features:

1. **Released Product Approval Workflow:** D365 supports a workflow for releasing products to legal entities. However, this controls the release from shared product to legal entity, not the initial creation approval.

2. **Product Lifecycle State:** D365 has Product Lifecycle States that can block a product from being used in transactions. A new item can be created with state "Draft" (blocked from transactions) and changed to "Active" after approval.

3. **Engineering Change Management (ECM):** If Technica has the Engineering Change Management module, it provides full product creation workflows with approval gates, version control, and readiness checks. This is the most complete solution.

**Recommendation:** Use **Product Lifecycle State** as the primary control mechanism:
- Create lifecycle states: "Draft" (blocked from transactions), "Active" (allowed), "Discontinued" (blocked)
- New items are created in "Draft" state
- Use a **Power Automate flow** or **simple custom workflow** to route draft items for approval
- On approval, lifecycle state changes to "Active"
- This prevents items from being used in POs, production orders, or sales orders until approved

If Engineering Change Management is licensed, use it instead -- it provides a complete product governance framework with approval workflows, readiness checks, and engineering categories.

**Complexity reduction:** Using Product Lifecycle State reduces this from a full custom workflow (Medium complexity) to a configuration + low-code solution (Low complexity).

**Cross-FRD dependency (FRD03 - R&D):** If R&D engineers create items as part of the product development process, the item approval workflow should integrate with the R&D release process. Ensure the R&D FRD accounts for the handoff from engineering item creation to inventory item approval.

---

### 6. WMS Device (W001 through W003) -- All FIT

**What was proposed:**
- Receive POs via WMS mobile device
- Receive POs in back office for trade items
- User permissions per warehouse on device
- RECV location for initial receipt, BULK for put-away
- Put-away location entered by user on device

**Assessment: Standard WMS mobile receiving -- properly scoped**

| Aspect | Rating | Comment |
|--------|--------|---------|
| PO receiving on mobile device | Standard OOB | WMS mobile app |
| Back office receiving for trade items | Standard OOB | PO product receipt posting |
| Permission per warehouse | Standard OOB | Worker/warehouse assignment |
| RECV to BULK put-away | Standard OOB | Standard location directives |
| User-directed put-away | Standard OOB | Manual location entry on device |

**Recommendations:**

1. **The dual receiving approach** (WMS device for warehouse items, back office for trade items) is practical. Ensure clear process documentation on which items go through which path.

2. **Consider system-directed put-away** instead of user-directed for the Components warehouse (many locations). D365 location directives can suggest the optimal put-away location based on zone, volume, or item grouping. This reduces warehouse worker decision-making and improves location utilization.

3. **Cross-FRD dependency (FRD06 - Logistics):** When goods arrive via Landed Cost voyage (FRD06), the receiving process may differ from direct PO receiving. Ensure the WMS device flow supports both scenarios: (a) direct PO receipt and (b) voyage receipt where the in-transit transfer has already occurred.

---

### 7. Transfer Order Between Warehouses (TO0001 through TO0004) -- 1 GAP

**What was proposed:**
- Transfer order created in back office
- Stock keeper reserves quantity by priority
- WMS release: only available quantities create work
- License plate printing for transferred goods
- LP-based receiving at destination
- Picking: system-directed with worker override
- Pick/put actions in two stages (zone to delivery location, delivery location to bay door)
- Confirm transfer shipment by stock keeper
- Receive via LP receiving and put-away (user-directed)

**Assessment: Well-designed WMS transfer process with one workflow GAP**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Transfer order creation | Standard OOB | Back office |
| Quantity reservation | Standard OOB | Priority-based |
| Release to warehouse | Standard OOB | WMS wave/work generation |
| License plate generation | Standard OOB | Standard WMS |
| System-directed picking | Standard OOB | Location directives |
| Two-stage pick/put | Standard OOB | Work template configuration |
| LP receiving | Standard OOB | Standard WMS |
| Shipment confirmation | Standard OOB | Standard |

#### GAP Analysis: TO0002 -- Workflow in Transfer Order

| Attribute | Detail |
|-----------|--------|
| **GAP ID** | TO0002 |
| **Description** | Workflow approval on transfer orders |
| **FRD Classification** | GAP |
| **Assessed Complexity** | Medium |
| **Is it truly a GAP?** | **Yes -- D365 does not have a standard workflow for transfer orders** |

**Analysis:** D365 F&O does not provide an out-of-the-box workflow for transfer order approval. This is a genuine GAP if Technica requires approval before inter-warehouse transfers can be processed.

**Recommendation:** Two options:

1. **Custom Workflow (X++ extension):** Build a transfer order workflow using the D365 workflow framework. This requires developer effort to register the transfer order document in the workflow engine, create approval steps, and add the workflow toolbar to the transfer order form. **Complexity: Medium (estimated 3-5 days development).**

2. **Power Automate Approval Flow (lower cost alternative):** When a transfer order is created, trigger a Power Automate flow that sends an approval request. On approval, update a custom "Approved" flag on the transfer order. Use a validation check that prevents release to warehouse unless the flag is set. **Complexity: Low-Medium (estimated 1-2 days).**

**Recommendation:** If the approval requirement is simple (single approver, no conditional routing), use the **Power Automate approach**. If complex approval chains are needed (different approvers by warehouse, amount thresholds), invest in the **custom workflow**.

**Cross-FRD dependency (FRD05 - Production):** Production FRD05 (PRD009) relies on transfer orders to move materials from main warehouse to manufacturing warehouses. If transfer order approval is added, ensure it does not create bottlenecks in the production material flow. Consider auto-approving transfer orders generated by Master Planning / production release to avoid delays.

---

### 8. Item Movement Within Warehouse (No explicit requirement IDs)

**What was proposed:**
- Warehouse worker uses mobile device
- Scan source location, item, quantity
- Scan destination location
- System moves inventory between locations within same warehouse

**Assessment: Standard WMS movement -- no issues**

This is a standard WMS inventory movement operation using the mobile device. No GAPs, no concerns.

**Recommendation:** Ensure movement menu items are configured on the mobile device with appropriate security. Consider restricting certain location types (RECV, staging) from being movement targets to prevent inventory from being moved to inappropriate locations.

---

### 9. Warehouse Stock Count (No explicit requirement IDs)

**What was proposed:**
- Counting journal with on-hand quantity display (optional to hide)
- Manual entry of counted quantity
- System auto-calculates adjustment quantity
- Profit/loss posting for differences
- Mobile device counting by location (alternative)
- Spot checks biweekly, full count annually

**Assessment: Standard D365 counting process -- well-described**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Counting journal | Standard OOB | Standard inventory journal |
| Hide on-hand option | Standard OOB | Counting list report configuration |
| Auto-adjustment calculation | Standard OOB | Built-in |
| Profit/loss posting | Standard OOB | Posting profile configuration |
| Mobile device counting | Standard OOB | WMS cycle count |
| Spot checks | Standard OOB | Cycle counting plans |

**Recommendations:**

1. **For biweekly spot checks**, use **D365 Cycle Counting Plans** rather than ad-hoc counting journals. Cycle counting plans can be configured to:
   - Automatically select items/locations for counting based on criteria
   - Set counting frequency per item/location
   - Trigger counting work on the mobile device
   This is more efficient than manual journal creation and ensures consistent coverage.

2. **For yearly full counts**, use the standard **Counting Journal** with batch generation for all items. Consider using the **Tag Counting** feature for large warehouses -- it generates pre-printed tags for each location, which workers fill in and then enter into the system.

3. **Approval workflow on counting journals** is mentioned later (inventory adjustment section). Ensure the counting journal also goes through approval if the adjustment amount exceeds a threshold.

---

### 10. Inventory Adjustment (No explicit requirement IDs)

**What was proposed:**
- Inventory adjustment journal for gains/losses
- Cost added on inventory receipt
- GL posting via item group posting profile
- Inventory journal approval workflow enabled

**Assessment: Standard D365 inventory adjustment -- properly configured**

The decision to enable inventory journal approval workflow is a **good governance practice**. This prevents unauthorized adjustments.

**Recommendation:** Configure **journal approval limits** so that:
- Small adjustments (below a threshold) can be approved by the warehouse manager
- Large adjustments require Head of Stock or Finance approval
- Use reason codes on adjustment journals to categorize adjustments (scrap, damage, count variance, splitting, etc.)

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | IN02-001 | Replace PR-based reservation with **Project Item Requirements** -- may eliminate this GAP entirely | **HIGH** |
| 2 | IN03-009 | Use **lightweight production orders (cutting BOMs)** for item splitting instead of adjustment journals | **HIGH** |
| 3 | IN04-002 | Use **Product Lifecycle State** (Draft/Active) to block unapproved items instead of building a full custom workflow | **HIGH** |
| 4 | TO0002 | Use **Power Automate approval** for transfer order workflow (simpler) unless complex routing is needed | **MEDIUM** |
| 5 | IN03-001 | Use **Production Picking List Journal** as the Material Delivery Form equivalent; add custom fields if needed | **MEDIUM** |
| 6 | Product Setup | Define **two Storage Dimension Groups** explicitly (with/without Location tracking) -- document clearly | **MEDIUM** |
| 7 | Product Setup | Formally close the product dimension debate -- confirm only Configuration and Size are used | **LOW** |
| 8 | Warehouse | Clarify **which warehouses are WMS-enabled vs. basic** -- recommend WMS for Components only | **MEDIUM** |
| 9 | Warehouse | Consider **system-directed put-away** for Components warehouse instead of user-directed | **LOW** |
| 10 | Stock Count | Use **Cycle Counting Plans** for biweekly spot checks instead of ad-hoc journals | **LOW** |
| 11 | Quality | **Reconsider excluding Quality Management** -- at minimum, use Inventory Status dimension as lightweight QC gate | **HIGH** |
| 12 | Transfer Orders | **Auto-approve transfer orders from Master Planning** to avoid production bottlenecks (if TO workflow is implemented) | **MEDIUM** |
| 13 | Cross-FRD | Ensure transfer order flow maps to Production FRD05 warehouse structure (1000/1001/1002/2001/2002) | **HIGH** |
| 14 | Cross-FRD | Align receiving process with Logistics FRD06 Landed Cost voyage receiving | **MEDIUM** |

---

## Risk Flags

### 1. Quality Management Exclusion -- HIGH RISK
The FRD explicitly states "Quality management module will not be used to avoid long cycles." For a manufacturing company that tracks non-conformances (referenced in both this FRD and Production FRD05), excluding Quality Management creates a gap in traceability. The proposed alternative (quick pass/fail on device + non-conformance location routing) is functional but lacks:
- Non-conformance documentation and root cause tracking
- Vendor quality scoring (needed for Procurement FRD07 vendor evaluation)
- Quality order integration with production
- Corrective action tracking

**Mitigation:** If full QM is rejected, at minimum configure **Inventory Status** values (Available, QC Hold, Non-Conforming, Blocked) and use **Quality Item Sampling** for incoming inspection. This provides basic quality gating without the full QM cycle overhead.

### 2. Raw Material Tracking Complexity -- MEDIUM RISK
The linear meter tracking, square meter calculations, item splitting, and scrap piece tracking for raw materials represent significant process complexity. If not designed carefully, this area will generate the most user complaints and data accuracy issues. The cutting BOM approach (recommended above) provides structure, but requires:
- Proper BOM definitions for each cutting scenario
- User training on when to use cutting orders vs. adjustments
- Clear scrap reporting procedures

### 3. Unresolved Reviewer Comments -- MEDIUM RISK
Multiple comments remain open:
- EA4: Spare parts process scope ("Why Spare parts process is included in stock management FRD?")
- EA7: Ordering process mismatch with Procurement FRD07
- EA9/EA11: Process flow corrections (some marked as resolved, but verify)
- EA1: Product dimension clarity
- EA13: Item creation workflow justification

These indicate the FRD has not been through a final review cycle. Requirements may shift during implementation if these are not closed before development starts.

### 4. Cross-FRD Warehouse Structure Alignment -- MEDIUM RISK
This FRD defines inventory warehouses (Components, Raw Material) but does not reference the Production warehouses defined in FRD05 (1001 Mfg In, 1002 Mfg Out, 2001 Assembly In, 2002 Assembly Out) or the In-Transit warehouse from FRD06 (Logistics). The end-to-end material flow across all warehouses needs a single integrated warehouse map that spans all three FRDs.

### 5. Master Planning Requester Assignment -- LOW RISK
The FRD states "The requester must be specified when running the master planning since several users may request the same item." Master Planning in D365 does not natively assign a requester to planned orders. If this is critical, it requires either: (a) the PM running Master Planning per project (filtering by project demand), or (b) a custom extension to stamp the requester on planned orders based on the source demand (Item Requirement). Clarify this requirement before go-live.

---

## Efficiency Verdict

The Inventory Management FRD **uses D365 standard inventory features appropriately for a make-to-order environment**. The product setup (dimensions, variants, costing), WMS receiving and put-away, transfer orders, stock counting, and inventory adjustments are all standard OOB processes that are correctly described.

**Where efficiency can be improved:**

1. **Two of the four GAPs (IN02-001 and IN04-002) can be reduced or eliminated** by using standard D365 features differently -- Project Item Requirements instead of PR reservation, and Product Lifecycle State instead of custom item approval workflow. This saves development effort and reduces maintenance burden.

2. **The raw material splitting GAP (IN03-009) has a better solution** than adjustment journals -- lightweight cutting production orders provide cost traceability and audit trail without manual cost entry.

3. **The Material Delivery Form GAP (IN03-001) is likely not a true GAP** -- the Production Picking List Journal already captures the required information and can be extended with minimal effort.

4. **The Quality Management exclusion is the biggest strategic concern.** While the immediate impact is limited (Inventory Status can serve as a lightweight alternative), the long-term cost of not having structured quality tracking in a manufacturing environment will compound as the business grows and customer quality requirements increase.

**Bottom line:** The FRD is **70% standard D365, 20% configuration-solvable, and 10% genuine customization**. With the recommendations above, the customization percentage can be reduced to approximately 5%, primarily the Transfer Order workflow (TO0002) and the raw material cutting process refinements.

---

*This review is based on D365 F&O Inventory Management, Warehouse Management, and Product Information Management best practices as of 2026.*
