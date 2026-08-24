# 🗄️ Modern SQL & Query Optimization Cheatsheet

A concise reference guide covering SQL queries, Joins, Aggregations, Window Functions, CTEs, Transactions, and Indexing optimizations.

---

## 🔍 Data Query Language (DQL) Basics

```sql
-- Retrieve distinct values with filtering and pagination
SELECT DISTINCT category, price
FROM products
WHERE price BETWEEN 10.00 AND 100.00
  AND status = 'ACTIVE'
  AND title LIKE '%laptop%'
ORDER BY price DESC
LIMIT 20 OFFSET 0;

-- Pattern matching with Wildcards & IN clause
SELECT * FROM users
WHERE role IN ('ADMIN', 'MANAGER')
  AND email LIKE '%@gmail.com'
  AND phone IS NOT NULL;
```

---

## 🔗 SQL Joins Reference

```
  INNER JOIN          LEFT JOIN          RIGHT JOIN        FULL OUTER JOIN
 [A  ( A∩B )  B]   [ A  ( A∩B )   ]   [   ( A∩B )  B]   [ A  ( A∩B )  B]
```

```sql
-- INNER JOIN: Only matching rows from both tables
SELECT e.id, e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN: All rows from left table, matching rows from right (NULL if no match)
SELECT u.name, o.order_id, o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: All rows from both tables
SELECT p.product_name, c.category_name
FROM products p
FULL OUTER JOIN categories c ON p.category_id = c.id;

-- SELF JOIN: Join table to itself (e.g. Employee -> Manager hierarchy)
SELECT emp.name AS employee_name, mgr.name AS manager_name
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.id;
```

---

## 📊 Aggregations & Grouping

```sql
-- Calculate summary metrics per category with HAVING filter
SELECT 
    category_id,
    COUNT(id) AS total_products,
    ROUND(AVG(price), 2) AS avg_price,
    MAX(price) AS highest_price,
    SUM(stock) AS total_inventory
FROM products
WHERE status = 'ACTIVE'
GROUP BY category_id
HAVING COUNT(id) > 5 AND AVG(price) >= 50.00
ORDER BY total_inventory DESC;
```

---

## 🪟 Window Functions

Window functions operate over a set of rows specified by `OVER()` clause without collapsing rows into a single summary.

```sql
-- Rank products by price within each category
SELECT 
    id,
    name,
    category_id,
    price,
    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS row_num,
    RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS dense_rank
FROM products;

-- Calculate running total and prior row value (LAG / LEAD)
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY order_date) AS running_total,
    LAG(daily_sales, 1) OVER (ORDER BY order_date) AS previous_day_sales,
    LEAD(daily_sales, 1) OVER (ORDER BY order_date) AS next_day_sales
FROM daily_revenue;
```

---

## 📑 Common Table Expressions (CTEs)

```sql
-- Readability CTE for multi-step analytics
WITH HighValueCustomers AS (
    SELECT user_id, SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY user_id
    HAVING SUM(total_amount) > 1000.00
),
ActiveUsers AS (
    SELECT id, name, email
    FROM users
    WHERE status = 'ACTIVE'
)
SELECT u.name, u.email, hvc.total_spent
FROM ActiveUsers u
JOIN HighValueCustomers hvc ON u.id = hvc.user_id
ORDER BY hvc.total_spent DESC;
```

---

## 🛠️ Data Definition & Manipulation (DDL & DML)

```sql
-- Table creation with constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Insert multiple records
INSERT INTO users (username, email) 
VALUES 
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com')
ON CONFLICT (username) DO UPDATE 
SET email = EXCLUDED.email;

-- Conditional Update
UPDATE orders
SET status = 'SHIPPED', updated_at = NOW()
WHERE status = 'PROCESSING' AND ship_date <= CURRENT_DATE;

-- Delete rows safely
DELETE FROM audit_logs
WHERE log_date < NOW() - INTERVAL '90 days';
```

---

## ⚡ Indexing & Query Optimization

```sql
-- Create Single-Column Index
CREATE INDEX idx_users_email ON users(email);

-- Create Composite Index (order matters: Most Selective First)
CREATE INDEX idx_orders_status_date ON orders(status, order_date);

-- Inspect query execution plan
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE status = 'COMPLETED' AND order_date >= '2026-01-01';
```

---

## 🔒 Transactions & ACID Guarantees

```sql
BEGIN TRANSACTION;

UPDATE accounts 
SET balance = balance - 250.00 
WHERE account_id = 101 AND balance >= 250.00;

UPDATE accounts 
SET balance = balance + 250.00 
WHERE account_id = 202;

-- Check and commit
COMMIT;

-- Rollback in case of error:
-- ROLLBACK;
```
