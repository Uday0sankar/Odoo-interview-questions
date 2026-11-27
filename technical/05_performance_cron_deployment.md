# Odoo Performance, Cron Jobs, PostgreSQL & Deployment

Technical Odoo interview questions covering ORM optimization, SQL execution, scheduled actions (`ir.cron`), worker sizing, and PostgreSQL tuning.

---

### 1. What is the N+1 Query Problem in Odoo, and how does Odoo mitigate it?

The **N+1 problem** happens when code runs 1 query to fetch N records and then triggers N separate queries inside a loop (or nested relational lookups).

**How Odoo helps automatically:**
- **Prefetching:** When reading a field on one record inside a loop over a recordset, Odoo's ORM prefetches that field (and related fields) in bulk for the entire recordset (in batches of ~1000 records).

**How developers still cause N+1 (and how to fix it):**
- Calling `search()` inside a `for` loop -> Instead, collect IDs before the loop, run **one** `search([('id', 'in', ids)])` or `read_group()`, and map results in a Python dictionary.
- Iterating over `One2many` lines inside another loop without `mapped()` or `read_group()`.
- Doing `len(records.search(...))` instead of `search_count(...)`.

---

### 2. When should you use Raw SQL (`cr.execute`) vs. the ORM, and what must you watch out for?

Use the ORM by default for security, computed triggers, and maintainability. Use raw SQL only for heavy bulk aggregations or batch migrations where the ORM is a proven bottleneck.

**Critical rules when mixing SQL with the ORM:**
1. **Flush pending ORM writes first:** Call `self.env.flush_all()` (or `Model.flush_model()`) before running a `SELECT`, otherwise uncommitted ORM changes in cache aren't in PostgreSQL yet.
2. **Invalidate cache after SQL writes:** Call `self.env.invalidate_all()` (or `Model.invalidate_model()`) after an `UPDATE`/`DELETE` so the ORM doesn't read stale cached values.
3. **Never bypass parameterization:** Always pass parameters via `%s` + tuple (`self.env.cr.execute(query, (param1, param2))`), never string formatting.
4. **Remember access control:** Raw SQL bypasses `ir.model.access` and `ir.rule`.

---

### 3. How do Scheduled Actions (`ir.cron`) work?

`ir.cron` executes a model method periodically in the background via Odoo's cron worker thread:

```xml
<record id="ir_cron_sync_inventory" model="ir.cron">
    <field name="name">Sync External Inventory</field>
    <field name="model_id" ref="stock.model_stock_quant"/>
    <field name="state">code</field>
    <field name="code">model._cron_sync_external_inventory()</field>
    <field name="interval_number">15</field>
    <field name="interval_type">minutes</field>
    <field name="active" eval="True"/>
</record>
```

**Best practices for large cron jobs:**
- Process records in **small batches** (e.g., `limit=200`) and catch per-record exceptions so one bad record doesn't roll back the entire batch.
- Keep execution time below `limit_time_cpu` / `limit_time_real` (or re-trigger with `self.env.ref(...)._trigger()` if needed).
- Avoid blocking HTTP workers by offloading heavy jobs to `ir.cron` or `queue_job`.

---

### 4. Explain Odoo Workers, Multiprocessing, and Memory Limits

In production (`odoo.conf`), Odoo runs in **multiprocessing mode** (`workers > 0`) behind a reverse proxy (Nginx):

- **HTTP Workers (`workers`):** Handle concurrent HTTP/RPC requests. General sizing rule of thumb: `workers = (CPU_cores * 2) + 1`, plus **1 `max_cron_threads`** (or dedicated cron workers) and **1 gevent/longpolling worker** (`gevent_port` / `longpolling_port` for live chat & bus notifications).
- **RAM Sizing:** Each worker typically consumes **~300–512 MB** under normal load (up to `limit_memory_hard`). Ensure `total_RAM >= (workers + max_cron_threads + 1) * ~512MB` plus PostgreSQL memory.
- **Key `odoo.conf` limits:**
  - `limit_memory_soft` (e.g., `2147483648` / 2 GB): Worker recycles gracefully after finishing its current request.
  - `limit_memory_hard` (e.g., `2684354560` / 2.5 GB): Worker is forcefully killed if exceeded.
  - `limit_time_cpu` (e.g., `60`) & `limit_time_real` (e.g., `120`): Prevents runaway requests from starving workers.
  - `db_maxconn`: Max PostgreSQL connections per worker process (keep `workers * db_maxconn` below PostgreSQL's `max_connections`).

---

### 5. What PostgreSQL optimizations matter most for Odoo?

1. **Indexes (`index=True` or `index='btree_not_null'` / `'trigram'`):** Add `index=True` on fields heavily used in `search()` domains, `ORDER BY`, or foreign keys (`Many2one` fields in Odoo are indexed automatically in modern versions unless disabled, though checking custom tables is good practice).
2. **`EXPLAIN (ANALYZE, BUFFERS)`:** Diagnose slow ORM queries (enable `--log-sql` or `log_min_duration_statement` in PostgreSQL) to spot sequential scans on large tables (`stock.move`, `account.move.line`, `mail.message`).
3. **Autovacuum & Bloat:** High-churn tables (`ir.attachment`, `mail.message`, `ir.logging`, `stock.quant`) need healthy PostgreSQL `autovacuum` settings so dead tuples don't degrade index scans.
4. **PgBouncer:** Useful in high-worker deployments to pool PostgreSQL connections cleanly.
