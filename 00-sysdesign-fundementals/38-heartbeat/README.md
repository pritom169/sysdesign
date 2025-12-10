## Heartbeat


### What is Heartbeat?

A **heartbeat** is a periodic signal sent between nodes in a distributed system to indicate liveness. If heartbeats stop, the node is presumed dead.

```
Normal Operation:
┌──────────┐                      ┌──────────┐
│  Node A  │ ────── ♥ ──────────▶ │  Node B  │
│          │ ◀───── ♥ ─────────── │          │
│  (alive) │                      │  (alive) │
└──────────┘                      └──────────┘
     │              every 1-5 seconds              │
     └─────────────────────────────────────────────┘

Failure Detection:
┌──────────┐                      ┌──────────┐
│  Node A  │ ────── ♥ ────────X   │  Node B  │
│          │         (no response)│   💀     │
│          │                      │  (dead?) │
└──────────┘                      └──────────┘
     │
     │  After N missed heartbeats
     ▼
Mark Node B as failed, trigger failover
```

### Heartbeat Mechanism

```
┌────────────────────────────────────────────────────────────────┐
│                     Heartbeat Timeline                          │
└────────────────────────────────────────────────────────────────┘

Time:    0s    1s    2s    3s    4s    5s    6s    7s    8s
         │     │     │     │     │     │     │     │     │
Node A:  ♥─────♥─────♥─────♥─────♥─────♥─────♥─────♥─────♥
                                         │
Node B:  ♥─────♥─────♥─────X─────X─────X─│─────X─────X
                           │             │
                    Last heartbeat    Timeout (3 missed)
                    at t=2s           │
                                      ▼
                              Node B marked DEAD
                              Failover initiated

Parameters:
- Heartbeat interval: 1 second
- Failure threshold: 3 missed heartbeats
- Timeout: 3 seconds
```

### Types of Heartbeat Patterns

**1. Push-based (Active):**
```
┌─────────┐         ┌─────────┐
│ Worker  │ ──♥──▶  │ Monitor │
│         │ ──♥──▶  │         │
│         │ ──♥──▶  │         │
└─────────┘         └─────────┘
Worker actively sends heartbeats
```

**2. Pull-based (Passive):**
```
┌─────────┐         ┌─────────┐
│ Worker  │ ◀──?──  │ Monitor │
│         │ ──ok─▶  │         │
│         │ ◀──?──  │         │
│         │ ──ok─▶  │         │
└─────────┘         └─────────┘
Monitor polls workers for status
```

**3. Gossip-based (Peer-to-Peer):**
```
    ┌─────────┐
    │ Node A  │
    └────┬────┘
    ♥    │    ♥
   ╱     │     ╲
  ▼      ▼      ▼
┌───┐  ┌───┐  ┌───┐
│ B │──│ C │──│ D │
└───┘  └───┘  └───┘
Each node shares health info with random peers
```

### Key Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| **Heartbeat interval** | Time between heartbeats | 1-5 seconds |
| **Failure threshold** | Missed heartbeats before failure | 3-5 |
| **Timeout** | interval × threshold | 3-15 seconds |
| **Jitter** | Random delay to prevent thundering herd | 0-20% of interval |

### Trade-offs

```
Fast Detection (short interval, low threshold)
├── ✓ Quick failover
├── ✗ False positives (network hiccups → wrongly marked dead)
└── ✗ Higher network/CPU overhead

Slow Detection (long interval, high threshold)
├── ✓ Fewer false positives
├── ✓ Lower overhead
└── ✗ Slow failover, longer downtime
```

### Use Cases

| System | Heartbeat Usage |
|--------|-----------------|
| **Kubernetes** | kubelet sends heartbeats to control plane |
| **ZooKeeper** | Session heartbeats; missed → ephemeral nodes deleted |
| **Cassandra** | Gossip protocol with heartbeat-like φ accrual detector |
| **Load Balancers** | Health checks to backend servers |

---

