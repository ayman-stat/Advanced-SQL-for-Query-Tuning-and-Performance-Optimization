
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
