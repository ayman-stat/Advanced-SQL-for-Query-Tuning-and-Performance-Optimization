# Advanced SQL for Query Tuning and Performance Optimization

PostgreSQL-focused learning repository covering query execution, indexing, joins, partitioning, statistics, materialized views, and performance tuning concepts.

## Status

Learning and reference repository based on advanced SQL performance optimization practice. This repo is intended to show database performance awareness for analytics engineering, BI, and data science workflows.

## Topics Covered

- SQL execution plans
- `EXPLAIN` and `EXPLAIN ANALYZE`
- Sequential scans, index scans, and bitmap scans
- B-tree, hash, GIN, GiST, and BRIN indexes
- Join strategy tuning
- Partitioning approaches
- Materialized views
- PostgreSQL statistics and planner estimates
- Autovacuum, `VACUUM`, and `ANALYZE`
- Query memory considerations such as `work_mem`
- Temporary tables, CTEs, and subqueries

## Why This Matters

Strong SQL performance skills help data professionals build faster dashboards, more reliable pipelines, and more scalable analytical systems. For business intelligence and machine learning work, query tuning can directly improve reporting speed, model dataset preparation, and stakeholder experience.

## Practical Checklist

- Use `EXPLAIN ANALYZE` for slow queries.
- Check whether filters and joins use indexes effectively.
- Avoid unnecessary `SELECT *` in production analytics.
- Use the right data types for join keys and filters.
- Monitor table statistics and refresh them when needed.
- Consider materialized views for repeated heavy dashboard queries.
- Avoid over-indexing tables with frequent writes.

## Recommended Next Improvements

- Add reproducible SQL scripts under `sql/`.
- Add a sample schema and synthetic dataset.
- Add before/after query plans for tuned examples.
- Add screenshots of execution plans or benchmark results.

## Professional Relevance

This repository supports my positioning in analytics engineering, BI, data engineering, and senior data science roles where SQL performance and scalable data access are critical.
