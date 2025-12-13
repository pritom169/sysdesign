## Checksum


### What is Checksum?

A **checksum** is a fixed-size value computed from data to detect errors or corruption. If the checksum doesn't match, data was altered.

```
┌─────────────────────────────────────────────────────────────────┐
│  Data Transmission with Checksum                                 │
└─────────────────────────────────────────────────────────────────┘

Sender                                              Receiver
┌────────────────┐                            ┌────────────────┐
│ Data: "Hello"  │                            │                │
│                │                            │                │
│ Compute:       │                            │ Receive:       │
│ checksum(data) │    ┌──────────────────┐    │ data + checksum│
│ = 0x4A3B       │───▶│ "Hello" | 0x4A3B │───▶│                │
│                │    └──────────────────┘    │ Verify:        │
└────────────────┘           Network          │ checksum(data) │
                                              │ = 0x4A3B ✓     │
                                              └────────────────┘

If corruption occurs:
┌──────────────────┐    ┌──────────────────┐
│ "Hello" | 0x4A3B │───▶│ "HeLLo" | 0x4A3B │  ← Bit flipped
└──────────────────┘    └──────────────────┘
                                  │
                                  ▼
                        checksum("HeLLo") = 0x5C2D
                        0x5C2D ≠ 0x4A3B
                        CORRUPTION DETECTED!
```

### Common Checksum Algorithms

| Algorithm | Output Size | Speed | Collision Resistance | Use Case |
|-----------|-------------|-------|---------------------|----------|
| **CRC32** | 32 bits | Very fast | Low | Network packets, files |
| **MD5** | 128 bits | Fast | Broken | File verification (non-security) |
| **SHA-1** | 160 bits | Medium | Weak | Legacy systems |
| **SHA-256** | 256 bits | Medium | Strong | Security, blockchain |
| **xxHash** | 64/128 bits | Fastest | Low | High-speed hashing |

### Uses of Checksum

**1. Data Integrity in Storage:**
```
┌─────────────────────────────────────────────────────────────────┐
│  Disk Block with Checksum                                        │
└─────────────────────────────────────────────────────────────────┘

Write:
┌──────────────────────────────────┐
│  Data Block (4KB)   │  Checksum  │
│  actual content     │   (4 bytes)│
└──────────────────────────────────┘
        │
        ▼  Write to disk
┌──────────────────────────────────┐
│░░░░░░░░░░░░░░░░░░░░│░░░░│  Disk
└──────────────────────────────────┘

Read:
1. Read data + checksum from disk
2. Compute checksum of data
3. Compare with stored checksum
4. If mismatch → bit rot detected → recover from replica
```

**2. Network Transmission:**
```
TCP Packet Structure:
┌──────────────────────────────────────────────────────────────┐
│  IP Header  │  TCP Header  │     Payload      │  Checksum   │
│             │  (checksum)  │                  │             │
└──────────────────────────────────────────────────────────────┘
                     │
                     └── Covers header + payload
```

**3. Distributed Systems:**
```
Replication with Checksum Verification:
┌─────────┐       ┌─────────┐       ┌─────────┐
│ Primary │──────▶│ Replica │       │ Replica │
│ CS: ABC │       │ CS: ABC │       │ CS: XYZ │◀── Corruption!
└─────────┘       └─────────┘       └─────────┘
                        ✓                ✗
                                         │
                                    Repair from
                                    primary or
                                    good replica
```

**4. Deduplication:**
```
Content-Addressable Storage:
┌────────────────────┐
│ File: report.pdf   │
│ SHA-256: a1b2c3... │──┐
└────────────────────┘  │
                        │     ┌────────────────┐
                        ├────▶│  Storage       │
                        │     │  Key: a1b2c3...│
┌────────────────────┐  │     │  Data: [bytes] │
│ File: copy.pdf     │  │     └────────────────┘
│ SHA-256: a1b2c3... │──┘           ▲
└────────────────────┘              │
     Same content = same hash = store once
```

### Checksum vs. Hash vs. MAC

| Type | Purpose | Example |
|------|---------|---------|
| **Checksum** | Detect accidental errors | CRC32 |
| **Cryptographic Hash** | Detect tampering, unique ID | SHA-256 |
| **MAC/HMAC** | Verify integrity + authenticity | HMAC-SHA256 |

---

