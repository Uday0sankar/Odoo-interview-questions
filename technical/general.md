# General Technical Interview Questions (Odoo)

Cross-cutting technical Odoo interview questions covering architecture, module structure (`__manifest__.py`), CLI & Odoo shell, Git workflows, debugging, and automated testing.

> **Looking for deeper technical or related topics?**
> - ORM & Models: [01_core_orm_models.md](./01_core_orm_models.md)
> - Fields, Sequences & API Decorators: [02_fields_api_decorators.md](./02_fields_api_decorators.md)
> - Views, Widgets, QWeb & Reports: [03_views_qweb_reports.md](./03_views_qweb_reports.md)
> - Security & Access Control: [04_security_access.md](./04_security_access.md)
> - Performance, Cron & Server Configuration: [05_performance_cron_deployment.md](./05_performance_cron_deployment.md)
> - PostgreSQL & SQL Essentials: [06_postgresql_sql.md](./06_postgresql_sql.md)
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

### 3. What is the structure of a standard Odoo Module and `__manifest__.py`?

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
├── data/                    # Sequences, default records & ir.cron definitions
├── wizard/                  # TransientModel wizards & their views
├── report/                  # QWeb report templates & report actions
├── controllers/             # HTTP/JSON routes
├── static/                  # JS (OWL), CSS/SCSS, XML templates, images
└── tests/                   # TransactionCase / HttpCase unit tests
```

Key `__manifest__.py` flags:
- `depends`: Modules that must be installed before this module (`['base', 'sale']`).
- `data`: XML/CSV files loaded on install and upgrade **in declared order** (`security/` groups and `ir.model.access.csv` must load before views/data referencing them; actions before menu items).
- `installable`: Whether the module can be installed (`True`).
- `application`: `True` if it appears as a top-level app in the Apps dashboard (`False` for technical/bridge modules).
- `auto_install`: `True` if Odoo should automatically install the module as soon as all modules in `depends` are installed.
- `license`: e.g. `'LGPL-3'` or `'OPL-1'`.

---

### 4. What are common Odoo CLI & Shell commands for development and troubleshooting?

- **Scaffold a new module:** `odoo-bin scaffold my_module /path/to/addons`
- **Install or upgrade a module:**
  ```bash
  ./odoo-bin -c /etc/odoo/odoo.conf -d mydb -i my_module --stop-after-init
  ./odoo-bin -c /etc/odoo/odoo.conf -d mydb -u my_module --stop-after-init
  ```
- **Odoo Shell (source, system user, or Docker):**
  ```bash
  ./odoo-bin shell -d test_db -c /etc/odoo/odoo.conf --addons-path=/opt/odoo/custom_addons
  docker exec -it odoo odoo shell -d test_db -c /etc/odoo/odoo.conf
  ```
  Inside the shell, `env` is pre-bound; ORM writes only persist if you call `env.cr.commit()`. In standalone Python scripts without pre-bound `env`, open a cursor via the registry:
  ```python
  from odoo import SUPERUSER_ID, api, registry

  with registry('test_db').cursor() as cr:
      env = api.Environment(cr, SUPERUSER_ID, {})
      env['res.partner'].create({'name': 'Script Partner'})
      cr.commit()
  ```
- **Developer reload mode:** `--dev=all` (or `--dev=reload,xml,qweb,assets`) reloads Python/XML/assets without full manual restarts.
- **Port `8069` already in use (`Address already in use`):**
  ```bash
  sudo lsof -i :8069
  sudo kill -9 <PID>
  ```

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
- **Upgrade (`-u`):** Updates PostgreSQL columns for modified/new fields, reloads `data` files (updating records with `noupdate="0"`, while skipping records marked `noupdate="1"`), and executes `pre-migrate` / `post-migrate` scripts if version migrations are provided.

---

### 7. Which Git Workflows and Recovery Commands do Odoo Developers use daily?

- **Syncing & linear history (`fetch` + `rebase` vs. `pull`):**
  - `git fetch origin && git rebase origin/feature-branch` (or `git pull --rebase origin feature-branch`) replays local commits on top of remote changes without creating noisy merge commits.
  - During conflicts: fix files -> `git add <file>` -> `git rebase --continue` (or `git rebase --abort`).
- **Stashing work in progress (`git stash`):**
  - `git stash push -u -m "WIP: invoice fix"` stashes tracked **and untracked** (`-u`) changes with a label.
  - `git stash push --keep-index -m "WIP"` stashes unstaged edits while keeping staged changes ready to test/commit.
  - `git stash list`, `git stash show -p stash@{0}` (inspect diff), `git stash apply` (keep stash) vs. `git stash pop` (apply + drop), `git stash branch new-branch stash@{0}` (move stash onto a clean branch).
- **Undoing & recovering mistakes (`reset`, `amend`, `checkout`, `reflog`):**
  - `git commit --amend`: Updates the last local commit (never amend already-shared/pushed commits without team agreement).
  - `git reset --soft HEAD~1`: Undoes the last commit while keeping changes staged.
  - `git reset --hard HEAD` + `git clean -fd`: Discards all uncommitted tracked changes and removes untracked files/directories.
  - `git reflog`: Shows the local history of `HEAD` movements, letting you recover commits lost after an accidental `git reset --hard` or bad rebase.
