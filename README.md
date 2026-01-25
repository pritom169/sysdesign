# System Design Fundamentals - Walkthrough

This document provides a brief walkthrough of the core system design concepts covered in the `00-sysdesign-fundementals` folder. These topics form the foundation for any system design interview or real-world distributed systems work.

---

## Table of Contents

1. [Load Balancing](#load-balancing)
2. [Scalability & Distributed Systems](#scalability--distributed-systems)
3. [Networking Fundamentals](#networking-fundamentals)
4. [Caching](#caching)
5. [Content Delivery Networks (CDN)](#content-delivery-networks-cdn)
6. [Data Management & Storage](#data-management--storage)
7. [Communication Patterns](#communication-patterns)
8. [Distributed System Patterns](#distributed-system-patterns)
9. [Security & Miscellaneous](#security--miscellaneous)

---

## Load Balancing

📁 **Covered in:** [01-load-balancing](00-sysdesign-fundementals/01-load-balancing/README.md) | [02-load-balancing-algorithms](00-sysdesign-fundementals/02-load-balancing-algorithms/README.md) | [03-uses-of-load-balancing](00-sysdesign-fundementals/03-uses-of-load-balancing/README.md) | [04-load-balancer-types](00-sysdesign-fundementals/04-load-balancer-types/README.md)

**What it is:** Load balancing distributes incoming traffic across multiple servers to ensure high availability, reliability, and performance.

**Key Concepts:**
- **Health Checks** - Periodic tests to determine server availability
- **Session Persistence** - Ensuring requests from the same client go to the same server
- **SSL/TLS Termination** - Offloading decryption at the load balancer level

**Algorithms:**
| Algorithm | Best For |
|-----------|----------|
| Round Robin | Equal server capacity |
| Weighted Round Robin | Varied server capacity |
| Least Connections | Long-lived connections |
| IP Hash | Session persistence |

**Placement:** Load balancers can be placed between clients → web servers → application servers → databases.

---

## Scalability & Distributed Systems

📁 **Covered in:** [05-scalability-and-performance](00-sysdesign-fundementals/05-scalability-and-performance/README.md) | [06-key-characteristics-of-distributed-systems](00-sysdesign-fundementals/06-key-characteristics-of-distributed-systems/README.md) | [07-latency-and-performance](00-sysdesign-fundementals/07-latency-and-performance/README.md) | [08-concurrency-and-coordination](00-sysdesign-fundementals/08-concurrency-and-coordination/README.md) | [09-monitoring-and-observability](00-sysdesign-fundementals/09-monitoring-and-observability/README.md) | [10-resilience-and-error-handling](00-sysdesign-fundementals/10-resilience-and-error-handling/README.md) | [11-fault-tolerance-vs.-high-availability](00-sysdesign-fundementals/11-fault-tolerance-vs.-high-availability/README.md)

**Scaling Types:**
- **Horizontal Scaling (Scale Out)** - Add more machines (Cassandra, MongoDB)
- **Vertical Scaling (Scale Up)** - Upgrade existing machines (MySQL)

**Key Characteristics:**
| Characteristic | Description |
|----------------|-------------|
| **Scalability** | Ability to handle growing workloads |
| **Availability** | System uptime and accessibility |
| **Reliability** | Consistent performance over time |
| **Efficiency** | Resource utilization and latency |
| **Manageability** | Ease of operation and maintenance |

**Consistency Models:**
- **Strong** - All nodes see the same data at all times
- **Weak** - Temporary inconsistencies allowed
- **Eventual** - All replicas converge over time

---

## Networking Fundamentals

📁 **Covered in:** [12-network-essentials](00-sysdesign-fundementals/12-network-essentials/README.md) | [13-tcp-vs.-udp](00-sysdesign-fundementals/13-tcp-vs.-udp/README.md) | [14-url-vs.-uri-vs.-urn](00-sysdesign-fundementals/14-url-vs.-uri-vs.-urn/README.md) | [15-dns-(domain-name-system)](00-sysdesign-fundementals/15-dns-(domain-name-system)/README.md)

**TCP vs UDP:**
| Aspect | TCP | UDP |
|--------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | Best effort |
| Use Cases | Web, email, file transfer | Streaming, gaming, DNS |

**DNS (Domain Name System):**
- Translates domain names to IP addresses
- Hierarchical structure: Root → TLD → Authoritative
- Caching at multiple levels for performance

---

## Caching

📁 **Covered in:** [16-caching](00-sysdesign-fundementals/16-caching/README.md) through [24-interview-checklist](00-sysdesign-fundementals/24-interview-checklist/README.md)

**What it is:** Storing frequently accessed data in fast storage (memory) to reduce latency and database load.

**Cache Strategies:**
| Strategy | Write Behavior | Read Behavior |
|----------|---------------|---------------|
| **Cache-Aside** | App manages both | Read: cache first, then DB |
| **Write-Through** | Write to cache + DB | Read: from cache |
| **Write-Behind** | Write to cache, async to DB | Read: from cache |
| **Read-Through** | Cache loads from DB | Read: cache handles miss |

**Eviction Policies:**
- **LRU** (Least Recently Used) - Most common
- **LFU** (Least Frequently Used)
- **FIFO** (First In, First Out)
- **TTL** (Time To Live)

**Cache Invalidation Challenges:**
- Cache coherence across distributed nodes
- Thundering herd problem
- Cold start issues

---

## Content Delivery Networks (CDN)

📁 **Covered in:** [25-cdn-(content-delivery-network)](00-sysdesign-fundementals/25-cdn-(content-delivery-network)/README.md) | [26-cdn-use-cases-and-content-types](00-sysdesign-fundementals/26-cdn-use-cases-and-content-types/README.md) | [27-cdn-challenges](00-sysdesign-fundementals/27-cdn-challenges/README.md) | [28-cdn-interview-checklist](00-sysdesign-fundementals/28-cdn-interview-checklist/README.md)

**What it is:** Geographically distributed network of servers that cache content closer to users.

**Content Types:**
- Static: Images, CSS, JS, videos
- Dynamic: API responses (with short TTL)

**Key Benefits:**
- Reduced latency (geographic proximity)
- Lower origin server load
- DDoS protection
- High availability

---

## Data Management & Storage

📁 **Covered in:** [29-data-partitioning](00-sysdesign-fundementals/29-data-partitioning/README.md) | [30-redundancy-and-replication](00-sysdesign-fundementals/30-redundancy-and-replication/README.md) | [31-proxy-servers](00-sysdesign-fundementals/31-proxy-servers/README.md) | [32-cap-theorem](00-sysdesign-fundementals/32-cap-theorem/README.md) | [33-databases](00-sysdesign-fundementals/33-databases/README.md) | [34-database-indexing](00-sysdesign-fundementals/34-database-indexing/README.md) | [35-bloom-filters](00-sysdesign-fundementals/35-bloom-filters/README.md)

### CAP Theorem

A distributed system can only guarantee **two of three** properties:
- **Consistency** - Every read gets the latest write
- **Availability** - Every request gets a response
- **Partition Tolerance** - System works despite network failures

**Reality:** P is mandatory in distributed systems. The real choice is C vs A during partitions.

| System Type | Examples | Use Case |
|-------------|----------|----------|
| **CP** | Zookeeper, etcd, MongoDB | Financial data, locks |
| **AP** | Cassandra, DynamoDB | Shopping carts, social feeds |

### Databases

**SQL vs NoSQL:**
| Aspect | SQL | NoSQL |
|--------|-----|-------|
| Schema | Fixed | Flexible |
| Scaling | Vertical | Horizontal |
| Transactions | ACID | BASE |
| Best For | Complex queries | Scale, flexibility |

**NoSQL Types:**
- **Document** (MongoDB) - JSON documents
- **Key-Value** (Redis) - Simple key-based access
- **Column-Family** (Cassandra) - Time-series, analytics
- **Graph** (Neo4j) - Relationship-heavy data

### Data Partitioning (Sharding)

**Methods:**
- **Horizontal** - Split rows across shards
- **Vertical** - Split columns/tables
- **Directory-based** - Lookup service determines shard

**Partitioning Schemes:**
- Hash-based (consistent hashing)
- Range-based
- Geographic

---

## Communication Patterns

📁 **Covered in:** [36-long-polling-vs.-websockets-vs.-server-sent-events](00-sysdesign-fundementals/36-long-polling-vs.-websockets-vs.-server-sent-events/README.md)

| Pattern | Direction | Use Case |
|---------|-----------|----------|
| **Long Polling** | Client → Server | Simple real-time (notifications) |
| **WebSockets** | Bidirectional | Chat, gaming, live collaboration |
| **Server-Sent Events** | Server → Client | Live feeds, dashboards |

---

## Distributed System Patterns

📁 **Covered in:** [37-quorum](00-sysdesign-fundementals/37-quorum/README.md) | [38-heartbeat](00-sysdesign-fundementals/38-heartbeat/README.md) | [39-checksum](00-sysdesign-fundementals/39-checksum/README.md) | [40-leader-and-follower](00-sysdesign-fundementals/40-leader-and-follower/README.md)

### Quorum
Minimum number of nodes that must agree for an operation to succeed.
- **Formula:** `W + R > N` guarantees consistency
- N = total replicas, W = write acknowledgments, R = read nodes

### Heartbeat
Periodic signals between nodes to detect failures.
- Failure detection through missed heartbeats
- Enables automatic failover

### Checksum
Data integrity verification using hash functions.
- Detects data corruption during transfer/storage
- Used in replication, networking, storage systems

### Leader-Follower
One node (leader) handles writes; followers replicate and serve reads.
- Leader election on failure (Raft, Paxos)
- Trade-off between consistency and availability

---

## Security & Miscellaneous

📁 **Covered in:** [41-security](00-sysdesign-fundementals/41-security/README.md) | [42-distributed-messaging-system](00-sysdesign-fundementals/42-distributed-messaging-system/README.md) | [43-distributed-file-systems](00-sysdesign-fundementals/43-distributed-file-systems/README.md) | [44-misc-concepts](00-sysdesign-fundementals/44-misc-concepts/README.md)

### Security Essentials
- Authentication vs Authorization
- Encryption (at rest, in transit)
- Rate limiting and DDoS protection
- API security (OAuth, JWT, API keys)

### Distributed Messaging
- **Message Queues** (RabbitMQ, SQS) - Point-to-point
- **Pub/Sub** (Kafka, SNS) - One-to-many
- Guarantees: At-most-once, at-least-once, exactly-once

### Distributed File Systems
- HDFS, GFS architecture
- Data replication and fault tolerance
- Metadata management

---

## Quick Reference: Interview Priorities

| Priority | Topic | Why It Matters |
|----------|-------|----------------|
| 🔴 High | CAP Theorem | Foundation of distributed systems trade-offs |
| 🔴 High | Databases (SQL/NoSQL) | Every system needs data storage |
| 🔴 High | Caching | Critical for performance at scale |
| 🔴 High | Load Balancing | Essential for availability |
| 🟡 Medium | Data Partitioning | Required for horizontal scaling |
| 🟡 Medium | Replication | High availability and disaster recovery |
| 🟡 Medium | Messaging Systems | Async communication, decoupling |
| 🟢 Lower | CDN | Important but often abstracted |
| 🟢 Lower | Bloom Filters | Specific use cases |

---

## How to Use This Material

1. **Start with fundamentals:** Load balancing, caching, databases
2. **Understand trade-offs:** CAP theorem, consistency models
3. **Learn patterns:** Quorum, leader-follower, heartbeat
4. **Practice application:** Apply concepts to real system design problems

Each topic folder contains detailed explanations with diagrams, examples, and interview-specific insights.