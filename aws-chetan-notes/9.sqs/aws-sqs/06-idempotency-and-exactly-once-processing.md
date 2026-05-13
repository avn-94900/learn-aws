# 06 — Idempotency & Exactly-Once Processing

---

## What is Idempotency?

An operation is **idempotent** if it can be executed multiple times and always produces the same result as the first execution, with no additional side effects.

**Why it matters for SQS:**

- Standard queues provide **at-least-once** delivery — the same message can be delivered more than once
- Consumer crashes, network issues, or visibility timeout expiry can all cause redelivery
- Without idempotency, you risk duplicate charges, duplicate database rows, or duplicate notifications

---

## Delivery Semantics: Standard vs FIFO

| | Standard Queue | FIFO Queue |
|---|---|---|
| **Delivery** | At-least-once (duplicates possible) | Exactly-once per `MessageGroupId` |
| **Ordering** | Best-effort (may arrive out of order) | Strict FIFO within each message group |
| **Throughput** | Nearly unlimited | 300–70,000 TPS |
| **Deduplication** | Must be handled by your application | Handled by SQS via deduplication ID |

---

## Handling Duplicates in Standard Queues

Since standard queues can deliver duplicates, your consumer must detect and ignore them.

### Strategy 1 — Deduplicate by SQS Message ID

```java
@SqsListener("${sqs.standard.queue}")
public void handleMessage(String message,
                          @Header("MessageId") String messageId) {

    if (trackingRepository.existsByMessageId(messageId)) {
        log.info("Duplicate message ignored, ID: {}", messageId);
        return;
    }

    processMessage(message);
    trackingRepository.save(new MessageTrack(messageId));
}
```

### Strategy 2 — Deduplicate by Business Identifier

More reliable — based on the actual business event, not just the SQS message ID.

```java
@SqsListener("${sqs.standard.queue}")
public void handleMessage(String message) {
    String orderId = extractOrderId(message);  // parse from JSON body

    if (orderRepository.existsById(orderId)) {
        log.info("Order already processed, skipping: {}", orderId);
        return;
    }

    processOrder(message);
}
```

### Strategy 3 — Database Upsert (Safest)

Use a unique constraint at the database level so even concurrent duplicates are handled correctly.

```sql
-- Unique constraint on business key
INSERT INTO orders (order_id, customer_id, amount, status)
VALUES (:orderId, :customerId, :amount, 'PENDING')
ON CONFLICT (order_id) DO NOTHING;
```

### Cleanup Old Tracking Records

```java
@Scheduled(fixedRate = 3600000) // every hour
public void cleanupOldTrackingRecords() {
    LocalDateTime cutoff = LocalDateTime.now().minusHours(24);
    trackingRepository.deleteByCreatedTimeBefore(cutoff);
}
```

---

## Idempotency Patterns Comparison

### Pattern 1 — Database-Based

```java
@Entity
public class ProcessedMessage {
    @Id
    private String messageId;
    private String businessId;
    private String contentHash;
    private LocalDateTime processedAt;
    private String result;
}

// Usage
if (processedMessageRepository.existsById(messageId)) {
    return; // already processed
}
processMessage(message);
processedMessageRepository.save(new ProcessedMessage(messageId, ...));
```

**Pros**: Persistent across restarts, queryable  
**Cons**: DB call on every message, requires cleanup job

### Pattern 2 — Cache-Based (Redis)

```java
@Service
public class CacheBasedIdempotency {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    public boolean isAlreadyProcessed(String key) {
        return Boolean.TRUE.equals(redisTemplate.hasKey("processed:" + key));
    }

    public void markProcessed(String key, String result) {
        redisTemplate.opsForValue().set(
            "processed:" + key,
            result,
            Duration.ofHours(24)  // TTL = 24 hours
        );
    }
}
```

**Pros**: Fast, TTL handles cleanup automatically  
**Cons**: Lost on Redis restart (unless persisted), may miss records after TTL

### Pattern 3 — Business Logic Idempotency (Preferred Where Possible)

```java
@Service
public class BusinessLogicIdempotency {

    @Autowired
    private OrderRepository orderRepository;

    public boolean isOrderAlreadyProcessed(String orderId) {
        Order order = orderRepository.findById(orderId).orElse(null);
        return order != null && order.getStatus() == OrderStatus.PROCESSED;
    }
}
```

**Pros**: No extra tracking table; business entity state is the source of truth  
**Cons**: Requires careful status management; can miss some scenarios

---

## Idempotency with Distributed Locks (Advanced)

For high-concurrency scenarios where multiple consumers might race to process the same message:

```java
@SqsListener("${sqs.queue.orders}")
public void processOrder(String orderMessage,
                         @Header("MessageId") String messageId) {

    String idempotencyKey = extractOrderId(orderMessage);
    String lockKey = "lock:order:" + idempotencyKey;

    // Acquire distributed lock (Redis SET NX with TTL)
    Boolean lockAcquired = redisTemplate.opsForValue()
        .setIfAbsent(lockKey, messageId, Duration.ofMinutes(5));

    if (!Boolean.TRUE.equals(lockAcquired)) {
        log.warn("Could not acquire lock for order {}, skipping", idempotencyKey);
        return;
    }

    try {
        if (isAlreadyProcessed(idempotencyKey)) {
            return; // double-check after acquiring lock
        }

        transactionTemplate.execute(status -> {
            Order order = parseOrder(orderMessage);
            orderService.createOrder(order);
            markAsProcessed(idempotencyKey, messageId);
            return null;
        });

    } finally {
        redisTemplate.delete(lockKey); // always release the lock
    }
}
```

---

## FIFO Queue Deduplication

FIFO queues handle deduplication at the SQS layer, preventing duplicates within a **5-minute window**.

### Option 1 — Explicit Deduplication ID

You provide a unique `MessageDeduplicationId` per message.

```java
public void sendWithExplicitDeduplication(String queueUrl, OrderEvent event) {
    String deduplicationId = DigestUtils.sha256Hex(
        event.getOrderId() +
        event.getEventType() +
        event.getTimestamp().toEpochSecond()
    );

    SendMessageRequest request = new SendMessageRequest()
        .withQueueUrl(queueUrl)
        .withMessageBody(objectMapper.writeValueAsString(event))
        .withMessageGroupId(event.getCustomerId())
        .withMessageDeduplicationId(deduplicationId);

    amazonSQS.sendMessage(request);
}
```

### Option 2 — Content-Based Deduplication

Enable on the FIFO queue — SQS automatically generates the deduplication ID from the MD5 hash of the message body. No need to set `MessageDeduplicationId` in code.

```java
public void sendWithContentDeduplication(String queueUrl, OrderEvent event) {
    SendMessageRequest request = new SendMessageRequest()
        .withQueueUrl(queueUrl)
        .withMessageBody(objectMapper.writeValueAsString(event))
        .withMessageGroupId(event.getCustomerId());
        // MessageDeduplicationId intentionally omitted — SQS uses content hash

    amazonSQS.sendMessage(request);
}
```

> **Use content-based deduplication when** the same message body should always represent the same logical event (i.e., identical bodies = duplicate). Use explicit IDs when the same body can represent distinct events.

---

## Idempotency Decision Guide

| Scenario | Recommended Approach |
|---|---|
| Standard queue, simple business entities | Business logic idempotency (check entity state) |
| Standard queue, high-throughput | Redis cache with TTL |
| Standard queue, auditability required | Database tracking table |
| High-concurrency consumers on same queue | Distributed lock (Redis) + database |
| FIFO queue | SQS-level deduplication (explicit ID or content-based) |
| GET requests | Naturally idempotent — no extra handling needed |
| PUT requests | Naturally idempotent (replace entire resource) |
| POST requests | Require explicit idempotency key |
