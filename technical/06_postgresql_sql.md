# PostgreSQL & SQL Interview Questions (for Odoo Developers)

Core PostgreSQL and SQL interview questions tailored for Odoo technical interviews: `psql` administration, data types, constraints, `btree` indexes, SQL clauses, `NULL` safety, date arithmetic, and query optimization.

---

### 1. What are essential `psql` CLI and Meta-Commands for Odoo Debugging?

- **Connect to a database:**
  `psql -h localhost -p 5432 -U odoo -d my_odoo_db`
- **Key `psql` meta-commands:**
  - `\l` / `\list`: List all databases.
  - `\c <db>`: Switch database.
  - `\dt`: List tables in the current database (`\d <table>` inspects columns, types, constraints, and indexes).
  - `\du`: List database roles/users.
  - `\x on`: Toggle expanded display (very helpful when inspecting wide Odoo tables or `jsonb` columns like translated fields).
  - `\q`: Quit `psql`.
- **Check database sizes in PostgreSQL:**
  ```sql
  SELECT datname AS database_name,
         pg_size_pretty(pg_database_size(datname)) AS size
  FROM pg_database
  ORDER BY pg_database_size(datname) DESC;
  ```

---

### 2. Compare `SERIAL` / `BIGSERIAL`, `PRIMARY KEY`, and `UNIQUE` Constraints

- **`SERIAL` / `BIGSERIAL`:** Shorthand for an `INTEGER` (4-byte) or `BIGINT` (8-byte) column backed by an auto-incrementing PostgreSQL **sequence** (`DEFAULT nextval('table_id_seq')`). It is **not** a constraint by itself—pair it with `PRIMARY KEY` for uniqueness and `NOT NULL`.
- **`PRIMARY KEY` vs. `UNIQUE`:**

| Feature | `PRIMARY KEY` | `UNIQUE` Constraint |
| :--- | :--- | :--- |
| **Uniqueness** | Enforced (creates unique B-tree index) | Enforced (creates unique B-tree index) |
| **`NOT NULL`** | Always implied | Optional (must add `NOT NULL` explicitly; multiple `NULL`s are allowed by default) |
| **Count per table** | Exactly 1 (single or composite) | Multiple allowed |

Other standard constraints: `FOREIGN KEY` (`REFERENCES`), `NOT NULL`, `CHECK (...)`, and `DEFAULT`.

---

### 3. How does a `btree` Index work in PostgreSQL, and when are other Index Types used?

A **B-tree (balanced tree)** is PostgreSQL's default index structure (created automatically for `PRIMARY KEY` and `UNIQUE` constraints, and by Odoo's `index=True` on model fields).

- **Best for:** Equality (`=`), range queries (`<`, `>`, `BETWEEN`), prefix `LIKE 'abc%'`, and `ORDER BY` sorting.
- **Other PostgreSQL index types in Odoo:**
  - **GIN (`gin`):** Indexing `jsonb` columns (e.g., Odoo 16+ translated fields), arrays, and `tsvector` full-text search.
  - **GIN / GiST with `pg_trgm` (`index='trigram'` in Odoo):** Speeds up substring searches (`ILIKE '%term%'`) on large partner/product name columns.

---

### 4. Differentiate `WHERE` vs. `HAVING`, and explain `GROUP BY` Rules

- **`WHERE`** filters **individual rows before** grouping/aggregation happens.
- **`GROUP BY`** collapses rows sharing identical key values into summary groups. Every non-aggregated column in `SELECT` **must** appear in `GROUP BY` (otherwise PostgreSQL raises `column must appear in the GROUP BY clause or be used in an aggregate function`).
- **`HAVING`** filters **aggregated groups after** `GROUP BY` has run.

```sql
SELECT partner_id, COUNT(*) AS order_count, SUM(amount_total) AS total_sales
FROM sale_order
WHERE state = 'sale'
GROUP BY partner_id
HAVING COUNT(*) > 5
ORDER BY total_sales DESC;
```

---

### 5. Compare `INNER JOIN`, `LEFT JOIN`, and `EXISTS`

- **`INNER JOIN`:** Returns only rows with matching keys in **both** tables.
- **`LEFT JOIN`:** Returns **all** rows from the left table, with `NULL`s for unmatched right-table columns.
- **`EXISTS (SELECT 1 FROM ...)`:** Checks semi-join existence and stops scanning on the first match—often cleaner and faster than `IN` (especially when subqueries might contain `NULL`s, where `NOT IN` can unexpectedly return 0 rows!).

---

### 6. Compare `LIKE` vs. `ILIKE`, and `IN` vs. `BETWEEN`

- **`LIKE` vs. `ILIKE`:** `LIKE` is case-sensitive (`%` matches 0+ chars, `_` matches 1 char); **`ILIKE`** is PostgreSQL's **case-insensitive** pattern matcher (used heavily by Odoo's `ilike` domain operator).
- **`IN` vs. `BETWEEN`:** `IN (v1, v2, ...)` matches a discrete list of values; `BETWEEN low AND high` matches an **inclusive** continuous range (`>= low AND <= high`).

---

### 7. How do `COALESCE` and `NULLIF` prevent `NULL` and Division-by-Zero errors?

- **`COALESCE(a, b, ...)`:** Returns the first non-`NULL` argument (ideal for fallback defaults).
- **`NULLIF(a, b)`:** Returns `NULL` if `a = b`, otherwise returns `a`.
- **Classic safe-division pattern:**

```sql
-- Prevents division by zero by turning 0 into NULL, then falling back to 0
SELECT COALESCE(amount / NULLIF(discount, 0), 0) AS avg_per_discount
FROM sales;
```

Note also: always check `NULL` with `IS NULL` / `IS NOT NULL`, never `= NULL` (since `NULL = NULL` evaluates to `UNKNOWN`).

---

### 8. Explain PostgreSQL Date/Time Types (`DATE`, `TIMESTAMP`, `TIMESTAMPTZ`) and `AGE` / `EXTRACT`

- **`DATE`:** Calendar date (`YYYY-MM-DD`). Subtracting two dates returns an **integer** number of days (`'2025-07-10'::date - '2025-07-01'::date` -> `9`).
- **`TIMESTAMP` vs. `TIMESTAMPTZ`:** `TIMESTAMP` (`WITHOUT TIME ZONE`) stores raw date/time (Odoo stores `Datetime` columns in **UTC** as `timestamp without time zone` and converts in Python/UI). `TIMESTAMPTZ` stores UTC internally and converts to/from the session timezone (`AT TIME ZONE 'Asia/Kolkata'`).
- **`INTERVAL` & `AGE()`:** `AGE(later, earlier)` returns a human-readable `INTERVAL` (e.g., `'5 years 11 mons 30 days'`), and `NOW() + INTERVAL '5 days'` performs calendar-aware arithmetic.
- **`EXTRACT(field FROM source)` / `DATE_PART()`:** Extracts subfields (`YEAR`, `MONTH`, `DOW`, `ISODOW`, `WEEK`, `QUARTER`, `EPOCH`).

---

### 9. How do you diagnose slow PostgreSQL Queries in Odoo?

1. **Log slow queries:** Enable `log_min_duration_statement = 500` (ms) in `postgresql.conf` or run Odoo with `--log-sql`.
2. **Run `EXPLAIN (ANALYZE, BUFFERS)`:** Look for unexpected `Seq Scan` on large tables (`account_move_line`, `stock_move`, `mail_message`), bad row estimates (stale statistics -> run `ANALYZE`), or disk spills.
3. **Check locks & active queries:** Query `pg_stat_activity` for `state = 'active'` or `wait_event_type = 'Lock'` (common in Odoo when long-running cron jobs or concurrent `stock.quant` updates contend on rows).
4. **Autovacuum & bloat:** Ensure PostgreSQL `autovacuum` keeps high-churn tables (`ir_attachment`, `mail_message`, `stock_quant`) clean so index scans stay fast.
