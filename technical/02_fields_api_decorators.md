# Odoo Fields, Constraints & API Decorators

Technical Odoo interview questions covering relational fields, `Command` syntax, `@api.depends`, `@api.onchange`, constraints, and `@api.model`.

---

### 1. Explain Relational Fields and `Command` Syntax (`One2many` / `Many2many`)

- **`Many2one`:** Foreign key pointing to a single record on another model (`partner_id = fields.Many2one('res.partner', ondelete='restrict')`).
- **`One2many`:** Virtual reverse relation of a `Many2one` (`line_ids = fields.One2many('sale.order.line', 'order_id')`).
- **`Many2many`:** Association table linking multiple records on both sides (`tag_ids = fields.Many2many('crm.tag')`).

When writing to `One2many` or `Many2many` fields in `create()`/`write()`, use `odoo.fields.Command` (or legacy tuple codes):

| `Command` Helper | Tuple | Action |
| :--- | :--- | :--- |
| `Command.create(vals)` | `(0, 0, vals)` | Create & link new record |
| `Command.update(id, vals)` | `(1, id, vals)` | Update linked record |
| `Command.delete(id)` | `(2, id, 0)` | Unlink & delete record from DB |
| `Command.unlink(id)` | `(3, id, 0)` | Unlink only (keep record in DB) |
| `Command.link(id)` | `(4, id, 0)` | Link existing record |
| `Command.clear()` | `(5, 0, 0)` | Unlink all records |
| `Command.set(ids)` | `(6, 0, ids)` | Replace links with `ids` |

```python
from odoo import Command

order.write({
    'order_line': [
        Command.create({'product_id': product.id, 'product_uom_qty': 2}),
    ],
    'tag_ids': [Command.set([tag1.id, tag2.id])],
})
```

---

### 2. Compare `@api.onchange` and `@api.depends` (Computed Fields)

| Feature | `@api.onchange('field')` | `@api.depends('field')` (`compute=...`) |
| :--- | :--- | :--- |
| **When it runs** | Only in the **UI form view** when a user edits a watched field | Whenever dependent fields change (UI, Python `create`/`write`, imports, RPC) |
| **Persistence** | Not stored by itself (updates form buffer values) | Stored if `store=True`; computed on-the-fly if `store=False` |
| **Record state** | Runs on unsaved in-memory `NewId` records too | Works on both saved and unsaved records |
| **Extra UI effects** | Can return `{'warning': ...}` (domain return is deprecated in modern Odoo; use dynamic domains on fields) | Purely computes field values |
| **Use Case** | Auto-fill default notes or payment terms on customer selection | Calculate `amount_total`, `margin`, or status fields reliably everywhere |

---

### 3. Why might a Computed Field fail to work or update?

Common pitfalls in Odoo interviews and production debugging:

1. **Missing or incomplete `@api.depends(...)`:** Without `@api.depends`, a stored computed field only computes once and never recomputes when dependencies change, and a non-stored field won't react to form changes.
2. **Wrong dotted dependency path:** Using `@api.depends('line_ids')` instead of `@api.depends('line_ids.price_subtotal')` won't retrigger when a line's price changes.
3. **Not looping over `self`:** Batch operations pass multiple records in `self`; forgetting `for record in self:` raises `ValueError: Expected singleton`.
4. **Failing to assign a value on all branches:** Every computed field **must** be assigned a value for every record in `self` (even `False` or `0.0`), otherwise Odoo raises `ValueError: Compute method failed to assign ...`.
5. **Searching/grouping a non-stored computed field:** A field with `store=False` cannot be used in `search()` domains or `groupby` unless you also define a `search='_search_my_field'` method.

```python
total_qty = fields.Float(compute='_compute_total_qty', store=True)

@api.depends('line_ids.product_uom_qty')
def _compute_total_qty(self):
    for rec in self:
        rec.total_qty = sum(rec.line_ids.mapped('product_uom_qty'))
```

---

### 4. Compare `_sql_constraints` and `@api.constrains`

- **`_sql_constraints`:** Enforced directly by **PostgreSQL** (`UNIQUE`, `CHECK`, `EXCLUDE`). Extremely fast, safe against concurrent transactions, and catches direct SQL/imports—but limited to row-level SQL expressions.
- **`@api.constrains`:** Python validation run by the ORM after `create()`/`write()` when the listed fields change. Ideal for complex cross-record, relational, or business-rule validations.

```python
from odoo import api, fields, models
from odoo.exceptions import ValidationError


class FleetVehicle(models.Model):
    _name = 'fleet.vehicle.custom'
    _description = 'Custom Vehicle'

    name = fields.Char(required=True)
    license_plate = fields.Char(required=True)
    odometer = fields.Float(default=0.0)

    _sql_constraints = [
        ('unique_license_plate', 'UNIQUE(license_plate)', 'License plate must be unique.'),
        ('positive_odometer', 'CHECK(odometer >= 0)', 'Odometer cannot be negative.'),
    ]

    @api.constrains('odometer')
    def _check_odometer_limit(self):
        for record in self:
            if record.odometer > 1_000_000:
                raise ValidationError("Odometer reading exceeds the maximum allowed limit (1,000,000).")
```

*(Note: In Odoo 19+, `models.Constraint` declarations are also introduced alongside `_sql_constraints`.)*

---

### 5. What is `@api.model`?

`@api.model` marks a method that operates at the **model level** rather than on specific records in `self` (`self` is an empty recordset carrying the environment `self.env`).

- **Use cases:** Scheduled cron entry points (`ir.cron`), RPC utility methods, custom search helpers, or factory methods that don't depend on existing record IDs.
- **Note:** For `create()`, modern Odoo uses `@api.model_create_multi` rather than `@api.model` so batch record creation runs in one call.

```python
@api.model
def cron_archive_stale_drafts(self):
    stale = self.search([('state', '=', 'draft')])
    stale.write({'active': False})
```
