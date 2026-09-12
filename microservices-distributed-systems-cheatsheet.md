# 🌐 Microservices & Distributed Systems Cheatsheet

A practical reference for microservice architecture patterns, resilience mechanisms, distributed transactions, event-driven integration, and observability.

---

## 🏛️ Core Architectural Patterns

### Saga Pattern (Distributed Transactions)
Maintains data consistency across multiple microservice databases without two-phase commit (2PC).

```
Orchestration-Based Saga:
[ Order Service ] ---> (Saga Orchestrator) ---> 1. Reserve Stock (Inventory)
                                          ---> 2. Charge Payment (Payment)
                                          ---> 3. Dispatch (Shipping)
                                          (On Failure -> Compensating Tx)
```

- **Choreography**: Services publish events; other services listen and act autonomously. Best for simple workflows with few services.
- **Orchestration**: A centralized coordinator controls step execution and compensating transactions. Best for complex enterprise workflows.
- **Compensating Transaction**: Reverses committed local transactions if a downstream step fails (e.g., refund payment if shipping fails).

### CQRS (Command Query Responsibility Segregation)
Separates read and write data models to optimize performance, scalability, and security.

```
Command Path: Client -> Command API -> Write DB (Normalized, Relational)
                                          |
                                   (Event Bus / Kafka)
                                          |
Query Path:   Client <-  Query API  <- Read DB (Denormalized, Elasticsearch/Redis)
```

### Transactional Outbox Pattern
Guarantees atomic updates to the database and event publishing to a message broker without dual-write race conditions.

```sql
-- Insert business record & outbox event within the SAME database transaction
BEGIN TRANSACTION;
INSERT INTO orders (id, user_id, total) VALUES ('ord-123', 'usr-456', 99.50);
INSERT INTO outbox (id, aggregate_type, payload) VALUES ('evt-789', 'OrderCreated', '{"id":"ord-123"}');
COMMIT;
-- CDC / Polling Publisher (e.g. Debezium) forwards outbox rows to Kafka & marks processed
```

---

## ⚡ Resilience & Fault Tolerance

### Circuit Breaker Pattern
Prevents cascading failures when a downstream service becomes unhealthy or unresponsive.

```
[ Closed ] --(Errors > Threshold)--> [ Open ] --(Timeout Expires)--> [ Half-Open ]
    ^                                   |                                  |
    |-------(Probing Succeeds)----------+--------(Probing Fails)-----------|
```

- **States**:
  - `Closed`: Requests flow normally. Failures are tracked.
  - `Open`: Fast-fail immediately without calling target service.
  - `Half-Open`: Allow limited test traffic to check downstream recovery.

```python
# Conceptual Python Circuit Breaker Decorator
class CircuitBreakerOpenException(Exception): pass

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30):
        self.threshold = failure_threshold
        self.timeout = recovery_timeout
        self.failures = 0
        self.state = "CLOSED"
        self.last_state_change = time.time()

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            if self.state == "OPEN":
                if time.time() - self.last_state_change > self.timeout:
                    self.state = "HALF-OPEN"
                else:
                    raise CircuitBreakerOpenException("Circuit open: service unavailable")
            try:
                result = func(*args, **kwargs)
                if self.state == "HALF-OPEN":
                    self.state = "CLOSED"
                    self.failures = 0
                return result
            except Exception as e:
                self.failures += 1
                if self.failures >= self.threshold:
                    self.state = "OPEN"
                    self.last_state_change = time.time()
                raise e
        return wrapper
```

### Rate Limiting & Bulkheads
- **Token Bucket / Leaky Bucket**: Smooths out traffic spikes and enforces API request rate limits.
- **Bulkhead Pattern**: Isolates resources (thread pools, connection pools) per downstream service so a slow service cannot exhaust all application threads.

---

## 🔍 Distributed Tracing & Observability

### W3C Trace Context Propagation
Pass trace headers across HTTP/gRPC RPC calls to reconstruct end-to-end request flows.

```
Traceparent Header Format:
version - trace_id (32 hex) - parent_id / span_id (16 hex) - trace_flags (2 hex)
00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

```python
# Injecting Traceparent Header in Python requests
headers = {
    "traceparent": f"00-{trace_id}-{span_id}-01",
    "Content-Type": "application/json"
}
response = requests.post("https://payment-service/api/charge", headers=headers)
```

---

## 🛠️ Communication Protocols: gRPC vs REST vs WebSockets

| Metric | REST (HTTP/1.1) | gRPC (HTTP/2) | WebSockets |
|---|---|---|---|
| **Payload Format** | JSON / XML (Text) | Protocol Buffers (Binary) | Frame Payload (Binary/Text) |
| **Transport** | HTTP/1.1 | HTTP/2 (Multiplexed) | TCP / WS Frame |
| **Streaming** | Server-Sent Events (SSE) | Unidirectional & Bi-directional | Full Duplex |
| **Performance** | Standard | High (Small payload size) | Low overhead real-time |
| **Use Case** | Public Web APIs | Internal Microservice Communication | Real-time chat & notifications |

---

## 📋 Best Practices Summary
1. **Database-per-Service**: Never allow microservices to share the same physical database tables directly.
2. **Idempotent Consumers**: Ensure event consumers can process duplicate messages safely (e.g., using message IDs as deduplication keys).
3. **Graceful Degradation**: Fall back to cached data or partial responses when non-critical downstream services fail.
4. **Health Endpoints**: Expose `/healthz/live` (liveness) and `/healthz/ready` (readiness) endpoints for Kubernetes probes.
