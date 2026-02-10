# AWS SNS

Amazon Simple Notification Service (SNS) is a fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications.

Key features include:

- Fully managed service with no infrastructure management required
- Push-based messaging model that proactively delivers messages to subscribers
- Fan-out messaging where a single message is delivered to multiple subscribers
- High availability with automatic scaling and redundancy

---

## Core Concepts

### SNS Topic

An SNS topic is a communication channel to which messages are published and from which they are delivered to subscribers.

- Acts as a logical access point and communication channel
- Enables publishers to send messages to multiple subscribers simultaneously
- Created in a specific AWS region
- Supports virtually unlimited number of subscribers per topic

| Topic Type | Throughput | Ordering | Delivery | Use Case |
|---|---|---|---|---|
| Standard | Nearly unlimited | Best-effort | At-least-once | High throughput, duplicate tolerance |
| FIFO | 300-3,000 per second | Strict FIFO | Exactly-once | Critical ordering, no duplicates |

### Publisher

Publishers are applications or services that send messages to SNS topics.

- Uses Publish API call to send messages to topics
- Can publish up to 10 messages at once using PublishBatch
- Requires proper IAM permissions to publish messages
- Can include metadata with published messages using message attributes

### Subscriber

Subscribers are endpoints that receive messages published to SNS topics.

Supported subscription types:

- HTTP/HTTPS endpoints
- Email and Email-JSON
- SMS
- SQS queues
- Lambda functions
- Mobile Push notifications

**Important:** Most subscriptions require confirmation before receiving messages, except SQS.

### Subscription

A subscription is the relationship between a topic and an endpoint that receives messages.

It defines:

- Protocol that specifies the delivery method
- Endpoint where messages are delivered
- Optional message filtering based on attributes
- Delivery policies for retry and dead letter queue settings

### Message Filtering

Message filtering allows subscribers to receive only a subset of messages published to a topic.

- Server-side filtering based on message attributes
- Uses JSON policy to define filtering criteria
- Supports exact match, numeric range, prefix matching, and more
- Reduces unnecessary message deliveries and processing costs

### Dead Letter Queue

A dead letter queue (DLQ) captures messages that cannot be delivered successfully to subscribers.

- Uses an SQS queue to receive failed message deliveries
- Isolates problematic messages for debugging and analysis
- Configured with maximum receive count before moving to DLQ
- Supported for HTTP/HTTPS, Lambda, and SQS subscriptions

### Message Attributes

Message attributes provide metadata about messages without affecting the message body.

- Key-value pairs that provide additional message metadata
- Up to 10 attributes per message
- Each attribute name and value can be up to 256 KB total
- Used for filtering, routing, and processing logic without parsing message body

| Data Type | Description |
|---|---|
| String | Text values |
| Number | Numeric values |
| Binary | Binary data |
| String.Array | Array of strings |

---

## Message Structure

### Message Anatomy

Each SNS message includes the following components:

- **Message** - The actual notification payload, up to 256 KB
- **Message ID** - Unique identifier assigned by SNS when message is published
- **Timestamp** - ISO 8601 formatted timestamp when message was published
- **Topic ARN** - Amazon Resource Name of the topic
- **Subject** - Optional subject line for the message
- **Message Attributes** - Optional metadata key-value pairs
- **Signature** - Digital signature for message authentication (HTTP/HTTPS)

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

### Message Delivery Lifecycle

SNS messages go through several stages during delivery:

1. **Published** - Message is successfully submitted to SNS topic
2. **Filtered** - Message evaluated against subscriber filter policies
3. **Delivered** - Message sent to subscriber endpoints
4. **Confirmed** - Delivery confirmation received from endpoint
5. **Retried** - Failed deliveries retried based on delivery policy
6. **Dead Lettered** - Messages moved to DLQ after retry exhaustion

---

## Subscription Protocols

### HTTP/HTTPS

- Delivers messages to web applications and REST APIs
- Requires HTTP GET confirmation via SubscribeURL
- Delivers JSON payload with message details
- HTTPS recommended for sensitive data

### Email/Email-JSON

- Sends notifications and alerts to email addresses
- Requires email confirmation link
- Supports plain text or JSON format
- Not suitable for high-volume automated processing

### SMS

- Sends mobile notifications and alerts
- No confirmation required
- Plain text message format, 160 characters recommended
- Charges apply per SMS sent

### SQS

- Provides reliable message queuing and processing
- No confirmation required
- JSON message wrapper format
- Seamless integration for fan-out patterns

### Lambda

- Triggers serverless function execution
- No confirmation required
- JSON event payload format
- Synchronous invocation of Lambda function

### Mobile Push

- Sends notifications to mobile applications
- Supports iOS (APNS), Android (FCM), Windows (WNS)
- Platform-specific JSON payload format
- Requires device token registration

---

## Message Delivery Policies

### Retry Policy

HTTP/HTTPS endpoints use a multi-phase retry approach:

- Immediate retry with no delay
- Pre-backoff phase
- Backoff phase with exponential backoff
- Post-backoff phase
- Exponential backoff with jitter to prevent thundering herd
- Configurable maximum attempts per delivery phase

### Throttling Policy

Controls delivery rate to protect subscribers:

- Rate limiting to prevent overwhelming downstream systems
- Configurable maximum receives per second
- Useful for protecting subscriber endpoints

### Dead Letter Queue Policy

Handles failed deliveries:

- Available for HTTP/HTTPS and Lambda subscriptions
- Messages moved to DLQ after all retries fail
- Enables debugging of delivery failures

---

## Security

### Topic Policies

Resource-based JSON policies that control access to specific topics.

Supported permissions:

- Publish
- Subscribe
- Receive
- AddPermission
- RemovePermission

Additional features:

- Cross-account access enablement
- Conditional access based on various factors

### IAM Policies

Identity-based policies attached to users, groups, and roles.

Common actions:

- sns:Publish
- sns:Subscribe
- sns:Unsubscribe
- sns:ListTopics
- sns:CreateTopic

Policies specify topic ARNs and wildcard patterns for resource access.

### Message Encryption

**Server-Side Encryption:**

- Encrypts messages at rest using AWS KMS
- Uses AWS managed keys or customer managed keys
- Automatic encryption/decryption for supported protocols

**In-Transit Encryption:**

- All API calls use HTTPS for encryption in transit

---

## Fan-out Patterns

### SNS to SQS Fan-out

This pattern allows a single message to be processed by multiple independent services.

**Benefits:**

- Decouples services
- Enables parallel processing
- Improves system resilience
- Simplifies architecture

**Common use cases:**

- Order processing across inventory, billing, and shipping services
- Event broadcasting to multiple microservices
- Parallel data processing pipelines

### SNS to Lambda Fan-out

Triggers multiple Lambda functions from a single event.

**Benefits:**

- Serverless event processing
- Independent function scaling
- Cost-effective for low-volume workloads

### SNS to Multiple Protocols

Delivers the same message to different endpoint types.

**Example:** Send order confirmation via email, SMS, and push notification simultaneously.

---

## Common Use Cases

### Application Alerts and Monitoring

- System health monitoring
- Error and exception notifications
- Performance threshold alerts
- Infrastructure monitoring

### User Notifications

- Account activity alerts
- Marketing campaigns
- Transactional emails
- Mobile push notifications

### Workflow Coordination

- Microservice orchestration
- Event-driven architectures
- Asynchronous processing triggers
- System integration events

### Data Processing Pipelines

- ETL job notifications
- Data ingestion events
- Batch processing coordination
- Real-time analytics triggers

---

## Best Practices

### Message Design

- Keep message payload under 256 KB
- Use message attributes for filtering and routing
- Include correlation IDs for tracking
- Design idempotent message consumers

### Topic Organization

- Create separate topics for different message types
- Use naming conventions for easy identification
- Consider FIFO topics only when ordering is critical
- Plan for cross-region requirements

### Security

- Enable server-side encryption for sensitive data
- Use IAM roles instead of access keys
- Implement least privilege access policies
- Regularly audit topic policies and subscriptions

### Monitoring and Debugging

- Enable CloudWatch metrics and alarms
- Use message attributes for tracing
- Configure dead letter queues for failed deliveries
- Implement proper error handling in subscribers

### Cost Optimization

- Use message filtering to reduce unnecessary deliveries
- Batch publish operations when possible
- Choose appropriate topic type for your use case
- Monitor and optimize subscription endpoints

---

## Spring Boot Integration

### Dependencies

Add the following dependencies to your project:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>sns</artifactId>
    <version>2.20.0</version>
</dependency>
```

### Configuration Class

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
                .region(Region.AP_SOUTH_1)
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }
}
```

### Service Implementation

```java
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.*;

import java.util.HashMap;
import java.util.Map;

@Service
public class SnsNotificationService {

    private final SnsClient snsClient;
    private final String orderEventsTopicArn = "arn:aws:sns:ap-south-1:123456789012:order-events";
    private final String userNotificationsTopicArn = "arn:aws:sns:ap-south-1:123456789012:user-notifications";
    private final String systemAlertsTopicArn = "arn:aws:sns:ap-south-1:123456789012:system-alerts";

    public SnsNotificationService(SnsClient snsClient) {
        this.snsClient = snsClient;
    }

    public String publishOrderEvent(OrderEvent orderEvent) {
        Map<String, MessageAttributeValue> attributes = new HashMap<>();
        
        attributes.put("eventType", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(orderEvent.getEventType())
                .build());
        
        attributes.put("status", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(orderEvent.getStatus())
                .build());
        
        attributes.put("priority", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(orderEvent.getPriority())
                .build());

        PublishRequest request = PublishRequest.builder()
                .topicArn(orderEventsTopicArn)
                .message(formatOrderEventMessage(orderEvent))
                .subject("Order Event: " + orderEvent.getEventType())
                .messageAttributes(attributes)
                .build();

        PublishResponse response = snsClient.publish(request);
        return response.messageId();
    }

    public String publishUserNotification(String userId, String message, NotificationType type) {
        Map<String, MessageAttributeValue> attributes = new HashMap<>();
        
        attributes.put("userId", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(userId)
                .build());
        
        attributes.put("notificationType", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(type.name())
                .build());

        PublishRequest request = PublishRequest.builder()
                .topicArn(userNotificationsTopicArn)
                .message(message)
                .subject("User Notification")
                .messageAttributes(attributes)
                .build();

        PublishResponse response = snsClient.publish(request);
        return response.messageId();
    }

    public String publishSystemAlert(String alertMessage, AlertSeverity severity, String component) {
        Map<String, MessageAttributeValue> attributes = new HashMap<>();
        
        attributes.put("severity", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(severity.name())
                .build());
        
        attributes.put("component", MessageAttributeValue.builder()
                .dataType("String")
                .stringValue(component)
                .build());

        PublishRequest request = PublishRequest.builder()
                .topicArn(systemAlertsTopicArn)
                .message(alertMessage)
                .subject("System Alert: " + severity.name())
                .messageAttributes(attributes)
                .build();

        PublishResponse response = snsClient.publish(request);
        return response.messageId();
    }

    public String subscribeToTopic(String topicArn, String protocol, String endpoint) {
        SubscribeRequest request = SubscribeRequest.builder()
                .topicArn(topicArn)
                .protocol(protocol)
                .endpoint(endpoint)
                .build();

        SubscribeResponse response = snsClient.subscribe(request);
        return response.subscriptionArn();
    }

    private String formatOrderEventMessage(OrderEvent orderEvent) {
        return String.format(
            "Order Event Details:\n" +
            "Order ID: %d\n" +
            "Event Type: %s\n" +
            "Status: %s\n" +
            "Amount: $%.2f\n" +
            "Customer ID: %s",
            orderEvent.getOrderId(),
            orderEvent.getEventType(),
            orderEvent.getStatus(),
            orderEvent.getAmount(),
            orderEvent.getCustomerId()
        );
    }
}
```

### Model Classes

```java
public class OrderEvent {
    private Long orderId;
    private String eventType;
    private String status;
    private Double amount;
    private String customerId;
    private String priority;

    public OrderEvent() {}

    public OrderEvent(Long orderId, String eventType, String status, Double amount, String customerId) {
        this.orderId = orderId;
        this.eventType = eventType;
        this.status = status;
        this.amount = amount;
        this.customerId = customerId;
    }
    
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

### REST Controller

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

### Application Configuration

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
    software.amazon.awssdk: DEBUG
```

### Fan-out Setup Example

```java
import org.springframework.stereotype.Component;
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.*;
import software.amazon.awssdk.services.sqs.SqsClient;
import software.amazon.awssdk.services.sqs.model.*;

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
        
        String inventoryQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/inventory-service";
        subscribeQueueToTopic(topicArn, inventoryQueueUrl);
        
        String billingQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/billing-service";
        subscribeQueueToTopic(topicArn, billingQueueUrl);
        
        String shippingQueueUrl = "https://sqs.ap-south-1.amazonaws.com/123456789012/shipping-service";
        subscribeQueueToTopic(topicArn, shippingQueueUrl);
    }

    private void subscribeQueueToTopic(String topicArn, String queueUrl) {
        GetQueueAttributesResponse queueAttributes = sqsClient.getQueueAttributes(
                GetQueueAttributesRequest.builder()
                        .queueUrl(queueUrl)
                        .attributeNames(QueueAttributeName.QUEUE_ARN)
                        .build());
        
        String queueArn = queueAttributes.attributes().get(QueueAttributeName.QUEUE_ARN);
        
        snsClient.subscribe(SubscribeRequest.builder()
                .topicArn(topicArn)
                .protocol("sqs")
                .endpoint(queueArn)
                .build());
        
        System.out.println("Subscribed queue " + queueArn + " to topic " + topicArn);
    }
}
```