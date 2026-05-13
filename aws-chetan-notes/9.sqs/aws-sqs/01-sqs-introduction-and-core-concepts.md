# 01 — SQS Introduction & Core Concepts

---

## What is Amazon SQS?

Amazon Simple Queue Service (SQS) is a **fully managed message queuing service** that enables you to decouple and scale microservices, distributed systems, and serverless applications.

- **Fully managed** — no infrastructure or server management required
- **Pull-based** — consumers poll the queue for messages (not push-based)
- **Decouples components** — producers and consumers operate independently
- **Reliable delivery** — guarantees at-least-once delivery (exactly-once for FIFO)
- **Pay-per-request** — no upfront costs, billed per API call

---

## Why Decouple Microservices?

| Problem (Tightly Coupled) | Solution (Decoupled with SQS) |
|---|---|
| Services block waiting for responses | Services process asynchronously |
| One failure cascades to others | Failures isolated to individual components |
| Hard to scale individual parts | Each component scales independently |
| Slow under heavy load | Queue absorbs traffic spikes |

---

## Queue Types

### Standard Queue

- **Throughput**: Nearly unlimited transactions per second
- **Ordering**: Best-effort (messages may arrive out of order)
- **Delivery**: At-least-once (duplicates are possible)
- **Naming**: Any valid name
- **Use case**: High-throughput workloads where duplicates are acceptable (e.g., log aggregation, image processing)

### FIFO Queue (First-In-First-Out)

- **Throughput**: Up to 300 TPS; up to 3,000 TPS with batching; up to 70,000 TPS with high-throughput mode (2023+)
- **Ordering**: Strict first-in-first-out per message group
- **Delivery**: Exactly-once (no duplicates)
- **Naming**: Queue name **must** end with `.fifo` suffix
- **Use case**: Order processing, financial transactions, inventory updates — anything requiring strict ordering and no duplicates

### Comparison Table

| Feature | Standard Queue | FIFO Queue |
|---|---|---|
| Throughput | Nearly unlimited | 300–70,000 TPS |
| Ordering | Best-effort | Strict FIFO |
| Delivery | At-least-once | Exactly-once |
| Duplicates | Possible | Not possible |
| Naming | Any name | Must end in `.fifo` |
| Cost | Lower | Slightly higher |

---

## Core Concepts Glossary

| Term | Definition |
|---|---|
| **Queue** | Named buffer that stores messages between producer and consumer |
| **Message** | Data payload sent to a queue (up to 256 KB) |
| **Producer** | Application that sends messages to a queue |
| **Consumer** | Application that polls and processes messages from a queue |
| **Visibility Timeout** | Period during which a received message is hidden from other consumers |
| **Dead Letter Queue (DLQ)** | Queue that receives messages which failed processing repeatedly |
| **Long Polling** | Consumer waits up to 20 seconds for messages, reducing empty responses |
| **Short Polling** | Consumer returns immediately, even if the queue is empty |
| **Receipt Handle** | Unique token required to delete or change visibility of a received message |
| **Message Group ID** | FIFO-only: groups related messages for ordered, sequential processing |
| **Deduplication ID** | FIFO-only: prevents duplicate messages within a 5-minute window |

---

## Message Lifecycle States

```
Sent → Available → Received (Invisible) → Processing → Deleted
                                                    ↓ (on failure)
                                             Returned to Queue
                                                    ↓ (after max retries)
                                             Dead Letter Queue (DLQ)
```

1. **Sent** — Producer successfully adds message to the queue
2. **Available** — Message is visible and waiting to be consumed
3. **Received** — Consumer retrieves message; it becomes invisible to others
4. **Processing** — Consumer is actively processing the message (still invisible)
5. **Deleted** — Consumer deletes message after successful processing
6. **Returned to Queue** — Message reappears if not deleted within visibility timeout
7. **Dead Letter Queue** — Message moved to DLQ after exceeding max receive count

---

## SQS Message Anatomy

Each SQS message contains:

| Component | Description | Example |
|---|---|---|
| **Message Body** | The actual data payload (up to 256 KB) | `{"orderId": 123, "amount": 5000}` |
| **Message ID** | Unique ID assigned by SQS when sent | `e2f76847-3d0e-42c9-...` |
| **Receipt Handle** | Token used to delete the message | `AQEBzZzO...shF+A==` |
| **MD5 of Body** | Hash for integrity verification | `7b270e59b47ff90a...` |
| **Message Attributes** | Optional metadata key-value pairs | `ContentType: application/json` |
| **Sent Timestamp** | Epoch time in milliseconds when sent | `1721011200000` |
| **First Received Timestamp** | Approximate time of first delivery | `1721011220000` |

### Sample Message JSON

```json
{
  "MessageId": "e2f76847-3d0e-42c9-a2dc-5b98e8b6d403",
  "ReceiptHandle": "AQEBzZzOaGf4L2hC5iCwUkN7...shF+A==",
  "MD5OfBody": "7b270e59b47ff90a553787216d55d91d",
  "Body": "{\"orderId\": 12345, \"amount\": 5000, \"currency\": \"INR\"}",
  "Attributes": {
    "SentTimestamp": "1721011200000",
    "ApproximateFirstReceiveTimestamp": "1721011220000"
  },
  "MessageAttributes": {
    "ContentType": {
      "DataType": "String",
      "StringValue": "application/json"
    },
    "SourceSystem": {
      "DataType": "String",
      "StringValue": "OrderService"
    }
  }
}
```

---

## Polling Mechanisms

### Short Polling

- Returns immediately, even if the queue is empty
- Queries only a subset of SQS servers
- Each empty response is still billed
- Use when you need an immediate response

### Long Polling (Recommended)

- Waits up to 20 seconds for messages to arrive
- Queries all SQS servers — fewer missed messages
- Fewer empty responses → lower cost
- Configure with `WaitTimeSeconds` parameter (1–20 seconds)

> **Best Practice**: Always prefer long polling. Set `WaitTimeSeconds` to `20` for maximum efficiency.

---

## Key Queue Settings & Limits

| Setting | Range | Default | Notes |
|---|---|---|---|
| **Message Size** | 1 B – 256 KB | — | Use S3 + Extended Client for larger payloads |
| **Message Retention** | 60 s – 14 days | 4 days | How long SQS stores undeleted messages |
| **Visibility Timeout** | 0 s – 12 hours | 30 seconds | Should be greater than processing time |
| **Delay Queue** | 0 – 15 minutes | 0 | Postpone delivery of all new messages |
| **Per-Message Delay** | 0 – 15 minutes | 0 | Individual message delivery delay |
| **Long Polling Wait** | 1 – 20 seconds | 0 (short polling) | Set to 20 for best efficiency |
| **Batch Size** | 1 – 10 messages | 1 | Per send/receive/delete API call |
| **In-flight Limit** | Standard: 120,000 / FIFO: 20,000 | — | Messages received but not yet deleted |

---

## SQS vs Other AWS Messaging Services

| Feature | SQS | SNS | EventBridge | Kinesis |
|---|---|---|---|---|
| **Type** | Queue (point-to-point) | Pub/Sub (fan-out) | Event bus (routing) | Streaming |
| **Ordering** | FIFO optional | No guarantee | Depends on rules | Per shard |
| **Retention** | Up to 14 days | No retention (push only) | Archive to S3 | 7 days default |
| **Delivery** | Pull (consumer polls) | Push (to subscribers) | Push (to targets) | Pull |
| **Message Size** | 256 KB (2 GB with S3) | 256 KB | 256 KB | 1 MB |
| **Use Case** | Decouple services, background jobs | Notifications, fan-out | Event-driven routing | High-volume streaming |

> **When to use what:**
> - **SQS** — reliable async decoupling, background processing
> - **SNS** — push notifications to many subscribers at once
> - **EventBridge** — routing events between AWS services or SaaS
> - **Kinesis/Kafka** — high-volume stream processing with replay capability
