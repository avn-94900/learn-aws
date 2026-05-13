# 10 — Quick Reference & Cheat Sheet

---

## SQS Key Numbers (Memorise These)

| Parameter | Value |
|---|---|
| Max message size | 256 KB (2 GB with S3 Extended Client) |
| Message retention | 60 seconds – 14 days (default: 4 days) |
| Visibility timeout | 0 seconds – 12 hours (default: 30 seconds) |
| Delay queue (all messages) | 0 – 15 minutes |
| Per-message delay | 0 – 15 minutes |
| Long polling wait | 1 – 20 seconds |
| Batch size | Up to 10 messages per API call |
| Max message attributes | 10 per message |
| FIFO deduplication window | 5 minutes |
| In-flight limit (Standard) | 120,000 messages |
| In-flight limit (FIFO) | 20,000 messages |
| Standard throughput | Nearly unlimited |
| FIFO classic throughput | 300 TPS (3,000 with batching) |
| FIFO high-throughput | 70,000 TPS (2023+) |

---

## SDK Operations Quick Map

| Operation | SDK v1 Method | SDK v2 Method |
|---|---|---|
| Send message | `amazonSQS.sendMessage()` | `sqsClient.sendMessage()` |
| Send batch | `amazonSQS.sendMessageBatch()` | `sqsClient.sendMessageBatch()` |
| Receive message | `amazonSQS.receiveMessage()` | `sqsClient.receiveMessage()` |
| Delete message | `amazonSQS.deleteMessage()` | `sqsClient.deleteMessage()` |
| Delete batch | `amazonSQS.deleteMessageBatch()` | `sqsClient.deleteMessageBatch()` |
| Change visibility | `amazonSQS.changeMessageVisibility()` | `sqsClient.changeMessageVisibility()` |
| Get queue URL | `amazonSQS.getQueueUrl()` | `sqsClient.getQueueUrl()` |
| Get queue attributes | `amazonSQS.getQueueAttributes()` | `sqsClient.getQueueAttributes()` |
| Purge queue | `amazonSQS.purgeQueue()` | `sqsClient.purgeQueue()` |

---

## Spring Boot Annotations Reference

```java
// Listen to a queue (auto-delete on success)
@SqsListener("queue-name")

// Listen with explicit deletion policy
@SqsListener(value = "queue-name", deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)
// ON_SUCCESS — delete if no exception (default)
// NEVER      — manual: call ack.acknowledge()
// NO_REDRIVE — delete only if no DLQ configured

// Inject payload and headers
public void listen(@Payload String message,
                   @Header Map<String, Object> headers,
                   @Header("MessageId") String messageId) { }

// Listen to multiple queues
@SqsListener({"queue-a", "queue-b"})

// From properties
@SqsListener("${sqs.queue.name}")
```

---

## CloudWatch Metrics Reference

| Metric | What It Measures | Alarm Threshold |
|---|---|---|
| `ApproximateNumberOfMessagesVisible` | Messages waiting in queue | Set based on consumer capacity |
| `ApproximateAgeOfOldestMessage` | Latency of oldest unprocessed message | > 300 seconds (alert) |
| `ApproximateNumberOfMessagesNotVisible` | Messages currently being processed | Monitor for stuck messages |
| `NumberOfMessagesSent` | Producer throughput | — |
| `NumberOfMessagesReceived` | Consumer throughput | — |
| `NumberOfMessagesDeleted` | Successfully processed messages | — |
| `NumberOfEmptyReceives` | Wasted polls (use long polling to reduce) | High count → enable long polling |

---

## Message Lifecycle State Machine

```
Producer
    │
    │ SendMessage()
    ▼
[AVAILABLE]
    │
    │ ReceiveMessage()
    ▼
[IN-FLIGHT / INVISIBLE]  ←── visibility timeout running
    │
    ├── Processing succeeds → DeleteMessage() → [DELETED] ✓
    │
    └── Visibility timeout expires OR consumer crashes
                │
                ▼
          [AVAILABLE again] → another consumer picks it up
                │
                │ (after MaxReceiveCount retries)
                ▼
          [MOVED TO DLQ]
```

---

## Deletion Policy Decision Tree

```
Is this a critical message where loss is unacceptable?
    │
    ├── YES → Use NEVER + manual ack.acknowledge()
    │          on success; don't ack on business failure
    │
    └── NO → Use ON_SUCCESS (default)
               Exception thrown? → message stays in queue
               No exception? → message auto-deleted
```

---

## Queue Type Decision Guide

```
Do you need strict message ordering?
    │
    ├── YES → Do you need exactly-once processing?
    │             │
    │             ├── YES → FIFO Queue
    │             └── NO  → FIFO Queue (ordering implies no duplicates)
    │
    └── NO → Do you tolerate duplicate messages?
                  │
                  ├── YES → Standard Queue (maximum throughput)
                  └── NO  → Implement idempotency + Standard Queue
                             (FIFO only if ordering also needed)
```

---

## Idempotency Strategy Cheat Sheet

| Scenario | Strategy |
|---|---|
| Business entity has natural unique key | Business logic check (check entity state in DB) |
| High-throughput, short TTL acceptable | Redis cache with TTL (24 h) |
| Audit trail required | DB tracking table |
| High-concurrency consumers | Distributed lock (Redis) + DB inside transaction |
| FIFO queue | SQS-level deduplication (explicit ID or content-based) |

---

## Common Pitfalls & Fixes

| Pitfall | Fix |
|---|---|
| Deleting message before processing is complete | Always delete **after** processing succeeds |
| Visibility timeout too short | Set timeout > max processing time; extend dynamically |
| Not checking batch partial failures | Always inspect `result.getFailed()` |
| Short polling in production | Always use long polling (`WaitTimeSeconds = 20`) |
| No DLQ configured | Every production queue needs a DLQ |
| No idempotency in consumer | Implement idempotency — standard queues CAN deliver duplicates |
| Storing secrets in message body | Use references (ARNs/IDs), not secrets, in messages |
| Creating a new SQS client per request | Reuse client instances — expensive to create |
| FIFO queue named without `.fifo` suffix | FIFO queue names MUST end in `.fifo` |

---

## Best Practices Checklist

**Design**
- [ ] Standard for high throughput; FIFO only when ordering is critical
- [ ] Every production queue has a DLQ configured
- [ ] Visibility timeout set > maximum processing time
- [ ] Long polling enabled (`WaitTimeSeconds = 20`)
- [ ] Batch operations used for send, receive, and delete

**Consumer**
- [ ] Message always deleted after successful processing
- [ ] Consumer is idempotent
- [ ] Exception handling distinguishes transient vs permanent errors
- [ ] DLQ alerted via CloudWatch

**Security**
- [ ] IAM roles used (not access keys hardcoded)
- [ ] Least-privilege: producers have only `SendMessage`, consumers have only `ReceiveMessage` + `DeleteMessage`
- [ ] SSE enabled (SSE-SQS minimum, SSE-KMS for regulated data)
- [ ] VPC endpoint for private environments

**Monitoring**
- [ ] Alarm on `ApproximateAgeOfOldestMessage` > threshold
- [ ] Alarm on DLQ depth > 0
- [ ] Alarm on queue depth for auto-scaling trigger
- [ ] Consumer auto-scales based on queue metrics

**IaC**
- [ ] Queues defined in CloudFormation / CDK / Terraform
- [ ] Queue names tagged with environment and owning team
- [ ] DLQ redrive tested in staging
