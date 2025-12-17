## Monitoring and Observability


In the world of distributed systems, if you cannot measure it, you cannot manage it. Monitoring and Observability are often used interchangeably, but they serve different purposes.

- **Monitoring** tells you **when** something is wrong (e.g., "The server is down"). It is reactive and based on known failure modes.
- **Observability** allows you to understand **why** it is wrong (e.g., "The server is down because a database lock caused a memory leak"). It is exploratory and helps you debug unknown unknowns.

To achieve true observability, we rely on three fundamental pillars: **Metrics**, **Traces**, and **Logs**.

---

### A. Metrics Collection (The "Vitals")

Metrics are numerical data points measured over time. They are cheap to store and fast to query because they are aggregated. They tell you "What" is happening but not "Why."

- **How it works:** Instead of recording every single login attempt (which would fill your disk), metrics simply increment a counter: `login_attempts = 502`.
- **Key Types:**
- **Counters:** Values that only go up (e.g., Total Requests, Errors).
- **Gauges:** Values that fluctuate (e.g., CPU usage, Memory available).
- **Histograms:** Distributions of values (e.g., "99% of requests finished in under 200ms").

- **High Cardinality Issues:** A common challenge in metrics is "cardinality." If you tag metrics with data that changes too often (like User IDs or IP addresses), your database will explode in size. Metrics are for _aggregate_ health, not individual user tracking.
- **Tools:** **Prometheus** (standard for Kubernetes), **InfluxDB**, **Datadog**.

### B. Distributed Tracing (The "Journey")

In a microservices architecture, a single user click might trigger requests across 50 different services. If the request is slow, metrics will show high latency on all 50 services, but they won't tell you _which one_ caused the delay.

- **How it works:** A unique ID (**Trace ID**) is attached to the request header at the entry point (e.g., the Load Balancer). As the request travels from Service A → Service B → Database, this ID is passed along ("Context Propagation").
- **Spans:** Each operation in the chain creates a "Span," which records the start time, end time, and metadata for that specific step.
- **The Waterfall View:** Tracing tools visualize this data as a waterfall, instantly highlighting the specific database query or API call that took 5 seconds while everything else took 5ms.
- **Tools:** **Jaeger**, **Zipkin**, **OpenTelemetry** (the industry standard for collecting traces).

### C. Logging (The "Journal")

Logs are discrete, timestamped records of specific events. Unlike metrics, logs act as the "Black Box" recorder of your system, providing the highest fidelity of data.

- **Structured vs. Unstructured:**
- _Old Way (Unstructured):_ `2023-10-01 Error: User failed login.` (Hard for machines to parse).
- _New Way (Structured/JSON):_ `{"timestamp": "2023-10-01", "level": "ERROR", "event": "login_failed", "user_id": 8421}`. This allows you to query logs like a database (e.g., "Show me all errors for user 8421").

- **Log Aggregation:** Since you have hundreds of servers, you cannot SSH into each one to read text files. You must ship all logs to a central "Log Aggregator" to index and search them.
- **Tools:** **ELK Stack** (Elasticsearch, Logstash, Kibana), **Splunk**, **Loki**.

### D. Alerting & Anomaly Detection (The "Alarm")

Collecting data is useless if no one looks at it. Alerting converts data into action.

- **SLIs, SLOs, and SLAs:** Modern alerting is built on these acronyms:
- **SLI (Indicator):** The real number (e.g., "Latency is 250ms").
- **SLO (Objective):** The internal goal (e.g., "Latency should be < 300ms 99% of the time").
- **SLA (Agreement):** The legal contract with users (e.g., "If latency > 500ms, we pay you back").

- **Alert Fatigue:** The biggest risk in monitoring. If you alert on _everything_ (e.g., "CPU is at 80%"), engineers will ignore the pager. You should only alert on **symptoms** that affect the user (e.g., "Checkout Error Rate > 1%"), not purely on causes.
- **Anomaly Detection:** Instead of static thresholds ("Alert if > 100 errors"), machine learning learns the daily pattern. It knows that 100 errors is normal on Black Friday but abnormal on a Tuesday morning.

### E. Visualization (The "Cockpit")

This is the "Single Pane of Glass" where all three pillars converge.

- **Correlating Data:** The best dashboards allow you to spot a spike in a **Metric** (High Latency), click it to see the **Trace** (The slow database query), and drill down into the **Logs** (The specific error message "Connection Timeout").
- **Golden Signals:** Google SRE recommends every dashboard track the "Four Golden Signals":

1. **Latency:** Time it takes to service a request.
2. **Traffic:** Demand placed on your system (req/sec).
3. **Errors:** Rate of requests that fail.
4. **Saturation:** How "full" your most constrained resource is (e.g., Memory).

