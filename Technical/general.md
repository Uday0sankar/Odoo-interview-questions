# 📘 General Technical Interview Questions (Odoo)

This file collects a variety of Odoo-related technical interview questions. If your question doesn't yet have a dedicated topic file (like ORM, QWeb, Security), add it here. Contributors are welcome to move questions into topic-specific files later as the collection grows.

---

### 1. What is a Webhook?
A webhook is a user-defined HTTP callback that triggers an action in another system when an event happens. In Odoo, webhooks can be used to notify external services when something changes, like sending order updates to a third-party system.

---

### 2. Explain Odoo MVC Architecture
- **Model:** Handles business logic and database interactions (Python files).
- **View:** Defines UI components using XML (tree, form, kanban, etc.).
- **Controller:** (Optional for web apps) Used for handling HTTP routes in custom controllers.  
Odoo uses a modified MVC where controllers are minimal for backend logic.

---

### 3. Differentiate list, tuple, dictionary

| Type       | Mutable | Syntax       | Use Case                           |
| ---------- | ------- | ------------ | ---------------------------------- |
| List       | Yes     | \[1, 2, 3]   | Ordered data that can change       |
| Tuple      | No      | (1, 2, 3)    | Ordered data that shouldn't change |
| Dictionary | Yes     | {key: value} | Key-value pairs                    |

---

### 4. Difference Between Serial Number and Lot Number

- **Serial Number:** Tracks individual items uniquely (1 serial = 1 item).
- **Lot Number:** Tracks groups of the same product (1 lot = multiple items).

---

### 5. Odoo Costing Methods (Average, FIFO)

| Method  | Description                           |
| ------- | ------------------------------------- |
| Average | Cost = average price of all purchases |
| FIFO    | Cost = price of first items bought    |

---

### 6. Odoo Reordering Rules

Automatically generate purchase/manufacture orders when stock falls below a minimum level.  
You can define Min/Max quantity, quantity multiple, and lead time.

---

### 7. Explain Multi-Company and How to Manage it in Data

- Each record (partner, product, etc.) can belong to a specific company.
- Use `company_id` field to associate records.
- Enable multi-company feature in settings.

---

### 8. Record Rules vs Access Rights

| Feature       | Record Rule               | Access Rights               |
| ------------- | ------------------------- | --------------------------- |
| Purpose       | Filters records by domain | Grants permission to models |
| Based on      | Conditions (domain)       | CRUD operations             |

---

### 9. Difference Between `search` and `browse`

- `search`: Returns a recordset matching a domain query.
- `browse`: Fetches record(s) by ID(s); doesn't hit DB unless accessed.

---

### 10. SQL Constraints in Odoo

Defined using `_sql_constraints` in models.

```python
_sql_constraints = [
    ('unique_name', 'UNIQUE(name)', 'Name must be unique')
]

Certainly! Here’s your content, structured in a similar Markdown style to the previous README section, with clear titles, code blocks, tables, and bullet points where appropriate:

## 11. @api.constrains

Python constraint method for validating records.

```python
@api.constrains('field1')
def _check_field1(self):
    if self.field1 
```

## 15. What is Worker (Server Spec Related)

- Odoo uses workers to handle multiple HTTP requests.
- More workers = better concurrency.
- Needs:
  - Multiple CPUs
  - Enough RAM (approx. 1 worker ≈ 512MB)

## 16. Reasons Delivery Not Created for Sales Order

- Product type is **Service**.
- Inventory route not configured.
- No stock available.
- Rule (e.g. Reordering) not triggered.

## 17. Product Types in Odoo

- **Storable Product**
- **Consumable**
- **Service**

## 18. Create Function Return Type

```python
@api.model
def create(self, vals):
    record = super().create(vals)
    return record
```
- Returns: recordset of newly created record(s).

## 19. Function That Works While Delete (unlink)

```python
def unlink(self):
    # Custom logic
    return super().unlink()
```

## 20. Add Logic While Deleting (Customize Delete)

- Override the `unlink()` method in inherited model.

## 21. Accounts Affected When Creating Invoice from Sales Order

- **Receivable Account** (Partner)
- **Income Account** (Product Category/Product)
- **Taxes** (Tax account)
- **Stock Valuation** (for stock moves)

## 22. What is @api.model

- Used for methods that **do not depend on recordset** (no self as record).
- Example methods: `create()`, cron jobs, etc.

## 23. Onchange vs Compute Difference with Use Case

| Feature     | @api.onchange         | @api.depends (@compute)         |
|-------------|----------------------|---------------------------------|
| Trigger     | UI form change       | Any change in field value       |
| Store?      | No (by default)      | Yes (if store=True)             |
| Example     | Fill default price   | Auto calculate total amount     |

## 24. Compute Field Not Working - Reasons

- Missing `@api.depends()`
- Wrong field(s) in depends
- Not stored, but accessed in domain/search
- Logic error

## 25. What is Landed Cost

- Additional costs on products (e.g., shipping, duty, handling) are added to product cost.
- Applied using the **landed cost** feature in Odoo Inventory.

Let me know if you want to add more questions or format in a different way!
