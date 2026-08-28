# 🔴 Redis & In-Memory Caching Reference Cheatsheet

A production cheatsheet for Redis data structures, CLI commands, persistence options, memory eviction strategies, and high-availability setups.

---

## 🔑 Core Data Structures & CLI Commands

### Strings & Key TTL
```bash
# Key-value string operations
SET user:1000 "Alice" EX 3600    # Set value with 1 hour expiration
GET user:1000                   # Retrieve value
INCR page_views:home            # Atomic integer increment
TTL user:1000                   # Check remaining time-to-live in seconds
EXPIRE user:1000 7200           # Update key expiration
```

### Hashes (Objects)
```bash
# Store user profile hash map
HSET user:1001 name "Bob" email "bob@example.com" role "admin"
HGET user:1001 email
HGETALL user:1001
HINCRBY user:1001 login_count 1
```

### Lists (Queues & Stacks)
```bash
# Push elements to queue
LPUSH task_queue "job_1" "job_2"
RPOP task_queue                 # FIFO Queue pop
BLPOP task_queue 5              # Blocking FIFO queue pop with 5s timeout
```

### Sets & Sorted Sets (Leaderboards)
```bash
# Unordered Sets
SADD online_users "user_1" "user_2"
SISMEMBER online_users "user_1"

# Sorted Sets (ZSet) - Ranked Leaderboard
ZADD leaderboard 1500 "PlayerA" 2100 "PlayerB" 1850 "PlayerC"
ZREVRANGE leaderboard 0 2 WITHSCORES   # Top 3 players with scores
ZRANK leaderboard "PlayerA"             # Get zero-based rank
```

---

## 🧹 Memory Eviction Policies

When Redis reaches `maxmemory`, it evicts keys according to the selected policy:

| Policy | Behavior |
| :--- | :--- |
| `volatile-lru` | Evicts least recently used keys with an explicit expiration TTL. |
| `allkeys-lru` | Evicts least recently used keys out of all keys. Recommended for caching. |
| `volatile-lfu` | Evicts least frequently used keys with an explicit expiration TTL. |
| `allkeys-random` | Randomly evicts keys out of all keys. |
| `noeviction` | Returns errors when memory limit is reached (default for DB usage). |

---

## 💾 Persistence Options

- **RDB (Redis Database Snapshots)**: Point-in-time binary snapshots written to disk at specified intervals.
  ```text
  save 900 1      # Save snapshot if at least 1 key changed in 900s
  save 300 10     # Save snapshot if at least 10 keys changed in 300s
  ```
- **AOF (Append Only File)**: Logs every write operation received by the server.
  ```text
  appendonly yes
  appendfsync everysec   # Write and sync log file every second (balanced)
  ```

---

## 🛰️ Pub/Sub & Sentinel Commands

```bash
# Publish & Subscribe messaging
SUBSCRIBE notifications:events
PUBLISH notifications:events "User registered: #1002"

# Redis Sentinel & Cluster Status
redis-cli -h 127.0.0.1 -p 26379 SENTINEL master mymaster
redis-cli INFO memory
redis-cli INFO stats
```
