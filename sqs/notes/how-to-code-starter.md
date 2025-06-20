# AWS SQS with Spring Boot - Developer's Complete Guide

## 1. Core Concepts

### 1.1 Amazon SQS Overview
- **SQS (Simple Queue Service)**: Fully managed message queuing service
- **Queue Types**: Standard Queue, FIFO Queue
- **Message Lifecycle**: Send → Receive → Process → Delete
- **Visibility Timeout**: Period during which other consumers can't receive the same message

### 1.2 Spring Cloud AWS SQS Integration
- **Auto-configuration**: Automatic setup of SQS components
- **Annotation-driven**: Use annotations for message handling
- **Template-based**: Programmatic message operations

## 2. Essential Dependencies

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws-messaging</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-sqs</artifactId>
    <version>1.12.261</version>
</dependency>
```

## 3. Core Classes and Methods

### 3.1 AmazonSQS Interface

#### Key Methods:
```java
// Queue Operations
CreateQueueResult createQueue(String queueName)
CreateQueueResult createQueue(CreateQueueRequest request)
DeleteQueueResult deleteQueue(String queueUrl)
DeleteQueueResult deleteQueue(DeleteQueueRequest request)
GetQueueUrlResult getQueueUrl(String queueName)
GetQueueUrlResult getQueueUrl(GetQueueUrlRequest request)

// Message Operations
SendMessageResult sendMessage(String queueUrl, String messageBody)
SendMessageResult sendMessage(SendMessageRequest request)
SendMessageBatchResult sendMessageBatch(SendMessageBatchRequest request)
ReceiveMessageResult receiveMessage(String queueUrl)
ReceiveMessageResult receiveMessage(ReceiveMessageRequest request)
DeleteMessageResult deleteMessage(String queueUrl, String receiptHandle)
DeleteMessageResult deleteMessage(DeleteMessageRequest request)
```

#### Example Usage:
```java
@Service
public class SQSService {
    
    @Autowired
    private AmazonSQS amazonSQS;
    
    public void sendMessage(String queueUrl, String message) {
        // Method 1: Simple send
        amazonSQS.sendMessage(queueUrl, message);
        
        // Method 2: Detailed send with attributes
        SendMessageRequest request = new SendMessageRequest()
            .withQueueUrl(queueUrl)
            .withMessageBody(message)
            .withDelaySeconds(10)
            .withMessageAttributes(Map.of(
                "Author", new MessageAttributeValue()
                    .withStringValue("Spring App")
                    .withDataType("String")
            ));
        amazonSQS.sendMessage(request);
    }
}
```

### 3.2 QueueMessagingTemplate Class

#### Key Methods:
```java
// Send Operations
void send(String destination, Object payload)
void send(String destination, Message<?> message)
void sendAsync(String destination, Object payload)
<T> void convertAndSend(String destination, T payload)
<T> void convertAndSend(String destination, T payload, Map<String, Object> headers)

// Receive Operations
Message<?> receive(String destination)
<T> T receiveAndConvert(String destination, Class<T> targetClass)
<T> T receiveAndConvert(String destination, ParameterizedTypeReference<T> targetType)
```

#### Example Usage:
```java
@Service
public class MessageService {
    
    @Autowired
    private QueueMessagingTemplate queueMessagingTemplate;
    
    public void sendOrder(Order order) {
        // Convert and send object
        queueMessagingTemplate.convertAndSend("order-queue", order);
        
        // Send with headers
        Map<String, Object> headers = Map.of(
            "priority", "high",
            "timestamp", System.currentTimeMillis()
        );
        queueMessagingTemplate.convertAndSend("order-queue", order, headers);
    }
    
    public Order receiveOrder() {
        return queueMessagingTemplate.receiveAndConvert("order-queue", Order.class);
    }
}
```

## 4. Annotation-Based Message Handling

### 4.1 @SqsListener

#### Overloaded Types:
```java
// Basic listener
@SqsListener("queue-name")
@SqsListener(value = "queue-name")

// With deletion policy
@SqsListener(value = "queue-name", deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)

// With multiple queues
@SqsListener({"queue1", "queue2"})

// With queue URL
@SqsListener(value = "${app.queue.url}")
```

#### Example Usage:
```java
@Component
public class OrderMessageListener {
    
    @SqsListener("order-processing-queue")
    public void handleOrderMessage(Order order) {
        // Process order
        System.out.println("Processing order: " + order.getId());
    }
    
    @SqsListener(value = "order-queue", deletionPolicy = SqsMessageDeletionPolicy.NEVER)
    public void handleOrderWithManualDeletion(Order order, Acknowledgment ack) {
        try {
            processOrder(order);
            ack.acknowledge(); // Manual acknowledgment
        } catch (Exception e) {
            // Handle error - message will remain in queue
            log.error("Failed to process order", e);
        }
    }
    
    @SqsListener("${app.high-priority-queue}")
    public void handleHighPriorityMessage(
            @Payload String message,
            @Header Map<String, String> headers,
            @Header("MessageId") String messageId) {
        
        System.out.println("Message ID: " + messageId);
        System.out.println("Headers: " + headers);
        System.out.println("Body: " + message);
    }
}
```

## 5. Configuration Classes

### 5.1 AmazonSQSAsync Configuration

```java
@Configuration
@EnableSqs
public class SQSConfig {
    
    @Bean
    @Primary
    public AmazonSQSAsync amazonSQSAsync() {
        return AmazonSQSAsyncClientBuilder.standard()
            .withRegion(Region.getRegion(Regions.US_EAST_1))
            .withCredentials(new DefaultAWSCredentialsProviderChain())
            .build();
    }
    
    @Bean
    public QueueMessagingTemplate queueMessagingTemplate(AmazonSQSAsync amazonSQSAsync) {
        return new QueueMessagingTemplate(amazonSQSAsync);
    }
}
```

## 6. Abstract Classes and Interfaces

### 6.1 AbstractAmazonSQS (Abstract Class)

```java
public abstract class AbstractAmazonSQS implements AmazonSQS {
    // Provides default implementations for some methods
    // Extend this for custom SQS implementations
}
```

### 6.2 QueueMessageHandler (Key Handler Class)

```java
@Bean
public QueueMessageHandler queueMessageHandler(AmazonSQSAsync amazonSQSAsync) {
    QueueMessageHandler messageHandler = new QueueMessageHandler();
    messageHandler.setAmazonSqs(amazonSQSAsync);
    return messageHandler;
}
```

## 7. Utility Methods and Helper Classes

### 7.1 SQS Utilities

```java
@Component
public class SQSUtilities {
    
    @Autowired
    private AmazonSQS amazonSQS;
    
    // Create queue with attributes
    public String createQueue(String queueName, Map<String, String> attributes) {
        CreateQueueRequest request = new CreateQueueRequest()
            .withQueueName(queueName)
            .withAttributes(attributes);
        return amazonSQS.createQueue(request).getQueueUrl();
    }
    
    // Get queue attributes
    public Map<String, String> getQueueAttributes(String queueUrl) {
        GetQueueAttributesRequest request = new GetQueueAttributesRequest()
            .withQueueUrl(queueUrl)
            .withAttributeNames("All");
        return amazonSQS.getQueueAttributes(request).getAttributes();
    }
    
    // Purge queue
    public void purgeQueue(String queueUrl) {
        amazonSQS.purgeQueue(new PurgeQueueRequest(queueUrl));
    }
    
    // Set queue attributes
    public void setQueueAttributes(String queueUrl, Map<String, String> attributes) {
        SetQueueAttributesRequest request = new SetQueueAttributesRequest()
            .withQueueUrl(queueUrl)
            .withAttributes(attributes);
        amazonSQS.setQueueAttributes(request);
    }
}
```

### 7.2 Message Utility Methods

```java
@Service
public class MessageUtilities {
    
    // Create message with attributes
    public SendMessageRequest createMessageRequest(String queueUrl, Object payload) {
        ObjectMapper mapper = new ObjectMapper();
        try {
            String messageBody = mapper.writeValueAsString(payload);
            return new SendMessageRequest()
                .withQueueUrl(queueUrl)
                .withMessageBody(messageBody)
                .withMessageAttributes(createDefaultAttributes());
        } catch (JsonProcessingException e) {
            throw new RuntimeException("Failed to serialize message", e);
        }
    }
    
    private Map<String, MessageAttributeValue> createDefaultAttributes() {
        return Map.of(
            "timestamp", new MessageAttributeValue()
                .withStringValue(String.valueOf(System.currentTimeMillis()))
                .withDataType("String"),
            "source", new MessageAttributeValue()
                .withStringValue("spring-boot-app")
                .withDataType("String")
        );
    }
}
```

## 8. Concept Comparisons

### 8.1 Queue Types Comparison

| Feature | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| **Throughput** | Nearly unlimited | Up to 3,000 messages/sec |
| **Ordering** | Best-effort ordering | Strict FIFO |
| **Delivery** | At-least-once | Exactly-once |
| **Naming** | Any valid name | Must end with `.fifo` |
| **Use Case** | High throughput, loose ordering | Critical ordering, deduplication |

### 8.2 Message Deletion Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **ON_SUCCESS** | Delete after successful processing | Default behavior |
| **NEVER** | Manual deletion required | Error handling, retry logic |
| **NO_REDRIVE** | Delete only if no redrive policy | Dead letter queue scenarios |

### 8.3 Visibility Timeout vs Delay Seconds

| Feature | Visibility Timeout | Delay Seconds |
|---------|-------------------|---------------|
| **Purpose** | Hide message after receipt | Delay initial delivery |
| **Scope** | Per message receive | Per message send |
| **Range** | 0 to 12 hours | 0 to 15 minutes |
| **Default** | 30 seconds | 0 seconds |

## 9. Advanced Examples

### 9.1 Batch Operations

```java
@Service
public class BatchMessageService {
    
    @Autowired
    private AmazonSQS amazonSQS;
    
    public void sendBatchMessages(String queueUrl, List<String> messages) {
        List<SendMessageBatchRequestEntry> entries = messages.stream()
            .map(msg -> new SendMessageBatchRequestEntry()
                .withId(UUID.randomUUID().toString())
                .withMessageBody(msg))
            .collect(Collectors.toList());
            
        SendMessageBatchRequest batchRequest = new SendMessageBatchRequest()
            .withQueueUrl(queueUrl)
            .withEntries(entries);
            
        SendMessageBatchResult result = amazonSQS.sendMessageBatch(batchRequest);
        
        // Handle successful and failed messages
        result.getSuccessful().forEach(success -> 
            log.info("Message sent successfully: {}", success.getId()));
        
        result.getFailed().forEach(failed -> 
            log.error("Failed to send message: {} - {}", 
                failed.getId(), failed.getMessage()));
    }
}
```

### 9.2 Error Handling and Retry

```java
@Component
public class RobustMessageListener {
    
    @SqsListener(value = "error-prone-queue", 
                deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)
    @Retryable(value = {Exception.class}, maxAttempts = 3)
    public void handleMessageWithRetry(String message) throws Exception {
        try {
            processMessage(message);
        } catch (Exception e) {
            log.error("Error processing message: {}", message, e);
            throw e; // Rethrow for retry mechanism
        }
    }
    
    @Recover
    public void recover(Exception e, String message) {
        log.error("Failed to process message after retries: {}", message, e);
        // Send to dead letter queue or alternative handling
    }
}
```

## 10. Best Practices

### 10.1 Configuration Best Practices
- Use environment-specific properties for queue names and URLs
- Configure appropriate visibility timeouts based on processing time
- Set up dead letter queues for failed messages
- Use batch operations for high throughput scenarios

### 10.2 Error Handling Best Practices
- Implement proper exception handling in message listeners
- Use manual acknowledgment for critical messages
- Set up monitoring and alerting for queue metrics
- Implement circuit breaker patterns for external service calls

### 10.3 Performance Best Practices
- Use long polling to reduce empty receives
- Optimize message size and avoid large payloads
- Implement message deduplication for FIFO queues
- Use connection pooling for high-volume applications