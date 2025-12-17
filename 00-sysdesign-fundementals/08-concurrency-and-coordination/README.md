## Concurrency and Coordination


In distributed systems, Concurrency and Coordination are the mechanisms used to tame non-determinism. Unlike a single computer where the OS controls the clock and memory, distributed systems consist of independent nodes connected by an unreliable network, each with its own clock.

### A. Concurrency Control

Concurrency control is the process of managing simultaneous access to shared resources or data in a distributed system.

- **Locking:** Locks are used to restrict access to shared resources or data, ensuring that only one process can access them at a time.

- **Optimistic concurrency control:** This approach assumes that conflicts are rare and allows multiple processes to work simultaneously. Conflicts are detected and resolved later, usually through a validation and rollback mechanism.

- **Transactional memory:** This technique uses transactions to group together multiple operations that should be executed atomically, ensuring data consistency and isolation.

### B. Synchronization

While Concurrency Control handles access, Synchronization handles timing and ordering. In distributed systems, "time" is relative because physical clocks drift.

- **Barriers:** Barriers are used to synchronize the execution of multiple processes or threads, ensuring that they all reach a specific point before proceeding.

- **Semaphores:** Semaphores are signaling mechanisms that control access to shared resources and maintain synchronization among multiple processes or threads.

- **Condition variables:** Condition variables allow processes or threads to wait for specific conditions to be met before proceeding with their execution.

### C. Coordination Services

Coordination services are specialized components or tools that help manage distributed systems' complexity by providing a set of abstractions and primitives for tasks like configuration management, service discovery, leader election, and distributed locking. Examples of coordination services include Apache ZooKeeper, etcd, and Consul.

### D. Consistency Models

Consistency models define the rules of engagement between your application and your data. They are the contract that tells you how current your data is at any given moment across a distributed cluster.

#### 1. Global / Data-Centric Models

These models view the system from the perspective of the database itself, ensuring that all nodes agree on a specific history of events.

**Linearizability (Strict Consistency)**
This is the "Holy Grail" of consistency. It provides a guarantee that the system behaves exactly like a single server, even if it is distributed across the world. In a linearizable system, once a write is successfully committed at a specific timestamp (say, 12:00:00), any read operation that begins after that timestamp (12:00:01) is guaranteed to return that new value, regardless of which node receives the request. This model respects **real-time** wall-clock order. It is extremely expensive to implement because it requires heavy coordination (like the Raft or Paxos algorithms) to ensure every node is perfectly synced before acknowledging a write.

- **Use Case:** Financial ledgers, password changes, or medical data where stale data could be catastrophic.

**Sequential Consistency**
This model is slightly more relaxed than Linearizability. It guarantees that all nodes see operations in the _same total order_, but that order does not necessarily have to match real-time wall-clock time. If User A writes "X" and User B writes "Y" simultaneously, the system can choose to process "Y" before "X" or "X" before "Y"—as long as _every_ node in the cluster agrees on that chosen sequence. The system respects the "program order" of a single client (if I write A then B, everyone sees A then B), but it does not guarantee that my write is instantly visible to you the moment I hit enter, only that we will eventually agree on the sequence of events.

- **Use Case:** Distributed queues (FIFO) or logging systems where the order of processing matters more than the exact millisecond the entry arrived.

---

#### 2. Relaxed / Weak Models

These models prioritize system uptime and low latency over immediate data accuracy. They are defined by the **BASE** philosophy (Basically Available, Soft state, Eventual consistency).

**Causal Consistency**
This model is a smart middle ground. It tracks which operations depend on others. If Operation B was caused by Operation A (e.g., a "Reply" to a "Comment"), the system guarantees that everyone will see the Comment (A) before the Reply (B). However, unrelated (concurrent) events can be seen in any order. If two people upload different photos to different albums at the same time, it doesn't matter which one you see first. This captures the vast majority of "correctness" required for human applications without the performance penalty of enforcing a global order on unrelated events.

- **Use Case:** Social media feeds, chat applications (Slack/Discord), where conversation threads must remain in order but unrelated channels don't matter.

**Eventual Consistency**
The weakest but fastest model. It guarantees that if no new updates are made to a given data item, _eventually_ all accesses to that item will return the last updated value. In the short term, however, different nodes may return different versions of the data. This allows the system to accept writes even when parts of the network are broken or slow. The system typically resolves conflicts using a "Last Write Wins" strategy or by asking the application to merge divergent data.

- **Use Case:** DNS (Domain Name System), YouTube view counters, or "Likes" on a viral post. It is acceptable if a user sees "99 likes" while another sees "101 likes" for a few seconds.

---

#### 3. Client-Centric Models

These models focus on the _user's_ experience rather than the global state of the database. They bridge the gap between weak and strong consistency by ensuring a single user sees a sensible view of the world.

**Read-Your-Writes Consistency**
This guarantees that a client will always see the effects of their _own_ previous writes. If you update your profile picture, this model ensures that when the page reloads, you see the new picture—even if other users in the world still see the old one for a few minutes. Without this, users would constantly feel like the application is broken ("I just saved this, why is it gone?").

- **Use Case:** User profile management, document editing tools, or posting a status update.

**Monotonic Read Consistency**
This guarantees that "time never moves backward." If a user reads a data value (Version 2), they will never subsequently read an older value (Version 1). This is crucial in distributed systems where a user might be load-balanced between two servers. If Server A has the new data and Server B is lagging, switching the user from A to B would effectively revert their view of history. Monotonic reads prevent this by ensuring the user is only routed to servers that are at least as up-to-date as the last one they visited.

- **Use Case:** Checking an email inbox (deleted emails shouldn't reappear) or banking transaction histories (a deposit shouldn't disappear after you've seen it).

**Session Consistency**
This wraps the above guarantees (Read-Your-Writes and Monotonic Reads) into the context of a specific user session. As long as your browser window is open or your session token is active, the system guarantees consistent data. If the session ends or expires, the guarantee resets. This is practical because it aligns the data guarantee with the user's interaction period.

- **Use Case:** E-commerce shopping carts. As long as you are shopping, your cart items must persist. If you log out and log back in days later, it is acceptable if the cart state is rebuilt from a slightly different backup, but during the active session, it must be rock solid.

