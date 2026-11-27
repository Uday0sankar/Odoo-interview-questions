# Odoo & Python Interview Questions

A structured, interview-ready collection of **Core Python**, **Odoo Technical**, and **Odoo Functional** questions and answers with clean code examples and business-process explanations.

---

## Repository Layout

```text
Odoo-interview-questions/
├── README.md
├── python/
│   ├── 01_data_structures.md
│   ├── 02_functions_decorators.md
│   ├── 03_oop_and_dunders.md
│   └── 04_memory_concurrency_exceptions.md
├── technical/
│   ├── general.md
│   ├── 01_core_orm_models.md
│   ├── 02_fields_api_decorators.md
│   ├── 03_views_qweb_reports.md
│   ├── 04_security_access.md
│   └── 05_performance_cron_deployment.md
└── functional/
    ├── general.md
    ├── 01_sales_crm.md
    ├── 02_purchase_inventory.md
    ├── 03_accounting_multicompany.md
    └── 04_manufacturing_hr.md
```

---

## Core Python (`python/`)

| File | Topics Covered |
| :--- | :--- |
| [01_data_structures.md](./python/01_data_structures.md) | `list`, `tuple`, `dict`, `set`, mutability & hashability, shallow vs. deep copy, `==` vs. `is`, comprehensions, slicing, Big-O complexity |
| [02_functions_decorators.md](./python/02_functions_decorators.md) | `*args`/`**kwargs`, mutable default arguments, LEGB & closures, late binding pitfall, decorators (`@functools.wraps`), iterators & generators (`yield`), `lambda`, `map`/`filter`/`reduce`, `partial` & `lru_cache` |
| [03_oop_and_dunders.md](./python/03_oop_and_dunders.md) | Class vs. instance attributes, `@classmethod` vs. `@staticmethod`, multiple inheritance & MRO (`super()`), `__new__` vs. `__init__`, dunder methods, `@property`, `@dataclass` & `__slots__` |
| [04_memory_concurrency_exceptions.md](./python/04_memory_concurrency_exceptions.md) | Reference counting & cyclic GC, GIL, `threading` vs. `multiprocessing` vs. `asyncio`, exception handling (`try`/`except`/`else`/`finally`), context managers (`with`), `__name__ == "__main__"`, type hints |

---

## Odoo Technical (`technical/`)

| File | Topics Covered |
| :--- | :--- |
| [general.md](./technical/general.md) | Webhooks, Odoo MVC architecture, module structure (`__manifest__.py`), CLI commands, automated testing (`TransactionCase`, `@tagged`), install vs. upgrade |
| [01_core_orm_models.md](./technical/01_core_orm_models.md) | `Model` / `TransientModel` / `AbstractModel`, `_name`/`_inherit`/`_inherits`, `search` vs. `browse`, `search_read` & `read_group`, `create`/`write`/`unlink`, `ensure_one`, `filtered`/`mapped`/`sorted`, `sudo`/`with_context`/`with_company` |
| [02_fields_api_decorators.md](./technical/02_fields_api_decorators.md) | Relational fields & `Command` syntax, `@api.onchange` vs. `@api.depends`, computed field pitfalls, `_sql_constraints` vs. `@api.constrains`, `@api.model` |
| [03_views_qweb_reports.md](./technical/03_views_qweb_reports.md) | XML views (`form`, `list`, `search`, `kanban`), XPath view inheritance, domains & context, QWeb PDF reports, Excel (`.xlsx`) exports, HTTP/JSON controllers |
| [04_security_access.md](./technical/04_security_access.md) | Security layers (`res.groups`), `ir.model.access.csv` vs. `ir.rule`, global vs. group record rules, multi-company enforcement (`check_company`), `sudo()` & SQL injection safety |
| [05_performance_cron_deployment.md](./technical/05_performance_cron_deployment.md) | N+1 queries & prefetching, ORM vs. raw SQL (`flush_all` / `invalidate_all`), `ir.cron` scheduled actions, worker sizing & memory limits, PostgreSQL indexing & tuning |

---

## Odoo Functional (`functional/`)

| File | Topics Covered |
| :--- | :--- |
| [general.md](./functional/general.md) | Record rules vs. access rights (functional view), user types (Internal/Portal/Public), chatter & activities, Developer Mode, implementation setup checklist |
| [01_sales_crm.md](./functional/01_sales_crm.md) | Sales order lifecycle, delivery creation issues, ordered vs. delivered invoicing policies, pricelists & discounts, Units of Measure (UoM), CRM leads vs. opportunities, pipeline stages & lost reasons |
| [02_purchase_inventory.md](./functional/02_purchase_inventory.md) | Purchase flow & 2-way/3-way matching, product types, serial vs. lot numbers, Standard/AVCO/FIFO costing, automated vs. manual valuation, reordering rules, MTO & dropshipping, landed costs, inventory adjustments, pickings/moves/quants |
| [03_accounting_multicompany.md](./functional/03_accounting_multicompany.md) | Accounts affected by SO invoicing, payment reconciliation & outstanding accounts, credit notes & refunds, fiscal positions, multi-currency & FX gains/losses, multi-company & inter-company transactions |
| [04_manufacturing_hr.md](./functional/04_manufacturing_hr.md) | BoM types (Manufacture, Kit, Subcontracting), Manufacturing Orders & Work Centers, subcontracting flow, scrap orders, `hr.employee` vs. `res.users`, contracts, time off, expenses & payroll basics |

---

## Contributing Guidelines

- Place questions in the matching topic file (`python/`, `technical/`, or `functional/`), or in `general.md` if cross-cutting.
- Use sequential `### N. Question Title` headings separated by `---` dividers.
- Close all fenced code blocks with explicit language tags (`python`, `xml`, `csv`, `text`) and avoid duplicating functional topics inside `technical/`.
