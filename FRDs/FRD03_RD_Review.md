# FRD03 - R&D - Solution Design Verification Report

**Document:** Technica - FRD03-RD-V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Engineering Change Management Module

---

## ENHANCEMENT SUMMARY

> The table below lists all areas requiring attention, their severity, and where to find them in this document.

| # | Severity | Process / Area | Enhancement | Section |
|---|----------|---------------|-------------|---------|
| 1 | **CRITICAL** | RD001 - New Engineering Item | CAD↔F&O integration architecture is undefined — must be resolved before any R&D development | [RD001 - New Engineering Item](#rd001---new-engineering-item) |
| 2 | **CRITICAL** | RD001 - New Engineering Item | Product creation ownership rules undefined — which system is master for which product type must be decided | [RD001 - New Engineering Item](#rd001---new-engineering-item) |
| 3 | **HIGH** | RD001 - New Engineering Item | Consider PLM connector (third-party) instead of custom CAD↔D365 integration | [RD001 - New Engineering Item](#rd001---new-engineering-item) |
| 4 | **HIGH** | RD001 - New Engineering Item | Double data entry risk — R&D engineers maintaining data in both CAD and F&O will undermine adoption | [RD001 - New Engineering Item](#rd001---new-engineering-item) |
| 5 | **HIGH** | Risk Flags | Unresolved reviewer comments from EA and RR still marked "To be discussed" — must be closed before development | [Risk Flags](#risk-flags) |
| 6 | **MEDIUM** | RD002 - Product Lifetime State | Lifecycle states and their transaction restrictions must be defined upfront | [RD002 - Product Lifetime State](#rd002---product-lifetime-state) |
| 7 | **MEDIUM** | RD002 - Product Lifetime State | Lifecycle states must sync with CAD — discontinued items in F&O should reflect in CAD | [RD002 - Product Lifetime State](#rd002---product-lifetime-state) |
| 8 | **MEDIUM** | RD003 - Engineering Change Request | Merge ECM requirements with FRD02 to avoid conflicting document versions | [RD003 - Engineering Change Request](#rd003---engineering-change-request-mostly-fit) |
| 9 | **MEDIUM** | RD000 - Setup | Versions vs. variants distinction must be clarified for Technica team | [RD000 - Setup](#rd000---setup) |
| 10 | **LOW** | RD003/RD004 | Deadline GAPs are trivial — custom date field + Power Automate alert | [RD003 - Engineering Change Request](#rd003---engineering-change-request-mostly-fit) |

**Totals:** 2 CRITICAL | 3 HIGH | 4 MEDIUM | 1 LOW

---

## Executive Summary

The FRD covers R&D operations using D365's Engineering Change Management (ECM) module: new product creation, lifecycle management, change requests/orders, and product categorization. The solution design correctly leverages the ECM module — **however, the document reveals a critical unresolved issue**: the CAD↔ERP integration architecture is undefined, and multiple reviewer comments (EA, RR) flag concerns about double data entry and unclear product creation ownership.

Overall assessment: **Correct module choice, but the document has significant open questions that must be resolved before development.**

---

## Process-by-Process Review

### RD001 - New Engineering Item

**What was proposed:**
- Sales Support or Marketing creates a job request for new solutions
- R&D creates new engineering products based on requirements
- Engineering Product Categories define attributes, versioning, lifecycle
- Product Readiness Policy ensures items are complete before release
- CAD draws the solution, BOM lines imported to F&O

**Assessment: Right approach, but CAD integration is the critical gap**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Engineering Product creation | Standard OOB | Correct use of ECM |
| Product Categories with attributes | Standard OOB | Well-structured |
| Product Readiness Policy | Excellent | Prevents incomplete items from being used |
| Engineering Attribute Search | Standard OOB | Prevents duplicates |
| CAD↔F&O data flow | **>> NEEDS ARCHITECTURE <<** | Multiple reviewer flags — integration undefined |

**>> ATTENTION AREA: CAD↔ERP ownership is unresolved**

> The document states two options but never resolves them: **Option 1:** Items created in F&O, pushed to CAD (for raw materials/low-level components). **Option 2:** Items created in CAD, pushed to F&O (for manufactured items/assemblies). Reviewer comments (EA, RR) repeatedly flag: *"R&D engineer should not have to work on ERP system to enter or update product data"*, *"Should third party solution be adopted to prevent rework?"*, *"Link between ERP and CAD environment is still not clear."*

**Recommendations:**

1. **>> ENHANCEMENT (CRITICAL):** This is the #1 risk for the R&D workstream. Before any development, Technica must decide:
   - **Which system is the master for which product types?** (Recommended: CAD is master for manufactured items, F&O is master for purchased items)
   - **What is the sync mechanism?** (API, file-based, middleware like Vaulted/PDM Link?)
   - **What is the sync frequency?** (Real-time, batch, on-demand?)

2. **>> ENHANCEMENT (HIGH):** Consider a PLM connector. Several third-party solutions exist for CAD↔D365 integration:
   - **Autodesk Vault ↔ D365** connectors
   - **SolidWorks PDM ↔ D365** integrations
   - **Custom middleware via Azure Integration Services**

   A third-party connector would be more reliable than a custom-built integration and would address the reviewer concerns about double data entry.

3. **>> ENHANCEMENT:** If custom integration is chosen, establish clear rules:
   - Raw materials / purchased items: Created in F&O → synced to CAD (read-only in CAD)
   - Manufactured items / assemblies: Created in CAD → synced to F&O via ECM
   - BOM structures: Maintained in CAD → imported to F&O
   - Engineering attributes: Maintained in F&O (CAD doesn't need all ERP attributes)

4. The Product Readiness Policy is an excellent safeguard. It ensures items aren't used in quotations or production until cost, BOM, and route are properly set. Keep this.

---

### RD002 - Product Lifetime State

**What was proposed:**
- Control product lifecycle (Active, Discontinued, custom states)
- State changes via manual update or Engineering Change Order

**Assessment: Standard OOB - no issues**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Lifecycle state management | Standard OOB | Built into ECM |
| State-driven restrictions | Standard OOB | Prevents transactions on discontinued items |
| Lifecycle state definitions | **>> NEEDS REVIEW <<** | States and restrictions must be defined upfront |
| CAD sync of lifecycle states | **>> NEEDS REVIEW <<** | Discontinued items must reflect in CAD |

**>> ATTENTION AREA: Lifecycle states need upfront definition**

> Without predefined lifecycle states and their associated transaction restrictions, implementation will be ad hoc. States like "In Development → Active → Phase Out → Discontinued" must be mapped to specific transaction blocks (e.g., "Phase Out" allows existing orders but blocks new quotations).

**Recommendations:**

1. **>> ENHANCEMENT (MEDIUM):** Define the lifecycle states upfront (e.g., In Development → Active → Phase Out → Discontinued). Map each state to transaction restrictions (e.g., "Phase Out" allows existing orders but blocks new quotations).

2. **>> ENHANCEMENT (MEDIUM):** Ensure lifecycle states sync with CAD — if an item is discontinued in F&O, CAD should reflect this.

---

### RD003 - Engineering Change Request (Mostly FIT)

**What was proposed:**
- Change requests initiated by Sales Support, Business Development, Sales, or from SNAGs
- Workflow approval by product category owner (Mechanical/Electrical HOD)
- GAP: Deadline field needed

**Assessment: Standard ECM — correct approach**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Change request creation | Standard OOB | Multiple initiators supported |
| Workflow approval | Standard OOB | Product owner-based |
| Stage tracking | Standard OOB | Built-in lifecycle |
| Deadline (GAP) | Trivial | Custom date field + alert |
| Document overlap with FRD02 | **>> NEEDS REVIEW <<** | Repetitive content risks conflicting versions |

**>> ATTENTION AREA: Document overlap with FRD02**

> This section is acknowledged as "repetitive from sales support document." Maintaining the same ECM requirements in two documents risks conflicting versions and implementation inconsistencies.

**Recommendations:**

1. **>> ENHANCEMENT (LOW):** Deadline GAP (RD003-05): Add a custom "Due Date" field + Power Automate notification. Identical to the same GAP in FRD02 (SS003-05) — implement once, reuse.

2. **>> ENHANCEMENT (MEDIUM):** Merge the Engineering Change Management requirements into a single reference to avoid conflicting versions between FRD02 and FRD03.

---

### RD004 - Engineering Change Order (Mostly FIT)

**What was proposed:**
- Change orders created from approved change requests
- Supports: new product creation, product updates, discontinuation
- Separate workflows per change type
- Attachments (PLC, HMI docs) as URL links
- SNAG feedback tracked

**Assessment: Standard ECM - well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Change order from request | Standard OOB | Proper ECM flow |
| Multiple change types | Standard OOB | New, update, discontinue |
| Workflow per type | Standard OOB | Flexible workflow framework |
| URL attachments | Standard OOB | Document management |
| Due date (GAP) | Trivial | Same as RD003-05 |

**Recommendations:**

1. SNAG tracking is well-placed here. Using Engineering Change Requests for customer feedback/snags ensures they go through proper approval before affecting products. Good design.

2. **>> ENHANCEMENT:** Consider enabling Engineering Change Order impact analysis (OOB feature) — it shows which BOMs, routes, and orders are affected by a proposed change before execution.

---

### RD000 - Setup

**What was proposed:**
- Engineering Product Categories (Packer, Palletizer, Elevator, Transfer Car)
- Product Owners (user groups with release authority)
- Engineering Attributes (brand, model, technical specs like belt size, motor speed)
- Product Variants from Product Masters

**Assessment: Well-structured setup plan**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Product categories | Standard OOB | Clear initial list provided |
| Product owners | Standard OOB | Good access control |
| Engineering attributes | Standard OOB | Flexible metadata model |
| Product variants | Standard OOB | Size, config dimensions |
| Versions vs. variants clarity | **>> NEEDS REVIEW <<** | Frequently confused — must be clarified |

**>> ATTENTION AREA: Versions vs. variants confusion risk**

> Engineering versions (design iterations of the same product — tracked by ECM) and product variants (deliberate size/color/config options — tracked by Product Dimensions) are often confused. Misunderstanding this distinction leads to incorrect data modeling.

**Recommendations:**

1. Generic product owner for early-stage items is a good workaround. The plan to reassign via ECO later is correct.

2. **>> ENHANCEMENT:** Engineering attributes should be defined carefully. They drive search, reporting, and category inheritance. Recommend:
   - Start with a minimal set of critical attributes per category
   - Add more attributes via ECO as needs emerge
   - Don't over-engineer the attribute model upfront

3. **>> ENHANCEMENT (MEDIUM):** Product variants (Product Masters): Ensure Technica understands the difference between:
   - **Engineering versions** (design iterations of the same product — tracked by ECM)
   - **Product variants** (deliberate size/color/config options — tracked by Product Dimensions)
   - These are often confused. Versions ≠ Variants.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | RD001 | **Resolve CAD↔F&O integration architecture immediately** — this blocks all R&D processes | CRITICAL |
| 2 | RD001 | Consider PLM connector (third-party) instead of custom integration | High |
| 3 | RD001 | Define product creation ownership rules (which system is master for which item type) | CRITICAL |
| 4 | RD002 | Define lifecycle states and their transaction restrictions upfront | Medium |
| 5 | RD003/04 | Deadline GAPs are trivial — custom date field + alert | Low |
| 6 | RD003 | Merge ECM requirements with FRD02 to avoid conflicting versions | Medium |
| 7 | RD000 | Clarify versions vs. variants distinction for Technica team | Medium |

---

## Risk Flags

1. **CAD↔F&O integration is the single biggest risk across FRD02 and FRD03.** Multiple reviewer comments flag this as unresolved. Without clarity on this, R&D processes cannot be properly implemented. This should be a **blocking decision item** for the project.

2. **>> ATTENTION AREA: Double data entry risk**

> If R&D engineers must maintain product data in both CAD and F&O, it will increase workload and frustration, create data inconsistency risks, and undermine adoption.

3. **>> ATTENTION AREA: Unresolved reviewer comments**

> Comments from EA and RR are still marked as "To be discussed internally at Technica and Advise Back." These must be closed before development begins.

---

## Efficiency Verdict

The solution **correctly uses D365 Engineering Change Management**, which is the right module for Technica's R&D operations. The ECM features (change requests, change orders, product readiness, lifecycle states, product owners) are all appropriate and well-mapped to Technica's needs.

However, the **efficiency of the overall R&D solution depends entirely on the CAD↔F&O integration**, which is currently undefined. A poorly designed integration will negate all the benefits of ECM. This must be resolved as a priority.

---

*This review is based on D365 F&O Engineering Change Management best practices as of 2026.*
