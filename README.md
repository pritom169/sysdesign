# System Design Fundamentals - Walkthrough

This guide covers the essential building blocks for designing scalable, reliable, and performant distributed systems. Whether you're preparing for system design interviews or building production systems, these 44 topics in `00-sysdesign-fundementals` provide the foundational knowledge you need—from load balancing and caching to databases, CAP theorem, and distributed patterns.

---

## Topics Overview

| # | Topic | Folder | Key Concept |
|---|-------|--------|-------------|
| 01 | Load Balancing | [01-load-balancing](00-sysdesign-fundementals/01-load-balancing/README.md) | Distributes traffic across servers for availability |
| 02 | Load Balancing Algorithms | [02-load-balancing-algorithms](00-sysdesign-fundementals/02-load-balancing-algorithms/README.md) | Round Robin, Least Connections, IP Hash, Weighted |
| 03 | Uses of Load Balancing | [03-uses-of-load-balancing](00-sysdesign-fundementals/03-uses-of-load-balancing/README.md) | Web tier, app tier, database tier placement |
| 04 | Load Balancer Types | [04-load-balancer-types](00-sysdesign-fundementals/04-load-balancer-types/README.md) | L4 (transport) vs L7 (application) load balancers |
| 05 | Scalability and Performance | [05-scalability-and-performance](00-sysdesign-fundementals/05-scalability-and-performance/README.md) | Horizontal vs vertical scaling strategies |
| 06 | Key Characteristics of Distributed Systems | [06-key-characteristics-of-distributed-systems](00-sysdesign-fundementals/06-key-characteristics-of-distributed-systems/README.md) | Scalability, availability, reliability, efficiency |
| 07 | Latency and Performance | [07-latency-and-performance](00-sysdesign-fundementals/07-latency-and-performance/README.md) | Response time optimization techniques |
| 08 | Concurrency and Coordination | [08-concurrency-and-coordination](00-sysdesign-fundementals/08-concurrency-and-coordination/README.md) | Locks, semaphores, distributed coordination |
| 09 | Monitoring and Observability | [09-monitoring-and-observability](00-sysdesign-fundementals/09-monitoring-and-observability/README.md) | Metrics, logging, tracing, alerting |
| 10 | Resilience and Error Handling | [10-resilience-and-error-handling](00-sysdesign-fundementals/10-resilience-and-error-handling/README.md) | Circuit breakers, retries, fallbacks |
| 11 | Fault Tolerance vs High Availability | [11-fault-tolerance-vs.-high-availability](00-sysdesign-fundementals/11-fault-tolerance-vs.-high-availability/README.md) | Redundancy, failover, recovery strategies |
| 12 | Network Essentials | [12-network-essentials](00-sysdesign-fundementals/12-network-essentials/README.md) | OSI model, protocols, networking basics |
| 13 | TCP vs UDP | [13-tcp-vs.-udp](00-sysdesign-fundementals/13-tcp-vs.-udp/README.md) | Connection-oriented vs connectionless protocols |
| 14 | URL vs URI vs URN | [14-url-vs.-uri-vs.-urn](00-sysdesign-fundementals/14-url-vs.-uri-vs.-urn/README.md) | Resource identification and addressing |
| 15 | DNS | [15-dns-(domain-name-system)](00-sysdesign-fundementals/15-dns-(domain-name-system)/README.md) | Domain name resolution, DNS hierarchy |
| 16 | Caching | [16-caching](00-sysdesign-fundementals/16-caching/README.md) | Introduction to caching concepts |
| 17 | What is Caching | [17-what-is-caching](00-sysdesign-fundementals/17-what-is-caching/README.md) | Cache fundamentals and benefits |
| 18 | Cache Strategies | [18-cache-strategies](00-sysdesign-fundementals/18-cache-strategies/README.md) | Write-through, write-behind, cache-aside |
| 19 | Cache Invalidation | [19-cache-invalidation](00-sysdesign-fundementals/19-cache-invalidation/README.md) | TTL, event-based, manual invalidation |
| 20 | Cache Eviction Policies | [20-cache-eviction-policies](00-sysdesign-fundementals/20-cache-eviction-policies/README.md) | LRU, LFU, FIFO, TTL-based eviction |
| 21 | Caching Challenges | [21-caching-challenges](00-sysdesign-fundementals/21-caching-challenges/README.md) | Thundering herd, cold start, coherence |
| 22 | Distributed Caching | [22-distributed-caching](00-sysdesign-fundementals/22-distributed-caching/README.md) | Redis, Memcached, distributed cache patterns |
| 23 | Cache Performance Metrics | [23-cache-performance-metrics](00-sysdesign-fundementals/23-cache-performance-metrics/README.md) | Hit ratio, miss ratio, latency metrics |
| 24 | Interview Checklist | [24-interview-checklist](00-sysdesign-fundementals/24-interview-checklist/README.md) | Caching interview preparation guide |
| 25 | CDN | [25-cdn-(content-delivery-network)](00-sysdesign-fundementals/25-cdn-(content-delivery-network)/README.md) | Geographically distributed content delivery |
| 26 | CDN Use Cases and Content Types | [26-cdn-use-cases-and-content-types](00-sysdesign-fundementals/26-cdn-use-cases-and-content-types/README.md) | Static assets, video streaming, API caching |
| 27 | CDN Challenges | [27-cdn-challenges](00-sysdesign-fundementals/27-cdn-challenges/README.md) | Cache invalidation, cost, origin protection |
| 28 | CDN Interview Checklist | [28-cdn-interview-checklist](00-sysdesign-fundementals/28-cdn-interview-checklist/README.md) | CDN interview preparation guide |
| 29 | Data Partitioning | [29-data-partitioning](00-sysdesign-fundementals/29-data-partitioning/README.md) | Horizontal, vertical, directory-based sharding |
| 30 | Redundancy and Replication | [30-redundancy-and-replication](00-sysdesign-fundementals/30-redundancy-and-replication/README.md) | Data copies for availability and durability |
| 31 | Proxy Servers | [31-proxy-servers](00-sysdesign-fundementals/31-proxy-servers/README.md) | Forward proxy, reverse proxy, use cases |
| 32 | CAP Theorem | [32-cap-theorem](00-sysdesign-fundementals/32-cap-theorem/README.md) | Consistency, Availability, Partition Tolerance |
| 33 | Databases | [33-databases](00-sysdesign-fundementals/33-databases/README.md) | SQL vs NoSQL, ACID vs BASE, database types |
| 34 | Database Indexing | [34-database-indexing](00-sysdesign-fundementals/34-database-indexing/README.md) | B-tree, hash indexes, query optimization |
| 35 | Bloom Filters | [35-bloom-filters](00-sysdesign-fundementals/35-bloom-filters/README.md) | Probabilistic data structure for membership |
| 36 | Long Polling vs WebSockets vs SSE | [36-long-polling-vs.-websockets-vs.-server-sent-events](00-sysdesign-fundementals/36-long-polling-vs.-websockets-vs.-server-sent-events/README.md) | Real-time communication patterns |
| 37 | Quorum | [37-quorum](00-sysdesign-fundementals/37-quorum/README.md) | Minimum nodes for consensus (W + R > N) |
| 38 | Heartbeat | [38-heartbeat](00-sysdesign-fundementals/38-heartbeat/README.md) | Failure detection via periodic signals |
| 39 | Checksum | [39-checksum](00-sysdesign-fundementals/39-checksum/README.md) | Data integrity verification |
| 40 | Leader and Follower | [40-leader-and-follower](00-sysdesign-fundementals/40-leader-and-follower/README.md) | Leader election, replication patterns |
| 41 | Security | [41-security](00-sysdesign-fundementals/41-security/README.md) | Auth, encryption, rate limiting, API security |
| 42 | Distributed Messaging System | [42-distributed-messaging-system](00-sysdesign-fundementals/42-distributed-messaging-system/README.md) | Message queues, pub/sub, Kafka, RabbitMQ |
| 43 | Distributed File Systems | [43-distributed-file-systems](00-sysdesign-fundementals/43-distributed-file-systems/README.md) | HDFS, GFS, data replication |
| 44 | Misc Concepts | [44-misc-concepts](00-sysdesign-fundementals/44-misc-concepts/README.md) | Additional system design concepts |

---

## Topics by Category

### Load Balancing

| Topic | Key Points | When to Use |
|-------|------------|-------------|
| [Load Balancing](00-sysdesign-fundementals/01-load-balancing/README.md) | Distributes traffic, health checks, session persistence | Any multi-server architecture |
| [Algorithms](00-sysdesign-fundementals/02-load-balancing-algorithms/README.md) | Round Robin, Weighted, Least Connections, IP Hash | Based on server capacity and session needs |
| [Uses](00-sysdesign-fundementals/03-uses-of-load-balancing/README.md) | Web tier, app tier, DB tier placement | At every layer for full redundancy |
| [Types](00-sysdesign-fundementals/04-load-balancer-types/README.md) | L4 (transport), L7 (application) | L4 for speed, L7 for content-based routing |

### Scalability & System Characteristics

| Topic | Key Points | Trade-offs |
|-------|------------|------------|
| [Scalability](00-sysdesign-fundementals/05-scalability-and-performance/README.md) | Horizontal (add machines) vs Vertical (bigger machine) | Cost vs complexity |
| [Distributed System Characteristics](00-sysdesign-fundementals/06-key-characteristics-of-distributed-systems/README.md) | Scalability, availability, reliability, efficiency | Balance based on requirements |
| [Latency](00-sysdesign-fundementals/07-latency-and-performance/README.md) | Network, processing, queue delays | Speed vs consistency |
| [Concurrency](00-sysdesign-fundementals/08-concurrency-and-coordination/README.md) | Locks, semaphores, distributed coordination | Throughput vs correctness |
| [Monitoring](00-sysdesign-fundementals/09-monitoring-and-observability/README.md) | Metrics, logs, traces, alerts | Visibility vs overhead |
| [Resilience](00-sysdesign-fundementals/10-resilience-and-error-handling/README.md) | Circuit breakers, retries, fallbacks | Availability vs complexity |
| [Fault Tolerance vs HA](00-sysdesign-fundementals/11-fault-tolerance-vs.-high-availability/README.md) | Redundancy, failover, recovery | Cost vs uptime guarantee |

### Networking

| Topic | Key Points | Use Case |
|-------|------------|----------|
| [Network Essentials](00-sysdesign-fundementals/12-network-essentials/README.md) | OSI model, IP, ports, protocols | Foundation for all networking |
| [TCP vs UDP](00-sysdesign-fundementals/13-tcp-vs.-udp/README.md) | TCP: reliable, ordered; UDP: fast, unreliable | TCP for web; UDP for streaming/gaming |
| [URL vs URI vs URN](00-sysdesign-fundementals/14-url-vs.-uri-vs.-urn/README.md) | URI = identifier; URL = locator; URN = name | Resource addressing |
| [DNS](00-sysdesign-fundementals/15-dns-(domain-name-system)/README.md) | Domain → IP resolution, hierarchy, caching | Every web request |

### Caching

| Topic | Key Points | When to Apply |
|-------|------------|---------------|
| [Caching Intro](00-sysdesign-fundementals/16-caching/README.md) | Store frequently accessed data in fast storage | Read-heavy workloads |
| [What is Caching](00-sysdesign-fundementals/17-what-is-caching/README.md) | Reduces latency, lowers DB load | High-traffic systems |
| [Strategies](00-sysdesign-fundementals/18-cache-strategies/README.md) | Cache-aside, write-through, write-behind, read-through | Based on read/write patterns |
| [Invalidation](00-sysdesign-fundementals/19-cache-invalidation/README.md) | TTL, event-based, manual purge | When data changes |
| [Eviction Policies](00-sysdesign-fundementals/20-cache-eviction-policies/README.md) | LRU, LFU, FIFO, TTL | When cache is full |
| [Challenges](00-sysdesign-fundementals/21-caching-challenges/README.md) | Thundering herd, cold start, coherence | Distributed systems |
| [Distributed Caching](00-sysdesign-fundementals/22-distributed-caching/README.md) | Redis, Memcached clusters | Multi-node deployments |
| [Performance Metrics](00-sysdesign-fundementals/23-cache-performance-metrics/README.md) | Hit ratio, miss ratio, latency | Monitoring cache health |

### CDN (Content Delivery Network)

| Topic | Key Points | Best For |
|-------|------------|----------|
| [CDN Basics](00-sysdesign-fundementals/25-cdn-(content-delivery-network)/README.md) | Geographically distributed cache servers | Global user base |
| [Use Cases](00-sysdesign-fundementals/26-cdn-use-cases-and-content-types/README.md) | Static assets, video, API responses | Media-heavy applications |
| [Challenges](00-sysdesign-fundementals/27-cdn-challenges/README.md) | Invalidation, cost, cache poisoning | Large-scale deployments |

### Data Management

| Topic | Key Points | When to Use |
|-------|------------|-------------|
| [Data Partitioning](00-sysdesign-fundementals/29-data-partitioning/README.md) | Horizontal, vertical, hash-based, range-based | Data exceeds single node |
| [Redundancy & Replication](00-sysdesign-fundementals/30-redundancy-and-replication/README.md) | Primary-replica, multi-master, sync vs async | HA and disaster recovery |
| [Proxy Servers](00-sysdesign-fundementals/31-proxy-servers/README.md) | Forward (client-side), reverse (server-side) | Security, caching, load distribution |
| [CAP Theorem](00-sysdesign-fundementals/32-cap-theorem/README.md) | Pick 2: Consistency, Availability, Partition Tolerance | Distributed database design |
| [Databases](00-sysdesign-fundementals/33-databases/README.md) | SQL (ACID) vs NoSQL (BASE), types | Based on data model and scale |
| [Indexing](00-sysdesign-fundementals/34-database-indexing/README.md) | B-tree, hash, composite indexes | Query optimization |
| [Bloom Filters](00-sysdesign-fundementals/35-bloom-filters/README.md) | Probabilistic membership test | Avoiding expensive lookups |

### Communication Patterns

| Pattern | Direction | Latency | Best For |
|---------|-----------|---------|----------|
| [Long Polling](00-sysdesign-fundementals/36-long-polling-vs.-websockets-vs.-server-sent-events/README.md) | Client → Server | Higher | Simple real-time (notifications) |
| [WebSockets](00-sysdesign-fundementals/36-long-polling-vs.-websockets-vs.-server-sent-events/README.md) | Bidirectional | Lowest | Chat, gaming, collaboration |
| [SSE](00-sysdesign-fundementals/36-long-polling-vs.-websockets-vs.-server-sent-events/README.md) | Server → Client | Low | Live feeds, dashboards |

### Distributed System Patterns

| Pattern | Purpose | Example Use |
|---------|---------|-------------|
| [Quorum](00-sysdesign-fundementals/37-quorum/README.md) | Consensus for reads/writes (W + R > N) | Distributed databases |
| [Heartbeat](00-sysdesign-fundementals/38-heartbeat/README.md) | Failure detection via periodic signals | Cluster health monitoring |
| [Checksum](00-sysdesign-fundementals/39-checksum/README.md) | Data integrity verification | Replication, file transfer |
| [Leader-Follower](00-sysdesign-fundementals/40-leader-and-follower/README.md) | Single writer, multiple readers | Database replication |

### Security & Systems

| Topic | Key Points | Importance |
|-------|------------|------------|
| [Security](00-sysdesign-fundementals/41-security/README.md) | AuthN/AuthZ, encryption, rate limiting | Every production system |
| [Distributed Messaging](00-sysdesign-fundementals/42-distributed-messaging-system/README.md) | Queues (RabbitMQ), Pub/Sub (Kafka) | Async, decoupled systems |
| [Distributed File Systems](00-sysdesign-fundementals/43-distributed-file-systems/README.md) | HDFS, GFS architecture | Big data storage |
| [Misc Concepts](00-sysdesign-fundementals/44-misc-concepts/README.md) | Additional patterns and concepts | Supplementary knowledge |

---
