






### OAuth vs. JWT for Authentication

**OAuth 2.0** is an authorization framework. **JWT** is a token format. They serve different purposes but are often used together.

#### OAuth 2.0

```
┌─────────────────────────────────────────────────────────────────┐
│              OAuth 2.0 Authorization Code Flow                   │
└─────────────────────────────────────────────────────────────────┘

User          App              Auth Server         Resource Server
 │             │                    │                     │
 │── Login ──▶│                    │                     │
 │             │── Redirect ──────▶│                     │
 │◀────────────────── Login Page ──│                     │
 │── Credentials ─────────────────▶│                     │
 │◀───────── Authorization Code ───│                     │
 │             │◀── Code ──────────│                     │
 │             │                    │                     │
 │             │── Code + Secret ─▶│                     │
 │             │◀── Access Token ──│                     │
 │             │                    │                     │
 │             │── Request + Token ─────────────────────▶│
 │             │◀── Protected Data ──────────────────────│
 │◀── Data ───│                    │                     │
```

**OAuth 2.0 Roles:**
- **Resource Owner:** User
- **Client:** Application requesting access
- **Authorization Server:** Issues tokens
- **Resource Server:** Holds protected resources

#### JWT (JSON Web Token)

```
JWT Structure:
┌─────────────────────────────────────────────────────────────────┐
│  Header          Payload            Signature                    │
│  (algorithm)     (claims)           (verification)               │
│                                                                  │
│  eyJhbGc...  .  eyJzdWI...   .   SflKxwRJSM...                  │
│  ─────────      ───────────       ─────────────                  │
│  Base64URL      Base64URL         HMAC/RSA                       │
└─────────────────────────────────────────────────────────────────┘

Decoded:
{                          {
  "alg": "HS256",            "sub": "user123",
  "typ": "JWT"               "name": "John",
}                            "exp": 1735689600,
                             "roles": ["admin"]
                           }
```

**JWT Properties:**
- Self-contained (no database lookup needed)
- Stateless (server doesn't store sessions)
- Signed (tamper-evident)
- Optionally encrypted

#### Comparison

| Aspect | OAuth 2.0 | JWT |
|--------|-----------|-----|
| **What it is** | Authorization framework | Token format |
| **Purpose** | Delegated access | Encode claims |
| **Stateful?** | Can be either | Stateless |
| **Revocation** | Easy (invalidate at server) | Hard (until expiry) |
| **Used for** | Third-party access, SSO | Session tokens, API auth |

**Common Pattern:** OAuth issues JWTs as access tokens.

---

### What is Encryption?

**Encryption** transforms data into unreadable format, reversible only with a key.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Symmetric Encryption                          │
└─────────────────────────────────────────────────────────────────┘

Same key for encryption and decryption:

   Plaintext          Key              Ciphertext
  ┌─────────┐     ┌─────────┐        ┌─────────────┐
  │ "Hello" │ ──▶ │ AES-256 │ ────▶  │ "xK9#mP..." │
  └─────────┘     └─────────┘        └─────────────┘
                       │
                  ┌────┴────┐
                  │Same Key │
                  └────┬────┘
                       │
  ┌─────────┐     ┌─────────┐        ┌─────────────┐
  │ "Hello" │ ◀── │ AES-256 │ ◀────  │ "xK9#mP..." │
  └─────────┘     └─────────┘        └─────────────┘
   Plaintext          Key              Ciphertext

Examples: AES, ChaCha20
Use: Data at rest, fast bulk encryption
```

```
┌─────────────────────────────────────────────────────────────────┐
│                   Asymmetric Encryption                          │
└─────────────────────────────────────────────────────────────────┘

Key pair: Public key (encrypt) + Private key (decrypt)

Sender                                          Receiver
┌────────────────────┐                    ┌────────────────────┐
│ Has: Bob's public  │                    │ Has: Private key   │
│      key           │                    │      Public key    │
└─────────┬──────────┘                    └─────────┬──────────┘
          │                                         │
          │ Encrypt with                           │
          │ Bob's public key                       │
          ▼                                         ▼
   ┌─────────────┐                          ┌─────────────┐
   │ Ciphertext  │ ────── Network ────────▶ │ Ciphertext  │
   └─────────────┘                          └──────┬──────┘
                                                   │
                                            Decrypt with
                                            private key
                                                   │
                                                   ▼
                                            ┌───────────┐
                                            │ Plaintext │
                                            └───────────┘

Examples: RSA, ECC
Use: Key exchange, digital signatures
```

**Encryption Types:**

| Type | Key | Speed | Use Case |
|------|-----|-------|----------|
| **Symmetric** | Same key both sides | Fast | Bulk data |
| **Asymmetric** | Public/private pair | Slow | Key exchange, signatures |
| **Hybrid** | Asymmetric for key, symmetric for data | Best of both | TLS/HTTPS |

**Encryption Layers:**

| Layer | Protects | Example |
|-------|----------|---------|
| **In transit** | Network traffic | TLS/HTTPS |
| **At rest** | Stored data | AES disk encryption |
| **End-to-end** | Data even from service provider | Signal, WhatsApp |

---

### What are DDoS Attacks?

**Distributed Denial of Service (DDoS)** overwhelms a system with traffic from many sources, making it unavailable.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DDoS Attack Structure                         │
└─────────────────────────────────────────────────────────────────┘

           Attacker
              │
              │ Commands
              ▼
    ┌─────────────────────┐
    │   Command & Control │
    │       Server        │
    └─────────┬───────────┘
              │
    ┌─────────┼─────────────────────────┐
    │         │         │         │     │
    ▼         ▼         ▼         ▼     ▼
  ┌───┐     ┌───┐     ┌───┐     ┌───┐ ┌───┐
  │Bot│     │Bot│     │Bot│     │Bot│ │Bot│  ← Compromised machines
  └─┬─┘     └─┬─┘     └─┬─┘     └─┬─┘ └─┬─┘
    │         │         │         │     │
    └────┬────┴────┬────┴────┬────┴──┬──┘
         │         │         │       │
         ▼         ▼         ▼       ▼
    ┌─────────────────────────────────────┐
    │            Target Server            │
    │         💀 OVERWHELMED              │
    └─────────────────────────────────────┘
```

**Attack Types:**

| Layer | Attack Type | Method |
|-------|-------------|--------|
| **Network (L3/4)** | Volumetric | UDP flood, ICMP flood |
| **Network (L3/4)** | Protocol | SYN flood, Ping of Death |
| **Application (L7)** | Application | HTTP flood, Slowloris |

**Common Attacks:**

```
SYN Flood:
┌────────┐         ┌────────┐
│Attacker│         │ Target │
└────┬───┘         └────┬───┘
     │                   │
     │── SYN (fake IP) ─▶│
     │                   │── SYN-ACK (to fake IP, no response)
     │── SYN (fake IP) ─▶│
     │                   │── SYN-ACK (waiting...)
     │── SYN (fake IP) ─▶│
     │       ...         │── SYN-ACK (waiting...)
     │                   │
     │  Target's connection table fills up
     │  Legitimate connections rejected
```

**Mitigation Strategies:**

| Strategy | How It Works |
|----------|--------------|
| **Rate limiting** | Cap requests per IP/user |
| **CDN/Edge network** | Absorb traffic at edge, cache responses |
| **Anycast** | Distribute attack across global PoPs |
| **WAF** | Filter malicious L7 traffic |
| **Blackholing** | Drop all traffic to target IP (last resort) |
| **CAPTCHA** | Verify human users |
| **SYN cookies** | Stateless SYN-ACK, no half-open connections |

```
DDoS Mitigation Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Traffic                                                        │
│      │                                                           │
│      ▼                                                           │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌────────┐  │
│  │   CDN     │───▶│   WAF     │───▶│   Rate    │───▶│ Origin │  │
│  │  (Edge)   │    │           │    │  Limiter  │    │ Server │  │
│  └───────────┘    └───────────┘    └───────────┘    └────────┘  │
│       │                │                 │                       │
│    Absorbs          Filters           Throttles                  │
│    volume           attacks           excess                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Distributed Messaging System

### Introduction to Messaging Systems

A **messaging system** enables asynchronous communication between services by decoupling producers from consumers.

```
┌─────────────────────────────────────────────────────────────────┐
│               Synchronous vs Asynchronous                        │
└─────────────────────────────────────────────────────────────────┘

Synchronous (Direct):
┌──────────┐         ┌──────────┐
│ Service A│────────▶│ Service B│  ← A waits for B
└──────────┘         └──────────┘    A fails if B down

Asynchronous (Message Queue):
┌──────────┐    ┌─────────────┐    ┌──────────┐
│ Service A│───▶│   Message   │───▶│ Service B│
└──────────┘    │    Queue    │    └──────────┘
     │          └─────────────┘         │
 Publishes,         Buffers         Consumes
 continues          messages        when ready
```

**Why Messaging Systems?**

| Benefit | Explanation |
|---------|-------------|
| **Decoupling** | Producers/consumers evolve independently |
| **Buffering** | Handle traffic spikes; consumers process at own pace |
| **Reliability** | Messages persist; survive consumer failures |
| **Scalability** | Add more consumers to scale processing |
| **Async processing** | Producers don't wait for response |

---

### Introduction to Kafka

**Apache Kafka** is a distributed event streaming platform for high-throughput, fault-tolerant messaging.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Kafka Architecture                            │
└─────────────────────────────────────────────────────────────────┘

Producers              Kafka Cluster                    Consumers
┌───────┐         ┌────────────────────────┐          ┌──────────┐
│ App 1 │────────▶│   Topic: orders        │─────────▶│ Consumer │
└───────┘         │  ┌─────────────────┐   │          │ Group A  │
                  │  │ Partition 0     │   │          └──────────┘
┌───────┐         │  │ [0][1][2][3]... │   │
│ App 2 │────────▶│  └─────────────────┘   │          ┌──────────┐
└───────┘         │  ┌─────────────────┐   │─────────▶│ Consumer │
                  │  │ Partition 1     │   │          │ Group B  │
┌───────┐         │  │ [0][1][2][3]... │   │          └──────────┘
│ App 3 │────────▶│  └─────────────────┘   │
└───────┘         │  ┌─────────────────┐   │
                  │  │ Partition 2     │   │
                  │  │ [0][1][2][3]... │   │
                  │  └─────────────────┘   │
                  └────────────────────────┘
```

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| **Topic** | Named feed/category of messages |
| **Partition** | Ordered, immutable log; unit of parallelism |
| **Offset** | Position of message in partition |
| **Producer** | Publishes messages to topics |
| **Consumer** | Reads messages from topics |
| **Consumer Group** | Consumers sharing partition load |
| **Broker** | Kafka server; cluster = multiple brokers |
| **Replication** | Partitions replicated across brokers |

```
Partition Detail:
┌─────────────────────────────────────────────────────────────────┐
│  Partition 0 (immutable append-only log)                         │
│                                                                  │
│  Offset:  0    1    2    3    4    5    6    7                  │
│         ┌────┬────┬────┬────┬────┬────┬────┬────┐               │
│         │ A  │ B  │ C  │ D  │ E  │ F  │ G  │    │◀── New writes │
│         └────┴────┴────┴────┴────┴────┴────┴────┘               │
│                              ▲                                   │
│                      Consumer position                           │
│                      (committed offset)                          │
└─────────────────────────────────────────────────────────────────┘

- Messages immutable once written
- Consumers track their own offset
- Messages retained for configurable period (not deleted on consume)
```

---

### Messaging Patterns

**1. Point-to-Point (Queue):**
```
Each message consumed by exactly one consumer.

Producer ──▶ [Queue] ──▶ Consumer A
                     ├──▶ Consumer B  (competing consumers)
                     └──▶ Consumer C

Message 1 → Consumer A
Message 2 → Consumer B
Message 3 → Consumer C
Message 4 → Consumer A
...

Use: Task distribution, work queues
```

**2. Publish-Subscribe (Pub/Sub):**
```
Each message delivered to all subscribers.

              ┌──▶ Subscriber A (gets all messages)
              │
Producer ──▶ [Topic] ──▶ Subscriber B (gets all messages)
              │
              └──▶ Subscriber C (gets all messages)

Message 1 → A, B, C (all get it)
Message 2 → A, B, C (all get it)

Use: Event broadcasting, notifications
```

**3. Fan-out:**
```
One message triggers multiple independent actions.

                    ┌──▶ Email Service
                    │
Order Created ──▶ [Topic] ──▶ Inventory Service
                    │
                    └──▶ Analytics Service
```

**4. Request-Reply:**
```
┌──────────┐    Request Queue    ┌──────────┐
│ Requester│───────────────────▶│ Responder│
└──────────┘                     └──────────┘
      ▲                                │
      │         Reply Queue            │
      └────────────────────────────────┘

Correlation ID links request to reply.
```

---

### Popular Messaging Queue Systems

| System | Type | Best For |
|--------|------|----------|
| **Kafka** | Distributed log | High-throughput streaming, event sourcing |
| **RabbitMQ** | Message broker | Complex routing, traditional queuing |
| **Amazon SQS** | Managed queue | Simple, serverless, AWS integration |
| **Amazon SNS** | Managed pub/sub | Fan-out, push notifications |
| **Redis Pub/Sub** | In-memory | Low-latency, ephemeral messaging |
| **Apache Pulsar** | Distributed log | Multi-tenancy, tiered storage |

---

### RabbitMQ vs. Kafka vs. ActiveMQ

| Feature | RabbitMQ | Kafka | ActiveMQ |
|---------|----------|-------|----------|
| **Model** | Message broker | Distributed log | Message broker |
| **Protocol** | AMQP, MQTT, STOMP | Custom (TCP) | JMS, AMQP, STOMP |
| **Message ordering** | Per-queue | Per-partition | Per-queue |
| **Throughput** | ~50K msg/sec | ~1M+ msg/sec | ~10K msg/sec |
| **Message retention** | Until consumed | Time/size-based | Until consumed |
| **Replay** | No (message deleted) | Yes (offset seek) | Limited |
| **Routing** | Complex (exchanges) | Topic/partition | Flexible |
| **Use case** | Task queues, RPC | Event streaming, logs | Enterprise integration |

```
RabbitMQ: Smart broker, dumb consumers
┌──────────┐     ┌─────────────────────────┐     ┌──────────┐
│ Producer │────▶│ Exchange ──▶ Queue      │────▶│ Consumer │
└──────────┘     │   (routing logic)       │     └──────────┘
                 └─────────────────────────┘
                 Message deleted after ACK

Kafka: Dumb broker, smart consumers
┌──────────┐     ┌─────────────────────────┐     ┌──────────┐
│ Producer │────▶│ Topic ──▶ Partitions    │────▶│ Consumer │
└──────────┘     │   (append-only log)     │     │ (tracks  │
                 └─────────────────────────┘     │  offset) │
                 Message retained for period     └──────────┘
```

---

### Scalability and Performance

**Kafka Scalability:**
```
Horizontal Scaling via Partitions:
┌─────────────────────────────────────────────────────────────────┐
│  Topic: orders (6 partitions)                                    │
│                                                                  │
│  Consumer Group (3 consumers)                                    │
│                                                                  │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐                    │
│  │Consumer A │  │Consumer B │  │Consumer C │                    │
│  │ P0, P1    │  │ P2, P3    │  │ P4, P5    │                    │
│  └───────────┘  └───────────┘  └───────────┘                    │
│                                                                  │
│  Add Consumer D:                                                 │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                │
│  │Consumer A│ │Consumer B│ │Consumer C│ │Consumer D│             │
│  │ P0, P1   │ │ P2, P3   │ │ P4       │ │ P5       │             │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘                │
│                                                                  │
│  Max parallelism = number of partitions                          │
└─────────────────────────────────────────────────────────────────┘
```

**Performance Tuning:**

| Factor | Impact | Tuning |
|--------|--------|--------|
| **Batch size** | Throughput vs latency | Larger batch = higher throughput, higher latency |
| **Partitions** | Parallelism | More partitions = more consumers possible |
| **Replication factor** | Durability vs latency | Higher = more durable, more write latency |
| **Acks** | Durability vs speed | `acks=all` durable; `acks=1` faster |
| **Compression** | Network/storage vs CPU | gzip/snappy reduce size |

---

## Distributed File Systems

### What is a Distributed File System?

A **Distributed File System (DFS)** stores files across multiple servers, providing unified access as if on a single machine.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Distributed File System                       │
└─────────────────────────────────────────────────────────────────┘

Client View:                         Physical Reality:
┌────────────────┐                   ┌──────────────────────────┐
│   /data/       │                   │  Server 1    Server 2    │
│   ├── file1    │ ◀── Appears ───▶ │  ┌───────┐  ┌───────┐    │
│   ├── file2    │     as single    │  │file1.a│  │file1.b│    │
│   └── file3    │     filesystem   │  │file2  │  │file3  │    │
└────────────────┘                   │  └───────┘  └───────┘    │
                                     │                          │
                                     │  Server 3                │
                                     │  ┌───────┐               │
                                     │  │file1.c│ (replica)     │
                                     │  └───────┘               │
                                     └──────────────────────────┘
```

**Why DFS?**

| Need | How DFS Addresses It |
|------|---------------------|
| **Scale beyond one machine** | Distribute data across nodes |
| **Fault tolerance** | Replicate data; survive node failures |
| **High availability** | No single point of failure |
| **Throughput** | Parallel reads from multiple nodes |
| **Geographic distribution** | Store data close to users |

---

### Architecture of a Distributed File System

**Master-Worker Architecture (HDFS model):**

```
┌─────────────────────────────────────────────────────────────────┐
│                    HDFS Architecture                             │
└─────────────────────────────────────────────────────────────────┘

                      ┌─────────────────────┐
                      │     NameNode        │  ← Metadata server
                      │  (Master/Primary)   │
                      │  - File → Block map │
                      │  - Block → Node map │
                      │  - Namespace        │
                      └──────────┬──────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
            ▼                    ▼                    ▼
    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
    │  DataNode 1  │    │  DataNode 2  │    │  DataNode 3  │
    │  ┌────┬────┐ │    │  ┌────┬────┐ │    │  ┌────┬────┐ │
    │  │B1  │B4  │ │    │  │B1  │B2  │ │    │  │B2  │B3  │ │
    │  ├────┼────┤ │    │  ├────┼────┤ │    │  ├────┼────┤ │
    │  │B3  │B5  │ │    │  │B5  │B6  │ │    │  │B4  │B6  │ │
    │  └────┴────┘ │    │  └────┴────┘ │    │  └────┴────┘ │
    └──────────────┘    └──────────────┘    └──────────────┘
          ↑                   ↑                   ↑
          └───────────────────┴───────────────────┘
                  Heartbeats + Block reports
```

**Read Flow:**
```
1. Client: "I want to read /data/file.txt"
           │
           ▼
2. NameNode: "file.txt = blocks [B1, B2, B3]"
             "B1 → DataNodes [1, 2]"
             "B2 → DataNodes [2, 3]"
             "B3 → DataNodes [1, 3]"
           │
           ▼
3. Client reads blocks directly from DataNodes
   (NameNode not in data path)
```

**Write Flow:**
```
1. Client: "Create /data/newfile.txt"
           │
           ▼
2. NameNode: "OK, allocate blocks, replicate to 3 nodes"
             Returns: DataNode pipeline [DN1 → DN2 → DN3]
           │
           ▼
3. Client: Streams data to DN1
   DN1: Writes locally, forwards to DN2
   DN2: Writes locally, forwards to DN3
   DN3: Writes locally, ACKs back up pipeline
           │
           ▼
4. Client: Tells NameNode "write complete"
```

---

### Key Components of a DFS

| Component | Role | Examples |
|-----------|------|----------|
| **Metadata Server** | Namespace, file→block mapping | HDFS NameNode, GFS Master |
| **Data Servers** | Store actual data blocks | HDFS DataNodes, GFS Chunkservers |
| **Client Library** | Translates file ops to DFS calls | HDFS client, S3 SDK |
| **Replication Manager** | Ensures replicas exist | Background process |
| **Block Manager** | Allocates, tracks blocks | Part of metadata server |

**Block Storage:**
```
Large file split into fixed-size blocks:

file.txt (640 MB)
┌────────────────────────────────────────────────────────────────┐
│                          Original File                          │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼ Split into 128 MB blocks
         ┌──────────────┬──────────────┬──────────────┬────────┐
         │   Block 0    │   Block 1    │   Block 2    │Block 3 │
         │   (128 MB)   │   (128 MB)   │   (128 MB)   │(256 MB)│
         └──────────────┴──────────────┴──────────────┴────────┘
                              │
                              ▼ Replicated (3x default)
         Block 0: DataNode 1, 4, 7
         Block 1: DataNode 2, 5, 8
         Block 2: DataNode 3, 6, 9
         Block 3: DataNode 1, 5, 9
```

**Popular DFS:**

| System | Use Case | Key Feature |
|--------|----------|-------------|
| **HDFS** | Big data (Hadoop) | Batch processing, high throughput |
| **GFS** | Google internal | Inspired HDFS |
| **Ceph** | Object/block/file storage | Unified, no single point of failure |
| **Amazon S3** | Cloud object storage | Durability (11 9s), scalability |
| **GlusterFS** | Scalable NAS | No metadata server (distributed) |

---

## Misc Concepts

### Batch Processing vs. Stream Processing

```
┌─────────────────────────────────────────────────────────────────┐
│                    Batch Processing                              │
└─────────────────────────────────────────────────────────────────┘

Data arrives     Collected over time    Processed together
    │                    │                      │
    ▼                    ▼                      ▼
┌───────┐            ┌───────┐              ┌───────┐
│Event 1│ ────▶      │       │              │       │
├───────┤            │ Batch │   Trigger    │ Job   │
│Event 2│ ────▶      │       │ ─────────▶   │       │
├───────┤            │       │ (scheduled)  │       │
│Event 3│ ────▶      │       │              │       │
└───────┘            └───────┘              └───────┘
     ⏳                                    Results after
   hours                                   hours/days

Examples: Daily reports, ETL pipelines, ML training
Tools: Hadoop MapReduce, Spark batch, AWS Batch


┌─────────────────────────────────────────────────────────────────┐
│                    Stream Processing                             │
└─────────────────────────────────────────────────────────────────┘

Data arrives     Processed immediately    Results in real-time
    │                    │                      │
    ▼                    ▼                      ▼
┌───────┐            ┌───────┐              ┌───────┐
│Event 1│ ─────────▶ │Process│ ─────────▶   │Result │
└───────┘            │ Event │              │   1   │
                     └───────┘              └───────┘
  ⏱️ ms               instant               immediate

Examples: Fraud detection, live dashboards, alerting
Tools: Kafka Streams, Flink, Spark Streaming, Storm
```

| Aspect | Batch | Stream |
|--------|-------|--------|
| **Latency** | High (hours/days) | Low (ms/seconds) |
| **Data scope** | Complete dataset | Unbounded, continuous |
| **Processing** | Scheduled jobs | Continuous |
| **State** | Stateless per job | Often stateful |
| **Throughput** | Higher (optimized) | Lower (per-event overhead) |
| **Use case** | Historical analysis | Real-time reactions |

---

### XML vs. JSON

```
XML:
<?xml version="1.0"?>
<user>
    <id>123</id>
    <name>John</name>
    <roles>
        <role>admin</role>
        <role>user</role>
    </roles>
</user>

JSON:
{
    "id": 123,
    "name": "John",
    "roles": ["admin", "user"]
}
```

| Aspect | XML | JSON |
|--------|-----|------|
| **Verbosity** | High (closing tags) | Low |
| **Readability** | Moderate | High |
| **Parsing** | DOM/SAX parsers | Native in JS, simple libs |
| **Schema** | XSD, DTD | JSON Schema (optional) |
| **Data types** | String only | String, number, bool, null, array, object |
| **Comments** | Supported | Not supported |
| **Attributes** | Supported | Not supported |
| **Use cases** | Enterprise, SOAP, config | REST APIs, web apps, config |

**Interview tip:** JSON is preferred for REST APIs due to simplicity; XML still used in enterprise/legacy systems (SOAP, SAML).

---

### Synchronous vs. Asynchronous Communication

```
┌─────────────────────────────────────────────────────────────────┐
│                    Synchronous                                   │
└─────────────────────────────────────────────────────────────────┘

Service A                              Service B
    │                                      │
    │────── Request ─────────────────────▶│
    │                                      │ Processing...
    │              BLOCKED                 │
    │              waiting...              │
    │                                      │
    │◀───── Response ─────────────────────│
    │                                      │
    ▼ continues                            │


┌─────────────────────────────────────────────────────────────────┐
│                    Asynchronous                                  │
└─────────────────────────────────────────────────────────────────┘

Service A           Queue           Service B
    │                 │                 │
    │── Message ────▶│                 │
    │                 │                 │
    ▼ continues       │                 │
    │              (buffer)             │
    │                 │───────────────▶│
    │                 │                 │ Processing...
    │                 │                 │
    │◀─ Callback/Poll ─────────────────│ (optional)
```

| Aspect | Synchronous | Asynchronous |
|--------|-------------|--------------|
| **Coupling** | Tight | Loose |
| **Latency** | Caller waits | Caller continues |
| **Failure handling** | Immediate error | Retry, dead-letter queue |
| **Complexity** | Simple | More complex (callbacks, queues) |
| **Scalability** | Limited by slowest service | Better (buffering) |
| **Use case** | Read operations, simple CRUD | Heavy processing, notifications |

---

### Push vs. Pull Notification Systems

```
┌─────────────────────────────────────────────────────────────────┐
│                    Push Notification                             │
└─────────────────────────────────────────────────────────────────┘

Server initiates delivery:

      ┌──────────┐
      │  Server  │
      └────┬─────┘
           │
    ┌──────┴──────┐──────────┐
    │             │          │
    ▼             ▼          ▼
┌───────┐   ┌───────┐   ┌───────┐
│Client │   │Client │   │Client │
│   A   │   │   B   │   │   C   │
└───────┘   └───────┘   └───────┘

Pros: Real-time, no wasted requests
Cons: Requires persistent connection, complex server


┌─────────────────────────────────────────────────────────────────┐
│                    Pull Notification                             │
└─────────────────────────────────────────────────────────────────┘

Client requests updates:

┌───────┐        ┌──────────┐
│Client │──────▶ │  Server  │
│       │  Poll  │          │
│       │◀────── │          │
└───────┘        └──────────┘
    │
    │ (repeat every N seconds)
    ▼

Pros: Simple, stateless server, works through firewalls
Cons: Latency, wasted requests, higher bandwidth
```

| Aspect | Push | Pull |
|--------|------|------|
| **Latency** | Low (immediate) | High (poll interval) |
| **Server load** | Manages connections | Handles repeated requests |
| **Efficiency** | High (only when needed) | Low (many empty responses) |
| **Implementation** | Complex (WebSocket, SSE) | Simple (HTTP) |
| **Scalability** | Challenging | Easier |
| **Use case** | Chat, real-time updates | Email check, RSS feeds |

---

### Microservices vs. Serverless Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices                                 │
└─────────────────────────────────────────────────────────────────┘

┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   User       │   │   Order      │   │  Inventory   │
│   Service    │   │   Service    │   │   Service    │
│              │   │              │   │              │
│ ┌──────────┐ │   │ ┌──────────┐ │   │ ┌──────────┐ │
│ │Container │ │   │ │Container │ │   │ │Container │ │
│ │ (always  │ │   │ │ (always  │ │   │ │ (always  │ │
│ │ running) │ │   │ │ running) │ │   │ │ running) │ │
│ └──────────┘ │   │ └──────────┘ │   │ └──────────┘ │
└──────────────┘   └──────────────┘   └──────────────┘
       │                  │                  │
       └──────── Kubernetes / Docker ────────┘


┌─────────────────────────────────────────────────────────────────┐
│                    Serverless                                    │
└─────────────────────────────────────────────────────────────────┘

        Event                Event                Event
          │                    │                    │
          ▼                    ▼                    ▼
    ┌──────────┐         ┌──────────┐         ┌──────────┐
    │ Function │         │ Function │         │ Function │
    │ (spins   │         │ (spins   │         │ (spins   │
    │  up)     │         │  up)     │         │  up)     │
    └────┬─────┘         └────┬─────┘         └────┬─────┘
         │                    │                    │
         ▼                    ▼                    ▼
    (terminates)         (terminates)         (terminates)

    Pay only for execution time
```

| Aspect | Microservices | Serverless |
|--------|---------------|------------|
| **Deployment unit** | Service (container) | Function |
| **Scaling** | Manual/auto per service | Automatic, per-request |
| **State** | Can be stateful | Stateless |
| **Cold start** | None (always running) | Yes (latency on first call) |
| **Billing** | Per instance-hour | Per execution + duration |
| **Control** | High (infra, runtime) | Low (managed) |
| **Long-running** | Supported | Limited (timeouts) |
| **Vendor lock-in** | Low | Higher |

---

### Message Queues vs. Service Bus

```
┌─────────────────────────────────────────────────────────────────┐
│                    Message Queue                                 │
└─────────────────────────────────────────────────────────────────┘

Simple point-to-point or pub/sub:

Producer ────▶ [Queue] ────▶ Consumer

Features: FIFO, persistence, basic routing


┌─────────────────────────────────────────────────────────────────┐
│                    Service Bus (ESB)                             │
└─────────────────────────────────────────────────────────────────┘

Enterprise features on top of messaging:

┌──────────┐     ┌─────────────────────────────────────┐
│ Service A│────▶│           Service Bus               │
└──────────┘     │  ┌─────────────────────────────────┐│
                 │  │ • Message transformation        ││
┌──────────┐     │  │ • Routing rules                 ││     ┌──────────┐
│ Service B│────▶│  │ • Protocol translation          ││────▶│ Service D│
└──────────┘     │  │ • Transaction support           ││     └──────────┘
                 │  │ • Dead-letter handling          ││
┌──────────┐     │  │ • Sessions / ordering           ││     ┌──────────┐
│ Service C│────▶│  └─────────────────────────────────┘│────▶│ Service E│
└──────────┘     └─────────────────────────────────────┘     └──────────┘
```

| Aspect | Message Queue | Service Bus |
|--------|---------------|-------------|
| **Complexity** | Simple | Complex |
| **Features** | Basic queuing | Transformation, routing, orchestration |
| **Use case** | Decoupling, async tasks | Enterprise integration |
| **Examples** | RabbitMQ, SQS, Redis | Azure Service Bus, MuleSoft, IBM MQ |

---

### Stateful vs. Stateless Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Stateless                                     │
└─────────────────────────────────────────────────────────────────┘

Each request is independent. No server-side session.

Request 1                Load Balancer               Servers
    │                         │              ┌──▶ Server A (any can handle)
    │── GET /user ──────────▶│──────────────┤
    │   + JWT token           │              └──▶ Server B
    │                         │                    Server C
    
Request contains all context needed to process it.


┌─────────────────────────────────────────────────────────────────┐
│                    Stateful                                      │
└─────────────────────────────────────────────────────────────────┘

Server stores client session state.

                         Load Balancer
Request 1                     │                    Server A
    │                         │                   ┌──────────┐
    │── Login ─────────────▶│──────────────────▶│ Session  │
    │                         │ (sticky session)  │ UserA... │
    │                         │                   └──────────┘
Request 2                     │
    │── Action ─────────────▶│───────┐
    │   + Session ID          │       │ Must go to same server!
    │                         │       ▼
                                   Server A
                                   (has session)
```

| Aspect | Stateless | Stateful |
|--------|-----------|----------|
| **Scalability** | Easy (add servers) | Hard (session affinity) |
| **Failover** | Simple (any server) | Complex (session replication) |
| **Performance** | May be slower (reconstruct state) | Faster (state cached) |
| **Memory** | Low per-server | High (storing sessions) |
| **Examples** | REST APIs, JWT auth | WebSocket servers, gaming |

**Interview tip:** Prefer stateless for web services. If state needed, externalize it (Redis, database).

---

### Event-Driven vs. Polling Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Event-Driven                                  │
└─────────────────────────────────────────────────────────────────┘

Components react to events as they occur:

┌──────────┐   Event: OrderCreated   ┌─────────────┐
│  Order   │ ────────────────────▶   │   Event     │
│  Service │                         │   Bus       │
└──────────┘                         └──────┬──────┘
                                            │
              ┌─────────────────────────────┼───────────────────┐
              │                             │                   │
              ▼                             ▼                   ▼
        ┌──────────┐                 ┌──────────┐        ┌──────────┐
        │ Inventory│                 │   Email  │        │Analytics │
        │  Service │                 │  Service │        │ Service  │
        └──────────┘                 └──────────┘        └──────────┘
        
Subscribers react immediately when events published.


┌─────────────────────────────────────────────────────────────────┐
│                    Polling                                       │
└─────────────────────────────────────────────────────────────────┘

Components periodically check for updates:

┌──────────┐         ┌──────────┐
│ Inventory│ ─poll─▶ │  Order   │   "Any new orders?"
│  Service │ ◀─────  │  Service │   "No" (empty response)
└──────────┘         └──────────┘
     │
     │ wait 5 seconds
     │
     │ ─poll─▶        "Any new orders?"
     │ ◀─────         "Yes, order 123"
     │
     ▼
 Process order
```

| Aspect | Event-Driven | Polling |
|--------|--------------|---------|
| **Latency** | Low (immediate reaction) | High (poll interval) |
| **Efficiency** | High (only when events occur) | Low (many empty polls) |
| **Complexity** | Higher (event bus, handlers) | Lower (simple loops) |
| **Coupling** | Loose (via events) | Tighter (direct calls) |
| **Debugging** | Harder (async flow) | Easier (synchronous) |
| **Use case** | Microservices, real-time | Batch jobs, legacy integration |

---

### System Design Interview Quick Reference

| Concept | Key Points to Remember |
|---------|------------------------|
| **Long-Polling** | Server holds request until data; simple but overhead on reconnect |
| **WebSockets** | Bidirectional, persistent; best for chat/gaming |
| **SSE** | Server→client only; auto-reconnect built-in |
| **Quorum** | R + W > N for consistency; majority prevents split-brain |
| **Heartbeat** | Periodic liveness signal; fast detection = more false positives |
| **Checksum** | Detect corruption; CRC32 fast, SHA-256 secure |
| **Leader-Follower** | Single writer; followers replicate; election on failure |
| **OAuth** | Authorization framework; delegated access |
| **JWT** | Stateless token; self-contained claims |
| **Encryption** | Symmetric (AES) fast; Asymmetric (RSA) for key exchange |
| **DDoS** | Volumetric or application layer; mitigate with CDN, rate limiting |
| **Kafka** | Distributed log; high throughput; consumer tracks offset |
| **DFS** | Files split into blocks; replicated across nodes |
| **Batch vs Stream** | Batch = high latency, complete data; Stream = real-time |
| **Stateless** | Prefer for scalability; externalize state |
| **Event-Driven** | Loose coupling; reactive; good for microservices |
