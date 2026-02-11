# FRD06 - Logistics - Solution Design Verification Report

**Document:** FRD06 - Logistics - V2.0
**Prepared by:** Abdo Khoury | Contributor: Antonio Saleh
**Review Date:** 2026-02-11
**Platform:** Dynamics 365 F&O - Landed Cost, Transportation Management, Warehouse Management

---

## Executive Summary

This FRD covers three logistics processes: Inbound Shipments (via Landed Cost module), Direct Delivery, and Outbound Shipments (via Transportation Management). The solution makes **excellent use of D365 Landed Cost** for voyage tracking, in-transit warehousing, and charge allocation. Most requirements are FIT with only 3 minor GAPs.

Overall assessment: **Well-designed, clean, and efficient logistics solution with minimal customization needed.**

---

## Process-by-Process Review

### LOG001 - Inbound Shipments (1 GAP)

**What was proposed:**
- Voyage creation for inbound shipments
- Journey templates with legs and lead times
- Charges added to voyage (landed cost allocation)
- Multiple PO lines assigned to single voyage
- Container tracking
- "Documents Received" status update with email notification
- In-Transit warehouse transfer when goods ship
- Live tracking of confirmed delivery dates on PO lines
- Finance can invoice while goods are in transit (ownership transferred)
- Warehouse receives goods physically from voyage

**Assessment: Textbook Landed Cost implementation — excellent**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Voyage creation & management | Standard OOB | Landed Cost module |
| Journey templates & legs | Standard OOB | With lead time tracking |
| Charge codes on voyage | Standard OOB | Proper cost allocation |
| Multi-PO assignment to voyage | Standard OOB | Mix & match lines |
| Container tracking | Standard OOB | All Shipping Containers view |
| In-Transit warehouse | Standard OOB | Proper goods-in-transit accounting |
| Delivery date tracking | Standard OOB | Auto-updates PO confirmed date |
| Invoice before receipt | Standard OOB | Standard when ownership transfers |

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| LOG001-08 | Email notification to logistics when documents received | **Power Automate** trigger on voyage status change → email to logistics team. Very low complexity (~1 hour). |

**Recommendations:**

1. **This is one of the best-designed processes across all FRDs.** The Landed Cost module is used exactly as intended. No changes recommended.

2. **Charge code reporting (mentioned as a need):** D365 Landed Cost supports charge code breakdown reporting OOB. Verify the standard "Voyage costing" reports meet Technica's needs before building custom reports.

3. **Consider enabling auto-costing rules** in Landed Cost setup. This can auto-apply standard charge percentages (e.g., freight = 5% of goods value) when creating voyages, reducing manual entry.

---

### LOG002 - Direct Delivery (All FIT)

**What was proposed:**
- Two options for direct-to-customer delivery:
  - **Option 1:** Sales order (Item Requirement) → Direct Delivery flag → PO → Voyage → Receive onsite → Auto-deliver sales order → Consume to project
  - **Option 2:** Customer site as separate warehouse → PO → Voyage → Receive at site warehouse

**Assessment: Both options are valid — Option 1 is more elegant**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Direct delivery from PO to customer | Standard OOB | Sales order ↔ PO linkage |
| Voyage for direct shipments | Standard OOB | Same Landed Cost flow |
| Auto-delivery on PO receipt | Standard OOB | D365 auto-updates sales order |
| Item consumption to project | Standard OOB | Project cost posting |
| Customer site as warehouse (Option 2) | Standard OOB | Virtual warehouse approach |

**Recommendations:**

1. **Option 1 is recommended** as the primary approach because:
   - It maintains the sales order ↔ PO direct delivery link (standard D365 feature)
   - It auto-updates the sales order when the PO is received
   - It integrates cleanly with project accounting (item consumption)

2. **Option 2 (customer site as warehouse)** should be reserved for special cases where goods need to remain in Technica's inventory at the customer site (e.g., consignment stock). Creating a warehouse per customer site can lead to warehouse proliferation if there are many customers.

3. **BOM line removal for direct delivery items:** The FRD states the factory must "remove the item/part from the BOM" when direct delivering. This is a manual step that could be error-prone. Consider using **BOM line type = "Phantom"** or a separate BOM version for items that will be directly delivered, rather than manually removing lines.

---

### LOG003 - Outbound Shipments (All FIT)

**What was proposed:**
- Load creation from item requirements (project WBS)
- Route assignment on load (country/port-based)
- Transportation tenders (RFQ) for carrier rate comparison
- Load release → carrier selection
- Commercial invoice (no accounting impact) for shipping documents
- Packing slip / delivery note posting

**Assessment: Standard Transportation Management — well-designed**

| Aspect | Rating | Comment |
|--------|--------|---------|
| Load creation from item requirements | Standard OOB | Links to project WBS |
| Route planning | Standard OOB | Zip/country/port based |
| Transportation tenders | Standard OOB | Rate comparison |
| Carrier selection | Standard OOB | Best rate selection |
| Commercial invoice (proforma) | Standard OOB | Print-only, no GL impact |
| Packing slip / delivery note | Standard OOB | Standard posting |

**Recommendations:**

1. **Transportation tenders for rate comparison** is a sophisticated feature that many implementations skip. Good that Technica is using it — it provides audit trail for carrier selection decisions.

2. **Commercial invoice:** Confirm this links back to the project invoice in FRD04 (PJ003). The Project Accounting FRD mentions commercial invoices in the context of milestone reporting — ensure the same commercial invoice number is accessible from both the logistics and project accounting sides.

---

### LOG004 - Packing (1 GAP)

**What was proposed:**
- Explode BOM to view all parts for packing
- Assign box numbers/names to BOM lines
- Print packing slip showing contents per box

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| LOG004-01 | Assign/reassign box numbers to parts | This can be addressed using D365 **Container packing** feature in Warehouse Management. The standard packing station functionality allows assigning items to containers with packing slip generation. Before building custom, verify if WMS packing station meets this need. If the BOM explode approach is preferred, a simple custom field on the packing line is sufficient. |

---

### LOG000 - Setup (1 GAP)

**What was proposed:**
- Journey templates (origin → destination)
- Journey legs with lead times
- Tracking control center for lead time per leg
- Mode of delivery (Ocean, Air, Land)

| GAP ID | Description | Recommendation |
|--------|-------------|----------------|
| LOG004-04 | Calculate average lead time per leg | This is a low-priority GAP. The standard approach is to set a single lead time per leg. For averaging, a simple **Power BI calculation** on historical voyage data would provide average lead times without modifying the system. Display the average in a dashboard rather than the setup form. |

**Recommendations:**

1. **Journey templates and legs should be set up early** — they affect all delivery date calculations for inbound shipments.

2. **The Tracking Control Center is the key configuration** for accurate delivery estimates. Ensure lead times are realistic and updated periodically based on actual voyage data.

---

## Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|------|---------------|----------|
| 1 | LOG001 | Power Automate for document received notification | Low |
| 2 | LOG001 | Consider auto-costing rules for standard charge allocation | Medium |
| 3 | LOG002 | Prefer Option 1 (direct delivery) over Option 2 (site warehouse) | Medium |
| 4 | LOG002 | Use BOM version/phantom line instead of manual BOM removal | Medium |
| 5 | LOG003 | Ensure commercial invoice number links to project accounting reports | High |
| 6 | LOG004 | Verify if WMS packing station handles box assignment before customizing | Medium |
| 7 | LOG000 | Use Power BI for average lead time calculation instead of system modification | Low |

---

## Risk Flags

1. **Cross-FRD dependency with Project Accounting (FRD04):** The commercial invoice linkage between logistics and project milestone invoicing must be designed end-to-end. Both FRDs reference this but neither fully specifies the data flow.

2. **Minimal risk overall.** This is the cleanest FRD with the fewest GAPs. Landed Cost and Transportation Management are well-suited for Technica's logistics needs.

---

## Efficiency Verdict

This is the **most efficient FRD** in the set — nearly everything is standard D365 functionality with only 3 minor GAPs. The use of Landed Cost for inbound tracking and Transportation Management for outbound is the **recommended D365 pattern** for companies with international logistics.

The solution is clean, well-structured, and requires minimal customization. No major design changes recommended.

---

*This review is based on D365 F&O Landed Cost and Transportation Management best practices as of 2026.*
