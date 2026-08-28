# 🏗️ System Design & Microservices Reference Cheatsheet

A high-level architectural reference guide for building scalable, reliable, and fault-tolerant distributed systems.

---

## ⚖️ Scalability & Load Balancing

| Strategy | Description | Common Use Cases |
| :--- | :--- | :--- |
| **Vertical Scaling (Scale-Up)** | Adding CPU/RAM resources to a single server node. | Monolithic apps, initial database servers. |
| **Horizontal Scaling (Scale-Out)** | Adding more machine nodes to pool workload capacity. | Microservices, stateless web servers. |
| **Round-Robin Load Balancing** | Distributes incoming requests sequentially across servers. | Uniform server capacity workloads. |
| **Least Connections** | Directs traffic to node with fewest active requests. | Long-lived WebSocket or database connections. |
| **IP Hash / Consistent Hashing** | Maps requests to nodes using client IP hash values. | Session affinity, distributed caching. |

---

## 🏛️ System Core Theorems

### CAP Theorem
In a network partition ($P$), a distributed system can guarantee only **Consistency ($C$)** or **Availability ($A$)**:
- **CP Systems** (e.g. HBase, MongoDB): Prioritize exact consistency over availability under network split.
- **AP Systems** (e.g. Cassandra, DynamoDB): Prioritize availability over immediate consistency under split.

### BASE Consistency Model
- **Basically Available**: System guarantees availability according to SLA.
- **Soft State**: System state may change over time without user interaction.
- **Eventual Consistency**: System will become consistent given sufficient time.

---

## 💾 Caching & Database Scaling Patterns

```
Client  ──►  Load Balancer  ──►  App Server  ──►  Cache (Redis)  ──►  Database (PostgreSQL)
```

### Caching Strategies
- **Cache-Aside (Lazy Loading)**: App checks cache first; on miss, reads DB and populates cache.
- **Write-Through**: App writes data to cache, cache synchronously writes to DB.
- **Write-Behind (Write-Back)**: App writes data to cache, cache asynchronously flushes updates to DB in batches.

### Database Partitioning & Sharding
- **Vertical Partitioning**: Splitting tables across databases by domain (e.g. User DB vs Order DB).
- **Horizontal Sharding**: Splitting table rows across database nodes by a shard key (e.g. `user_id % 4`).

---

## 🚦 Rate Limiting Algorithms

1. **Token Bucket**: Refills tokens at constant rate; allows burst traffic up to bucket capacity.
2. **Leaky Bucket**: Queues incoming requests and processes them at a fixed output rate.
3. **Fixed Window**: Tracks request counts per fixed timeframe (e.g. 100 reqs/min); vulnerable to boundary bursts.
4. **Sliding Window Log / Counter**: Smooths request rates by calculating weighted moving averages over time windows.

---

## 💬 Message Queues & Event-Driven Architecture

- **Point-to-Point Queue** (e.g. RabbitMQ): Messages consumed by exactly one worker node.
- **Publish-Subscribe (Pub/Sub)** (e.g. Apache Kafka): Event logs published to topics and consumed independently by multiple consumer groups.
- **Idempotency**: Designing message consumers so receiving duplicate messages produces the exact same system state.
