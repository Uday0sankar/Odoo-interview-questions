# Odoo Views, QWeb, Reports & Controllers

Technical Odoo interview questions covering XML views, XPath inheritance, domains/context, QWeb PDF reports, Excel exports, and HTTP controllers.

---

### 1. Explain the main Odoo XML View Types and Actions

- **`form`:** Detailed view for creating/editing a single record (`<sheet>`, `<header>`, `<group>`, `<notebook>`).
- **`list` (formerly `tree` in Odoo < 18):** Tabular view supporting `editable="bottom|top"`, `decoration-*`, and `sum`.
- **`search`:** Controls search bar fields, `<filter>` presets, and `<group expand="0">` group-by options.
- **`kanban`:** Card-based QWeb layout (`t-name="card"` / `kanban-box`), commonly paired with stage columns.
- **`ir.actions.act_window`:** Binds a model, `view_mode`, optional `domain`, and `context` to a menu item or button.

---

### 2. How does XPath View Inheritance work?

To customize an existing view without modifying core files, declare a view with `inherit_id` referencing the parent XML ID and use `<xpath>` (or field shorthand):

```xml
<record id="view_order_form_custom" model="ir.ui.view">
    <field name="name">sale.order.form.custom</field>
    <field name="model">sale.order</field>
    <field name="inherit_id" ref="sale.view_order_form"/>
    <field name="arch" type="xml">
        <!-- Insert after partner_id -->
        <xpath expr="//field[@name='partner_id']" position="after">
            <field name="customer_ref"/>
        </xpath>

        <!-- Modify an attribute on an existing element -->
        <xpath expr="//button[@name='action_confirm']" position="attributes">
            <attribute name="string">Approve &amp; Confirm</attribute>
        </xpath>
    </field>
</record>
```

Common `position` values: `inside`, `after`, `before`, `replace`, `attributes`, and `move`. Always target unique attributes (`name`, `id`, or unambiguous nesting) rather than brittle positional indexes like `div[3]`.

---

### 3. How do Domains and Context work in Views and Actions?

- **Domain:** A prefix-notation list of tuples filtering records, e.g. `['&', ('state', '=', 'sale'), '|', ('user_id', '=', uid), ('team_id', '!=', False)]`.
- **Context:** A dictionary passed through actions, fields, and `env` to carry defaults (`default_partner_id`), default search filters (`search_default_my_orders`: 1), language (`lang`), or flags (`active_test`: False).

```xml
<field name="partner_id"
       domain="[('is_company', '=', True), ('country_id', '=', country_id)]"
       context="{'default_is_company': True}"/>
```

---

### 4. How do QWeb PDF Reports work in Odoo?

Odoo uses **QWeb** (XML templating) compiled to HTML and rendered to PDF via `wkhtmltopdf`:

1. Declare an `ir.actions.report` record (`report_type="qweb-pdf"`, `report_name="my_module.report_invoice_doc"`).
2. Define the QWeb `<template>` using directives:
   - `t-foreach="docs" t-as="o"`: Iterate over the selected recordset.
   - `t-call="web.external_layout"`: Reuse standard company header/footer.
   - `t-field="o.amount_total"`: Render formatted field values (currency, dates, translations).
   - `t-out="val"` (replaces older `t-esc` / `t-raw`): Output computed Python expressions safely.
   - `t-if` / `t-elif` / `t-else` and `t-set="var" t-value="..."`.
3. Optional **AbstractModel** (`report.<module>.<template_name>`) implementing `_get_report_values(self, docids, data=None)` lets you inject custom Python calculations into the template context.

---

### 5. How do you generate Excel (`.xlsx`) Reports in Odoo?

Odoo Core does not ship a built-in `qweb-xlsx` report engine out of the box (though Enterprise/OCA modules add it). Common interview-ready approaches:

1. **Wizard / Action + HTTP Controller (Core-only approach):**
   - A `TransientModel` wizard collects filter parameters and returns an `ir.actions.act_url` pointing to `/my_module/export_xlsx?wizard_id=...`.
   - The `@http.route` controller uses `xlsxwriter` (bundled with Odoo) writing into an in-memory `io.BytesIO()` buffer and returns `request.make_response(...)` with `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` and `Content-Disposition: attachment; filename="report.xlsx"`.
2. **OCA `report_xlsx` module:** Inherit `report.report_xlsx.abstract` and implement `generate_xlsx_report(self, workbook, data, records)`.

---

### 6. How do HTTP Controllers work in Odoo?

Controllers inherit from `odoo.http.Controller` and expose web or API routes via `@http.route`:

```python
from odoo import http
from odoo.http import request


class OrderStatusController(http.Controller):
    @http.route('/api/v1/orders/<int:order_id>', type='json', auth='user', methods=['POST'], csrf=False)
    def get_order_status(self, order_id):
        order = request.env['sale.order'].browse(order_id).exists()
        if not order:
            return {'error': 'Not found'}
        return {'name': order.name, 'state': order.state}
```

- **`type='http'`:** Standard HTTP responses (HTML pages, file downloads, REST endpoints parsing `request.httprequest.data`).
- **`type='json'`:** JSON-RPC 2.0 endpoint (`{"jsonrpc": "2.0", "params": {...}}`).
- **`auth`:** `'user'` (authenticated session), `'public'` (session or public user), `'none'` (no DB user/session—used for DB-agnostic routes).
