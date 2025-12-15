## Leader and Follower


### What is Leader and Follower Pattern?

A **leader-follower** (or master-slave) pattern designates one node as the leader that handles writes and coordinates the cluster, while followers replicate data and handle reads.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Leader-Follower Architecture                  │
└─────────────────────────────────────────────────────────────────┘

                        ┌──────────────┐
          Writes ──────▶│    LEADER    │
                        │   (primary)  │
                        └──────┬───────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        ┌──────────┐     ┌──────────┐     ┌──────────┐
        │ FOLLOWER │     │ FOLLOWER │     │ FOLLOWER │
        │ (replica)│     │ (replica)│     │ (replica)│
        └──────────┘     └──────────┘     └──────────┘
              ▲                ▲                ▲
              │                │                │
              └────────────────┴────────────────┘
                         Reads (scaled)
```

### Why Leader-Follower?

| Benefit | Explanation |
|---------|-------------|
| **Write consistency** | Single point for writes prevents conflicts |
| **Read scalability** | Distribute reads across many followers |
| **Simpler consensus** | Only leader decides order of operations |
| **Fault tolerance** | Followers can become leader if primary fails |

### Replication Modes

**1. Synchronous Replication:**
```
Client        Leader           Follower 1       Follower 2
  │             │                  │                │
  │── Write ──▶│                  │                │
  │             │── Replicate ───▶│                │
  │             │── Replicate ────────────────────▶│
  │             │                  │                │
  │             │◀─── ACK ────────│                │
  │             │◀─── ACK ───────────────────────── │
  │◀── OK ─────│                  │                │
  │             │                  │                │

✓ Strong consistency (read any replica, get latest)
✗ High latency (wait for all replicas)
✗ Availability impacted if replica slow/down
```

**2. Asynchronous Replication:**
```
Client        Leader           Follower 1       Follower 2
  │             │                  │                │
  │── Write ──▶│                  │                │
  │◀── OK ─────│                  │                │
  │             │                  │                │
  │             │── Replicate ───▶│   (later)      │
  │             │── Replicate ────────────────────▶│
  │             │                  │                │

✓ Low latency (respond immediately)
✓ Higher availability
✗ Data loss if leader fails before replication
✗ Followers may serve stale reads
```

**3. Semi-synchronous:**
```
Wait for at least 1 follower ACK before responding.
Balance between consistency and latency.
```

### Leader Election

When leader fails, a new leader must be elected:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Leader Election Process                       │
└─────────────────────────────────────────────────────────────────┘

1. Leader Failure Detection:
   ┌────────┐     ┌────────┐     ┌────────┐
   │Leader A│ 💀  │Follower│     │Follower│
   │  DEAD  │     │   B    │     │   C    │
   └────────┘     └────────┘     └────────┘
                       │               │
                  Heartbeat timeout detected

2. Election:
   - Candidates propose themselves
   - Nodes vote (often for most up-to-date candidate)
   - Candidate with quorum becomes leader

3. New Leader:
                  ┌────────┐     ┌────────┐
                  │Leader B│     │Follower│
                  │  NEW   │     │   C    │
                  └────────┘     └────────┘
```

### Common Election Algorithms

| Algorithm | Used By | Key Feature |
|-----------|---------|-------------|
| **Raft** | etcd, Consul | Understandable, leader-based |
| **Paxos** | Chubby, Spanner | Proven correct, complex |
| **ZAB** | ZooKeeper | Optimized for primary-backup |
| **Bully** | Simple systems | Highest ID wins |

### Split-Brain Prevention

```
Network Partition:
┌───────────────────┐     ┌───────────────────┐
│ Partition A       │     │ Partition B       │
│                   │     │                   │
│ ┌──────┐ ┌──────┐│  X  │┌──────┐ ┌──────┐  │
│ │Node 1│ │Node 2││◀───▶││Node 3│ │Node 4│  │
│ └──────┘ └──────┘│     │└──────┘ └──────┘  │
│                   │     │                   │
│ 2 nodes           │     │ 2 nodes           │
│ < quorum (3)      │     │ < quorum (3)      │
│ Cannot elect      │     │ Cannot elect      │
│ leader            │     │ leader            │
└───────────────────┘     └───────────────────┘

Solution: Require majority (quorum) to elect leader.
Neither partition has majority → system waits for network heal.
```

### Real-World Examples

| System | Leader Role |
|--------|-------------|
| **MySQL** | Primary handles writes; replicas handle reads |
| **Kafka** | Partition leader handles produce/consume |
| **MongoDB** | Primary in replica set handles writes |
| **Redis Sentinel** | Master handles commands; slaves replicate |

---

