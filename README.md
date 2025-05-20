
# Advanced SQL for Query Tuning and Performance Optimization

**📘 Course:** Advanced SQL for Query Tuning and Performance Optimization  
**🧑‍🏫 Instructor:** Dan Sullivan  
**🎯 Platform:** LinkedIn Learning  
**💻 DBMS Focus:** PostgreSQL  

---

## 🔰 Module 1: SQL Execution Overview

### 🔍 Concepts:
- SQL is **declarative** — you specify what data you want, not how to get it.
- The DBMS converts this into a **procedural execution plan**.
- Execution plan steps:
  1. **Parsing**: SQL syntax is checked and converted to a parse tree.
  2. **Rewriting**: Logical transformations are applied.
  3. **Planning**: The optimizer chooses the best plan based on costs.
  4. **Execution**: The DB engine runs the chosen plan.

### 🔧 Tools:
- `EXPLAIN` → estimates the plan and costs.
- `EXPLAIN ANALYZE` → shows actual execution with timings and row counts.

### 🧠 Tips:
- Always inspect execution plans for:
  - Full table scans on large tables
  - Joins with high cost (especially nested loops on big sets)
  - Inefficient filters or function-based filters that block indexes

---

## 📂 Module 2: Understanding Query Plans in PostgreSQL

### 🌳 Key Operations:
- **Seq Scan**: Reads the whole table; used when no suitable index exists or for low selectivity queries.
- **Index Scan**: Uses B-tree or another index to fetch rows directly.
- **Bitmap Index Scan**: Combines multiple indexes efficiently and reduces random I/O.
- **Nested Loop Join**: Simple, used when one input is small or well-indexed.
- **Hash Join**: Builds a hash table in memory; good for large joins.
- **Merge Join**: Sorts inputs first, then merges; efficient on already sorted/indexed data.

### 🛠️ Example:
```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 5;
```

### 🧠 Tip:
- Look at “rows” and “loops” values to see if estimates are realistic.
- Large mismatch → bad stats → consider running `ANALYZE`.

---

## 📚 Module 3: Indexing for Performance

### 🧱 Index Types:

| Type     | Use Case                                   | Example                                                   |
|----------|---------------------------------------------|-----------------------------------------------------------|
| B-tree   | Default. Best for ranges and equality       | `CREATE INDEX ON staff(salary);`                          |
| Hash     | Fast equality (`=`) only                    | `CREATE INDEX USING HASH ON users(email);`                |
| GIN      | Full-text search, arrays, JSON              | `CREATE INDEX USING GIN ON articles(to_tsvector(...));`   |
| GiST     | Geospatial, complex types                   | `CREATE INDEX USING GiST ON shapes(geom);`                |
| BRIN     | Huge tables with naturally sorted data      | `CREATE INDEX USING BRIN ON logs(created_at);`            |

### Details:
- **B-tree** supports sorting, `BETWEEN`, `>`, `<`, `=`.
- **GIN** stores inverted lists of terms or keys.
- **BRIN** stores min/max values per block — minimal storage.

### ⚠️ Tips:
- Avoid functions inside WHERE (like `LOWER(name)`), unless you index it as an expression.
- Drop unused indexes — they slow down writes.
- Use `pg_stat_user_indexes` to check usage.

---

## 🔄 Module 4: Join Tuning Strategies

### 🤝 Join Algorithms:

| Join Type    | Best For                          | Notes |
|--------------|-----------------------------------|-------|
| Nested Loop  | Small datasets or indexed join    | Bad for large sets without index |
| Hash Join    | No indexes, large tables          | Uses memory — can spill to disk |
| Merge Join   | Sorted/indexed inputs             | Fastest if data is already ordered |

### 🛠️ Example:
```sql
EXPLAIN ANALYZE
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

### 🧠 Tip:
- Control join types using `enable_hashjoin`, `enable_mergejoin`, etc. for testing.

---

## 🧩 Module 5: Partitioning Strategies

### 📦 Partition Types:
- **Range**: e.g. `sale_date < '2023-01-01'`
- **List**: e.g. `region IN ('north', 'south')`
- **Hash**: distributes rows evenly

### Benefits:
- Faster queries when filtered by partition key.
- Smaller indexes per partition.
- Easier data archiving and maintenance.

### 🧠 Example:
```sql
CREATE TABLE sales (
  id INT,
  region TEXT,
  sale_date DATE
) PARTITION BY LIST (region);
```

---

## 🪞 Module 6: Materialized Views

### 📌 Use Case:
- When queries are slow and data doesn’t change often.
- E.g. heavy `JOIN + GROUP BY` queries on dashboards.

### 🔁 Refresh Strategies:
- Manual: `REFRESH MATERIALIZED VIEW`
- Concurrent refresh: allows querying while refreshing (add `CONCURRENTLY`)

### 🧠 Example:
```sql
CREATE MATERIALIZED VIEW top_customers AS
SELECT customer_id, SUM(amount)
FROM orders
GROUP BY customer_id;
```

---

## 📊 Module 7: Using Statistics for Optimization

### 📈 Tools:
- `ANALYZE`: Updates planner stats
- `pg_stats`: Shows histograms, null ratios, distinct counts
- `pg_stat_statements`: Logs execution data for all queries
- `auto_explain`: Can log plans automatically for long-running queries

### 🔍 Example:
```sql
SELECT * FROM pg_stats WHERE tablename = 'orders';
```

### 🧠 Tips:
- Check misestimates → rebuild stats
- Tune `default_statistics_target` if needed

---

## ⚙️ Module 8: Additional Optimization Techniques

### 🧠 Tips:
- CTEs are not always optimized unless marked `INLINE`.
- Avoid `SELECT *` in production — only fetch what’s needed.
- Consider `LIMIT` for large result sets.
- Use proper data types (e.g., `INT` vs `TEXT` for IDs).
- Enable parallelism: check `max_parallel_workers_per_gather`.

---

## 🧪 Final Thoughts

### ✅ Checklist for Tuning:
- [x] Use `EXPLAIN ANALYZE` always
- [x] Check index usage and scan types
- [x] Avoid unnecessary CTE materialization
- [x] Reduce data scanned (apply filters early)
- [x] Use partitioning for large tables
- [x] Monitor performance via `pg_stat_statements`
- [x] Keep statistics updated using `ANALYZE`


---

## 📎 Appendix: What is Random I/O?

### 🔤 Definition:
**Random I/O** stands for **Random Input/Output** — a term used to describe reading or writing data from/to **non-sequential (scattered)** locations on disk.

### 📦 In Databases:
- Occurs during **Index Scans** when PostgreSQL retrieves each row from different locations in the table.
- Requires the system to **jump around** the storage instead of reading continuously.

### 🔁 Comparison:

| Type            | Description                               | Speed       |
|------------------|-------------------------------------------|-------------|
| Sequential I/O   | Continuous read/write (like file streaming) | Faster ✅    |
| Random I/O       | Jumping to different disk spots             | Slower ⚠️    |

### 🧠 Why it matters:
- **Random I/O is slower**, especially on spinning disks (HDDs).
- PostgreSQL may prefer **Sequential Scan** over **Index Scan** if the index causes too much random I/O.

### 🛠️ Example:
Query:
```sql
SELECT * FROM staff WHERE salary > 70000;
```
- If many rows match, and PostgreSQL must fetch each one from different blocks:
  → Index Scan becomes **Random I/O-heavy** and **less efficient**.

### ✅ Tip:
Use `EXPLAIN ANALYZE` to understand whether the planner is choosing Index Scan or Seq Scan and why.



---

## 📎 Appendix: work_mem — Temporary Memory for Sorting & Hashing

### 🔤 What is `work_mem`?
- It's the amount of memory PostgreSQL uses **per operation** for sorting, hashing, and aggregation **in RAM**.
- If the operation needs more memory → it spills to disk → becomes much slower.

### 📈 When it’s used:
- `ORDER BY`, `DISTINCT`, `GROUP BY`
- Hash joins and hash aggregates

### 🧠 Example:
```sql
SET work_mem = '64MB';  -- Temporary for the current session
```

### ⚠️ Tip:
- Don't set it too high globally — it's per-operation per-user!
- Use higher value temporarily for complex queries.

---

## 🧹 Appendix: VACUUM — Cleaning Up PostgreSQL Tables

### 🔤 What is `VACUUM`?
- PostgreSQL uses **MVCC** (Multi-Version Concurrency Control).
- When rows are updated or deleted, old versions remain → table grows.
- `VACUUM` reclaims that wasted space.

### 🛠️ Types:
- `VACUUM`: cleans dead tuples.
- `VACUUM FULL`: fully compacts the table but locks it.
- `ANALYZE`: updates table statistics for the planner.

### 🧠 Best Practice:
- Autovacuum is usually enough.
- Run `VACUUM ANALYZE` after bulk inserts or deletes.

---

## 🔍 Appendix: CTE (WITH) Performance in PostgreSQL

### 🔤 What is a CTE?
- A **Common Table Expression** is a temporary result set defined using `WITH`.

### ⚠️ Performance Warning:
- By default, PostgreSQL **materializes** CTEs:
  - It computes them **once**, stores in memory/disk, then reuses them.
  - Even if they are simple, the planner treats them as a black box.

### 🧠 Example:
```sql
WITH top_sellers AS (
  SELECT seller_id, COUNT(*) AS total
  FROM sales
  GROUP BY seller_id
)
SELECT * FROM top_sellers WHERE total > 100;
```

### ✅ Tip:
- From PostgreSQL 12+, use `WITH ... AS MATERIALIZED / NOT MATERIALIZED`
- Inline simple CTEs if possible or rewrite as subqueries.



---

## 🔁 Appendix: autovacuum — Automatic Table Maintenance

### 🔤 What is autovacuum?
- PostgreSQL's built-in background process that automatically runs `VACUUM` and `ANALYZE`.
- It keeps tables healthy and planner stats updated without manual work.

### 🔧 Key Parameters (in postgresql.conf):
- `autovacuum_vacuum_threshold` – min row changes before vacuum
- `autovacuum_vacuum_scale_factor` – proportion of table changes to trigger vacuum
- `autovacuum_naptime` – how often to check for tables needing attention

### 🧠 Tips:
- Don’t disable it unless you have a scheduled manual vacuum plan.
- Use `pg_stat_user_tables` to monitor how often autovacuum runs.

---

## 🧵 Appendix: Parallel Query Execution

### 🔤 What is parallel execution?
- PostgreSQL can split parts of a query (e.g., scans, joins, aggregates) to multiple CPU cores.

### 🧠 When it works:
- Large tables and costly operations like `Seq Scan`, `Hash Join`, or `Aggregate`.
- Only certain operations are parallel-safe.

### 🔧 Parameters to enable/tune:
- `max_parallel_workers_per_gather`
- `parallel_setup_cost`
- `parallel_tuple_cost`

### 🛠️ Example:
```sql
SET max_parallel_workers_per_gather = 4;
EXPLAIN ANALYZE SELECT COUNT(*) FROM big_table;
```

---

## 🧾 Appendix: Temporary Tables vs CTEs vs Subqueries

| Feature             | Temporary Table         | CTE (`WITH`)            | Subquery               |
|---------------------|-------------------------|-------------------------|------------------------|
| Visibility          | Global in session       | Only inside query       | Only inside query      |
| Performance         | Materialized            | Materialized by default | Inlined if simple      |
| When to Use         | Reused across queries   | For clarity or reuse    | For simple conditions  |

### 🧠 Tips:
- Use temporary tables when intermediate results are reused in multiple queries.
- Prefer subqueries for lightweight, inline filtering.
- Avoid deep nesting of CTEs unless needed.

---

## 🧩 Appendix: plpgsql Performance Tips

### 🔤 What is PL/pgSQL?
- PostgreSQL’s procedural language, used to write functions, triggers, and logic.

### 🧠 Tips for performance:
- Use `RETURN QUERY` instead of looping over SELECTs.
- Avoid repeated queries in loops; cache results in variables.
- Always declare **strict typing** for parameters and returns.

### 🛠️ Example:
```sql
CREATE OR REPLACE FUNCTION top_sales()
RETURNS TABLE(customer_id INT, total NUMERIC) AS $$
BEGIN
  RETURN QUERY
  SELECT customer_id, SUM(amount)
  FROM orders
  GROUP BY customer_id
  ORDER BY SUM(amount) DESC
  LIMIT 10;
END;
$$ LANGUAGE plpgsql;
```

---

## 🧠 Appendix: General PostgreSQL Tuning Checklist

- 🔍 Always inspect query plans using `EXPLAIN ANALYZE`
- 🧠 Use proper data types (don’t use `TEXT` for everything)
- 🗂️ Normalize where appropriate, denormalize for reads
- 🪪 Avoid over-indexing — it slows down writes
- 🚀 Tune memory settings: `work_mem`, `shared_buffers`, `effective_cache_size`
- 🧹 Let autovacuum work, monitor with `pg_stat_user_tables`
- 💥 Batch inserts/updates when possible
- 📊 Use `pg_stat_statements` to track slow queries
