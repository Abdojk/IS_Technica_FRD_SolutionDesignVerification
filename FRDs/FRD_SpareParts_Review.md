# FRD - Spare Parts - Solution Design Verification Report

**Document:** FRD - Spare Parts - V3.0
**Prepared by:** Edward Khoury | Contributors: Abdo Khoury, Jalal Iskandar
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Sales & Marketing, Procurement & Sourcing, Master Planning

---

## Executive Summary

This FRD covers 4 core spare parts processes: Spare Parts Request (quotation preparation), Service Requests, Spare Parts Packages, and Order Execution, plus a Master Planning section for spare parts replenishment. The solution leverages **standard D365 Sales Quotation, RFQ, Purchase Requisition, and Master Planning** functionality. Most requirements are FIT (all 8 Master Planning requirements and 3 of 5 Order Execution requirements). There are **3 GAPs**, all in Order Execution, related to email notifications and customer communication.

However, this review identifies **several process design concerns** beyond the formal GAP list. Multiple steps rely on manual email exchanges outside D365 (drawings to factory, execution cost requests, shipping cost estimates), which undermines the value of the ERP and creates traceability gaps. The FRD also lacks detail on pricing policy configuration, BOM/route integration for make-to-order spare parts, and the handoff between Sales Quotation and Production Order.

Overall assessment: **Solid use of standard D365 for quotation, RFQ, and master planning. However, the order execution process for Technica-made parts has significant manual/offline steps that should be brought into D365 using Production Orders, Case Management, or Power Automate to close traceability gaps.**

---

## Process-by-Process Review

### SP-001 - Spare Parts Request / Quotation Preparation (0 formal GAPs)

**What was proposed:**
- Sales Quotation creation in the correct legal entity (Technica International, Technica Europe) with predefined number sequences (SPTXXX-YY, SPEXX-YY, SPT Project#)
- Customer and contact person populated on quotation
- Items can be Technica-made or foreign parts
- For Technica-made parts: check Released Products, calculate price from pricing policies and product catalog, or retrieve drawings and email Factory Manager for cost estimate
- For foreign parts: create RFQ in D365 for items priced > 2 weeks ago, receive quotes, update costs
- Shipping cost calculated via DHL website (<70kg) or logistics officer email (>70kg)
- Weight verification in D365
- Final Sales Quotation generated in D365

**Assessment: Standard Sales Quotation process, but several offline steps reduce ERP value**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Sales Quotation creation | Standard OOB | Multi-entity with number sequences |
| Number sequence per entity | Standard OOB | Configurable per legal entity |
| Customer/contact population | Standard OOB | Customer master + contacts |
| Released Product check | Standard OOB | Item existence validation |
| Price list / price history | Standard OOB | Trade agreements, price journal history |
| RFQ for foreign parts | Standard OOB | With auto-email to suppliers |
| Item weight management | Standard OOB | Product dimension on Released Product |
| Quotation generation/print | Standard OOB | Print Management |

**Process Concerns (not listed as GAPs but should be addressed):**

| # | Concern | Impact | Recommendation |
|---|---------|--------|----------------|
| 1 | Cost estimate for Technica-made parts is done via email to Factory Manager (steps 10-12) | No audit trail, delays, no visibility in D365 | **Use D365 Cost Estimation on BOM/Route.** If the item has a BOM and route, D365 can calculate standard cost automatically. For new items, create BOM + route first, then run cost rollup. This eliminates the 3-day email wait and provides auditable cost calculations. |
| 2 | New items are created in D365 only after Factory Manager provides cost | Item master creation is late in the process | **Create the Released Product early** with a "Pending Costing" status. Use the item lifecycle state to prevent it from being sold/ordered until cost is confirmed. This keeps item data in D365 from the start. |
| 3 | Shipping cost is calculated outside D365 (DHL website or logistics email) | Manual, not auditable, not reusable | **Configure D365 Transportation Management (TMS)** with carrier rate engines, or at minimum configure **Auto Charges** with weight-based shipping tiers. For DHL specifically, consider DHL's API integration via Power Automate to auto-fetch rates. |
| 4 | Pricing policies and product catalog are referenced but not detailed | Risk of inconsistent pricing | **Define Trade Agreements** (price/discount journals) in D365 with customer group/item group matrix. Use **Supplementary Items** for frequently paired parts. Set up **Price simulation** on quotation to test margins before sending. |
| 5 | No mention of quotation expiry or follow-up | Lost revenue opportunity | **Set Quotation expiry dates** and use **Sales Quotation follow-up** activities. Configure quotation status reasons (Won, Lost, Cancelled) for pipeline tracking. |

**Recommendations:**

1. **BOM/Route-based costing is the single biggest improvement opportunity.** For a make-to-order manufacturer, every manufactured spare part should have a BOM and route in D365. The cost rollup engine replaces the 3-day email cycle for cost estimation and provides consistent, auditable pricing. Even if drawings need to be reviewed, the BOM can be created in D365 and the cost calculated instantly.

2. **Trade Agreement automation:** Set up trade agreements with price validity periods. When the quotation is created, D365 automatically pulls the current price. The "price items priced more than two weeks ago" rule for RFQs can be enforced via trade agreement expiry dates rather than manual checking.

3. **Consider Quotation Templates** for frequently requested spare parts packages. D365 supports quotation templates that pre-populate common item combinations.

---

### SP-002 - Service Requests (0 formal GAPs)

**What was proposed:**
- Spare Parts Coordinator receives Spare Parts & Services DFI (Demand for Information) from Customer Service via email
- Coordinator creates Quotation in D365 and fills DFI data in D365 CRM
- Two-level workflow approval: HSP first, then CSM
- Rejection triggers revision and resubmission
- Approved quotation confirmation sent to customer

**Assessment: Standard workflow-driven quotation approval -- clean design**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Sales Quotation creation | Standard OOB | From service inquiry |
| Two-level workflow approval | Standard OOB | Sequential approval with rejection loop |
| Automated email on approval | Standard OOB | Workflow email notification |
| Quotation confirmation to customer | Standard OOB | Print Management with email delivery |
| CRM integration for DFI | Depends | D365 CRM (CE) integration via Dual Write or Dataverse |

**Process Concerns:**

| # | Concern | Impact | Recommendation |
|---|---------|--------|----------------|
| 1 | DFI received by email and manually entered into CRM | Duplicate data entry, potential errors | **Use D365 Customer Service Case Management** to capture DFI as a Case, then link the Case to the Sales Quotation. This provides end-to-end traceability from inquiry to order. Alternatively, use **Power Automate to parse incoming DFI emails** and auto-create cases. |
| 2 | CRM data entry is mentioned but integration detail is missing | Risk of CRM/ERP data mismatch | **Clarify whether D365 CE (CRM) is in scope.** If yes, use Dual Write for real-time sync. If not, consider using the D365 F&O Prospect-to-Cash integration or simply keep everything in F&O Sales Quotation with custom fields for DFI data. |

**Recommendations:**

1. **The two-level approval workflow is appropriate** for service quotations given the margin and compliance implications. No changes needed.

2. **Link quotations to Service Orders** (referenced in the FRD as "See Service Order FRD for more details"). Ensure the Sales Quotation has a reference field to the Service Order for full traceability.

---

### SP-003 - Spare Parts Packages (0 formal GAPs)

**What was proposed:**
- SP Sales Engineer receives email from Project Manager about spare parts package inclusion
- Receives notification when Purchase Requisitions are done
- Prepares SP Package based on specified items, confirmed with PM or R&D
- Generates offer on D365

**Assessment: Minimal process -- essentially a quotation with project linkage**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Sales Quotation creation for packages | Standard OOB | Quotation with multiple lines |
| Project linkage | Standard OOB | Project reference on quotation header |
| Item selection from project BOMs | Needs Clarity | How are items identified for the package? |

**Process Concerns:**

| # | Concern | Impact | Recommendation |
|---|---------|--------|----------------|
| 1 | Package items are specified via email from PM/R&D | No structured data flow | **Use D365 Project BOM or Maintenance BOM** to define standard spare parts packages per machine type. The Sales Engineer can then pull recommended spare parts lists from the BOM structure instead of relying on ad-hoc email communication. |
| 2 | No mention of discount structure for packages | Missed revenue optimization | **Configure Multiline Discount Groups** in D365 to auto-apply package discounts when a customer orders a complete spare parts package. This incentivizes larger orders. |
| 3 | The process is very light on detail (only 4 steps) | Implementation risk from ambiguity | Expand this process to include: item validation, pricing, margin check, approval workflow (if needed), and quotation delivery. Align with SP-001 process for consistency. |

**Recommendations:**

1. **Create Spare Parts Package templates as Quotation Templates or Supplementary Item groups** in D365. For each machine model Technica manufactures, define a recommended spare parts list. This standardizes packages and speeds up quotation preparation.

2. **The "SPT Project#" numbering convention** for package-related offers is a good practice. Ensure the project reference is carried through to the Sales Order and eventual invoicing for project cost tracking.

---

### SP-004 - Order Execution (3 GAPs)

**What was proposed:**
- Customer PO received by email
- Sales Order confirmation generated in D365 with PO number and delivery date
- Order confirmation sent automatically from D365 to customer, Finance, Purchasing/CS with attachments
- Foreign parts: Purchase Requisition created and approved through 3-level workflow (HSP, Purchasing Manager, FM)
- Technica-made parts: Execution request with drawings sent to factory by email
- Factory Manager approves execution drawings by email
- Parts tracked at every stage in factory and on D365 PR
- Shipment list prepared, sent to Manufacturing Assistant
- Local customers: notified by email when order is ready
- Foreign customers: follow-up until delivery
- Order closed in D365

**Assessment: Foreign parts process is solid. Technica-made parts process has significant offline gaps.**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Sales Order creation from quotation | Standard OOB | Quotation → Order conversion |
| PO number on Sales Order | Standard OOB | Customer requisition/PO reference field |
| Order confirmation print/email | Standard OOB | Print Management with email delivery |
| PR creation for foreign parts | Standard OOB | With workflow approval |
| 3-level PR approval workflow | Standard OOB | Sequential: HSP → Purchasing → FM |
| Sales Order closure | Standard OOB | Invoice and close |

| GAP ID | Description | Complexity | Current Proposal | Recommendation |
|--------|-------------|-----------|-----------------|----------------|
| IN05-001 | Send acknowledgement receipt to customer for order confirmation | Low | Gap - custom notification | **Use Power Automate** triggered on Sales Order creation/confirmation to send a formatted acknowledgement email. Alternatively, use **D365 Print Management** with email destination on the Sales Order Confirmation document. This is essentially the same as the order confirmation in step 2-3, so verify this isn't already covered by the standard confirmation email. May not actually be a GAP. |
| IN05-002 | Order confirmation notification sent to Logistics, Finance, and Purchasing departments | Low | Gap - internal notification | **Use D365 Workflow notifications** or **Power Automate**. When the Sales Order is confirmed, trigger an email to distribution lists for each department. D365 standard **Alerts** can also be configured: "Notify me when Sales Order status = Confirmed" per user. Consider using **Business Events** framework in D365 to publish order confirmation events that Power Automate subscribes to. |
| IN05-003 | SP&S department notified when spare parts have been received | Low | Fit (marked as Fit in FRD) | Standard. Use **Product Receipt posting** as trigger for notification. D365 Alerts on product receipt journal posting, or Power Automate on the Business Event for product receipt. |

**Process Concerns (beyond formal GAPs):**

| # | Concern | Impact | Recommendation |
|---|---------|--------|----------------|
| 1 | Technica-made parts: execution request sent to factory by email with drawings (step 8) | **Critical gap** -- production is outside D365 | **Create Production Orders in D365** linked to the Sales Order line. This is fundamental for a manufacturing ERP. The BOM and Route define what to produce and how. The Production Order provides: scheduling, material reservation, job tracking, cost accumulation, and completion reporting -- all within D365. This replaces the email-based execution request entirely. |
| 2 | Factory Manager approves execution drawings by email (step 9) | No audit trail, no D365 visibility | **Use Production Order workflow** for approval before release. Attach drawings to the Production Order using D365 Document Management. The Factory Manager approves within D365, which releases the order to the shop floor. |
| 3 | Parts tracked "at every stage in the factory and on PR D365FO" (step 10) | Tracking on PR is incorrect -- PRs are for procurement, not production | **Track via Production Order status** (Created → Estimated → Scheduled → Released → Started → Reported as Finished → Ended). This provides real-time production visibility. If shop floor tracking is needed, consider **D365 Manufacturing Execution System (MES)** or **Job Card Device** for operators to report progress. |
| 4 | Delivery voucher prepared manually and sent to MA and Finance (step 13) | Manual document handling | **Use D365 Packing Slip** for delivery documentation. The packing slip is generated from the Sales Order, updates inventory, and provides the delivery voucher equivalent. Finance visibility is automatic through the posting. |
| 5 | Order "closed" in D365 but no mention of invoicing process | Revenue recognition gap | **Clarify the invoicing process.** The standard flow is: Sales Order → Packing Slip (delivery) → Invoice (revenue recognition). Ensure the invoicing step is documented and linked to the Accounts Receivable process. |

**Recommendations:**

1. **Production Orders for Technica-made parts is the most critical recommendation in this entire FRD.** As a make-to-order manufacturer, Technica should use Sales Order → Production Order → BOM Explosion → Shop Floor Execution → Report as Finished → Delivery. This is core D365 Manufacturing functionality and eliminates the email-based factory coordination entirely.

2. **The 3-level PR approval (HSP → Purchasing → FM) for foreign parts** is thorough but may be slow. For low-value spare parts, consider a **spending threshold** where items below a certain value (e.g., $500) require only 1 level of approval. D365 workflow conditions support amount-based routing.

3. **GAPs IN05-001 and IN05-002 are both notification requirements** that should be solved together using a single approach: either Power Automate Business Event subscriptions or D365 Alert rules. Do not build two separate custom solutions for what is essentially the same pattern (event-driven email notification).

---

### SP-005 - Spare Parts Master Planning (0 GAPs -- All FIT)

**What was proposed:**
- Inventory/Maintenance department defines Min/Max quantities for spare parts
- Master Planning engine runs periodically to generate Planned Purchase Orders
- Requesting department reviews and firms Planned POs
- Procurement department processes confirmed POs per standard PO process

**Assessment: Standard Min/Max Master Planning -- clean and correct**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Min/Max definition on items | Standard OOB | Item coverage settings |
| Master Planning scheduling | Standard OOB | Periodic batch job |
| Planned PO generation | Standard OOB | Auto-calculated based on Min/Max and on-hand |
| Planned PO firming | Standard OOB | Manual review + convert to PO |
| Vendor lead time definition | Standard OOB | Item default order settings or trade agreements |
| Demand Forecast registration | Standard OOB | Forecast planning integration |
| PO processing | Standard OOB | Links to Procurement FRD |

| Requirement ID | Description | Status | Comment |
|----------------|-------------|--------|---------|
| PR002-001 | Master Planning Configuration | FIT | Standard setup |
| PR002-002 | Vendor/Product Lead Time Definition | FIT | Default order settings |
| PR002-003 | Demand Forecast Planning Registration | FIT | Demand forecast tables |
| PR002-004 | Master Planning Scheduling | FIT | Batch scheduling |
| PR002-005 | Generation of Planned PO | FIT | Auto from Min/Max |
| PR002-006 | Firming PO | FIT | Manual firm action |
| PR002-007 | Process PO | FIT | Standard procurement |
| PR002-008 | Define Min/Max Qty on Items | FIT | Item coverage rules |

**Recommendations:**

1. **Consider using Planning Optimization** (the modern Master Planning engine in D365) instead of the deprecated built-in Master Planning. Planning Optimization runs in near-real-time, is cloud-native, and is Microsoft's strategic direction. The deprecated engine will eventually be removed.

2. **Min/Max is appropriate for spare parts** with relatively stable demand. However, for critical spare parts with long lead times, consider adding **Safety Stock** quantities and **Reorder Point** coverage in addition to Min/Max.

3. **Define Item Coverage Groups** to differentiate planning behavior: consumable spare parts (Min/Max), critical spare parts (Safety Stock + Reorder Point), and project-specific spare parts (Make-to-Order, driven by Sales Order demand). This prevents over-ordering of expensive, project-specific parts.

4. **Ensure vendor lead times are accurate and periodically reviewed.** Master Planning accuracy is directly dependent on lead time data. Consider a quarterly lead time review process.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority | Effort |
|---|------|---------------|----------|--------|
| 1 | SP-001 / SP-004 | **Implement Production Orders for Technica-made spare parts** instead of email-based factory coordination. Use BOM + Route → Production Order → Shop Floor tracking. This is the single most impactful improvement. | **Critical** | Medium-High |
| 2 | SP-001 | **Use D365 BOM/Route cost rollup** for Technica-made part costing instead of emailing Factory Manager for cost estimates. Eliminates 3-day delay. | **High** | Medium |
| 3 | SP-001 | **Configure Trade Agreements** with validity periods for automated pricing. Replace manual price checking with system-driven price expiry. | **High** | Low-Medium |
| 4 | SP-001 | **Evaluate Transportation Management (TMS)** or Auto Charges for shipping cost calculation instead of manual DHL website lookups and logistics emails. | **Medium** | Medium |
| 5 | SP-004 | **Solve all notification GAPs (IN05-001, IN05-002) with a unified Power Automate / Business Events approach.** Do not build separate custom solutions for each notification. | **Medium** | Low |
| 6 | SP-004 | **Replace delivery voucher manual process with D365 Packing Slip** for standard delivery documentation and inventory updates. | **Medium** | Low |
| 7 | SP-004 | **Add spending thresholds to PR approval workflow** for foreign parts to reduce approval cycles on low-value items. | **Low** | Low |
| 8 | SP-003 | **Create Quotation Templates / Supplementary Item groups** for standard spare parts packages per machine model. | **Medium** | Low |
| 9 | SP-005 | **Migrate to Planning Optimization** instead of deprecated Master Planning engine. | **Medium** | Low-Medium |
| 10 | SP-005 | **Define differentiated Item Coverage Groups** (consumable vs. critical vs. project-specific spare parts) for smarter planning behavior. | **Medium** | Low |
| 11 | SP-002 | **Clarify CRM integration approach** -- Dual Write vs. Dataverse virtual entities vs. F&O-only for DFI capture. | **Medium** | Depends |
| 12 | SP-001 | **Set quotation expiry dates and configure follow-up activities** for pipeline management. | **Low** | Low |

---

## Risk Flags

1. **Production execution outside D365 is the highest risk.** For Technica-made spare parts, the entire production process (execution request, drawing approval, shop floor tracking) happens via email outside of D365. This means D365 has no visibility into production status, no cost accumulation, no material consumption tracking, and no capacity planning for spare parts production. For a make-to-order manufacturer, this is a fundamental gap in ERP utilization. If Production Orders are not implemented for spare parts, Technica is essentially running a parallel manual system alongside D365.

2. **Costing accuracy risk.** Without BOM/Route-based cost rollup, spare part costs are manually estimated by the Factory Manager via email. This creates risk of inconsistent costing, margin erosion, and inability to perform standard cost variance analysis. D365's costing engine is being bypassed.

3. **Tracking spare parts production on Purchase Requisitions (step 10 of Order Execution) is architecturally incorrect.** Purchase Requisitions are procurement documents, not production tracking tools. If this approach is implemented, it will create confusion in procurement reporting and make it impossible to distinguish between parts being purchased externally and parts being manufactured internally.

4. **Notification GAPs (IN05-001, IN05-002) are low-risk but high-frequency.** These are simple email notifications that should not be over-engineered. Using Power Automate with D365 Business Events is the most maintainable approach. Avoid X++ customization for notifications.

5. **Master Planning deprecation risk.** If Technica implements the legacy Master Planning engine, they will need to migrate to Planning Optimization before Microsoft removes the deprecated engine. Implementing Planning Optimization from the start avoids this future migration cost.

6. **No mention of Quality Management for spare parts.** For a manufacturing company, incoming inspection of foreign parts and quality checks on manufactured spare parts should be integrated. The absence of quality controls creates risk of shipping defective parts to customers.

---

## Efficiency Verdict

The Spare Parts FRD demonstrates **competent use of D365 standard functionality for quotation management, RFQ processing, and master planning**. The Service Request workflow approval and Master Planning configuration are well-designed and fully leverage OOB capabilities. The GAP count is low (3 formal GAPs, all minor notifications), which is a positive indicator.

However, the FRD has a **critical structural weakness**: the production execution process for Technica-made spare parts operates entirely outside D365 via email. For a make-to-order manufacturer implementing D365 F&O, this represents an underutilization of the ERP's core manufacturing capabilities. The Production Control module (Production Orders, BOM management, Route operations, Shop Floor Execution) exists precisely for this purpose and should be the backbone of spare parts manufacturing, not an afterthought.

**The solution as designed is approximately 60% efficient.** Quotation, procurement, and planning processes are well-handled. But the manufacturing execution gap means Technica will not achieve full visibility, cost control, or process automation for internally-produced spare parts -- which, for a make-to-order manufacturer, is likely the majority of their spare parts business.

**To reach 90%+ efficiency**, the top 3 actions are:
1. Implement Production Orders for Technica-made spare parts (eliminates email-based factory coordination)
2. Use BOM/Route cost rollup for automated costing (eliminates 3-day cost estimation delay)
3. Unify all notification GAPs under Power Automate + Business Events (clean, maintainable, no X++)

---

*This review is based on D365 F&O Sales & Marketing, Production Control, Procurement & Sourcing, and Master Planning best practices as of 2026.*
