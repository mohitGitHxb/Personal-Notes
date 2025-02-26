# SQL Pagination: Keyset vs Offset-Limit

Pagination is a critical technique for efficiently handling large datasets in database applications. This document provides an in-depth comparison between traditional offset-limit pagination and keyset pagination (also known as cursor-based pagination), highlighting their implementations, use cases, and performance characteristics.

## Traditional Offset-Limit Pagination

Offset-limit pagination works by skipping a certain number of rows (the offset) and then taking a specific number of rows (the limit).

### Syntax

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY sort_column
LIMIT n OFFSET m;
```

Where:
- `n` is the number of rows to return (page size)
- `m` is the number of rows to skip

### Examples

**Example 1: Basic implementation**

Retrieving the first page (10 rows):

```sql
SELECT id, username, created_at 
FROM users 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 0;
```

Retrieving the second page:

```sql
SELECT id, username, created_at 
FROM users 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 10;
```

**Example 2: With dynamic parameters**

```sql
-- Page number starts at 1
-- Page size is 25 rows
DECLARE @PageNumber INT = 3;
DECLARE @PageSize INT = 25;

SELECT product_id, name, price
FROM products
ORDER BY price DESC
LIMIT @PageSize OFFSET (@PageNumber - 1) * @PageSize;
```

### Advantages

1. **Simple to implement**: Straightforward SQL that is easy to understand and implement
2. **Works with arbitrary ordering**: Can be used with any ORDER BY clause
3. **Direct navigation**: Allows jumping to any page directly
4. **Stateless**: No need to store additional information between requests

### Disadvantages

1. **Performance degradation with large offsets**: The database must scan and discard all rows before the offset, which becomes increasingly inefficient as the offset grows
2. **Inconsistent results with data changes**: If rows are added or removed between page requests, rows might be duplicated or skipped
3. **Full table scan**: Often requires scanning a significant portion of the table
4. **Resource intensive**: High CPU and memory usage for large datasets

## Keyset Pagination

Keyset pagination (also called cursor-based or seek-method pagination) uses the values of the sorted column(s) from the last row of the current page to fetch the next page.

### Syntax

```sql
-- First page
SELECT column1, column2, ...
FROM table_name
ORDER BY sort_column
LIMIT n;

-- Subsequent pages
SELECT column1, column2, ...
FROM table_name
WHERE sort_column < last_row_sort_value  -- For DESC order
ORDER BY sort_column DESC
LIMIT n;

-- Or for ascending order
SELECT column1, column2, ...
FROM table_name
WHERE sort_column > last_row_sort_value  -- For ASC order
ORDER BY sort_column ASC
LIMIT n;
```

### Examples

**Example 1: Simple keyset with a timestamp**

First page:

```sql
SELECT id, username, created_at 
FROM users 
ORDER BY created_at DESC 
LIMIT 10;
```

Subsequent page (assuming last row had created_at = '2023-06-15 14:30:00'):

```sql
SELECT id, username, created_at 
FROM users 
WHERE created_at < '2023-06-15 14:30:00'
ORDER BY created_at DESC 
LIMIT 10;
```

**Example 2: Compound keyset (multiple columns)**

When using multiple columns for sorting, all must be included in the where clause:

First page:

```sql
SELECT id, name, score, updated_at 
FROM leaderboard 
ORDER BY score DESC, updated_at DESC 
LIMIT 20;
```

Next page (assuming last row had score = 95 and updated_at = '2023-07-01 09:15:00'):

```sql
SELECT id, name, score, updated_at 
FROM leaderboard 
WHERE 
    (score < 95) OR 
    (score = 95 AND updated_at < '2023-07-01 09:15:00')
ORDER BY score DESC, updated_at DESC 
LIMIT 20;
```

**Example 3: Handling non-unique values with tie-breaker**

Using a unique identifier (like an ID) as a tie-breaker:

```sql
-- First page
SELECT id, product_name, category, price 
FROM products 
ORDER BY category ASC, id ASC 
LIMIT 15;

-- Next page (last row: category = 'Electronics', id = 342)
SELECT id, product_name, category, price 
FROM products 
WHERE 
    (category > 'Electronics') OR 
    (category = 'Electronics' AND id > 342)
ORDER BY category ASC, id ASC 
LIMIT 15;
```

**Example 4: Bidirectional navigation with encoded cursor**

Typically, the "cursor" (last row values) is base64-encoded in a real application:

```sql
-- Assuming cursor decodes to: {"price": 79.99, "id": 1024}

-- Next page (forward)
SELECT id, name, price
FROM products
WHERE (price < 79.99) OR (price = 79.99 AND id < 1024)
ORDER BY price DESC, id DESC
LIMIT 10;

-- Previous page (backward)
SELECT id, name, price
FROM products
WHERE (price > 79.99) OR (price = 79.99 AND id > 1024)
ORDER BY price DESC, id DESC
LIMIT 10;
```

### Advantages

1. **Consistent performance**: Performance is independent of offset depth, making it O(1) instead of O(n)
2. **Efficient index usage**: Leverages database indexes effectively
3. **Result consistency**: Even if data changes between requests, no rows will be duplicated or skipped
4. **Reduced server load**: Requires far fewer resources for large datasets

### Disadvantages

1. **No random access**: Cannot jump directly to an arbitrary page; must traverse pages sequentially
2. **Implementation complexity**: More complex to implement, especially with compound indexes
3. **Client-side state**: Requires storing the "cursor" between requests
4. **ORDER BY constraints**: Must include all columns used for sorting in the WHERE clause

## Performance Comparison

### Small Datasets (< 10,000 rows)
Both methods perform similarly for small datasets. Offset-limit pagination might be preferable due to its simplicity.

### Medium Datasets (10,000 - 100,000 rows)
Performance differences become noticeable. Keyset pagination provides better performance as page number increases.

### Large Datasets (> 100,000 rows)
Keyset pagination significantly outperforms offset-limit, especially for later pages:

| Page Number | Rows | Offset-Limit | Keyset |
|-------------|------|--------------|--------|
| 1           | 50   | 15 ms        | 10 ms  |
| 10          | 50   | 25 ms        | 10 ms  |
| 100         | 50   | 150 ms       | 10 ms  |
| 1000        | 50   | 1200 ms      | 10 ms  |

*Note: Actual performance varies based on database engine, indexing, and hardware.*

## Implementation Tips

### For Offset-Limit Pagination

1. **Count optimization**: Use approximate count techniques for large tables
   ```sql
   SELECT COUNT(*) FROM users;  -- This can be slow for large tables
   ```

2. **Avoid deep pagination**: Consider limiting maximum page depth

3. **Index optimization**: Ensure proper indexes on sorted columns

### For Keyset Pagination

1. **Ensure unique order**: Always include a unique column as a tie-breaker
   ```sql
   ORDER BY created_at DESC, id DESC
   ```

2. **Cursor encoding/decoding**: Base64 encode cursor values for API responses
   ```javascript
   const cursor = Buffer.from(JSON.stringify({
     created_at: "2023-07-01T10:30:00Z",
     id: 12345
   })).toString('base64');
   ```

3. **Handle NULL values**: Special handling may be needed for nullable columns
   ```sql
   WHERE 
       (column1 IS NULL AND (column2 > value2 OR column2 IS NULL)) OR
       (column1 IS NOT NULL AND column1 > value1)
   ```

4. **Direction control**: Add a "direction" parameter for bidirectional navigation

## Real-World Example: API Implementation

**Offset-Limit API:**
```
GET /api/products?page=3&per_page=20
```

**Keyset API:**
```
GET /api/products?cursor=eyJpZCI6MTIzNCwicHJpY2UiOjQ5Ljk5fQ==&limit=20
```

## Example: Complete Keyset Pagination Function (PostgreSQL)

```sql
CREATE OR REPLACE FUNCTION paginate_products(
    p_limit INT,
    p_cursor TEXT DEFAULT NULL,
    p_direction TEXT DEFAULT 'next'
)
RETURNS TABLE (
    id INT,
    name TEXT,
    price DECIMAL(10,2),
    created_at TIMESTAMP,
    next_cursor TEXT,
    prev_cursor TEXT
) AS $$
DECLARE
    v_id INT;
    v_price DECIMAL(10,2);
    v_where TEXT;
    v_order TEXT;
    v_results RECORD;
BEGIN
    -- Default order direction
    IF p_direction = 'next' THEN
        v_order := 'DESC';
    ELSE
        v_order := 'ASC';
    END IF;
    
    -- Parse cursor if provided
    IF p_cursor IS NOT NULL THEN
        SELECT * FROM jsonb_to_record(convert_from(decode(p_cursor, 'base64'), 'UTF8')::jsonb) 
        AS x(id INT, price DECIMAL(10,2)) INTO v_id, v_price;
        
        -- Build WHERE clause based on direction
        IF p_direction = 'next' THEN
            v_where := format('(price < %L OR (price = %L AND id < %L))', v_price, v_price, v_id);
        ELSE
            v_where := format('(price > %L OR (price = %L AND id > %L))', v_price, v_price, v_id);
        END IF;
    ELSE
        v_where := 'TRUE';
    END IF;
    
    -- Execute the paginated query
    RETURN QUERY EXECUTE format('
        WITH results AS (
            SELECT 
                p.id, 
                p.name, 
                p.price, 
                p.created_at,
                row_number() OVER () as rn
            FROM products p
            WHERE %s
            ORDER BY price %s, id %s
            LIMIT %L
        )
        SELECT 
            r.id, 
            r.name, 
            r.price, 
            r.created_at,
            encode(convert_to(json_build_object(
                ''id'', CASE WHEN count(*) OVER () = %L THEN first_value(r.id) OVER (ORDER BY r.rn DESC) ELSE NULL END,
                ''price'', CASE WHEN count(*) OVER () = %L THEN first_value(r.price) OVER (ORDER BY r.rn DESC) ELSE NULL END
            )::text, ''UTF8''), ''base64'') as next_cursor,
            encode(convert_to(json_build_object(
                ''id'', CASE WHEN count(*) OVER () > 0 THEN first_value(r.id) OVER () ELSE NULL END,
                ''price'', CASE WHEN count(*) OVER () > 0 THEN first_value(r.price) OVER () ELSE NULL END
            )::text, ''UTF8''), ''base64'') as prev_cursor
        FROM results r
    ', v_where, v_order, v_order, p_limit, p_limit, p_limit);
END;
$$ LANGUAGE plpgsql;

-- Usage:
-- First page
SELECT * FROM paginate_products(10);

-- Next page with cursor
SELECT * FROM paginate_products(10, 'eyJpZCI6MzQyLCJwcmljZSI6NDkuOTl9');

-- Previous page
SELECT * FROM paginate_products(10, 'eyJpZCI6MzQyLCJwcmljZSI6NDkuOTl9', 'prev');
```

## When to Use Each Method

### Use Offset-Limit When:
- Building admin interfaces with direct page navigation
- Working with small to medium datasets
- Random access to pages is required
- Simplicity is more important than performance

### Use Keyset Pagination When:
- Building infinite scroll or "load more" interfaces
- Working with large datasets
- Performance is critical
- Consistent results are essential (e.g., real-time data feeds)
- Implementing RESTful or GraphQL APIs

## Database-Specific Implementations

### MySQL

MySQL supports both pagination methods:

```sql
-- Offset-Limit
SELECT * FROM products ORDER BY created_at DESC LIMIT 10 OFFSET 30;

-- Keyset
SELECT * FROM products 
WHERE created_at < '2023-07-01 10:30:00' 
ORDER BY created_at DESC 
LIMIT 10;
```

### PostgreSQL

PostgreSQL has similar syntax but offers additional optimization for keyset pagination:

```sql
-- Using ROWS FETCH syntax (SQL standard)
SELECT * FROM orders 
ORDER BY order_date DESC 
OFFSET 50 ROWS FETCH FIRST 25 ROWS ONLY;

-- Keyset with exclusion operator
SELECT * FROM orders 
WHERE (order_date, id) < ('2023-07-01', 12345)
ORDER BY order_date DESC, id DESC 
LIMIT 25;
```

### SQL Server

SQL Server uses a different syntax for offset-limit:

```sql
-- Offset-Limit
SELECT * FROM customers
ORDER BY last_name
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Keyset
SELECT TOP 10 * FROM customers
WHERE 
    (last_name > 'Smith') OR 
    (last_name = 'Smith' AND first_name > 'John')
ORDER BY last_name, first_name;
```

## Conclusion

Keyset pagination is the recommended approach for most modern applications, especially those dealing with large datasets or requiring real-time data consistency. While traditional offset-limit pagination is simpler to implement, its performance characteristics make it unsuitable for deep pagination scenarios.

For optimal performance:
1. Always include proper indexes on the columns used in the ORDER BY clause
2. Use a unique identifier as a tie-breaker in keyset pagination
3. Consider the specific needs of your application's UI when choosing a pagination method
4. Test both methods with realistic data volumes to evaluate performance differences

By implementing proper pagination techniques, you'll ensure your application remains responsive and efficient even as your data grows.
```

This comprehensive guide should serve as an excellent reference in your GitHub repository. It covers both pagination methods in detail, with multiple examples, performance comparisons, implementation tips, and database-specific information.
