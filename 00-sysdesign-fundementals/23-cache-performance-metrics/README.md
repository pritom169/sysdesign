## Cache Performance Metrics


Monitor these metrics to understand cache health:

| Metric | Formula | Target |
|--------|---------|--------|
| **Hit Rate** | Hits / (Hits + Misses) | > 90% for most workloads |
| **Miss Rate** | Misses / (Hits + Misses) | < 10% |
| **Latency (p50/p99)** | Time to serve request | p99 < 5ms for Redis |
| **Eviction Rate** | Evictions per second | Low (if high, increase memory) |
| **Memory Usage** | Used / Total | 70-85% (leave room for spikes) |

**Hit rate interpretation:**
- **< 50%:** Cache is ineffective. Check key design, TTL, or access patterns.
- **50-80%:** Room for improvement. Tune TTL or increase capacity.
- **> 90%:** Healthy cache. Monitor for changes.

---

