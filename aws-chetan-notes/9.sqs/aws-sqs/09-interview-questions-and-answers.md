# 09 — Interview Questions & Answers

Questions are grouped by topic and ordered from beginner to advanced.

---

## 1. SQS Basics & Core Architecture

**Q: What is Amazon SQS and what are its key features?**

A: SQS is a fully managed, pull-based message queuing service that decouples producers and consumers in distributed architectures. Key features include: at-least-once delivery (exactly-once for FIFO), virtually unlimited throughput for standard queues, message retention up to 14 days, long polling, dead letter queues, batch operations, and server-side encryption. It requires no infrastructure management and scales automatically.

---

**Q: What are the differences between Standard and FIFO queues?**

| | Standard | FIFO |
|---|---|---|
| Ordering | Best-effort | Strict FIFO |
| Delivery | At-least-once (duplicates possible) | Exactly-once |
| Throughput | Unlimited | 300–70,000 TPS |
| Naming | Any name | Must end in `.fifo` |
| Use case | High-throughput, duplicate-tolerant | Ordered, no-duplicate workflows |

---

**Q: What is the difference between short polling and long polling?**

- **Short polling** returns immediately even if the queue is empty, querying only a subset of SQS servers. This results in more API calls and higher cost.
- **Long polling** waits up to 20 seconds for a message to arrive and queries all SQS servers. It reduces empty responses, lowers cost, and is recommended for almost all use cases. Set `WaitTimeSeconds` to `20`.

---

**Q: What is Visibility Timeout and why is it important?**

Visibility Timeout is the period during which a message is hidden from other consumers after one consumer has received it. It prevents two consumers from processing the same message simultaneously. If the consumer successfully processes the message, it deletes it. If the consumer crashes or the timeout expires, the message becomes visible again for another consumer to pick up.

> **Rule**: Always set visibility timeout greater than your maximum expected processing time.

---

**Q: What is the maximum message size in SQS and how do you handle larger payloads?**

The hard limit is 256 KB. For larger payloads, use the **SQS Extended Client Library** — the actual data is stored in S3, and only a reference pointer is stored in the SQS message. The library handles this transparently.

---

**Q: What is the default and maximum message retention period?**

- Default: 4 days
- Maximum: 14 days
- Minimum: 60 seconds

---

## 2. Producers & Publishing

**Q: How do you send messages to an SQS queue programmatically?**

In Spring Boot, use `QueueMessagingTemplate.convertAndSend()`. For lower-level control, use the `AmazonSQS.sendMessage()` SDK method. AWS SDK v2 uses `SqsClient.sendMessage()` with a `SendMessageRequest` builder.

---

**Q: What is batch sending and what are its benefits?**

Batch sending allows sending up to 10 messages in a single `SendMessageBatch` API call. Benefits include reduced API calls (up to 10× fewer), lower cost (billed per call, not per message), better throughput, and reduced latency. Always check the response for partial failures — some messages may succeed while others fail.

---

**Q: What is the difference between `MessageGroupId` and `MessageDeduplicationId`?**

- **`MessageGroupId`**: Groups related messages together for ordered, sequential processing. All messages in a group are processed by one consumer at a time, in order. Different groups can be processed in parallel. Required for FIFO queues.
- **`MessageDeduplicationId`**: Prevents duplicate messages from being delivered within a 5-minute deduplication window. SQS will reject messages with the same ID sent within this window. Required for FIFO queues (or enable content-based deduplication).

---

## 3. Consumers & Processing

**Q: What happens when a message is received but not deleted?**

The message becomes visible again to other consumers once the visibility timeout expires. This is the mechanism SQS uses to enable retries — if a consumer crashes mid-processing, another consumer will eventually pick up the message.

---

**Q: What is the difference between `ReceiveMessageWaitTimeSeconds` and `VisibilityTimeout`?**

- **`ReceiveMessageWaitTimeSeconds`** (0–20 s): Controls long polling — how long the `ReceiveMessage` call waits for messages if the queue is empty. Set to 20 for maximum efficiency.
- **`VisibilityTimeout`** (0 s – 12 h): Controls how long a received message stays hidden from other consumers while being processed. Set to be greater than your processing time.

---

**Q: What happens if a consumer crashes while processing a message?**

The message becomes visible again after the visibility timeout expires and can be picked up by another consumer. This is why idempotent processing is essential — the same message may be processed more than once.

---

## 4. Dead Letter Queues & Error Handling

**Q: What are Dead Letter Queues (DLQ) and when should you use them?**

A DLQ is a special queue that receives messages that failed processing too many times (exceeded `MaxReceiveCount`). Use DLQs to isolate "poison" messages that would otherwise loop indefinitely, enabling debugging and analysis without blocking the main queue. Every production queue should have a DLQ configured.

---

**Q: What is the maximum number of receives before a message goes to DLQ?**

`MaxReceiveCount` is configurable from 1 to 1,000. A typical value is 3–5.

---

**Q: How do you recover and reprocess messages from a DLQ?**

Since 2022, AWS supports DLQ redrive to source via the console or `StartMessageMoveTask` API — messages are moved back to the original queue for reprocessing. For custom logic, you can read from the DLQ and re-publish to the main queue, optionally transforming or filtering messages first.

---

**Q: How do you implement exponential backoff?**

`WaitTime = base × 2^n + jitter`. AWS SDK v2 applies exponential backoff automatically for throttling errors. For application-level retry, use Spring Retry's `@Retryable` with `@Backoff(delay = 1000, multiplier = 2)`, or configure `RetryPolicy` on the `SqsClient` directly.

---

## 5. Message Delivery Guarantees & Idempotency

**Q: What delivery guarantees does SQS provide?**

- **Standard queues**: At-least-once — a message is guaranteed to be delivered at least once, but may be delivered more than once.
- **FIFO queues**: Exactly-once — no duplicates, strict ordering within each message group, within a 5-minute deduplication window.

---

**Q: How do you implement idempotent message processing?**

Options from simplest to most robust:
1. Check business entity state (e.g., if order already exists, skip)
2. Track `MessageId` or business identifier in a DB or Redis cache with TTL
3. Use database upsert with a unique constraint on the business key
4. For high-concurrency: distributed lock (Redis) + database check inside a transaction

---

**Q: How do you handle duplicate messages in Standard queues?**

Implement idempotency in the consumer. Common strategies: check if the business entity already exists in the DB (business logic idempotency), track `MessageId` in Redis with a 24-hour TTL, or use a database `INSERT ... ON CONFLICT DO NOTHING` pattern.

---

## 6. Integration & Architecture

**Q: What is the difference between SQS and SNS?**

- **SQS** is a queue for **point-to-point** messaging — one producer, one consumer. Messages are stored until consumed, and each message is processed by one consumer.
- **SNS** is a **pub/sub** service — one message is fan-out-delivered to all subscribers (can be SQS queues, Lambda functions, HTTP endpoints, email, etc.). No storage; messages are pushed immediately.

> **Common pattern**: SNS → multiple SQS queues (fan-out). One SNS publish reaches multiple independent consumer groups.

---

**Q: How do you integrate SQS with AWS Lambda?**

Configure SQS as a Lambda event source. Lambda polls the queue, receives a batch of messages, and invokes your function. Lambda automatically handles message deletion on success. Configure `batchSize`, `maximumConcurrency`, and a DLQ for failed batches.

---

**Q: How do you implement a fan-out pattern?**

Publish to an SNS topic → SNS delivers to multiple subscribed SQS queues → each queue has its own consumer group. This decouples the publisher from the number and identity of consumers.

---

## 7. Performance & Scaling

**Q: How do you optimize SQS costs?**

- Use long polling (`WaitTimeSeconds = 20`) to eliminate empty responses
- Use batch operations (send, receive, delete) — billed per API call
- Set appropriate message retention (don't over-retain)
- Use Standard queues where ordering isn't critical (lower overhead)
- Monitor and delete unused queues

---

**Q: What are the throughput limits for Standard vs FIFO queues?**

- Standard: Nearly unlimited TPS
- FIFO Classic: 300 TPS (3,000 with batching)
- FIFO High-Throughput (2023+): 70,000 TPS per API action

---

**Q: How do you scale SQS consumers?**

Auto-scale based on `ApproximateNumberOfMessagesVisible` (queue depth) and `ApproximateAgeOfOldestMessage` (latency). Use CloudWatch alarms to trigger ECS/EC2 Auto Scaling Group actions. For FIFO, scale by adding more `MessageGroupId` values to enable parallel processing across groups.

---

## 8. Security

**Q: How do you implement security for SQS queues using IAM policies?**

Use identity-based IAM policies on each role (least privilege: producers get `sqs:SendMessage`, consumers get `sqs:ReceiveMessage` + `sqs:DeleteMessage`). Use queue policies (resource-based) for cross-account access. Enable SSE for encryption at rest. Use VPC endpoints for private network access.

---

**Q: What is the difference between IAM policies and SQS resource-based policies?**

- **IAM policies** are attached to identities (users/roles) and grant permissions from the identity's perspective
- **Queue policies** are attached to the SQS queue and grant permissions from the resource's perspective. They are required for cross-account access (you can't use just an IAM policy to grant another account access to your queue — the queue must also grant it)

---

**Q: What are the encryption options available in SQS?**

1. **SSE-SQS**: Default, SQS-managed keys, zero configuration, automatic rotation
2. **SSE-KMS (AWS Managed)**: KMS audit trail, no key management overhead
3. **SSE-KMS (Customer Managed)**: Full control, fine-grained access policies, required for some compliance frameworks
4. **In-transit**: HTTPS for all API calls (default, cannot be disabled)

---

## 9. Advanced & Design Questions

**Q: Design a microservices order processing system using SQS.**

```
Order Service (producer)
    │
    └── orders-queue (Standard, high throughput)
            │
            ├── Payment Service (consumer) → payment-events-queue
            ├── Inventory Service (consumer) → inventory-events-queue
            └── Notification Service (consumer) → notification-queue
                        │
                        └── orders-dlq (DLQ for failed messages)

Monitoring:
- CloudWatch alarm on orders-queue depth > 10,000
- CloudWatch alarm on orders-dlq depth > 0
- Auto-scaling on ECS payment-service based on queue depth
```

---

**Q: What are common SQS anti-patterns?**

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Tight polling loop | High cost from empty receives | Use long polling |
| Deleting before processing | Message lost on crash | Delete only after successful processing |
| Ignoring partial batch failures | Silent data loss | Always check `failed` in `SendMessageBatchResult` |
| Hardcoded visibility timeout | Timeout shorter than processing time → duplicates | Tune timeout; extend dynamically |
| No DLQ | Poison messages loop forever | Always configure DLQ |
| Not idempotent consumer | Duplicate side effects | Implement idempotency in every consumer |
| Giant message bodies | Hits 256 KB limit, expensive | Use S3 + Extended Client for large payloads |
