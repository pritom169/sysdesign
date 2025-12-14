## Quorum


### What is Quorum?

A **quorum** is the minimum number of nodes that must agree on an operation for it to be considered successful in a distributed system. It ensures consistency while allowing some nodes to be unavailable.

```
5-Node Cluster with Quorum = 3
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   ┌───┐     ┌───┐     ┌───┐     ┌───┐     ┌───┐          │
│   │ A │     │ B │     │ C │     │ D │     │ E │          │
│   │ ✓ │     │ ✓ │     │ ✓ │     │ ✗ │     │ ✗ │          │
│   └───┘     └───┘     └───┘     └───┘     └───┘          │
│                                                           │
│   3 nodes responded ✓  =  Quorum reached  =  SUCCESS     │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Why Quorum Matters

**Without quorum (any node can accept writes):**
```
Network Partition
┌─────────────────┐          ┌─────────────────┐
│  Partition A    │          │  Partition B    │
│                 │          │                 │
│   ┌───┐ ┌───┐  │          │  ┌───┐ ┌───┐   │
│   │ A │ │ B │  │    ✗     │  │ D │ │ E │   │
│   └───┘ └───┘  │◀────────▶│  └───┘ └───┘   │
│                 │          │                 │
│ Write X = 10   │          │  Write X = 20   │
│                 │          │                 │
└─────────────────┘          └─────────────────┘

Result: Split-brain! Both partitions accept conflicting writes.
```

**With quorum (majority required):**
```
5 nodes, Quorum = 3

┌─────────────────┐          ┌─────────────────┐
│  Partition A    │          │  Partition B    │
│  (3 nodes)      │          │  (2 nodes)      │
│                 │          │                 │
│ Has quorum ✓    │          │  No quorum ✗    │
│ Can accept      │          │  Read-only or   │
│ writes          │          │  reject writes  │
└─────────────────┘          └─────────────────┘

Only one partition can make progress → No split-brain
```

### Quorum Formula

For a cluster of **N** nodes:

| Quorum Type | Formula | Purpose |
|-------------|---------|---------|
| **Majority Quorum** | ⌊N/2⌋ + 1 | Standard consensus |
| **Read Quorum (R)** | Configurable | Nodes to read from |
| **Write Quorum (W)** | Configurable | Nodes to write to |

**Consistency Rule:** `R + W > N` ensures reads see latest writes.

```
N = 5 nodes

Strong Consistency:     R = 3, W = 3  →  R + W = 6 > 5 ✓
                        Any read overlaps with any write

Faster Reads:           R = 2, W = 4  →  R + W = 6 > 5 ✓
                        Reads from 2 nodes (fast), writes to 4

Faster Writes:          R = 4, W = 2  →  R + W = 6 > 5 ✓
                        Writes to 2 nodes (fast), reads from 4

Eventual Consistency:   R = 1, W = 1  →  R + W = 2 < 5 ✗
                        Fast but may read stale data
```

### Read/Write Quorum Visualization

```
                    5-Node Cluster
        ┌─────┬─────┬─────┬─────┬─────┐
Nodes:  │  A  │  B  │  C  │  D  │  E  │
        └─────┴─────┴─────┴─────┴─────┘

Write with W=3:
        ┌─────┬─────┬─────┬─────┬─────┐
        │  A  │  B  │  C  │  D  │  E  │
        │ ✓W  │ ✓W  │ ✓W  │     │     │
        └─────┴─────┴─────┴─────┴─────┘

Read with R=3:
        ┌─────┬─────┬─────┬─────┬─────┐
        │  A  │  B  │  C  │  D  │  E  │
        │     │     │ ✓R  │ ✓R  │ ✓R  │
        └─────┴─────┴─────┴─────┴─────┘
                      ↑
              Overlap guarantees
              reading latest write
```

### Quorum in Practice

| System | Quorum Implementation |
|--------|----------------------|
| **Cassandra** | Tunable consistency: ONE, QUORUM, ALL |
| **ZooKeeper** | Majority quorum for leader election |
| **etcd/Raft** | Majority quorum for log replication |
| **DynamoDB** | Sloppy quorum with hinted handoff |

---
