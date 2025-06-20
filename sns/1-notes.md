# AWS SNS Core Concepts

Amazon Simple Notification Service (SNS) is a fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications. Understanding the following core concepts is essential for working with SNS effectively.

- **Fully managed service** (no infrastructure management required)
- **Push-based messaging model** (proactively delivers messages to subscribers)
- **Fan-out messaging** (single message delivered to multiple subscribers)
- **High availability** with automatic scaling and redundancy

## 1. SNS Topic

An SNS topic is a communication channel to which messages are published and from which they are delivered to subscribers.

- **Definition**: A logical access point that acts as a communication channel
- **Purpose**: Enables publishers to send messages to multiple subscribers simultaneously
- **Types**: Standard Topics and FIFO (First-In-First-Out) Topics
- **Capacity**: Virtually unlimited number of subscribers per topic
- **Regional**: Created in a specific AWS region

## 2. Standard Topic

Standard topics provide maximum throughput, best-effort ordering, and at-least-once delivery.

- **Definition**: Default SNS topic type optimizing for high throughput
- **Throughput**: Nearly unlimited messages per second
- **Ordering**: Best-effort ordering (messages may arrive out of order)
- **Delivery**: At-least-once delivery (messages may be delivered more than once)
- **Use Case**: Ideal for applications that can handle duplicate messages and don't require strict ordering

## 3. FIFO Topic

FIFO topics are designed to guarantee that messages are delivered exactly once, in the exact order they are published.

- **Definition**: Topic type that preserves message order and prevents duplicates
- **Throughput**: Up to 300 messages per second (can be increased to 3,000 with batching)
- **Ordering**: Strict first-in-first-out message ordering
- **Delivery**: Exactly-once delivery (no duplicates)
- **Naming**: Topic names must end with `.fifo` suffix
- **Use Case**: Critical for applications requiring strict ordering and no duplicates

## 4. Publisher

Publishers are applications or services that send messages to SNS topics.

- **Definition**: Any application that publishes messages to an SNS topic
- **Method**: Uses `Publish` API call to send messages to topics
- **Batching**: Can publish up to 10 messages at once using `PublishBatch`
- **Authentication**: Requires proper IAM permissions to publish messages
- **Message Attributes**: Can include metadata with published messages

## 5. Subscriber

Subscribers are endpoints that receive messages published to SNS topics.

- **Definition**: Endpoints that receive notifications from SNS topics
- **Types**: HTTP/HTTPS, Email, SMS, SQS, Lambda, Mobile Push
- **Subscription**: Must be confirmed before receiving messages (except SQS)
- **Filtering**: Can apply message filtering based on attributes
- **Delivery Policies**: Configurable retry and dead letter queue settings

## 6. Subscription

A subscription is the relationship between a topic and an endpoint that receives messages.

- **Definition**: Configuration that defines how messages are delivered to endpoints
- **Protocol**: Specifies the delivery method (HTTP, Email, SMS, etc.)
- **Endpoint**: The destination where messages are delivered
- **Confirmation**: Most subscriptions require confirmation before activation
- **Filtering**: Optional message filtering based on attributes

## 7. Message Filtering

Message filtering allows subscribers to receive only a subset of messages published to a topic.

- **Definition**: Server-side filtering based on message attributes
- **Filter Policy**: JSON policy that defines filtering criteria
- **Attributes**: Messages must include attributes for filtering to work
- **Operators**: Supports exact match, numeric range, prefix matching, and more
- **Cost Savings**: Reduces unnecessary message deliveries and processing

## 8. Dead Letter Queue (DLQ)

A dead letter queue captures messages that cannot be delivered successfully to subscribers.

- **Definition**: SQS queue that receives failed message deliveries
- **Purpose**: Isolates problematic messages for debugging and analysis
- **Configuration**: Set maximum receive count before moving to DLQ
- **Supported Protocols**: HTTP/HTTPS, Lambda, SQS subscriptions
- **Retention**: Messages in DLQ follow SQS retention policies

## 9. Message Attributes

Message attributes provide metadata about messages without affecting the message body.

- **Definition**: Key-value pairs that provide additional message metadata
- **Types**: String, Number, Binary, String.Array data types
- **Limit**: Up to 10 attributes per message
- **Size**: Each attribute name and value can be up to 256 KB total
- **Use Case**: Filtering, routing, and processing logic without parsing message body

---

## SNS Message Anatomy
Each SNS message includes the following components:

* **Message** – The actual notification payload (up to 256 KB)
* **Message ID** – Unique identifier assigned by SNS when message is published
* **Timestamp** – ISO 8601 formatted timestamp when message was published
* **Topic ARN** – Amazon Resource Name of the topic
* **Subject** – Optional subject line for the message
* **Message Attributes** (optional) – Metadata key-value pairs
* **Signature** – Digital signature for message authentication (HTTP/HTTPS)

---

## Message Delivery Lifecycle
SNS messages go through several stages during delivery:

* **Published** – Message is successfully submitted to SNS topic
* **Filtered** – Message evaluated against subscriber filter policies
* **Delivered** – Message sent to subscriber endpoints
* **Confirmed** – Delivery confirmation received from endpoint
* **Retried** – Failed deliveries retried based on delivery policy
* **Dead Lettered** – Messages moved to DLQ after retry exhaustion

---

## Subscription Protocols

### HTTP/HTTPS
* **Use Case**: Web applications and REST APIs
* **Confirmation**: Requires HTTP GET confirmation via SubscribeURL
* **Format**: JSON payload with message details
* **Security**: HTTPS recommended for sensitive data

### Email/Email-JSON
* **Use Case**: Human notifications and alerts
* **Confirmation**: Email confirmation link required
* **Format**: Plain text or JSON format
* **Limitations**: Not suitable for high-volume automated processing

### SMS
* **Use Case**: Mobile notifications and alerts
* **Confirmation**: No confirmation required
* **Format**: Plain text message (160 characters recommended)
* **Costs**: Charges apply per SMS sent

### SQS
* **Use Case**: Reliable message queuing and processing
* **Confirmation**: No confirmation required
* **Format**: JSON message wrapper
* **Integration**: Seamless integration with SQS for fan-out patterns

### Lambda
* **Use Case**: Serverless function triggers
* **Confirmation**: No confirmation required
* **Format**: JSON event payload
* **Execution**: Synchronous invocation of Lambda function

### Mobile Push
* **Use Case**: Mobile app notifications
* **Platforms**: iOS (APNS), Android (FCM), Windows (WNS)
* **Format**: Platform-specific JSON payload
* **Registration**: Requires device token registration

---

## Message Delivery Policies

### Retry Policy
* **HTTP/HTTPS**: Immediate retry, then exponential backoff
* **Retry Phases**: Immediate (no delay), pre-backoff, backoff, post-backoff
* **Maximum Attempts**: Configurable per delivery phase
* **Backoff**: Exponential backoff with jitter to prevent thundering herd

### Throttling Policy
* **Rate Limiting**: Controls delivery rate to protect subscribers
* **Maximum Receives Per Second**: Configurable throttling limit
* **Use Case**: Prevent overwhelming downstream systems

### Dead Letter Queue Policy
* **Failed Deliveries**: Configure DLQ for HTTP/HTTPS and Lambda
* **Retry Exhaustion**: Messages moved to DLQ after all retries fail
* **Analysis**: Enable debugging of delivery failures

---

## Topic Access Control and Security

### Topic Policies
* **Resource-based**: JSON policies that control access to specific topics
* **Permissions**: Publish, Subscribe, Receive, AddPermission, RemovePermission
* **Cross-account**: Enable access from other AWS accounts
* **Conditions**: Support for conditional access based on various factors

### IAM Policies
* **Identity-based**: Policies attached to users, groups, and roles
* **Actions**: Include `sns:Publish`, `sns:Subscribe`, `sns:Unsubscribe`, etc.
* **Resources**: Specify topic ARNs and wildcard patterns
* **Integration**: Works with other AWS services for fine-grained control

### Message Encryption
* **Server-Side Encryption**: Encrypts messages at rest using AWS KMS
* **In-Transit**: All API calls use HTTPS for encryption in transit
* **Key Management**: Uses AWS managed keys or customer managed keys
* **Integration**: Automatic encryption/decryption for supported protocols

---

## Fan-out Patterns

### SNS to SQS
* **Pattern**: Single SNS message delivered to multiple SQS queues
* **Benefits**: Decoupled processing, reliable delivery, independent scaling
* **Use Case**: Order processing with inventory, billing, and shipping queues
* **Configuration**: Create SQS subscriptions to SNS topic

### SNS to Lambda
* **Pattern**: Trigger multiple Lambda functions from single message
* **Benefits**: Serverless processing, automatic scaling, cost-effective
* **Use Case**: Image processing with thumbnail, metadata, and storage functions
* **Invocation**: Synchronous Lambda invocation

### SNS to HTTP Endpoints
* **Pattern**: Deliver messages to multiple web services
* **Benefits**: Integration with external systems and microservices
* **Use Case**: Webhook notifications to multiple partner systems
* **Security**: Support for HTTPS and message signature verification

---

## Message Deduplication and Ordering (FIFO Topics)

### Message Group ID
* **Purpose**: Groups related messages together for ordered processing
* **Requirement**: Required for FIFO topics
* **Behavior**: Messages with the same Group ID are delivered in order
* **Parallelism**: Different Group IDs can be processed in parallel

### Deduplication ID
* **Purpose**: Prevents duplicate messages within 5-minute deduplication interval
* **Generation**: Can be provided explicitly or generated from message body hash
* **Scope**: Applies within the entire FIFO topic
* **Benefit**: Ensures exactly-once message delivery

### Content-based Deduplication
* **Automatic**: SNS generates deduplication ID from message body
* **Enable**: Set ContentBasedDeduplication attribute to true
* **Hash**: SHA-256 hash of message body used as deduplication ID
* **Convenience**: Eliminates need to provide explicit deduplication ID

---

## Monitoring and Observability

### CloudWatch Metrics
* **Publish Metrics**: NumberOfMessagesPublished, PublishSize
* **Delivery Metrics**: NumberOfNotificationsDelivered, NumberOfNotificationsFailed
* **Subscription Metrics**: Per-protocol delivery statistics
* **Custom Metrics**: Application-specific metrics via CloudWatch API

### CloudWatch Alarms
* **Failed Deliveries**: Alert on high failure rates
* **DLQ Messages**: Monitor messages in dead letter queues
* **Subscription Health**: Track subscription confirmation status
* **Cost Monitoring**: Alert on unexpected SNS charges

### AWS X-Ray Integration
* **Tracing**: End-to-end tracing of message flows
* **Performance**: Identify bottlenecks in message delivery
* **Dependencies**: Visualize service dependencies and interactions
* **Debugging**: Correlate failures across distributed systems

---

## Best Practices and Patterns

### Topic Design
* **Single Responsibility**: One topic per event type or business domain
* **Naming Convention**: Use consistent, descriptive topic names
* **Regional Strategy**: Consider cross-region replication for disaster recovery
* **Lifecycle Management**: Plan for topic creation, updates, and deletion

### Message Design
* **Size Optimization**: Keep messages small for better performance
* **JSON Format**: Use structured JSON for complex data
* **Attributes**: Use message attributes for filtering and routing
* **Idempotency**: Design subscribers to handle duplicate messages

### Subscriber Management
* **Health Checks**: Monitor subscriber endpoint health
* **Graceful Degradation**: Handle subscriber failures gracefully
* **Rate Limiting**: Implement subscriber-side rate limiting
* **Error Handling**: Robust error handling and retry logic

### Cost Optimization
* **Message Filtering**: Reduce unnecessary deliveries with filtering
* **Batch Publishing**: Use batch operations when possible
* **Right-size Throughput**: Choose appropriate topic type for workload
* **Monitor Usage**: Track costs and optimize based on usage patterns

<br/><br/>

---
Here's a clear breakdown with **realistic examples** showing what **SNS operations look like** — both **programmatically** and via **AWS SDK** (Java/Spring Boot).

---

## 📦 **SNS Message Operations**

### ✅ **Message Structure Example**

| Component                    | Description                 | Example                                |
| ---------------------------- | --------------------------- | -------------------------------------- |
| **Topic ARN**                | Topic identifier            | `arn:aws:sns:ap-south-1:123456789012:order-events` |
| **Message**                  | Notification payload        | `{"orderId": 12345, "status": "confirmed"}` |
| **Subject**                  | Message subject             | `Order Confirmation`                   |
| **Message ID**               | Unique message identifier   | `e2f76847-3d0e-42c9-a2dc-5b98e8b6d403` |
| **Timestamp**                | Publication time            | `2024-01-15T10:30:45.123Z`            |
| **Message Attributes**       | Metadata                    | `eventType=ORDER_CONFIRMED`           |

---

## 🔍 **Sample SNS Message (JSON format)**

```json
{
  "Type": "Notification",
  "MessageId": "e2f76847-3d0e-42c9-a2dc-5b98e8b6d403",
  "TopicArn": "arn:aws:sns:ap-south-1:123456789012:order-events",
  "Subject": "Order Confirmation",
  "Message": "{\"orderId\":12345,\"customerId\":\"CUST001\",\"status\":\"confirmed\",\"amount\":5000,\"currency\":\"INR\"}",
  "Timestamp": "2024-01-15T10:30:45.123Z",
  "SignatureVersion": "1",
  "Signature": "Base64EncodedSignature==",
  "SigningCertURL": "https://sns.ap-south-1.amazonaws.com/SimpleNotificationService.pem",
  "MessageAttributes": {
    "eventType": {
      "Type": "String",
      "Value": "ORDER_CONFIRMED"
    },
    "priority": {
      "Type": "String",
      "Value": "HIGH"
    },
    "department": {
      "Type": "String",
      "Value": "sales"
    }
  }
}
```

---

## 🧪 **Spring Boot SNS Service Example**

```java
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.HashMap;

@Service
public class SnsNotificationService {

    private final SnsClient snsClient;
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    // Topic ARNs - typically configured via properties
    private final String orderEventsTopicArn = "arn:aws:sns:ap-south-1:123456789012:order-events";
    private final String userNotificationsTopicArn = "arn:aws:sns:ap-south-1:123456789012:user-notifications";
    private final String alertsTopicArn = "arn:aws:sns:ap-south-1:123456789012:system-alerts";

    public SnsNotificationService(SnsClient snsClient) {
        this.snsClient = snsClient;
    }

    // Publish order event with attributes
    public String publishOrderEvent(OrderEvent orderEvent) {
        try {
            String message = objectMapper.writeValueAsString(orderEvent);
            
            Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
            messageAttributes.put("eventType", MessageAttributeValue.builder()
                    .dataType("String")
                    .stringValue(orderEvent.getEventType())
                    .build());
            messageAttributes.put("orderId", MessageAttributeValue.builder()
                    .dataType("Number")
                    .stringValue(String.valueOf(orderEvent.getOrderId()))
                    .build());
            messageAttributes.put("priority", MessageAttributeValue.builder()
                    .dataType("String")
                    .stringValue(orderEvent.getPriority())
                    .build());

            PublishRequest publishRequest = PublishRequest.builder()
                    .topicArn(orderEventsTopicArn)
                    .message(message)
                    .subject("Order Event - " + orderEvent.getEventType())
                    .messageAttributes(messageAttributes)
                    .build();

            PublishResponse response = snsClient.publish(publishRequest);
            
            System.out.println("Order event published: " + response.messageId());
            return response.messageId();

        } catch (Exception e) {
            throw new RuntimeException("Failed to publish order event", e);
        }
    }

    // Publish user notification (Email/SMS)
    public String publishUserNotification(String userId, String message, NotificationType type) {
        Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
        messageAttributes.put("userId", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(userId)
                .build());
        messageAttributes.put("notificationType", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(type.name())
                .build());

        PublishRequest publishRequest = PublishRequest.builder()
                .topicArn(userNotificationsTopicArn)
                .message(message)
                .subject("User Notification")
                .messageAttributes(messageAttributes)
                .build();

        PublishResponse response = snsClient.publish(publishRequest);
        
        System.out.println("User notification published: " + response.messageId());
        return response.messageId();
    }

    // Publish system alert
    public String publishSystemAlert(String alertMessage, AlertSeverity severity, String component) {
        Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
        messageAttributes.put("severity", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(severity.name())
                .build());
        messageAttributes.put("component", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(component)
                .build());
        messageAttributes.put("timestamp", MessageAttributeValue.builder()
                .dataType("Number")
                .stringValue(String.valueOf(System.currentTimeMillis()))
                .build());

        PublishRequest publishRequest = PublishRequest.builder()
                .topicArn(alertsTopicArn)
                .message(alertMessage)
                .subject("System Alert - " + severity.name())
                .messageAttributes(messageAttributes)
                .build();

        PublishResponse response = snsClient.publish(publishRequest);
        
        System.out.println("System alert published: " + response.messageId());
        return response.messageId();
    }

    // Batch publish messages (for FIFO topics)
    public List<String> publishBatchMessages(List<OrderEvent> events, String messageGroupId) {
        List<PublishBatchRequestEntry> entries = new ArrayList<>();
        
        for (int i = 0; i < events.size() && i < 10; i++) { // Max 10 messages per batch
            OrderEvent event = events.get(i);
            try {
                String message = objectMapper.writeValueAsString(event);
                
                Map<String, MessageAttributeValue> messageAttributes = new HashMap<>();
                messageAttributes.put("eventType", MessageAttributeValue.builder()
                        .dataType("String")
                        .stringValue(event.getEventType())
                        .build());

                PublishBatchRequestEntry entry = PublishBatchRequestEntry.builder()
                        .id(String.valueOf(i))
                        .message(message)
                        .subject("Batch Order Event")
                        .messageAttributes(messageAttributes)
                        .messageGroupId(messageGroupId) // Required for FIFO
                        .messageDeduplicationId(event.getOrderId() + "-" + event.getEventType()) // For deduplication
                        .build();
                
                entries.add(entry);
            } catch (Exception e) {
                System.err.println("Failed to create batch entry for event: " + event.getOrderId());
            }
        }

        PublishBatchRequest batchRequest = PublishBatchRequest.builder()
                .topicArn(orderEventsTopicArn + ".fifo") // FIFO topic
                .publishRequestEntries(entries)
                .build();

        PublishBatchResponse response = snsClient.publishBatch(batchRequest);
        
        return response.successful().stream()
                .map(PublishBatchResultEntry::messageId)
                .collect(Collectors.toList());
    }

    // Create subscription programmatically
    public String subscribeToTopic(String topicArn, String protocol, String endpoint) {
        SubscribeRequest subscribeRequest = SubscribeRequest.builder()
                .topicArn(topicArn)
                .protocol(protocol) // "sqs", "email", "http", "https", "lambda"
                .endpoint(endpoint)
                .build();

        SubscribeResponse response = snsClient.subscribe(subscribeRequest);
        
        System.out.println("Subscription created: " + response.subscriptionArn());
        return response.subscriptionArn();
    }

    // Create topic with attributes
    public String createTopic(String topicName, boolean isFifo) {
        CreateTopicRequest.Builder requestBuilder = CreateTopicRequest.builder()
                .name(isFifo ? topicName + ".fifo" : topicName);

        if (isFifo) {
            Map<String, String> attributes = new HashMap<>();
            attributes.put("FifoTopic", "true");
            attributes.put("ContentBasedDeduplication", "false"); // Manual deduplication
            requestBuilder.attributes(attributes);
        }

        CreateTopicResponse response = snsClient.createTopic(requestBuilder.build());
        
        System.out.println("Topic created: " + response.topicArn());
        return response.topicArn();
    }
}

// Supporting classes
class OrderEvent {
    private Long orderId;
    private String eventType;
    private String status;
    private Double amount;
    private String customerId;
    private String priority = "NORMAL";
    
    // Constructors, getters, setters
    public OrderEvent() {}
    
    public OrderEvent(Long orderId, String eventType, String status, Double amount, String customerId) {
        this.orderId = orderId;
        this.eventType = eventType;
        this.status = status;
        this.amount = amount;
        this.customerId = customerId;
    }
    
    // Getters and setters
    public Long getOrderId() { return orderId; }
    public void setOrderId(Long orderId) { this.orderId = orderId; }
    
    public String getEventType() { return eventType; }
    public void setEventType(String eventType) { this.eventType = eventType; }
    
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    
    public Double getAmount() { return amount; }
    public void setAmount(Double amount) { this.amount = amount; }
    
    public String getCustomerId() { return customerId; }
    public void setCustomerId(String customerId) { this.customerId = customerId; }
    
    public String getPriority() { return priority; }
    public void setPriority(String priority) { this.priority = priority; }
}

enum NotificationType {
    EMAIL, SMS, PUSH
}

enum AlertSeverity {
    LOW, MEDIUM, HIGH, CRITICAL
}
```

---

## 🔐 **REST Controller Example**

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/notifications")
public class NotificationController {

    private final SnsNotificationService snsService;

    public NotificationController(SnsNotificationService snsService) {
        this.snsService = snsService;
    }

    @PostMapping("/order-events")
    public ResponseEntity<Map<String, String>> publishOrderEvent(@RequestBody OrderEvent orderEvent) {
        String messageId = snsService.publishOrderEvent(orderEvent);
        
        return ResponseEntity.ok(Map.of(
            "message", "Order event published successfully",
            "messageId", messageId,
            "eventType", orderEvent.getEventType()
        ));
    }

    @PostMapping("/user-notifications")
    public ResponseEntity<Map<String, String>> publishUserNotification(
            @RequestParam String userId,
            @RequestParam String message,
            @RequestParam NotificationType type) {
        
        String messageId = snsService.publishUserNotification(userId, message, type);
        
        return ResponseEntity.ok(Map.of(
            "message", "User notification published successfully",
            "messageId", messageId,
            "userId", userId
        ));
    }

    @PostMapping("/system-alerts")
    public ResponseEntity<Map<String, String>> publishSystemAlert(
            @RequestParam String alertMessage,
            @RequestParam AlertSeverity severity,
            @RequestParam String component) {
        
        String messageId = snsService.publishSystemAlert(alertMessage, severity, component);
        
        return ResponseEntity.ok(Map.of(
            "message", "System alert published successfully",
            "messageId", messageId,
            "severity", severity.name()
        ));
    }

    @PostMapping("/subscriptions")
    public ResponseEntity<Map<String, String>> createSubscription(
            @RequestParam String topicArn,
            @RequestParam String protocol,
            @RequestParam String endpoint) {
        
        String subscriptionArn = snsService.subscribeToTopic(topicArn, protocol, endpoint);
        
        return ResponseEntity.ok(Map.of(
            "message", "Subscription created successfully",
            "subscriptionArn", subscriptionArn,
            "protocol", protocol
        ));
    }
}
```

---

## ⚙️ **Configuration Example**

```java
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.sns.SnsClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SnsConfig {

    @Bean
    public SnsClient snsClient() {
        return SnsClient.builder()
                .region(Region.AP_SOUTH_1) // Mumbai region
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }
}
```

---

## 📋 **Sample application.yml**

```yaml
aws:
  sns:
    region: ap-south-1
    topics:
      order-events: arn:aws:sns:ap-south-1:123456789012:order-events
      user-notifications: arn:aws:sns:ap-south-1:123456789012:user-notifications
      system-alerts: arn:aws:sns:ap-south-1:123456789012:system-alerts
  credentials:
    access-key: ${AWS_ACCESS_KEY_ID}
    secret-key: ${AWS_SECRET_ACCESS_KEY}

logging:
  level:
    software.amazon.awssdk: DEBUG # For AWS SDK debugging
```

---

## 🎯 **SNS to SQS Fan-out Example**

```java
// Example of setting up SNS to SQS fan-out pattern
@Component
public class FanOutSetup {

    private final SnsClient snsClient;
    private final SqsClient sqsClient;

    public FanOutSetup(SnsClient snsClient, SqsClient sqsClient) {
        this.snsClient = snsClient;
        this.sqsClient = sqsClient;
    }

    public void setupOrderProcessingFanOut() {
        String topicArn = "arn:aws:sns:ap-south-1:123456789012:order-events";
        
        // Subscribe inventory service queue
        String inventoryQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/inventory-service";
        subscribeQueueToTopic(topicArn, inventoryQueueUrl);
        
        // Subscribe billing service queue
        String billingQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/billing-service";
        subscribeQueueToTopic(topicArn, billingQueueUrl);
        
        // Subscribe shipping service queue
        String shippingQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/shipping-service";
        subscribeQueueToTopic(topicArn, shippingQueueUrl);
    }

    private void subscribeQueueToTopic(String topicArn, String queueUrl) {
        // Get queue ARN
        GetQueueAttributesResponse queueAttributes = sqsClient.getQueueAttributes(
                GetQueueAttributesRequest.builder()
                        .queueUrl(queueUrl)
                        .attributeNames(QueueAttributeName.QUEUE_ARN)
                        .build());
        
        String queueArn = queueAttributes.attributes().get(QueueAttributeName.QUEUE_ARN);
        
        // Subscribe queue to topic
        snsClient.subscribe(SubscribeRequest.builder()
                .topicArn(topicArn)
                .protocol("sqs")
                .endpoint(queueArn)
                .build());
        
        System.out.println("Subscribed queue " + queueArn + " to topic " + topicArn);
    }
}
```

---