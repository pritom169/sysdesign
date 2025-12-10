## Interview Checklist


| Topic | Key Points to Mention |
|-------|----------------------|
| **Why cache** | Reduce latency, offload DB, enable scale |
| **Cache-aside vs read-through** | Application vs cache manages loading |
| **Write strategies** | Write-through (consistent), write-behind (fast), write-around (write-heavy) |
| **Invalidation** | TTL (simple), event-based (consistent), version-based (assets) |
| **Eviction** | LRU (default), LFU (stable hot set), understand Redis policies |
| **Challenges** | Stampede (locking), penetration (bloom filter), avalanche (jitter) |
| **Distributed** | Consistent hashing, Redis Cluster vs Memcached |
| **Metrics** | Hit rate > 90%, p99 latency, eviction rate |

### Common Interview Questions

**"How would you cache user sessions?"**
- Use Redis with `allkeys-lru` eviction
- Key: `session:{session_id}`, Value: user data (JSON)
- TTL: 30 minutes, refresh on activity
- Consider: sticky sessions vs distributed cache trade-off

**"Design a cache for a news feed"**
- Cache recent posts per user: `feed:{user_id}`
- Cache individual posts: `post:{post_id}`
- Invalidation: Event-driven when new post created
- Challenge: Fan-out on write vs fan-out on read

**"How do you handle cache warming after deployment?"**
- Pre-populate cache before routing traffic
- Gradual traffic shift (canary deployment)
- Accept higher miss rate initially, monitor hit rate recovery
- Consider: lazy loading vs eager loading trade-offs

**"Cache consistency between microservices?"**
- Shared Redis cluster vs per-service cache
- Event bus for invalidation (Kafka, Redis Pub/Sub)
- Accept eventual consistency with short TTL
- Consider: who owns the data vs who caches it

---

