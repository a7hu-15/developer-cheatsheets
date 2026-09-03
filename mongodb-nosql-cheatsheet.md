# 🍃 MongoDB & NoSQL Database Architecture Cheatsheet

A production reference guide for MongoDB document modeling, indexing strategies, Aggregation Pipeline stages, Replica Set & Sharding architectures, and query diagnostics.

---

## 🍃 MongoDB Core Concepts & Index Types

| Index Type | Purpose & Behavior | Example Usage |
|---|---|---|
| **Single Field** | Default index on `_id` or single field | `db.users.createIndex({ email: 1 })` |
| **Compound Index** | Multi-field index following Equality, Sort, Range (ESR) rule | `db.orders.createIndex({ status: 1, createdAt: -1 })` |
| **Multikey Index** | Indexes array elements automatically | `db.posts.createIndex({ tags: 1 })` |
| **Text Index** | Full-text search over string fields | `db.articles.createIndex({ title: "text", content: "text" })` |
| **TTL Index** | Automatically deletes documents after specified seconds | `db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })` |
| **Partial Index** | Indexes documents matching filter expression | `db.users.createIndex({ email: 1 }, { partialFilterExpression: { active: true } })` |
| **Wildcard Index** | Indexes dynamic or arbitrary document subfields | `db.products.createIndex({ "attributes.$**": 1 })` |

---

## 🛠️ CRUD & Index Management Syntax

### CRUD Operations
```js
// Insert documents
db.users.insertOne({ name: "Alice", email: "alice@example.com", status: "ACTIVE", roles: ["admin"] });

// Query with Operators
db.users.find({ status: "ACTIVE", age: { $gte: 21, $lte: 65 } }).sort({ createdAt: -1 }).limit(10);

// Atomic Updates with $set, $inc, and $push
db.users.updateOne(
  { _id: ObjectId("60d5ec49f1a2c80015f8e4a1") },
  { $set: { lastLogin: new Date() }, $inc: { loginCount: 1 } }
);

// Upsert (Insert if absent, Update if present)
db.analytics.updateOne(
  { date: "2026-09-03", page: "/home" },
  { $inc: { visits: 1 } },
  { upsert: true }
);
```

### Index Operations
```js
// Create Compound Index with Unique constraint
db.users.createIndex({ tenantId: 1, email: 1 }, { unique: true });

// View existing indexes & sizes
db.users.getIndexes();

// Drop inefficient or unused index
db.users.dropIndex("idx_status_old");
```

---

## ⚡ Aggregation Pipeline Reference

```js
db.orders.aggregate([
  // Stage 1: Filter documents early (reduce dataset size)
  { $match: { status: "DELIVERED", orderDate: { $gte: ISODate("2026-01-01") } } },

  // Stage 2: Join with customer collection ($lookup)
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  },

  // Stage 3: Unwind array from join
  { $unwind: "$customerInfo" },

  // Stage 4: Group and calculate aggregates
  {
    $group: {
      _id: "$customerInfo.tier",
      totalRevenue: { $sum: "$totalAmount" },
      avgOrderValue: { $avg: "$totalAmount" },
      orderCount: { $sum: 1 }
    }
  },

  // Stage 5: Project & format final fields
  {
    $project: {
      _id: 0,
      tier: "$_id",
      totalRevenue: { $round: ["$totalRevenue", 2] },
      avgOrderValue: { $round: ["$avgOrderValue", 2] },
      orderCount: 1
    }
  },

  // Stage 6: Sort results
  { $sort: { totalRevenue: -1 } }
]);
```

---

## 🏗️ Data Modeling Patterns (Embed vs. Reference)

| Pattern | Rule of Thumb | Best Use Case |
|---|---|---|
| **Embedding (1-to-1 / 1-to-Few)** | Data accessed together, bounded array size (< 16MB document limit) | User address, invoice line items, metadata |
| **Referencing (1-to-Many / Many-to-Many)** | Unbounded arrays, frequently modified independently | E-commerce orders, user followers, audit logs |
| **Subset Pattern** | Store top N elements embedded, reference full history | Product reviews (embed top 5, reference full review collection) |
| **Bucket Pattern** | Group time-series data into aggregated time buckets | IoT sensor metrics, hourly pageview counters |

---

## 🌐 High Availability & Sharding Architecture

### Replica Set Write Concerns & Read Preferences
- **Write Concern `w: "majority"`**: Ensures commit is acknowledged by majority of data-bearing nodes before returning.
- **Write Concern `j: true`**: Guarantees write is flushed to on-disk journal.
- **Read Preference `primary`**: Default. All reads go to primary node (strong consistency).
- **Read Preference `secondaryPreferred`**: Offloads read traffic to secondaries (eventual consistency).

### Sharding Architecture Components
```
[ Client Application ]
        │
        ▼
   [ mongos ]  ◄── (Router & Query Coordinator)
    │     │
    │     └───────────────────┐
    ▼                         ▼
[ Shard A ] (Replica Set)   [ Shard B ] (Replica Set)
    ▲
    └────── [ Config Server Replica Set ] (Metadata & Chunk mappings)
```

- **Hashed Sharding**: Uniform data distribution based on hash value of shard key.
- **Ranged Sharding**: Groups documents by range. Ideal for prefix queries, but watch for monotonically increasing keys (write hotspots).

---

## 🩺 Diagnostics & Query Performance Tuning

```js
// 1. Inspect execution plan (index usage & examined vs returned docs)
db.orders.find({ customerId: "C123", status: "PENDING" }).explain("executionStats");

// 2. Identify long-running queries (> 3 seconds)
db.currentOp({
  "active": true,
  "secs_running": { $gt: 3 },
  "op": { $ne: "none" }
});

// 3. Terminate runaway query by OpID
db.killOp(12345);

// 4. Check cluster database stats & index sizes
db.stats();
db.orders.stats();
```
