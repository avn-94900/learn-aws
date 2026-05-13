# 08 — Performance, Scaling & Monitoring

---

## Throughput Limits

| Queue Type | Mode | Throughput |
|---|---|---|
| Standard | — | Nearly unlimited TPS |
| FIFO | Classic | 300 TPS (3,000 with batching) |
| FIFO | High-Throughput (2023+) | 70,000 TPS per API action |

> **Note**: High-throughput FIFO mode must be explicitly enabled on the queue. It trades strict per-group ordering for massively higher throughput.

---

## Batch Operations for Efficiency

Always use batch APIs when processing more than one message — they reduce API calls and cost.

| Operation | API | Limit |
|---|---|---|
| Send | `SendMessageBatch` | Up to 10 messages |
| Receive | `ReceiveMessage` with `MaxNumberOfMessages` | Up to 10 messages |
| Delete | `DeleteMessageBatch` | Up to 10 messages |
| Change Visibility | `ChangeMessageVisibilityBatch` | Up to 10 messages |

```java
// Receive up to 10 messages with long polling
ReceiveMessageRequest request = new ReceiveMessageRequest()
    .withQueueUrl(queueUrl)
    .withMaxNumberOfMessages(10)
    .withWaitTimeSeconds(20);

ReceiveMessageResult result = amazonSQS.receiveMessage(request);
```

---

## Scaling Consumers

### Scale Based on Queue Metrics

The key CloudWatch metrics to watch:

| Metric | What It Tells You |
|---|---|
| `ApproximateNumberOfMessagesVisible` | Messages waiting to be processed — primary scaling signal |
| `ApproximateAgeOfOldestMessage` | How long the oldest message has been waiting — latency indicator |
| `ApproximateNumberOfMessagesNotVisible` | Messages currently being processed |
| `NumberOfMessagesSent` | Producer throughput |
| `NumberOfMessagesReceived` | Consumer throughput |
| `NumberOfMessagesDeleted` | Successfully processed messages |

### Auto-Scaling Strategy (ECS / EC2)

Scale consumer instances based on `ApproximateNumberOfMessagesVisible`:

```
Target: 1 consumer per N messages in queue (e.g., 1 per 1000 messages)

Scale out: queue depth rising → add consumer instances
Scale in:  queue depth falling → remove consumer instances
```

Example CloudWatch alarm to trigger scaling:

```
Alarm: ApproximateNumberOfMessagesVisible > 5000
Action: Increase ECS service desired count by 2
```

### Lambda Consumers — Key Settings

| Setting | Description |
|---|---|
| **Batch Size** | Number of messages Lambda receives per invocation (1–10,000 for standard, 1–10 for FIFO) |
| **Maximum Concurrency** | Limit parallel Lambda executions to protect downstream services |
| **Bisect on Error** | Split failing batch in half to isolate problematic messages |
| **Maximum Retry Attempts** | How many times Lambda retries a failed batch |

---

## FIFO Queue Parallelism

FIFO queues process one `MessageGroupId` at a time per consumer. To achieve parallelism:

- Use **multiple `MessageGroupId` values** — different groups are processed in parallel
- One consumer handles all messages in a group; different groups can run concurrently

```
Group A: [msg1] → [msg2] → [msg3]   (Consumer 1 — sequential)
Group B: [msg4] → [msg5]            (Consumer 2 — parallel to Consumer 1)
Group C: [msg6]                     (Consumer 3 — parallel)
```

```java
// Assign group IDs based on business partitioning
String messageGroupId = "customer-" + order.getCustomerId();

SendMessageRequest request = new SendMessageRequest()
    .withQueueUrl(queueUrl)
    .withMessageBody(orderJson)
    .withMessageGroupId(messageGroupId);
```

---

## Cost Optimization

| Strategy | How |
|---|---|
| **Long polling** | Set `WaitTimeSeconds = 20` — eliminates most empty responses |
| **Batch operations** | Use batch send/receive/delete — billed per API call, not per message |
| **Right-size retention** | Don't keep messages longer than needed (default 4 days is usually fine) |
| **Standard over FIFO** | Use FIFO only when strict ordering is required — same price per API call but FIFO has lower throughput |
| **S3 for large payloads** | Messages > 256 KB stored in S3; only a reference pointer in SQS — cheaper than splitting |
| **Monitor idle queues** | Delete or archive queues with no traffic |

---

## CloudWatch Alarms — Recommended Setup

### Queue Depth Alarm

```
Metric: ApproximateNumberOfMessagesVisible
Threshold: > 10,000
Period: 5 minutes
Action: Scale out consumers OR send alert to ops
```

### Message Age Alarm (Latency Alert)

```
Metric: ApproximateAgeOfOldestMessage
Threshold: > 300 seconds (5 minutes)
Period: 5 minutes
Action: Send alert — consumers may be falling behind
```

### DLQ Depth Alarm

```
Metric: ApproximateNumberOfMessagesVisible on DLQ
Threshold: > 0
Period: 5 minutes
Action: Send alert to dev team — messages are failing
```

---

## Performance Best Practices Summary

| Area | Best Practice |
|---|---|
| **Polling** | Always use long polling (`WaitTimeSeconds = 20`) |
| **Batching** | Use batch receive (10 messages) and batch delete |
| **Visibility Timeout** | Set greater than max processing time; extend with `ChangeMessageVisibility` for long tasks |
| **FIFO Parallelism** | Shard by `MessageGroupId` to enable parallel processing across groups |
| **Consumer Scaling** | Auto-scale based on `ApproximateNumberOfMessagesVisible` |
| **Large Payloads** | Use SQS Extended Client (S3) for messages > 256 KB |
| **Monitoring** | Set alarms on queue depth, message age, and DLQ depth |
| **Connection Pooling** | Reuse SQS client instances — don't create a new client per request |

---

## SQS Extended Client (Large Payloads > 256 KB)

```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>amazon-sqs-java-extended-client-lib</artifactId>
    <version>2.0.1</version>
</dependency>
```

```java
AmazonS3 s3Client = AmazonS3ClientBuilder.defaultClient();
ExtendedClientConfiguration config = new ExtendedClientConfiguration()
    .withPayloadSupportEnabled(s3Client, "my-sqs-payloads-bucket")
    .withAlwaysThroughS3(false)       // only use S3 when message > 256 KB
    .withPayloadSizeThreshold(256);   // threshold in KB

AmazonSQS sqsExtended = new AmazonSQSExtendedClient(
    AmazonSQSClientBuilder.defaultClient(), config
);

// Use exactly like normal SQS — the library handles S3 storage automatically
sqsExtended.sendMessage(queueUrl, largeMessageBody);
```
