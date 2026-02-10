# AWS SQS Complete Study Guide

## What is Amazon SQS?

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

**Key Benefits:**

- Fully managed service with no infrastructure management required
- Pull-based messaging model where consumers poll for messages
- Decouples components in distributed architectures
- Reliable message delivery with guaranteed at-least-once delivery
- Elastic scaling that automatically handles load
- Pay-per-use pricing with no upfront costs
- Integration with other AWS services like Lambda, EC2, and ECS
- Security through encryption in transit and at rest
- Monitoring via CloudWatch integration for metrics and alarms

---

## SQS Queue

An SQS queue is a temporary repository for messages that are waiting to be processed.

| Property | Description |
|----------|-------------|
| **Definition** | A buffer that stores messages between sending and receiving applications |
| **Purpose** | Decouples producers from consumers, allowing independent scaling |
| **Types** | Standard Queues and FIFO Queues |
| **Capacity** | Virtually unlimited number of messages |
| **Retention** | Messages can be retained for up to 14 days with a default of 4 days |

### Queue Naming Conventions

- Standard Queue: Can be any valid name like `order-processing-queue`
- FIFO Queue: Must end with `.fifo` suffix like `payment-queue.fifo`

---

## Standard Queue vs FIFO Queue

| Feature | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| **Throughput** | Nearly unlimited transactions per second | 300 TPS (3,000 with batching) |
| **Ordering** | Best-effort ordering, may arrive out of order | Strict first-in-first-out ordering |
| **Delivery** | At-least-once delivery, may have duplicates | Exactly-once processing, no duplicates |
| **Cost** | Lower cost per request | Higher cost per request |
| **Naming** | Any valid name | Must end with `.fifo` |
| **Requirements** | None | MessageGroupId and optional MessageDeduplicationId |
| **Availability** | Available in all AWS regions | Available in all AWS regions |

### When to Choose Which

**Standard Queue:**

- High-volume applications
- Log processing
- Image processing
- Email sending
- Applications that can handle duplicate messages
- Applications that do not require strict ordering

**FIFO Queue:**

- Financial transactions
- Order processing
- Inventory management
- Chat applications
- Applications requiring strict ordering
- Applications that cannot tolerate duplicates

---

## SQS vs Other Messaging Services

### SQS vs Apache Kafka

| Feature | SQS | Apache Kafka |
|---------|-----|-------------|
| **Management** | Fully managed | Self-managed or managed via MSK |
| **Message Ordering** | FIFO queues only | Partition-level ordering |
| **Throughput** | 300 TPS (FIFO), unlimited (Standard) | Very high, millions of messages per second |
| **Message Retention** | Up to 14 days | Configurable, default 7 days, can be indefinite |
| **Consumer Model** | Pull-based | Pull-based |
| **Replay Capability** | No, messages deleted after processing | Yes, offset-based |
| **Use Case** | Simple queuing, microservices | Event streaming, real-time analytics |

### SQS vs RabbitMQ

| Feature | SQS | RabbitMQ |
|---------|-----|----------|
| **Hosting** | AWS managed | Self-hosted or Amazon MQ |
| **Message Routing** | Simple queue-based | Advanced with exchanges and routing keys |
| **Protocols** | HTTP/HTTPS APIs | AMQP, MQTT, STOMP |
| **Message Patterns** | Point-to-point | Point-to-point, publish-subscribe |
| **Persistence** | Always persistent | Configurable |
| **Clustering** | Built-in across AZs | Manual setup required |

### SQS vs SNS

**SQS:** Point-to-point messaging using queues

**SNS:** Pub-sub messaging using topic broadcast

---

## Message Size Limits and Large Payload Handling

### Message Size Constraints

| Constraint | Limit |
|------------|-------|
| **Maximum message size** | 256 KB including message body and attributes |
| **Minimum message size** | 1 byte |
| **Message attributes** | Each attribute can be up to 256 KB |

### Handling Large Payloads

| Option | Description | Details |
|--------|-------------|---------|
| **S3 + SQS Pattern** | Store payload in S3, send reference in SQS | Store actual payload in S3 and send a reference in the SQS message |
| **SQS Extended Client Library** | Automatic handling for Java | Stores messages over 256KB in S3, sends S3 reference in SQS. Application code does not change |
| **Message Chunking** | Split messages manually | Split large messages into smaller chunks, use message attributes to track chunk information, reassemble on consumer side |

#### Example: S3 + SQS Pattern

```json
{
  "messageType": "large-payload",
  "s3Bucket": "my-app-large-messages",
  "s3Key": "orders/2024/order-12345.json",
  "messageId": "abc-123-def",
  "contentType": "application/json",
  "size": 5242880
}
```

---

## Message Producer

Producers are applications or services that send messages to SQS queues.

| Property | Description |
|----------|-------------|
| **Definition** | Any application that sends messages to an SQS queue |
| **Method** | Uses `SendMessage` API call to add messages to queues |
| **Batching** | Can send up to 10 messages at once using `SendMessageBatch` |
| **Authentication** | Requires proper IAM permissions to send messages |

### Sending Messages Programmatically

```java
@Service
public class SQSMessageProducer {
    
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

### Batch Sending in SQS

```java
public void sendBatchMessages(String queueUrl, List<String> messages) {
    List<SendMessageBatchRequestEntry> entries = new ArrayList<>();
    
    for (int i = 0; i < messages.size(); i++) {
        entries.add(new SendMessageBatchRequestEntry()
            .withId(String.valueOf(i))
            .withMessageBody(messages.get(i)));
    }
    
    SendMessageBatchRequest batchRequest = new SendMessageBatchRequest()
        .withQueueUrl(queueUrl)
        .withEntries(entries);
    
    SendMessageBatchResult result = amazonSQS.sendMessageBatch(batchRequest);
}
```

**Benefits of Batch Sending:**

| Benefit | Description |
|---------|-------------|
| **Reduced API calls** | Send up to 10 messages per batch |
| **Lower costs** | Fewer API requests mean reduced charges |
| **Better throughput** | Process more messages in less time |
| **Reduced latency** | Less network overhead |

### Handling Message Publishing Failures

```java
@Service
public class RobustSQSProducer {
    
    @Retryable(value = {Exception.class}, maxAttempts = 3, backoff = @Backoff(delay = 1000))
    public void sendMessageWithRetry(String queueUrl, String message) {
        try {
            SendMessageRequest request = new SendMessageRequest()
                .withQueueUrl(queueUrl)
                .withMessageBody(message);
            
            amazonSQS.sendMessage(request);
        } catch (AmazonSQSException e) {
            log.error("Failed to send message: {}", e.getMessage());
            throw e;
        }
    }
    
    @Recover
    public void recover(Exception ex, String queueUrl, String message) {
        log.error("Failed to send message after retries: {}", message);
    }
}
```

### Message Payload Design Best Practices

```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class OrderMessage {
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private LocalDateTime timestamp;
    private String eventType;
    
    // Keep payload small and focused
    // Use references instead of embedding large objects
    // Include versioning for schema evolution
}
```

### Message Deduplication in FIFO Queues

```java
public void sendFIFOMessage(String queueUrl, String messageBody, String groupId) {
    SendMessageRequest request = new SendMessageRequest()
        .withQueueUrl(queueUrl)
        .withMessageBody(messageBody)
        .withMessageGroupId(groupId)
        .withMessageDeduplicationId(generateDeduplicationId(messageBody));
    
    amazonSQS.sendMessage(request);
}

private String generateDeduplicationId(String messageBody) {
    return DigestUtils.sha256Hex(messageBody + System.currentTimeMillis());
}
```

**Key Differences:**

- MessageGroupId: Groups related messages for ordered processing
- MessageDeduplicationId: Prevents duplicate messages within 5-minute window

---

## Message Consumer

Consumers are applications that retrieve and process messages from SQS queues.

| Property | Description |
|----------|-------------|
| **Definition** | Applications that poll queues and process messages |
| **Method** | Uses `ReceiveMessage` API call to retrieve messages |
| **Polling Types** | Short polling (immediate return) and Long polling (waits up to 20 seconds) |
| **Processing** | Must explicitly delete messages after successful processing |
| **Concurrency** | Multiple consumers can process messages from the same queue simultaneously |

### Receiving and Processing Messages

```java
@Component
public class SQSMessageConsumer {
    
    @SqsListener("${sqs.queue.name}")
    public void processMessage(@Payload String message, 
                              @Header Map<String, Object> headers) {
        try {
            processBusinessLogic(message);
        } catch (Exception e) {
            log.error("Error processing message: {}", e.getMessage());
            throw e;
        }
    }
    
    public void manualMessageProcessing(String queueUrl) {
        ReceiveMessageRequest request = new ReceiveMessageRequest()
            .withQueueUrl(queueUrl)
            .withMaxNumberOfMessages(10)
            .withWaitTimeSeconds(20);
        
        ReceiveMessageResult result = amazonSQS.receiveMessage(request);
        
        for (Message message : result.getMessages()) {
            try {
                processMessage(message.getBody());
                amazonSQS.deleteMessage(queueUrl, message.getReceiptHandle());
            } catch (Exception e) {
                log.error("Failed to process message: {}", e.getMessage());
            }
        }
    }
}
```

### Batch Message Processing

```java
@SqsListener(value = "${sqs.queue.name}", deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)
public void processBatchMessages(@Payload List<String> messages,
                               @Header List<Map<String, Object>> headers) {
    for (int i = 0; i < messages.size(); i++) {
        try {
            processMessage(messages.get(i));
        } catch (Exception e) {
            log.error("Failed to process message {}: {}", i, e.getMessage());
        }
    }
}
```

### Message Processing Failure Handling

```java
@Bean
@SqsListener(value = "${sqs.queue.name}", deletionPolicy = SqsMessageDeletionPolicy.NEVER)
public void processWithManualAck(String message, Acknowledgment ack) {
    try {
        processBusinessLogic(message);
        ack.acknowledge();
    } catch (BusinessException e) {
        log.error("Business logic failed: {}", e.getMessage());
    } catch (Exception e) {
        log.error("Unexpected error: {}", e.getMessage());
        ack.acknowledge();
    }
}
```

---

## Message Lifecycle

1. **Message Sent:** Producer sends message to queue
2. **Message Available:** Message is available for consumption
3. **Message Received:** Consumer receives message and it becomes invisible
4. **Visibility Timeout:** Message invisible to other consumers
5. **Processing:** Consumer processes the message
6. **Message Deleted:** Successful processing leads to message deletion
7. **Message Returns:** Failed processing makes message visible again
8. **Dead Letter Queue:** After max retries, message sent to DLQ

---

## Message Visibility Timeout

Visibility timeout is the period during which a message is hidden from other consumers after being retrieved.

| Property | Description |
|----------|-------------|
| **Definition** | Time period when a message is invisible to other consumers |
| **Purpose** | Prevents multiple consumers from processing the same message simultaneously |
| **Default** | 30 seconds |
| **Range** | Configurable from 0 seconds to 12 hours |
| **Behavior** | Message becomes visible again if not deleted within timeout period |
| **Extension** | Can be extended using `ChangeMessageVisibility` API |

### Why Visibility Timeout is Important

| Benefit | Description |
|---------|-------------|
| **Prevents duplicate processing** | Ensures only one consumer processes a message at a time |
| **Handles consumer failures** | Message becomes visible again if consumer crashes |
| **Allows processing time** | Gives consumers time to process complex messages |
| **Enables retry logic** | Failed messages can be retried by other consumers |

### Best Practices

- Set timeout slightly longer than your expected processing time
- Monitor processing times and adjust accordingly
- Use CloudWatch metrics to track visibility timeout extensions
- Consider using `ChangeMessageVisibility` for long-running processes

---

## Message Retention Period

| Property | Value |
|----------|-------|
| **Default retention** | 4 days |
| **Minimum retention** | 60 seconds (1 minute) |
| **Maximum retention** | 14 days |
| **Configuration** | Set via queue attributes or during queue creation |

### Importance

| Aspect | Description |
|--------|-------------|
| **Message Persistence** | Ensures messages are not lost during consumer downtime |
| **Replay Capability** | Allows reprocessing of messages within retention window |
| **Cost Optimization** | Longer retention means higher storage costs |
| **Compliance** | Some industries require specific retention periods |

---

## Dead Letter Queue (DLQ)

A dead letter queue is a special queue that receives messages that cannot be processed successfully.

| Property | Description |
|----------|-------------|
| **Definition** | Queue for messages that have failed processing multiple times |
| **Purpose** | Isolates problematic messages for debugging and analysis |
| **Configuration** | Set maximum receive count (1-1000) before moving to DLQ |
| **Requirements** | Must be the same type (Standard or FIFO) as the source queue |
| **Retention** | Messages in DLQ can be retained for up to 14 days |

---

## Message Attributes

Message attributes provide metadata about messages without affecting the message body.

| Property | Description |
|----------|-------------|
| **Definition** | Key-value pairs that provide additional message metadata |
| **Types** | String, Number, Binary data types |
| **Limit** | Up to 10 attributes per message |
| **Size** | Each attribute name and value can be up to 256 KB |
| **Use Case** | Filtering, routing, and processing logic without parsing message body |

### Common Message Attributes

| Attribute Name | DataType | When to Use | Example Value |
|---------------|----------|-------------|---------------|
| ContentType | String | Indicate the type of message body | `application/json` |
| SourceSystem | String | Identify which system sent the message | `OrderService` |
| EventType | String | Route different types of events | `USER_REGISTERED`, `PAYMENT_SUCCESS` |
| CorrelationId | String | Track and trace across systems and logs | `abc-123-def-456` |
| RetryCount | Number | Implement custom retry logic | `2` |
| Priority | Number | Decide message processing priority | `5` |
| X-Tenant-Id | String | Identify customer or tenant in multi-tenant apps | `tenant_42` |
| Language | String | For localization or message translation | `en-US`, `fr-FR` |
| UserRole | String | Access-based processing or filtering | `admin`, `customer` |

### Use Case Examples

#### Order Service to Payment Service

```json
{
  "ContentType": "application/json",
  "SourceSystem": "OrderService",
  "EventType": "ORDER_PLACED",
  "CorrelationId": "txn-001-xyz"
}
```

Helps downstream services know the source, content format, and event type.

#### Retry Logic in Background Job

```json
{
  "RetryCount": 3,
  "EventType": "PAYMENT_RETRY"
}
```

Helps consumer know how many times a message was retried.

#### Multi-Region User Signup

```json
{
  "Language": "en-IN",
  "EventType": "USER_SIGNUP",
  "X-Tenant-Id": "india-west-zone"
}
```

Used by notification services to send region or language-specific emails or SMS.

### Why Use Message Attributes

| Benefit | Description |
|---------|-------------|
| **Separation of concerns** | Keep actual payload (business data) in Body and meta information in MessageAttributes |
| **Avoid duplicating info** | No need to add system-level information in the body that is only needed for routing or filtering |
| **Leverage filtering** | Use in SNS to SQS fanout patterns |

### Filtering Messages (SNS to SQS)

In SNS + SQS architecture, you can use MessageAttributes to filter which messages a queue receives.

```json
"MessageAttributes": {
  "EventType": {
    "DataType": "String",
    "StringValue": "ORDER_PLACED"
  }
}
```

Configure the SNS subscription filter policy:

```json
{
  "EventType": ["ORDER_PLACED"]
}
```

Only messages with that attribute will be delivered to the target SQS queue.

### Routing Logic

The consumer can inspect MessageAttributes and take different actions without parsing the body.

```java
if (msg.getMessageAttributes().get("UserRole").getStringValue().equals("admin")) {
    processAdminLogic();
} else {
    processUserLogic();
}
```

### Passing Metadata

Add tracing info or content type without cluttering your payload.

```json
"MessageAttributes": {
  "ContentType": {
    "DataType": "String",
    "StringValue": "application/json"
  },
  "CorrelationId": {
    "DataType": "String",
    "StringValue": "abc-123-def-456"
  }
}
```

Your logging system or distributed tracing tool can use CorrelationId without needing to parse the body.

### Custom Message Attributes for Filtering

```java
public void sendMessageWithAttributes(String queueUrl, String messageBody, 
                                    Map<String, String> attributes) {
    Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
    
    attributes.forEach((key, value) -> 
        messageAttributes.put(key, new MessageAttributeValue()
            .withStringValue(value)
            .withDataType("String"))
    );
    
    SendMessageRequest request = new SendMessageRequest()
        .withQueueUrl(queueUrl)
        .withMessageBody(messageBody)
        .withMessageAttributes(messageAttributes);
    
    amazonSQS.sendMessage(request);
}
```

---

## SQS Message Anatomy

Each SQS message includes the following components:

| Component | Description |
|-----------|-------------|
| **Message Body** | The actual data payload up to 256 KB |
| **Message ID** | Unique identifier assigned by SQS when message is sent |
| **Receipt Handle** | Unique identifier for each time a message is received, required for deletion |
| **MD5 Hash** | Hash of the message body for integrity verification |
| **Message Attributes** | Optional metadata key-value pairs |
| **Timestamps** | Sent timestamp and approximate first receive timestamp |

### Sample Message in JSON Format

```json
{
  "MessageId": "e2f76847-3d0e-42c9-a2dc-5b98e8b6d403",
  "ReceiptHandle": "AQEBzZzOaGf4L2hC5iCwUkN7...shF+A==",
  "MD5OfBody": "7b270e59b47ff90a553787216d55d91d",
  "Body": "{\"orderId\":12345, \"amount\":5000, \"currency\":\"INR\"}",
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

| Polling Type | Behavior | Cost | Use Case |
|--------------|----------|------|----------|
| **Short Polling** | Returns immediately, even if no messages are available | Charges for each request, regardless of message availability | When you need immediate response or have predictable message flow |
| **Long Polling** | Waits up to 20 seconds for messages to become available | More cost-effective as it reduces empty responses | Most common use case for cost optimization |

### Long Polling Configuration

| Property | Description |
|----------|-------------|
| **Configuration** | Set `WaitTimeSeconds` parameter (1-20 seconds) |
| **Benefit** | Reduces the number of API calls and improves message delivery efficiency |

### Key Timeout Configurations

| Configuration | Description | Range |
|---------------|-------------|-------|
| **ReceiveMessageWaitTimeSeconds** | Long polling duration. 0 = Short polling (immediate return). Greater than 0 = Long polling (waits for messages) | 0-20 seconds |
| **VisibilityTimeout** | Time message is hidden after receipt. Should be longer than processing time. Prevents duplicate processing | 0 seconds to 12 hours |

---

## Message Ordering and Deduplication (FIFO Queues)

| Feature | Purpose | Details |
|---------|---------|---------|
| **Message Group ID** | Groups related messages for ordered processing | Required for FIFO queues. Messages with the same Group ID are processed in order. Different Group IDs can be processed in parallel |
| **Deduplication ID** | Prevents duplicate messages | Prevents duplicates within 5-minute deduplication interval. Can be provided explicitly or generated from message body hash. Applies within the same Message Group ID. Ensures exactly-once message processing |

---

## Access Control and Security

### IAM Policies

| Policy Type | Description |
|-------------|-------------|
| **Resource-based** | Queue policies that control access to specific queues |
| **Identity-based** | IAM user or role policies that grant SQS permissions |
| **Actions** | Include `sqs:SendMessage`, `sqs:ReceiveMessage`, `sqs:DeleteMessage` |

### Encryption

| Encryption Type | Description |
|-----------------|-------------|
| **Server-Side Encryption (SSE)** | Encrypts messages at rest using AWS KMS |
| **In-Transit** | All API calls use HTTPS for encryption in transit |
| **Key Management** | Uses AWS managed keys or customer managed keys |

---

## Best Practices and Patterns

### Batch Processing

| Practice | Description |
|----------|-------------|
| **Send Batching** | Use `SendMessageBatch` to send up to 10 messages at once |
| **Receive Batching** | Use `MaxNumberOfMessages` parameter (up to 10) to receive multiple messages |
| **Benefit** | Reduces API calls and improves throughput |

### Error Handling

| Practice | Description |
|----------|-------------|
| **Retry Logic** | Implement exponential backoff for failed operations |
| **Dead Letter Queues** | Configure DLQ for messages that fail repeatedly |
| **Alarming** | Set up CloudWatch alarms for queue depth and DLQ messages |

### Scaling Considerations

| Practice | Description |
|----------|-------------|
| **Consumer Scaling** | Scale consumers based on queue depth and processing time |
| **Queue Separation** | Use separate queues for different message types or priorities |
| **Regional Distribution** | Consider cross-region replication for disaster recovery |

### Best Practices for SQS Message Consumption

| Practice | Description |
|----------|-------------|
| **Idempotency** | Ensure processing is idempotent |
| **Error Handling** | Implement proper exception handling |
| **Timeouts** | Configure appropriate visibility timeouts |
| **Batching** | Use batch processing for higher throughput |
| **Dead Letter Queues** | Configure DLQ for failed messages |
| **Monitoring** | Implement CloudWatch metrics and alarms |
| **Scaling** | Use multiple consumers for high volume |

---

## Spring Boot Integration

### Dependencies

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws-messaging</artifactId>
</dependency>
```

### Configuration

```yaml
cloud:
  aws:
    region:
      static: us-east-1
    credentials:
      access-key: ${AWS_ACCESS_KEY}
      secret-key: ${AWS_SECRET_KEY}
    sqs:
      queue:
        name: my-queue
```

### Advanced Spring Boot SQS Configuration

```java
@Configuration
public class SQSConfig {
    
    @Bean
    public AmazonSQSAsync amazonSQSAsync() {
        return AmazonSQSAsyncClientBuilder.standard()
            .withRegion(Regions.US_EAST_1)
            .withCredentials(new DefaultAWSCredentialsProviderChain())
            .build();
    }
    
    @Bean
    public SimpleMessageListenerContainerFactory simpleMessageListenerContainerFactory() {
        SimpleMessageListenerContainerFactory factory = new SimpleMessageListenerContainerFactory();
        factory.setAmazonSqs(amazonSQSAsync());
        factory.setMaxNumberOfMessages(10);
        factory.setWaitTimeOut(20);
        return factory;
    }
}
```

### Example with Spring Boot Listener

```java
@SqsListener("order-queue")
public void processOrder(String messageBody,
                         @Header("SenderId") String senderId,
                         @Header("ContentType") String contentType) {
    System.out.println("Received order: " + messageBody);
    System.out.println("Sender: " + senderId + " - ContentType: " + contentType);
}
```

### Example with Attributes in SendMessage

```java
@Service
public class SqsMessageSender {

    private final SqsClient sqsClient;
    private final ObjectMapper objectMapper = new ObjectMapper();

    public SqsMessageSender(SqsClient sqsClient) {
        this.sqsClient = sqsClient;
    }

    public void sendOrderMessage(String queueUrl, OrderMessage order) {
        try {
            String body = objectMapper.writeValueAsString(order);

            SendMessageRequest sendRequest = SendMessageRequest.builder()
                    .queueUrl(queueUrl)
                    .messageBody(body)
                    .messageAttributes(Map.of(
                            "ContentType", MessageAttributeValue.builder()
                                    .stringValue("application/json")
                                    .dataType("String")
                                    .build(),
                            "SourceSystem", MessageAttributeValue.builder()
                                    .stringValue("OrderService")
                                    .dataType("String")
                                    .build()
                    ))
                    .build();

            sqsClient.sendMessage(sendRequest);
            System.out.println("Message sent: " + body);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## Performance Optimization and Monitoring

### Key Metrics to Monitor

| Metric | Description |
|--------|-------------|
| **ApproximateNumberOfMessages** | Messages in queue |
| **ApproximateNumberOfMessagesNotVisible** | Messages being processed |
| **NumberOfMessagesSent** | Messages sent to queue |
| **NumberOfMessagesReceived** | Messages received from queue |
| **NumberOfMessagesDeleted** | Successfully processed messages |

### Scaling Strategies

| Strategy | Description |
|----------|-------------|
| **Horizontal scaling** | Multiple consumer instances |
| **Vertical scaling** | Increase processing power per consumer |
| **Auto-scaling** | Scale based on queue depth |
| **Batch processing** | Process multiple messages together |

---

## Common Interview Questions and Answers

**Q: How do you ensure message ordering in SQS?**

A: Use FIFO queues with MessageGroupId to maintain order within message groups.

**Q: What happens if a consumer crashes while processing a message?**

A: The message becomes visible again after the visibility timeout expires and can be processed by another consumer.

**Q: How do you handle duplicate messages in standard queues?**

A: Implement idempotent processing logic and use unique identifiers to detect duplicates.

**Q: When would you use Dead Letter Queues?**

A: For messages that repeatedly fail processing, to prevent infinite retries and enable manual investigation.

**Q: How do you optimize SQS costs?**

A: Use long polling, batch operations, appropriate message retention periods, and right-size your queues.

**Q: Difference between SQS and SNS?**

A: SQS is for point-to-point messaging using queues. SNS is for pub-sub messaging using topic broadcast.

**Q: What is the difference between MessageGroupId and MessageDeduplicationId?**

A: MessageGroupId groups related messages for ordered processing. MessageDeduplicationId prevents duplicate messages within a 5-minute window.

**Q: How do you handle large payloads in SQS?**

A: Use the S3 + SQS pattern where you store the payload in S3 and send a reference in the SQS message, or use the SQS Extended Client Library.

**Q: What is the benefit of long polling over short polling?**

A: Long polling reduces the number of empty responses, lowers costs, and improves message delivery efficiency by waiting up to 20 seconds for messages.

**Q: How do you implement retry logic for failed messages?**

A: Configure a Dead Letter Queue with a maximum receive count. After a message fails processing the specified number of times, it gets moved to the DLQ for investigation.