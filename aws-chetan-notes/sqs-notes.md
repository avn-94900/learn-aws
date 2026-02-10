# Amazon SQS and Messaging

## Decoupling Microservices

**Why Decouple?**

- Avoid blocking between services
- Isolate failures to prevent cascading effects
- Scale components independently
- Make systems resilient to traffic spikes
- Improve system maintainability

**Decoupling Patterns**

| Pattern | Service | Purpose |
|---------|---------|---------|
| Queue | SQS | Asynchronous processing, point-to-point |
| Pub/Sub | SNS | Notifications, fan-out to multiple subscribers |
| Event Sourcing/Streams | Kinesis, Kafka | High-volume streams, replayability, event history |
| Circuit Breakers | Application-level | Prevent overload, graceful degradation |
| Backpressure | Application-level | Control flow, prevent overwhelming consumers |

**Tightly Coupled Architecture (Without Decoupling)**

- Services wait for responses and block execution
- Users may experience freezing or timeouts
- Single service failure cascades to others
- Hard to scale individual components
- Synchronous waits slow overall response time
- Risk of crashes under heavy load

**Loosely Coupled Architecture (With Decoupling)**

- Services do not block waiting for responses
- Lost connections handled gracefully
- Failures isolated to individual components
- Scales automatically based on demand
- Better fault tolerance and resilience

---

## Amazon SQS

**What is SQS?**

SQS (Simple Queue Service) is a fully managed message queuing service that decouples microservices and enables asynchronous communication.

**Benefits of SQS**

- Decouples services to prevent blocking
- Handles high traffic smoothly
- Reliable, scalable, and fault-tolerant
- Easier to scale compared to tightly coupled systems
- No servers to manage
- Unlimited throughput (Standard queue)
- Message retention up to 14 days

---

## SQS Queue Types

**Queue Type Comparison**

| Feature | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| Delivery | At-least-once | Exactly-once |
| Ordering | Best-effort | Strict ordering |
| Throughput | Unlimited | 3000 messages/sec (or 300 with batching) |
| Deduplication | No | Yes (5-minute window) |
| Use Case | High throughput, order not critical | Strict ordering and deduplication required |

**When to Use Standard Queue**

- High throughput is priority
- Order does not matter
- Can handle duplicate messages in application logic
- Most cost-effective option

**When to Use FIFO Queue**

- Strict message ordering required
- Exactly-once processing needed
- No duplicate messages allowed
- Examples: financial transactions, command execution sequences

---

## SQS Polling

**Short Polling**

- Immediately returns available messages
- May return empty even if messages exist on other servers
- Less efficient for sparse queues
- More API calls and higher cost

**Long Polling**

- Holds connection open up to 20 seconds until messages arrive or timeout
- Reduces empty responses
- Lowers API calls and cost
- Improves throughput and latency
- Recommended for most use cases

**Recommendation**

Use long polling for better efficiency and throughput, especially when message arrival is bursty or unpredictable.

**Configuration**

Set ReceiveMessageWaitTimeSeconds to 20 for long polling.

---

## High Throughput SQS Patterns

**Queue Design**

- Use Standard queues for maximum throughput
- Use FIFO queues only when strict ordering and deduplication are required
- Consider queue depth and consumer capacity when designing

**Consumer Scaling**

- Autoscale consumers based on CloudWatch metrics
- Key metrics: ApproximateNumberOfMessagesVisible and ApproximateAgeOfOldestMessage
- Use ECS, EKS, or EC2 Auto Scaling Groups for consumer instances
- For Lambda consumers: configure batch size, maximum concurrency, and provisioned concurrency

**Processing Optimization**

- Read and process messages in batches (up to 10 per request)
- Enable long polling (20 seconds recommended)
- Tune visibility timeout to be greater than processing time
- Use heartbeats or extend visibility for long-running tasks
- Delete messages only after successful processing

**Parallelism and Sharding**

- Increase parallel consumers to improve throughput
- For FIFO queues, use MessageGroupId to enable parallelism across groups
- Shard into multiple queues for independent parallel processing
- Each MessageGroupId is processed by one consumer at a time

---

## Error Handling

**Dead Letter Queues (DLQ)**

- Configure DLQ for poison messages that fail repeatedly
- Set maximum receive count before moving to DLQ
- Monitor DLQ and set up alarms
- Analyze DLQ messages to identify recurring issues

**Retry Strategy**

- Implement exponential backoff on retry
- Avoid tight retry loops that overload consumers
- Set appropriate visibility timeout for retry attempts
- Consider using separate queues for different retry intervals

**Visibility Timeout**

- Time during which a message is invisible to other consumers after being received
- Must be greater than processing time
- Can be extended using ChangeMessageVisibility API
- Default is 30 seconds, maximum is 12 hours

---

## Large Messages

**S3 Payload Offload**

Use SQS Extended Client for messages over 256 KB:

1. Store message payload in S3
2. Send S3 object reference in SQS message
3. Consumer retrieves payload from S3 using reference
4. Maximum payload size: 2 GB

**When to Use**

- Messages contain large documents, images, or files
- Message size exceeds 256 KB SQS limit
- Want to separate message metadata from payload

---

## Monitoring and Metrics

**Key Metrics to Monitor**

| Metric | Description | Alert Threshold |
|--------|-------------|----------------|
| ApproximateNumberOfMessagesVisible | Messages available for retrieval | High value indicates backlog |
| ApproximateNumberOfMessagesNotVisible | Messages in-flight being processed | Monitor for stuck messages |
| ApproximateAgeOfOldestMessage | Age of oldest message in queue | Trigger scaling if too high |
| NumberOfMessagesDeleted | Messages successfully processed | Low rate indicates issues |
| NumberOfMessagesReceived | Messages added to queue | Track incoming rate |
| NumberOfMessagesSent | Messages published to queue | Compare with deleted for throughput |

**CloudWatch Alarm Examples**

- ApproximateAgeOfOldestMessage greater than threshold: trigger scaling action
- MessagesVisible greater than threshold: scale out consumers
- Monitor ratio of MessagesSent to NumberOfMessagesDeleted for anomalies
- DLQ message count greater than zero: alert for failed messages

---

## Idempotency

**Definition**

An operation that can be retried without changing the result beyond the first application.

**Why Needed**

- Prevents duplicate side effects (double charges, duplicate database rows)
- Handles network errors and retries safely
- Works with at-least-once delivery semantics (SQS Standard queue, HTTP retries)
- Essential for reliable distributed systems

**Implementation Patterns**

**Idempotency Key**

- Client sends unique key (UUID) with request
- Server stores key and result
- Repeated calls with same key return same result without re-execution

**Database Upsert**

- Use unique constraint on business key
- Insert or update based on unique identifier
- Database enforces deduplication

**SQS FIFO Deduplication**

- Use message deduplication ID
- Or enable content-based deduplication
- SQS prevents duplicates within 5-minute window

**Stateless Idempotent Endpoints**

- GET requests are naturally idempotent
- PUT requests replace entire resource
- Design POST operations to be idempotent using unique identifiers

---

## Messaging Services Comparison

| Feature | SQS | SNS | EventBridge | Kafka |
|---------|-----|-----|-------------|-------|
| Type | Queue (point-to-point) | Pub/Sub (fan-out) | Event bus (routing) | Distributed streaming |
| Ordering | FIFO optional | No guarantee | Depends on rules | Strong ordering per partition |
| Use Case | Decouple services | Notifications | Event-driven apps | High-volume stream processing |
| Throughput | Very high (unlimited for Standard) | Very high | High | Very high |
| Retention | Up to 14 days | No retention (push only) | Archive to S3 | Configurable, long retention |
| Delivery | Pull (consumer polls) | Push (to subscribers) | Push (to targets) | Pull (consumer reads) |
| Message Size | 256 KB (2 GB with S3) | 256 KB | 256 KB | 1 MB default |

**When to Use Each**

- SQS: Reliable point-to-point asynchronous decoupling, background job processing
- SNS: Push notifications to many subscribers, fan-out pattern
- EventBridge: Event-driven architectures, routing rules, SaaS integration
- Kafka: High-volume streaming, replay capability, complex event processing

---

## SQS Best Practices Summary

**Design**

- Choose Standard for high throughput, FIFO for strict ordering
- Enable long polling to reduce API calls
- Use batching for efficiency
- Configure appropriate visibility timeout

**Scaling**

- Autoscale consumers based on queue metrics
- Monitor ApproximateAgeOfOldestMessage
- Set up CloudWatch alarms for queue depth

**Reliability**

- Implement idempotency in message processing
- Configure Dead Letter Queues
- Use exponential backoff for retries
- Delete messages only after successful processing

**Performance**

- Process messages in parallel
- Use multiple consumers
- Optimize batch size based on workload
- Consider message size and use S3 for large payloads

**Monitoring**

- Track key metrics in CloudWatch
- Set up alarms for anomalies
- Monitor DLQ for failed messages
- Analyze message processing time and throughput