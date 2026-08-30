# ⚡ Apache Kafka & Event-Driven Architecture Cheatsheet

A production reference guide for Apache Kafka, event-driven messaging, partition strategies, consumer group mechanics, delivery guarantees, log compaction, and CLI administration commands.

---

## 🏗️ Core Concepts & Architecture

| Concept | Description |
|---|---|
| **Topic** | Logical stream of records/events categorized by name. |
| **Partition** | Ordered, immutable log of records appended sequentially; unit of parallelism and scale in Kafka. |
| **Offset** | Sequential integer assigned to each record in a partition identifying its position. |
| **Broker** | Kafka server instance handling storage, partition leadership, and request processing. |
| **Consumer Group** | Set of consumers working cooperatively to consume data from topic partitions. |
| **Zookeeper / KRaft** | Metadata management and consensus protocol (KRaft replaces ZooKeeper in modern Kafka). |

---

## 📤 Producer Delivery Semantics & Partitioning

### Delivery Guarantees (`acks`)

```properties
# Producer configuration properties
acks=0          # Fire and forget; highest throughput, potential data loss
acks=1          # Leader acknowledgement; balance of durability and speed
acks=all (-1)   # Leader + all in-sync replicas (ISR) acknowledge; max durability
enable.idempotence=true  # Prevents duplicate messages caused by retries
```

### Partitioning Strategies
- **Key-Based Partitioning**: Messages with the same key are hashed (`murmur2`) and routed to the exact same partition, guaranteeing strict per-key ordering.
- **Round-Robin / Sticky Partitioning**: Messages without keys are batched together efficiently across partitions to maximize batch throughput.

---

## 📥 Consumer Groups & Offset Management

### Partition Rebalancing Mechanisms
- **RangeAssignor**: Assigns contiguous partition ranges per consumer.
- **CooperativeStickyAssignor**: Incremental rebalancing avoiding "stop-the-world" consumer group pauses.

### Offset Commit Strategies

```python
# Python confluent-kafka / kafka-python offset commit example
# Manual synchronous offset commit after business logic execution
try:
    msg = consumer.poll(1.0)
    if msg is not None:
        process_event(msg.value())
        consumer.commit(asynchronous=False)
except Exception as err:
    log.error("Failed processing event: %s", err)
```

---

## 💻 Essential Kafka CLI Administration Commands

### Topic Management (`kafka-topics.sh`)

```bash
# Create a topic with 6 partitions and replication factor of 3
kafka-topics.sh --bootstrap-server localhost:9092 --create \
  --topic order-events \
  --partitions 6 \
  --replication-factor 3

# List and describe topic details
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic order-events

# Alter topic configuration (e.g. increase partition count)
kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic order-events --partitions 12
```

### Console Producer & Consumer (`kafka-console-producer.sh` / `kafka-console-consumer.sh`)

```bash
# Produce messages with Key-Value separator
kafka-console-producer.sh --bootstrap-server localhost:9092 \
  --topic order-events \
  --property "parse.key=true" \
  --property "key.separator=:"

# Consume messages from beginning with keys and timestamps displayed
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic order-events \
  --from-beginning \
  --property "print.key=true" \
  --property "print.timestamp=true"
```

### Consumer Group Monitoring (`kafka-consumer-groups.sh`)

```bash
# List all active consumer groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# Inspect consumer group lag per partition
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group order-processing-service

# Reset consumer group offset to earliest / latest
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group order-processing-service \
  --topic order-events \
  --reset-offsets --to-earliest --execute
```

---

## 🧹 Retention & Log Compaction

```properties
# Topic-level retention configurations
cleanup.policy=compact          # Retain only the latest value for each key
cleanup.policy=delete           # Delete segments exceeding time/size limits
retention.ms=604800000          # Retention time in milliseconds (7 days)
retention.bytes=10737418240     # Max topic log size before deletion (10 GB)
min.cleanable.dirty.ratio=0.5   # Ratio of dirty log before log compaction triggers
```
