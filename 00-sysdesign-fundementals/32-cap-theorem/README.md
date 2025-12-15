## CAP Theorem


### Introduction to CAP Theorem

CAP theorem states that a distributed system can only guarantee **two of three** properties simultaneously:

- **Consistency (C):** Every read receives the most recent write or an error
- **Availability (A):** Every request receives a non-error response (no guarantee it's the latest data)
- **Partition Tolerance (P):** System continues operating despite network partitions between nodes

**The catch:** Network partitions are inevitable in distributed systems. You don't choose P—you have it forced upon you. The real choice is **C vs A** when a partition occurs.

```
         Consistency
            /\
           /  \
          /    \
         / CP   \
        /________\
       /\        /\
      /  \  CA  /  \
     / AP \    /    \
    /______\  /______\
Availability    Partition
                Tolerance
```

**Interview insight:** "CAP is about behavior during partitions. When network splits, do you return stale data (choose A) or return an error (choose C)? You can't have both."

---

### Components of CAP Theorem

#### Consistency

All nodes see the same data at the same time. After a write completes, all subsequent reads return that value.

```
Write X=5 to Node A
         │
         ▼
    ┌─────────┐
    │ Node A  │──sync──► Node B, Node C
    │  X=5    │
    └─────────┘
         │
Read from ANY node → returns X=5
```

**Strong consistency requires coordination:** Every write must propagate to all nodes before acknowledgment. This adds latency and reduces availability (if a node is unreachable, writes block).

#### Availability

Every request receives a response. No request times out or returns an error due to system state.

**Key distinction:** Availability in CAP doesn't mean "high availability" (99.99% uptime). It means every non-failing node must respond to every request. A node that's up must answer—it can't say "I don't know, ask someone else."

#### Partition Tolerance

System continues to function when network communication between nodes fails.

```
Normal:           Partition:
┌──────┐         ┌──────┐
│Node A│◄──────► │Node B│
└──────┘         └──────┘
    │                ╳     ← network split
    ▼                │
┌──────┐         ┌──────┐
│Node C│◄──────► │Node D│
└──────┘         └──────┘
```

**Partitions happen:** Network cables cut, switches fail, cloud availability zones lose connectivity. Any production distributed system must handle partitions—making P non-negotiable.

---

### Trade-offs in CAP Theorem

Since P is mandatory, the real choice is:

#### CP Systems (Consistency + Partition Tolerance)

During partition, refuse requests that can't guarantee consistency.

```
Partition occurs:
┌──────────┐      ╳      ┌──────────┐
│  Node A  │─────────────│  Node B  │
│ (leader) │             │(follower)│
└──────────┘             └──────────┘

Client → Node B: "Write X=10"
Node B: "ERROR: Cannot reach leader, refusing write"
```

| Behavior | Impact |
|----------|--------|
| Writes may fail | Users see errors |
| Reads may block | Higher latency |
| Data always correct | Strong consistency guaranteed |

**Use cases:** Financial transactions, inventory systems, leader election, distributed locks.

**Examples:** Zookeeper, etcd, HBase, MongoDB (with majority write concern), Spanner.

#### AP Systems (Availability + Partition Tolerance)

During partition, accept requests but allow inconsistency.

```
Partition occurs:
┌──────────┐      ╳      ┌──────────┐
│  Node A  │─────────────│  Node B  │
│   X=5    │             │   X=5    │
└──────────┘             └──────────┘
     │                         │
  Write X=10               Write X=20
     │                         │
     ▼                         ▼
┌──────────┐             ┌──────────┐
│  Node A  │             │  Node B  │
│   X=10   │             │   X=20   │
└──────────┘             └──────────┘

Partition heals → conflict: X=10 or X=20?
```

| Behavior | Impact |
|----------|--------|
| Writes always succeed | Users never see errors |
| Reads return something | May be stale |
| Conflicts possible | Need resolution strategy (LWW, merge, CRDT) |

**Use cases:** Shopping carts, social media feeds, DNS, session stores.

**Examples:** Cassandra, DynamoDB, CouchDB, Riak.

#### CA Systems?

CA (Consistency + Availability without Partition Tolerance) is **theoretical only**.

In practice: If you have a single-node database (no distribution), you have CA—but that's not a distributed system. The moment you distribute data, partitions become possible.

**Interview insight:** "CA systems don't exist in distributed computing. If someone says their system is CA, either it's a single node, or they're sacrificing one of C/A during partitions without admitting it."

---

### CAP Theorem Decision Matrix

| Scenario | Choose | Why |
|----------|--------|-----|
| Bank account balance | CP | Wrong balance = money lost |
| Shopping cart | AP | Unavailable cart = lost sale |
| Distributed lock | CP | Two holders = data corruption |
| Social media likes | AP | Stale count is acceptable |
| Inventory count (last item) | CP | Overselling = fulfillment nightmare |
| User session | AP | Logged out user = bad UX |
| Leader election | CP | Two leaders = split brain disaster |

---

### Beyond CAP Theorem

CAP is a simplification. Real systems are more nuanced.

#### PACELC Theorem

Extends CAP: Even when there's **no partition (E)**, there's still a trade-off between **latency (L)** and **consistency (C)**.

```
If Partition:
  Choose A or C (same as CAP)
Else (normal operation):
  Choose L or C
```

**Full form:** If **P**artition, choose **A**vailability or **C**onsistency; **E**lse, choose **L**atency or **C**onsistency.

| System | During Partition (PAC) | Normal Operation (ELC) |
|--------|------------------------|------------------------|
| DynamoDB | PA (available) | EL (low latency) |
| Cassandra | PA | EL |
| MongoDB | PC (consistent) | EC (consistent) |
| Spanner | PC | EC (pays latency cost) |

**Interview insight:** "PACELC reveals that even without failures, you're still choosing between fast responses and strong consistency. Spanner chooses consistency always (PC/EC), paying with latency. DynamoDB chooses speed always (PA/EL), requiring conflict resolution."

#### Tunable Consistency

Modern databases let you choose consistency per-operation, not per-system.

**Cassandra example:**
```
// Strong consistency (quorum)
SELECT * FROM users WHERE id=1 CONSISTENCY QUORUM;

// Fast but potentially stale
SELECT * FROM users WHERE id=1 CONSISTENCY ONE;
```

**Quorum formula:** `W + R > N` guarantees consistency
- N = replicas, W = write acknowledgments, R = read nodes
- N=3, W=2, R=2 → always read your writes
- N=3, W=1, R=1 → fast but inconsistent

---

### System Design Trade-offs in Interviews

#### How to Apply CAP in Interviews

1. **Identify the data type:** Is incorrect data catastrophic (financial) or tolerable (social)?

2. **Consider failure modes:** What happens during network issues? Users see errors vs. stale data?

3. **State your choice explicitly:**
   - "For payment processing, I'll choose CP because double-charging is worse than temporary unavailability"
   - "For the news feed, I'll choose AP because showing a slightly stale feed is better than an error page"

4. **Mention tunable consistency:** "We can use strong consistency for writes and eventual consistency for reads to optimize both correctness and performance"

#### Common Interview Mistakes

| Mistake | Reality |
|---------|---------|
| "I'll use CA for high availability" | CA doesn't exist in distributed systems |
| "CAP means pick any 2" | P is mandatory; you're choosing C vs A during partitions |
| "Eventual consistency means inconsistent" | It means temporarily inconsistent, eventually correct |
| "Strong consistency is always better" | It's slower and less available—often overkill |

#### CAP Interview Checklist

| Topic | Key Points |
|-------|------------|
| **CAP components** | C = same data everywhere, A = always respond, P = survive network splits |
| **Real choice** | P is mandatory; choose C or A during partitions |
| **CP examples** | Zookeeper, etcd, Spanner, MongoDB (majority writes) |
| **AP examples** | Cassandra, DynamoDB, CouchDB, DNS |
| **PACELC** | Even without partition, trade-off between latency and consistency |
| **Tunable consistency** | Quorum reads/writes let you choose per-operation |
| **When CP** | Financial data, locks, leader election, inventory |
| **When AP** | Social feeds, carts, sessions, caches |

---

