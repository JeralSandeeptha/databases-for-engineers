# Database Indexing

## What is indexing

`Database indexing` is a data structure technique used to speed up data retrieval operations in a database

Just like an index in the back of a book, it allows the database engine to find specific data quickly without having to scan every single row in a table

By default dbms has `O(N)` time complexity because it needs to go through every records and ervry page

But with indexing behind the scenes it uses `B Trees` and `B+Trees`. So it reduces into `O(log N)`

<br/>

## Indexing types

clustered indexing
non-clustered indexing

<br/>

## Database Scan Types

Database engines choose between scanning **indexes** or scanning the **physical table** based on your query structure, data volume, and existing index configurations. This document compiles standard, hybrid, and advanced scan methods utilized by modern database management systems (DBMS).

---

## Part 1: Standard & Common Scan Types

### 1. Index Only Scan (Covering Index Scan)

The engine reads **only the index** and completely skips touching the physical table.

* **Mechanism:** Every column requested in your `SELECT`, `WHERE`, `JOIN`, and `ORDER BY` clauses already exists inside the index structure itself (known as a covering index).
* **Performance:** Fastest retrieval method. It eliminates table lookups and cuts input/output (I/O) operations roughly in half.
* **Example:**

  ```sql
  -- Assuming a composite index exists on columns (status, user_id)
  SELECT user_id FROM orders WHERE status = 'active';
  ```

### 2. Index Scan (Index Lookup / Seek)

The engine navigates the index tree to pinpoint row locations, then hops to the **physical table** to fetch the remaining data.

* **Mechanism:** It uses the index structure to locate specific entries matching your condition, then performs a "bookmark lookup" or "heap fetch" to grab columns not included in the index.
* **Performance:** Highly efficient for selective queries (retrieving a small percentage of total rows). It slows down drastically if forced to make millions of random-access disk jumps to the table.
* **Example:**

  ```sql
  -- Index exists on (user_id). Engine finds user_id = 42, then fetches "email" from the table.
  SELECT email FROM users WHERE user_id = 42;
  ```

### 3. Bitmap Table Scan

A two-step hybrid approach that uses indexes to build a **memory map**, then reads the table in a clean, sequential path.

* **Mechanism:** 
  1. **Bitmap Index Scan:** The engine checks the index, finds all matching row locations, and marks them as `1`s in a memory bitmap.
  2. **Bitmap Heap Scan:** It sorts those locations by physical disk order and scans the table sequentially, jumping only to the data blocks containing the marks.
* **Performance:** Optimal for medium-sized data sets or when combining multiple independent indexes (using `AND` or `OR`). It transforms costly, random disk jumps into an organized, sequential read.
* **Example:**

  ```sql
  -- Index 1 on (status), Index 2 on (country). The engine merges both bitmaps in memory.
  SELECT * FROM orders WHERE status = 'Pending' AND country = 'US';
  ```

### 4. Full Table Scan (Sequential Scan)
The engine bypasses all indexes and reads **every single row** of the physical table from start to finish.

* **Mechanism:** It reads the table blocks sequentially from disk or memory, evaluating the query's `WHERE` clause conditions against every single record.
* **Performance:** Slowest for large tables. However, it is the *fastest* choice for very small tables (where loading an index takes more effort than reading the table) or when a query requests a massive percentage of the table.
* **Example:**

  ```sql
  -- No index exists on the "notes" column, forcing a full crawl of the table.
  SELECT * FROM users WHERE notes LIKE '%vague_keyword%';
  ```

---

## Part 2: Advanced & Database-Specific Scan Types

### 5. Index Skip Scan (Loose Index Scan / Jump Scan)

The engine traverses a multi-column (composite) index even when the leading column is omitted from the `WHERE` clause.

* **Mechanism:** If an index is built on `(country, user_id)` and a query only filters by `user_id`, the engine "skips" across the distinct values of the first column (`country`) to look up the target entries in the second column (`user_id`).
* **Performance:** Highly efficient if the omitted leading column has very low cardinality (few unique values, e.g., gender, status, country) and the secondary column is highly selective.
* **Database Terminology:** Called **Index Skip Scan** in Oracle, **Loose IndexScan** in MySQL, and **Jump Scan** in DB2.
* **Example:**

  ```sql
  -- Even if the index starts with 'gender', the engine skips through 'M' and 'F' blocks directly to find the ID.
  SELECT * FROM profiles WHERE user_id = 9901; 
  ```

### 6. Parallel Scans (Parallel Seq Scan / Parallel Index Scan)
The database engine splits a heavy scanning workload across multiple CPU execution threads concurrently.

* **Mechanism:** The query planner divides a massive table or index into smaller chunks. A coordinating process assigns background worker threads to scan these chunks simultaneously, aggregating the final results together at the end.
* **Performance:** Drastically minimizes execution times for large analytic queries by maximizing multi-core hardware infrastructure.
* **Database Terminology:** Commonly seen as **Parallel Seq Scan** or **Parallel Index Scan** in PostgreSQL and SQL Server.
* **Example:**

  ```sql
  -- Initiates multiple parallel workers to aggregate millions of historical records.
  SELECT year, SUM(total_amount) FROM massive_sales_archive GROUP BY year;
  ```

### 7. TID Scan (Tuple ID Scan / Rowid Scan)

The database engine fetches rows directly using their precise physical storage coordinates.

* **Mechanism:** It bypasses the logical query execution framework completely by utilizing the internal record pointer tuple identifier—which maps explicitly to a physical data block and offset location on the disk.
* **Performance:** Extremely fast, constant-time $O(1)$ complexity. This is the absolute fastest possible retrieval method, but it is rarely used in standard application logic.
* **Database Terminology:** Known as **TID Scan** in PostgreSQL and **Rowid Scan** in Oracle.
* **Example:**

  ```sql
  -- Explicitly pointing to an internal database physical block identifier
  SELECT * FROM users WHERE ctid = '(0,1)';
  ```

### 8. Clustered Index Scan
The engine searches the physical table sequentially, using the order dictated by its primary key or clustered structure.

* **Mechanism:** In databases where the table *is* structurally sorted by an index (a Clustered Index), reading the entire table under this order behaves similarly to a full table scan, but preserves sorted index properties.
* **Performance:** Identical to a Full Table Scan in terms of reading the entire dataset, but optimal when your output requires an implicit sort on the primary cluster key.
* **Database Terminology:** Primarily found in **SQL Server** and **MySQL (InnoDB)**.
* **Example:**

  ```sql
  -- Scans the entire table structure in exact order of its primary cluster key layout.
  SELECT * FROM employees ORDER BY employee_id;
  ```

---

## Master Comparison Matrix

| Scan Type | Touches Index? | Touches Table? | Primary Advantage / Best For... | Disk I/O Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **Index Only Scan** | Yes | No | Covering queries; minimal physical storage hits | Sequential (Index) |
| **Index Scan** | Yes | Yes (Row) | Low volume / Unique lookups | Random (Table) |
| **Bitmap Table Scan**| Yes | Yes (Block) | Medium volume / Merging multi-index queries | Sorted / Sequential |
| **Full Table Scan** | No | Yes (All) | Tiny tables / Querying a massive % of a table | Sequential (Table) |
| **Index Skip Scan** | Yes | Depends | Queries skipping the prefix column of a composite index | Multi-range Index jumps |
| **Parallel Scan** | Variant | Variant | Processing large datasets faster using multiple CPUs | Multi-threaded reading |
| **TID / Rowid Scan**| No | Yes (Direct)| Instant $O(1)$ data block retrieval | Direct Block Access |
| **Clustered Index Scan**| Yes | Yes (All) | Reading whole table while keeping implicit order | Sequential (Clustered) |
