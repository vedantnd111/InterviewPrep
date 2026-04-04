## Difference between WHERE and HAVING
- where is done before group by and having is done after group by.
## Types of JOINs with real use cases
- **INNER JOIN**: Returns only the matching rows from both tables. 
  - *Use Case*: Get a list of customers who have actually placed an order.
- **LEFT JOIN**: Returns all rows from the left table, and the matched rows from the right table. (NULL if no match).
  - *Use Case*: Show all registered users over the last month and any orders they placed (even if they ordered nothing).
- **RIGHT JOIN**: Returns all rows from the right table, and the matched rows from the left table.
  - *Use Case*: Show all products in our catalog, and the users who bought them (even if a product has never been bought).
- **FULL OUTER JOIN**: Returns all rows when there is a match in either the left or right table.
  - *Use Case*: Find missing data between two systems (e.g., all employees and all departments to see which employees have no department, and which departments have no employees).
- **CROSS JOIN**: Returns the Cartesian product (all possible combinations) of rows.
  - *Use Case*: Generating all combinations of "Sizes" and "Colors" for an e-commerce inventory table.

## How to optimize slow-running queries
- **Use Indexes**: Add indexes to columns commonly used in `WHERE`, `JOIN`, and `ORDER BY` clauses.
- **Avoid SELECT ***: Only fetch columns you actually need. `SELECT id, name` is faster and uses less memory/network than `SELECT *`.
- **Analyze Query Execution Plan**: Use `EXPLAIN` or `EXPLAIN ANALYZE` before your query to find bottlenecks (like full table scans).
- **Filter Early**: Make your `WHERE` clauses as strict as possible early on to reduce the dataset size before joining or grouping.
- **Avoid Functions on Indexed Columns**: e.g., `WHERE YEAR(date_column) = 2023` prevents the database from using an index on `date_column`. Instead, use `WHERE date_column >= '2023-01-01' AND date_column <= '2023-12-31'`.

## GROUP BY vs DISTINCT
- **DISTINCT**: Used to remove duplicate rows from the final result set.
  - *Example*: `SELECT DISTINCT country FROM Users;` (Simply lists all unique countries).
- **GROUP BY**: Used with aggregate functions (like `SUM`, `COUNT`, `AVG`) to group rows that have the same values into summary rows.
  - *Example*: `SELECT country, COUNT(*) FROM Users GROUP BY country;` (Tells you *how many* users are from each country).

## Indexing and when to use it
- **What it is**: An index is like a book's table of contents. Instead of scanning every single page (Full Table Scan) to find a word, the database looks at the index to instantly find the row's location.
- **When to use it**: 
  - On Primary Keys and Foreign Keys.
  - On columns frequently used in `WHERE` clauses (e.g., searching by `email` or `username`).
  - On columns used frequently in sorting (`ORDER BY`).
- **When NOT to use it**: 
  - Indexes slow down `INSERT`, `UPDATE`, and `DELETE` queries because the index must also be updated.
  - Avoid them on small tables, or columns that have very few unique values (like a `gender` or `is_active` boolean column).

## Window functions (ROW_NUMBER, RANK, DENSE_RANK)
These functions allow you to calculate rankings or running totals without collapsing the rows (unlike `GROUP BY`).
*Example Scenario*: Ranking students based on their exam scores (90, 90, 85, 80).
- **ROW_NUMBER()**: Gives a unique sequential number to each row `(1, 2, 3, 4)`.
- **RANK()**: Gives the same rank to ties, but skips the next numbers. `(1, 1, 3, 4)`. Notice how rank `2` is skipped!
- **DENSE_RANK()**: Gives the same rank to ties, but does *not* skip numbers. `(1, 1, 2, 3)`.

**Query Demonstration**:
```sql
SELECT 
    student_name, 
    exam_score,
    -- Strictly gives every row a unique number (1, 2, 3, 4)
    ROW_NUMBER() OVER (ORDER BY exam_score DESC) as row_num,
    
    -- Ties get the same rank, but skips the next number (1, 1, 3)
    RANK() OVER (ORDER BY exam_score DESC) as rank_val,
    
    -- Ties get the same rank, firmly does NOT skip numbers (1, 1, 2)
    DENSE_RANK() OVER (ORDER BY exam_score DESC) as dense_rank_val
FROM students;
```

**Time Complexity and Indexing Impacts**:
- **Computational Time Complexity**: Running a window function generally results in an **$O(N \log N)$** time cost. This is because the `OVER (ORDER BY ...)` clause brutally forces the database engine to perform a full internal sorting algorithm on the dataset *before* it can physically assign the ranks.
- **How Indexes drop the complexity**: If you already have a persistent physical **Index** perfectly matching the exact column you are sorting by (e.g., an index physically placed on `exam_score`), the database brilliantly **skips** the $O(N \log N)$ sorting step entirely! It just linearly reads the data straight off the pre-sorted B-Tree index, instantly dropping the window function's computational overhead down to **$O(N)$**.
- **Structural Effect**: Window functions strictly operate in RAM on the final result set. They have absolutely zero structural effect on the underlying tables or existing indexes, and the database does not permanently save/index the dynamically calculated ranks.

## Subqueries vs Joins
- **Subquery**: A query nested inside another query. 
  - *Pros*: Very easy to read and intuitive to write.
  - *Cons*: Often slower because the inner query might be executed repeatedly for *every* row of the outer query (correlated subquery).
- **Join**: Combines rows from two or more tables based on a related column.
  - *Pros*: Generally much faster and optimized by the database engine execution planner.
- *Rule of Thumb*: Use `JOIN`s for combining tables and best performance. Use Subqueries when you need to calculate a standalone aggregated value (like an `AVG`) to compare against.

## ACID properties and transaction handling
ACID ensures that database transactions (like a bank transfer) are processed reliably.
- **Atomicity (All or Nothing)**: A transaction must completely succeed or completely fail. If checking out an order fails at the payment step, the items shouldn't be decremented from inventory.
- **Consistency**: The database must go from one valid state to another without breaking constraints. (e.g., a bank balance cannot drop below zero if there is a constraint against it).
- **Isolation**: Concurrent transactions don't interfere with each other. If Person A and Person B withdraw money simultaneously, the database locks the row to handle them one at a time.
- **Durability**: Once a transaction is committed, its changes are saved permanently, even if the server crashes right after.

## SQL Query Optimization
If an interviewer asks how you optimize slow queries, you can bring up these key strategies:

### 1. Indexed WHERE/JOIN fields
- **What it is**: An index works exactly like an index at the back of a textbook. It prevents the database from reading every single row (a Full Table Scan).
- **How to use it**: Add indexes to columns heavily used in `WHERE`, `JOIN`, or `ORDER BY` clauses.
- **Trade-off**: Indexes speed up `SELECT` statements significantly, but they slow down `INSERT`/`UPDATE` operations because the index must also be updated.

### 2. Removed SELECT *
- **What it is**: Specifying the exact columns you need (e.g., `SELECT id, first_name`) instead of fetching everything.
- **Why it matters**: `SELECT *` forces the database to read unnecessary data off the disk and consumes much more memory and network bandwidth.

### 3. Used EXPLAIN
- **What it is**: By appending `EXPLAIN` (or `EXPLAIN ANALYZE`) before your query, the database spits out its internal "Execution Plan" instead of actually running the query.
- **Why it matters**: It tells you exactly *how* it plans to fetch the data. You can instantly see if it is using your indexes or getting stuck doing a massive full table scan.

### 4. Discussed denormalization & slow log analysis
- **Denormalization**: While Normalization prevents duplicate data, it sometimes leads to dozens of complex, slow `JOIN`s. Denormalization means purposely storing some duplicate data directly on a table to avoid joining, trading storage space for a massive boost in read-speed.
- **Slow Query Log Analysis**: Databases have a "Slow Query Log" setting that records any queries taking longer than a certain limit (e.g., 2 seconds). Periodically analyzing this log is the best way to find and fix the exact bottlenecks affecting real users in production.

## What is database indexing and how does it improve query performance?
- **What it is**: An index is a separate data structure (usually a B-Tree) created strictly to speed up searches. Without an index, the database must blindly scan the table row-by-row from top to bottom (a computationally heavy "**Full Table Scan**"). 
- **The Logic (How it works)**: With an index, the database physically re-arranges the indexed column into a sorted B-Tree. When you search, the B-Tree allows `O(log n)` time complexity lookups instead of `O(n)`. It's exactly like the index at the back of a textbook: you don't read every single page to find a topic; you check the index, which gives you the exact page number to jump to, massively dropping heavy I/O operations from the disk.

## What is a composite index and when should it be used?
- **What it is**: A composite index is a single index placed on *multiple* columns simultaneously (e.g., an index structurally grouped by `(last_name, first_name)`).
- **When to use it**: When your queries frequently filter on those specific multiple columns together (e.g., `WHERE last_name = 'Smith' AND first_name = 'John'`).
- **The Logic ("Leftmost Prefix Rule")**: A DB evaluates composite columns precisely from left to right. This means the index `(last_name, first_name)` is totally useless if you only search by `first_name`! The database can only utilize the index if you query the left-most foundational column (`last_name` alone, or `last_name AND first_name`).

## What is the difference between optimistic locking and pessimistic locking?
- **Optimistic Locking**: Assumes conflicts are highly unlikely. Instead of physically locking database rows, it uses a `version` column. 
  - *Logic*: When updating a row, the app checks if the `version` hasn't changed since it originally read it (e.g., `UPDATE users SET name='X', version=3 WHERE id=1 AND version=2`). If the version changed in the meantime (because another thread modified it), the update safely fails, and the app retries. This completely avoids database deadlocks and scales perfectly for read-heavy apps.
- **Pessimistic Locking**: Assumes conflicts will happen constantly. It physically locks the table row upfront (`SELECT ... FOR UPDATE`).
  - *Logic*: While one thread is updating the row, EVERY other thread trying to read or write to that specific row is frozen in place waiting in line. Use only when absolute data integrity is necessary and conflicts are extremely likely (like concert ticket bookings at absolute maximum capacity).

## How do you handle database transactions across microservices?
You cannot reliably use standard ACID database transactions (like Spring's `@Transactional`) across a network between multiple isolated microservice databases, as network fallibility (latency, dropped packets) destroys basic ACID guarantees.
Instead, you must use **distributed patterns**:
1. **Two-Phase Commit (2PC)**: A centralized coordinator halts all databases, prepares them, and forces a synchronous universal commit. 
   - *Logic*: Highly frowned upon today. It causes massively severe performance bottlenecks because it strictly locks multiple databases synchronously over the network. If the network stutters, all databases freeze.
2. **The Saga Pattern**: The modern industry standard. You break the massive transaction into a sequence of isolated, local micro-transactions.
   - *Logic*: Microservice A updates its DB and triggers a message broker event. Microservice B receives the event and updates its DB. If Microservice B functionally fails, it explicitly fires a mathematically reversed "**Compensating Transaction**" back to Microservice A to manually undo the previous change. Instead of a massive database lock, you use code-level business logic to quickly achieve **Eventual Consistency**.

