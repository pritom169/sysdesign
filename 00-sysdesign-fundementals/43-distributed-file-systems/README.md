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

