## Databases


### Introduction to Databases

A database is an organized collection of data stored and accessed electronically. In system design, database choice directly impacts scalability, consistency, performance, and development complexity.

**Two fundamental questions in interviews:**
1. What type of data are you storing? (structured vs. unstructured)
2. What are your access patterns? (read-heavy, write-heavy, complex queries)

```
┌─────────────────────────────────────────────────────┐
│                   Database Types                     │
├─────────────────────┬───────────────────────────────┤
│        SQL          │           NoSQL               │
│   (Relational)      │      (Non-Relational)         │
├─────────────────────┼───────────────────────────────┤
│ MySQL, PostgreSQL   │ Document: MongoDB, CouchDB    │
│ Oracle, SQL Server  │ Key-Value: Redis, DynamoDB    │
│                     │ Column: Cassandra, HBase      │
│                     │ Graph: Neo4j, Amazon Neptune  │
└─────────────────────┴───────────────────────────────┘
```

---

### SQL Databases

Relational databases store data in tables with predefined schemas. Tables relate to each other through foreign keys.

#### Core Concepts

**Schema:** Fixed structure defined before data insertion. Every row conforms to the schema.

```
Users Table
┌────┬──────────┬─────────────────┬─────────┐
│ id │   name   │      email      │  age    │
├────┼──────────┼─────────────────┼─────────┤
│ 1  │ Alice    │ alice@mail.com  │   28    │
│ 2  │ Bob      │ bob@mail.com    │   34    │
└────┴──────────┴─────────────────┴─────────┘

Orders Table
┌────┬─────────┬────────────┬────────┐
│ id │ user_id │   product  │ amount │
├────┼─────────┼────────────┼────────┤
│ 1  │    1    │   Laptop   │  999   │
│ 2  │    1    │   Mouse    │   25   │
└────┴─────────┴────────────┴────────┘
       ↑
    Foreign Key → References Users.id
```

**JOINS:** Combine data from multiple tables in a single query.

```sql
SELECT users.name, orders.product, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id
WHERE users.id = 1;
```

#### Strengths

| Strength | Why It Matters |
|----------|----------------|
| **ACID compliance** | Guaranteed data integrity for transactions |
| **Complex queries** | JOINs, aggregations, subqueries natively supported |
| **Data integrity** | Foreign keys, constraints, triggers enforce rules |
| **Mature tooling** | Decades of optimization, monitoring, backup tools |

#### Weaknesses

| Weakness | Impact |
|----------|--------|
| **Rigid schema** | Schema changes require migrations, downtime risk |
| **Horizontal scaling is hard** | Sharding breaks JOINs, requires application changes |
| **Not ideal for hierarchical data** | Nested objects require multiple tables and JOINs |

**When to use:** Financial systems, e-commerce orders, user management, any data with complex relationships and need for transactions.

**Examples:** PostgreSQL, MySQL, Oracle, SQL Server, SQLite.

---

### NoSQL Databases

NoSQL ("Not Only SQL") databases sacrifice some SQL guarantees for flexibility and scalability.

#### Types of NoSQL Databases

**1. Document Stores**

Store data as JSON/BSON documents. Each document can have different structure.

```json
// User document in MongoDB
{
  "_id": "user_123",
  "name": "Alice",
  "email": "alice@mail.com",
  "orders": [
    {"product": "Laptop", "amount": 999},
    {"product": "Mouse", "amount": 25}
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

**Use case:** Content management, user profiles, catalogs with varying attributes.

**Examples:** MongoDB, CouchDB, Amazon DocumentDB.

**2. Key-Value Stores**

Simplest model. Store values indexed by unique keys. No query by value—only by key.

```
┌─────────────────┬──────────────────────────┐
│      Key        │         Value            │
├─────────────────┼──────────────────────────┤
│ session:abc123  │ {user_id: 1, expires:...}│
│ user:1:cart     │ [item1, item2, item3]    │
│ config:feature_x│ {"enabled": true}        │
└─────────────────┴──────────────────────────┘
```

**Use case:** Session storage, caching, real-time leaderboards, shopping carts.

**Examples:** Redis, Memcached, Amazon DynamoDB, etcd.

**3. Column-Family Stores**

Store data in columns rather than rows. Optimized for queries over large datasets with specific columns.

```
Row Key: user_123
┌─────────────────────────────────────────────────┐
│ Column Family: profile                          │
│ ┌──────────┬──────────┬───────────────────────┐ │
│ │   name   │   age    │        email          │ │
│ │  Alice   │    28    │   alice@mail.com      │ │
│ └──────────┴──────────┴───────────────────────┘ │
│ Column Family: activity                         │
│ ┌──────────────┬──────────────┬───────────────┐ │
│ │  last_login  │  page_views  │   purchases   │ │
│ │  2024-01-15  │     342      │      12       │ │
│ └──────────────┴──────────────┴───────────────┘ │
└─────────────────────────────────────────────────┘
```

**Use case:** Time-series data, analytics, write-heavy workloads, IoT sensor data.

**Examples:** Cassandra, HBase, ScyllaDB, Google Bigtable.

**4. Graph Databases**

Store entities as nodes and relationships as edges. Optimized for traversing relationships.

```
       ┌───────┐
       │ Alice │
       └───┬───┘
           │ FOLLOWS
           ▼
       ┌───────┐      LIKES      ┌─────────┐
       │  Bob  │────────────────►│ Post #1 │
       └───┬───┘                 └─────────┘
           │ WORKS_AT
           ▼
       ┌─────────┐
       │ Google  │
       └─────────┘
```

**Use case:** Social networks, recommendation engines, fraud detection, knowledge graphs.

**Examples:** Neo4j, Amazon Neptune, ArangoDB, JanusGraph.

---

### SQL vs NoSQL

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (bigger server) | Horizontal (more servers) |
| **Transactions** | ACID guaranteed | BASE (eventual consistency) |
| **Query language** | Standardized SQL | Database-specific APIs |
| **Relationships** | JOINs across tables | Embedded or application-level |
| **Data model** | Tables with rows | Documents, key-value, columns, graphs |
| **Best for** | Complex queries, transactions | Scale, flexibility, specific access patterns |

#### Decision Framework

```
Start
  │
  ▼
Need ACID transactions? ──Yes──► SQL (PostgreSQL, MySQL)
  │
  No
  │
  ▼
Data has complex relationships? ──Yes──► SQL or Graph DB
  │
  No
  │
  ▼
Schema changes frequently? ──Yes──► Document DB (MongoDB)
  │
  No
  │
  ▼
Simple key-based access? ──Yes──► Key-Value (Redis, DynamoDB)
  │
  No
  │
  ▼
Write-heavy time-series? ──Yes──► Column Store (Cassandra)
  │
  No
  │
  ▼
Traversing relationships? ──Yes──► Graph DB (Neo4j)
```

**Interview insight:** "Most production systems use multiple databases. SQL for transactions, Redis for caching, Elasticsearch for search. The skill is knowing which tool for which job."

---

### ACID vs BASE Properties

Two opposing philosophies for database transactions.

#### ACID (SQL Databases)

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All operations succeed or all fail | Transfer $100: debit AND credit both happen or neither |
| **Consistency** | Data moves from one valid state to another | Balance can't go negative if rules forbid it |
| **Isolation** | Concurrent transactions don't interfere | Two transfers from same account don't overwrite each other |
| **Durability** | Committed data survives crashes | Power loss after commit → data still there |

```
ACID Transaction Example:

BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Debit
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Credit
COMMIT;

If any statement fails → entire transaction rolls back
Account 1 won't lose money without Account 2 receiving it
```

#### BASE (NoSQL Databases)

| Property | Meaning | Trade-off |
|----------|---------|-----------|
| **Basically Available** | System always responds | May return stale data |
| **Soft state** | State may change over time without input | Due to eventual consistency |
| **Eventually consistent** | Given time, all replicas converge | Not immediately consistent |

```
BASE Example (Shopping Cart):

User adds item on Node A
  │
  ▼
Node A: cart = [item1, item2, item3]
Node B: cart = [item1, item2]  ← hasn't received update yet
  │
  (milliseconds later)
  │
  ▼
Node B: cart = [item1, item2, item3]  ← now consistent
```

#### When to Use Each

| ACID | BASE |
|------|------|
| Banking transactions | Social media feeds |
| Order processing | Shopping carts |
| Inventory management | Analytics/metrics |
| User authentication | Session storage |
| Anything where "almost correct" = wrong | Anything where "eventually correct" = acceptable |

**Interview insight:** "ACID costs performance and availability. BASE costs consistency. Choose based on business impact of inconsistency—financial data needs ACID, like counts can use BASE."

---

### Real-World Database Choices

| Company | Use Case | Database Choice | Why |
|---------|----------|-----------------|-----|
| **Uber** | Trip data | PostgreSQL + Cassandra | SQL for transactions, Cassandra for scale |
| **Netflix** | User data | Cassandra | Global scale, availability over consistency |
| **Airbnb** | Listings | MySQL + Elasticsearch | MySQL for data, ES for search |
| **Twitter** | Tweets | MySQL + Manhattan (custom) | Started SQL, built custom for scale |
| **Instagram** | Posts/Users | PostgreSQL + Cassandra | Postgres for relations, Cassandra for feeds |
| **Discord** | Messages | Cassandra → ScyllaDB | Write-heavy, time-series nature |

**Pattern:** Most large systems are polyglot—multiple databases for different purposes.

---

### SQL Normalization and Denormalization

#### Normalization

Organizing data to reduce redundancy. Split data into multiple related tables.

**Normal Forms (simplified):**

**1NF:** No repeating groups, atomic values only
```
Bad:  | id | phones           |
      | 1  | 123-456, 789-012 |

Good: | id | phone   |
      | 1  | 123-456 |
      | 1  | 789-012 |
```

**2NF:** 1NF + no partial dependencies (every non-key column depends on entire primary key)

**3NF:** 2NF + no transitive dependencies (non-key columns don't depend on other non-key columns)

```
Normalized Design:
┌─────────┐      ┌──────────┐      ┌──────────┐
│  Users  │      │  Orders  │      │ Products │
├─────────┤      ├──────────┤      ├──────────┤
│ id (PK) │◄────►│ user_id  │      │ id (PK)  │
│ name    │      │ prod_id  │◄────►│ name     │
│ email   │      │ quantity │      │ price    │
└─────────┘      └──────────┘      └──────────┘
```

**Pros:** No data duplication, easier updates, data integrity
**Cons:** Complex queries (JOINs), slower reads

#### Denormalization

Intentionally adding redundancy for read performance.

```
Denormalized Design:
┌─────────────────────────────────────────────┐
│                   Orders                     │
├─────────────────────────────────────────────┤
│ id │ user_id │ user_name │ product │ price │
├────┼─────────┼───────────┼─────────┼───────┤
│ 1  │    1    │   Alice   │ Laptop  │  999  │
│ 2  │    1    │   Alice   │ Mouse   │   25  │
└────┴─────────┴───────────┴─────────┴───────┘
         ↑           ↑
    user_name and price duplicated for fast reads
```

**Pros:** Faster reads (no JOINs), simpler queries
**Cons:** Data duplication, update anomalies, storage overhead

#### When to Denormalize

| Scenario | Action |
|----------|--------|
| Read-heavy workload (>90% reads) | Denormalize |
| Write-heavy workload | Keep normalized |
| Need fast dashboards/reports | Denormalize or use read replicas |
| Data integrity critical | Keep normalized |
| Scaling horizontally (sharding) | Denormalize (JOINs break across shards) |

**Interview insight:** "Normalize for writes, denormalize for reads. Most systems start normalized and selectively denormalize hot paths based on profiling."

---

### In-Memory Database vs. On-Disk Database

#### On-Disk Databases

Data persisted to disk (SSD/HDD). Survives restarts.

```
Write Path:
Application → Database → Write-Ahead Log (disk) → Memory → Background flush to disk

Read Path:
Application → Database → Check memory cache → If miss, read from disk
```

**Latency:** Milliseconds (disk I/O bound)
**Durability:** Data survives power loss
**Capacity:** Limited by disk size (terabytes+)

**Examples:** PostgreSQL, MySQL, MongoDB, Cassandra.

#### In-Memory Databases

Data stored primarily in RAM. Optionally persisted to disk.

```
Write Path:
Application → Database (RAM) → Optional async persistence

Read Path:
Application → Database (RAM) → Return immediately
```

**Latency:** Microseconds (memory-speed)
**Durability:** Data lost on crash unless persisted
**Capacity:** Limited by RAM (expensive at scale)

**Examples:** Redis, Memcached, VoltDB, SAP HANA.

#### Comparison

| Aspect | On-Disk | In-Memory |
|--------|---------|-----------|
| **Latency** | ~1-10ms | ~0.1-1ms |
| **Durability** | Default | Requires configuration |
| **Cost** | Lower (disk is cheap) | Higher (RAM is expensive) |
| **Capacity** | Terabytes+ | Gigabytes (practically) |
| **Use case** | Primary data store | Cache, session, real-time |

#### Redis Persistence Options

```
RDB (Snapshotting):
  - Point-in-time snapshots every N minutes
  - Fast recovery, but lose data since last snapshot

AOF (Append-Only File):
  - Log every write operation
  - Better durability, slower recovery
  - Can replay log to rebuild state

Hybrid (RDB + AOF):
  - Best of both: snapshots + recent operations log
```

**Interview insight:** "Redis is not just a cache—with AOF persistence, it can be a primary database for appropriate use cases. But always plan for data loss scenarios."

---

### Data Replication vs. Data Mirroring

Both create copies of data, but for different purposes.

#### Data Replication

Copies data across multiple nodes for availability and read scaling. Replicas may lag behind primary.

```
        ┌──────────┐
        │  Primary │ ◄── All writes
        └────┬─────┘
             │ async replication
     ┌───────┼───────┐
     ▼       ▼       ▼
┌────────┐┌────────┐┌────────┐
│Replica1││Replica2││Replica3│ ◄── Handle reads
└────────┘└────────┘└────────┘

Replicas may be seconds/minutes behind (replication lag)
```

**Purpose:** High availability, read scaling, geographic distribution
**Consistency:** Eventually consistent (unless synchronous)
**Latency impact:** Minimal (async)

#### Data Mirroring

Creates exact, synchronous copy. Mirror is always identical to source.

```
┌──────────┐         ┌──────────┐
│  Primary │◄──────► │  Mirror  │
└──────────┘  sync   └──────────┘

Write completes only after both acknowledge
Mirror is ALWAYS identical to Primary
```

**Purpose:** Disaster recovery, zero data loss
**Consistency:** Strongly consistent (synchronous)
**Latency impact:** Higher (must wait for mirror ACK)

#### Comparison

| Aspect | Replication | Mirroring |
|--------|-------------|-----------|
| **Synchronization** | Async (usually) | Synchronous |
| **Lag** | Seconds to minutes | Zero |
| **Data loss risk** | Possible (lag window) | None |
| **Performance impact** | Minimal | Higher latency |
| **Use case** | Read scaling, HA | DR, compliance |
| **Failover** | Promote replica (may lose recent data) | Instant, no data loss |

**Interview insight:** "Replication is for scaling and availability. Mirroring is for disaster recovery with zero RPO. Use replication for read replicas, mirroring for critical DR requirements."

---

### Database Federation

Splitting databases by function/domain. Each service owns its database.

```
Monolithic:                    Federated:
┌───────────────┐             ┌─────────┐  ┌─────────┐  ┌─────────┐
│   Single DB   │             │ User DB │  │Order DB │  │Product DB│
│ Users         │     →       └────┬────┘  └────┬────┘  └────┬────┘
│ Orders        │                  │            │            │
│ Products      │             ┌────┴────┐  ┌────┴────┐  ┌────┴────┐
│ Payments      │             │User Svc │  │Order Svc│  │Product  │
└───────────────┘             └─────────┘  └─────────┘  │   Svc   │
                                                        └─────────┘
```

#### Benefits

| Benefit | Description |
|---------|-------------|
| **Independent scaling** | Scale each database based on its load |
| **Isolation** | User DB issues don't affect Order DB |
| **Technology freedom** | Use PostgreSQL for users, Cassandra for events |
| **Team autonomy** | Each team owns their data |

#### Challenges

| Challenge | Mitigation |
|-----------|------------|
| **Cross-database queries** | API calls between services, or data duplication |
| **Distributed transactions** | Saga pattern, eventual consistency |
| **Data consistency** | Event-driven sync, accept eventual consistency |
| **Operational complexity** | More databases to manage, monitor, backup |

#### Federation vs. Sharding

| Aspect | Federation | Sharding |
|--------|------------|----------|
| **Split by** | Function/domain | Data (same schema) |
| **Schema** | Different per database | Same across shards |
| **Use case** | Microservices | Single service at scale |
| **Example** | Users DB, Orders DB, Products DB | Users shard 1, Users shard 2, Users shard 3 |

**Interview insight:** "Federation aligns with microservices—each service owns its data. But it introduces distributed systems complexity. Start with a monolith, federate when team/scale demands it."

---

### Database Interview Checklist

| Topic | Key Points |
|-------|------------|
| **SQL vs NoSQL** | SQL = ACID, complex queries; NoSQL = scale, flexibility |
| **NoSQL types** | Document, Key-Value, Column-Family, Graph |
| **ACID** | Atomicity, Conssistency, Isolation, Durability |
| **BASE** | Basically Available, Soft state, Eventually consistent |
| **When SQL** | Transactions, complex relationships, data integrity |
| **When NoSQL** | Scale, flexible schema, specific access patterns |
| **Normalization** | Reduce redundancy, easier writes, slower reads |
| **Denormalization** | Add redundancy, faster reads, update complexity |
| **In-memory vs disk** | RAM = fast/volatile, Disk = slow/durable |
| **Replication vs mirroring** | Async/scale vs sync/DR |
| **Federation** | Split by function, microservices pattern |
| **Polyglot persistence** | Use multiple databases for different needs |

