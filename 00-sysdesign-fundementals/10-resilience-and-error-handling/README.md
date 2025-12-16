## Resilience and Error Handling


Resilience and error handling help minimize the impact of failures and ensure that the system can recover gracefully from unexpected events.

### A. Fault Tolerance

Fault tolerance is the architectural capacity of a system to continue delivering its intended service, possibly at a reduced level, rather than failing completely when some part of the system fails.

Designing a fault-tolerant system involves incorporating redundancy at various levels (data, services, nodes) and implementing strategies like replication, sharding, and load balancing to ensure that the system can withstand failures

### B. Graceful Degradation

When a system faces extreme load or component failure, it must prioritize the "Critical User Journey." Graceful degradation ensures that the system fails progressively rather than catastrophically.

System health is not a boolean (up/down). If a recommendation engine fails on an e-commerce site, the "Buy" button (core functionality) must still work, perhaps displaying a generic list of items instead of personalized ones.

### C. Retry and Backoff Strategies

Distributed systems frequently encounter "transient faults"—temporary glitches like network jitter or momentary service unavailability. Handling these requires a balance between persistence and resource conservation.

Before implementing retries, operations must be idempotent, meaning performing the same operation multiple times yields the same result (e.g., charging a credit card only once despite three retry attempts).

### D. Error Handling and Reporting

Resilience requires visibility. It is not enough to handle errors; the system must communicate the context of the error to engineers to facilitate Root Cause Analysis (RCA).

### E. Chaos Engineering

Chaos engineering is the practice of intentionally injecting failures into a distributed system to test its resilience and identify weaknesses. By simulating real-world failure scenarios, you can evaluate the system's ability to recover and adapt

