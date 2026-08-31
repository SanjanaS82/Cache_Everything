# Cache_Everything

1. What is caching? Explain its usage taking different use cases like in computer organization and web apps.
Caching is the practice of storing copies of data in a high-speed, temporary storage layer (the cache) so that future requests for that data can be served faster than fetching it from the primary, slower source (like main memory, a database, or a remote server).
The fundamental goal is to exploit locality of reference:
- Temporal Locality: Data accessed recently is likely to be accessed again soon.
- Spatial Locality: Data stored near recently accessed data is likely to be accessed soon.
Key Use Cases
Computer Architecture & Organization
In hardware, CPU speeds far outpace main memory access times (the memory wall). Hardware caches sit between CPU registers and System RAM (DDR).
- L1 Cache: Built directly into the CPU core, runs at CPU clock speed (~1–2 ns latency), very small (32–128 KB).
- L2 Cache: Slightly larger (512 KB–2 MB per core), slightly slower (~3–5 ns latency).
- L3 Cache: Shared across all CPU cores (16–128+ MB), slower than L2 (~10–20 ns latency), but dramatically faster than RAM (~60–100 ns).
- TLB (Translation Lookaside Buffer): A specialized hardware cache storing page table translations (Virtual Memory Address --> Physical Memory Address) to bypass expensive page table walks in RAM.
Web Applications & Distributed Systems
In software, network latency and database I/O are the primary bottlenecks.
- Database Query Caching: Storing the results of complex SQL queries (e.g., product catalogs or user profiles) in memory using tools like Redis to avoid heavy database CPU execution.
- CDN (Content Delivery Network): Geographically distributed proxy servers (e.g., Cloudflare, CloudFront) caching static assets (HTML, CSS, JS, images, videos) close to end users to cut down round-trip times (RTT).
- Browser Caching: Browsers store static HTTP responses in local disk memory based on Cache-Control headers, avoiding network requests on repeat page loads.

2. Introduction of Redis and Memcache
Both Redis and Memcached are open-source, in-memory key-value stores used to reduce database load and speed up response times, but they serve different architectural needs.
- Use Memcached for simple, static key-value caching where speed and multi-core efficiency are paramount (e.g., simple HTML fragment caching).
- Use Redis when complex data structures, data persistence, publish/subscribe messaging, or high availability via failover are required.

3. What are cache eviction strategies? Explain need and when to use which one.
Because cache memory is limited and expensive, when the cache reaches full capacity, an eviction strategy dictates which item to remove to make room for new data.
Standard Strategies: 
1. Least Recently Used (LRU)
Mechanism: Evicts the item that has not been accessed for the longest duration of time.
When to use: General-purpose caching (web sessions, database query results). Works best when recent accesses strongly predict near-future accesses.

2. Least Frequently Used (LFU)
Mechanism: Tracks access counters for each key and evicts the item with the smallest hit count.
When to use: Long-term caching where popularity matters more than recency (e.g., static assets, viral content, frequency-based recommendations).
Drawback: Old popular items can permanently occupy memory even after their popularity wanes ( mitigated using "LFU with decay").

3. First-In, First-Out (FIFO)
Mechanism: Evicts items in the exact order they were inserted, regardless of how often or recently they were accessed.
When to use: Simple systems with minimal overhead requirements or where access patterns follow a simple linear stream.

4. Time-To-Live (TTL) / Eviction by Expiry
Mechanism: Removes keys once a specified lifetime duration expires.
When to use: Volatile data like authentication tokens, OTPs, or API rate limit counters.

5. Random Replacement (RR)
Mechanism: Randomly selects a candidate key for eviction.
When to use: Hardware implementations or memory-constrained edge devices where the overhead of maintaining metadata (pointers or access counters) is too costly.
