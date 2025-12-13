## Redundancy and Replication


### What is Redundancy?

Redundancy is duplicating critical system components to eliminate single points of failure (SPOF). If Component A fails, Component A' takes over instantly or with minimal interruption.

**Core Principle:** Any component whose failure brings down the system is a SPOF. Redundancy eliminates SPOFs by ensuring no single failure can cause total outage.

**Types:**

| Type | What's Duplicated | Example |
|------|-------------------|---------|
| **Hardware** | Physical components | RAID disks, dual power supplies, standby servers |
| **Data** | Information copies | Database replicas, backup files |
| **Network** | Connectivity paths | Multiple NICs, dual ISPs, redundant switches |
| **Geographic** | Entire locations | Multi-region deployments, DR sites |

**Redundancy Patterns:**

**Active-Active:** All redundant components handle traffic simultaneously. Load is distributed. If one fails, others absorb its share.
```
[Load Balancer]
     |
  ┌──┴──┐
  ▼     ▼
[App1] [App2]  ← Both serving requests
```

**Active-Passive (Hot Standby):** Primary handles all traffic. Standby is running but idle, ready for instant failover.
```
[Primary] ──heartbeat──► [Standby]
    ▲                        │
    └── traffic              └── takes over on failure
```

**Active-Passive (Cold Standby):** Standby is powered off. Requires manual or automated boot on failure. Cheaper but slower recovery.

**N+1 Redundancy:** For N required components, deploy N+1. Common in server clusters—if you need 4 servers for load, deploy 5.

**2N Redundancy:** Full duplication. Critical for systems requiring zero downtime (e.g., hospital equipment, financial trading).

**Interview insight:** "Redundancy level depends on failure cost. A blog can use N+1. A stock exchange uses 2N with geographic redundancy. Always ask: what's the cost of 1 hour downtime vs. cost of redundancy?"

---

### What is Replication?

Replication continuously copies data from one node (source/primary) to others (replicas/secondaries). Unlike backup, replicas are live and queryable.

**Why Replicate:**

1. **High Availability:** If primary dies, promote replica to primary. No data reconstruction needed.

2. **Read Scaling:** Primary handles writes. Replicas handle reads. A single MySQL primary can support 10+ read replicas, multiplying read throughput.

3. **Geographic Distribution:** Replica in Europe serves European users with low latency while primary sits in US.

4. **Fault Isolation:** Analytical queries on replica don't impact production primary.

**Replication Lag:**

The delay between write on primary and visibility on replica. Critical concept.

```
Time 0ms: Write "balance=100" to Primary
Time 0ms: Primary acknowledges write to client
Time 50ms: Replica receives and applies write
         ↑
    50ms replication lag
```

**Consequences of lag:**
- User writes, then reads from replica, sees stale data
- Read-after-write inconsistency breaks user experience

**Mitigation:** Read-your-writes consistency (route user's reads to primary after their writes), or use synchronous replication.

---

### Replication Methods

#### 1. Synchronous Replication

Primary waits for ALL replicas to acknowledge before confirming write to client.

```
Client ──write──► Primary ──replicate──► Replica1
                     │                      │
                     │◄────────ACK──────────┘
                     │
                     ├──replicate──► Replica2
                     │                   │
                     │◄───────ACK────────┘
                     │
                  commit
                     │
Client ◄───ACK───────┘
```

| Pros | Cons |
|------|------|
| Zero data loss (RPO = 0) | Write latency = slowest replica |
| Strong consistency | One slow/dead replica blocks all writes |
| Simple failover (any replica is up-to-date) | Doesn't scale geographically (cross-region latency kills performance) |

**Use case:** Financial systems where losing a single transaction is unacceptable. Primary-standby within same data center.

#### 2. Asynchronous Replication

Primary confirms write immediately. Replication happens in background.

```
Client ──write──► Primary
                     │
Client ◄───ACK───────┘  (immediate)
                     │
                     └──replicate (background)──► Replicas
```

| Pros | Cons |
|------|------|
| Low write latency | Data loss window (RPO > 0) |
| Primary unaffected by replica failures | Replicas can fall behind (lag) |
| Works across regions | Failover may lose recent writes |

**Data loss scenario:**
```
Time 0: Write W1 to Primary, ACK to client
Time 1: Write W2 to Primary, ACK to client
Time 2: Primary crashes (W1, W2 not yet replicated)
Time 3: Promote Replica → W1, W2 lost forever
```

**Use case:** Read replicas, cross-region replication where latency matters more than zero data loss.

#### 3. Semi-Synchronous Replication

Primary waits for at least ONE replica to acknowledge. Others replicate asynchronously.

```
Client ──write──► Primary ──replicate──► Replica1 (sync)
                     │                      │
                     │◄────────ACK──────────┘
                  commit
                     │
Client ◄───ACK───────┘
                     │
                     └──replicate (async)──► Replica2, Replica3
```

**Trade-off:** Guaranteed durability on at least 2 nodes (primary + 1 replica) without waiting for all replicas.

**Use case:** MySQL semi-synchronous replication. PostgreSQL synchronous_commit with `remote_write`.

---

### Replication Topologies

#### 1. Single-Leader (Primary-Replica)

One node accepts writes. All others are read-only replicas.

```
        [Primary]
       /    |    \
      ▼     ▼     ▼
   [R1]   [R2]   [R3]

Writes: Primary only
Reads: Primary + all replicas
```

**Conflict handling:** None needed—only one writer.

**Failover:** Promote one replica to primary. Other replicas must re-point to new primary.

**Bottleneck:** Write throughput limited to single node.

**Use case:** PostgreSQL streaming replication, MySQL replication, Redis Sentinel.

#### 2. Multi-Leader (Active-Active)

Multiple nodes accept writes. Each replicates to others.

```
[Leader1] ◄──────► [Leader2]
    │                  │
    ▼                  ▼
  [R1]               [R2]
```

**Conflict handling:** Required. Same row updated on both leaders simultaneously.

**Conflict resolution strategies:**
- **Last-Write-Wins (LWW):** Timestamp determines winner. Simple but loses data.
- **Custom merge:** Application logic merges conflicting values.
- **CRDTs:** Data structures that mathematically guarantee conflict-free merges.

**Use case:** Multi-datacenter writes (each DC has a leader), collaborative editing, CouchDB.

**Interview insight:** "Multi-leader is complex. Only use when you truly need write availability in multiple regions. Most systems are fine with single-leader + async cross-region replicas."

#### 3. Leaderless (Dynamo-style)

No leader. Any node accepts reads and writes. Uses quorums for consistency.

```
     Client
    /   |   \
   ▼    ▼    ▼
 [N1]  [N2]  [N3]
```

**Quorum formula:** `W + R > N`
- N = total replicas
- W = nodes that must ACK a write
- R = nodes that must respond to a read

**Example (N=3, W=2, R=2):**
- Write succeeds if 2/3 nodes ACK
- Read queries 2/3 nodes, returns latest version
- Guarantees read sees latest write (overlap guaranteed)

**Sloppy quorum:** During partition, write to any available nodes (not necessarily the designated replicas). "Hinted handoff" repairs later.

**Use case:** Cassandra, DynamoDB, Riak. High availability, partition tolerance, eventual consistency.

**Interview insight:** "Leaderless trades consistency complexity for availability. Great for use cases tolerating eventual consistency (shopping carts, session data). Avoid for transactions requiring strong consistency."

---

### Data Backup vs. Disaster Recovery

**Backup:** Point-in-time copy of data. Insurance against data loss.

**Disaster Recovery (DR):** Comprehensive plan to restore entire business operations after catastrophic failure.

| Aspect | Backup | Disaster Recovery |
|--------|--------|-------------------|
| **Purpose** | Restore lost/corrupted data | Resume business operations |
| **Scope** | Data files, databases | Data + servers + network + applications + DNS |
| **Protects Against** | Accidental deletion, corruption, ransomware | Data center fire, regional outage, natural disaster |
| **Speed** | Hours to days acceptable | Minutes to hours required |
| **Location** | Can be same site (local backup) | Must be geographically separate |
| **Testing** | Restore individual files | Full failover drill |

**Key Metrics:**

**RPO (Recovery Point Objective):** Maximum tolerable data loss measured in time.

```
Last backup: 6:00 AM
Disaster: 10:00 AM
Data loss: 4 hours of transactions

If RPO = 1 hour → VIOLATED (lost 4 hours)
If RPO = 24 hours → MET (lost only 4 hours)
```

Lower RPO = more frequent backups/replication = higher cost.

**RTO (Recovery Time Objective):** Maximum tolerable downtime.

```
Disaster: 10:00 AM
Systems restored: 2:00 PM
Downtime: 4 hours

If RTO = 1 hour → VIOLATED
If RTO = 8 hours → MET
```

Lower RTO = hot standby, automated failover = higher cost.

**DR Strategies (Cold → Hot):**

| Strategy | RTO | RPO | Cost | Description |
|----------|-----|-----|------|-------------|
| **Backup & Restore** | Days | Hours | $ | Restore from backups to new infrastructure |
| **Pilot Light** | Hours | Minutes | $$ | Minimal core infrastructure running, scale up on disaster |
| **Warm Standby** | Minutes | Seconds | $$$ | Scaled-down replica running, scale up on disaster |
| **Hot Standby (Active-Active)** | Seconds | Zero | $$$$ | Full replica running, instant failover |

**Interview insight:** "Always ask business stakeholders: what's 1 hour of downtime cost? What's losing 1 hour of data cost? That determines RTO/RPO targets, which determines architecture and budget."

---

### Redundancy vs. Replication vs. Backup

| Concept | What | Purpose | Protects Against | Recovery Time |
|---------|------|---------|------------------|---------------|
| **Redundancy** | Duplicate components | Eliminate SPOF | Hardware failure | Instant (automatic failover) |
| **Replication** | Live data copies | Availability + read scaling | Node failure, read load | Instant to seconds |
| **Backup** | Point-in-time snapshots | Data recovery | Corruption, deletion, ransomware | Minutes to hours |

**Why all three are needed:**

```
Scenario: Corrupted write propagates to all replicas

Redundancy: ✗ Doesn't help (corruption isn't hardware failure)
Replication: ✗ Corruption replicated to all nodes
Backup: ✓ Restore from pre-corruption snapshot
```

```
Scenario: Primary server catches fire

Redundancy: ✓ Standby server takes over
Replication: ✓ Replica promoted to primary
Backup: Unnecessary (no data corruption, just hardware loss)
```

```
Scenario: User accidentally deletes production database

Redundancy: ✗ Deletion executed on primary
Replication: ✗ DELETE replicated to all replicas instantly
Backup: ✓ Restore from last backup
```

**Interview insight:** "Replication is not backup. If you `DROP TABLE users`, that command replicates to all replicas within milliseconds. You need backup for logical errors. You need replication for physical failures. You need redundancy for component failures. Defense in depth."

---

