# FRD05 - Production - Solution Design Verification Report

**Document:** FRD05 - Production - V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Production Control, Manufacturing Execution, Master Planning

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | All / CAD Integration | CAD integration architecture is undefined — nearly every process depends on it, must be defined before development | [Risk Flags](#risk-flags) |
| 2 | **CRITICAL** | Setup - Full Reservation | "Require Full Reservation = TRUE" conflicts with partial assembly start — use Reservation = Explosion instead | [Setup](#setup) |
| 3 | **HIGH** | PRD004 - Routes | Verify if standard route version selection rules can handle product family routing before building custom logic | [PRD004 - Routes](#prd004---routes-1-gap) |
| 4 | **HIGH** | PRD011 - Shop Floor Execution | Verify estimated vs. actual time data already exists in D365 before treating PRD010-03 as a GAP | [PRD011 - Shop Floor Execution](#prd011---shop-floor-execution-2-gaps) |
| 5 | **HIGH** | PRD013 - Non-Conformities | Use D365 Quality Management module for non-conformance tracking instead of informal approach | [PRD013 - Non-Conformities](#prd013---non-conformities) |
| 6 | **HIGH** | Reviewer Comments | Many unresolved reviewer comments (EA, GB) — requirements still being clarified | [Risk Flags](#risk-flags) |
| 7 | **MEDIUM** | PRD001-002 - Production Lifecycle | Build reusable Power Automate notification template — don't build each notification GAP separately | [PRD001-002 - Production Lifecycle & 3D Drawing](#prd001-002---production-lifecycle--3d-drawing-5-gaps) |
| 8 | **MEDIUM** | PRD005-006 - Order Generation | Add Power Automate notification when manufacturing order completes to notify assembly team | [PRD005-006 - PO Tracking & Order Generation](#prd005-006---po-tracking--order-generation-all-fit) |
| 9 | **MEDIUM** | PRD009 - Resource Consumption | Warehouse In/Out separation may create unnecessary transfer steps — verify against physical layout | [PRD009 - Resource Consumption](#prd009---resource-consumption-all-fit) |
| 10 | **MEDIUM** | PRD013 - Non-Conformities | Rework flow is tightly coupled to CAD integration, adding risk | [PRD013 - Non-Conformities](#prd013---non-conformities) |
| 11 | **MEDIUM** | PRD014 - Assembly | Review if manual POC% is truly a GAP or if partial Report-as-Finished achieves the same | [PRD014 - Assembly](#prd014---assembly-2-gaps) |
| 12 | **MEDIUM** | PRD015 - SNAGs | Ensure SNAG categories roll up properly in project reports for quality KPIs | [PRD015 - SNAGs](#prd015---snags) |
| 13 | **MEDIUM** | All / Notifications | Five notification GAPs total — must be built as a cohesive framework, not piecemeal | [Risk Flags](#risk-flags) |
| 14 | **LOW** | PRD001-002 - Production Lifecycle | PRD002-03 (hold during rework) may not be a true GAP — verify if standard status change suffices | [PRD001-002 - Production Lifecycle & 3D Drawing](#prd001-002---production-lifecycle--3d-drawing-5-gaps) |
| 15 | **LOW** | PRD001-002 - Production Lifecycle | PRD002-05 (rework counter) is a simple X++ extension — low risk | [PRD001-002 - Production Lifecycle & 3D Drawing](#prd001-002---production-lifecycle--3d-drawing-5-gaps) |

**Totals:** 2 CRITICAL | 4 HIGH | 7 MEDIUM | 2 LOW

---

## Executive Summary

This FRD covers 15 production processes from 3D drawing through manufacturing, assembly, and SNAG management. The solution leverages D365 Production Control with Master Planning, Manufacturing Execution (Shop Floor), and integrates heavily with CAD systems. The document has **numerous unresolved reviewer comments** (EA, GB) indicating several areas still require clarification. While the core production flow is sound, the **CAD integration dependency and several notification GAPs** are the main concerns.

Overall assessment: **Solid production design with standard D365 usage, but multiple open issues and GAPs need resolution.**

---

## Process-by-Process Review

### PRD001-002 - Production Lifecycle & 3D Drawing (5 GAPs)

**What was proposed:**
- R&D or Project Engineers create 3D drawings
- Production manager checks feasibility; if not feasible → rework
- Drawing integrated from CAD to F&O
- BOM lines auto-populated from CAD integration
- Notifications for drawing release and rework

**Assessment: CAD integration dependency is the bottleneck**

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| PRD002-01 | Notify production manager on drawing release | **Power Automate** flow triggered on drawing URL field update. Low complexity. |
| PRD002-02 | Notify key positions on Re-Work | **Power Automate** with dynamic recipient list based on role. Low complexity. |
| PRD002-03 | Put production order on Hold during Re-Work | Standard OOB — production order status can be set to "Stopped". This may not be a true GAP — verify if standard status change suffices. |
| PRD002-04 | Import drawing URL from CAD | Part of the broader CAD integration. If CAD can push a URL via API, store it in a custom field on production order. Low complexity once integration exists. |
| PRD002-05 | Count integration runs (rework counter) | Custom integer field incremented on each CAD import. Simple X++ extension. |

**>> ATTENTION AREA: Notification GAPs should use a reusable pattern**

> GAPs PRD002-01 through 02 are notification patterns that recur across this entire FRD. Building each notification separately will create maintenance burden and inconsistent user experience.

**>> ENHANCEMENT:** Build a reusable Power Automate notification template and apply it across all similar GAPs in this FRD. Don't build each notification separately.

---

### PRD003 - Bill of Material (All FIT)

**What was proposed:**
- Nested BOMs (multi-level)
- Multiple BOM versions per item for different configurations
- BOM approval and activation workflow
- Cost calculation per BOM version/configuration

**Assessment: Standard D365 BOM management — excellent**

No issues. The nested BOM approach with version control and approval/activation gates is exactly how D365 is designed to work. The configuration variant tracking via product dimensions is the correct mechanism.

---

### PRD004 - Routes (1 GAP)

**What was proposed:**
- Two route types: Manufacturing Routes (auto from CAD attributes) and Assembly Routes (manual from test strategy meeting)
- Route determines operations sequence
- Operations assigned to resources

**Assessment: Good design, one GAP**

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| PRD004-02 | Identify route based on product family | **>> NEEDS REVIEW <<** This can potentially be addressed using **formula/route approval by product dimension group** or **route version selection rules** (by item group, configuration). Before building custom logic, verify if standard route version validity rules (by date, quantity, configuration) can achieve this. If product "family" maps to a product dimension, standard route selection may work. |

**>> ATTENTION AREA: Route selection may not need customization**

> Standard D365 route version selection rules support filtering by date, quantity, and configuration. If "product family" maps to a product dimension or item group, the standard mechanism may suffice without custom development.

**>> ENHANCEMENT:** Verify if standard route version rules can select route by product family before customizing. The manufacturing vs. assembly route separation is well-designed. Ensure route groups are configured differently for each — manufacturing routes can use automatic reporting, while assembly routes use manual operation completion.

---

### PRD005-006 - PO Tracking & Order Generation (All FIT)

**What was proposed:**
- Open PO line tracking for inbound material visibility
- Master Planning generates planned production orders, purchase orders, transfer orders
- Manufacturing vs. Assembly production order groups
- Production orders created by firming planned orders

**Assessment: Standard Master Planning flow — properly designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Master Planning engine | Standard OOB | Correct use for demand-driven planning |
| Planned order types | Standard OOB | Production, purchase, transfer |
| Firming process | Standard OOB | Production manager controls timing |
| Manufacturing vs. Assembly groups | Good | Pool ID grouping is appropriate |

**>> ATTENTION AREA: Cross-order notification gap between manufacturing and assembly**

> The reviewer comment about notifying assembly when manufacturing is complete (GB31) is important. D365 doesn't have built-in cross-order notifications, and without this notification the assembly team has no automatic signal to begin work.

**>> ENHANCEMENT:** Add a cross-order notification mechanism. Options:
- **Power Automate:** Trigger when manufacturing order status changes to "Reported as Finished" → notify assembly team
- **Production Floor Management workspace:** Assembly manager monitors component availability via the dashboard (already mentioned in PRD008)

---

### PRD007 - Schedule Production Orders (All FIT)

**What was proposed:**
- Automatic scheduling based on capacity
- Gantt chart for manual overrides
- Forward/backward scheduling

**Assessment: Standard capacity scheduling — no issues**

The Gantt chart visual scheduling is a good tool for production managers. Ensure the production calendar (Section 1.3.1.14) is configured correctly — capacity planning accuracy depends entirely on accurate calendar and resource setup.

---

### PRD008 - Release Production Order (All FIT)

**What was proposed:**
- Component availability check before release (checkmark system)
- Production Floor Management dashboard for visibility
- Covers on-hand, open POs, and planned orders

**Assessment: Excellent use of the Production Floor Management workspace**

The component availability check (available = checkmark, missing = exclamation) before release is a best practice. This prevents releasing orders that will stall on the floor.

---

### PRD009 - Resource Consumption (All FIT)

**What was proposed:**
- Transfer orders move materials from Main → Manufacturing warehouse
- Master Planning generates planned transfer orders
- Pick journal auto-posted on production order release
- Remaining materials transferred back after completion

**Assessment: Standard material flow — well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Transfer order flow | Standard OOB | Main → Manufacturing → Assembly |
| Auto pick journal | Standard OOB | On release |
| Return of excess material | Standard OOB | Manual transfer back |

**>> ATTENTION AREA: Warehouse structure may create unnecessary transfer overhead**

> The warehouse structure (1000 Main, 1001 Mfg In, 1002 Mfg Out, 2001 Assembly In, 2002 Assembly Out) is clear but creates many transfer steps. If materials flow linearly (Main → Mfg → Assembly), separate In/Out warehouses may add extra transfer transactions without operational value.

**>> ENHANCEMENT:** Verify the 5-warehouse structure against the physical warehouse layout. Consider whether separate In/Out warehouses add value or just create extra transfer transactions for a linear material flow.

---

### PRD010 - Splitting Production Order (FIT)

**Assessment:** The guidance to update quantity rather than split is correct. Splitting creates tracking complexity. The alternative (partial release) is the better practice and is properly documented.

---

### PRD011 - Shop Floor Execution (2 GAPs)

**What was proposed:**
- D365 Manufacturing Execution (Shop Floor App) for job tracking
- Workers start/stop jobs to track time
- Team leader monitors via Production Floor Management
- Estimated vs. actual time comparison needed

| GAP ID | Description | Complexity | Recommendation |
|--------|-------------|-----------|----------------|
| PRD010-02 | Notification when job exceeds estimated time | Medium | Two options: **(A)** Power Automate scheduled flow that checks running jobs every 15-30 min and compares elapsed time to estimated. **(B)** Custom periodic batch job in X++. Option A is faster to implement but has latency. Option B is more precise. |
| PRD010-03 | Track estimated vs actual operation time | **>> NEEDS REVIEW <<** | This data already exists in D365 — the route card journal captures actual time, and the route has estimated time. The gap is likely a **reporting gap**, not a data gap. Build a Power BI report comparing estimated vs. actual per operation/resource. |

**>> ATTENTION AREA: PRD010-03 may not be a true GAP**

> D365 Production Control already tracks planned vs. actual time on route operations via route card journals. The Production Floor Management workspace may already surface this data. This should be verified before treating it as a GAP requiring custom development.

**>> ENHANCEMENT:** Verify estimated vs. actual time data already exists in D365 before treating PRD010-03 as a GAP. If it is a reporting gap only, build a Power BI report rather than custom functionality.

---

### PRD012 - Report As Finished (FIT)

Standard process. Auto-status update when all jobs complete. Put-away location assignment is standard warehouse management.

---

### PRD013 - Non-Conformities

**What was proposed:**
- Bad quantity reported on production order
- Cost stays on bad quantity until scrapped
- Rework tracked via new WBS activity → CAD re-integration → new production order

**Assessment: Rework process needs more structure**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Bad quantity reporting | Standard OOB | Error quantity on production order |
| Cost on bad quantity | Standard OOB | Scrap accounting |
| Rework flow | **>> NEEDS IMPROVEMENT <<** | See below |

**>> ATTENTION AREA: Non-conformance tracking is informal**

> The current approach lacks the structured tracking, corrective action management, and audit trail that D365 Quality Management provides out of the box. For a manufacturing environment, this is a significant gap in traceability.

**>> ATTENTION AREA: Rework flow is tightly coupled to CAD integration**

> The rework flow (add WBS activity → push to CAD → re-integrate BOM → new production order) is conceptually correct but depends entirely on the undefined CAD integration, compounding risk.

**Recommendations:**

1. **>> ENHANCEMENT:** Use D365 Quality Management for non-conformance tracking instead of the informal approach described. D365 has a dedicated Non-Conformance module with:
   - Non-conformance types (internal, customer, vendor)
   - Corrective actions tracking
   - Quality order integration
   - Root cause analysis

   This is more structured than the approach described and provides better audit trail.

2. **>> ENHANCEMENT:** Decouple the rework flow from CAD integration where possible. The rework flow (add WBS activity → push to CAD → re-integrate BOM → new production order) is conceptually correct but tightly coupled to CAD integration, adding risk.

---

### PRD014 - Assembly (2 GAPs)

**What was proposed:**
- Test Strategy Meeting determines assembly route
- Assembly manager manually selects/updates route
- Assembly can start before all manufacturing parts are complete
- Rework during assembly follows same pattern as manufacturing

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| PRD013-01 | Manual POC% on assembly | **>> NEEDS REVIEW <<** Not a true GAP — D365 allows manual quantity reporting on production orders. Use partial "Report as Finished" to track completion. Or use job completion % on the shop floor. |
| PRD013-02 | Generate test report for standard assemblies | This requires custom SSRS/Power BI report pulling data from production order completion, quality tests, and route operations. Medium complexity. |

**>> ATTENTION AREA: Manual POC% may not require custom development**

> D365 already supports partial Report-as-Finished and job completion percentage tracking on the shop floor. PRD013-01 should be verified against standard functionality before treating as a GAP.

**>> ENHANCEMENT:** Review if manual POC% is truly a GAP or if partial Report-as-Finished achieves the same result. Also consider using **Production order sequencing** (available in D365) for better visual management of assembly order priorities. The manual assembly scheduling is appropriate for make-to-order manufacturing.

---

### PRD015 - SNAGs

**What was proposed:**
- Before execution: Rework tracked as extra effort on same WBS activity
- After execution: New WBS activity "SNAG" + separate category
- Separate SNAG categories for manufacturing vs. design
- Case management for workflow
- Linked to Engineering Change Request (FRD03)

**Assessment: Creative SNAG tracking via WBS + categories**

The approach of using WBS activities with SNAG categories and separate production order pools ("SNAG" group) is a **practical workaround** that enables cost tracking within the project accounting framework. This is well-integrated with FRD04 (Project Accounting).

**>> ATTENTION AREA: SNAG cost visibility in project reports**

> If SNAG categories do not roll up properly in project reports, management will lack visibility into total SNAG cost as a percentage of project cost — a key KPI for quality improvement.

**>> ENHANCEMENT:** Ensure SNAG categories roll up properly in project reports so management can see total SNAG cost as a % of project cost. This is a key KPI for quality improvement.

---

### Setup

| Setup Item | Rating | Comment |
|-----------|--------|---------|
| Full Reservation parameter = TRUE | **>> RISKY <<** | This means NO production order can release until ALL components are available. In a make-to-order environment with long lead times, this can cause delays. Consider using **Partial reservation** with manual override for urgent orders. |
| Calendar (8:00-17:00, breaks) | Standard | Straightforward |
| Resources (machines, humans, vendors) | Standard | Calendar per resource is good for capacity accuracy |
| Warehouse structure (5 warehouses) | Acceptable | Verify against physical layout |
| Operations | Standard | To be collected during migration |

**>> ATTENTION AREA: Full Reservation parameter directly contradicts assembly requirements**

> The **"Require Full Reservation" = TRUE** is a restrictive parameter. While it prevents releasing orders with missing components (safety measure), it can also **block production when a few minor components are missing but main materials are available**. In practice, Technica's factory reviewers (GB) noted that "assembly can start before completing all manufacturing parts." This directly conflicts with the full reservation requirement.

**>> ENHANCEMENT (CRITICAL):** Use **Reservation = "Explosion"** level instead, which reserves available material and flags shortages without blocking. Combine with the component availability dashboard (PRD008) for manual decision-making. This resolves the conflict between the full reservation parameter and the documented assembly start-before-complete requirement.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | PRD002 | Build reusable Power Automate notification template for all production alerts | Medium |
| 2 | PRD004 | Verify if standard route version rules can select route by product family before customizing | High |
| 3 | PRD010 | Verify estimated vs. actual time data already exists before treating as GAP | High |
| 4 | PRD013 | Use D365 Quality Management module for non-conformance tracking | Medium |
| 5 | Setup | Reconsider "Require Full Reservation = TRUE" — conflicts with partial assembly start | CRITICAL |
| 6 | PRD014 | Review if manual POC% is truly a GAP or if partial RAF achieves the same | Medium |
| 7 | All | Resolve CAD integration architecture — many processes depend on it | CRITICAL |
| 8 | PRD006 | Add Power Automate notification when manufacturing order completes → notify assembly | Medium |

---

## Risk Flags

1. **CAD integration is referenced in nearly every process** but architecture is still undefined (deferred to Integration FRD). This is the single biggest production risk.

2. **"Require Full Reservation = TRUE" contradicts "assembly can start before all parts ready."** This conflict must be resolved before go-live or the factory floor will be blocked.

3. **Many unresolved reviewer comments** (EA, GB) indicate requirements are still being clarified. Several comments are marked "Not clear" without resolution.

4. **SNAG/Rework process is functional but informal.** Consider using Quality Management module for better structure and auditability.

5. **Notification GAPs (5 total)** are all solvable with Power Automate but need to be built as a cohesive notification framework, not piecemeal.

---

## Efficiency Verdict

The production solution **uses D365 Production Control appropriately** for a make-to-order manufacturing environment. The master planning → scheduling → shop floor execution → report as finished pipeline is standard and well-designed. The SNAG tracking via WBS categories is a creative cross-module solution.

Key improvements: resolve the **Full Reservation parameter conflict**, implement **Quality Management** for non-conformances, and establish the **CAD integration architecture** that so many processes depend on.

---

*This review is based on D365 F&O Production Control and Manufacturing Execution best practices as of 2026.*
