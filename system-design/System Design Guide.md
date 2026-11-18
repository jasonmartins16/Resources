# System Design: Interview Study Guide

All 51 system design topics from the study list, each explained with what it is, why it matters and the trade-offs interviewers expect you to discuss.

## Part 1: Foundations

### 1. Client-Server Architecture

Clients (browsers, apps) send requests; servers process them and respond. The server holds the logic and data, the client handles presentation. Most systems are layered: client, web/app server, database. Statelessness of the server tier is what makes scaling it out easy.

&#91;embedded content: client-server architecture\]

### 2. Scalability (Vertical vs Horizontal)

**Vertical** scaling adds CPU/RAM to one machine: simple but capped and a single point of failure. **Horizontal** scaling adds more machines behind a load balancer: nearly unlimited and fault tolerant, but needs stateless services and distributed-data solutions.

&#91;embedded content: vertical vs horizontal scaling\]

### 3. Load Balancing

Distributes incoming traffic across servers to improve throughput and availability. Algorithms: round robin, least connections, IP hash, weighted. Layer 4 balances on TCP/IP; Layer 7 on HTTP content (paths, headers). Uses health checks to remove failed nodes. The balancer itself must be redundant.

&#91;embedded content: load balancing with health checks\]

### 4. Caching (CDN, Redis, Memcached)

Storing frequently read data in fast storage to cut latency and database load. Layers: browser, CDN, application cache (Redis/Memcached), database cache. Redis supports rich data types and persistence; Memcached is a simple in-memory key-value store. Hard part: invalidation and staleness.

&#91;embedded content: cache layers on the request path\]

### 5. Database Sharding

Splitting data across multiple databases (shards) by a shard key so no single node holds everything. Enables write scaling. Challenges: choosing the key (avoid hotspots), cross-shard joins and transactions, and resharding. Strategies: range, hash, directory-based.

&#91;embedded content: database sharding by key\]

### 6. Database Replication

Keeping copies of data on multiple nodes. **Leader-follower**: writes go to the leader, reads can use followers (scales reads, gives failover). **Multi-leader** and **leaderless** exist for multi-region writes. Replication can be sync (safe, slower) or async (fast, risks replication lag and data loss on failover).

&#91;embedded content: leader-follower replication\]

### 7. SQL vs NoSQL

SQL: structured schema, joins, ACID transactions: good for relational, consistent data. NoSQL: key-value, document, column-family, graph; flexible schemas and easier horizontal scaling, often weaker consistency. Choose by access patterns, consistency needs and scale, not by fashion.

&#91;embedded content: SQL vs NoSQL\]

### 8. CAP Theorem

During a network **P**artition, a distributed system must choose between **C**onsistency (every read sees the latest write) and **A**vailability (every request gets a response). Partitions are unavoidable, so the real choice is CP vs AP. Refinement: PACELC adds latency vs consistency trade-offs when there is no partition.

&#91;embedded content: CAP theorem\]

### 9. Consistency Models (Strong vs Eventual)

**Strong**: after a write completes, all reads see it (linearizability): simple to reason about, higher latency. **Eventual**: replicas converge if writes stop: fast and available but reads may be stale. In between: read-your-writes, monotonic reads, causal consistency.

&#91;embedded content: strong vs eventual consistency\]

### 10. Indexing

A data structure (usually B-tree, or hash/LSM) that speeds lookups at the cost of extra storage and slower writes. Know composite indexes (column order matters), covering indexes, and that indexing low-cardinality columns or over-indexing hurts.

&#91;embedded content: database index (B-tree)\]

## Part 2: Communication and Architecture

### 11. Message Queues (Kafka, RabbitMQ)

Decouple producers from consumers via asynchronous messages, smoothing spikes and enabling retries. **RabbitMQ** is a traditional broker with routing and per-message acknowledgement. **Kafka** is a distributed, partitioned, replayable log built for high-throughput streaming. Discuss delivery guarantees (at-most/at-least/exactly once) and ordering within partitions.

&#91;embedded content: message queue\]

### 12. Microservices vs Monolith

A monolith is one deployable unit: simple to build, test and deploy early on. Microservices split by business capability, each independently deployable and scalable, but add network calls, distributed data, observability and ops complexity. Start monolithic; split when team size or scale demands it.

&#91;embedded content: monolith vs microservices\]

### 13. API Design (REST, GraphQL, gRPC)

**REST**: resource-oriented HTTP, simple and cacheable. **GraphQL**: clients specify exactly the fields they need in one request, solving over/under-fetching, but caching and query-cost control are harder. **gRPC**: binary Protocol Buffers over HTTP/2, fast and typed, ideal for internal service-to-service calls.

&#91;embedded content: REST vs GraphQL vs gRPC\]

### 14. API Gateway

Single entry point in front of services handling routing, authentication, rate limiting, TLS termination, request aggregation and logging. Simplifies clients but must be kept highly available and not become a bottleneck or business-logic dumping ground.

&#91;embedded content: API gateway\]

### 15. Rate Limiting

Caps how many requests a client can make in a time window to protect services from abuse and overload. Typically keyed by user, API key or IP and implemented at the gateway with a shared store like Redis. Returns HTTP 429. See item 43 for algorithms.

&#91;embedded content: rate limiter in the request path\]

### 16. Consistent Hashing

Maps both keys and nodes onto a hash ring; each key goes to the next node clockwise. Adding or removing a node moves only about 1/N of keys, unlike `hash(key) % N` which reshuffles nearly everything. **Virtual nodes** balance load. Used in caches, DynamoDB, Cassandra.

&#91;embedded content: consistent hashing ring\]

### 17. Proxies (Forward and Reverse)

A **forward proxy** sits in front of clients (anonymity, filtering, caching). A **reverse proxy** sits in front of servers (load balancing, TLS termination, caching, protection), e.g. Nginx or HAProxy. Clients of a reverse proxy do not know the real backends.

&#91;embedded content: forward vs reverse proxy\]

### 18. Content Delivery Network (CDN)

A geographically distributed network of edge servers that cache static (and some dynamic) content near users, cutting latency and origin load. Pull CDNs fetch on first request; push CDNs are pre-populated. Control freshness with TTLs, cache headers and purges.

&#91;embedded content: content delivery network\]

### 19. Fault Tolerance and High Availability

Fault tolerance is continuing to operate despite failures; high availability is maximizing uptime ("nines"). Techniques: redundancy, replication, failover, health checks, multi-AZ deployment, graceful degradation and eliminating single points of failure.

&#91;embedded content: fault tolerance across zones\]

### 20. Observability (Logging, Metrics, Tracing)

The ability to understand a system from its outputs. **Logs**: discrete events. **Metrics**: numeric time series (latency, error rate, saturation). **Traces**: follow a request across services. Together with alerting, they let you debug distributed systems. Know the RED and USE methods.

&#91;embedded content: logs, metrics and traces\]

## Part 3: Data and Reliability Concepts

### 21. Back-of-the-envelope Estimation

Quick calculations of QPS, storage, bandwidth and memory to sanity-check a design. Memorize powers of two/ten, seconds per day (\~86,400), and latency numbers (RAM ns, SSD tens of µs, cross-region ms). Always state assumptions and round aggressively.

&#91;embedded content: back-of-the-envelope estimation steps\]

### 22. Latency vs Throughput

**Latency** is time per request; **throughput** is requests (or bytes) per second. They are related but not the same: batching raises throughput while hurting latency. Track tail latency (p95/p99), not just averages.

&#91;embedded content: latency vs throughput\]

### 23. ACID vs BASE

**ACID** (Atomicity, Consistency, Isolation, Durability) is the traditional transactional guarantee. **BASE** (Basically Available, Soft state, Eventual consistency) is the relaxed model of many distributed NoSQL stores, trading strictness for availability and scale.

&#91;embedded content: ACID vs BASE\]

### 24. Transaction Isolation Levels

Control what concurrent transactions can see: **Read Uncommitted** (dirty reads), **Read Committed** (no dirty reads), **Repeatable Read** (stable row reads, phantoms possible in some DBs), **Serializable** (as if run one at a time). Higher isolation means fewer anomalies but less concurrency.

&#91;embedded content: isolation levels vs anomalies\]

### 25. Normalization vs Denormalization

Normalization removes redundancy by splitting data into related tables, which keeps writes consistent. Denormalization deliberately duplicates data to avoid joins and speed reads, which is common at scale but needs care to keep copies in sync.

&#91;embedded content: normalization vs denormalization\]

### 26. Cache Eviction Policies (LRU, LFU, FIFO)

What to remove when the cache is full. **LRU** evicts the least recently used (the common default). **LFU** evicts the least frequently used. **FIFO** evicts the oldest inserted. LRU is often implemented with a hash map plus doubly linked list for O(1) operations.

&#91;embedded content: LRU eviction\]

### 27. Caching Strategies (write-through, write-back, cache-aside)

**Cache-aside**: the app checks the cache, on a miss reads the DB and fills the cache; the most common. **Write-through**: writes go to cache and DB together (consistent, slower writes). **Write-back**: write to cache first and flush to the DB later (fast but risks data loss).

&#91;embedded content: cache-aside, write-through, write-back\]

### 28. Idempotency

An operation that produces the same result however many times it is applied. Essential for safe retries, e.g. payments. Implement with idempotency keys, unique constraints or conditional writes. GET, PUT and DELETE are idempotent by definition; POST usually is not.

&#91;embedded content: idempotent retry\]

### 29. Distributed Transactions (2PC, Saga)

**Two-Phase Commit**: a coordinator asks all participants to prepare, then commit: strongly consistent but blocking and fragile. **Saga**: a sequence of local transactions with compensating actions to undo on failure: more available, eventually consistent, and typical in microservices.

&#91;embedded content: two-phase commit vs saga\]

### 30. Consensus and Leader Election (Raft, Paxos)

Algorithms that let nodes agree on a value or leader despite failures. **Raft** is designed to be understandable: leader election by terms and majority votes, then log replication. **Paxos** is the older, harder equivalent. Used by etcd, ZooKeeper and Consul.

&#91;embedded content: Raft leader election and replication\]

### 31. Quorum (read/write)

Requiring a majority of replicas to acknowledge. With N replicas, write quorum W and read quorum R, **R + W > N** guarantees reads overlap the latest write. Tunable in Dynamo-style systems, trading latency against consistency.

&#91;embedded content: read/write quorum overlap\]

### 32. Pub/Sub and Event-Driven Architecture

Publishers emit events to topics without knowing who consumes them; subscribers react independently. Gives loose coupling and easy extension. Costs: harder tracing, eventual consistency, ordering and duplicate handling.

&#91;embedded content: publish/subscribe\]

### 33. CQRS and Event Sourcing

**CQRS** separates the write model (commands) from the read model (queries) so each can be optimized and scaled. **Event Sourcing** stores every state change as an immutable event; current state is derived by replaying events. They pair well but add significant complexity.

&#91;embedded content: CQRS and event sourcing\]

### 34. Service Discovery

How services find each other's changing addresses. **Client-side** (client queries a registry like Consul or Eureka) or **server-side** (load balancer or Kubernetes DNS resolves). Needs health checking and registration/deregistration.

&#91;embedded content: service discovery\]

### 35. Circuit Breaker, Retries and Exponential Backoff

A **circuit breaker** stops calling a failing dependency (closed, open, half-open) to avoid cascading failure. **Retries** handle transient errors; **exponential backoff with jitter** spaces them out to avoid retry storms. Retries require idempotent operations.

&#91;embedded content: circuit breaker states\]

### 36. Backpressure

Signaling upstream to slow down when a component is overloaded, rather than letting queues grow unbounded. Tactics: bounded queues, blocking, dropping or rejecting load (shedding), and rate-limiting producers.

&#91;embedded content: backpressure\]

### 37. Bloom Filters

A compact probabilistic set: it can say "definitely not present" or "probably present" (false positives, never false negatives). Used to skip disk lookups, check cache membership or detect duplicates cheaply.

&#91;embedded content: Bloom filter lookup\]

### 38. Gossip Protocol and Heartbeats

**Gossip**: nodes periodically exchange state with random peers so information spreads epidemically; scalable and fault tolerant (used by Cassandra). **Heartbeats**: periodic "I am alive" messages used for failure detection; missing several marks a node as suspect or dead.

&#91;embedded content: gossip and heartbeats\]

### 39. WebSockets, Long Polling and Server-Sent Events

Ways to push data to clients. **Long polling**: the client holds a request open until data is ready. **SSE**: one-way server-to-client stream over HTTP. **WebSockets**: persistent full-duplex connection, best for chat and games. WebSockets need sticky or stateful connection management.

&#91;embedded content: long polling vs SSE vs WebSockets\]

### 40. Unique ID Generation (Snowflake, UUID)

**UUID**: easy and decentralized but large and non-sortable (v4). **Snowflake**: 64-bit IDs combining timestamp, machine ID and sequence: roughly time-ordered, compact and generated without coordination. Auto-increment IDs do not work across shards.

&#91;embedded content: Snowflake ID layout\]

## Part 4: Security, Storage and Operations

### 41. Authentication and Authorization (OAuth 2.0, JWT, sessions)

**Authentication** proves who you are; **authorization** decides what you may do. **Sessions** store state server-side (easy to revoke). **JWT** is a signed, self-contained token (stateless but hard to revoke; keep short-lived). **OAuth 2.0** delegates limited access to third parties via tokens (with OIDC adding identity).

&#91;embedded content: OAuth 2.0 authorization code flow\]

### 42. Database Partitioning (horizontal vs vertical)

**Horizontal** partitioning splits rows across tables or nodes (sharding is its distributed form). **Vertical** partitioning splits columns, moving rarely used or large columns elsewhere. Partitioning improves manageability and performance on large tables.

&#91;embedded content: horizontal vs vertical partitioning\]

### 43. Rate Limiting Algorithms (token bucket, leaky bucket, sliding window)

**Token bucket**: tokens refill at a steady rate, each request spends one; allows bursts. **Leaky bucket**: requests drain at a fixed rate, smoothing output. **Fixed window**: simple counter per window, but allows boundary bursts. **Sliding window** (log or counter) is more accurate.

&#91;embedded content: token bucket and sliding window\]

### 44. Distributed Locks

Mutual exclusion across machines, e.g. via Redis (`SET key value NX PX ttl`), ZooKeeper or etcd. Pitfalls: lock expiry while work continues, clock drift, and process pauses. Use TTLs and fencing tokens, and prefer designs that avoid locks when possible.

&#91;embedded content: distributed lock with fencing token\]

### 45. Object/Blob Storage

Stores unstructured data (images, video, backups) as objects with keys and metadata in a flat namespace, e.g. S3. Highly durable, cheap and scalable, but not suited for low-latency random writes. Use pre-signed URLs and put a CDN in front.

&#91;embedded content: object storage with CDN\]

### 46. Search Systems (inverted index, Elasticsearch)

An **inverted index** maps each term to the documents containing it, enabling fast full-text search with ranking (TF-IDF/BM25). **Elasticsearch** is a distributed search engine built on Lucene. It is a search index, not a primary database: keep your source of truth elsewhere.

&#91;embedded content: inverted index\]

### 47. Geospatial Indexing (Geohash, Quadtree)

**Geohash** encodes a location into a string where shared prefixes mean proximity, enabling range queries. **Quadtree** recursively divides space into four quadrants, adapting to density. Used for "nearby" features in ride-sharing and maps.

&#91;embedded content: geohash vs quadtree\]

### 48. Data Warehouses, Data Lakes and ETL

A **data warehouse** holds structured, modeled data for analytics (Redshift, BigQuery). A **data lake** holds raw data of any format cheaply (S3). **ETL/ELT** moves and transforms data between systems. Separate analytics (OLAP) from transactional (OLTP) workloads.

&#91;embedded content: data lake, warehouse and ETL\]

### 49. SLA, SLO and SLI

**SLI**: a measured indicator (e.g. request success rate). **SLO**: an internal target for it (99.9%). **SLA**: a contractual commitment with consequences. Error budgets derived from SLOs guide how fast a team can ship.

&#91;embedded content: SLI, SLO and SLA\]

### 50. Disaster Recovery (RPO, RTO) and Multi-Region Design

**RPO**: how much data loss is tolerable; **RTO**: how long recovery may take. Strategies from cheapest/slowest to costliest/fastest: backup and restore, pilot light, warm standby, active-active multi-region. Multi-region adds data replication and consistency challenges.

&#91;embedded content: RPO, RTO and recovery strategies\]

### 51. Classic Design Problems

Practice with a repeatable framework: clarify requirements, estimate scale, define APIs, sketch high-level design, then deep-dive on bottlenecks.

&#91;embedded content: design interview framework\]

- **URL shortener**: base62 IDs, key-value store, heavy caching, redirect 301/302.
- **Chat app**: WebSockets, message queue, per-conversation ordering, presence, offline delivery.
- **News feed**: fan-out on write vs on read (hybrid for celebrities), ranking, caching.
- **Ride-sharing**: geospatial index, matching service, location updates, surge pricing.
- **Video streaming**: upload and transcoding pipeline, adaptive bitrate (HLS), CDN.
- **Notification system**: queue, templates, multi-channel providers, retries, dedup, user preferences.
- **Rate limiter**: token bucket in Redis at the gateway, per-key rules, distributed counters.

---

## Answer Framework

1. Clarify functional and non-functional requirements.
2. Estimate scale (users, QPS, storage).
3. Define APIs and data model.
4. Draw the high-level design.
5. Deep-dive on bottlenecks, then discuss trade-offs, failure modes and monitoring.

---

## Part 5: Complete Worked Examples

Three full walkthroughs that follow the five-step framework: clarify requirements, estimate scale, define APIs and data, sketch the design, then deep-dive on trade-offs. The numbers are assumptions chosen to be easy to reason about; in an interview, state your own assumptions out loud and round aggressively.

## Example 1: URL Shortener

### 1. Requirements

**Functional:** create a short link from a long URL (optionally with a custom alias and expiry); redirect a short link to the original URL; optionally count clicks.

**Non-functional:** redirects must be fast (tens of milliseconds) and highly available; short codes must be unique; the system is read-heavy (many more redirects than creations); links should be hard to guess.

### 2. Estimation (assumptions)

- 100 million new URLs per month is about 40 writes per second (100M / \~2.6M seconds).
- A 100:1 read-to-write ratio gives about 4,000 redirects per second on average; plan for roughly 10,000 per second at peak.
- Over 5 years: 100M x 12 x 5 = 6 billion URLs. At about 500 bytes each, that is roughly 3 TB: small enough for a sharded key-value store.
- Code length: base62 (a-z, A-Z, 0-9) with 7 characters gives 62^7, about 3.5 trillion codes, far more than 6 billion.
- Cache: about 350 million redirects per day; caching the hottest 20% at 500 bytes each needs roughly 35 GB, which fits in a Redis cluster.

### 3. API and data model

```
POST   /api/v1/urls     body: { longUrl, alias?, expiresAt? }  ->  { shortUrl }
GET    /{code}          ->  HTTP 302 redirect to the long URL
DELETE /api/v1/urls/{code}
```

A single table is enough: `urls(code PK, long_url, user_id, created_at, expires_at)`. Access is always by key, so a key-value or wide-column store (DynamoDB, Cassandra) shards naturally by `code`.

### 4. High-level design

&#91;embedded content: URL shortener architecture\]

1. **Create:** the app server gets a unique ID, encodes it in base62, writes `code -> longUrl` to the database, and returns the short URL.
2. **Redirect:** look up `code` in Redis; on a miss read the database, fill the cache, and return a redirect.
3. **Analytics:** publish a click event to a queue and process it asynchronously so it never slows the redirect.

### 5. Deep dive and trade-offs

- **Generating unique codes.** Options: (a) hash the long URL and take the first 7 characters, which risks collisions that you must detect and retry; (b) **a counter encoded in base62**, which is collision-free and simple; (c) a pre-generated key service. A good choice is (b) with a central **ID range service** that hands each app server a block of IDs (for example, 1 million at a time) so servers generate codes locally with no per-request coordination. Sequential IDs are guessable, so scramble them with a reversible permutation or add a few random bits if links should be unguessable.
- **301 vs 302.** A 301 (permanent) is cached by browsers, which cuts load but hides later clicks from you; a 302 (temporary) means every click reaches your servers, which is what click analytics need. Pick 302 if you count clicks.
- **Custom aliases.** Use a conditional write (insert only if the key does not exist) so two users cannot claim the same alias.
- **Caching.** Cache-aside with an LRU eviction policy; the hot set is small relative to the whole dataset.
- **Scaling the database.** Shard by hash of `code` using consistent hashing; replicate each shard for availability.
- **Expiry.** Check `expires_at` lazily on read and run a background job to purge old entries.
- **Abuse.** Rate-limit creation per user or IP, and scan submitted URLs for malware or phishing.
- **Availability.** Stateless app servers behind a load balancer across multiple AZs; a database replica takes over if a primary fails. Redirects can survive a brief analytics outage because analytics is asynchronous.

## Example 2: Chat Application

### 1. Requirements

**Functional:** one-to-one and group chat; real-time delivery; message history; delivery states (sent, delivered, read); online presence; works across a user's multiple devices; push notifications for offline users.

**Non-functional:** low latency (a few hundred milliseconds end to end); no lost messages; correct ordering within a conversation; high availability. Media sharing and end-to-end encryption are noted as extensions.

### 2. Estimation (assumptions)

- 50 million daily active users sending 40 messages each is 2 billion messages per day, about 23,000 per second on average and roughly 70,000 per second at peak.
- At about 100 bytes per message including metadata, that is about 200 GB per day, or about 73 TB per year.
- If 10 million users are connected at once and a tuned gateway server holds about 100,000 idle connections, you need roughly 100 gateway servers (plus headroom).

### 3. Protocol, APIs and data model

Clients keep a **WebSocket** connection open to a gateway. Main events:

```
client -> server:  send_message { conversationId, clientMsgId, body }
server -> client:  ack { clientMsgId, messageId }           (persisted)
server -> client:  message { conversationId, messageId, senderId, body, ts }
client -> server:  delivered / read { conversationId, messageId }
REST:              GET /conversations/{id}/messages?before={messageId}&limit=50
```

**Storage:** messages go in a wide-column store (Cassandra or DynamoDB) with partition key `conversation_id` and clustering key `message_id` (a time-sortable ID such as a Snowflake ID), so a conversation's history is one contiguous read. Users, conversations and memberships fit a relational database. Presence lives in Redis with a short TTL refreshed by heartbeats.

### 4. High-level design

&#91;embedded content: chat application architecture\]

1. The sender's app sends the message over its WebSocket to **Gateway A**.
2. The **chat service** assigns a message ID, **persists** it, and sends an ack so the sender knows it was stored.
3. It looks up each recipient's gateway in the **connection registry** (user to gateway mapping in Redis).
4. It forwards the message to **Gateway B**, which pushes it down the recipient's WebSocket; the recipient app acks delivery.
5. If the recipient is offline, the **push service** sends a mobile notification, and the app syncs missed messages when it reconnects.

### 5. Deep dive and trade-offs

- **Persist before delivering.** Writing first means a crash never loses a message that was acked; delivery can be retried from storage.
- **Ordering.** Messages in a conversation are ordered by their monotonically increasing message ID. Clients display by that ID, not arrival time.
- **Duplicates and retries.** The client generates a `clientMsgId`; the server treats a repeat with the same ID as the same message (idempotency), so retries after timeouts are safe.
- **Reconnection and catch-up.** On reconnect the client sends its last seen `messageId` and the server returns everything after it. Heartbeats detect dead connections; the gateway cleans up the registry entry.
- **Groups.** For small groups, fan out to each member's gateway. For very large groups, put delivery on a queue (Kafka) so slow recipients do not block the sender, and let clients pull history.
- **Multi-device.** Each device has its own connection and its own sync cursor; the registry stores a set of gateways per user.
- **Scaling.** Gateways are stateful (they hold connections), so scale them horizontally behind a layer-4 load balancer; keep chat services stateless; shard the message store by `conversation_id`.
- **Extensions.** Media goes to object storage and is delivered through a CDN using pre-signed URLs; end-to-end encryption (for example the Signal protocol) means servers store only ciphertext.

## Example 3: News Feed (Social Timeline)

### 1. Requirements

**Functional:** users publish posts; users follow other users; opening the app shows a feed of recent posts from people they follow, newest or best first; the feed paginates infinitely.

**Non-functional:** fast feed loads (a couple of hundred milliseconds); high availability; eventual consistency is acceptable (a new post may take a few seconds to appear); very high read volume.

### 2. Estimation (assumptions)

- 100 million daily active users, each opening the feed 10 times a day: 1 billion feed reads per day, about 11,600 per second on average and roughly 35,000 at peak.
- 50 million posts per day is about 580 writes per second.
- With an average of 200 followers, fan-out on write means 50M x 200 = 10 billion feed inserts per day, about 115,000 per second. That is manageable for normal users but dangerous for accounts with millions of followers.
- Feed cache: keep the latest 500 post IDs per user at 8 bytes each, about 4 KB per user, or about 400 GB for 100 million users, spread across a Redis cluster.
- Posts: about 1 KB of text and metadata each is about 50 GB per day; images and video live in object storage behind a CDN.

### 3. APIs and data model

```
POST /v1/posts                { text, mediaIds[] }
GET  /v1/feed?cursor=<id>&limit=20
POST /v1/follow/{userId}      DELETE /v1/follow/{userId}
```

- **posts:** `post_id` (time-sortable), `author_id`, `text`, `media_url`, `created_at`, sharded by `post_id` or `author_id`.
- **follows:** edges `(follower_id, followee_id)` with lookups in both directions (who do I follow, who follows me).
- **feed cache:** per user, a sorted list of post IDs in Redis.

### 4. High-level design

&#91;embedded content: news feed write and read paths\]

**Write path:** the post service stores the post and publishes an event to a queue. **Fan-out workers** read the author's followers and push the post ID into each follower's feed cache.

**Read path:** the feed service reads the user's post IDs from the feed cache, merges in recent posts from any celebrity accounts they follow, **hydrates** the IDs into full posts (batch fetch from a post cache or database), ranks them, and returns one page with a cursor.

### 5. Deep dive and trade-offs

- **Fan-out on write vs on read.** Fan-out on write makes reads very fast but costs a lot when an author has millions of followers. Fan-out on read (compute the feed when requested) is cheap to write but slow to read. The practical answer is **hybrid**: fan out on write for normal users, and for celebrity accounts (above a follower threshold) skip fan-out and pull their latest posts at read time.
- **Inactive users.** Do not fan out to users who have not opened the app recently; rebuild their feed on their next login.
- **Pagination.** Use a cursor (the last seen post ID or timestamp), not page offsets, so new posts do not shift pages and duplicates do not appear.
- **Ranking.** Start with reverse chronological order; add a ranking service that scores posts by recency, relationship strength and engagement, often using an ML model, applied to a few hundred candidates.
- **Hot keys and caching.** Cache popular posts and counters separately; shard the feed cache by `user_id`; replicate hot posts to avoid a single overloaded node.
- **Deletes and edits.** Hydration looks up the current post, so a deleted or hidden post is simply filtered out at read time.
- **Consistency.** Feeds are eventually consistent; a short delay between posting and appearing is acceptable, and the author's own post can be inserted into their feed immediately.
- **Failure handling.** The queue absorbs bursts and lets workers retry; if fan-out lags, feeds are slightly stale, not broken.

## Example 4: Rate Limiter

### 1. Requirements

**Functional:** limit how many requests a client (user, API key or IP) can make per time window according to configurable rules, for example 100 requests per minute per user for one endpoint; reject excess requests with HTTP 429 and tell the client when to retry.

**Non-functional:** adds only a few milliseconds of latency; works correctly across many gateway servers (a distributed counter); highly available; accurate enough that bursts cannot slip through; rules can change without a redeploy.

### 2. Estimation (assumptions)

- 500,000 requests per second at peak across all gateways means 500,000 limiter checks per second, so a single Redis node is not enough: shard the counters.
- 50 million active clients in an hour with about 100 bytes of state each is about 5 GB, which fits comfortably in a Redis cluster.
- A token bucket needs only two numbers per key (tokens and last refill time).

### 3. API, rules and data model

The check is an internal call or in-process middleware, not a public API:

```
allow(clientKey, ruleId)  ->  { allowed, remaining, retryAfterSeconds }

Response headers:  X-RateLimit-Limit, X-RateLimit-Remaining, Retry-After
```

**Rules** (`service, endpoint, tier, limit, window, algorithm`) are small and change rarely, so keep them in a config store and cache them in each gateway. **Counters** are key-value pairs in Redis keyed by something like `rl:{ruleId}:{clientKey}` with a TTL so idle clients disappear.

### 4. High-level design

&#91;embedded content: rate limiter architecture\]

1. A request arrives at the API gateway, which identifies the client (API key, user ID, or IP as a fallback).
2. The limiter middleware loads the matching rule from its local cache and runs an atomic check against Redis.
3. If allowed, the request goes to the backend; otherwise the gateway returns 429 with a `Retry-After` header.

### 5. Deep dive and trade-offs

- **Choosing the algorithm.** **Token bucket** allows controlled bursts and needs tiny state (a good default). **Leaky bucket** smooths output to a steady rate. **Fixed window** is simplest but allows up to twice the limit across a window boundary. **Sliding window counter** blends two windows for good accuracy at low cost; **sliding window log** is exact but stores every timestamp.
- **Atomicity.** Reading, updating and writing a counter from many gateways races unless the whole step is atomic. Run it as a single Redis Lua script (Redis executes it without interleaving). Example token bucket:

```lua
-- KEYS[1] bucket key; ARGV: capacity, refill per second, now in ms
local data = redis.call('HMGET', KEYS[1], 'tokens', 'ts')
local tokens = tonumber(data[1]) or tonumber(ARGV[1])
local ts = tonumber(data[2]) or tonumber(ARGV[3])
local elapsed = (tonumber(ARGV[3]) - ts) / 1000
tokens = math.min(tonumber(ARGV[1]), tokens + elapsed * tonumber(ARGV[2]))
local allowed = 0
if tokens >= 1 then
  tokens = tokens - 1
  allowed = 1
end
redis.call('HSET', KEYS[1], 'tokens', tokens, 'ts', ARGV[3])
redis.call('PEXPIRE', KEYS[1], 120000)
return allowed
```

- **Clock skew.** Pass time from one source (Redis `TIME` or a single service) instead of each gateway's clock.
- **Scaling Redis.** Shard keys with consistent hashing; give each shard a replica. Watch for hot keys (one very busy client).
- **Latency.** To avoid a network call per request, keep a small local allowance per gateway and sync with Redis in batches. This is approximate but much faster; use it where slight over-admission is acceptable.
- **Failure mode.** If Redis is unreachable, choose deliberately: **fail open** (allow traffic, protect availability) for most APIs, or **fail closed** for sensitive endpoints such as login or payments.
- **Identifying clients.** Prefer API key or user ID over IP, since many users can share one IP behind a NAT; combine several limits (per user and per IP).
- **Multi-region.** Either enforce per-region limits (simple, slightly loose globally) or replicate counters asynchronously (accurate eventually).
- **Scope.** Application limits are not a DDoS defense. Put coarse protection at the edge (CDN or WAF) and fine-grained business limits in the gateway.

## Example 5: Notification System

### 1. Requirements

**Functional:** send notifications over several channels (mobile push, email, SMS, in-app); support event-triggered messages (order shipped, one-time password) and bulk campaigns; respect user preferences, opt-outs and quiet hours; use templates and localization; support scheduling; track delivery status.

**Non-functional:** urgent messages (one-time passwords) arrive within seconds even during a campaign; messages are not lost (at-least-once) and not duplicated visibly; the system scales to bursts; third-party providers can fail without taking the system down.

### 2. Estimation (assumptions)

- 10 million users receiving about 5 notifications a day is 50 million per day, about 580 per second on average; campaigns can create 10 times that, around 6,000 per second.
- Suppose 70% push, 25% email, 5% SMS: about 35M push, 12.5M email and 2.5M SMS messages a day. SMS is the costly channel, so it deserves tight controls.
- A delivery log of about 200 bytes per notification is about 10 GB per day.

### 3. API and data model

```
POST /v1/notifications
  { userId | segmentId, templateId, params, channels?, priority, sendAt?, idempotencyKey }
GET  /v1/notifications/{id}          -> status per channel
PUT  /v1/users/{id}/preferences      -> channels, opt-outs, quiet hours
```

Tables: `devices(user_id, platform, token)`, `preferences(user_id, channel, opted_in, quiet_hours)`, `templates(id, version, locale, body)`, `notification_log(id, user_id, channel, status, ts)`.

### 4. High-level design

&#91;embedded content: notification system architecture\]

1. **Notification API** validates the request, checks the `idempotencyKey` to drop duplicates, and loads the user's preferences and the template.
2. It renders the message and places one job per channel on that channel's queue (and priority level).
3. **Workers** for each channel call the provider (APNs or FCM for push, an email provider, an SMS provider) at a rate the provider allows.
4. Providers report results through **callbacks**, which update the delivery status store.
5. **Campaigns:** a batch job expands a segment into messages in chunks and feeds the same queues at a controlled rate.

### 5. Deep dive and trade-offs

- **Isolation by queue.** Separate queues per channel and per priority so a marketing blast cannot delay a one-time password. Give high-priority queues dedicated workers.
- **Reliability.** Use at-least-once delivery with retries and exponential backoff; after the last attempt, send the job to a dead-letter queue for inspection. De-duplicate with the idempotency key so retries do not send twice.
- **Provider failures.** Add a secondary provider (for SMS and email) with automatic failover; use circuit breakers to stop hammering a failing provider.
- **User respect and compliance.** Check opt-outs and quiet hours (in the user's time zone) before sending; honor unsubscribe links; cap notifications per user per day.
- **Provider limits and cost.** Throttle workers to each provider's rate limits; batch where the provider supports it; monitor SMS spend.
- **Device hygiene.** Remove invalid push tokens when providers report them; support several devices per user.
- **Templates.** Version templates and render per locale; validate parameters so a bad template does not reach users.
- **Ordering.** Exact ordering across notifications is usually unnecessary; use timestamps and collapse keys where only the latest matters.
- **Observability.** Track queue depth, send latency, delivery and failure rates per channel and provider, and alert on anomalies.

## Example 6: Ride-Sharing Service

### 1. Requirements

**Functional:** a rider requests a trip with pickup and drop-off; sees nearby drivers, a fare estimate and an ETA; the system matches a nearby available driver who can accept or decline; both sides see live location during the trip; trip states are tracked from request to completion; payment happens at the end.

**Non-functional:** matching within a few seconds; location data is fresh; a driver is never assigned two rides at once; the system stays available during rush hour and in busy areas (airports, events).

### 2. Estimation (assumptions)

- 5 million rides per day is about 58 per second on average, perhaps 300 per second at peak.
- 1 million drivers online, each sending a location update every 4 seconds, is 250,000 updates per second. At about 100 bytes each that is roughly 25 MB per second, or about 2 TB per day if everything were stored.
- The matching system only needs each driver's **latest** position: 1 million drivers x 100 bytes is about 100 MB, which fits easily in memory. Store only sampled trip traces long term.

### 3. APIs and data model

```
POST /v1/rides            { pickup, dropoff, type }          -> { rideId, fareEstimate }
POST /v1/drivers/location { lat, lng, heading, ts }          (every few seconds)
POST /v1/rides/{id}/accept   POST /v1/rides/{id}/decline
GET  /v1/rides/{id}       -> status, driver, location, eta
```

- **rides:** `ride_id, rider_id, driver_id, status, pickup, dropoff, fare, timestamps` in a sharded relational or key-value database. Status is a state machine: requested, matched, driver arriving, in progress, completed or cancelled.
- **driver state:** current location and availability in a geospatial index, for example Redis GEO or a custom index over geohash, S2 or H3 cells.

### 4. High-level design

&#91;embedded content: ride-sharing architecture\]

1. The driver app streams location to the **location service**, which updates the **geo index** (sharded by city or region).
2. A rider requests a ride; the **ride service** creates it in the trip database and asks the geo index for available drivers near the pickup point.
3. It asks the **ETA and routing** service for real travel times over the road network and ranks candidates.
4. It offers the ride to the best driver (push or WebSocket) with a short timeout; if declined or ignored, it offers to the next one.
5. On acceptance it marks the driver unavailable and the ride matched, then both apps get live updates until completion; payment runs afterward.

### 5. Deep dive and trade-offs

- **Geospatial indexing.** Divide the map into cells (geohash, S2 or H3). To find nearby drivers, query the rider's cell plus its neighbors, and widen the search ring if there are too few. A quadtree adapts to density: small cells downtown, large cells in sparse areas.
- **Write load.** 250,000 location updates per second must not hit a disk database. Keep the latest position in memory, shard by region, and use adaptive update rates (less often when idle or parked).
- **Avoiding double assignment.** Driver availability must be consistent: use an atomic conditional update (set status to reserved only if currently available) or a per-driver single writer. Offer to one driver at a time, or a small batch with a very short timeout, and cancel the rest on first acceptance.
- **ETA, not straight-line distance.** A driver 500 meters away across a river may be 15 minutes away. Rank by routing-based ETA that uses the road graph and live traffic.
- **Surge pricing.** Compute supply and demand per cell every minute and apply a multiplier where demand outstrips supply; smooth the changes so prices do not flicker.
- **Failure handling.** If a location shard loses its memory, drivers re-send their position within seconds and the index rebuilds itself. Trip state is persisted in the database so rides survive service restarts. Make payment calls idempotent.
- **Scaling.** Partition everything by city or region; scale the ride service horizontally; use a queue (Kafka) for trip events feeding analytics, notifications and billing.
- **Hot spots.** Airports and stadiums create sudden local demand: reserve queues for pickup zones and pre-warm capacity near scheduled events.
- **Privacy and safety.** Share exact location only while a trip is active, retain traces briefly, and give both sides trip-sharing and emergency options.
