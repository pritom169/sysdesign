## Database Indexing


### What is an Index?

An index is a data structure that improves query speed at the cost of additional storage and write overhead. Without an index, the database performs a full table scan—examining every row to find matches.

```
Without Index (Full Table Scan):
┌─────────────────────────────────────┐
│         Users Table (1M rows)        │
├────┬──────────┬─────────────────────┤
│ id │   name   │        email        │
├────┼──────────┼─────────────────────┤
│ 1  │ Alice    │ alice@mail.com      │ ← Check
│ 2  │ Bob      │ bob@mail.com        │ ← Check
│ 3  │ Charlie  │ charlie@mail.com    │ ← Check
│... │ ...      │ ...                 │ ← Check ALL rows
│ 1M │ Zara     │ zara@mail.com       │ ← Check
└────┴──────────┴─────────────────────┘
Query: SELECT * FROM users WHERE email = 'bob@mail.com'
Time: O(n) — must scan all 1M rows

With Index on email:
┌──────────────────┐      ┌─────────────────┐
│   Email Index    │      │   Users Table   │
│   (B+ Tree)      │      │                 │
├──────────────────┤      ├─────────────────┤
│ alice@mail.com →─┼──────│► Row 1          │
│ bob@mail.com →───┼──────│► Row 2          │
│ charlie@mail... →┼──────│► Row 3          │
└──────────────────┘      └─────────────────┘
Time: O(log n) — direct lookup via tree
```

**Core trade-off:** Faster reads, slower writes. Every INSERT/UPDATE/DELETE must also update the index.

---

### How Indexes Work Internally

Indexes use specialized data structures to enable fast lookups.

#### B+ Tree (Most Common)

The default index structure for most relational databases. Balanced tree where:
- All values stored in leaf nodes
- Leaf nodes linked for range queries
- Internal nodes contain only keys for navigation

```
                    ┌─────────────┐
                    │   [50, 100] │  ← Root (navigation only)
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      ┌─────────┐    ┌──────────┐    ┌───────────┐
      │[10,30,40]│   │[60,70,80]│    │[110,120]  │ ← Internal
      └────┬────┘    └────┬─────┘    └─────┬─────┘
           ▼              ▼                ▼
    ┌──────────────────────────────────────────┐
    │ 10→20→30→40→50→60→70→80→100→110→120      │ ← Leaf nodes
    └──────────────────────────────────────────┘   (linked list)
              ↑ All data here, linked for range scans
```

**Why B+ Tree?**
- Balanced: All leaf nodes at same depth → predictable O(log n)
- High fanout: Few disk reads to reach any value
- Range-friendly: Linked leaves enable efficient `BETWEEN`, `ORDER BY`

**Lookup:** `WHERE id = 70`
1. Start at root: 70 > 50, go right
2. At [60,70,80]: Find 70
3. Follow pointer to data row

**Range query:** `WHERE id BETWEEN 60 AND 100`
1. Find 60 in leaf
2. Scan linked list until 100

---

### Types of Indexes

#### 1. Primary Index (Clustered Index)

Determines physical storage order of table data. Only one per table.

```
Clustered Index on id:
┌─────────────────────────────────────┐
│ Data stored in id order on disk     │
├────┬──────────┬─────────────────────┤
│ 1  │ Alice    │ alice@mail.com      │ ← Physically first
│ 2  │ Bob      │ bob@mail.com        │ ← Physically second
│ 3  │ Charlie  │ charlie@mail.com    │ ← Physically third
└────┴──────────┴─────────────────────┘
```

**Characteristics:**
- Table data IS the index (no separate structure)
- Range queries on clustered key are fast (sequential disk read)
- Inserts in middle require physical reorganization

#### 2. Secondary Index (Non-Clustered Index)

Separate structure pointing to row locations. Multiple allowed per table.

```
Secondary Index on email:
┌──────────────────────┐     ┌─────────────────┐
│     Email Index      │     │  Actual Table   │
├──────────────────────┤     │  (clustered by  │
│ alice@mail.com → ────┼────►│   id on disk)   │
│ bob@mail.com → ──────┼────►│                 │
│ charlie@mail... → ───┼────►│                 │
└──────────────────────┘     └─────────────────┘
          ↑
   Stores pointer (row ID or primary key)
```

**Characteristics:**
- Extra lookup required (index → pointer → data)
- Multiple secondary indexes allowed
- Can become stale, requires maintenance

#### 3. Composite Index (Multi-Column Index)

Index on multiple columns. Order matters.

```sql
CREATE INDEX idx_name_age ON users(last_name, first_name, age);
```

```
Composite Index Structure:
┌────────────────────────────────────┐
│     last_name, first_name, age     │
├────────────────────────────────────┤
│ Adams, Alice, 25 → Row 5           │
│ Adams, Bob, 30 → Row 12            │
│ Baker, Alice, 22 → Row 3           │
│ Baker, Carol, 28 → Row 8           │
└────────────────────────────────────┘
```

**Leftmost prefix rule:** Index used only when query includes leftmost columns.

| Query | Uses Index? |
|-------|-------------|
| `WHERE last_name = 'Adams'` | Yes |
| `WHERE last_name = 'Adams' AND first_name = 'Alice'` | Yes |
| `WHERE last_name = 'Adams' AND first_name = 'Alice' AND age = 25` | Yes |
| `WHERE first_name = 'Alice'` | No (skipped last_name) |
| `WHERE age = 25` | No (skipped last_name, first_name) |
| `WHERE last_name = 'Adams' AND age = 25` | Partial (only last_name) |

#### 4. Unique Index

Enforces uniqueness constraint. Rejects duplicate values.

```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

**Characteristics:**
- Constraint + performance in one
- NULL handling varies by database (some allow multiple NULLs)

#### 5. Covering Index

Index contains all columns needed for query. No table lookup required.

```sql
-- Query
SELECT email, name FROM users WHERE email = 'bob@mail.com';

-- Covering index
CREATE INDEX idx_cover ON users(email, name);
```

```
Query uses only index data:
┌────────────────────────────┐
│   Index (email, name)      │
├────────────────────────────┤
│ alice@mail.com, Alice      │ ← All data here
│ bob@mail.com, Bob          │ ← No table access needed
└────────────────────────────┘
```

**Performance:** Eliminates secondary lookup. Significant for read-heavy queries.

#### 6. Hash Index

Uses hash function for O(1) lookups. Equality only—no ranges.

```
Hash Index:
hash('alice@mail.com') = 42 → Bucket 42 → Row pointer
hash('bob@mail.com') = 17 → Bucket 17 → Row pointer

┌─────────────────────────────┐
│ Bucket 17: bob@mail.com → 2 │
│ Bucket 42: alice@...    → 1 │
└─────────────────────────────┘
```

| Use Case | Hash Index |
|----------|------------|
| `WHERE email = 'x'` | O(1) |
| `WHERE email LIKE 'a%'` | Not supported |
| `WHERE email > 'a'` | Not supported |
| `ORDER BY email` | Not supported |

**Use case:** Memory databases (Redis), exact-match lookups.

#### 7. Full-Text Index

Specialized for text search. Tokenizes content into searchable terms.

```sql
CREATE FULLTEXT INDEX idx_content ON articles(title, body);

SELECT * FROM articles WHERE MATCH(title, body) AGAINST('database indexing');
```

**Characteristics:**
- Supports natural language queries
- Handles stemming (running → run)
- Relevance ranking

#### 8. Bitmap Index

Uses bit arrays for low-cardinality columns. Efficient for AND/OR operations.

```
Column: status (3 values: active, pending, closed)

active:  [1,0,0,1,1,0,0,1...]  ← 1 where row is active
pending: [0,1,0,0,0,1,0,0...]
closed:  [0,0,1,0,0,0,1,0...]

Query: status = 'active' AND type = 'premium'
→ Bitwise AND between active bitmap and premium bitmap
```

**Best for:** Data warehouses, OLAP, columns with few distinct values.
**Avoid for:** OLTP, frequently updated columns (bitmap rebuild expensive).

---

### Index Comparison

| Index Type | Best For | Limitations |
|------------|----------|-------------|
| **B+ Tree** | Range queries, sorting, most use cases | Write overhead |
| **Hash** | Exact equality lookups | No range, no ordering |
| **Bitmap** | Low cardinality, analytics | Poor for updates |
| **Full-text** | Text search | Storage, complexity |
| **Covering** | Eliminating table lookups | Index size |

---

### When NOT to Index

Indexes are not always beneficial.

| Scenario | Why Index Hurts |
|----------|-----------------|
| **Small tables** | Full scan faster than index lookup overhead |
| **Write-heavy workloads** | Every write updates all indexes |
| **Low selectivity columns** | Boolean/status columns return too many rows |
| **Frequently updated columns** | Constant index maintenance |
| **Wide columns** | Large index size, poor cache utilization |

**Rule of thumb:** If a query returns >15-20% of table rows, full scan often beats index.

---

### Index Performance Considerations

#### Write Amplification

```
Single INSERT without indexes:
→ 1 write operation

Single INSERT with 5 indexes:
→ 1 table write + 5 index writes = 6 operations
```

#### Index Size

Indexes consume disk space and memory. Large indexes may not fit in buffer pool, causing disk I/O.

#### Fragmentation

Over time, indexes become fragmented from updates/deletes. Periodic REBUILD/REORGANIZE needed.

#### Query Planner

Database optimizer decides whether to use index. Check with:
```sql
EXPLAIN SELECT * FROM users WHERE email = 'bob@mail.com';
```

Common reasons index ignored:
- Query returns too many rows
- Statistics outdated (run ANALYZE)
- Wrong index for query pattern
- Type mismatch in WHERE clause

---

### Indexing Interview Checklist

| Topic | Key Points |
|-------|------------|
| **What is an index** | Data structure for O(log n) lookups vs O(n) scan |
| **B+ Tree** | Balanced, leaf-linked, range-friendly, most common |
| **Clustered vs non-clustered** | Physical order vs separate pointer structure |
| **Composite index** | Multi-column, leftmost prefix rule |
| **Covering index** | All query columns in index, no table lookup |
| **Hash index** | O(1) equality only, no ranges |
| **Trade-off** | Faster reads, slower writes, more storage |
| **When to avoid** | Small tables, write-heavy, low selectivity |
| **EXPLAIN** | Always verify index usage with query plan |

