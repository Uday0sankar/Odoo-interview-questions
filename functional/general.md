# General Functional Interview Questions (Odoo)

Cross-functional Odoo interview questions for business analysts, functional consultants, and implementation leads covering security concepts, users, chatter, developer mode, and system-wide configuration.

> **Specialized Functional Topics:**
> - Sales & CRM: [01_sales_crm.md](./01_sales_crm.md)
> - Purchase & Inventory: [02_purchase_inventory.md](./02_purchase_inventory.md)
> - Accounting & Multi-Company: [03_accounting_multicompany.md](./03_accounting_multicompany.md)
> - Manufacturing (MRP) & HR: [04_manufacturing_hr.md](./04_manufacturing_hr.md)
> - Technical / Developer Questions: [technical/general.md](../technical/general.md)

---

### 1. Differentiate Record Rules vs. Access Rights from a Functional Perspective

| Feature | Access Rights | Record Rules |
| :--- | :--- | :--- |
| **Purpose** | Controls whether a user group can **Read, Create, Edit, or Delete** a model/menu at all | Filters **which specific records** in that model a user can access |
| **Example** | A *Salesperson* can create and edit Sales Orders, but cannot delete them | A *Salesperson* can only see their **own** Sales Orders, while a *Sales Manager* sees all |
| **Configuration** | *Settings -> Technical -> Security -> Access Rights* | *Settings -> Technical -> Security -> Record Rules* |

---

### 2. What are the User Types in Odoo?

1. **Internal User:** Employees/staff who access backend apps (Sales, Inventory, Accounting, etc.). Licensed per active internal user in Odoo Enterprise.
2. **Portal User:** External customers, vendors, or candidates who log into the `/my` web portal to view quotations, sales orders, invoices, tickets, or shipments—without backend access.
3. **Public User:** Unauthenticated website visitors browsing eCommerce pages, blogs, or contact forms.

---

### 3. Explain the Chatter, Followers, and Scheduled Activities

Available on core business documents (Sales Orders, Invoices, Tasks, Employees):
- **Log Note:** Internal-only communication visible to backend employees; does **not** email external customers.
- **Send Message:** External communication emailed to selected **Followers** (customers, vendors, team members).
- **Followers:** Users/partners subscribed to a record to receive notifications on specific subtypes (e.g., *Order Confirmed*, *Stage Changed*).
- **Activities:** Actionable, deadline-driven to-dos (*Call*, *Email*, *Meeting*, *To-Do*) assigned to specific users with visual indicators (green/yellow/red) across list, kanban, and systray views.

---

### 4. What is Developer Mode (Debug Mode), and when do Functional Consultants use it?

Activated via *Settings -> Activate the developer mode* (or `?debug=1` in the URL). It unlocks:
- **Technical Menu** (*Models*, *Views*, *Automated Actions*, *Scheduled Actions*, *Sequences*, *Record Rules*).
- **Field Inspection:** Hovering over any field shows its technical name, model, field type, and relations.
- **View Metadata & Edit View:** Inspecting `XML ID`, `Created By/On`, and viewing inherited XPath structure.
- **Debug with Assets (`?debug=assets`):** Disables asset bundling for troubleshooting frontend/JS/CSS issues.

---

### 5. What is a Fiscal Year / Sequence / Company Setup Checklist for a New Implementation?

When onboarding a new Odoo database, functional consultants typically configure:
1. **Company Details & Logo:** Address, tax ID, base currency, and document layout.
2. **Chart of Accounts & Fiscal Localization:** Country-specific taxes, fiscal positions, and default accounts.
3. **Users, Access Groups & Multi-Company:** Internal users, role permissions, and company visibility.
4. **Product Categories, UoMs & Costing Methods:** Inventory valuation (Manual vs Automated) and costing (Standard, AVCO, FIFO).
5. **Warehouses, Locations & Routes:** Multi-step receipts/deliveries, dropshipping, or MTO if needed.
6. **Document Sequences & Email Gateways:** Outgoing SMTP / incoming mail servers and invoice/order numbering.
