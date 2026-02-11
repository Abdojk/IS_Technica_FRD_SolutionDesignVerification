# FRD13 - Integration - Solution Design Verification Report

**Document:** Technica - FRD13-Integration-V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O with CAD System Integration (Staging-Based Architecture)

---

## Executive Summary

FRD13 is the **most critical cross-cutting document** in Technica's FRD set. Every other FRD (Sales, R&D, Production, Project Accounting, Logistics) references integrations that should be defined here. However, this document is **severely underdeveloped** relative to its importance. At only 8 pages (including cover and sign-off), it covers only **one integration domain: CAD-to-F&O via a staging database**. It entirely omits the CRM-to-F&O integrations (Opportunities, DFI, Accounts, Contacts, Timesheets, Trips/Missions) that FRD01, FRD02, and FRD04 depend on.

The document defines three integration processes (INT001 overview, INT002 F&O-to-Staging, INT003 CAD-to-Staging) using a **staging table middleware pattern**. While the staging approach is viable for CAD integration, the document lacks critical technical details: no data mapping specifications, no error handling strategy, no conflict resolution rules, no API or protocol definitions, and no integration monitoring approach.

Overall assessment: **Incomplete and insufficient as an integration architecture document. Requires significant expansion before development can proceed. The CAD staging pattern is directionally correct but under-specified. CRM integrations are entirely missing.**

---

## Integration-by-Integration Review

### INT001 - Integration Overview (Architecture Summary)

**What was proposed:**
- A staging database sits between F&O and the CAD system
- F&O PUSHes data to staging; CAD PULLs from staging (and vice versa)
- Data flows are periodic (interval TBD) or on-demand
- Entities exchanged: Projects/WBS, Raw Materials, BOM Lines, Routes, Serial Numbers, Production Order Status

**Assessment: Staging pattern is acceptable but under-specified**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Staging database as middleware | Acceptable | Common pattern for non-Microsoft CAD systems; avoids direct coupling |
| Data flow direction clarity | Good | PUSH/PULL semantics are clearly stated per entity |
| Frequency definition ("periodically") | Insufficient | "30 minutes or 2 hours" is not a requirement; needs SLA-driven definition |
| Error handling | Missing | No mention of retry logic, dead-letter queues, or failure notifications |
| Data mapping / field specifications | Missing | No field-level mapping for any entity |
| Conflict resolution | Missing | No rules for concurrent edits (e.g., item modified in both CAD and F&O) |
| Monitoring & alerting | Missing | No integration dashboard, health checks, or failure alerts |
| Security & authentication | Missing | No mention of how staging database is secured or accessed |
| Technology platform | Missing | No specification of whether staging is SQL Server, Azure SQL, API layer, or file-based |

**Recommendations:**

1. **Define the staging platform explicitly.** The staging database could be:
   - **Option A (Recommended): Azure SQL Database** with stored procedures for data validation. This provides cloud-native monitoring, geo-redundancy, and easy connectivity from both F&O (via Data Management Framework or custom services) and CAD.
   - **Option B: Azure Data Lake + Azure Data Factory (ADF)** for more complex transformation pipelines. Better if data volumes are large or transformations are complex.
   - **Option C: D365 F&O Data Entities exposed via OData** with the CAD system calling F&O APIs directly (no staging). Simpler but creates tighter coupling.

2. **Define polling intervals based on business requirements, not convenience.** For each data flow, answer: "What is the maximum acceptable delay between source change and target availability?" For example:
   - Production Order Status: If assembly managers need real-time visibility, 30-minute polling may be too slow. Consider event-driven push via Azure Service Bus.
   - Raw Material master data: Daily sync may suffice since new raw materials are infrequent.
   - BOM Lines (on-demand): This is already correctly specified as pull-on-demand.

3. **Add an integration monitoring layer.** Every integration must have:
   - Success/failure logging per record
   - Dashboard showing last sync time, records processed, errors pending
   - Alert notifications when sync fails or exceeds SLA
   - **Recommended tool: Azure Monitor + Logic Apps alerting**, or a simple Power BI dashboard reading from staging audit tables.

---

### INT002 - F&O to Staging (3 Data Flows)

**What was proposed:**

| # | Data Flow | Trigger | Direction |
|---|-----------|---------|-----------|
| 1 | Project ID, Project Name, WBS Activities | Periodic | F&O --> Staging |
| 2 | Raw Material Products (Replenishment items) | Periodic | F&O --> Staging |
| 3 | Production Order Status Update | Periodic | F&O --> Staging |

**Assessment: Correct direction, but implementation details are absent**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Data flow #1: Project/WBS push | Needs Detail | Which WBS fields? Only published WBS? What about WBS updates/deletions? |
| Data flow #2: Raw material push | Needs Detail | Which product fields? What triggers "new" vs "updated"? How are deletions handled? |
| Data flow #3: Production order status | Needs Detail | Which statuses? All transitions or only specific ones (e.g., Released, Completed)? |
| "Flagged as PUSHED" mechanism | Good Concept | Delta tracking via flag is a valid pattern, but needs an "Acknowledged" flag from the consumer side |
| Handling of updates to pushed records | Partially Addressed | Doc mentions "if a pushed item is edited the updated fields will be pushed" -- but field-level delta tracking is complex |

**Recommended D365 Integration Pattern:**

| Data Flow | Recommended Pattern | Complexity | Rationale |
|-----------|-------------------|-----------|-----------|
| Project/WBS to Staging | **D365 Data Management Framework (DMF) + Recurring Integration via Azure Service Bus** | Medium | DMF already has Project and WBS data entities. Use recurring data export jobs to push to Azure SQL staging. The "Published only" filter can be applied via a custom data entity filter. |
| Raw Materials to Staging | **DMF Recurring Export with Change Tracking** | Low-Medium | Enable D365 change tracking on Released Products entity. DMF incremental export captures only new/modified records. Eliminates the need for a custom "PUSHED" flag. |
| Production Order Status | **Business Events (D365 F&O) + Azure Service Bus** | Medium | D365 Business Events can fire when production order status changes. This is event-driven (near real-time) rather than periodic, eliminating polling latency. Route events to staging via Azure Function or Logic App. |

**Detailed Recommendations:**

1. **Replace the custom "PUSHED" flag with D365 Change Tracking.** D365 F&O has built-in change tracking on data entities. When enabled, the system automatically tracks which records have changed since the last export. This eliminates the need for custom flag fields and handles updates/inserts/deletes natively.

2. **For Production Order Status, use D365 Business Events instead of polling.** Business Events are purpose-built for this scenario:
   - Configure a business event for production order status change
   - Route the event to an Azure Service Bus queue
   - An Azure Function processes the event and writes to staging
   - This gives near real-time status updates (seconds, not minutes/hours)
   - No polling load on the F&O database

3. **Define what "WBS Activities" means at the field level.** The minimum mapping should include:

   | F&O Field | Staging Field | Notes |
   |-----------|--------------|-------|
   | Project ID | ProjectId | Primary key |
   | Project Name | ProjectName | |
   | WBS Activity Number | ActivityNumber | Hierarchical ID |
   | Activity Description | ActivityDescription | |
   | Activity Type | ActivityType | Phase, Task, Milestone |
   | Planned Start/End | PlannedStartDate, PlannedEndDate | |
   | Status | ActivityStatus | Only "Published" records |

   This level of detail is absent from the FRD and must be added.

---

### INT003 - CAD to Staging (4 Data Flows)

**What was proposed:**

| # | Data Flow | Trigger | Direction |
|---|-----------|---------|-----------|
| 1 | Manufactured Items (Item Number, Item Name) + Attributes | Periodic | CAD --> Staging --> F&O |
| 2 | BOM Lines for Quotation | On Demand | CAD --> Staging, F&O PULLs |
| 3 | BOM Lines for Manufacturing | On Demand | CAD --> Staging, F&O PULLs |
| 4 | Serial Numbers for manufactured items | Periodic | CAD --> Staging --> F&O |

**Assessment: Most detailed section, but significant complexity is underestimated**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Level 1 item validation | Good | Requiring Level 1 products to pre-exist in F&O prevents orphan records |
| Attribute-to-Route mapping | Complex - Needs Architecture | Custom screen with multi-attribute route selection is a significant customization |
| Auto-creation of missing attributes | Risky | Automatically creating attribute values in F&O from CAD could pollute master data |
| BOM for Quotation vs. Manufacturing separation | Good | Different business contexts justify separate flows |
| Serial Number tracking | Standard | Well-suited for serialized manufacturing |

**Deep-Dive: Attribute-to-Route Mapping (INT003 Item 1.b)**

This is the most complex integration requirement in the document. The FRD describes:
- Items from CAD arrive with attributes
- A custom screen in F&O maps Item + Attribute combinations to Route IDs
- If an attribute value does not exist in F&O, it is auto-created

| Attribute Combination | Result |
|----------------------|--------|
| Item + Attribute1 + Attribute2 + Attribute3 | Route ID 1 |
| Item + Attribute1 + Attribute4 + Attribute3 | Route ID 2 |
| Item + Attribute3 + Attribute5 | Route ID 3 |

**This is essentially a rules engine for route determination.** This goes beyond simple data integration -- it requires:
- A custom configuration table (Item + Attributes --> Route mapping)
- A custom lookup screen for users to maintain the mapping rules
- Integration logic that evaluates incoming attributes against the rules
- Fallback behavior when no matching rule exists

**Recommended D365 Integration Pattern:**

| Data Flow | Recommended Pattern | Complexity | Rationale |
|-----------|-------------------|-----------|-----------|
| Manufactured Items + Attributes | **Custom D365 Data Entity + DMF Import from Staging** | High | Create a composite data entity that handles item creation + attribute assignment + route lookup in a single transaction. Use DMF recurring import to pull from staging. |
| Attribute-to-Route Mapping | **Custom X++ Configuration Table + Form** | High | Build a dedicated configuration form in F&O where users define attribute-route rules. The integration logic consults this table when processing incoming items. Consider using D365 Product Configurator if attribute combinations are complex. |
| BOM Lines (Quotation) | **Custom OData Action or Custom Service** | Medium | On-demand pull requires an API endpoint in F&O that the user (or a UI button) triggers. A custom OData-bound action on the BOM entity is cleaner than a batch job for on-demand scenarios. |
| BOM Lines (Manufacturing) | **Custom OData Action or Custom Service** | Medium | Same pattern as quotation BOM, but writes to production BOM version instead of quotation BOM. |
| Serial Numbers | **DMF Recurring Import with Serial Number Group** | Low-Medium | Standard serial number data entity exists in D365. Map CAD serial numbers to the Released Product serial number group. Periodic DMF import handles the batch. |

**Detailed Recommendations:**

1. **The attribute auto-creation policy is dangerous and should be gated.** If CAD sends an attribute value that does not exist in F&O, automatically creating it could:
   - Introduce misspellings or duplicates (e.g., "Stainless Steel" vs "stainless steel" vs "SS")
   - Create orphan attributes that clutter the system
   - Bypass any attribute governance process

   **Recommendation:** Instead of auto-creating, place unknown attributes in a **quarantine/review queue**. A data steward reviews and either approves (creates the attribute) or rejects (maps to an existing attribute). This adds a human checkpoint without blocking the integration for known attributes.

2. **BOM for Quotation vs. BOM for Manufacturing should use the same integration mechanism with different BOM types.** D365 supports BOM versions by type. Rather than building two separate integrations:
   - Use a single BOM import entity
   - Distinguish by a "Purpose" field in the staging table (Quotation vs. Manufacturing)
   - The F&O import logic creates the appropriate BOM version type based on this field

3. **Level 1 item pre-existence validation is correct but needs an error workflow.** The FRD states that if a Level 1 item does not exist, "the integration will return an error." Define what happens next:
   - Who gets notified?
   - Does the entire batch fail or just the offending record?
   - Is the failed record retried automatically after the Level 1 item is created?
   - **Recommendation:** Failed records should be moved to an error queue with automatic notification to R&D/Engineering. Successful records in the same batch should proceed. Retry should be automatic on the next polling cycle.

4. **Consider D365 Engineering Change Management (ECM) for manufactured item creation** instead of direct item creation. Since FRD03 already establishes ECM as the R&D framework:
   - CAD items should arrive as Engineering Products (not standard Released Products)
   - This ensures Product Readiness Policy checks are applied
   - Items go through proper engineering release before they can be used in production
   - This aligns the integration with the process already defined in FRD03

---

## Missing Integrations: CRM to F&O

**This is the most significant gap in FRD13.** The following integrations are referenced across FRD01, FRD02, and FRD04 but are completely absent from this document:

### MISSING-01: CRM Opportunities to F&O Projects/Quotations

**Referenced in:** FRD01 (SP003), FRD04 (PJ001)

| Aspect | Requirement | Recommended Pattern | Complexity |
|--------|------------|-------------------|-----------|
| Trigger | DFI approval in CRM triggers F&O opportunity/quotation creation | **Power Automate** (cloud flow) | Medium |
| Data | Opportunity name, customer, products, DFI details, estimated value | Dataverse trigger --> F&O connector | |
| Direction | CRM --> F&O | One-way push | |
| Feedback | F&O quotation number written back to CRM opportunity | Power Automate return step | |

**Recommendation:** Use **Power Automate with the D365 F&O connector** (Fin & Ops Apps connector). The flow triggers on DFI approval status change in Dataverse, calls F&O data entities to create the opportunity/project quotation, and writes the F&O reference number back to the CRM record. This is more appropriate than Dual-Write because it is event-driven (not continuous sync) and involves data transformation (DFI fields mapped to F&O quotation fields).

### MISSING-02: Account & Contact Synchronization (CRM to F&O)

**Referenced in:** FRD01 (Accounts & Contacts section)

| Aspect | Requirement | Recommended Pattern | Complexity |
|--------|------------|-------------------|-----------|
| Trigger | Real-time or near real-time sync | **Dual-Write** | Low-Medium |
| Data | Account name, address, contacts, parent-child hierarchy | Standard Dual-Write maps | |
| Direction | Bidirectional | CRM <--> F&O | |
| Conflict | Last-write-wins or source-system-wins | Configurable in Dual-Write | |

**Recommendation:** **Dual-Write is the standard Microsoft-recommended pattern** for Account/Contact sync between Dataverse and F&O. Out-of-the-box mapping templates exist. Key considerations:
- Enable Dual-Write for Accounts (CRM) <--> Customers/Vendors (F&O)
- Enable Dual-Write for Contacts (CRM) <--> Contact Persons (F&O)
- Parent-child hierarchy in CRM must map to Customer Hierarchy or Invoice Account relationships in F&O
- Dual-Write requires careful initial data load sequencing (F&O first, then sync to CRM, or vice versa)

### MISSING-03: Timesheets from CRM to F&O

**Referenced in:** FRD04 (PJ002)

| Aspect | Requirement | Recommended Pattern | Complexity |
|--------|------------|-------------------|-----------|
| Trigger | Timesheet submitted/approved in CRM | **Power Automate** | Medium-High |
| Data | Worker, project, activity, hours, date, description | Dataverse trigger --> F&O TSTimesheetLine entity | |
| Direction | CRM --> F&O | One-way push | |
| Validation | Must pass F&O timesheet approval workflow | F&O workflow handles approval | |

**Recommendation:** Use **Power Automate** to push approved CRM timesheets to F&O as hour journal lines or timesheet lines. Critical design decisions:
- **Option A:** Push as F&O Timesheet entries (subject to F&O approval workflow). This means double approval (CRM + F&O). May be required for audit reasons.
- **Option B:** Push as approved hour journal entries directly to project cost. Bypasses F&O approval since CRM already approved. Faster but less controlled.
- **Recommendation:** Option A for the initial implementation. It is safer and provides audit trail in both systems. Optimize to Option B later if dual approval proves burdensome.

### MISSING-04: Trips/Missions from CRM to F&O Travel Requisitions

**Referenced in:** FRD01 (SP001)

| Aspect | Requirement | Recommended Pattern | Complexity |
|--------|------------|-------------------|-----------|
| Trigger | Trip/Mission approved in CRM | **Power Automate** | Medium |
| Data | Employee, destination, dates, purpose, estimated expenses | Dataverse trigger --> F&O Travel Requisition entity | |
| Direction | CRM --> F&O | One-way push | |
| Post-creation | User completes expense details in F&O | Manual in F&O | |

**Recommendation:** Use **Power Automate** triggered on CRM Trip/Mission approval. Create a draft Travel Requisition in F&O using the Travel Requisition data entity. Pre-populate as many fields as possible from CRM (traveler, dates, destination, purpose) to minimize re-entry in F&O. As noted in the FRD01 review, consider whether a **Model-Driven Power App** could replace the CRM custom entity for trips entirely.

### MISSING-05: Won/Lost Status Sync (F&O back to CRM)

**Referenced in:** FRD01 (SP003), FRD04 (PJ001)

| Aspect | Requirement | Recommended Pattern | Complexity |
|--------|------------|-------------------|-----------|
| Trigger | Quotation won/lost in F&O | **Power Automate** or **Business Events** | Low-Medium |
| Data | Opportunity reference, win/loss status, reason | F&O Business Event --> Power Automate --> Dataverse update | |
| Direction | F&O --> CRM | One-way push | |

**Recommendation:** Use **D365 F&O Business Events** to fire when a project quotation status changes to Won or Lost. Route the event via Azure Service Bus to a Power Automate flow that updates the CRM Opportunity status. This closes the feedback loop so sales reps see outcomes in CRM without logging into F&O.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Complexity |
|---|------|---------------|----------|------------|
| 1 | Overall | **Expand FRD13 to include ALL integrations** (CRM-F&O, not just CAD-F&O). This document must be the single source of truth for integration architecture. | CRITICAL | N/A (Documentation) |
| 2 | Overall | **Define the staging platform technology** (Azure SQL, Azure Data Factory, or direct API). Do not leave this to developer interpretation. | CRITICAL | Low (Decision) |
| 3 | Overall | **Add an integration monitoring and alerting layer** (Azure Monitor, Power BI dashboard, or Application Insights). Every integration needs health visibility. | High | Medium |
| 4 | Overall | **Add field-level data mapping specifications** for every data flow. The current document specifies what moves but not which fields, formats, or transformations. | CRITICAL | Medium (Documentation) |
| 5 | Overall | **Define error handling strategy** per integration: retry policy, dead-letter queue, notification recipients, manual intervention process. | CRITICAL | Medium |
| 6 | INT002 | Replace custom "PUSHED" flag with **D365 Change Tracking** on data entities for delta detection. | High | Low |
| 7 | INT002 | Use **D365 Business Events + Service Bus** for Production Order Status instead of periodic polling. Near real-time with no polling overhead. | High | Medium |
| 8 | INT002 | Use **DMF Recurring Export** for Project/WBS and Raw Material pushes. Standard D365 pattern with built-in scheduling and monitoring. | High | Low-Medium |
| 9 | INT003 | **Gate attribute auto-creation** with a quarantine/review queue instead of blind auto-creation. Prevents master data pollution. | High | Medium |
| 10 | INT003 | Route incoming manufactured items through **Engineering Change Management** (FRD03) rather than direct product creation. Ensures Product Readiness checks. | High | Medium |
| 11 | INT003 | Unify Quotation BOM and Manufacturing BOM imports into a **single integration with a purpose flag**, not two separate integrations. | Medium | Low |
| 12 | INT003 | Build **attribute-to-route mapping as a custom configuration table** in F&O with a dedicated maintenance form. This is a rules engine, not simple data mapping. | High | High |
| 13 | MISSING | Add **Dual-Write for Account/Contact sync** (CRM <--> F&O). Standard Microsoft pattern with OOB templates. | CRITICAL | Low-Medium |
| 14 | MISSING | Add **Power Automate for DFI-approval-triggered Opportunity/Quotation creation** in F&O (CRM --> F&O). | CRITICAL | Medium |
| 15 | MISSING | Add **Power Automate for Timesheet push** (CRM --> F&O Project Hour Journals). | High | Medium-High |
| 16 | MISSING | Add **Power Automate for Trip/Mission --> Travel Requisition** (CRM --> F&O). | High | Medium |
| 17 | MISSING | Add **Business Events + Power Automate for Won/Lost status** feedback (F&O --> CRM). | Medium | Low-Medium |

---

## Recommended Integration Architecture (Consolidated)

The following table summarizes the recommended D365 integration pattern for every integration point across all FRDs:

| Integration | Source | Target | Pattern | Frequency | Complexity |
|------------|--------|--------|---------|-----------|------------|
| Accounts & Contacts | CRM (Dataverse) | F&O (Customers/Contacts) | **Dual-Write** | Real-time, bidirectional | Low-Medium |
| Opportunity/DFI --> Quotation | CRM | F&O | **Power Automate** (Dataverse trigger --> F&O connector) | Event-driven (on DFI approval) | Medium |
| Won/Lost Status Feedback | F&O | CRM | **Business Events + Power Automate** | Event-driven (on status change) | Low-Medium |
| Timesheets | CRM | F&O | **Power Automate** | Event-driven (on timesheet approval) | Medium-High |
| Trips/Missions --> Travel Req | CRM | F&O | **Power Automate** | Event-driven (on trip approval) | Medium |
| Project/WBS --> Staging | F&O | Staging DB | **DMF Recurring Export** | Periodic (every 30-60 min) | Low-Medium |
| Raw Materials --> Staging | F&O | Staging DB | **DMF Recurring Export with Change Tracking** | Periodic (every 2-4 hours) | Low |
| Production Order Status | F&O | Staging DB | **Business Events + Azure Function** | Event-driven (near real-time) | Medium |
| Manufactured Items + Attributes | CAD --> Staging | F&O | **DMF Recurring Import + Custom Entity** | Periodic (every 30-60 min) | High |
| BOM Lines (Quotation & Mfg) | CAD --> Staging | F&O | **Custom OData Action (on-demand pull)** | On-demand (user-triggered) | Medium |
| Serial Numbers | CAD --> Staging | F&O | **DMF Recurring Import** | Periodic (every 1-2 hours) | Low-Medium |
| Attribute-to-Route Lookup | Internal F&O | Internal F&O | **Custom X++ Configuration Table** | N/A (configuration, not integration) | High |

---

## Risk Flags

1. **FRD13 is dangerously incomplete.** It covers approximately 30% of the integrations that other FRDs depend on. The CRM-to-F&O integrations (5 integration points) are entirely absent. Any development work on FRD01, FRD02, or FRD04 that touches integration will be blocked or will proceed with ad-hoc designs if this document is not expanded immediately.

2. **No integration architect or pattern is named.** The document does not specify who is responsible for integration architecture decisions, what middleware/platform is being used, or what standards developers should follow. This will result in inconsistent implementations across the project.

3. **The "periodically" frequency is undefined and deferred.** The document states "we can discuss with Technica later" regarding polling intervals. This is a capacity planning and infrastructure sizing decision that affects cost and performance. Deferring it creates a risk of discovering at UAT that polling intervals are insufficient.

4. **Attribute-to-Route mapping is a hidden rules engine.** The FRD describes it as part of integration, but it is actually a product configuration rules engine that requires its own design document. Underestimating this as "just an integration" will lead to scope and timeline overruns.

5. **No data volume estimates.** The document does not state how many items, BOMs, serial numbers, or projects will flow through these integrations. Volume directly affects architecture choices (batch vs. real-time, SQL staging vs. message queue, API throttling limits). For D365 F&O, OData calls are throttled at approximately 6,000 requests per 5 minutes by default -- high-volume integrations must account for this.

6. **No testing or validation strategy.** Integration testing is inherently complex (two or more systems must be available). The FRD does not mention how integrations will be tested, what test environments are needed, or what constitutes acceptance criteria for each data flow.

7. **Staging database creates a maintenance burden.** The staging database is a custom component that Technica will need to maintain indefinitely. It requires: database backups, monitoring, data purging policies, schema migration when either F&O or CAD changes, and security management. Ensure Technica's IT team is aware of this ongoing operational responsibility.

8. **Cross-FRD dependency chain is fragile.** The dependency graph is:
   - FRD01 (Sales/CRM) --> FRD13 (Integration) --> FRD04 (Project Accounting)
   - FRD03 (R&D) --> FRD13 (Integration) --> FRD05 (Production)
   - FRD05 (Production) --> FRD13 (Integration) --> CAD System

   If FRD13 is delayed or incomplete, it creates a cascading delay across at least 4 other FRDs. This should be treated as the **critical path** for the project.

---

## Efficiency Verdict

FRD13 as written is **not sufficient to serve as the integration architecture document** for this implementation. The CAD-to-F&O staging pattern is a reasonable starting point, but it covers only one of at least three integration domains (CAD-F&O, CRM-F&O, and potentially third-party systems). The document must be expanded to include:

1. **All CRM-to-F&O integrations** with the same level of detail (or more) as the CAD integrations.
2. **Field-level data mappings** for every data flow -- the current "push Project ID & Project Name" level of detail is insufficient for development.
3. **Technology stack specification** -- which Azure services, which D365 connectors, which authentication mechanisms.
4. **Error handling, monitoring, and operational runbook** for each integration.
5. **Data volume and performance requirements** to size the solution correctly.

The positive aspects are: the staging pattern is pragmatically chosen for CAD integration (avoids direct coupling to a non-Microsoft system), the PUSH/PULL direction semantics are clear, the Level 1 item validation rule is a good data integrity safeguard, and the on-demand vs. periodic distinction for BOM pulls shows understanding of business urgency.

**Bottom line:** This FRD needs to be treated as a **version 0.5 draft**, not a version 2.0. It should be returned to the solution architect for expansion into a full integration design document before any integration development begins. Given its position on the critical path, this expansion should be the **highest priority work item** for the project team.

---

*This review is based on D365 F&O and D365 Sales integration best practices as of 2026, including Dual-Write, Power Platform connectors, Data Management Framework, Business Events, and Azure Integration Services.*
