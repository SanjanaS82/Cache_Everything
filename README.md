# Cache_Everything

## 1. What is Caching?

**Caching** is the practice of storing copies of data in a high-speed, temporary storage layer (the **cache**) so that future requests for that data can be served faster than fetching it from the primary, slower storage layer (such as main memory, a database, or a remote API).

### Locality of Reference
The efficacy of caching relies on the principle of locality:

* **Temporal Locality:** Data accessed recently is likely to be accessed again in the near future.
* **Spatial Locality:** Data stored near recently accessed data addresses is likely to be accessed soon.

---

### Key Use Cases

#### A. Computer Architecture & Organization
Hardware caches mitigate the **"Memory Wall"**—the performance gap caused by CPU clock speeds outpacing main memory (RAM) access times.

| Cache Tier | Location / Characteristics | Access Latency | Typical Capacity |
| :--- | :--- | :--- | :--- |
| **L1 Cache** | Built directly into CPU core | ~1–2 ns | 32 KB – 128 KB |
| **L2 Cache** | Dedicated per core or shared pair | ~3–5 ns | 512 KB – 2 MB |
| **L3 Cache** | Shared across all CPU cores | ~10–20 ns | 16 MB – 128+ MB |
| **TLB** | Translation Lookaside Buffer (Page table hardware cache) | < 1 ns | Hundreds of entries |

> **System RAM Access Latency:** ~60–100 ns (up to 100x slower than L1).

#### B. Web Applications & Distributed Systems
Software caching alleviates network latency, disk I/O bottlenecks, and heavy database computation.

* **Database Query Caching:** In-memory caching of heavy computational query results (e.g., product catalogs, user profiles) using key-value stores.
* **Content Delivery Networks (CDNs):** Geographically distributed edge proxies (e.g., Cloudflare, AWS CloudFront) caching static assets (HTML, CSS, images, video segments) near end users to reduce Round-Trip Time (RTT).
* **Browser Caching:** Local disk/memory storage of static HTTP responses guided by `Cache-Control` headers to prevent redundant network trips on repeated page views.

---

### Quick Decision Matrix

* **Choose Memcached when:**
  * You need a straightforward, multi-threaded cache layer for simple key-value pairs (e.g., HTML fragment caching).
  * Scale-up performance on multi-core machines without persistence is your sole requirement.

* **Choose Redis when:**
  * You require rich data structures (Hashes, Sorted Sets, Bitmaps, Streams).
  * You need data persistence across server restarts or high-availability failovers (Replication, Cluster).
  * You need advanced capabilities like Pub/Sub messaging, geospatial queries, or Lua scripting.

---

## 3. Cache Eviction Strategies

Because cache memory is limited and costly, an **eviction strategy** dictates which key to purge when capacity is reached.
                      [ Cache at Capacity ]
                              |
           +------------------+------------------+
           |                                     |
  [ Frequency / Recency ]                  [ Time / Order ]
    ├── LRU (Least Recently Used)            ├── FIFO (First-In, First-Out)
    └── LFU (Least Frequently Used)          ├── TTL (Time-To-Live)
                                             └── RR (Random Replacement)
| Strategy | Mechanism | Recommended Use Case |
| :--- | :--- | :--- |
| **LRU** *(Least Recently Used)* | Evicts items that haven't been accessed for the longest duration. | General-purpose caching (web sessions, database query results). |
| **LFU** *(Least Frequently Used)* | Tracks access frequency counters and purges keys with the lowest hit counts. | Long-term caching where item popularity outweighs recency (e.g., viral media). |
| **FIFO** *(First-In, First-Out)* | Evicts keys in exact insertion order, regardless of access patterns. | Simple pipelines with low metadata overhead requirements. |
| **TTL** *(Time-To-Live)* | Automatically purges keys once an explicit expiration interval completes. | Volatile time-sensitive data (e.g., OTP tokens, API rate limiters). |
| **RR** *(Random Replacement)* | Randomly selects a candidate key to evict. | Hardware implementations where maintaining access tracking metadata is too costly. |

> **Note on LFU:** LFU can suffer from "cache pollution" if historical items accumulate high counters and remain stuck in memory. Use **LFU with decay** to systematically lower counters over time.

---

## 4. LRU Cache Architecture: CDLL vs. Arrays & SLL

An optimal **LRU Cache** requires **$O(1)$ constant time** execution for both operations:
1. `get(key)` — Fetch the item and mark it as most recently used.
2. `put(key, value)` — Insert or update an item. Evict the least recently used item if capacity is exceeded.

To achieve $O(1)$ operations, an LRU Cache pairs a **Hash Map** with a **Circular Doubly Linked List (CDLL)** or **Doubly Linked List (DLL)**.
