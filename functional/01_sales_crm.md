# Odoo Sales & CRM Functional Questions

Functional interview questions covering the Sales Order lifecycle, invoicing policies, delivery creation, pricelists, Units of Measure, and CRM pipelines.

---

### 1. Walk through the standard Sales Order Lifecycle in Odoo

1. **Quotation (`draft`):** Created for a customer with products, quantities, prices, and payment terms.
2. **Quotation Sent (`sent`):** Emailed to the customer (with optional online signature/payment via the customer portal).
3. **Sales Order (`sale`):** Customer confirms (or salesperson clicks *Confirm*). This locks in the commitment and triggers downstream operations:
   - Generates a **Delivery Order** (if storable/consumable goods are included).
   - Generates a **Project/Task** (if a configured Service product creates tasks).
4. **Invoicing:** A customer invoice is created based on the product's Invoicing Policy.
5. **Locked / Done or Cancelled (`cancel`):** Completed orders can be locked to prevent unintended edits.

---

### 2. Why might a Delivery Order NOT be created when confirming a Sales Order?

Common functional causes include:
- **Product Type is Service:** Service products are non-physical and do not generate warehouse stock transfers.
- **No Inventory Module / Route Misconfiguration:** The Inventory app isn't installed, or the product/warehouse route configuration has no valid outgoing rule.
- **No Stock / Procurement Rule Not Triggered:** Stock isn't available under strict reservation/MTO rules, or a required replenishment/reordering rule wasn't triggered.
- **Order is still a Quotation:** Deliveries are only generated once a Quotation is **confirmed** into a Sales Order.
- **Zero / Negative Quantities or Dropship Setup:** Note that if **Dropship** is enabled on the order line, Odoo generates a **RFQ/PO** to the vendor and a direct vendor-to-customer dropship transfer rather than a standard warehouse delivery.

---

### 3. Compare Invoicing Policies: Ordered Quantities vs. Delivered Quantities

Configured on the Product template (*Sales -> Invoicing Policy*):
- **Ordered Quantities:** You can invoice the full Sales Order immediately after confirmation—even before goods are shipped (common for upfront billing, retail, or advance services).
- **Delivered Quantities:** You can only invoice the quantity that has actually been **delivered/validated** in the warehouse (or logged on timesheets/milestones for services). Prevents overbilling when partial deliveries happen.

---

### 4. How do Pricelists and Discounts work in Odoo Sales?

Pricelists dynamically calculate unit prices based on **customer, currency, quantity, and date**:
- **Multiple Prices per Product:** Different price tiers based on minimum quantity (`Min. Qty`) and validity dates.
- **Advanced Price Rules:** Compute prices using a **Fixed Price**, **Percentage Discount**, or **Formula** (e.g., *Public Price - 10%*, or *Cost + 25% markup* with rounding).
- **Line Discounts (`Discount %`):** Enabled in *Sales Settings* to show explicit discount percentages on quotation/invoice lines (as opposed to baking the discount directly into the unit price).

---

### 5. Explain Units of Measure (UoM) and UoM Categories

Odoo allows purchasing, stocking, and selling the same product in different units as long as they belong to the **same UoM Category** (e.g., *Unit*, *Weight*, *Volume*):
- **Reference Unit:** Every category has one base reference unit (e.g., `Units` or `kg`).
- **Conversion Ratio:** Other units in the category are defined as multiples or fractions of the reference unit (e.g., `Pack of 12` = 12 × `Units`).
- **Business Example:** You buy cables in `Rolls (100m)`, store them in `Meters`, and sell them in `Meters` or `Packs of 10m`—Odoo automatically converts stock quantities behind the scenes.

---

### 6. In CRM, what is the difference between a Lead and an Opportunity?

- **Lead:** An unqualified raw contact or inquiry (e.g., from a website form, trade show, or email alias) before you know if there is a real deal. Managed in *CRM -> Leads* (must be enabled in CRM Settings).
- **Opportunity:** A qualified sales prospect with estimated **Expected Revenue**, **Closing Date**, **Customer**, and a **Pipeline Stage** (e.g., *New -> Qualified -> Proposition -> Won*).

---

### 7. How do Pipeline Stages, Probability, and Lost Reasons work in CRM?

- **Stages:** Represent steps in the sales funnel. Each stage can be team-specific and has a default win probability or `Won` flag.
- **Expected Revenue vs. Prorated Revenue:** `Prorated Revenue = Expected Revenue × (Probability / 100)`. Odoo can auto-predict win probabilities using historical won/lost data (predictive lead scoring).
- **Lost Reasons:** When marking an opportunity **Lost**, capturing structured Lost Reasons (*Too Expensive*, *No Stock*, *Lost to Competitor*) enables management reporting on why deals fail.
