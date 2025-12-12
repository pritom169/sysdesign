## Long-Polling vs. WebSockets vs. Server-Sent Events


### Introduction

Real-time communication between clients and servers is fundamental to modern applications. Understanding when to use each technique is a common interview question.

### HTTP Polling (Regular Polling)

Client repeatedly requests data at fixed intervals.

```
Client                           Server
  │                                │
  │──── GET /updates ────────────▶│
  │◀─── Response (empty) ─────────│
  │                                │
  │     ⏱️ Wait 5 seconds          │
  │                                │
  │──── GET /updates ────────────▶│
  │◀─── Response (empty) ─────────│
  │                                │
  │     ⏱️ Wait 5 seconds          │
  │                                │
  │──── GET /updates ────────────▶│
  │◀─── Response (data!) ─────────│
```

**Problem:** Wasteful. Most requests return empty responses. High latency between event and delivery.

---

### Long-Polling

Server holds the request open until data is available or timeout occurs.

```
Client                           Server
  │                                │
  │──── GET /updates ────────────▶│
  │         (request held open)    │
  │              ...               │
  │         ⏳ waiting...          │
  │              ...               │
  │◀─── Response (data!) ─────────│  ← Event occurred
  │                                │
  │──── GET /updates ────────────▶│  ← Immediately reconnect
  │         (request held open)    │
  │              ...               │
```

**How It Works:**
1. Client sends request
2. Server holds connection open (30-60 seconds typical)
3. When data available OR timeout: server responds
4. Client immediately sends new request

**Pros:**
- Works everywhere (standard HTTP)
- Simple to implement
- Firewall/proxy friendly

**Cons:**
- HTTP overhead on each reconnection
- Server must manage many pending connections
- Not truly bidirectional

---

### WebSockets

Full-duplex, persistent TCP connection with low overhead.

```
Client                           Server
  │                                │
  │──── HTTP Upgrade Request ────▶│
  │◀─── 101 Switching Protocols ──│
  │                                │
  │════════ WebSocket Open ════════│
  │                                │
  │◀──── Push: "new message" ─────│
  │                                │
  │───── Send: "my response" ────▶│
  │                                │
  │◀──── Push: "another msg" ─────│
  │                                │
  │───── Send: "typing..." ──────▶│
  │                                │
  │════════════════════════════════│
```

**Connection Upgrade:**
```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**Pros:**
- True bidirectional communication
- Minimal overhead after connection (2-byte frame header)
- Low latency
- Efficient for high-frequency updates

**Cons:**
- Requires WebSocket support (load balancers, proxies)
- Connection state must be managed
- More complex to scale (sticky sessions or pub/sub needed)
- No built-in reconnection

---

### Server-Sent Events (SSE)

Unidirectional stream from server to client over HTTP.

```
Client                           Server
  │                                │
  │──── GET /events ─────────────▶│
  │     Accept: text/event-stream  │
  │                                │
  │◀─── HTTP 200 ─────────────────│
  │     Content-Type: text/event-stream
  │                                │
  │◀─── data: {"msg": "hello"} ───│
  │                                │
  │◀─── data: {"msg": "update"} ──│
  │                                │
  │◀─── data: {"msg": "news"} ────│
  │                                │
  │     (connection stays open)    │
```

**Event Format:**
```
event: notification
data: {"userId": 123, "message": "New comment"}
id: 1001
retry: 5000

event: heartbeat
data: ping

```

**Pros:**
- Simple HTTP - works through most proxies
- Built-in reconnection and event IDs
- Automatic resume from last event
- Native browser support (EventSource API)

**Cons:**
- Unidirectional only (server → client)
- Limited to text data (no binary)
- Connection limit per domain (6 in HTTP/1.1)

---

### Comparison Table

| Feature | Long-Polling | WebSockets | SSE |
|---------|--------------|------------|-----|
| **Direction** | Client → Server (simulated push) | Bidirectional | Server → Client |
| **Protocol** | HTTP | WS/WSS (TCP) | HTTP |
| **Connection** | New per response | Persistent | Persistent |
| **Overhead** | High (HTTP headers each time) | Low (2-byte frames) | Low |
| **Binary data** | Yes | Yes | No (text only) |
| **Auto-reconnect** | Must implement | Must implement | Built-in |
| **Browser support** | Universal | Universal | Universal (except IE) |
| **Proxy/firewall** | Always works | May have issues | Usually works |
| **Scaling** | Easier | Harder (stateful) | Medium |

---

### When to Use What

```
┌─────────────────────────────────────────────────────────────────┐
│                    Decision Tree                                 │
└─────────────────────────────────────────────────────────────────┘

Need bidirectional real-time communication?
    │
    ├── YES ──▶ WebSockets
    │           (chat, gaming, collaborative editing)
    │
    └── NO ──▶ Need server push only?
                    │
                    ├── YES ──▶ SSE
                    │           (notifications, live feeds, dashboards)
                    │
                    └── NO ──▶ Long-Polling
                                (legacy support, simple notifications)
```

| Use Case | Best Choice | Why |
|----------|-------------|-----|
| Chat application | WebSockets | Bidirectional, low latency |
| Live sports scores | SSE | Server push only, auto-reconnect |
| Stock ticker | WebSockets or SSE | Depends on client interaction needs |
| Notification system | SSE | Simple, reliable, auto-resume |
| Multiplayer game | WebSockets | Bidirectional, minimal latency |
| Legacy browser support | Long-Polling | Works everywhere |
| Social media feed | SSE | Server push, handles reconnection |

---

### Scaling Considerations

**WebSockets at Scale:**
```
┌─────────────────────────────────────────────────────────────────┐
│  Challenge: WebSocket connections are stateful                   │
└─────────────────────────────────────────────────────────────────┘

Solution 1: Sticky Sessions
┌────────┐     ┌────────────────┐     ┌──────────┐
│ Client │────▶│ Load Balancer  │────▶│ Server A │  ← Always same server
└────────┘     │ (IP hashing)   │     └──────────┘
               └────────────────┘

Solution 2: Pub/Sub Backend
┌────────┐     ┌──────────┐     ┌─────────────┐
│ Client │◀───▶│ Server A │◀───▶│             │
└────────┘     └──────────┘     │   Redis     │
                                │   Pub/Sub   │
┌────────┐     ┌──────────┐     │             │
│ Client │◀───▶│ Server B │◀───▶│             │
└────────┘     └──────────┘     └─────────────┘

Any server can push to any client via shared message bus
```

---

