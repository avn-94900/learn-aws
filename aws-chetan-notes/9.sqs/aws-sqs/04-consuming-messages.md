# 04 — Consuming Messages

---

## Core Consumer Operations

| Operation | Method | Notes |
|---|---|---|
| Receive messages | `receiveMessage()` | Up to 10 per call |
| Delete a message | `deleteMessage()` | **Required** after successful processing |
| Delete batch | `deleteMessageBatch()` | Efficient cleanup after batch processing |
| Change visibility | `changeMessageVisibility()` | Extend timeout for long-running tasks |

---

## Annotation-Based Listener (`@SqsListener`)

The simplest and most common way to consume messages in Spring Boot.

```java
@Component
public class OrderMessageListener {

    // Basic listener — Spring deletes message automatically on success
    @SqsListener("orders-queue")
    public void handleOrder(Order order) {
        System.out.println("Processing order: " + order.getId());
        // throw any exception → message returns to queue for retry
    }

    // With headers and payload annotations
    @SqsListener("${sqs.queue.orders}")
    public void handleOrderWithHeaders(
            @Payload String message,
            @Header Map<String, Object> headers,
            @Header("MessageId") String messageId) {

        log.info("Message ID: {}", messageId);
        log.info("Headers: {}", headers);
        log.info("Body: {}", message);
        processOrder(message);
    }

    // Multiple queues
    @SqsListener({"orders-queue", "priority-orders-queue"})
    public void handleMultipleQueues(String message) {
        processOrder(message);
    }
}
```

---

## Deletion Policies

Control when SQS deletes a message after your listener runs.

| Policy | Behavior | Use Case |
|---|---|---|
| `ON_SUCCESS` (default) | Delete if no exception thrown | Standard processing |
| `NEVER` | Never delete automatically — you must call `ack.acknowledge()` | Critical messages requiring explicit confirmation |
| `NO_REDRIVE` | Delete only if no DLQ redrive policy is configured | DLQ-aware scenarios |

### Manual Acknowledgment with `NEVER` Policy

```java
@SqsListener(value = "orders-queue", deletionPolicy = SqsMessageDeletionPolicy.NEVER)
public void handleWithManualAck(String message, Acknowledgment ack) {
    try {
        processBusinessLogic(message);
        ack.acknowledge();  // explicitly delete the message
    } catch (BusinessException e) {
        log.error("Business logic failed: {}", e.getMessage());
        // do NOT acknowledge — message returns to queue and will be retried
    } catch (Exception e) {
        log.error("Unexpected error, acknowledging to prevent infinite loop: {}", e.getMessage());
        ack.acknowledge();  // acknowledge to avoid poison message loop
    }
}
```

---

## Manual Message Processing (Without `@SqsListener`)

Use this pattern when you need full control over polling, batching, and deletion.

```java
@Component
public class ManualConsumer {

    @Autowired
    private AmazonSQS amazonSQS;

    public void pollAndProcess(String queueUrl) {
        ReceiveMessageRequest request = new ReceiveMessageRequest()
            .withQueueUrl(queueUrl)
            .withMaxNumberOfMessages(10)   // receive up to 10 at once
            .withWaitTimeSeconds(20);      // long polling

        ReceiveMessageResult result = amazonSQS.receiveMessage(request);

        for (Message message : result.getMessages()) {
            try {
                processMessage(message.getBody());

                // Delete after successful processing
                amazonSQS.deleteMessage(queueUrl, message.getReceiptHandle());

            } catch (Exception e) {
                log.error("Failed to process message {}: {}",
                    message.getMessageId(), e.getMessage());
                // Do NOT delete — message will become visible again after visibility timeout
            }
        }
    }
}
```

---

## SDK v2 Consumer

```java
@Service
public class SqsConsumerService {

    @Autowired
    private SqsClient sqsClient;

    public void receiveAndProcess(String queueUrl) {
        ReceiveMessageRequest request = ReceiveMessageRequest.builder()
            .queueUrl(queueUrl)
            .maxNumberOfMessages(10)
            .waitTimeSeconds(20)
            .build();

        ReceiveMessageResponse response = sqsClient.receiveMessage(request);

        for (software.amazon.awssdk.services.sqs.model.Message message
                : response.messages()) {
            try {
                processMessage(message.body());
                deleteMessage(queueUrl, message.receiptHandle());
            } catch (Exception e) {
                log.error("Processing failed for message {}: {}",
                    message.messageId(), e.getMessage());
            }
        }
    }

    private void deleteMessage(String queueUrl, String receiptHandle) {
        DeleteMessageRequest deleteRequest = DeleteMessageRequest.builder()
            .queueUrl(queueUrl)
            .receiptHandle(receiptHandle)
            .build();
        sqsClient.deleteMessage(deleteRequest);
    }
}
```

---

## Batch Processing

### With `@SqsListener`

```java
@SqsListener(value = "${sqs.queue.orders}", deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)
public void processBatch(@Payload List<String> messages,
                         @Header List<Map<String, Object>> headers) {

    for (int i = 0; i < messages.size(); i++) {
        try {
            processMessage(messages.get(i));
        } catch (Exception e) {
            log.error("Failed to process message {}: {}", i, e.getMessage());
            // Handle individual failure — other messages in batch can still succeed
        }
    }
}
```

### Batch Delete (Manual)

```java
public void batchDelete(String queueUrl, List<Message> processedMessages) {
    List<DeleteMessageBatchRequestEntry> entries = processedMessages.stream()
        .map(msg -> new DeleteMessageBatchRequestEntry()
            .withId(msg.getMessageId())
            .withReceiptHandle(msg.getReceiptHandle()))
        .collect(Collectors.toList());

    DeleteMessageBatchRequest request = new DeleteMessageBatchRequest()
        .withQueueUrl(queueUrl)
        .withEntries(entries);

    amazonSQS.deleteMessageBatch(request);
}
```

---

## Visibility Timeout Deep Dive

The visibility timeout is the period during which a received message is **hidden from all other consumers**.

```
Consumer A receives message
    │
    ├── Message becomes invisible (visibility timeout clock starts)
    │
    ├── Consumer A successfully processes → deletes message ✓
    │
    └── Consumer A crashes / timeout expires → message becomes visible again
                                               → another consumer can pick it up
```

**Configuration rules:**
- Set visibility timeout > your maximum expected processing time
- If processing is taking longer, call `ChangeMessageVisibility` to extend it
- Default: 30 seconds. Maximum: 12 hours.

### Extending Visibility Timeout During Processing

```java
public void extendVisibilityIfNeeded(String queueUrl, String receiptHandle, int additionalSeconds) {
    ChangeMessageVisibilityRequest request = new ChangeMessageVisibilityRequest()
        .withQueueUrl(queueUrl)
        .withReceiptHandle(receiptHandle)
        .withVisibilityTimeout(additionalSeconds);

    amazonSQS.changeMessageVisibility(request);
    log.info("Extended visibility timeout by {} seconds", additionalSeconds);
}
```

---

## Visibility Timeout vs Delay Seconds

| Setting | Purpose | Scope | Range |
|---|---|---|---|
| **Visibility Timeout** | Hides message after receipt while being processed | Per receive operation | 0 s – 12 h |
| **Delay Seconds** | Postpones initial delivery of a newly sent message | Per send operation | 0 s – 15 min |

---

## Best Practices for Consumers

1. **Always delete messages** after successful processing — never assume auto-deletion
2. **Set visibility timeout greater than processing time** to prevent duplicate processing
3. **Use long polling** (`WaitTimeSeconds = 20`) to reduce empty responses and cost
4. **Implement idempotency** — standard queues can deliver a message more than once
5. **Use DLQ** to handle messages that keep failing — see `05-error-handling-and-dlq.md`
6. **Use batch receive** (`MaxNumberOfMessages = 10`) for higher throughput
7. **Scale consumers** based on queue depth — monitor `ApproximateNumberOfMessages`
8. **Handle partial batch failures** — don't let one bad message block the rest
