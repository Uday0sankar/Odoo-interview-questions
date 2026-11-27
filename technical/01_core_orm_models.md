# Odoo ORM, Models & Recordsets

Technical Odoo interview questions covering model types, inheritance mechanisms, recordset methods, and CRUD lifecycle overrides.

---

### 1. Compare `Model`, `TransientModel`, and `AbstractModel`

| Class | Database Table | Persistence | Typical Use Case |
| :--- | :--- | :--- | :--- |
| `models.Model` | Yes | Permanent | Core business entities (`sale.order`, `res.partner`) |
| `models.TransientModel` | Yes | Temporary (vacuumed periodically by `ir.autovacuum`) | Wizards / pop-up dialogs (mass updates, export options) |
| `models.AbstractModel` | No | Mixin only (no table created) | Shared logic inherited by multiple models (`mail.thread`, `image.mixin`) |

---

### 2. Explain `_name`, `_inherit`, and `_inherits` (Odoo Model Inheritance)

1. **Class / Extension Inheritance (`_inherit = 'res.partner'` without `_name`):**
   Modifies an existing model in place—adds fields or overrides methods directly on the target model's table.
2. **Prototype Inheritance (`_name = 'custom.partner'` + `_inherit = 'res.partner'`):**
   Creates a **new** model and database table that copies all fields and methods from the parent model into the new model.
3. **Delegation Inheritance (`_inherits = {'res.partner': 'partner_id'}`):**
   Links a new model to an existing model via a `Many2one` field (`required=True, ondelete='cascade'`). Fields of the parent model are transparently accessible on the child without duplicating columns (classic example: `res.users` delegates to `res.partner`).

```python
class ResPartner(models.Model):
    _inherit = 'res.partner'

    loyalty_tier = fields.Selection(
        [('silver', 'Silver'), ('gold', 'Gold')],
        default='silver',
    )
```

---

### 3. What is the difference between `search()` and `browse()`?

- **`search(domain, offset=0, limit=None, order=None)`:** Queries PostgreSQL using an Odoo domain and returns a recordset of matching records (respecting access rights and record rules unless `.sudo()` is used). Use `search_count(domain)` when you only need the count.
- **`browse(ids)`:** Takes an integer ID or list of IDs and returns a recordset **without querying the database immediately**. The database is queried lazily only when a field is read; if an ID doesn't exist, accessing a field raises `MissingError` (check with `.exists()`).

```python
partners = self.env['res.partner'].search([('is_company', '=', True)], limit=10)
partner = self.env['res.partner'].browse(42).exists()
```

---

### 4. Compare `search_read()` and `read_group()` / `_read_group()`

- **`search_read(domain, fields, ...)`:** Combines `search()` and `read()` in one call, returning a list of dictionaries—ideal for RPC/API calls or fetching plain data without instantiating full recordset field caches.
- **`read_group()` / `_read_group()`:** Performs SQL `GROUP BY` and aggregate queries (`SUM`, `COUNT`, `AVG`, etc.) directly in PostgreSQL. Always prefer `read_group` / `_read_group` over fetching thousands of records and summing in Python.

```python
totals_by_partner = self.env['sale.order'].read_group(
    domain=[('state', '=', 'sale')],
    fields=['partner_id', 'amount_total:sum'],
    groupby=['partner_id'],
)
```

---

### 5. Explain `create()`, `write()`, and `unlink()` Overrides

All three are standard CRUD ORM methods that should call `super()` cleanly:

```python
from odoo import api, models
from odoo.exceptions import UserError


class ProjectTask(models.Model):
    _inherit = 'project.task'

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if not vals.get('description'):
                vals['description'] = 'Created via workflow'
        return super().create(vals_list)  # Returns recordset of created records

    def write(self, vals):
        res = super().write(vals)         # Returns True
        # Post-write logic (avoid calling self.write() inside write without guard!)
        return res

    def unlink(self):
        if any(task.stage_id.is_closed for task in self):
            raise UserError("Closed tasks cannot be deleted.")
        return super().unlink()           # Returns True
```

**Note:** Modern Odoo uses `@api.model_create_multi` (accepting `vals_list: list[dict]`) for batch creation performance instead of the deprecated single-dict `@api.model def create(self, vals)`.

---

### 6. What does `ensure_one()` do?

`self.ensure_one()` asserts that the recordset `self` contains **exactly one record**. If `len(self) != 1` (either empty or multiple records), it raises `ValueError: Expected singleton`.

Use it at the start of methods, button actions, or computed logic designed only for a single record so multi-record bugs fail fast instead of silently misbehaving.

---

### 7. Explain `filtered()`, `mapped()`, and `sorted()` on Recordsets

These high-level recordset helpers operate in memory (after prefetching) and avoid clunky loops:

- **`filtered(func_or_field)`:** Returns a subset recordset matching a predicate or truthy field.
- **`mapped(path_or_func)`:** Extracts a list of scalar values or a deduplicated recordset of related records across a dotted path.
- **`sorted(key=..., reverse=False)`:** Returns a reordered recordset.

```python
confirmed_orders = orders.filtered(lambda o: o.state == 'sale' and o.amount_total > 0)
customers = confirmed_orders.mapped('partner_id')          # res.partner recordset
amounts = confirmed_orders.mapped('amount_total')          # list[float]
by_date = confirmed_orders.sorted(key='date_order', reverse=True)
```

You can also combine recordsets with set operators: `a | b` (union), `a & b` (intersection), `a - b` (difference).

---

### 8. How do `sudo()`, `with_context()`, and `with_company()` work?

None of these mutate `self`; they return a **new recordset** bound to a modified `Environment`:

- **`records.sudo(flag=True)`:** Bypasses `ir.model.access` and `ir.rule` security checks (executes as superuser). Always restrict `sudo()` to the smallest necessary operation after validating business permissions yourself.
- **`records.with_context(**kwargs)`:** Adds or overrides keys in `self.env.context` (e.g., `lang='fr_FR'`, `active_test=False` to include archived records, `tracking_disable=True`).
- **`records.with_company(company)`:** Runs operations in the context of a specific `res.company` (affecting `company_dependent` fields and default company values).
