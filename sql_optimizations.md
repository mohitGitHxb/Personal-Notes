# SQL Query Optimization Guide 🚀

_A collection of best practices, techniques, and examples to supercharge your SQL queries!_

---

## 📊 **1. Indexing Strategies**
- **Use Indexes Wisely**  
  Create indexes on columns in `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY` clauses.
  - 🎯 **Composite Indexes**: Combine columns ordered by selectivity (most unique first).
  - 🚫 **Avoid Over-Indexing**: Too many indexes slow down writes.
- **Covering Indexes**  
  Include all columns needed in a query to avoid table scans.
- **Monitor Unused Indexes**  
  Remove redundant indexes to reduce overhead.

---

## ✨ **2. Efficient Query Design**
- **Avoid `SELECT *`**  
  Fetch only necessary columns to reduce I/O and network load.
- **Use Explicit Joins**  
  Replace comma-separated joins with `INNER JOIN`/`LEFT JOIN` for clarity.
- **Filter Early**  
  - Apply `WHERE` before `JOIN` or `GROUP BY` to shrink datasets.
  - Use `EXISTS()` instead of `IN` for subqueries (stops at the first match).
- **Limit Results**  
  Use `LIMIT`, `TOP`, or `FETCH FIRST` to restrict rows returned.

---

## 🔗 **3. Optimize Joins & Subqueries**
- **Join on Indexed Columns**  
  Ensure joined columns are indexed for faster lookups.
- **Avoid Redundant Joins**  
  Remove unnecessary tables from queries.
- **Replace Subqueries with Joins**  
  Use `JOIN` instead of correlated subqueries where possible.

---

## 📈 **4. Aggregation & Sorting**
- **Pre-Filter with `WHERE`**  
  Use `WHERE` instead of `HAVING` to filter rows before aggregation.
- **Avoid Sorting Overhead**  
  Minimize `ORDER BY` unless required. Use indexes for efficient sorting.

---

## ⚠️ **5. Avoid Costly Operations**
- **Beware of Functions on Indexed Columns**  
  ❌ `WHERE YEAR(date) = 2023` → ✅ `WHERE date BETWEEN '2023-01-01' AND '2023-12-31'`.
- **Prefer `UNION ALL` Over `UNION`**  
  Skip duplicate checks if duplicates are acceptable.
- **Minimize Triggers & Cursors**  
  Use set-based operations instead of row-by-row processing.

---

## 🏗️ **6. Data Structure Optimization**
- **Normalize vs. Denormalize**  
  Balance redundancy reduction (normalized) with read speed (denormalized).
- **Partition Large Tables**  
  Split tables by date ranges or key values to limit scans.
- **Use Appropriate Data Types**  
  Smaller types (e.g., `INT` vs `BIGINT`) save space and improve speed.

---

## 🔍 **7. Execution Plan Analysis**
- **Use `EXPLAIN`**  
  Analyze execution plans to spot full scans, missing indexes, or inefficient joins.
- **Update Statistics**  
  Ensure the optimizer has accurate data distribution info.

---

## ⏳ **8. Transactions & Concurrency**
- **Keep Transactions Short**  
  Minimize locks and resource contention.
- **Batch Operations**  
  Use bulk inserts/updates instead of row-by-row operations.

---

## 💾 **9. Caching & Materialization**
- **Materialized Views**  
  Precompute complex aggregations for frequent queries.
- **Application-Level Caching**  
  Use tools like Redis to cache frequent results.

---

## 🔒 **10. Security & Maintenance**
- **Parameterize Queries**  
  Prevent SQL injection and enable plan reuse.
- **Regular Maintenance**  
  Rebuild indexes, update statistics, and archive old data.

---

## 🛠️ **11. Database-Specific Features**
- **Leverage Advanced Features**  
  Use columnstore indexes, in-memory tables, or query hints (e.g., `NOLOCK` in SQL Server).
- **CTEs & Temp Tables**  
  Use Common Table Expressions (CTEs) or temp tables for complex subqueries.

---

## 🌟 **12. Pro Tips**
- **Use Window Functions**  
  Efficient for calculations over row sets (e.g., `ROW_NUMBER()`, `RANK()`).
- **Avoid NULLs**  
  Use `NOT NULL` constraints where possible to simplify queries and indexes.
- **Test & Benchmark**  
  Compare performance before/after optimizations.
- **Keyset Pagination**  
  Use `WHERE id > X` instead of `OFFSET` for large datasets.

---

# SQL Query Optimization Examples 💡

Practical examples of common SQL anti-patterns and their optimized counterparts.

---

### **1. Avoid Functions on Indexed Columns**  
**Bad** (Index scan due to function):  
```sql
SELECT * FROM orders WHERE YEAR(order_date) = 2023;
```  
**Good** (Index-friendly range check):  
```sql
SELECT id, total FROM orders 
WHERE order_date >= '2023-01-01' AND order_date < '2024-01-01';
```

---

### **2. Replace `IN` with `EXISTS`**  
**Bad** (Slow subquery):  
```sql
SELECT name FROM users 
WHERE id IN (SELECT user_id FROM orders);
```  
**Good** (Faster with `EXISTS`):  
```sql
SELECT name FROM users u 
WHERE EXISTS (SELECT 1 FROM orders WHERE user_id = u.id);
```

---

### **3. Use Joins Instead of Subqueries**  
**Bad** (Subquery inefficiency):  
```sql
SELECT * FROM products 
WHERE category_id IN (SELECT id FROM categories WHERE name = 'Electronics');
```  
**Good** (Optimized with `JOIN`):  
```sql
SELECT p.* FROM products p 
JOIN categories c ON p.category_id = c.id 
WHERE c.name = 'Electronics';
```

---

### **4. Optimize Aggregation**  
**Bad** (Unnecessary function in `WHERE`):  
```sql
SELECT COUNT(*) FROM orders 
WHERE status = 'shipped' AND YEAR(order_date) = 2023;
```  
**Good** (Range-based filtering):  
```sql
SELECT COUNT(*) FROM orders 
WHERE status = 'shipped' 
AND order_date >= '2023-01-01' AND order_date < '2024-01-01';
```

---

### **5. Avoid `SELECT DISTINCT`**  
**Bad** (Expensive distinct):  
```sql
SELECT DISTINCT user_id FROM orders;
```  
**Good** (Faster with `GROUP BY`):  
```sql
SELECT user_id FROM orders GROUP BY user_id;
```

---

### **6. Efficient Pagination**  
**Bad** (Slow `OFFSET`):  
```sql
SELECT * FROM products 
ORDER BY id OFFSET 100 ROWS FETCH NEXT 10 ROWS ONLY;
```  
**Good** (Keyset pagination):  
```sql
SELECT * FROM products 
WHERE id > 100 ORDER BY id LIMIT 10;
```

---

### **7. Use `UNION ALL` Instead of `UNION`**  
**Bad** (Unnecessary deduplication):  
```sql
SELECT id FROM orders_2023 
UNION 
SELECT id FROM orders_2024;
```  
**Good** (Skip deduplication):  
```sql
SELECT id FROM orders_2023 
UNION ALL 
SELECT id FROM orders_2024;
```

---

### **8. Avoid Cursors with Set-Based Operations**  
**Bad** (Row-by-row processing):  
```sql
DECLARE @id INT;
DECLARE cursor CURSOR FOR SELECT id FROM users;
OPEN cursor;
FETCH NEXT FROM cursor INTO @id;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE orders SET status = 'processed' WHERE user_id = @id;
    FETCH NEXT FROM cursor INTO @id;
END;
CLOSE cursor;
DEALLOCATE cursor;
```  
**Good** (Bulk update):  
```sql
UPDATE orders SET status = 'processed' 
WHERE user_id IN (SELECT id FROM users);
```

---

### **9. Use Window Functions**  
**Bad** (Subquery for ranking):  
```sql
SELECT name, 
    (SELECT COUNT(*) FROM employees e2 
     WHERE e2.salary >= e1.salary) AS rank
FROM employees e1;
```  
**Good** (Efficient window function):  
```sql
SELECT name, 
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

---

### **10. Batch Inserts**  
**Bad** (Row-by-row insert):  
```sql
INSERT INTO orders (id, total) VALUES (1, 100);
INSERT INTO orders (id, total) VALUES (2, 200);
```  
**Good** (Bulk insert):  
```sql
INSERT INTO orders (id, total) 
VALUES (1, 100), (2, 200);
```

---

### **11. Avoid `LIKE` with Leading Wildcards**  
**Bad** (Full table scan):  
```sql
SELECT * FROM products WHERE name LIKE '%phone%';
```  
**Good** (Use full-text search or indexed columns):  
```sql
SELECT * FROM products 
WHERE name LIKE 'phone%'; -- Prefix search (uses index)
```

---

### **12. Use `CASE` for Conditional Aggregation**  
**Bad** (Multiple queries):  
```sql
SELECT COUNT(*) FROM orders WHERE status = 'shipped';
SELECT COUNT(*) FROM orders WHERE status = 'pending';
```  
**Good** (Single query):  
```sql
SELECT 
    COUNT(CASE WHEN status = 'shipped' THEN 1 END) AS shipped_count,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending_count
FROM orders;
```

---

**Happy Optimizing!** 🚀  
