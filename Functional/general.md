Absolutely! Here’s your content, formatted in a clean, professional Markdown style suitable for a README or documentation file, and matching your previous structure. I’ve organized each question and answer with clear numbering, tables, and bullet points as appropriate.

# 📘 Functional Interview Questions (Odoo)

This file contains commonly asked functional-level interview questions about Odoo, ideal for business analysts, consultants, and functional users.  
**Technical questions (Python, API, ORM, etc.) are excluded.**

## 2. Difference Between Serial Number and Lot Number

- **Serial Number**: Tracks individual items uniquely (one serial = one item).
- **Lot Number**: Tracks groups of the same product (one lot = multiple items).

## 3. Odoo Costing Methods (Average, FIFO)

| Method  | Description                         |
|---------|-------------------------------------|
| Average | Cost = average price of all purchases |
| FIFO    | Cost = price of first items bought    |

## 4. Odoo Reordering Rules

Odoo can automatically generate purchase or manufacturing orders when stock drops below a minimum level.

You can define:
- **Min/Max Quantity**
- **Quantity Multiple**
- **Lead Time**

## 5. Explain Multi-Company and How to Manage it in Data

- Each record (partner, product, etc.) can be linked to a specific company using `company_id`.
- Multi-company mode must be enabled in Settings.
- Users can switch companies; company-specific security and access rules apply.

## 6. Record Rules vs Access Rights

| Feature      | Record Rule                         | Access Rights                       |
|--------------|-------------------------------------|-------------------------------------|
| Purpose      | Filters records by domain           | Grants permission to models         |
| Based on     | Conditions (domain)                 | CRUD operations                     |

## 7. Reasons Delivery Not Created for Sales Order

- Product type is **Service** (doesn't require delivery)
- No inventory route configured
- Stock is unavailable
- Reordering rule not triggered

## 8. Product Types in Odoo

- **Storable Product**: Tracked for inventory and stock valuation.
- **Consumable**: Used but not tracked per unit.
- **Service**: Non-physical items such as installation or consultancy.

## 9. Accounts Affected When Creating Invoice from Sales Order

- **Receivable Account** (from partner)
- **Income Account** (from product or product category)
- **Tax Account** (if taxes are applied)
- **Stock Valuation Account** (if products are delivered and tracked)

## 10. What is Landed Cost

- Landed cost adds extra charges (shipping, duties, freight, insurance, etc.) to the cost of purchased products.
- Odoo allows you to allocate these costs across products via the **Landed Cost** feature in Inventory.

Let me know if you want further formatting, more functional questions, or anything else! 
