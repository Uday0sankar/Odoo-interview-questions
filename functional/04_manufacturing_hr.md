# Odoo Manufacturing (MRP) & HR Functional Questions

Functional interview questions covering Bills of Materials, Manufacturing Orders, Work Centers, Subcontracting, and core HR/Payroll concepts.

---

### 1. What are the Bill of Materials (BoM) types in Odoo Manufacturing?

1. **Manufacture this product:** Standard production BoM—confirming a Manufacturing Order (MO) consumes the listed components and produces the finished product.
2. **Kit:** Pseudo-BoM used for bundles/kits sold together. No Manufacturing Order is created; instead, confirming a Sales Order automatically explodes the kit into its **individual components** on the Delivery Order.
3. **Subcontracting:** Used when outsourcing assembly to a vendor. Ordering the finished product from the subcontracting vendor triggers component resupply/consumption tracking at the subcontractor's virtual location.

---

### 2. Explain the Manufacturing Order (MO) Lifecycle

1. **Draft / Confirmed:** Reserves raw components from stock (or triggers MTO/reordering if unavailable).
2. **Work Orders (Optional Routing):** Breaks production into step-by-step operations (*Cutting*, *Assembly*, *Quality Check*) across **Work Centers**, tracking expected vs. real duration and Overall Equipment Effectiveness (OEE).
3. **Component Consumption & By-Products:** Records actual raw materials used and any secondary outputs (**By-Products**) produced alongside the main item.
4. **Mark as Done:** Moves components out of stock, moves finished goods into stock, and updates finished product valuation/cost.

---

### 3. How does Subcontracting work in Odoo?

1. Enable **Subcontracting** in Manufacturing Settings and create a BoM of type *Subcontracting* naming the subcontractor(s).
2. Confirm a **Purchase Order** to the subcontractor for the finished product.
3. Either send raw components to the **Subcontracting Location** via a resupply transfer (or dropship components directly from another supplier to the subcontractor).
4. When you receive the finished product on the PO Receipt, Odoo automatically consumes the components from the subcontractor's location and includes both component cost + subcontracting service fee in the finished product's valuation.

---

### 4. How does Odoo handle Scrap Orders in Manufacturing and Inventory?

A **Scrap Order** removes damaged, defective, or wasted components or finished goods from usable stock into a virtual **Scrap Location** (`Scrap`).
- Can be triggered standalone from Inventory or directly inside an active **Manufacturing Order / Work Order** (e.g., if a component breaks during assembly, scrapping it deducts it and lets you replenish a replacement).
- Under automated valuation, scrapping posts an accounting entry crediting Stock Valuation and debiting the configured scrap/loss expense account.

---

### 5. In Odoo HR, what is the difference between an Employee (`hr.employee`) and a User (`res.users`)?

- **User (`res.users`):** Represents a **system login account** with credentials, access rights, and license consumption.
- **Employee (`hr.employee`):** Represents a **person on payroll/HR** (department, job position, manager, work address, badge ID, bank account, contracts).
- **Relationship:** An employee *can* be linked to a user (optional), but doesn't have to be—for example, factory floor workers or contractors tracked in HR/Attendances without needing an Odoo backend login.

---

### 6. Summarize Core HR Modules: Time Off, Contracts, Expenses & Payroll

- **Contracts:** Track an employee's start/end dates, working schedule, wage, and salary structure type (must be in *Running* state for payroll calculation).
- **Time Off (Leaves):** Supports accrual plans, leave types (*Paid Time Off*, *Sick Leave*), multi-level manager/HR approvals, and public holidays—integrating directly with Project timesheets and Payroll.
- **Expenses:** Employees submit receipts (manual or OCR/email), managers approve, and Accounting posts/reimburses the employee (or bills back to a customer Sales Order).
- **Payroll (Enterprise / OCA):** Uses **Salary Structures** and Python/code **Salary Rules** (Basic, Allowances, Deductions, Net) grouped into **Payslip Batches** to generate payslips and post payroll journal entries.
