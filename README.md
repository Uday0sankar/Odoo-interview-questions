# Python & Odoo Interview Questions (Beginner to Advanced)

A practical, developer-maintained collection of **Python (Beginner to Advanced Interview Questions)** alongside **Odoo Technical**, **PostgreSQL / SQL**, and **Odoo Functional Interview Questions**—built for quick revision, real-world debugging, and peer collaboration.

---

## Repository Layout

```text
Odoo-interview-questions/
├── README.md
├── python/
│   ├── 01_basics_beginner_to_core.md
│   ├── 02_data_structures.md
│   ├── 03_functions_decorators.md
│   ├── 04_oop_and_dunders.md
│   └── 05_advanced_internals_concurrency.md
├── technical/
│   ├── general.md
│   ├── 01_core_orm_models.md
│   ├── 02_fields_api_decorators.md
│   ├── 03_views_qweb_reports.md
│   ├── 04_security_access.md
│   ├── 05_performance_cron_deployment.md
│   └── 06_postgresql_sql.md
└── functional/
    ├── general.md
    ├── 01_sales_crm.md
    ├── 02_purchase_inventory.md
    ├── 03_accounting_multicompany.md
    └── 04_manufacturing_hr.md
```

---

## Python — Beginner to Advanced Interview Questions (`python/`)

A step-by-step track starting from beginner basics up through production Python internals:

| File | Level | Topics Covered |
| :--- | :--- | :--- |
| [01_basics_beginner_to_core.md](./python/01_basics_beginner_to_core.md) | Beginner | Variables, dynamic typing, basic types (`int`, `float`, `str`, `bool`, `None`), `==` vs. `is`, truthy/falsy, `if`/`elif`/`else`, `for`/`while`, `break`/`continue`/`pass`, string formatting & methods, `return` vs. `print` |
| [02_data_structures.md](./python/02_data_structures.md) | Intermediate | `list`, `tuple`, `dict`, `set`, mutability & hashability, slicing, shallow vs. deep copy, comprehensions & generator expressions, `get`/`setdefault`/`defaultdict`/`Counter`, Big-O complexity |
| [03_functions_decorators.md](./python/03_functions_decorators.md) | Intermediate | `*args`/`**kwargs`, mutable default arguments, LEGB & closures, late binding pitfall, decorators (`@functools.wraps`), iterators & generators (`yield`), `lambda`, `map`/`filter`/`reduce`, `partial` & `lru_cache` |
| [04_oop_and_dunders.md](./python/04_oop_and_dunders.md) | Upper-Intermediate | Class vs. instance attributes, `@classmethod` vs. `@staticmethod`, multiple inheritance & MRO (`super()`), `__new__` vs. `__init__`, dunder methods, `@property`, `@dataclass` & `__slots__` |
| [05_advanced_internals_concurrency.md](./python/05_advanced_internals_concurrency.md) | Advanced | Reference counting & cyclic GC, GIL, `threading` vs. `multiprocessing` vs. `asyncio`, `try`/`except`/`else`/`finally` & custom exceptions, context managers (`with` / `@contextmanager`), `if __name__ == "__main__":`, type hints |

---

## Odoo Technical & PostgreSQL Interview Questions (`technical/`)

| File | Topics Covered |
| :--- | :--- |
| [general.md](./technical/general.md) | Webhooks, Odoo MVC architecture, module structure & `__manifest__.py` flags, CLI & `odoo-bin shell` (`registry().cursor()`, port 8069 troubleshooting), automated testing (`TransactionCase`, `@tagged`), Git workflows (`rebase`, `stash`, `reset`, `reflog`) |
| [01_core_orm_models.md](./technical/01_core_orm_models.md) | `Model` / `TransientModel` / `AbstractModel`, `_name`/`_inherit`/`_inherits`, `search` vs. `browse` (`exists`, `search_count`), `search_read` & `read_group`, `create`/`write`/`unlink`, `ensure_one`, `filtered`/`mapped`/`sorted`, `sudo`/`with_context`/`with_company`, `_is_public()` |
| [02_fields_api_decorators.md](./technical/02_fields_api_decorators.md) | Relational fields & `Command` syntax, `@api.onchange` vs. `@api.depends`, computed field pitfalls, `_sql_constraints` vs. `@api.constrains`, `@api.model`, `ir.sequence`, Decimal Precision (`digits`) |
| [03_views_qweb_reports.md](./technical/03_views_qweb_reports.md) | XML views (`form`, `list`, `search`, `kanban`, `pivot`, `calendar`), view/field attributes & widgets (`sum`, `decoration-*`, `statusbar`, `badge`, `handle`, smart buttons), XPath inheritance, `domain`/`context`/`eval`, QWeb PDF reports & `paperformat`, Excel (`.xlsx`) exports, HTTP/JSON controllers |
| [04_security_access.md](./technical/04_security_access.md) | Security layers (`ir.module.category`, `res.groups` hierarchy), `ir.model.access.csv` vs. `ir.rule`, global vs. group record rules & `domain_force` patterns, multi-company enforcement (`check_company`), `sudo()` & SQL injection safety |
| [05_performance_cron_deployment.md](./technical/05_performance_cron_deployment.md) | N+1 queries & prefetching, ORM vs. raw SQL (`flush_all` / `invalidate_all`), `ir.cron` scheduled actions, worker sizing & memory formulas, production `odoo.conf` tuning |
| [06_postgresql_sql.md](./technical/06_postgresql_sql.md) | `psql` commands & DB size checks, `SERIAL`/`BIGSERIAL` & `PRIMARY KEY` vs. `UNIQUE`, `btree` & GIN/trigram indexes, `WHERE`/`GROUP BY`/`HAVING`, `JOIN`s & `EXISTS`, `LIKE` vs. `ILIKE`, `COALESCE` & `NULLIF`, `DATE`/`TIMESTAMP`/`AGE`/`EXTRACT`, `EXPLAIN ANALYZE` |

---

## Odoo Functional Interview Questions (`functional/`)

| File | Topics Covered |
| :--- | :--- |
| [general.md](./functional/general.md) | Record rules vs. access rights (functional view), user types (Internal/Portal/Public), chatter & activities, Developer Mode, implementation setup checklist |
| [01_sales_crm.md](./functional/01_sales_crm.md) | Sales order lifecycle, delivery creation issues, ordered vs. delivered invoicing policies, pricelists & discounts, Units of Measure (UoM), CRM leads vs. opportunities, pipeline stages & lost reasons |
| [02_purchase_inventory.md](./functional/02_purchase_inventory.md) | Purchase flow & 2-way/3-way matching, product types, serial vs. lot numbers, Standard/AVCO/FIFO costing, automated vs. manual valuation, reordering rules, MTO & dropshipping, landed costs, inventory adjustments, pickings/moves/quants |
| [03_accounting_multicompany.md](./functional/03_accounting_multicompany.md) | Accounts affected by SO invoicing, payment reconciliation & outstanding accounts, credit notes & refunds, fiscal positions, multi-currency & FX gains/losses, multi-company & inter-company transactions |
| [04_manufacturing_hr.md](./functional/04_manufacturing_hr.md) | BoM types (Manufacture, Kit, Subcontracting), Manufacturing Orders & Work Centers, subcontracting flow, scrap orders, `hr.employee` vs. `res.users`, contracts, time off, expenses & payroll basics |

---

## Contributing & Spot-a-Fix Welcome

This repo is meant to grow like shared developer notes—not a rigid textbook. If you're a friend, teammate, or fellow Odoo/Python developer and spot a typo, want to add an edge case (like `async with`, Odoo 18/19 version diffs, or extra real-world interview questions), feel free to open a PR or raise an issue!

- Put questions in the matching topic file (`python/`, `technical/`, or `functional/`), or in `general.md` if cross-cutting.
- Use sequential `### N. Question Title` headings separated by `---` dividers.
- Close fenced code blocks cleanly (`python`, `xml`, `sql`, `bash`, `csv`, `text`) and keep functional business-process questions inside `functional/`.
