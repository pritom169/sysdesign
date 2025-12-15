## Cache Eviction Policies


When the cache is full and a new entry needs to be added, the eviction policy determines which existing entry to remove. This is different from invalidation (removing stale data) — eviction removes valid data due to space constraints.

### Common Eviction Policies

#### 1. LRU (Least Recently Used)

Evicts the entry that hasn't been accessed for the longest time.

```
Access pattern: A, B, C, D, A, E (cache size = 4)

[A]                    → A added
[A, B]                 → B added
[A, B, C]              → C added
[A, B, C, D]           → D added (cache full)
[B, C, D, A]           → A accessed (moved to end)
[C, D, A, E]           → E added, B evicted (least recent)
```

| Pros | Cons |
|------|------|
| Good general-purpose policy | Scan pollution (one-time reads evict hot data) |
| Adapts to access patterns | Overhead of maintaining access order |
| Simple to understand | Doesn't consider frequency |

**Best for:** Most general workloads. Default choice for Redis, Memcached.

---

#### 2. LFU (Least Frequently Used)

Evicts the entry with the lowest access count.

```
Access pattern: A, A, A, B, C, D, E (cache size = 4)

A: 3 accesses
B: 1 access
C: 1 access
D: 1 access
E: needs to be added → evict B, C, or D (tied at 1)
```

| Pros | Cons |
|------|------|
| Keeps frequently accessed items | Old popular items never evicted |
| Resistant to scan pollution | New items vulnerable (low count) |

**Best for:** Workloads with stable hot set (e.g., popular products, trending content).

---

#### 3. FIFO (First In, First Out)

Evicts the oldest entry regardless of access pattern.

| Pros | Cons |
|------|------|
| Simplest to implement | Ignores access patterns entirely |
| Predictable behavior | May evict hot items |
| Low overhead | Poor cache efficiency |

**Best for:** Time-sensitive data where age matters more than popularity.

---

#### 4. Random

Randomly selects an entry to evict.

| Pros | Cons |
|------|------|
| Zero overhead | Unpredictable performance |
| No metadata required | May evict hot items |

**Best for:** When simplicity matters more than optimization. Surprisingly effective in some workloads.

---

#### 5. TTL-Based Eviction

Evict entries closest to expiration first.

| Pros | Cons |
|------|------|
| Natural pairing with TTL | Doesn't consider access patterns |
| Proactive space management | May evict frequently accessed items |

---

### Eviction Policy Comparison

| Policy | Hit Rate | Overhead | Best For |
|--------|----------|----------|----------|
| **LRU** | High | Medium | General purpose (default choice) |
| **LFU** | Highest (stable workload) | High | Stable popularity distribution |
| **FIFO** | Low | Very Low | Time-sensitive data |
| **Random** | Medium | Very Low | Simple systems |
| **TTL-Based** | Medium | Low | Data with natural expiration |

### Redis Eviction Policies

Redis offers several eviction policies that combine these strategies:

| Policy | Behavior |
|--------|----------|
| `noeviction` | Return error when memory limit reached |
| `allkeys-lru` | LRU across all keys |
| `volatile-lru` | LRU only among keys with TTL |
| `allkeys-lfu` | LFU across all keys |
| `volatile-lfu` | LFU only among keys with TTL |
| `allkeys-random` | Random eviction |
| `volatile-ttl` | Evict keys with shortest TTL |

**Interview tip:** "We used `allkeys-lru` because our workload was general-purpose with no clear frequency pattern. If we had a stable hot set, we'd consider `allkeys-lfu`."

---

