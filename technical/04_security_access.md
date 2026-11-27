# Odoo Security, Access Rights & Multi-Company Rules

Technical Odoo interview questions covering `ir.module.category`, `res.groups`, `ir.model.access.csv`, `ir.rule` record rules, `sudo()`, and multi-company security.

---

### 1. What are the 4 Layers of Security in Odoo?

1. **Security Categories & Groups (`ir.module.category`, `res.groups`):** Roles assigned to users (e.g., *User < Manager < Administrator*), built via `implied_ids` hierarchies.
2. **Access Rights (`ir.model.access.csv`):** Model-level CRUD (`perm_read`, `perm_write`, `perm_create`, `perm_unlink`) granted per group.
3. **Record Rules (`ir.rule`):** Row-level domain filters restricting *which specific records* inside a model a user can read, write, create, or delete.
4. **Field / View / Menu Restrictions (`groups="..."`):** UI-level visibility controls (note: hiding a field in XML alone does **not** prevent RPC access—use `groups="..."` on the Python `fields.*` definition for server-side field security).

```xml
<record id="module_category_fleet_custom" model="ir.module.category">
    <field name="name">Custom Fleet</field>
    <field name="sequence">10</field>
</record>

<record id="group_fleet_user" model="res.groups">
    <field name="name">User</field>
    <field name="category_id" ref="module_category_fleet_custom"/>
</record>

<record id="group_fleet_manager" model="res.groups">
    <field name="name">Manager</field>
    <field name="category_id" ref="module_category_fleet_custom"/>
    <field name="implied_ids" eval="[(4, ref('group_fleet_user'))]"/>
</record>
```

---

### 2. Explain `ir.model.access.csv` vs. `ir.rule`

| Aspect | Access Rights (`ir.model.access.csv`) | Record Rules (`ir.rule`) |
| :--- | :--- | :--- |
| **Granularity** | Entire model (table level) | Individual records (row level) |
| **Definition** | CSV file in `security/` | XML record with `domain_force` |
| **Evaluation** | **OR** across user's groups (if any group grants `read`, model read is allowed) | **Global rules** (`groups` empty) are **AND**ed; **group rules** are **OR**ed together, then **AND**ed with global rules |
| **Default when absent** | Deny all (except superuser) | Allow all (within Access Rights) |

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_fleet_user,fleet.custom.user,model_fleet_vehicle_custom,group_fleet_user,1,0,0,0
access_fleet_manager,fleet.custom.manager,model_fleet_vehicle_custom,group_fleet_manager,1,1,1,1
```

---

### 3. How do Global Record Rules differ from Group Record Rules, and what are common `domain_force` patterns?

- **Global Rule (`groups` field not set):** Applies to **every** internal/portal user (except superuser) and is combined with **AND**. Even if a manager group has a permissive rule, a restrictive global rule still applies—this is how Odoo enforces **multi-company isolation**.
- **Group Rule (`groups` field set):** Applies only to users belonging to that group. Multiple group rules for the same model are combined with **OR** (permissive union), so a user who is both *Sales User (own documents)* and *Sales Manager (all documents)* sees all documents.

Common `domain_force` patterns in `ir.rule`:

| Scenario | `domain_force` Pattern |
| :--- | :--- |
| **Multi-company** | `['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]` |
| **Owner only** | `[('user_id', '=', user.id)]` |
| **Owner or manager** | `['|', ('user_id', '=', user.id), ('manager_id', '=', user.id)]` |
| **Department-based** | `[('department_id', '=', user.department_id.id)]` |
| **State-based** | `[('state', 'in', ['draft', 'confirmed'])]` |
| **Date-based** | `[('date', '>=', context_today().strftime('%Y-%m-%d'))]` |

---

### 4. How is Multi-Company Isolation enforced technically?

1. Add a `company_id = fields.Many2one('res.company', default=lambda self: self.env.company)` field on the model.
2. Add a global `ir.rule` using `company_ids` (the companies currently selected/allowed in the user's session):
   `['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]`
3. Set `_check_company_auto = True` on the model and `check_company=True` on relational fields (`partner_id = fields.Many2one(..., check_company=True)`) so Odoo automatically prevents linking records from mismatched companies.
4. For fields whose values differ per company on a shared record (e.g., a product's income account or cost), use `company_dependent=True`.

---

### 5. What are the Security Risks of `sudo()` and Raw SQL?

- **`sudo()`** bypasses both `ir.model.access` and `ir.rule` (including multi-company rules!). Only use `sudo()` when a standard user legitimately triggers a system side effect (e.g., incrementing a sequence or logging an audit entry), and validate inputs/permissions *before* calling `.sudo()`.
- **Raw SQL (`self.env.cr.execute(...)`)** bypasses the entire ORM, security checks, and record rules. Never interpolate user input via f-strings or `%`; always pass query parameters as a tuple/list (`cr.execute("SELECT id FROM res_partner WHERE name = %s", (name,))`) to prevent SQL injection.
