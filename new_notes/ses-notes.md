# AWS SES

Amazon Simple Email Service (SES) is a cloud-based email sending service designed to help digital marketers and application developers send marketing, notification, and transactional emails.

Key features include:

- Fully managed email service with no email server management required
- High deliverability with built-in reputation management
- Global availability with regional email sending
- Cost-effective pay-as-you-go pricing model
- Integration ready with AWS services and third-party applications

---

## Core Concepts

### SES Identity

An SES identity is a verified email address or domain that you use to send emails through Amazon SES.

- Establishes sender authenticity and prevents spam
- Can be individual email addresses or entire domains
- Verification required before sending emails

**Verification Status:**

| Status | Description |
|---|---|
| Pending | Verification in progress |
| Success | Successfully verified |
| Failed | Verification failed |
| TemporaryFailure | Temporary verification issue |
| NotStarted | Verification not initiated |

### Verified Email Addresses

Individual email addresses that have been verified for sending emails.

- Specific email addresses authorized to send emails
- AWS sends confirmation email to the address for verification
- Each address must be individually verified
- Ideal for testing, small-scale sending, or specific sender addresses
- Can be added, removed, or re-verified through console or API

### Verified Domains

Entire domains that have been verified for sending emails from any address within that domain.

- Domain names authorized for email sending
- Verification uses DNS TXT record
- Send from any address within the verified domain
- Automatic verification for all subdomains
- Supports Domain-based email authentication (DKIM)

### Sandbox Mode

SES starts in sandbox mode with restricted sending capabilities for security.

**Sandbox Restrictions:**

| Limit Type | Sandbox Value | Purpose |
|---|---|---|
| Daily Quota | 200 emails per 24 hours | Prevent abuse |
| Send Rate | 1 email per second | Maintain service quality |
| Recipients | Only verified addresses | Security |
| Max Recipients | 50 per message | Control |

You must request production access to move out of sandbox mode.

### Sending Limits

SES enforces sending limits to maintain service quality and prevent abuse.

**Key Limits:**

- **Send Rate** - Maximum emails per second, starts at 1 per second
- **Send Quota** - Maximum emails per 24-hour period, starts at 200
- **Bounce Rate** - Must stay below 5% for good reputation
- **Complaint Rate** - Must stay below 0.1% for good reputation

Limits automatically increase based on sending behavior and reputation.

### Email Types

| Email Type | Use Case |
|---|---|
| Transactional | Order confirmations, password resets, notifications |
| Marketing | Newsletters, promotional emails, campaigns |
| Bulk | Large volume emails to multiple recipients |
| Test | Development and testing emails in sandbox mode |
| System | Automated system notifications and alerts |

### Bounce and Complaint Handling

SES tracks email delivery issues and recipient feedback to maintain reputation.

**Bounce Types:**

- **Hard Bounce** - Permanent delivery failure due to invalid email address
- **Soft Bounce** - Temporary delivery failure due to full mailbox or temporary server issues
- **Complaint** - Recipient marked email as spam

**Suppression List:**

- Automatically maintained list of problematic addresses
- Prevents sending to addresses that previously bounced or complained
- High bounce and complaint rates affect sending ability and reputation

### Configuration Sets

Configuration sets allow you to organize and track email sending with custom settings.

Features include:

- Named groups of rules for email sending and tracking
- Event publishing to CloudWatch, SNS, or Kinesis
- Reputation tracking for bounce and complaint rates
- Delivery options including IP pools and delivery delays
- Tags to organize and categorize email sending

---

## Email Structure

### Email Anatomy

Each SES email includes the following components:

- **Message ID** - Unique identifier assigned by SES
- **From Address** - Sender email address, must be verified
- **To/CC/BCC** - Recipient email addresses
- **Subject** - Email subject line
- **Message Body** - HTML and/or text content
- **Headers** - Additional email headers and metadata
- **Attachments** - File attachments (optional)
- **Configuration Set** - Applied rules and tracking (optional)

### Email Lifecycle States

SES emails progress through different states during delivery:

1. **Send** - Email submitted to SES for processing
2. **Reject** - Email rejected due to reputation or content issues
3. **Bounce** - Email bounced by recipient's mail server
4. **Complaint** - Recipient marked email as spam
5. **Delivery** - Email successfully delivered to recipient
6. **Open** - Recipient opened the email (if tracking enabled)
7. **Click** - Recipient clicked link in email (if tracking enabled)

---

## Authentication Methods

### DKIM (DomainKeys Identified Mail)

Cryptographic authentication of email sender.

- Uses DNS CNAME records for domain verification
- SES manages DKIM keys automatically
- Improves email deliverability and sender reputation

### SPF (Sender Policy Framework)

Authorizes IP addresses to send email for your domain.

- Uses DNS TXT record specifying allowed senders
- Include SES IP ranges in SPF record
- Example: `v=spf1 include:amazonses.com ~all`

### DMARC (Domain-based Message Authentication)

Policy framework using SPF and DKIM results.

- Uses DNS TXT record with policy instructions
- Actions include none, quarantine, or reject for failed authentication
- Provides aggregate and forensic reports on email authentication

---

## Email Templates

### Template Structure

Email templates consist of:

- **Template Name** - Unique identifier for the template
- **Subject** - Email subject with placeholder support
- **HTML Part** - Rich HTML email content
- **Text Part** - Plain text fallback content
- **Template Data** - JSON data for placeholder replacement

### Template Variables

- **Syntax** - Use `{{variable_name}}` for placeholders
- **Types** - String, number, boolean, array, object
- **Default Values** - Specify fallback values for missing data
- **Conditional Logic** - Basic if/else statements supported

---

## Sending Methods

| Method | Purpose | Use Case | Features |
|---|---|---|---|
| Simple Send | Individual emails with basic content | Simple notifications and alerts | Direct HTML/text in API call |
| Template Send | Emails using predefined templates | Consistent branding and dynamic content | JSON object with replacement values |
| Bulk Template Send | Template-based emails to multiple recipients | Newsletters, marketing campaigns | Individual template data per recipient |

---

## Event Publishing and Tracking

### Email Events

SES tracks the following events:

- **Send** - Email was accepted by SES
- **Reject** - Email was rejected by SES
- **Bounce** - Email bounced from recipient server
- **Complaint** - Email was marked as spam
- **Delivery** - Email was delivered successfully
- **Open** - Email was opened by recipient
- **Click** - Link in email was clicked
- **Rendering Failure** - Template rendering failed

### Event Destinations

| Destination | Purpose |
|---|---|
| CloudWatch | Metrics and monitoring integration |
| SNS | Real-time event notifications |
| Kinesis Data Firehose | Stream events to data stores |
| Event Bridge | Route events to AWS services |

### Event Data Structure

```json
{
  "eventType": "delivery",
  "mail": {
    "timestamp": "2024-01-15T10:30:00.000Z",
    "source": "sender@example.com",
    "messageId": "0000014a3f-12345678-abcd-1234-abcd-1234567890ab-000000",
    "destination": ["recipient@example.com"]
  },
  "delivery": {
    "timestamp": "2024-01-15T10:30:05.000Z",
    "recipients": ["recipient@example.com"],
    "smtpResponse": "250 2.0.0 OK"
  }
}
```

---

## Common Use Cases

### Transactional Emails

Send automated emails triggered by user actions.

Examples:

- Order confirmations and receipts
- Password reset requests
- Account verification emails
- Shipping notifications
- Payment confirmations

### Marketing Campaigns

Send promotional content to subscribers.

Examples:

- Newsletters and updates
- Product announcements
- Seasonal promotions
- Event invitations
- Customer surveys

### System Notifications

Send automated alerts and monitoring emails.

Examples:

- Server health alerts
- Error notifications
- Performance warnings
- Backup completion reports
- Security alerts

---

## Best Practices

### Reputation Management

- Monitor bounce and complaint rates daily
- Remove invalid email addresses from your lists
- Use double opt-in for subscription confirmations
- Provide clear unsubscribe options
- Authenticate your domain with DKIM, SPF, and DMARC

### Email Content

- Use responsive HTML templates for better rendering
- Include both HTML and plain text versions
- Keep subject lines clear and relevant
- Avoid spam trigger words and excessive punctuation
- Test emails across different email clients

### Deliverability

- Warm up new IP addresses gradually
- Segment your email lists for targeted sending
- Monitor engagement metrics (opens, clicks)
- Clean your email lists regularly
- Use configuration sets to track performance

### Security

- Use IAM roles instead of access keys
- Enable encryption for sensitive data
- Implement proper authentication methods
- Regularly rotate credentials
- Monitor for unauthorized access

### Cost Optimization

- Use templates to reduce payload size
- Batch emails when possible
- Remove inactive subscribers
- Monitor sending quotas and usage
- Use the right pricing tier for your volume

---

## Spring Boot Integration

### Dependencies

Add the following dependencies to your project:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>ses</artifactId>
    <version>2.20.0</version>
</dependency>
```

### Configuration Class

```java
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.ses.SesClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SesConfig {

    @Bean
    public SesClient sesClient() {
        return SesClient.builder()
                .region(Region.AP_SOUTH_1)
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }
}
```

### Service Implementation

```java
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.ses.SesClient;
import software.amazon.awssdk.services.ses.model.*;

import java.util.Map;

@Service
public class SESEmailService {

    private final SesClient sesClient;
    private final String fromEmail = "noreply@example.com";

    public SESEmailService(SesClient sesClient) {
        this.sesClient = sesClient;
    }

    public String sendSimpleEmail(String to, String subject, String body) {
        Destination destination = Destination.builder()
                .toAddresses(to)
                .build();

        Content subjectContent = Content.builder()
                .data(subject)
                .build();

        Content bodyContent = Content.builder()
                .data(body)
                .build();

        Body emailBody = Body.builder()
                .text(bodyContent)
                .build();

        Message message = Message.builder()
                .subject(subjectContent)
                .body(emailBody)
                .build();

        SendEmailRequest emailRequest = SendEmailRequest.builder()
                .source(fromEmail)
                .destination(destination)
                .message(message)
                .build();

        SendEmailResponse response = sesClient.sendEmail(emailRequest);
        return response.messageId();
    }

    public String sendHtmlEmail(String to, String subject, String htmlBody) {
        Destination destination = Destination.builder()
                .toAddresses(to)
                .build();

        Content subjectContent = Content.builder()
                .data(subject)
                .build();

        Content htmlContent = Content.builder()
                .data(htmlBody)
                .build();

        Body emailBody = Body.builder()
                .html(htmlContent)
                .build();

        Message message = Message.builder()
                .subject(subjectContent)
                .body(emailBody)
                .build();

        SendEmailRequest emailRequest = SendEmailRequest.builder()
                .source(fromEmail)
                .destination(destination)
                .message(message)
                .build();

        SendEmailResponse response = sesClient.sendEmail(emailRequest);
        return response.messageId();
    }

    public String sendTemplatedEmail(String to, String templateName, Map<String, String> templateData) {
        Destination destination = Destination.builder()
                .toAddresses(to)
                .build();

        String templateDataJson = convertMapToJson(templateData);

        SendTemplatedEmailRequest request = SendTemplatedEmailRequest.builder()
                .source(fromEmail)
                .destination(destination)
                .template(templateName)
                .templateData(templateDataJson)
                .build();

        SendTemplatedEmailResponse response = sesClient.sendTemplatedEmail(request);
        return response.messageId();
    }

    public String sendEmailWithAttachment(String to, String subject, String body, byte[] attachmentData, String attachmentName) {
        try {
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            
            MimeMessage message = new MimeMessage(Session.getDefaultInstance(new Properties()));
            message.setFrom(new InternetAddress(fromEmail));
            message.setRecipients(Message.RecipientType.TO, InternetAddress.parse(to));
            message.setSubject(subject);
            
            MimeBodyPart textPart = new MimeBodyPart();
            textPart.setText(body);
            
            MimeBodyPart attachmentPart = new MimeBodyPart();
            DataSource source = new ByteArrayDataSource(attachmentData, "application/octet-stream");
            attachmentPart.setDataHandler(new DataHandler(source));
            attachmentPart.setFileName(attachmentName);
            
            Multipart multipart = new MimeMultipart();
            multipart.addBodyPart(textPart);
            multipart.addBodyPart(attachmentPart);
            
            message.setContent(multipart);
            message.writeTo(outputStream);
            
            ByteBuffer buffer = ByteBuffer.wrap(outputStream.toByteArray());
            RawMessage rawMessage = RawMessage.builder()
                    .data(SdkBytes.fromByteBuffer(buffer))
                    .build();
            
            SendRawEmailRequest rawEmailRequest = SendRawEmailRequest.builder()
                    .rawMessage(rawMessage)
                    .build();
            
            SendRawEmailResponse response = sesClient.sendRawEmail(rawEmailRequest);
            return response.messageId();
            
        } catch (Exception e) {
            throw new RuntimeException("Failed to send email with attachment", e);
        }
    }

    private String convertMapToJson(Map<String, String> data) {
        StringBuilder json = new StringBuilder("{");
        data.forEach((key, value) -> 
            json.append("\"").append(key).append("\":\"").append(value).append("\",")
        );
        if (json.length() > 1) {
            json.setLength(json.length() - 1);
        }
        json.append("}");
        return json.toString();
    }
}
```

### Template Management Service

```java
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.ses.SesClient;
import software.amazon.awssdk.services.ses.model.*;

@Service
public class SESTemplateService {

    private final SesClient sesClient;

    public SESTemplateService(SesClient sesClient) {
        this.sesClient = sesClient;
    }

    public void createTemplate(String templateName, String subject, String htmlBody, String textBody) {
        Template template = Template.builder()
                .templateName(templateName)
                .subjectPart(subject)
                .htmlPart(htmlBody)
                .textPart(textBody)
                .build();

        CreateTemplateRequest request = CreateTemplateRequest.builder()
                .template(template)
                .build();

        sesClient.createTemplate(request);
        System.out.println("Template created: " + templateName);
    }

    public void updateTemplate(String templateName, String subject, String htmlBody, String textBody) {
        Template template = Template.builder()
                .templateName(templateName)
                .subjectPart(subject)
                .htmlPart(htmlBody)
                .textPart(textBody)
                .build();

        UpdateTemplateRequest request = UpdateTemplateRequest.builder()
                .template(template)
                .build();

        sesClient.updateTemplate(request);
        System.out.println("Template updated: " + templateName);
    }

    public void deleteTemplate(String templateName) {
        DeleteTemplateRequest request = DeleteTemplateRequest.builder()
                .templateName(templateName)
                .build();

        sesClient.deleteTemplate(request);
        System.out.println("Template deleted: " + templateName);
    }

    public String getTemplate(String templateName) {
        GetTemplateRequest request = GetTemplateRequest.builder()
                .templateName(templateName)
                .build();

        GetTemplateResponse response = sesClient.getTemplate(request);
        return response.template().toString();
    }
}
```

### REST Controller

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.Map;

@RestController
@RequestMapping("/api/email")
public class EmailController {

    private final SESEmailService emailService;
    private final SESTemplateService templateService;

    public EmailController(SESEmailService emailService, SESTemplateService templateService) {
        this.emailService = emailService;
        this.templateService = templateService;
    }

    @PostMapping("/send/simple")
    public ResponseEntity<Map<String, String>> sendSimpleEmail(
            @RequestParam String to,
            @RequestParam String subject,
            @RequestParam String body) {
        
        String messageId = emailService.sendSimpleEmail(to, subject, body);
        
        return ResponseEntity.ok(Map.of(
            "message", "Email sent successfully",
            "messageId", messageId
        ));
    }

    @PostMapping("/send/html")
    public ResponseEntity<Map<String, String>> sendHtmlEmail(
            @RequestParam String to,
            @RequestParam String subject,
            @RequestParam String htmlBody) {
        
        String messageId = emailService.sendHtmlEmail(to, subject, htmlBody);
        
        return ResponseEntity.ok(Map.of(
            "message", "HTML email sent successfully",
            "messageId", messageId
        ));
    }

    @PostMapping("/send/template")
    public ResponseEntity<Map<String, String>> sendTemplatedEmail(
            @RequestParam String to,
            @RequestParam String templateName,
            @RequestBody Map<String, String> templateData) {
        
        String messageId = emailService.sendTemplatedEmail(to, templateName, templateData);
        
        return ResponseEntity.ok(Map.of(
            "message", "Template email sent successfully",
            "messageId", messageId
        ));
    }

    @PostMapping("/send/attachment")
    public ResponseEntity<Map<String, String>> sendEmailWithAttachment(
            @RequestParam String to,
            @RequestParam String subject,
            @RequestParam String body,
            @RequestParam MultipartFile file) throws Exception {
        
        String messageId = emailService.sendEmailWithAttachment(
            to, subject, body, file.getBytes(), file.getOriginalFilename()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Email with attachment sent successfully",
            "messageId", messageId
        ));
    }

    @PostMapping("/templates")
    public ResponseEntity<Map<String, String>> createTemplate(
            @RequestParam String templateName,
            @RequestParam String subject,
            @RequestParam String htmlBody,
            @RequestParam String textBody) {
        
        templateService.createTemplate(templateName, subject, htmlBody, textBody);
        
        return ResponseEntity.ok(Map.of(
            "message", "Template created successfully",
            "templateName", templateName
        ));
    }
}
```

### Rate Limiter Service

```java
import org.springframework.stereotype.Service;
import java.time.Instant;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Service
public class EmailRateLimiter {
    
    private final Map<String, RateLimitInfo> rateLimits = new ConcurrentHashMap<>();
    private final int maxEmailsPerSecond = 14;
    private final int maxEmailsPerDay = 50000;
    
    public boolean canSendEmail(String identifier) {
        RateLimitInfo info = rateLimits.computeIfAbsent(identifier, k -> new RateLimitInfo());
        
        long now = Instant.now().getEpochSecond();
        
        if (now != info.lastSecond) {
            info.lastSecond = now;
            info.emailsThisSecond = 0;
        }
        
        long dayStart = now - (now % 86400);
        if (dayStart != info.lastDayStart) {
            info.lastDayStart = dayStart;
            info.emailsToday = 0;
        }
        
        return info.emailsThisSecond < maxEmailsPerSecond && info.emailsToday < maxEmailsPerDay;
    }
    
    public void recordEmailSent(String identifier) {
        RateLimitInfo info = rateLimits.get(identifier);
        if (info != null) {
            info.emailsThisSecond++;
            info.emailsToday++;
        }
    }
    
    private static class RateLimitInfo {
        long lastSecond = 0;
        int emailsThisSecond = 0;
        long lastDayStart = 0;
        int emailsToday = 0;
    }
}
```

### Email Analytics Service

```java
import org.springframework.stereotype.Service;
import java.time.LocalDateTime;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class EmailAnalyticsService {
    
    private final Map<String, EmailMetrics> campaignMetrics = new ConcurrentHashMap<>();
    
    public void recordEmailSent(String campaignId, String messageId, String recipient) {
        EmailMetrics metrics = campaignMetrics.computeIfAbsent(campaignId, k -> new EmailMetrics());
        metrics.sent.incrementAndGet();
        metrics.lastActivity = LocalDateTime.now();
    }
    
    public void recordEmailDelivered(String campaignId, String messageId) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics != null) {
            metrics.delivered.incrementAndGet();
        }
    }
    
    public void recordEmailOpened(String campaignId, String messageId) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics != null) {
            metrics.opens.incrementAndGet();
        }
    }
    
    public void recordEmailClicked(String campaignId, String messageId, String link) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics != null) {
            metrics.clicks.incrementAndGet();
        }
    }
    
    public void recordEmailBounced(String campaignId, String messageId, String bounceType) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics != null) {
            metrics.bounces.incrementAndGet();
        }
    }
    
    public void recordEmailComplaint(String campaignId, String messageId) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics != null) {
            metrics.complaints.incrementAndGet();
        }
    }
    
    public EmailAnalytics getCampaignAnalytics(String campaignId) {
        EmailMetrics metrics = campaignMetrics.get(campaignId);
        if (metrics == null) {
            return new EmailAnalytics(0, 0, 0, 0, 0, 0, 0.0, 0.0, 0.0, 0.0);
        }
        
        long sent = metrics.sent.get();
        long delivered = metrics.delivered.get();
        long opens = metrics.opens.get();
        long clicks = metrics.clicks.get();
        long bounces = metrics.bounces.get();
        long complaints = metrics.complaints.get();
        
        double deliveryRate = sent > 0 ? (double) delivered / sent * 100 : 0.0;
        double openRate = delivered > 0 ? (double) opens / delivered * 100 : 0.0;
        double clickRate = delivered > 0 ? (double) clicks / delivered * 100 : 0.0;
        double bounceRate = sent > 0 ? (double) bounces / sent * 100 : 0.0;
        
        return new EmailAnalytics(sent, delivered, opens, clicks, bounces, complaints, 
                                deliveryRate, openRate, clickRate, bounceRate);
    }
    
    private static class EmailMetrics {
        AtomicLong sent = new AtomicLong(0);
        AtomicLong delivered = new AtomicLong(0);
        AtomicLong opens = new AtomicLong(0);
        AtomicLong clicks = new AtomicLong(0);
        AtomicLong bounces = new AtomicLong(0);
        AtomicLong complaints = new AtomicLong(0);
        LocalDateTime lastActivity = LocalDateTime.now();
    }
    
    public record EmailAnalytics(
        long sent,
        long delivered,
        long opens,
        long clicks,
        long bounces,
        long complaints,
        double deliveryRate,
        double openRate,
        double clickRate,
        double bounceRate
    ) {}
}
```

### Health Check

```java
import org.springframework.boot.actuator.health.Health;
import org.springframework.boot.actuator.health.HealthIndicator;
import org.springframework.stereotype.Component;
import software.amazon.awssdk.services.ses.SesClient;
import software.amazon.awssdk.services.ses.model.GetSendQuotaRequest;

@Component
public class SESHealthIndicator implements HealthIndicator {
    
    private final SesClient sesClient;
    
    public SESHealthIndicator(SesClient sesClient) {
        this.sesClient = sesClient;
    }
    
    @Override
    public Health health() {
        try {
            GetSendQuotaRequest request = GetSendQuotaRequest.builder().build();
            var response = sesClient.getSendQuota(request);
            
            double quotaUsage = (response.sentLast24Hours() / response.max24HourSend()) * 100;
            
            Health.Builder builder = quotaUsage < 90 ? Health.up() : Health.down();
            
            return builder
                .withDetail("maxSendRate", response.maxSendRate())
                .withDetail("max24HourSend", response.max24HourSend())
                .withDetail("sentLast24Hours", response.sentLast24Hours())
                .withDetail("quotaUsagePercent", String.format("%.2f%%", quotaUsage))
                .build();
                
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

### Batch Email Service

```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.CompletableFuture;

@Service
public class BatchEmailService {
    
    private final SESEmailService sesService;
    private final EmailRateLimiter rateLimiter;
    
    public BatchEmailService(SESEmailService sesService, EmailRateLimiter rateLimiter) {
        this.sesService = sesService;
        this.rateLimiter = rateLimiter;
    }
    
    @Async
    public CompletableFuture<BatchResult> sendBatchEmails(List<EmailRequest> emailRequests) {
        List<String> successful = new ArrayList<>();
        List<String> failed = new ArrayList<>();
        
        for (EmailRequest request : emailRequests) {
            if (rateLimiter.canSendEmail("batch-" + request.campaignId())) {
                try {
                    String messageId = sesService.sendTemplatedEmail(
                        request.recipient(),
                        request.templateName(),
                        request.templateData()
                    );
                    successful.add(messageId);
                    rateLimiter.recordEmailSent("batch-" + request.campaignId());
                    
                    Thread.sleep(75);
                    
                } catch (Exception e) {
                    failed.add(request.recipient());
                    System.err.println("Failed to send to " + request.recipient() + ": " + e.getMessage());
                }
            } else {
                failed.add(request.recipient());
                System.out.println("Rate limit exceeded for campaign: " + request.campaignId());
            }
        }
        
        return CompletableFuture.completedFuture(new BatchResult(successful, failed));
    }
    
    public record EmailRequest(String recipient, String templateName, 
                              Map<String, String> templateData, String campaignId) {}
    public record BatchResult(List<String> successful, List<String> failed) {}
}
```

### Application Configuration

```yaml
aws:
  ses:
    region: ap-south-1
    from-email: noreply@example.com
    configuration-set: default-config-set
  credentials:
    access-key: ${AWS_ACCESS_KEY_ID}
    secret-key: ${AWS_SECRET_ACCESS_KEY}

spring:
  task:
    execution:
      pool:
        core-size: 5
        max-size: 10

logging:
  level:
    software.amazon.awssdk: DEBUG
```