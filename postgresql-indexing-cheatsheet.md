# 🐘 PostgreSQL Indexing & Query Optimization Cheatsheet

A production reference guide for PostgreSQL index types, query execution plans (`EXPLAIN ANALYZE`), index scan strategies, maintenance operations, and performance tuning.

---

## 🔍 PostgreSQL Index Types & Selection Guide

| Index Type | Structure | Best Use Cases | Example Operators |
|---|---|---|---|
| **B-Tree** | Self-balancing search tree | Equality, range queries, sorting | `=`, `<`, `<=`, `>`, `>=`, `BETWEEN`, `IN` |
| **GIN** | Generalized Inverted Index | Arrays, JSONB documents, Full-Text search | `@>`, `?`, `?|`, `@@` |
| **GiST** | Generalized Search Tree | Geometric shapes, range types, nearest neighbor | `&&`, `@>`, `<@`, `<->` |
| **BRIN** | Block Range Index | Large sequential data (time-series, logs) | Range queries on correlated columns |
| **Hash** | Bucket-based hash lookup | Exact equality matches only (smaller size than B-Tree) | `=` |

---

## 🛠️ Index Creation & Optimization Syntax

### B-Tree & Multi-Column Composite Indexes

```sql
-- Single-column B-Tree index
CREATE INDEX idx_users_email ON users(email);

-- Composite Index (Leftmost Prefix Rule applies!)
-- Useful for queries filtering on (tenant_id) OR (tenant_id AND status)
CREATE INDEX idx_orders_tenant_status ON orders(tenant_id, status, created_at DESC);
```

### Partial & Expression Indexes

```sql
-- Partial Index: Indexes only active users, keeping index size tiny
CREATE INDEX idx_active_users ON users(email) WHERE status = 'ACTIVE';

-- Expression Index: Case-insensitive email lookup
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

### JSONB & GIN Indexes

```sql
-- GIN Index on JSONB document
CREATE INDEX idx_metadata_gin ON audit_logs USING GIN(metadata);

-- Query utilizing JSONB containment operator (@>)
SELECT * FROM audit_logs WHERE metadata @> '{"action": "LOGIN", "status": "FAILED"}';
```

### BRIN Index (Time-Series & Big Data)

```sql
-- BRIN index consumes < 1% space of B-Tree for sorted time-series tables
CREATE INDEX idx_telemetry_timestamp ON sensor_readings USING BRIN(recorded_at);
```

---

## 📊 Analyzing Query Execution Plans (`EXPLAIN ANALYZE`)

```sql
-- Comprehensive execution plan analysis
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, FORMAT TEXT)
SELECT u.id, u.name, COUNT(o.id) AS total_orders
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at >= '2026-01-01'
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 5;
```

### Key Plan Indicators & Terminology

- **Seq Scan**: Full table scan (missing index or high table selectivity ratio).
- **Index Scan**: Traverses index and fetches matching table rows from heap.
- **Index Only Scan**: All requested columns exist in the index (no heap fetch required!).
- **Bitmap Index Scan / Heap Scan**: Combines multiple indexes or handles scattered row matches.
- **Shared Hit / Read Buffers**: Indicates memory cache hits vs disk reads.

---

## ⚡ Index Maintenance & Zero-Downtime Operations

```sql
-- Non-blocking index creation on live production tables
CREATE INDEX CONCURRENTLY idx_transactions_user_id ON transactions(user_id);

-- Rebuild bloated index without holding exclusive table locks
REINDEX INDEX CONCURRENTLY idx_transactions_user_id;

-- Check unused indexes causing write overhead
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND idx_unique = FALSE
ORDER BY pg_relation_size(indexrelid) DESC;
```
