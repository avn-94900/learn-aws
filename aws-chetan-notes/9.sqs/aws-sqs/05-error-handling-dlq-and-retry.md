# 05 — Error Handling, DLQ & Retry

---

## Dead Letter Queue (DLQ)

A Dead Letter Queue receives messages that **failed processing too many times**. It isolates problematic ("poison") messages so they don't block the main queue.

### How It Works

```
Main Queue
    │
    ├── Consumer receives message
    ├── Processing fails
    ├── Message becomes visible again (visibility timeout expires)
    ├── Consumer receives message again (retry)
    │
    ↓   (after MaxReceiveCount failures)
    
Dead Letter Queue ← message moved here automatically by SQS
```

### Configuration Rules

- DLQ must be the **same type** as the source queue (Standard → Standard, FIFO → FIFO)
- Set `maxReceiveCount` between 1–1000 (how many times a message can be received before moving to DLQ)
- Messages in DLQ retain their original attributes and body
- DLQ messages can be retained for up to 14 days

### Configuring DLQ (AWS Console / CloudFormation)

```json
{
  "RedrivePolicy": {
    "deadLetterTargetArn": "arn:aws:sqs:us-east-1:123456789012:my-queue-dlq",
    "maxReceiveCount": "5"
  }
}
```

### DLQ Monitoring

Always set a CloudWatch alarm on your DLQ:

```
Alarm: ApproximateNumberOfMessagesVisible > 0 on my-queue-dlq
Action: Send SNS notification to ops team
```

---

## Retry Strategy

### Built-In SQS Retry

SQS automatically retries messages by making them visible again after the visibility timeout expires. No code needed — just don't delete the message on failure.

```java
@SqsListener("orders-queue")
public void processOrder(String message) {
    try {
        orderService.process(message);
        // Message auto-deleted on success (ON_SUCCESS policy)
    } catch (TransientException e) {
        log.error("Transient failure, will retry: {}", e.getMessage());
        throw e;  // re-throw → message stays in queue → retried after visibility timeout
    } catch (PermanentException e) {
        log.error("Permanent failure, acknowledging to prevent DLQ spam: {}", e.getMessage());
        // Handle gracefully — store to DB, alert team, etc.
        // do NOT re-throw → message gets deleted (or you can push to a custom error store)
    }
}
```

### Application-Level Retry with `@Retryable` (Spring Retry)

Use for transient errors like network blips when **sending** messages.

```java
@Service
public class RobustSQSProducer {

    @Autowired
    private AmazonSQS amazonSQS;

    @Retryable(
        value = {AmazonSQSException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2) // exponential backoff
    )
    public void sendMessageWithRetry(String queueUrl, String message) {
        SendMessageRequest request = new SendMessageRequest()
            .withQueueUrl(queueUrl)
            .withMessageBody(message);

        amazonSQS.sendMessage(request);
    }

    @Recover
    public void recover(AmazonSQSException ex, String queueUrl, String message) {
        log.error("All retries exhausted for message: {}", message);
        // Persist to DB for manual replay or alert team
        failedMessageRepository.save(new FailedMessage(queueUrl, message, ex.getMessage()));
    }
}
```

---

## Exponential Backoff

Instead of retrying at fixed intervals, wait **progressively longer** between retries to reduce load on the service and avoid retry storms.

```
WaitTime = base × 2^n + jitter
```

| Retry Attempt | Wait Time (base = 100ms, no jitter) |
|---|---|
| 1st | 100 ms |
| 2nd | 200 ms |
| 3rd | 400 ms |
| 4th | 800 ms |
| 5th | 1,600 ms |

Most SDKs also apply a **maximum cap** (e.g., 20 seconds) to stop indefinite doubling.

### Why Use Jitter?

Without jitter, all retrying clients hit the service at the same moment. Jitter adds a small random offset to spread out the load:

```
WaitTime = base × 2^n + Random(0, base)
```

### AWS SDK v2 — Configuring Exponential Backoff

```java
RetryPolicy retryPolicy = RetryPolicy.builder()
    .numRetries(5)
    .backoffStrategy(BackoffStrategy.defaultStrategy()) // exponential backoff with jitter
    .build();

SqsClient sqsClient = SqsClient.builder()
    .region(Region.US_EAST_1)
    .overrideConfiguration(
        ClientOverrideConfiguration.builder()
            .retryPolicy(retryPolicy)
            .build()
    )
    .build();
```

> **Note**: AWS SDKs apply exponential backoff automatically for throttling (HTTP 429/503). You can customise it for application-level retries.

---

## Exception Handling Cheat Sheet for `sendMessage()`

| Exception | When Raised | Root Cause | Mitigation | Fallback |
|---|---|---|---|---|
| `SqsException` | AWS service returns an error | `QueueDoesNotExistException`, `InvalidMessageContentsException`, `AccessDeniedException`, throttling (429/503) | Validate queue name; check IAM permissions; validate message size and format; use retry with exponential backoff | Let `RetryTemplate` reattempt; log and alert on repeated failure; persist message to DB for replay |
| `SdkClientException` | Client-side issue prevents request reaching AWS | DNS failure, no internet/VPC connectivity, socket timeout, credential misconfiguration, serialization error | Check network/VPC; validate SDK configuration; set reasonable client timeouts | Retry (transient errors often recover); use circuit breaker if repeated; alert ops; persist for retry |
| `Exception` (generic) | Unexpected error during processing | `NullPointerException`, invalid input, `IllegalArgumentException`, serialization failure | Add null checks; validate input with Bean Validation; defensive coding in attribute builders | Log full stack trace; store failed message to DB; trigger alert for dev team; build admin replay API |
| **Recovery (all retries exhausted)** | All retry attempts failed | Persistent network/server errors or permanently malformed message | Review retry config; add circuit breaker to avoid overwhelming system | Store in `FailedMessageRepository`; notify team via email/Slack/alert; expose REST endpoint to replay |

---

## Enterprise Error Handling Pattern

```java
@Service
public class EnterpriseProducer {

    @Autowired
    private AmazonSQS amazonSQS;

    @Autowired
    private FailedMessageRepository failedMessageRepository;

    public void sendMessage(String queueUrl, MessageEntity messageEntity) {
        RetryTemplate retryTemplate = RetryTemplate.builder()
            .maxAttempts(3)
            .exponentialBackoff(1000, 2, 8000)  // start 1s, multiplier 2, max 8s
            .retryOn(SqsException.class)
            .build();

        try {
            retryTemplate.execute(ctx -> {
                SendMessageRequest request = buildRequest(queueUrl, messageEntity);
                return amazonSQS.sendMessage(request);
            });

        } catch (SqsException e) {
            log.error("SQS service error after retries — queue: {}, error: {}", queueUrl, e.getMessage());
            failedMessageRepository.save(messageEntity, "SQS_ERROR", e.getMessage());
            alertOpsTeam(e, queueUrl);

        } catch (SdkClientException e) {
            log.error("Client-side error (network/credentials) — queue: {}", queueUrl, e);
            failedMessageRepository.save(messageEntity, "CLIENT_ERROR", e.getMessage());

        } catch (Exception e) {
            log.error("Unexpected error sending message — queue: {}", queueUrl, e);
            failedMessageRepository.save(messageEntity, "UNKNOWN_ERROR", e.getMessage());
            throw new RuntimeException("Failed to send message", e);
        }
    }
}
```

---

## DLQ Redrive to Source Queue

Since 2022, AWS supports **redrive to source** — pushing DLQ messages back to the original queue for reprocessing, via the console or API (`StartMessageMoveTask`).

```java
// AWS SDK v2 — start a DLQ redrive
StartMessageMoveTaskRequest redriveRequest = StartMessageMoveTaskRequest.builder()
    .sourceArn("arn:aws:sqs:us-east-1:123456789012:my-queue-dlq")
    .destinationArn("arn:aws:sqs:us-east-1:123456789012:my-queue")  // optional — defaults to source
    .maxNumberOfMessagesPerSecond(10)  // throttle to avoid overwhelming consumers
    .build();

sqsClient.startMessageMoveTask(redriveRequest);
```

---

## Tips for Enterprise Error Handling

| Concern | Best Practice |
|---|---|
| **DLQ Usage** | DLQ is for **consumers** (failed processing). Producers should use app-level retry + DB persistence. |
| **Logging** | Always log queue name, retry count, and root cause |
| **Alerting** | Integrate with CloudWatch alarms, PagerDuty, or Prometheus |
| **Replay Mechanism** | Build an admin UI or scheduled job to retry failed messages from DB |
| **Monitoring** | Monitor DLQ depth, retry rate, and failure counts over time |
