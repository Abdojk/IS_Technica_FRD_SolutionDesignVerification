# FRD03 - R&D - Solution Design Verification Report

**Document:** Technica - FRD03-RD-V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Engineering Change Management Module

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
| CAD↔F&O data flow | UNRESOLVED | Multiple reviewer flags |

**Critical Issue - CAD↔ERP Ownership:**

The document states two options but never resolves them:
- **Option 1:** Items created in F&O, pushed to CAD (for raw materials/low-level components)
- **Option 2:** Items created in CAD, pushed to F&O (for manufactured items/assemblies)

Reviewer comments (EA, RR) repeatedly flag:
- *"R&D engineer should not have to work on ERP system to enter or update product data"*
- *"Should third party solution be adopted to prevent rework?"*
- *"Link between ERP and CAD environment is still not clear"*

**Recommendations:**

1. **This is the #1 risk for the R&D workstream.** Before any development, Technica must decide:
   - **Which system is the master for which product types?** (Recommended: CAD is master for manufactured items, F&O is master for purchased items)
   - **What is the sync mechanism?** (API, file-based, middleware like Vaulted/PDM Link?)
   - **What is the sync frequency?** (Real-time, batch, on-demand?)

2. **Consider a PLM connector.** Several third-party solutions exist for CAD↔D365 integration:
   - **Autodesk Vault ↔ D365** connectors
   - **SolidWorks PDM ↔ D365** integrations
   - **Custom middleware via Azure Integration Services**

   A third-party connector would be more reliable than a custom-built integration and would address the reviewer concerns about double data entry.

3. **If custom integration is chosen**, establish clear rules:
   - Raw materials / purchased items: Created in F&O → synced to CAD (read-only in CAD)
   - Manufactured items / assemblies: Created in CAD → synced to F&O via ECM
   - BOM structures: Maintained in CAD → imported to F&O
   - Engineering attributes: Maintained in F&O (CAD doesn't need all ERP attributes)

4. **The Product Readiness Policy is an excellent safeguard.** It ensures items aren't used in quotations or production until cost, BOM, and route are properly set. Keep this.

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

**Recommendation:**
- Define the lifecycle states upfront (e.g., In Development → Active → Phase Out → Discontinued)
- Map each state to transaction restrictions (e.g., "Phase Out" allows existing orders but blocks new quotations)
- **Ensure lifecycle states sync with CAD** — if an item is discontinued in F&O, CAD should reflect this

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

**Recommendations:**
1. **Deadline GAP (RD003-05):** Add a custom "Due Date" field + Power Automate notification. Identical to the same GAP in FRD02 (SS003-05) — implement once, reuse.

2. **Note:** This section is acknowledged as "repetitive from sales support document." Consider merging the Engineering Change Management requirements into a single reference to avoid conflicting versions.

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
1. **SNAG tracking is well-placed here.** Using Engineering Change Requests for customer feedback/snags ensures they go through proper approval before affecting products. Good design.

2. **Consider enabling Engineering Change Order impact analysis** (OOB feature) — it shows which BOMs, routes, and orders are affected by a proposed change before execution.

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

**Recommendations:**

1. **Generic product owner for early-stage items** is a good workaround. The plan to reassign via ECO later is correct.

2. **Engineering attributes should be defined carefully.** They drive search, reporting, and category inheritance. Recommend:
   - Start with a minimal set of critical attributes per category
   - Add more attributes via ECO as needs emerge
   - Don't over-engineer the attribute model upfront

3. **Product variants (Product Masters):** Ensure Technica understands the difference between:
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

2. **Double data entry risk.** If R&D engineers must maintain product data in both CAD and F&O, it will:
   - Increase workload and frustration
   - Create data inconsistency risks
   - Undermine adoption

3. **Document has unresolved reviewer comments.** Comments from EA and RR are still marked as "To be discussed internally at Technica and Advise Back." These must be closed before development begins.

---

## Efficiency Verdict

The solution **correctly uses D365 Engineering Change Management**, which is the right module for Technica's R&D operations. The ECM features (change requests, change orders, product readiness, lifecycle states, product owners) are all appropriate and well-mapped to Technica's needs.

However, the **efficiency of the overall R&D solution depends entirely on the CAD↔F&O integration**, which is currently undefined. A poorly designed integration will negate all the benefits of ECM. This must be resolved as a priority.

---

*This review is based on D365 F&O Engineering Change Management best practices as of 2026.*
