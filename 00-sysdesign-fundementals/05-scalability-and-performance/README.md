## Scalability and Performance


#### Horizontal and vertical scaling of load balancers

When a single load balancer becomes the bottleneck, you must scale the load balancing layer itself. This is distinct from scaling the backend application servers.

##### **A. Vertical Scaling (Scaling Up)**

This strategy involves beefing up the single load balancer instance with more hardware power—adding more CPU cores, increasing RAM, or upgrading network interfaces (e.g., moving from 10Gbps to 40Gbps NICs).

How it works: You replace the existing machine with a bigger one.

- **The Ceiling:** Vertical scaling has a "hard limit." Eventually, you hit the physical limits of current hardware technology.

- **The Software Bottleneck:** Even with infinite RAM, you are limited by software constraints, such as the maximum number of ephemeral ports (approx 65k) available for opening connections.

- **Use Case:** Best for small to medium workloads where simplicity is key. Managing one giant machine is easier than managing a cluster.

##### B. Horizontal Scaling (Scaling Out)

This is the standard for high-availability enterprise systems. Instead of one giant load balancer, you deploy a cluster of multiple load balancers that work in parallel.

- The "Bootstrap" Problem: If you have 5 load balancers, how does the client know which one to visit? You need a mechanism to distribute traffic to the load balancers.

  - DNS Load Balancing (Round Robin): The simplest method. You associate a single domain name (e.g., api.example.com) with multiple IP addresses (one for each load balancer) in your DNS records. The DNS server rotates the order of IPs it returns to clients.

    - Drawback: DNS propagation is slow. If one load balancer dies, clients might keep trying to hit it until their local DNS cache expires.

  - Anycast VIP: The advanced method. You assign the same IP address to multiple load balancers located in different places. The core network routers (using BGP - Border Gateway Protocol) automatically route the user's packet to the geographically nearest load balancer advertising that IP.

  - ECMP (Equal-Cost Multi-Path): Used within data centers. The network router spreads packets across multiple load balancers based on a hash of the packet headers, treating them as equal paths to the destination.

#### Connection and Request Rate Limits

Unchecked traffic can crash even the most robust systems. Load balancers act as the "bouncer" for your application, enforcing strict entry rules.

##### A. Connection Limiting (Layer 4 Protection)

This limits the number of open TCP connections a single client (or the total global traffic) can establish.

    - Why it matters: Every open connection consumes a file descriptor and memory on the server. If a hacker opens 100,000 connections but sends no data (a Slowloris attack), they can starve the server of resources.

    - Mechanism: The load balancer monitors the counter of active connections. If the limit is reached, new SYN packets (connection requests) are dropped or rejected immediately, protecting the backend servers from ever seeing the traffic.

##### B. Request Rate Limiting (Layer 7 Protection)

This limits the number of HTTP requests over a specific time window (e.g., "100 requests per minute").

**Algorithms:**

- **Token Bucket:** The user is given "tokens" at a steady rate. Every request costs a token. If the bucket is empty, the request is denied. This allows for short "bursts" of traffic.

- **Leaky Bucket:** Requests are processed at a constant, steady rate, regardless of how fast they come in. Good for smoothing out traffic spikes.

**Granularity:**

- **By IP:** Limits a specific user.

- **By Path:** Limits expensive endpoints (e.g., /search might have a lower limit than /home).

- **By API Key:** Ensures "Gold Tier" customers get more throughput than "Free Tier" users.

#### 3. Caching and Content Optimization

A load balancer is the perfect place to optimize content because it sits at the edge of your network, closest to the user.

##### A. Static Content Caching

Instead of asking the backend web server for logo.png every single time, the load balancer stores a copy in its memory.

- The "Hit" vs. "Miss": When a request comes in, the LB checks its cache. If the file is there (Cache Hit), it serves it instantly without touching the backend. This drastically lowers the CPU load on backend servers.

- Cache-Control Headers: The LB respects headers like max-age or ETag to know when the content is stale and needs to be refreshed from the backend.

##### B. Compression (Gzip/Brotli)

Sending large text files (HTML, CSS, JSON) is slow. Compression shrinks these files by up to 70%.

- Offloading: Compressing data requires CPU power. By performing compression on the load balancer (often using specialized hardware), you free up the backend application servers to focus on business logic rather than compressing bits.

##### C. SSL/TLS Termination

Decrypting HTTPS traffic is mathematically expensive.

- Termination: The load balancer decrypts the incoming traffic, inspects it, and passes it to the backend (often over unencrypted HTTP within the secure private network). This is called "SSL Offloading," saving the backend servers significant CPU cycles.

#### 4. Impact of load balancers on latency

Placing a load balancer in the middle inherently adds a small amount of latency (the time it takes to process and forward the packet). However, smart optimizations can actually make the total request time faster than if the load balancer weren't there.

##### A. Connection Multiplexing (Keep-Alive)

Establishing a TCP connection requires a "3-Way Handshake" (SYN, SYN-ACK, ACK), which takes time.

- The Optimization: The load balancer maintains a pool of "warm," already-established connections to the backend servers.

- The Flow: When a client request arrives, the LB doesn't open a new connection to the backend. It grabs an existing open connection from the pool. This eliminates the handshake latency for the backend leg of the journey.

##### B. Geographical Distribution (GSLB)

If your user is in London but your server is in New York, latency is governed by the speed of light.

- Global Server Load Balancing (GSLB): You deploy load balancers in multiple regions (e.g., US-East, EU-West, Asia-Pacific).

- DNS Routing: The GSLB system detects the user's location (via their IP) and updates DNS to point them to the physically closest load balancer (e.g., the London user is routed to the EU-West VIP).

##### C. Protocol Upgrades (HTTP/2 & HTTP/3)

Modern load balancers can translate protocols to improve speed.

- HTTP/2: Supports Multiplexing, allowing multiple requests (images, scripts, CSS) to be sent over a single TCP connection simultaneously, rather than waiting for one to finish before starting the next.

- HTTP/3 (QUIC): Uses UDP instead of TCP. It solves "Head-of-Line Blocking," meaning if one packet is lost, it doesn't pause the entire stream of data. This is crucial for users on unstable mobile networks.

### Challenges of Load Balancers

Here is a condensed, high-impact breakdown of the challenges associated with Load Balancers (LBs) and their architectural solutions.

---

#### **1. The Single Point of Failure (SPOF)**

Since the LB acts as the "front door," if it fails, the entire application goes offline (), regardless of backend health.

- **The Remedy: High Availability (HA) Pairs**
- **Active-Passive:** Deploy two LBs. The "Active" node handles traffic while the "Passive" node monitors its heartbeat.
- **VRRP (Virtual Router Redundancy Protocol):** If the Active node dies, the Passive node instantly claims the shared Virtual IP (VIP) to resume traffic flow without manual intervention.

#### **2. Configuration Complexity & Drift**

Modern LBs are programmable logic centers. Misconfigured timeouts or cipher suites can drop users or create security holes. Manual changes lead to **Configuration Drift**, where the live server diverges from documentation.

- **The Remedy: Infrastructure as Code (IaC)**
- Avoid manual tweaks. Use tools like **Terraform** or **Ansible** to define configurations in code. This ensures consistency and allows for rapid recovery or rollback.

#### **3. Scalability Bottlenecks**

The LB itself has limits, primarily **Port Exhaustion** (running out of ephemeral ports) and **CPU Saturation** (from heavy SSL decryption).

- **The Remedy: Offloading & DNS Scaling**
- **SSL Offloading:** Move the heavy lifting of decryption to specialized cloud services (like AWS ALB or Cloudflare) to spare the LB's CPU.
- **DNS Load Balancing:** Use DNS to distribute traffic across multiple LB clusters rather than relying on a single giant instance.

#### **4. Latency (The "Extra Hop")**

Introducing a middleman doubles the network packets needed for a request (Client LB Server), potentially increasing wait times.

- **The Remedy: Direct Server Return (DSR)**
- In DSR, the LB forwards the request to the backend, but the backend replies **directly to the client**, bypassing the LB on the return trip. This prevents the LB from becoming a bottleneck for large outbound data (like video streams).

#### **5. Sticky Sessions vs. Statelessness**

If an application stores user data (e.g., shopping carts) in a specific server's RAM, the LB must mistakenly keep sending that user to the same server ("Sticky Sessions"). This causes uneven load distribution.

- **The Remedy: Distributed Caching (Redis)**
- Adopt a **Stateless Architecture**. Don't store sessions on the web server. Store them in a shared database like **Redis**. Now, the LB can send traffic to _any_ server, and the user's session remains intact.

#### **6. Cost Management**

Hardware LBs require expensive upfront capital (CAPEX), while cloud LBs (OPEX) can scale costs linearly with traffic, leading to "bill shock" during attacks.

- **The Remedy:** Use **Autoscaling** to ensure you only pay for the active LBs you need at that moment, and implement strict **Rate Limiting** to prevent DDoS attacks from inflating your data processing bills.

#### **7. Health Checks & "Flapping"**

If an LB is too sensitive, it might mark a struggling server as "Dead," then "Healthy," then "Dead" again rapidly. This is called **Flapping**, and it destabilizes the cluster.

- **The Remedy: Hysteresis**
- Configure distinct thresholds for status changes. For example, require **3 consecutive failures** to mark a server down, but **5 consecutive successes** to mark it back up. This buffer ensures a server is truly stable before it receives traffic again.

---

### Introduction to API Gateway

An API Gateway acts as a single entry point for all client requests. It takes a request, understands what the user wants (the business intent), and routes it to the correct service. Crucially, it performs tasks that the backend services shouldn't have to worry about.

Key Responsibilities:

- Authentication & Security: "Do you have an ID badge?" It checks if the user is logged in (OAuth, JWT) before the request ever reaches your servers.

- Protocol Translation: "I speak English, but the engineering team speaks German." It can take a REST request from a phone and convert it to gRPC or SOAP for an old legacy system.

- Rate Limiting: "You can only visit 5 times a minute." It prevents users from spamming the system.

- Response Aggregation: If a user's profile page needs data from the User Service, Billing Service, and Shipping Service, the Gateway calls all three and combines the answers into one neat JSON response for the phone.

#### API Gateways and Load Balancer: How are they different?

The confusion often arises because **API Gateways can do load balancing**, but Load Balancers cannot do API management.

| Feature           | API Gateway                                                                                  | Load Balancer                                                                                   |
| ----------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Primary Goal**  | Expose, secure, and manage APIs as a product.                                                | Distribute traffic to ensure uptime and speed.                                                  |
| **OSI Layer**     | **Layer 7 (Application):** It reads the actual data/content of the message.                  | **Layer 4 (Transport):** Mostly looks at IP/Port. (Some L7 LBs exist but are less logic-heavy). |
| **Intelligence**  | **High:** Can read a user's ID, route based on specific headers, or rewrite URLs.            | **Low/Medium:** Cares mostly about server health and traffic volume.                            |
| **Security**      | **Deep Security:** Authentication (OAuth), SQL Injection blocking, Rate limiting by user ID. | **Infrastructure Security:** DDoS protection, Firewalling IPs, SSL Offloading.                  |
| **Routing Logic** | "Route `/billing` to Service A and `/profile` to Service B."                                 | "Route the next 10 packets to Server #3 because it's free."                                     |

---

### Key Usage of API Gateways

- Centralized Security: It provides a single point to enforce security policies. Instead of every microservice handling authentication (who you are) and authorization (what you are allowed to do), the gateway handles this once, verifying tokens or keys before passing the request further.

- Rate Limiting and Throttling: It protects backend services from being overwhelmed by traffic spikes or malicious attacks (DDoS). The gateway can limit how many requests a user or service can make per second.

- Protocol Translation: It allows clients to communicate via one protocol (e.g., HTTP/REST) while backend services might use another (e.g., gRPC or AMQP), ensuring seamless communication without the client needing to know backend details.

- Load Balancing: The gateway distributes incoming network traffic across multiple servers to ensure no single server bears too much load, improving application responsiveness and availability.

- Request/Response Transformation: It can modify requests and responses on the fly. For example, it might aggregate data from multiple services into a single response for the client, reducing the number of round trips the client needs to make.

- Analytics and Monitoring: Because all traffic flows through the gateway, it is the perfect place to log usage data, monitor response times, and track errors, providing a unified view of system health.

### Disadvantages of API Gateways

However, the very features that make an API Gateway powerful also create its downsides. By centralizing logic, you create a potential bottleneck and a critical dependency.

- **Single Point of Failure (SPOF):** This is the most critical risk. If the Product Service fails, only the product page breaks. **If the API Gateway fails, the entire application goes offline.** This necessitates expensive, high-availability setups with redundancy to ensure the Gateway never crashes.
- **Increased Latency:** Physics is unavoidable; the Gateway adds an extra "hop" in the network journey. Every request must go from Client Gateway Service, rather than Client Service. While usually measured in milliseconds, this additional step can be problematic for high-frequency trading or real-time gaming applications where every microsecond counts.
- **Operational Complexity:** Managing a Gateway is not free. It requires its own infrastructure, monitoring, and scaling rules. It can also become a **development bottleneck**; if a team adds a new microservice but has to wait for the "Gateway Team" to update the routing configuration before they can go live, agility is lost.
- **Risk of the "Monolithic Gateway":** There is a temptation to put too much business logic (like complex code or data processing) into the Gateway. If this happens, the Gateway essentially becomes a new monolith—bloated, hard to update, and difficult to test—recreating the exact problem microservices were meant to solve.

