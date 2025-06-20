# AWS SQS Core Concepts

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications. Understanding the following core concepts is essential for working with SQS effectively.

- **Fully managed service** (no infrastructure management required)
- **Pull-based messaging model** (consumers poll for messages)
- **Decouples components** in distributed architectures
- **Reliable message delivery** with guaranteed at-least-once delivery

## 1. SQS Queue

An SQS queue is a temporary repository for messages that are waiting to be processed.

- **Definition**: A buffer that stores messages between sending and receiving applications
- **Purpose**: Decouples producers from consumers, allowing independent scaling
- **Types**: Standard Queues and FIFO (First-In-First-Out) Queues
- **Capacity**: Virtually unlimited number of messages
- **Retention**: Messages can be retained for up to 14 days (default: 4 days)

## 2. Standard Queue

Standard queues provide maximum throughput, best-effort ordering, and at-least-once delivery.

- **Definition**: Default SQS queue type optimizing for high throughput
- **Throughput**: Nearly unlimited transactions per second
- **Ordering**: Best-effort ordering (messages may arrive out of order)
- **Delivery**: At-least-once delivery (messages may be delivered more than once)
- **Use Case**: Ideal for applications that can handle duplicate messages and don't require strict ordering

## 3. FIFO Queue

FIFO queues are designed to guarantee that messages are processed exactly once, in the exact order they are sent.

- **Definition**: Queue type that preserves message order and prevents duplicates
- **Throughput**: Up to 300 transactions per second (can be increased to 3,000 with batching)
- **Ordering**: Strict first-in-first-out message ordering
- **Delivery**: Exactly-once processing (no duplicates)
- **Naming**: Queue names must end with `.fifo` suffix
- **Use Case**: Critical for applications requiring strict ordering and no duplicates

## 4. Message Producer

Producers are applications or services that send messages to SQS queues.

- **Definition**: Any application that sends messages to an SQS queue
- **Method**: Uses `SendMessage` API call to add messages to queues
- **Batching**: Can send up to 10 messages at once using `SendMessageBatch`
- **Authentication**: Requires proper IAM permissions to send messages

## 5. Message Consumer

Consumers are applications that retrieve and process messages from SQS queues.

- **Definition**: Applications that poll queues and process messages
- **Method**: Uses `ReceiveMessage` API call to retrieve messages
- **Polling Types**: Short polling (immediate return) and Long polling (waits up to 20 seconds)
- **Processing**: Must explicitly delete messages after successful processing
- **Concurrency**: Multiple consumers can process messages from the same queue simultaneously

## 6. Message Visibility Timeout

Visibility timeout is the period during which a message is hidden from other consumers after being retrieved.

- **Definition**: Time period when a message is invisible to other consumers
- **Purpose**: Prevents multiple consumers from processing the same message simultaneously
- **Default**: 30 seconds (configurable from 0 seconds to 12 hours)
- **Behavior**: Message becomes visible again if not deleted within timeout period
- **Extension**: Can be extended using `ChangeMessageVisibility` API

## 7. Dead Letter Queue (DLQ)

A dead letter queue is a special queue that receives messages that cannot be processed successfully.

- **Definition**: Queue for messages that have failed processing multiple times
- **Purpose**: Isolates problematic messages for debugging and analysis
- **Configuration**: Set maximum receive count (1-1000) before moving to DLQ
- **Requirements**: Must be the same type (Standard or FIFO) as the source queue
- **Retention**: Messages in DLQ can be retained for up to 14 days

## 8. Message Attributes

Message attributes provide metadata about messages without affecting the message body.

- **Definition**: Key-value pairs that provide additional message metadata
- **Types**: String, Number, Binary data types
- **Limit**: Up to 10 attributes per message
- **Size**: Each attribute name and value can be up to 256 KB
- **Use Case**: Filtering, routing, and processing logic without parsing message body

---

## SQS Message Anatomy
Each SQS message includes the following components:

* **Message Body** – The actual data payload (up to 256 KB)
* **Message ID** – Unique identifier assigned by SQS when message is sent
* **Receipt Handle** – Unique identifier for each time a message is received (required for deletion)
* **MD5 Hash** – Hash of the message body for integrity verification
* **Message Attributes** (optional) – Metadata key-value pairs
* **Timestamps** – Sent timestamp and approximate first receive timestamp

---

## Message Lifecycle States
SQS messages go through several states during their lifecycle:

* **Sent** – Message is successfully stored in the queue
* **Received** – Message is retrieved by a consumer (becomes invisible to others)
* **Processing** – Consumer is working on the message (still invisible)
* **Deleted** – Message is successfully processed and removed from queue
* **Returned to Queue** – Message becomes visible again if not deleted within visibility timeout

---

## Polling Mechanisms

### Short Polling
* **Behavior**: Returns immediately, even if no messages are available
* **Cost**: Charges for each request, regardless of message availability
* **Use Case**: When you need immediate response or have predictable message flow

### Long Polling
* **Behavior**: Waits up to 20 seconds for messages to become available
* **Cost**: More cost-effective as it reduces empty responses
* **Configuration**: Set `WaitTimeSeconds` parameter (1-20 seconds)
* **Benefit**: Reduces the number of API calls and improves message delivery efficiency

---

## Message Ordering and Deduplication (FIFO Queues)

### Message Group ID
* **Purpose**: Groups related messages together for ordered processing
* **Requirement**: Required for FIFO queues
* **Behavior**: Messages with the same Group ID are processed in order
* **Parallelism**: Different Group IDs can be processed in parallel

### Deduplication ID
* **Purpose**: Prevents duplicate messages within 5-minute deduplication interval
* **Generation**: Can be provided explicitly or generated from message body hash
* **Scope**: Applies within the same Message Group ID
* **Benefit**: Ensures exactly-once message processing

---

## Access Control and Security

### IAM Policies
* **Resource-based**: Queue policies that control access to specific queues
* **Identity-based**: IAM user/role policies that grant SQS permissions
* **Actions**: Include `sqs:SendMessage`, `sqs:ReceiveMessage`, `sqs:DeleteMessage`, etc.

### Encryption
* **Server-Side Encryption (SSE)**: Encrypts messages at rest using AWS KMS
* **In-Transit**: All API calls use HTTPS for encryption in transit
* **Key Management**: Uses AWS managed keys or customer managed keys

---

## Best Practices and Patterns

### Batch Processing
* **Send Batching**: Use `SendMessageBatch` to send up to 10 messages at once
* **Receive Batching**: Use `MaxNumberOfMessages` parameter (up to 10) to receive multiple messages
* **Benefit**: Reduces API calls and improves throughput

### Error Handling
* **Retry Logic**: Implement exponential backoff for failed operations
* **Dead Letter Queues**: Configure DLQ for messages that fail repeatedly
* **Alarming**: Set up CloudWatch alarms for queue depth and DLQ messages

### Scaling Considerations
* **Consumer Scaling**: Scale consumers based on queue depth and processing time
* **Queue Separation**: Use separate queues for different message types or priorities
* **Regional Distribution**: Consider cross-region replication for disaster recovery

<br/><br/>

---
Here's a clear breakdown with **realistic examples** showing what an AWS **SQS message looks like** — both **logically** and in **JSON format** (as seen via AWS SDKs, console, or `ReceiveMessage` API).

---

## 📦 **SQS Message Anatomy**

### ✅ **Components Explained with Example**

| Component                    | Description                 | Example                                |
| ---------------------------- | --------------------------- | -------------------------------------- |
| **Message Body**             | The main payload            | `"OrderID: 12345, Amount: ₹5000"`      |
| **Message ID**               | Unique ID assigned on send  | `e2f76847-3d0e-42c9-a2dc-5b98e8b6d403` |
| **Receipt Handle**           | Token to delete the message | `AQEBzZzO...shF+A==`                   |
| **MD5 of Body**              | Integrity check hash        | `7b270e59b47ff90a553787216d55d91d`     |
| **Message Attributes**       | Optional metadata           | `ContentType=application/json`         |
| **Sent Timestamp**           | Epoch time in ms            | `1721011200000`                        |
| **First Received Timestamp** | Approximate first delivery  | `1721011220000`                        |

---

## 🔍 **Sample Message (JSON-like format)**

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

## 🧪 Example Use in Spring Boot Listener

```java
@SqsListener("order-queue")
public void processOrder(String messageBody,
                         @Header("SenderId") String senderId,
                         @Header("ContentType") String contentType) {
    System.out.println("Received order: " + messageBody);
    System.out.println("Sender: " + senderId + " - ContentType: " + contentType);
}
```

---

## 🔐 Example with Attributes in SendMessage (Java SDK)

```java
import software.amazon.awssdk.services.sqs.SqsClient;
import software.amazon.awssdk.services.sqs.model.*;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.stereotype.Service;

import java.util.Map;

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