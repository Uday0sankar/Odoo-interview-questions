# Odoo Purchase & Inventory Functional Questions

Functional interview questions covering procurement, 2-way/3-way matching, product types, tracking (serial/lot), costing methods, reordering rules, landed costs, and inventory adjustments.

---

### 1. Explain the standard Purchase Flow and 2-Way vs. 3-Way Matching

**Flow:** **RFQ (Request for Quotation)** -> **Purchase Order (PO)** -> **Incoming Receipt (`stock.picking`)** -> **Vendor Bill**.

- **Control Policy:**
  - *On Ordered Quantities:* Bill can be validated as soon as the PO is confirmed (**2-way match**: PO vs. Vendor Bill).
  - *On Received Quantities:* Bill expects goods to be received first (**3-way match**: PO vs. Receipt vs. Vendor Bill).
- **3-Way Matching Status:** When enabled in Purchase Settings, Odoo shows *Should Be Paid: Yes / No / Exception* on the vendor bill by comparing billed quantities against received quantities.

---

### 2. What are the Product Types in Odoo?

- **Storable Product** *(in Odoo 18+, **Goods** with **Track Inventory** enabled)*: Physical items whose on-hand quantities, locations, routes, and stock valuation are tracked by the Inventory app.
- **Consumable** *(in Odoo 18+, **Goods** with **Track Inventory** disabled)*: Physical items you buy/deliver (generating pickings) without maintaining exact on-hand stock counts or inventory valuation (e.g., office stationery, packaging tape).
- **Service:** Non-physical offerings (consulting, installation, shipping fees) that do not create warehouse transfers.

---

### 3. Differentiate Serial Numbers and Lot Numbers

- **Serial Number:** Tracks **individual units uniquely** (1 serial number = 1 unit). Essential for electronics, vehicles, or warranty/repair tracking.
- **Lot Number:** Tracks **batches/groups** of the same product (1 lot number = many units). Essential for food, pharmaceuticals, chemicals, or expiration-date management (FEFO removal strategy).

---

### 4. Compare Odoo Costing Methods: Standard, AVCO, and FIFO

Configured on the **Product Category**:

| Method | How Cost Is Calculated | Best Use Case |
| :--- | :--- | :--- |
| **Standard Price** | Fixed manually on the product; purchase price differences go to a price variance account | Stable manufacturing costs or fixed-budget goods |
| **Average Cost (AVCO)** | Weighted average of existing stock value + incoming receipt cost | Commodities, homogeneous retail goods |
| **FIFO (First In, First Out)** | Cost of the oldest remaining stock layer is used when goods are delivered | Goods with fluctuating supplier prices or perishable/batch items |

---

### 5. Explain Automated vs. Manual Inventory Valuation

- **Manual (Periodic) Valuation:** Accounting entries are **not** posted automatically on warehouse receipts or deliveries; inventory value on the balance sheet is updated manually via periodic journal entries.
- **Automated (Perpetual) Valuation:** Every stock move into or out of a valuation location automatically posts an accounting journal entry using **Stock Valuation**, **Stock Input (Interim Received)**, and **Stock Output (Interim Delivered)** accounts.

---

### 6. How do Reordering Rules, MTO, and Dropshipping work?

- **Reordering Rules (Min/Max):** Automatically generate an RFQ or Manufacturing Order when forecasted stock drops below the **Min Quantity**, replenishing up to the **Max Quantity** (respecting **Multiple Quantity** and vendor/manufacturing **Lead Times**).
- **Make to Order (MTO):** Instead of pulling from existing warehouse stock, confirming a Sales Order immediately triggers a linked Purchase Order or Manufacturing Order specifically for that SO.
- **Dropshipping:** Ships directly from the **Vendor -> Customer** without ever entering your warehouse (SO confirmation creates a PO with the customer's shipping address).

---

### 7. What is a Landed Cost, and how is it applied?

**Landed Cost** adds extra acquisition expenses (international freight, customs duties, port handling, insurance) onto the inventory valuation of purchased products so your COGS and unit cost reflect the true landed value.

**How it works in Odoo:**
1. Validate the incoming **Receipt** (and ensure the Product Category uses **FIFO** or **AVCO** with **Automated Valuation**).
2. Create a **Vendor Bill** for the freight/customs service product (marked *Is a Landed Cost*).
3. Create and validate a **Landed Cost** record linked to the receipt(s), choosing a **Split Method**: *Equal*, *By Quantity*, *By Current Cost*, *By Weight*, or *By Volume*.

---

### 8. What is an Inventory Adjustment, and when would you use it?

An **Inventory Adjustment** reconciles physical warehouse counts with Odoo's system quantities. Common use cases include:
- **Physical Stocktake:** Periodic full warehouse counts to correct discrepancies.
- **Cycle Counting:** Scheduled partial counts by location or high-value product category without halting operations.
- **Shrinkage & Damaged Goods:** Writing off stolen, expired, broken, or lost stock into an Inventory Loss virtual location.
- **Inventory Audits:** Verifying system accuracy before financial period close.

When you apply an Inventory Adjustment in Odoo, the system creates compensating **Stock Moves** between the physical location and the virtual *Inventory Adjustment* location so both stock quantities and accounting valuation stay accurate.

---

### 9. Differentiate Stock Picking, Stock Move, Stock Move Line, and Stock Quant

- **Stock Picking (`stock.picking`):** The overall warehouse transfer document (Receipt, Internal Transfer, Delivery Order).
- **Stock Move (`stock.move`):** The planned movement of a specific product and demanded quantity within a picking.
- **Stock Move Line (`stock.move.line`):** Detailed execution lines specifying exact source/destination sub-locations and Lot/Serial numbers.
- **Stock Quant (`stock.quant`):** Real-time snapshot of how much of a product (and lot/serial) is physically on hand and reserved in a specific location.
