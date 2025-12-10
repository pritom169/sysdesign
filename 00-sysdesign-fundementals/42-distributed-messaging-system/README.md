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

