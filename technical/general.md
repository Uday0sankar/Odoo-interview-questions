# General Technical Interview Questions (Odoo)

Cross-cutting technical Odoo interview questions covering architecture, module structure, integrations, CLI tooling, debugging, and automated testing.

> **Looking for deeper technical or related topics?**
> - ORM & Models: [01_core_orm_models.md](./01_core_orm_models.md)
> - Fields & API Decorators: [02_fields_api_decorators.md](./02_fields_api_decorators.md)
> - Views, QWeb & Reports: [03_views_qweb_reports.md](./03_views_qweb_reports.md)
> - Security & Access Control: [04_security_access.md](./04_security_access.md)
> - Performance, Cron & Deployment: [05_performance_cron_deployment.md](./05_performance_cron_deployment.md)
> - Core Python Questions: [python/01_data_structures.md](../python/01_data_structures.md)
> - Functional / Business Process Questions: [functional/general.md](../functional/general.md)

---

### 1. What is a Webhook, and how is it used in Odoo?

A webhook is an event-driven HTTP callback (typically an HTTP `POST` with a JSON payload) sent from one system to another when a specific event occurs, avoiding inefficient polling.

- **Outbound webhooks from Odoo:** Triggered via **Automated Actions** (`base.automation` / `ir.actions.server` webhook action in modern Odoo) or model method overrides (`create`, `write`, `action_confirm`) using `requests` or a queued job to notify external services (e.g., shipping carriers, ERPs, Slack).
- **Inbound webhooks to Odoo:** Implemented via an `@http.route(..., type='json', auth='public', csrf=False)` controller that validates a shared secret/HMAC signature and delegates processing with `request.env(su=True)` or a dedicated service user.

---

### 2. Explain Odoo's Architecture (Modified MVC / MTV)

Odoo follows a multi-tier architecture built on Python, PostgreSQL, and a JavaScript/OWL web client:

- **Model (Python + PostgreSQL via Odoo ORM):** Defines business objects, fields, constraints, and business logic (`models.Model`).
- **View (XML + OWL/QWeb):** Defines UI layouts (`form`, `list`/`tree`, `kanban`, `search`) and QWeb templates rendered by the web client.
- **Controller (Python `http.Controller`):** Handles custom HTTP/JSON endpoints (website pages, portals, REST/webhook endpoints). Standard backend CRUD goes through generic JSON-RPC dataset endpoints (`/web/dataset/call_kw`), so custom controllers are optional for internal backend modules.

---

### 3. What is the structure of a standard Odoo Module?

```text
my_module/
├── __init__.py            # Imports models/, controllers/, wizard/, report/
├── __manifest__.py          # Metadata: name, version, depends, data, assets, license
├── models/
│   ├── __init__.py
│   └── sale_order.py
├── views/
│   └── sale_order_views.xml
├── security/
│   ├── ir.model.access.csv
│   └── security_rules.xml
├── data/                    # Default/seed records & ir.cron definitions
├── wizard/                  # TransientModel wizards & their views
├── report/                  # QWeb report templates & report actions
├── controllers/             # HTTP/JSON routes
├── static/                  # JS (OWL), CSS/SCSS, XML templates, images
└── tests/                   # TransactionCase / HttpCase unit tests
```

**Important:** Every file in `data` inside `__manifest__.py` is loaded in order—security groups and `ir.model.access.csv` must be declared **before** views or data that reference them, and XML actions must be declared **before** menu items that reference them.

---

### 4. What are common Odoo CLI commands for development?

- **Scaffold a new module:** `odoo-bin scaffold my_module /path/to/addons`
- **Install or upgrade a module:** `odoo-bin -c odoo.conf -d mydb -i my_module` / `-u my_module --stop-after-init`
- **Interactive shell:** `odoo-bin shell -c odoo.conf -d mydb` (gives access to `env` for testing ORM queries safely; changes require `env.cr.commit()` to persist)
- **Developer reload mode:** `--dev=all` (or `--dev=reload,xml,qweb,assets`) reloads Python/XML/assets without restarting the server for every view change.

---

### 5. How do you write and run Automated Tests in Odoo?

Odoo tests extend `odoo.tests.common` classes (`TransactionCase`, `SingleTransactionCase`, `HttpCase`) and are tagged with `@tagged`:

```python
from odoo.tests import TransactionCase, tagged
from odoo.exceptions import ValidationError


@tagged('post_install', '-at_install')
class TestSaleApproval(TransactionCase):
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.partner = cls.env['res.partner'].create({'name': 'Test Customer'})

    def test_discount_limit(self):
        with self.assertRaises(ValidationError):
            self.env['sale.order'].create({
                'partner_id': self.partner.id,
                'custom_discount': 150.0,
            })
```

- Run tests with: `odoo-bin -c odoo.conf -d test_db -i my_module --test-enable --stop-after-init` (or filter with `--test-tags /my_module`).
- `TransactionCase` wraps each test method in a savepoint/rollback so tests stay isolated.

---

### 6. What happens during a Module Upgrade (`-u`) vs. Fresh Install (`-i`)?

- **Install (`-i`):** Runs schema creation/updates for the module's models, loads all `data` and `demo` XML/CSV files (creating `ir.model.data` XML IDs), and registers menus, views, and security rules.
- **Upgrade (`-u`):** Updates PostgreSQL columns for modified/new fields, reloads `data` files (updating records whose `noupdate="0"`, while skipping records marked `noupdate="1"`), and executes `pre-migrate` / `post-migrate` scripts if version migrations are provided.
