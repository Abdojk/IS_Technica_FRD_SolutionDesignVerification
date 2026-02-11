# FRD10 - Asset Management & Maintenance - Solution Design Verification Report

**Document:** FRD10 - Asset Management & Maintenance - V2.0
**Status:** OUT OF SCOPE (Current Implementation Phase)
**Prepared by:** Nicolas Majdalani | Contributors: Antonio Saleh, Abdo Khoury
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Asset Management (Supply Chain Management Module)

---

## Executive Summary

This FRD covers the D365 F&O Asset Management module for Technica International, encompassing 6 processes: Functional Location Creation, Asset Creation, Maintenance Requests/Work Orders, Work Order Processing, Preventive Maintenance Plans, and Reports & Inquiries. **This FRD has been marked as "Out of Scope" for the current implementation phase** and will not be deployed during the initial go-live.

Despite the out-of-scope designation, the document is well-developed and reflects a mature understanding of D365 Asset Management capabilities. **All 33 requirements listed across the 6 processes are marked as FIT** -- there are zero GAPs identified. The solution relies entirely on out-of-the-box D365 Asset Management functionality with very few customizations anticipated.

The document does contain **several unresolved reviewer comments** (GB, NM) indicating open questions around functional location coding, condition assessment usage, service level definitions, counter-based maintenance plans, and the Fixed Asset/Asset Management number sequence relationship. These would need resolution before any future implementation phase.

Overall assessment: **Clean, standard-functionality design with zero GAPs. When brought into scope, this module should be one of the lower-risk implementations -- provided prerequisite modules (Fixed Assets, Project Accounting, Inventory) are already stable.**

---

## Process Overview

### AM-001: Functional Location Creation (All FIT)

**What was proposed:**
- Hierarchical functional location structure (areas, rooms, warehouses, shelves)
- Functional location types controlling single vs. multi-asset installation
- Lifecycle states (New, Active, Burnt, Ended) with configurable transition rules
- Lifecycle models controlling allowed state transitions
- Optional: maintenance plans attached to functional location types (auto-applied to all installed assets)

**Assessment: Standard D365 Asset Management -- no issues**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Functional location hierarchy | Standard OOB | Flexible structure |
| Location types | Standard OOB | Controls asset installation rules |
| Lifecycle states/models | Standard OOB | Configurable transitions |

**Open Issue:** Reviewer comment GB1 asks how the location coding is structured -- whether a specific coding scheme for areas needs to be implemented. NM responded that the location is set by asset and location nature is flexible. **This needs a definitive answer before implementation: Technica should define a location naming convention/hierarchy standard (e.g., Site > Building > Floor > Room) during migration planning.**

---

### AM-002: Asset Creation -- Manual/Automatic (All FIT)

**What was proposed:**
- Manual asset creation with configurable sequence numbering
- Automatic asset creation from Fixed Asset module ("Create maintenance asset" action)
- Technica decision: automatic sequence numbers (different from Fixed Asset numbers)
- Asset lifecycle states (New, Active, In Repair, Damaged, Sold, Scrapped)
- Asset lifecycle models with configurable state transitions
- Asset service levels (priority-based scheduling with SLA dates)
- Asset criticalities (production impact scoring)
- Asset counters (production hours, quantities) for counter-based maintenance
- Condition assessment templates for post-maintenance evaluation
- Asset type defaults (suggested spare parts, alternative items, maintenance plans)
- Asset transfer between functional locations (Move, Remove, Install)

**Assessment: Comprehensive asset master setup -- well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Manual/Auto creation | Standard OOB | Fixed Asset integration |
| Sequence numbering | Standard OOB | Technica chose separate numbering |
| Lifecycle management | Standard OOB | States + models |
| Service levels | Standard OOB | SLA-driven scheduling |
| Criticalities | Standard OOB | Production impact scoring |
| Counters | Standard OOB | For counter-based maintenance |
| Asset type defaults | Standard OOB | Spare parts + plans |

**Open Issues:**

1. **Reviewer comment GB3:** "All locations are active at Technica. We don't create an asset unless the location is already available." This simplifies the lifecycle model -- consider removing the "New" state for functional locations if Technica always has locations ready before asset creation.

2. **Reviewer comment GB5:** Condition assessment template purpose was unclear. NM clarified it is an optional post-maintenance checklist for asset condition validation. **Technica needs to decide during migration planning whether condition assessments add value to their maintenance workflow or introduce unnecessary overhead.**

3. **Reviewer comment GB7:** Asset service levels were not clear. NM explained these control work order priority and expected start/end dates. **Technica must define their service level matrix (what priority level for what asset type/criticality combination) before configuration.**

---

### AM-003: Manual Maintenance Request / Work Order Creation (All FIT)

**What was proposed:**
- Maintenance request creation by maintenance workers or production workers
- Request captures: asset, functional location, description, service level, downtime info
- Duplicate request detection (system warns if asset already has an open request)
- Request lifecycle: New > In Progress > Finished/Rejected
- Responsible maintenance worker auto-assignment (right-to-left matching: Trade > Job Variant > Job Type > Category > Location > Asset > Asset Type)
- Work order creation from maintenance request (manual or batch)
- System prevents work order creation if asset is already in repair or restricted state

**Assessment: Standard reactive maintenance workflow -- well-documented**

The process flow is clear: Request > Enrich Data > Create Work Order. The worker assignment logic (right-to-left most-specific-match) is standard D365 behavior and is well-explained.

---

### AM-004: Work Order Processing Details (All FIT)

**What was proposed:**
- Work order creation (manual or auto from maintenance request)
- Multiple maintenance job lines per work order (inspection, ad-hoc, calibration, etc.)
- Forecasted figures (hours, items, expenses) auto-populated from maintenance job type defaults
- Worker scheduling via Dispatch (manual assignment) or Schedule (availability-based)
- Maintenance checklists auto-populated from job type defaults
- Condition assessment on work order completion
- Cost capture via Project journals (Hours, Items, Expenses)
- Journal validation and posting (creates voucher transactions)
- Project integration: each work order line gets a project activity number
- Forecast vs. Actual cost comparison reports
- Asset status auto-updated: Active > In Repair (on WO start) > Active (on WO finish)
- Document attachment capability on work order lines

**Assessment: This is the most detailed and well-structured process in the FRD**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Multi-line work orders | Standard OOB | Different job types per line |
| Forecast auto-population | Standard OOB | From job type defaults |
| Dispatch/Schedule | Standard OOB | Manual vs. capacity-based |
| Checklists | Standard OOB | Template-driven |
| Project journal costing | Standard OOB | Hours/Items/Expense |
| Asset status automation | Standard OOB | Lifecycle-driven |
| Forecast vs. Actual | Standard OOB | Built-in inquiry |

**Key Design Decision:** Info-Sys and Technica agreed to use **a single project ID for all maintenance work orders** to centralize cost tracking. This is a practical choice for a first implementation phase. However, if Technica later needs cost analysis by department, location, or asset group, they may want to revisit this and use multiple projects or rely on activity-level breakdowns.

---

### AM-005: Preventive Maintenance Plans -- Automation (All FIT)

**What was proposed:**
- Time-based maintenance plans (periodic intervals by maintenance job type)
- Counter-based maintenance plans (triggered by counter thresholds -- e.g., running hours)
- Technica will use both methods: time-based for general equipment, counter-based for diesel generators and air compressors
- Maintenance schedule generation via batch job (periodic)
- Manual "Auto create" button for on-demand plan execution
- Schedule adjustment before work order creation (shift dates as needed)
- Work order auto-creation from maintenance schedule lines (optional toggle)
- Service items: single asset created for service items with maintenance tracked via work orders

**Assessment: Strong preventive maintenance design using standard capabilities**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Time-based plans | Standard OOB | Periodic scheduling |
| Counter-based plans | Standard OOB | Threshold-triggered |
| Batch scheduling | Standard OOB | Automated execution |
| Schedule adjustment | Standard OOB | Pre-creation flexibility |
| Auto work order creation | Standard OOB | Configurable toggle |

**Open Issue:** Reviewer comment GB9/GB11 raised the need to include running hours criteria for counter-based plans (specifically for diesel generators and air compressors). NM confirmed this was added. **When brought into scope, Technica must ensure counter registration processes are defined -- who updates the counters, how often, and whether IoT/automated counter feeds are planned.**

---

### AM-006: Reports & Inquiries (Standard OOB)

**What was proposed:**
17 out-of-the-box D365 reports and inquiries:

| Report | Category |
|--------|----------|
| Asset Consumption Report | Asset analysis |
| Maintenance Status Inquiry | Status monitoring |
| Capacity Load | Resource planning |
| Item Where Used | Spare parts tracking |
| Asset Cost Control | Cost analysis |
| Hour Cost Control | Labor analysis |
| Functional Location Cost Control | Location-based costing |
| Functional Location Hour Control | Location-based labor |
| Maintenance Schedule Cost | Preventive cost planning |
| Work Order Cost Control | WO cost analysis |
| Work Order Date Control | SLA compliance |
| Work Order Hour Control | WO labor analysis |
| Maintenance Request Details | Request tracking |
| Maintenance Request List | Request overview |
| Work Order Consumption Report | Materials/labor consumption |
| Work Order Report (PDF) | Formal work order document |
| Capacity Load (scheduling) | Scheduling analysis |

Technica also provided 6 Excel reports from their current system. Info-Sys confirmed all requested data exists in D365 standard inquiries, primarily under "Asset Cost Control."

**Assessment: No custom reports needed. All reporting requirements met by OOB inquiries.**

If deeper analytics are needed in a future phase, consider Power BI dashboards for:
- Asset downtime trending
- Maintenance cost per asset over time
- Preventive vs. corrective maintenance ratio
- MTBF (Mean Time Between Failures) and MTTR (Mean Time To Repair) metrics

---

## Key Considerations If Brought Into Scope

### 1. Data Migration is the Primary Workstream

Since all 33 requirements are FIT, the implementation effort will be dominated by **data migration and master data setup**, not development. The following data must be prepared:

| Data Set | Source | Complexity |
|----------|--------|-----------|
| Functional location hierarchy | Technica facility layout | Medium -- needs naming convention |
| Asset master records | Current asset register / Fixed Asset module | High -- volume-dependent |
| Asset type definitions | Technica engineering team | Medium -- needs classification |
| Maintenance job types and categories | Technica maintenance team | Medium -- needs standardization |
| Maintenance checklists / templates | Current paper or Excel checklists | High -- requires content digitization |
| Preventive maintenance plans | Current PM schedules | Medium -- needs interval/counter data |
| Counter initial values | Equipment logs / meters | Medium -- needs baseline readings |
| Spare parts mapping to asset types | Technica maintenance/procurement | High -- cross-module |
| Worker/technician assignments | HR records | Low |

**Recommendation:** Start data collection worksheets for asset master data and maintenance plans **before the module formally enters scope.** This data preparation is typically the longest lead-time item in Asset Management implementations.

### 2. Prerequisite Module Dependencies Must Be Stable

Asset Management cannot function in isolation. The following modules must be live and stable:

| Dependency | FRD | Why Required |
|------------|-----|-------------|
| Fixed Assets | FRD12 (Fixed Assets) | Auto-creation of maintenance assets from fixed assets; number sequence alignment |
| Project Accounting | FRD04 | Work order cost tracking via project journals; single project ID for all maintenance |
| Inventory/Spare Parts | FRD (Spare Parts) / FRD09 | Spare part availability for work orders; item forecasts on maintenance jobs |
| Procurement | FRD07 | Purchase of spare parts and maintenance services; planned purchase orders from maintenance |
| HR/Workers | -- | Maintenance worker assignment; resource calendars |
| Production | FRD05 | Asset counters based on production hours/quantities; production equipment maintenance |

**Recommendation:** Do not bring Asset Management into scope until Fixed Assets (FRD12), Project Accounting (FRD04), and Inventory (FRD09) are in production and stabilized. Attempting to go live with Asset Management concurrently with its dependencies creates excessive integration risk.

### 3. Fixed Asset Integration Requires Design Decisions

The FRD describes automatic asset creation from the Fixed Asset module. Several decisions remain open:

- **Number sequence strategy:** Technica chose separate numbering (Asset Management number differs from Fixed Asset number). This is confirmed but the actual sequence format must be defined.
- **Which fixed assets become maintenance assets?** Not all fixed assets need maintenance tracking. Technica must define the scope: production machines, vehicles, infrastructure equipment, etc.
- **Timing of migration:** Should existing fixed assets be bulk-migrated to Asset Management, or only new assets going forward? A phased approach (critical production assets first, then expand) is recommended.

### 4. Counter Registration Process Needs Operational Design

Counter-based preventive maintenance (for diesel generators and air compressors) depends on regular counter updates. Without reliable counter data, maintenance plans will not trigger correctly.

**Options to evaluate:**
- **Manual counter registration:** Technicians log readings periodically (weekly/monthly). Simple but error-prone.
- **IoT integration:** Automated counter feeds from equipment sensors. Higher accuracy but requires IoT infrastructure investment.
- **Production system integration:** If equipment counters correlate with production output, D365 production data could feed asset counters automatically via batch jobs.

**Recommendation:** For initial implementation, use manual counter registration with a defined schedule. Plan IoT integration as a Phase 2 enhancement if Technica has the infrastructure appetite.

### 5. Single Project ID Decision -- Long-Term Implications

The agreement to use a single project ID for all maintenance work orders simplifies initial setup but limits analytical flexibility. Consider these implications:

| Scenario | Single Project | Multiple Projects |
|----------|---------------|-------------------|
| Total maintenance cost visibility | Yes | Yes |
| Cost by asset type | Via activity breakdown | Via project breakdown |
| Cost by department | Requires filtering | Direct project-level |
| Budget by maintenance category | Activity-level only | Project-level budgeting |
| Project-level reporting | One massive project | Meaningful segments |

**Recommendation:** When this module enters scope, revisit whether a single project is still appropriate or whether splitting by asset type (as the setup screen already supports) provides better cost governance.

### 6. Unresolved Reviewer Comments

The following reviewer comments in the FRD remain open and must be resolved before implementation:

| Comment | Topic | Action Needed |
|---------|-------|---------------|
| GB1/NM2 | Functional location coding scheme | Define naming convention and hierarchy standard |
| GB3/NM4 | Location lifecycle states | Confirm if "New" state is needed given all locations are active |
| GB5/NM6 | Condition assessment purpose | Decide whether to use condition assessments or skip them |
| GB7/NM8 | Asset service level definitions | Define service level matrix with SLA times per priority |
| GB9/NM10 | Counter-based PM for air compressors | Confirmed added; validate counter types and thresholds |
| GB11/NM12 | Running hours criteria | Confirmed added; define specific hour intervals |
| GB13/NM14 | Fixed Asset to Asset Mgmt number link | Confirmed separate numbering; define sequence format |
| Document Approvals | Unsigned | FRD has not been formally signed off (comments section has open questions from Technica) |

---

## Dependencies on Other FRDs

| FRD | Dependency Type | Impact on Asset Management |
|-----|----------------|---------------------------|
| **FRD12 - Fixed Assets** | **Critical** | Asset creation flow starts from Fixed Assets. Number sequence alignment. Asset acquisition cost feeds into maintenance cost analysis. Must be live before AM. |
| **FRD04 - Project Accounting** | **Critical** | All work order costs are posted through Project journals. The single project ID decision lives here. Project must be created and configured before first work order. |
| **FRD09 - Inventory Management** | **High** | Spare parts availability on work orders. Item forecasts on maintenance job defaults. Spare parts procurement triggers from maintenance demand. |
| **FRD07 - Procurement** | **High** | Purchase of spare parts not in stock. Vendor management for maintenance service providers. Planned purchase orders from maintenance plans. |
| **FRD05 - Production** | **Medium** | Production equipment is the primary asset category. Production counters (hours, quantities) feed asset counter-based maintenance. Asset downtime impacts production scheduling. |
| **FRD06 - Logistics** | **Low** | Spare parts warehouse transfers. Potential transfer orders for maintenance materials to maintenance locations. |
| **FRD01 - Sales** | **None** | No direct dependency. |
| **FRD03 - R&D** | **Low** | Engineering changes may affect asset configurations. Asset modifications from R&D design changes. |
| **FRD13 - Integration** | **Medium** | If IoT counter integration is pursued, integration architecture applies. Any external maintenance system data migration. |

---

## Brief Efficiency Assessment

### Strengths

1. **100% FIT rate.** All 33 requirements are met by standard D365 Asset Management functionality. This is the cleanest FRD in the Technica implementation suite -- zero customizations needed.

2. **Good use of out-of-the-box capabilities.** The solution correctly leverages lifecycle models, service levels, criticalities, counter-based maintenance, forecasting, and project integration without overcomplicating the design.

3. **Preventive maintenance strategy is practical.** The combination of time-based plans (general equipment) and counter-based plans (generators, compressors) is appropriate for a manufacturing environment. The flexibility to run plans manually or via batch is a sensible operational choice.

4. **Project integration for cost tracking** provides financial visibility into maintenance spending without requiring a separate cost accounting system.

5. **Comprehensive reporting** using only OOB inquiries means no custom report development effort.

### Areas for Improvement When Brought Into Scope

1. **Resolve all open reviewer comments** before starting configuration. The GB/NM comment threads indicate business decisions that are still pending.

2. **Define the asset scope** -- not every fixed asset needs maintenance tracking. A clear scoping exercise (which asset types enter Asset Management) prevents over-migration.

3. **Establish counter registration SOPs** before go-live. Counter-based maintenance is only as good as the counter data feeding it.

4. **Consider the succeeding maintenance jobs feature** (mentioned in the FRD as "for future use" -- where completing an Ad Hoc job auto-generates an Inspection work order). This is a standard D365 feature that could improve maintenance quality with minimal effort.

5. **Plan for mobile/tablet access.** The FRD does not mention mobile capabilities, but maintenance workers typically need field access to work orders, checklists, and condition assessments. D365 Asset Management has a mobile workspace that should be evaluated.

6. **The single project ID for all maintenance** will eventually become unwieldy as work order volume grows. Plan for segmentation by asset type or department in a future phase.

### Implementation Effort Estimate (When In Scope)

| Phase | Effort | Notes |
|-------|--------|-------|
| Data migration preparation | High | Asset masters, maintenance plans, checklists, counters |
| Module configuration | Low-Medium | All FIT; standard setup only |
| Integration testing | Medium | Fixed Assets, Project Accounting, Inventory |
| User training | Medium | New module for maintenance team |
| Custom development | None | Zero GAPs |
| UAT | Low-Medium | Standard workflows to validate |

**Total estimated implementation risk: LOW** -- contingent on prerequisite modules being stable.

---

## Final Verdict

FRD10 is a **well-written, standard-functionality Asset Management design** that requires no customization. Its out-of-scope status is likely a smart phasing decision -- Asset Management has multiple dependencies on core modules (Fixed Assets, Project Accounting, Inventory, Procurement) that should be stabilized first.

**When this module re-enters scope, the primary effort will be data migration and operational readiness (counter SOPs, checklist digitization, asset scoping) rather than system development.** The recommendation is to begin data collection and business decision-making (the open reviewer comments) well in advance of the formal implementation start date.

---

*This review is based on D365 F&O Asset Management (Supply Chain Management) best practices as of 2026. The out-of-scope designation reflects Technica's current phasing decision, not a deficiency in the solution design.*
