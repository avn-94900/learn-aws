# 02 — Spring Boot Setup & Configuration

---

## Dependencies

Add to your `pom.xml`:

```xml
<!-- Spring Cloud AWS (for @SqsListener, QueueMessagingTemplate) -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws-messaging</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>

<!-- AWS Java SDK v1 (used by Spring Cloud AWS 2.x) -->
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-sqs</artifactId>
    <version>1.12.261</version>
</dependency>
```

> **Note**: For AWS SDK v2 standalone (without Spring Cloud AWS), use:
> ```xml
> <dependency>
>     <groupId>software.amazon.awssdk</groupId>
>     <artifactId>sqs</artifactId>
> </dependency>
> ```

---

## Application Configuration

```yaml
# application.yml
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
    url: https://sqs.us-east-1.amazonaws.com/123456789012/my-queue
    dlq:
      name: my-queue-dlq
```

---

## Core Configuration Beans

```java
@Configuration
@EnableSqs
public class SQSConfig {

    /**
     * AmazonSQSAsync is required by Spring Cloud AWS for @SqsListener.
     * It supports both synchronous and asynchronous operations.
     */
    @Bean
    @Primary
    public AmazonSQSAsync amazonSQSAsync() {
        return AmazonSQSAsyncClientBuilder.standard()
            .withRegion(Regions.US_EAST_1)
            .withCredentials(new DefaultAWSCredentialsProviderChain())
            .build();
    }

    /**
     * QueueMessagingTemplate — used for SENDING messages programmatically.
     * Think of it as JdbcTemplate but for SQS.
     */
    @Bean
    public QueueMessagingTemplate queueMessagingTemplate(AmazonSQSAsync amazonSQSAsync) {
        return new QueueMessagingTemplate(amazonSQSAsync);
    }

    /**
     * SimpleMessageListenerContainerFactory — configures the background
     * listener container used by @SqsListener for RECEIVING messages.
     */
    @Bean
    public SimpleMessageListenerContainerFactory simpleMessageListenerContainerFactory(
            AmazonSQSAsync amazonSQSAsync) {

        SimpleMessageListenerContainerFactory factory =
            new SimpleMessageListenerContainerFactory();
        factory.setAmazonSqs(amazonSQSAsync);
        factory.setMaxNumberOfMessages(10);   // batch size per poll
        factory.setWaitTimeOut(20);           // long polling (seconds)
        return factory;
    }
}
```

---

## `QueueMessagingTemplate` vs `SimpleMessageListenerContainerFactory`

| Aspect | `QueueMessagingTemplate` | `SimpleMessageListenerContainerFactory` |
|---|---|---|
| **Direction** | Outbound (Send) | Inbound (Receive/Listen) |
| **Style** | Pull-based / manual | Push-based / automatic |
| **Usage** | `convertAndSend(...)`, `receiveAndConvert(...)` | Powers `@SqsListener` methods |
| **Threading** | No built-in concurrency | Configurable background threads |
| **Error Handling** | Manual try/catch | Centralized via `setErrorHandler` |
| **Batch / Long Polling** | Not configurable | Configurable via factory |
| **Use When** | Sending messages or on-demand receive | Continuous event-driven consumption |

---

## AWS SDK v2 Configuration (Standalone)

If you use AWS SDK v2 directly (without Spring Cloud AWS):

```java
@Configuration
public class SdkV2SQSConfig {

    @Bean
    public SqsClient sqsClient() {
        return SqsClient.builder()
            .region(Region.US_EAST_1)
            .credentialsProvider(DefaultCredentialsProvider.create())
            .build();
    }

    @Bean
    public SqsAsyncClient sqsAsyncClient() {
        return SqsAsyncClient.builder()
            .region(Region.US_EAST_1)
            .credentialsProvider(DefaultCredentialsProvider.create())
            .build();
    }
}
```

---

## Local Development with LocalStack

For testing SQS locally without an AWS account:

```yaml
# application-local.yml
cloud:
  aws:
    sqs:
      endpoint: http://localhost:4566
    region:
      static: us-east-1
    credentials:
      access-key: test
      secret-key: test
```

```bash
# Start LocalStack with Docker
docker run -d -p 4566:4566 localstack/localstack

# Create a local queue
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name my-queue
```

---

## Application Properties Reference

```properties
# Queue names (externalized for environment-specific config)
sqs.queue.orders=orders-queue
sqs.queue.orders.dlq=orders-queue-dlq
sqs.queue.notifications=notifications-queue

# Visibility timeout (seconds) — set higher than your max processing time
sqs.visibility.timeout=60

# Long polling wait time (seconds)
sqs.wait.time.seconds=20

# Batch size
sqs.max.messages=10
```
