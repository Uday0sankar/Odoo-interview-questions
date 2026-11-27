# Odoo Views, Widgets, QWeb, Reports & Controllers

Technical Odoo interview questions covering XML views, field/view attributes, widgets, XPath inheritance, `eval` in XML data, QWeb PDF reports, Excel exports, and HTTP controllers.

---

### 1. Explain the main Odoo XML View Types and Actions

- **`form`:** Detailed view for creating/editing a single record (`<sheet>`, `<header>`, `<group>`, `<notebook>`, `<page>`).
- **`list` (formerly `tree` in Odoo < 18):** Tabular view supporting `editable="top|bottom"`, `multi_edit="1"`, `default_order`, `decoration-*`, and numeric aggregations (`sum`, `avg`, `min`, `max` on stored numeric fields).
- **`search`:** Controls search bar fields, `<filter>` presets, `filter_domain`, and `<group expand="0">` group-by options.
- **`kanban`:** Card-based QWeb layout (`t-name="card"` / `kanban-box`) with `default_group_by` and `quick_create`.
- **`calendar` / `pivot` / `graph`:** Date-based (`date_start`, `color`), multidimensional (`type="row|col|measure"`), and chart analytics views.
- **`ir.actions.act_window`:** Binds a model, `view_mode`, optional `domain`, and `context` to a menu item (`<menuitem>`) or smart button.

---

### 2. What are important View & Field Attributes vs. Widgets?

- **List/Form view attributes:**
  - `create="0"`, `edit="0"`, `delete="0"`, `duplicate="0"` disable CRUD actions at the view level.
  - `optional="show"` / `optional="hide"` lets users toggle columns in list views; `column_invisible="1"` hides an entire column conditionally.
  - `force_save="1"` ensures a `readonly` field modified by an `@api.onchange` is still sent to the server on save.
  - `no_open="1"`, `no_create="1"`, `no_create_edit="1"`, `no_quick_create="1"` restrict `Many2one` creation/navigation in `options="{...}"`.
  - Dynamic visibility/readonly/required: Odoo 17+ uses direct Python expressions (`invisible="state != 'draft'"`) replacing the legacy `attrs="{'invisible': [...]}"` syntax.
- **Common Odoo Field Widgets (`widget="..."`):**
  - `handle` (drag-and-drop sequence ordering in list views), `boolean_toggle`, `many2many_tags`, `statusbar` (`statusbar_visible="draft,sent,done"`), `badge`, `radio`, `remaining_days`, `many2one_avatar_user`, `many2one_avatar_employee`, `kanban_activity`, `statinfo` (inside `<button class="oe_stat_button" ...>` smart buttons), `color_picker`, `code` (`options="{'mode': 'python'}"`), `domain`.
- **What is NOT a widget:** `sum`, `decoration-*` (`decoration-danger`, `decoration-success`, `decoration-bf`, `decoration-it`), `eval`, and CSS classes like `oe_highlight`, `oe_read_only`, `oe_edit_only`, `oe_inline` are attributes/classes—not `widget` values.

---

### 3. How does XPath View Inheritance work?

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

        <!-- Add a new tab inside notebook -->
        <xpath expr="//notebook" position="inside">
            <page string="Extra Info" name="extra_info">
                <group>
                    <field name="custom_notes"/>
                </group>
            </page>
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

### 4. How do `domain`, `context`, and XML `eval` work?

- **Domain:** Prefix-notation list of tuples filtering records, e.g. `['&', ('state', '=', 'sale'), '|', ('user_id', '=', uid), ('team_id', '!=', False)]`.
- **Context:** Dictionary carrying defaults (`default_partner_id`), default search filters (`search_default_my_orders: 1`), language (`lang`), or flags (`active_test: False`).
- **XML `eval="..."`:** By default, text inside `<field>` tags is parsed as a **string** (`<field name="active">False</field>` would be a non-empty string!). Using `eval="..."` evaluates the expression via `safe_eval` with access to `ref('module.xml_id')`, `time`, `datetime`, `relativedelta`, and `Command` tuples—essential for booleans (`eval="False"`), numbers (`eval="2.5"`), and `Many2many` commands (`eval="[(4, ref('base.group_user'))]"`).

---

### 5. How do QWeb PDF Reports and Paper Formats work in Odoo?

Odoo uses **QWeb** (XML templating) compiled to HTML and rendered to PDF via `wkhtmltopdf`:

1. Optional **`report.paperformat`** defines page size (`A4`, `Letter`, `Custom`), `orientation` (`Portrait`/`Landscape`), margins (`margin_top`, `margin_bottom`, `margin_left`, `margin_right`), `header_spacing`, and `dpi`.
2. **`ir.actions.report`** registers `report_type="qweb-pdf"` (or `qweb-html`), `report_name="my_module.report_doc"`, `paperformat_id`, `binding_model_id`, and `binding_view_types` (`form,list`).
3. **QWeb `<template>`** wraps output in `<t t-call="web.html_container">` and `<t t-call="web.external_layout">` (or `web.basic_layout`), iterating over `docs` (`<t t-foreach="docs" t-as="o">`).
4. **Core QWeb T-directives:**
   - `t-foreach` / `t-as` / `t-key`: Loops and DOM tracking keys.
   - `t-if` / `t-elif` / `t-else`: Conditional rendering.
   - `t-set="total" t-value="..."`: Variable assignment.
   - `t-field="o.amount_total"`: Formatted field rendering (currency, dates, translations).
   - `t-esc` vs. `t-out`: In modern Odoo, `t-out` escapes values by default unless wrapped in `markupsafe.Markup` (preventing XSS), whereas legacy `t-raw` rendered unescaped HTML.
   - `t-call`: Reuses sub-templates.
5. **Custom Report Model:** An `AbstractModel` named `report.<module>.<template_id>` implementing `_get_report_values(self, docids, data=None)` injects custom Python calculations into the QWeb context.

---

### 6. How do you generate Excel (`.xlsx`) Reports in Odoo?

Odoo Core does not ship a built-in `qweb-xlsx` report engine out of the box (though Enterprise/OCA modules add it). Common interview-ready approaches:

1. **Wizard / Action + HTTP Controller (Core-only approach):**
   - A `TransientModel` wizard collects filter parameters and returns an `ir.actions.act_url` pointing to `/my_module/export_xlsx?wizard_id=...`.
   - The `@http.route` controller uses `xlsxwriter` (bundled with Odoo) writing into an in-memory `io.BytesIO()` buffer and returns `request.make_response(...)` with `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` and `Content-Disposition: attachment; filename="report.xlsx"`.
2. **OCA `report_xlsx` module:** Inherit `report.report_xlsx.abstract` and implement `generate_xlsx_report(self, workbook, data, records)`.

---

### 7. How do HTTP Controllers work in Odoo?

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
- **`sitemap=False`:** Prevents public website routes from being indexed in `/sitemap.xml`.
