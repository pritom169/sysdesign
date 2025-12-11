## Bloom Filters


### What is a Bloom Filter?

A probabilistic data structure that tests whether an element is a member of a set. It can tell you:
- **"Definitely not in set"** — 100% accurate
- **"Probably in set"** — may have false positives

Space-efficient alternative to storing actual elements. Uses a bit array and multiple hash functions.

```
Bloom Filter Structure:
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │  ← Bit array (m bits)
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
      ↑       ↑           ↑       ↑
      └───────┴───────────┴───────┘
              Multiple hash functions set these bits
```

**Why "probabilistic"?** The filter never stores actual elements—only bits. When checking membership, matching bits could be from the element you're looking for OR from other elements that happened to hash to the same positions.

---

### How It Works

#### Insert Operation

Hash the element with k hash functions. Set corresponding bits to 1.

```
Insert "apple" (k=3 hash functions):
  h1("apple") = 2
  h2("apple") = 5
  h3("apple") = 9

Bit array (m=10):
Index:   0  1  2  3  4  5  6  7  8  9
Before: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
After:  [0, 0, 1, 0, 0, 1, 0, 0, 0, 1]
               ↑        ↑           ↑
              h1       h2          h3
```

**Key insight:** Setting a bit to 1 is idempotent. If the bit is already 1, nothing changes.

#### Lookup Operation

Hash the element. Check if ALL corresponding bits are 1.

```
Check "apple":
  h1("apple") = 2 → bit[2] = 1 ✓
  h2("apple") = 5 → bit[5] = 1 ✓
  h3("apple") = 9 → bit[9] = 1 ✓
  All bits set → "Probably in set"

Check "banana":
  h1("banana") = 1 → bit[1] = 0 ✗
  At least one bit is 0 → "Definitely NOT in set"
```

**Why "definitely not"?** If an element was inserted, ALL its hash positions would be 1. Finding even one 0 proves the element was never added.

#### False Positives Explained

As more elements are inserted, more bits become 1. Eventually, an element that was never inserted might have all its hash positions coincidentally set by other elements.

```
State after inserting "apple", "orange", "mango":

Index:   0  1  2  3  4  5  6  7  8  9
Bits:   [0, 1, 1, 0, 1, 1, 0, 1, 0, 1]
              ↑     ↑  ↑     ↑     ↑
         apple(2) orange mango  apple(9)
                   sets    sets
                  1,4,5   5,7

Check "grape" (never inserted):
  h1("grape") = 1 → bit[1] = 1 ✓ (was set by "orange")
  h2("grape") = 4 → bit[4] = 1 ✓ (was set by "orange")
  h3("grape") = 9 → bit[9] = 1 ✓ (was set by "apple")
  All bits set → "Probably in set" ← FALSE POSITIVE!
```

**The filter is "lying"** — grape was never inserted, but all its bit positions happen to be 1 from other insertions.

---

### Mathematical Properties

#### False Positive Probability

After inserting n elements into a filter with m bits using k hash functions:

```
p ≈ (1 - e^(-kn/m))^k

Intuition:
- Each hash sets 1 bit out of m
- After kn hash operations, probability a specific bit is still 0: e^(-kn/m)
- Probability a specific bit is 1: (1 - e^(-kn/m))
- False positive = all k bits are 1 by chance: raise to power k
```

#### Optimal Number of Hash Functions

Given m bits and n expected elements, the optimal k minimizes false positives:

```
k_optimal = (m/n) × ln(2) ≈ 0.693 × (m/n)

Example: m = 1000 bits, n = 100 elements
k = 0.693 × (1000/100) = 6.93 ≈ 7 hash functions
```

**Why not more hash functions?** Too few → not enough bits checked. Too many → too many bits set to 1, increasing collisions.

#### Space Estimation

For target false positive rate p with n elements:

```
m = -n × ln(p) / (ln(2))²
m ≈ -1.44 × n × ln(p)

Example calculations:

| Elements (n) | Target FP Rate | Bits Needed (m) | Size     |
|--------------|----------------|-----------------|----------|
| 1 million    | 1% (0.01)      | 9.6M bits       | 1.2 MB   |
| 1 million    | 0.1% (0.001)   | 14.4M bits      | 1.8 MB   |
| 10 million   | 1% (0.01)      | 96M bits        | 12 MB    |
```

**Space comparison:**
```
Storing 1M email addresses (avg 25 bytes each):
- HashSet: ~25 MB + overhead ≈ 40 MB
- Bloom filter (1% FP): 1.2 MB

Bloom filter uses ~3% of the space!
```

---

### Benefits

| Benefit | Explanation |
|---------|-------------|
| **Space efficient** | Bits vs actual data. 1M elements in 1.2 MB vs 40+ MB |
| **O(k) operations** | Insert and lookup are O(k), typically k < 10 |
| **No false negatives** | "Not present" is always correct—safe for filtering |
| **Parallelizable** | k hash computations are independent |
| **Union operation** | Bitwise OR merges two filters instantly |
| **No resizing needed** | Fixed memory footprint once allocated |

---

### Limitations

| Limitation | Explanation |
|------------|-------------|
| **False positives** | Must handle "maybe yes" cases with secondary lookup |
| **No deletion** | Cannot unset a bit—might affect other elements |
| **No enumeration** | Cannot list what's in the filter |
| **No count** | Cannot tell how many elements were inserted |
| **Fixed capacity** | Exceeding planned n increases FP rate dramatically |
| **Hash quality matters** | Poor hash functions increase collisions |

#### Why No Deletion?

```
Insert "apple": sets bits 2, 5, 9
Insert "orange": sets bits 2, 4, 7

Bits: [0, 0, 1, 0, 1, 1, 0, 1, 0, 1]
            ↑     ↑  ↑     ↑     ↑
         shared  orange apple orange apple

Delete "apple" by clearing bits 2, 5, 9:
Bits: [0, 0, 0, 0, 1, 0, 0, 1, 0, 0]
            ↑        ↑
         WRONG!   WRONG!
         Bit 2 was shared with "orange"
         Now "orange" lookup fails → FALSE NEGATIVE
```

False negatives break the core guarantee. This is why standard Bloom filters don't support deletion.

---

### Variants

#### Counting Bloom Filter

Replace each bit with a counter (typically 4 bits). Increment on insert, decrement on delete.

```
Standard Bloom (bits):
[0, 1, 1, 0, 1]  ← Can't tell which element set each bit

Counting Bloom (4-bit counters):
[0, 2, 1, 0, 3]  ← Bit 1 was set by 2 elements

Insert "apple" (positions 1, 2):
Before: [0, 2, 1, 0, 3]
After:  [0, 3, 2, 0, 3]

Delete "apple":
Before: [0, 3, 2, 0, 3]
After:  [0, 2, 1, 0, 3]  ← Safe! Other elements still tracked
```

**Trade-offs:**
- 4x more space (4-bit counters vs 1-bit)
- Counter overflow possible (rare with 4 bits)
- Enables deletion without false negatives

#### Scalable Bloom Filter

Dynamically grows by chaining filters. Each new filter has tighter FP rate.

```
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│    Filter 0      │   │    Filter 1      │   │    Filter 2      │
│   FP rate: p₀    │ → │  FP rate: p₀×r   │ → │ FP rate: p₀×r²   │
│    (full)        │   │     (full)       │   │   (active)       │
└──────────────────┘   └──────────────────┘   └──────────────────┘

Insert: Add to rightmost (active) filter
Lookup: Check ALL filters, return "present" if ANY says yes
```

**Maintains overall FP rate:** p_total = p₀ + p₀×r + p₀×r² + ... converges if r < 1

#### Cuckoo Filter

Uses cuckoo hashing with fingerprints. Supports deletion without counters.

```
Structure: Array of buckets, each holds b fingerprints

Insert "apple":
1. Compute fingerprint: f = fingerprint("apple") = 0xA3
2. Compute two bucket positions: h1("apple")=5, h2=h1 XOR hash(f)=12
3. Store fingerprint in bucket 5 or 12

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  0  │  1  │ ... │  5  │ ... │ 12  │ ... │
│     │     │     │[0xA3]│    │     │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Delete "apple":
- Find fingerprint 0xA3 in bucket 5 or 12
- Remove it
```

**Advantages over Counting Bloom:**
- Better space efficiency at FP < 3%
- Supports deletion
- Higher lookup performance (better cache locality)

| Filter Type | Deletion | Space | Best For |
|-------------|----------|-------|----------|
| Standard Bloom | No | Smallest | Static sets |
| Counting Bloom | Yes | 4x larger | Small sets needing deletion |
| Cuckoo Filter | Yes | Similar to Bloom | Dynamic sets, low FP rates |

---

### Applications

| Use Case | How Bloom Filter Helps |
|----------|------------------------|
| **Database queries** | Skip disk reads for non-existent keys |
| **Cache systems** | Avoid thundering herd for uncacheable keys |
| **Spell checkers** | Fast dictionary lookup before expensive check |
| **Network routers** | Packet deduplication at line speed |
| **Web crawlers** | Track billions of visited URLs in memory |
| **CDN** | Check edge cache before origin fetch |
| **Weak password detection** | Check against known breached passwords |

#### Real-World Example: HBase/Cassandra

LSM-tree databases use Bloom filters to avoid unnecessary disk reads.

```
Write path (no Bloom filter involvement):
Data → MemTable (RAM) → Flush → SSTable (Disk)

Read path WITH Bloom filter:
┌─────────────────────────────────────────────────────────┐
│  Read request: GET "user:12345"                         │
└────────────────────┬────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│  Check MemTable (RAM) → Not found                       │
└────────────────────┬────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────┐
│  SSTable 1: Bloom filter says "No" → SKIP (no disk I/O) │
│  SSTable 2: Bloom filter says "No" → SKIP (no disk I/O) │
│  SSTable 3: Bloom filter says "Maybe" → Read from disk  │
│  SSTable 4: Bloom filter says "No" → SKIP (no disk I/O) │
└─────────────────────────────────────────────────────────┘

Result: 1 disk read instead of 4. 75% I/O reduction.
```

**Impact:** With 1% FP rate, 99% of unnecessary disk reads are avoided.

#### Real-World Example: Chrome Safe Browsing

Checks URLs against known malicious sites without sending every URL to Google.

```
User visits https://example.com
           │
           ▼
┌─────────────────────────────┐
│ Local Bloom filter          │  ← Updated periodically
│ (millions of bad URL hashes)│     from Google
└──────────────┬──────────────┘
               │
       ┌───────┴───────┐
       │               │
   "Definitely      "Maybe
    safe"            unsafe"
       │               │
       ▼               ▼
   Allow           Query Google
   immediately     servers to confirm
```

**Privacy benefit:** Only potential matches are sent to Google, not every URL visited.

---

### Bloom Filter Interview Checklist

| Topic | Key Points |
|-------|------------|
| **What it is** | Probabilistic set membership; bit array + k hash functions |
| **Core guarantee** | No false negatives; possible false positives |
| **Insert** | Hash k times, set k bits to 1 |
| **Lookup** | Hash k times, check if all k bits are 1 |
| **False positive rate** | p ≈ (1 - e^(-kn/m))^k |
| **Optimal k** | k = 0.693 × (m/n) |
| **Space formula** | m = -1.44 × n × ln(p) |
| **Why no deletion** | Clearing shared bits causes false negatives |
| **Counting Bloom** | Counters instead of bits; enables deletion; 4x space |
| **Cuckoo filter** | Fingerprints + cuckoo hashing; deletion + space efficient |
| **Key use cases** | Database lookups, caching, deduplication, safe browsing |

