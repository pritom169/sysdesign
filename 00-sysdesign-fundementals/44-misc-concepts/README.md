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

