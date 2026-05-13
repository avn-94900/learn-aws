# 03 — Producing Messages

---

## Core SQS Producer Operations

| Operation | Method | Notes |
|---|---|---|
| Send single message | `sendMessage()` | Most common operation |
| Send batch of messages | `sendMessageBatch()` | Up to 10 messages per call |
| Get queue URL | `getQueueUrl()` | Required before any operation |
| Get queue attributes | `getQueueAttributes()` | Queue depth, settings, etc. |

---

## Sending a Single Message

### Using `QueueMessagingTemplate` (Spring Cloud AWS — Recommended)

```java
@Service
public class OrderProducer {

    @Autowired
    private QueueMessagingTemplate queueMessagingTemplate;

    // Simple string message
    public void sendOrder(String orderJson) {
        queueMessagingTemplate.convertAndSend("orders-queue", orderJson);
    }

    // Object — Spring handles JSON serialization automatically
    public void sendOrderObject(Order order) {
        queueMessagingTemplate.convertAndSend("orders-queue", order);
    }

    // With custom headers
    public void sendOrderWithHeaders(Order order) {
        Map<String, Object> headers = Map.of(
            "priority", "high",
            "sourceSystem", "OrderService",
            "timestamp", System.currentTimeMillis()
        );
        queueMessagingTemplate.convertAndSend("orders-queue", order, headers);
    }
}
```

### Using `AmazonSQS` SDK v1 (Lower-Level)

```java
@Service
public class SQSProducerV1 {

    @Autowired
    private AmazonSQS amazonSQS;

    public void sendMessage(String queueUrl, String messageBody) {
        SendMessageRequest request = new SendMessageRequest()
            .withQueueUrl(queueUrl)
            .withMessageBody(messageBody);

        SendMessageResult result = amazonSQS.sendMessage(request);
        System.out.println("Message ID: " + result.getMessageId());
    }
}
```

### Using AWS SDK v2 `SqsClient`

```java
@Service
public class SqsProducerService {

    @Autowired
    private SqsClient sqsClient;

    @Autowired
    private SqsUtil sqsUtil; // helper to resolve queue URL from name

    public SendMessageResponse sendMessage(String queueName, MessageEntity messageEntity) {
        String queueUrl = sqsUtil.getQueueUrl(queueName);

        SendMessageRequest request = SendMessageRequest.builder()
            .queueUrl(queueUrl)
            .messageBody(messageEntity.getBody())
            .messageAttributes(
                sqsUtil.convertToMessageAttributes(messageEntity.getAttributes())
            )
            .delaySeconds(messageEntity.getDelaySeconds())
            .build();

        SendMessageResponse response = sqsClient.sendMessage(request);
        log.info("Message sent. ID: {}", response.messageId());
        return response;
    }
}
```

---

## Sending with Message Attributes

Message attributes are metadata attached to the message body — useful for filtering and routing without parsing the body.

- Maximum 10 attributes per message
- Supported types: `String`, `Number`, `Binary`

```java
@Service
public class SqsMessageSender {

    @Autowired
    private SqsClient sqsClient;

    public void sendWithAttributes(String queueUrl, OrderMessage order) throws Exception {
        String body = new ObjectMapper().writeValueAsString(order);

        SendMessageRequest request = SendMessageRequest.builder()
            .queueUrl(queueUrl)
            .messageBody(body)
            .messageAttributes(Map.of(
                "ContentType", MessageAttributeValue.builder()
                    .dataType("String")
                    .stringValue("application/json")
                    .build(),
                "SourceSystem", MessageAttributeValue.builder()
                    .dataType("String")
                    .stringValue("OrderService")
                    .build()
            ))
            .build();

        sqsClient.sendMessage(request);
    }
}
```

---

## Sending a Delayed Message

Use `delaySeconds` to postpone delivery of a specific message (0–900 seconds / up to 15 minutes).

```java
SendMessageRequest request = new SendMessageRequest()
    .withQueueUrl(queueUrl)
    .withMessageBody(messageBody)
    .withDelaySeconds(30); // deliver after 30 seconds

amazonSQS.sendMessage(request);
```

> **Delay Queue vs Per-Message Delay:**
> - **Delay Queue**: Sets a default delay for ALL messages to that queue (queue-level attribute)
> - **Per-Message Delay**: `delaySeconds` on `SendMessageRequest` overrides queue default for that single message

---

## Batch Sending

Batch up to 10 messages per API call to reduce costs and improve throughput.

### SDK v1

```java
public void sendBatchMessages(String queueUrl, List<String> messages) {
    List<SendMessageBatchRequestEntry> entries = new ArrayList<>();

    for (int i = 0; i < messages.size(); i++) {
        entries.add(new SendMessageBatchRequestEntry()
            .withId(String.valueOf(i))          // unique ID within this batch
            .withMessageBody(messages.get(i)));
    }

    SendMessageBatchRequest batchRequest = new SendMessageBatchRequest()
        .withQueueUrl(queueUrl)
        .withEntries(entries);

    SendMessageBatchResult result = amazonSQS.sendMessageBatch(batchRequest);

    // Always check for partial failures
    result.getFailed().forEach(failed ->
        log.error("Failed to send message {}: {}", failed.getId(), failed.getMessage())
    );
}
```

### SDK v2

```java
public SendMessageBatchResponse sendMessageBatch(String queueName,
                                                  List<MessageEntity> messageEntities) {
    String queueUrl = sqsUtil.getQueueUrl(queueName);

    List<SendMessageBatchRequestEntry> entries = messageEntities.stream()
        .map(entity -> SendMessageBatchRequestEntry.builder()
            .id(entity.getId())
            .messageBody(entity.getBody())
            .messageAttributes(
                sqsUtil.convertToMessageAttributes(entity.getAttributes())
            )
            .delaySeconds(entity.getDelaySeconds())
            .build())
        .collect(Collectors.toList());

    SendMessageBatchRequest request = SendMessageBatchRequest.builder()
        .queueUrl(queueUrl)
        .entries(entries)
        .build();

    SendMessageBatchResponse response = sqsClient.sendMessageBatch(request);

    log.info("Sent: {}, Failed: {}",
        response.successful().size(), response.failed().size());

    response.failed().forEach(failed ->
        log.error("Failed ID: {} - Reason: {}", failed.id(), failed.message())
    );

    return response;
}
```

**Benefits of Batch Sending:**
- Reduces API calls by up to 10×
- Lower cost (billed per API call, not per message)
- Better throughput and reduced latency

---

## Sending FIFO Messages

FIFO queues require two additional parameters:

- **`MessageGroupId`** — groups messages for ordered, sequential processing
- **`MessageDeduplicationId`** — prevents duplicates within a 5-minute window

```java
public void sendFIFOMessage(String queueUrl, String messageBody, String groupId) {
    String deduplicationId = DigestUtils.sha256Hex(
        messageBody + System.currentTimeMillis()
    );

    SendMessageRequest request = new SendMessageRequest()
        .withQueueUrl(queueUrl)
        .withMessageBody(messageBody)
        .withMessageGroupId(groupId)
        .withMessageDeduplicationId(deduplicationId);

    amazonSQS.sendMessage(request);
}
```

> **Key difference:**
> - `MessageGroupId` — determines ordering (one consumer handles one group at a time)
> - `MessageDeduplicationId` — prevents the same message from being delivered twice

### Content-Based Deduplication (Alternative to Explicit ID)

Enable **content-based deduplication** on the FIFO queue to have SQS automatically generate a deduplication ID from the message body's MD5 hash — no need to set `MessageDeduplicationId` manually.

---

## Asynchronous Sending (SDK v2)

```java
public CompletableFuture<SendMessageResponse> sendMessageAsync(String queueName,
                                                                MessageEntity messageEntity) {
    String queueUrl = sqsUtil.getQueueUrl(queueName);

    SendMessageRequest request = SendMessageRequest.builder()
        .queueUrl(queueUrl)
        .messageBody(messageEntity.getBody())
        .messageAttributes(
            sqsUtil.convertToMessageAttributes(messageEntity.getAttributes())
        )
        .build();

    return sqsAsyncClient.sendMessage(request)
        .whenComplete((response, throwable) -> {
            if (throwable != null) {
                log.error("Async send failed for queue: {}", queueName, throwable);
            } else {
                log.info("Async send succeeded. MessageId: {}", response.messageId());
            }
        });
}
```

---

## Message Payload Design Best Practices

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class OrderMessage {

    private String orderId;       // include business identifier for deduplication
    private String customerId;
    private BigDecimal amount;
    private LocalDateTime timestamp;
    private String eventType;     // versioning / event type for schema evolution
    private String schemaVersion; // e.g., "v1" — helps handle backwards compatibility

    // Keep payload small and focused
    // Use references (IDs) instead of embedding large nested objects
    // Include eventType and schemaVersion for schema evolution
}
```

**Rules for good message design:**
- Keep body under 64 KB for Lambda compatibility (hard limit is 256 KB)
- Store references (IDs) instead of full nested objects
- Include an `eventType` field for routing and schema evolution
- Use `schemaVersion` to handle backwards-compatible changes

---

## Sync vs Async — When to Use Which

| | Synchronous (`SqsClient`) | Asynchronous (`SqsAsyncClient`) |
|---|---|---|
| **Execution** | Blocks current thread until complete | Non-blocking, returns `CompletableFuture` |
| **Thread Usage** | Occupies your thread | AWS manages threads internally |
| **Throughput** | Moderate | High |
| **Best For** | Simple apps, quick scripts | Reactive apps, high-concurrency services |
