# Odoo Performance, Cron Jobs & Server Deployment

Technical Odoo interview questions covering ORM optimization, SQL execution, scheduled actions (`ir.cron`), worker calculation, memory limits, and `odoo.conf` tuning.

---

### 1. What is the N+1 Query Problem in Odoo, and how does Odoo mitigate it?

The **N+1 problem** happens when code runs 1 query to fetch N records and then triggers N separate queries inside a loop (or nested relational lookups).

**How Odoo helps automatically:**
- **Prefetching:** When reading a field on one record inside a loop over a recordset, Odoo's ORM prefetches that field (and related fields) in bulk for the entire recordset (in batches of ~1000 records).

**How developers still cause N+1 (and how to fix it):**
- Calling `search()` inside a `for` loop -> Collect IDs before the loop, run **one** `search([('id', 'in', ids)])` or `read_group()`, and map results in a Python dictionary.
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

### 4. How do you calculate Odoo Workers and Memory Limits in Multiprocessing Mode?

In production (`odoo.conf`), Odoo runs with `workers > 0` behind Nginx (plus 1 gevent/longpolling worker on `gevent_port` / `longpolling_port` and dedicated `max_cron_threads`):

**1. Worker Count Formula:**
- **CPU ceiling:** `max_workers_cpu = (CPU_cores * 2) + 1`
- **Concurrent-user rule of thumb:** **1 worker ≈ 6 concurrent active users** (`needed_workers ≈ concurrent_users / 6`)
- Pick `workers = min(needed_workers, max_workers_cpu)` while leaving CPU/RAM headroom for the OS, PostgreSQL, and cron threads.

**2. Memory Estimation Formula (per worker):**
- Assume ~80% light requests (~150 MB) and ~20% heavy requests like PDF/XLSX exports or imports (~1 GB):
  `Avg RAM per worker ≈ (0.8 × 150 MB) + (0.2 × 1000 MB) = ~320 MB` (typically budgeted at **320–512 MB** per worker).
- **Important:** `limit_memory_soft` and `limit_memory_hard` are **per worker**, not global. Always verify `workers × limit_memory_hard <= Available RAM − (OS + PostgreSQL overhead)`.

**Worked Example (16 GB RAM, 8 CPU cores, ~50–60 concurrent users):**
- CPU ceiling = `(8 × 2) + 1 = 17`; user need ≈ `60 / 6 = 10` -> choose **`workers = 8`** (leaving headroom).
- Reserve ~2 GB for OS/PostgreSQL -> ~14 GB available.
- Set `limit_memory_soft = 1610612736` (~1.5 GB) and `limit_memory_hard = 2147483648` (~2.0 GB) so even worst-case spikes stay within RAM, with `max_cron_threads = 1` (or `2` for heavy periodic jobs).

---

### 5. Explain the most critical `odoo.conf` Settings for Production

| Category | Key Settings | Purpose & Production Best Practice |
| :--- | :--- | :--- |
| **Paths & DB** | `addons_path`, `data_dir`, `db_host`, `db_port`, `db_user`, `db_password`, `db_maxconn` | `data_dir` holds filestore & sessions; keep `workers * db_maxconn` below PostgreSQL's `max_connections` |
| **Multi-DB & Security** | `admin_passwd`, `list_db`, `dbfilter` | Use a strong hashed `admin_passwd`, set `list_db = False` in production, and filter DBs by hostname (`dbfilter = ^%d$`) |
| **Workers & Limits** | `workers`, `max_cron_threads`, `limit_memory_soft`, `limit_memory_hard`, `limit_time_cpu`, `limit_time_real`, `limit_request` | `limit_memory_soft` recycles worker after current request; `limit_memory_hard` kills worker immediately; `limit_request` (e.g. `8192`) prevents long-lived memory leaks |
| **Web & Proxy** | `http_interface = 127.0.0.1`, `http_port = 8069`, `proxy_mode = True` | Bind locally behind Nginx/TLS and enable `proxy_mode = True` so `X-Forwarded-*` headers work properly |
| **Logging** | `log_level`, `log_handler`, `logfile` | Set `log_level = info` or `warn` and write to a rotated `logfile` |
