# AWS SES Core Concepts

Amazon Simple Email Service (SES) is a cloud-based email sending service designed to help digital marketers and application developers send marketing, notification, and transactional emails. Understanding the following core concepts is essential for working with SES effectively.

- **Fully managed email service** (no email server management required)
- **High deliverability** with built-in reputation management
- **Global availability** with regional email sending
- **Cost-effective** pay-as-you-go pricing model
- **Integration ready** with AWS services and third-party applications

## 1. SES Identity

An SES identity is a verified email address or domain that you use to send emails through Amazon SES.

- **Definition**: A verified email address or domain authorized to send emails
- **Purpose**: Establishes sender authenticity and prevents spam
- **Types**: Individual email addresses or entire domains
- **Verification**: Required before sending emails
- **Status**: Pending, Success, Failed, TemporaryFailure, NotStarted

## 2. Verified Email Addresses

Individual email addresses that have been verified for sending emails.

- **Definition**: Specific email addresses authorized to send emails
- **Verification Process**: AWS sends confirmation email to the address
- **Use Case**: Testing, small-scale sending, or specific sender addresses
- **Limitations**: Each address must be individually verified
- **Management**: Can be added, removed, or re-verified through console/API

## 3. Verified Domains

Entire domains that have been verified for sending emails from any address within that domain.

- **Definition**: Domain names authorized for email sending
- **Verification Process**: DNS TXT record verification
- **Flexibility**: Send from any address within the verified domain
- **Subdomain Support**: Automatic verification for all subdomains
- **DKIM**: Domain-based email authentication support

## 4. Sandbox Mode

SES starts in sandbox mode with restricted sending capabilities for security.

- **Definition**: Limited sending environment for new accounts
- **Restrictions**: Can only send to verified email addresses
- **Send Limits**: 200 emails per 24 hours, 1 email per second
- **Recipients**: Maximum 50 recipients per message
- **Production Access**: Must request to move out of sandbox
- **Purpose**: Prevents abuse and maintains service reputation

## 5. Sending Limits

SES enforces sending limits to maintain service quality and prevent abuse.

- **Send Rate**: Maximum emails per second (starts at 1 per second)
- **Send Quota**: Maximum emails per 24-hour period (starts at 200)
- **Bounce Rate**: Must stay below 5% for good reputation
- **Complaint Rate**: Must stay below 0.1% for good reputation
- **Automatic Increases**: Limits increase based on sending behavior

## 6. Email Types

SES supports different types of email communication with varying requirements.

- **Transactional**: Order confirmations, password resets, notifications
- **Marketing**: Newsletters, promotional emails, campaigns
- **Bulk**: Large volume emails to multiple recipients
- **Test**: Development and testing emails in sandbox mode
- **System**: Automated system notifications and alerts

## 7. Bounce and Complaint Handling

SES tracks email delivery issues and recipient feedback to maintain reputation.

- **Hard Bounce**: Permanent delivery failure (invalid email address)
- **Soft Bounce**: Temporary delivery failure (full mailbox)
- **Complaint**: Recipient marked email as spam
- **Suppression List**: Automatically maintained list of problematic addresses
- **Reputation Impact**: High bounce/complaint rates affect sending ability

## 8. Configuration Sets

Configuration sets allow you to organize and track email sending with custom settings.

- **Definition**: Named groups of rules for email sending and tracking
- **Event Publishing**: Send email events to CloudWatch, SNS, or Kinesis
- **Reputation Tracking**: Monitor bounce and complaint rates
- **Delivery Options**: Configure IP pools and delivery delays
- **Tags**: Organize and categorize email sending

---

## SES Email Anatomy
Each SES email includes the following components:

* **Message ID** – Unique identifier assigned by SES
* **From Address** – Sender email address (must be verified)
* **To/CC/BCC** – Recipient email addresses
* **Subject** – Email subject line
* **Message Body** – HTML and/or text content
* **Headers** – Additional email headers and metadata
* **Attachments** – File attachments (optional)
* **Configuration Set** – Applied rules and tracking (optional)

---

## Email Lifecycle States
SES emails progress through different states during delivery:

* **Send** – Email submitted to SES for processing
* **Reject** – Email rejected due to reputation or content issues
* **Bounce** – Email bounced by recipient's mail server
* **Complaint** – Recipient marked email as spam
* **Delivery** – Email successfully delivered to recipient
* **Open** – Recipient opened the email (if tracking enabled)
* **Click** – Recipient clicked link in email (if tracking enabled)

---

## Authentication Methods

### DKIM (DomainKeys Identified Mail)
* **Purpose**: Cryptographic authentication of email sender
* **Implementation**: DNS CNAME records for domain verification
* **Key Management**: SES manages DKIM keys automatically
* **Reputation**: Improves email deliverability and sender reputation

### SPF (Sender Policy Framework)
* **Purpose**: Authorize IP addresses to send email for domain
* **Implementation**: DNS TXT record specifying allowed senders
* **SES Integration**: Include SES IP ranges in SPF record
* **Example**: `v=spf1 include:amazonses.com ~all`

### DMARC (Domain-based Message Authentication)
* **Purpose**: Policy framework using SPF and DKIM results
* **Implementation**: DNS TXT record with policy instructions
* **Actions**: None, quarantine, or reject for failed authentication
* **Reporting**: Aggregate and forensic reports on email authentication

---

## Email Templates

### Template Structure
* **Template Name**: Unique identifier for the template
* **Subject**: Email subject with placeholder support
* **HTML Part**: Rich HTML email content
* **Text Part**: Plain text fallback content
* **Template Data**: JSON data for placeholder replacement

### Template Variables
* **Syntax**: Use `{{variable_name}}` for placeholders
* **Types**: String, number, boolean, array, object
* **Default Values**: Specify fallback values for missing data
* **Conditional Logic**: Basic if/else statements supported

---

## Sending Methods

### Simple Send
* **Purpose**: Send individual emails with basic content
* **Use Case**: Simple notifications and alerts
* **Content**: Direct HTML/text in API call
* **Limitations**: No template support or bulk sending

### Template Send
* **Purpose**: Send emails using predefined templates
* **Use Case**: Consistent branding and dynamic content
* **Template Data**: JSON object with replacement values
* **Efficiency**: Reduced payload size for bulk sending

### Bulk Template Send
* **Purpose**: Send template-based emails to multiple recipients
* **Use Case**: Newsletters, marketing campaigns
* **Personalization**: Individual template data per recipient
* **Performance**: Optimized for large recipient lists

---

## Event Publishing and Tracking

### Email Events
* **Send**: Email was accepted by SES
* **Reject**: Email was rejected by SES
* **Bounce**: Email bounced from recipient server
* **Complaint**: Email was marked as spam
* **Delivery**: Email was delivered successfully
* **Open**: Email was opened by recipient
* **Click**: Link in email was clicked
* **Rendering Failure**: Template rendering failed

### Event Destinations
* **CloudWatch**: Metrics and monitoring integration
* **SNS**: Real-time event notifications
* **Kinesis Data Firehose**: Stream events to data stores
* **Event Bridge**: Route events to AWS services

### Event Data Structure
```json
{
  "eventType": "delivery",
  "mail": {
    "messageId": "0000014a-f4d9-4df4-a0a0-12345678901a",
    "timestamp": "2024-01-15T10:30:45.000Z",
    "source": "sender@example.com",
    "destination": ["recipient@example.com"]
  },
  "delivery": {
    "timestamp": "2024-01-15T10:30:47.000Z",
    "processingTimeMillis": 2000,
    "recipients": ["recipient@example.com"],
    "smtpResponse": "250 2.0.0 OK"
  }
}
```

---

## IP Pools and Dedicated IPs

### Shared IPs
* **Default**: All accounts start with shared IP addresses
* **Reputation**: Shared reputation across all SES users
* **Cost**: Included in SES pricing at no additional cost
* **Management**: AWS manages IP reputation automatically

### Dedicated IPs
* **Purpose**: Exclusive IP addresses for your sending
* **Reputation**: Full control over IP reputation
* **Warm-up**: Gradual increase in sending volume required
* **Cost**: Additional monthly fee per dedicated IP
* **Use Case**: High-volume senders with established reputation

### IP Pools
* **Definition**: Groups of dedicated IP addresses
* **Purpose**: Organize IPs by email type or sending behavior
* **Configuration**: Assign configuration sets to specific pools
* **Reputation Isolation**: Separate reputation for different email types

---

## Reputation Management

### Bounce Rate Management
* **Target**: Keep bounce rate below 5%
* **Hard Bounces**: Remove from sending lists immediately
* **Soft Bounces**: Retry with exponential backoff
* **Monitoring**: Track bounce rates in CloudWatch
* **Suppression**: Automatic suppression of bouncing addresses

### Complaint Rate Management
* **Target**: Keep complaint rate below 0.1%
* **Feedback Loops**: Process ISP complaint notifications
* **Unsubscribe**: Provide easy unsubscribe mechanisms
* **List Hygiene**: Regular cleaning of recipient lists
* **Content Quality**: Ensure relevant, expected content

### Sender Reputation
* **Factors**: Bounce rate, complaint rate, sending patterns
* **Impact**: Affects deliverability and sending limits
* **Monitoring**: Use SES reputation metrics
* **Recovery**: Gradual improvement through good practices

---

## Security and Compliance

### IAM Integration
* **Policies**: Control access to SES operations
* **Roles**: Use roles for application access
* **Users**: Individual user permissions for console access
* **Cross-Account**: Share SES resources across accounts

### VPC Integration
* **VPC Endpoints**: Private connectivity to SES
* **Security Groups**: Control network access
* **Private Subnets**: Send emails without internet gateway

### Compliance Features
* **GDPR**: Data protection and privacy controls
* **HIPAA**: Healthcare compliance support
* **SOC**: Security compliance certifications
* **Encryption**: TLS encryption for email transmission

---

## Monitoring and Analytics

### CloudWatch Metrics
* **Send**: Number of emails sent
* **Bounce**: Bounce rate and count
* **Complaint**: Complaint rate and count
* **Delivery**: Successful delivery count
* **Reject**: Rejected email count
* **Reputation**: Sender reputation metrics

### SES Sending Statistics
* **24-Hour Stats**: Recent sending activity
* **Account Level**: Overall account statistics
* **Identity Level**: Per-identity statistics
* **Configuration Set**: Per-configuration-set statistics

### Event Publishing Analytics
* **Real-time Events**: Immediate event notifications
* **Batch Processing**: Aggregate event analysis
* **Custom Dashboards**: Build monitoring dashboards
* **Alerting**: Set up alerts for reputation issues

---

## Best Practices and Patterns

### List Management
* **Double Opt-in**: Confirm subscriptions with verification email
* **Segmentation**: Group recipients by interests and behavior
* **Suppression Lists**: Maintain lists of addresses to avoid
* **Regular Cleaning**: Remove inactive and bouncing addresses

### Content Optimization
* **Personalization**: Use recipient names and relevant content
* **Mobile-Friendly**: Responsive design for mobile devices
* **Clear CTAs**: Obvious call-to-action buttons
* **Unsubscribe Links**: Easy-to-find unsubscribe options

### Sending Patterns
* **Consistent Volume**: Maintain steady sending rates
* **Warm-up Period**: Gradually increase sending volume
* **Time Optimization**: Send at optimal times for recipients
* **Frequency Management**: Avoid overwhelming recipients

### Technical Implementation
* **Error Handling**: Implement retry logic for temporary failures
* **Rate Limiting**: Respect SES sending limits
* **Event Processing**: Handle bounces and complaints automatically
* **Testing**: Use sandbox mode for development and testing

<br/><br/>

---
Here's a clear breakdown with **realistic examples** showing what **SES operations look like** — both **programmatically** and via **AWS SDK** (Java/Spring Boot).

---

## 📧 **SES Email Operations**

### ✅ **Email Structure Example**

| Component                    | Description                 | Example                                |
| ---------------------------- | --------------------------- | -------------------------------------- |
| **Message ID**               | Unique email identifier     | `0000014a-f4d9-4df4-a0a0-12345678901a` |
| **From Address**             | Sender email                | `noreply@mycompany.com`                |
| **To Recipients**            | Primary recipients          | `["user@example.com", "admin@company.com"]` |
| **Subject**                  | Email subject line          | `Welcome to Our Service!`              |
| **HTML Body**                | Rich HTML content           | `<h1>Welcome!</h1><p>Thank you...</p>` |
| **Text Body**                | Plain text fallback        | `Welcome! Thank you for joining...`    |
| **Configuration Set**        | Tracking configuration      | `transactional-emails`                 |
| **Tags**                     | Email categorization        | `{EmailType: "welcome", Campaign: "signup"}` |

---

## 🔍 **Sample SES Email Event (JSON format)**

```json
{
  "eventType": "delivery",
  "mail": {
    "messageId": "0000014a-f4d9-4df4-a0a0-12345678901a",
    "timestamp": "2024-01-15T10:30:45.000Z",
    "source": "noreply@mycompany.com",
    "sourceArn": "arn:aws:ses:ap-south-1:123456789012:identity/mycompany.com",
    "destination": ["user@example.com"],
    "headersTruncated": false,
    "headers": [
      {
        "name": "From",
        "value": "My Company <noreply@mycompany.com>"
      },
      {
        "name": "To",
        "value": "user@example.com"
      },
      {
        "name": "Subject",
        "value": "Welcome to Our Service!"
      }
    ],
    "commonHeaders": {
      "from": ["My Company <noreply@mycompany.com>"],
      "to": ["user@example.com"],
      "subject": "Welcome to Our Service!"
    }
  },
  "delivery": {
    "timestamp": "2024-01-15T10:30:47.000Z",
    "processingTimeMillis": 2000,
    "recipients": ["user@example.com"],
    "smtpResponse": "250 2.0.0 OK 1642248047 example-smtp-server"
  }
}
```

---

## 🧪 **Spring Boot SES Service Example**

```java
import software.amazon.awssdk.services.ses.SesClient;
import software.amazon.awssdk.services.ses.model.*;
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Value;

import java.util.List;
import java.util.Map;
import java.util.HashMap;
import java.time.LocalDateTime;

@Service
public class SESEmailService {

    private final SesClient sesClient;
    
    @Value("${aws.ses.from-email}")
    private String fromEmail;
    
    @Value("${aws.ses.configuration-set}")
    private String configurationSet;

    public SESEmailService(SesClient sesClient) {
        this.sesClient = sesClient;
    }

    // Send simple email
    public String sendSimpleEmail(String toEmail, String subject, String htmlBody, String textBody) {
        try {
            Destination destination = Destination.builder()
                    .toAddresses(toEmail)
                    .build();

            Content subjectContent = Content.builder()
                    .data(subject)
                    .charset("UTF-8")
                    .build();

            Body body = Body.builder()
                    .html(Content.builder()
                            .data(htmlBody)
                            .charset("UTF-8")
                            .build())
                    .text(Content.builder()
                            .data(textBody)
                            .charset("UTF-8")
                            .build())
                    .build();

            Message message = Message.builder()
                    .subject(subjectContent)
                    .body(body)
                    .build();

            SendEmailRequest emailRequest = SendEmailRequest.builder()
                    .source(fromEmail)
                    .destination(destination)
                    .message(message)
                    .configurationSetName(configurationSet)
                    .tags(
                        MessageTag.builder()
                            .name("EmailType")
                            .value("simple")
                            .build(),
                        MessageTag.builder()
                            .name("Timestamp")
                            .value(LocalDateTime.now().toString())
                            .build()
                    )
                    .build();

            SendEmailResponse response = sesClient.sendEmail(emailRequest);
            String messageId = response.messageId();
            
            System.out.println("Email sent successfully. Message ID: " + messageId);
            return messageId;

        } catch (Exception e) {
            System.err.println("Failed to send email: " + e.getMessage());
            throw new RuntimeException("Email sending failed", e);
        }
    }

    // Send bulk emails to multiple recipients
    public List<String> sendBulkEmails(List<String> recipients, String subject, 
                                      String htmlTemplate, String textTemplate) {
        List<String> messageIds = new ArrayList<>();
        
        for (String recipient : recipients) {
            try {
                String messageId = sendSimpleEmail(recipient, subject, htmlTemplate, textTemplate);
                messageIds.add(messageId);
                
                // Add small delay to respect rate limits
                Thread.sleep(100);
                
            } catch (Exception e) {
                System.err.println("Failed to send email to " + recipient + ": " + e.getMessage());
            }
        }
        
        return messageIds;
    }

    // Send templated email
    public String sendTemplatedEmail(String toEmail, String templateName, Map<String, String> templateData) {
        try {
            Destination destination = Destination.builder()
                    .toAddresses(toEmail)
                    .build();

            // Convert template data to JSON string
            String templateDataJson = convertMapToJson(templateData);

            SendTemplatedEmailRequest templateRequest = SendTemplatedEmailRequest.builder()
                    .source(fromEmail)
                    .destination(destination)
                    .template(templateName)
                    .templateData(templateDataJson)
                    .configurationSetName(configurationSet)
                    .tags(
                        MessageTag.builder()
                            .name("EmailType")
                            .value("templated")
                            .build(),
                        MessageTag.builder()
                            .name("Template")
                            .value(templateName)
                            .build()
                    )
                    .build();

            SendTemplatedEmailResponse response = sesClient.sendTemplatedEmail(templateRequest);
            String messageId = response.messageId();
            
            System.out.println("Templated email sent successfully. Message ID: " + messageId);
            return messageId;

        } catch (Exception e) {
            System.err.println("Failed to send templated email: " + e.getMessage());
            throw new RuntimeException("Templated email sending failed", e);
        }
    }

    // Send bulk templated emails
    public String sendBulkTemplatedEmails(String templateName, List<BulkEmailDestination> destinations) {
        try {
            SendBulkTemplatedEmailRequest bulkRequest = SendBulkTemplatedEmailRequest.builder()
                    .source(fromEmail)
                    .template(templateName)
                    .defaultTemplateData("{}")
                    .destinations(destinations)
                    .configurationSetName(configurationSet)
                    .tags(
                        MessageTag.builder()
                            .name("EmailType")
                            .value("bulk-templated")
                            .build(),
                        MessageTag.builder()
                            .name("Template")
                            .value(templateName)
                            .build()
                    )
                    .build();

            SendBulkTemplatedEmailResponse response = sesClient.sendBulkTemplatedEmail(bulkRequest);
            String messageId = response.messageId();
            
            System.out.println("Bulk templated emails sent successfully. Message ID: " + messageId);
            return messageId;

        } catch (Exception e) {
            System.err.println("Failed to send bulk templated emails: " + e.getMessage());
            throw new RuntimeException("Bulk templated email sending failed", e);
        }
    }

    // Create or update email template
    public void createEmailTemplate(String templateName, String subject, 
                                   String htmlPart, String textPart) {
        try {
            Template template = Template.builder()
                    .templateName(templateName)
                    .subjectPart(subject)
                    .htmlPart(htmlPart)
                    .textPart(textPart)
                    .build();

            CreateTemplateRequest createRequest = CreateTemplateRequest.builder()
                    .template(template)
                    .build();

            sesClient.createTemplate(createRequest);
            System.out.println("Email template created: " + templateName);

        } catch (AlreadyExistsException e) {
            // Template exists, update it instead
            updateEmailTemplate(templateName, subject, htmlPart, textPart);
        } catch (Exception e) {
            System.err.println("Failed to create template: " + e.getMessage());
            throw new RuntimeException("Template creation failed", e);
        }
    }

    // Update existing email template
    public void updateEmailTemplate(String templateName, String subject, 
                                   String htmlPart, String textPart) {
        try {
            Template template = Template.builder()
                    .templateName(templateName)
                    .subjectPart(subject)
                    .htmlPart(htmlPart)
                    .textPart(textPart)
                    .build();

            UpdateTemplateRequest updateRequest = UpdateTemplateRequest.builder()
                    .template(template)
                    .build();

            sesClient.updateTemplate(updateRequest);
            System.out.println("Email template updated: " + templateName);

        } catch (Exception e) {
            System.err.println("Failed to update template: " + e.getMessage());
            throw new RuntimeException("Template update failed", e);
        }
    }

    // Get sending statistics
    public SendDataPoint getSendingStatistics() {
        try {
            GetSendStatisticsRequest request = GetSendStatisticsRequest.builder().build();
            GetSendStatisticsResponse response = sesClient.getSendStatistics(request);
            
            List<SendDataPoint> dataPoints = response.sendDataPoints();
            if (!dataPoints.isEmpty()) {
                return dataPoints.get(dataPoints.size() - 1); // Return latest data point
            }
            
            return null;

        } catch (Exception e) {
            System.err.println("Failed to get sending statistics: " + e.getMessage());
            throw new RuntimeException("Failed to retrieve statistics", e);
        }
    }

    // Verify email address
    public void verifyEmailAddress(String emailAddress) {
        try {
            VerifyEmailIdentityRequest verifyRequest = VerifyEmailIdentityRequest.builder()
                    .emailAddress(emailAddress)
                    .build();

            sesClient.verifyEmailIdentity(verifyRequest);
            System.out.println("Verification email sent to: " + emailAddress);

        } catch (Exception e) {
            System.err.println("Failed to verify email address: " + e.getMessage());
            throw new RuntimeException("Email verification failed", e);
        }
    }

    // Get identity verification status
    public String getIdentityVerificationStatus(String identity) {
        try {
            GetIdentityVerificationAttributesRequest request = 
                GetIdentityVerificationAttributesRequest.builder()
                    .identities(identity)
                    .build();

            GetIdentityVerificationAttributesResponse response = 
                sesClient.getIdentityVerificationAttributes(request);

            Map<String, IdentityVerificationAttributes> attributes = 
                response.verificationAttributes();
            
            if (attributes.containsKey(identity)) {
                return attributes.get(identity).verificationStatus().toString();
            }
            
            return "NOT_FOUND";

        } catch (Exception e) {
            System.err.println("Failed to get verification status: " + e.getMessage());
            return "ERROR";
        }
    }

    private String convertMapToJson(Map<String, String> data) {
        // Simple JSON conversion - use Jackson ObjectMapper in production
        StringBuilder json = new StringBuilder("{");
        boolean first = true;
        for (Map.Entry<String, String> entry : data.entrySet()) {
            if (!first) json.append(",");
            json.append("\"").append(entry.getKey()).append("\":\"")
                .append(entry.getValue()).append("\"");
            first = false;
        }
        json.append("}");
        return json.toString();
    }
}
```

---

## 🔐 **REST Controller Example**

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import software.amazon.awssdk.services.ses.model.BulkEmailDestination;
import software.amazon.awssdk.services.ses.model.Destination;
import software.amazon.awssdk.services.ses.model.SendDataPoint;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/emails")
public class EmailController {

    private final SESEmailService sesService;

    public EmailController(SESEmailService sesService) {
        this.sesService = sesService;
    }

    @PostMapping("/send-simple")
    public ResponseEntity<Map<String, String>> sendSimpleEmail(
            @RequestBody SimpleEmailRequest request) {
        
        String messageId = sesService.sendSimpleEmail(
            request.toEmail(),
            request.subject(),
            request.htmlBody(),
            request.textBody()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Email sent successfully",
            "messageId", messageId,
            "recipient", request.toEmail()
        ));
    }

    @PostMapping("/send-templated")
    public ResponseEntity<Map<String, String>> sendTemplatedEmail(
            @RequestBody TemplatedEmailRequest request) {
        
        String messageId = sesService.sendTemplatedEmail(
            request.toEmail(),
            request.templateName(),
            request.templateData()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Templated email sent successfully",
            "messageId", messageId,
            "template", request.templateName()
        ));
    }

    @PostMapping("/send-bulk")
    public ResponseEntity<Map<String, Object>> sendBulkEmails(
            @RequestBody BulkEmailRequest request) {
        
        List<String> messageIds = sesService.sendBulkEmails(
            request.recipients(),
            request.subject(),
            request.htmlBody(),
            request.textBody()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Bulk emails sent",
            "totalSent", messageIds.size(),
            "messageIds", messageIds
        ));
    }

    @PostMapping("/send-bulk-templated")
    public ResponseEntity<Map<String, String>> sendBulkTemplatedEmails(
            @RequestBody BulkTemplatedEmailRequest request) {
        
        List<BulkEmailDestination> destinations = request.recipients().stream()
            .map(recipient -> BulkEmailDestination.builder()
                .destination(Destination.builder()
                    .toAddresses(recipient.email())
                    .build())
                .replacementTemplateData(convertMapToJson(recipient.templateData()))
                .build())
            .collect(Collectors.toList());
        
        String messageId = sesService.sendBulkTemplatedEmails(
            request.templateName(),
            destinations
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Bulk templated emails sent successfully",
            "messageId", messageId,
            "recipientCount", String.valueOf(request.recipients().size())
        ));
    }

    @PostMapping("/templates")
    public ResponseEntity<Map<String, String>> createTemplate(
            @RequestBody CreateTemplateRequest request) {
        
        sesService.createEmailTemplate(
            request.templateName(),
            request.subject(),
            request.htmlPart(),
            request.textPart()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Email template created successfully",
            "templateName", request.templateName()
        ));
    }

    @PutMapping("/templates/{templateName}")
    public ResponseEntity<Map<String, String>> updateTemplate(
            @PathVariable String templateName,
            @RequestBody CreateTemplateRequest request) {
        
        sesService.updateEmailTemplate(
            templateName,
            request.subject(),
            request.htmlPart(),
            request.textPart()
        );
        
        return ResponseEntity.ok(Map.of(
            "message", "Email template updated successfully",
            "templateName", templateName
        ));
    }

    @GetMapping("/statistics")
    public ResponseEntity<EmailStatistics> getStatistics() {
        SendDataPoint stats = sesService.getSendingStatistics();
        
        if (stats != null) {
            EmailStatistics emailStats = new EmailStatistics(
                stats.deliveryAttempts(),
                stats.bounces(),
                stats.complaints(),
                stats.rejects(),
                stats.timestamp().toString()
            );
            return ResponseEntity.ok(emailStats);
        }
        
        return ResponseEntity.noContent().build();
    }

    @PostMapping("/verify/{email}")
    public ResponseEntity<Map<String, String>> verifyEmail(@PathVariable String email) {
        sesService.verifyEmailAddress(email);
        
        return ResponseEntity.ok(Map.of(
            "message", "Verification email sent",
            "email", email
        ));
    }

    @GetMapping("/verify-status/{identity}")
    public ResponseEntity<Map<String, String>> getVerificationStatus(@PathVariable String identity) {
        String status = sesService.getIdentityVerificationStatus(identity);
        
        return ResponseEntity.ok(Map.of(
            "identity", identity,
            "status", status
        ));
    }

    private String convertMapToJson(Map<String, String> data) {
        // Simple implementation - use Jackson in production
        return data.entrySet().stream()
            .map(entry -> "\"" + entry.getKey() + "\":\"" + entry.getValue() + "\"")
            .collect(Collectors.joining(",", "{", "}"));
    }

    // Request DTOs
    record SimpleEmailRequest(String toEmail, String subject, String htmlBody, String textBody) {}
    record TemplatedEmailRequest(String toEmail, String templateName, Map<String, String> templateData) {}
    record BulkEmailRequest(List<String> recipients, String subject, String htmlBody, String textBody) {}
    record BulkTemplatedEmailRequest(String templateName, List<BulkRecipient> recipients) {}
    record BulkRecipient(String email, Map<String, String> templateData) {}
    record CreateTemplateRequest(String templateName, String subject, String htmlPart, String textPart) {}
    record EmailStatistics(Long deliveryAttempts, Long bounces, Long complaints, Long rejects, String timestamp) {}
}
```

---

## ⚙️ **Configuration Example**

```java
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.ses.SesClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;

@Configuration
@EnableConfigurationProperties(SesConfig.SesProperties.class)
public class SesConfig {

    @Bean
    public SesClient sesClient(SesProperties sesProperties) {
        return SesClient.builder()
                .region(Region.of(sesProperties.getRegion()))
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
    }

    @ConfigurationProperties(prefix = "aws.ses")
    public static class SesProperties {
        private String region = "ap-south-1";
        private String fromEmail;
        private String configurationSet;
        private int maxSendRate = 14; // emails per second
        private int maxSendQuota = 200; // emails per 24 hours
        private boolean sandboxMode = true;

        // Getters and setters
        public String getRegion() { return region; }
        public void setRegion(String region) { this.region = region; }
        
        public String getFromEmail() { return fromEmail; }
        public void setFromEmail(String fromEmail) { this.fromEmail = fromEmail; }
        
        public String getConfigurationSet() { return configurationSet; }
        public void setConfigurationSet(String configurationSet) { this.configurationSet = configurationSet; }
        
        public int getMaxSendRate() { return maxSendRate; }
        public void setMaxSendRate(int maxSendRate) { this.maxSendRate = maxSendRate; }
        
        public int getMaxSendQuota() { return maxSendQuota; }
        public void setMaxSendQuota(int maxSendQuota) { this.maxSendQuota = maxSendQuota; }
        
        public boolean isSandboxMode() { return sandboxMode; }
        public void setSandboxMode(boolean sandboxMode) { this.sandboxMode = sandboxMode; }
    }
}
```

---

## 📋 **Sample application.yml**

```yaml
aws:
  ses:
    region: ap-south-1
    from-email: noreply@mycompany.com
    configuration-set: transactional-emails
    max-send-rate: 14
    max-send-quota: 200
    sandbox-mode: false
  credentials:
    access-key: ${AWS_ACCESS_KEY_ID}
    secret-key: ${AWS_SECRET_ACCESS_KEY}

spring:
  application:
    name: email-service
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}

logging:
  level:
    software.amazon.awssdk: INFO
    com.mycompany.email: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when-authorized
```

---

## 📊 **Email Templates Examples**

### Welcome Email Template
```html
<!-- Template Name: welcome-email -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Welcome to {{companyName}}</title>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background-color: #4CAF50; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f9f9f9; }
        .button { background-color: #4CAF50; color: white; padding: 12px 24px; text-decoration: none; border-radius: 4px; display: inline-block; }
        .footer { text-align: center; padding: 20px; font-size: 12px; color: #666; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Welcome to {{companyName}}!</h1>
        </div>
        <div class="content">
            <h2>Hello {{firstName}},</h2>
            <p>Thank you for joining {{companyName}}! We're excited to have you on board.</p>
            <p>Your account has been successfully created with the email address: <strong>{{email}}</strong></p>
            <p>To get started, please click the button below to verify your email address:</p>
            <p style="text-align: center;">
                <a href="{{verificationLink}}" class="button">Verify Email Address</a>
            </p>
            <p>If you have any questions, feel free to contact our support team at {{supportEmail}}.</p>
            <p>Best regards,<br>The {{companyName}} Team</p>
        </div>
        <div class="footer">
            <p>&copy; {{currentYear}} {{companyName}}. All rights reserved.</p>
            <p>{{companyAddress}}</p>
        </div>
    </div>
</body>
</html>
```

### Text Version
```text
Welcome to {{companyName}}!

Hello {{firstName}},

Thank you for joining {{companyName}}! We're excited to have you on board.

Your account has been successfully created with the email address: {{email}}

To get started, please verify your email address by visiting: {{verificationLink}}

If you have any questions, feel free to contact our support team at {{supportEmail}}.

Best regards,
The {{companyName}} Team

© {{currentYear}} {{companyName}}. All rights reserved.
{{companyAddress}}
```

### Password Reset Template
```html
<!-- Template Name: password-reset -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Password Reset Request</title>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background-color: #FF6B6B; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background-color: #f9f9f9; }
        .button { background-color: #FF6B6B; color: white; padding: 12px 24px; text-decoration: none; border-radius: 4px; display: inline-block; }
        .warning { background-color: #FFF3CD; border: 1px solid #FFEAA7; padding: 15px; border-radius: 4px; margin: 15px 0; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Password Reset Request</h1>
        </div>
        <div class="content">
            <h2>Hello {{firstName}},</h2>
            <p>We received a request to reset your password for your {{companyName}} account.</p>
            <p>Click the button below to reset your password:</p>
            <p style="text-align: center;">
                <a href="{{resetLink}}" class="button">Reset Password</a>
            </p>
            <div class="warning">
                <strong>Important:</strong> This link will expire in {{expirationHours}} hours. If you didn't request this password reset, please ignore this email.
            </div>
            <p>For security reasons, we recommend:</p>
            <ul>
                <li>Choose a strong, unique password</li>
                <li>Don't share your password with anyone</li>
                <li>Use two-factor authentication when available</li>
            </ul>
            <p>If you need help, contact us at {{supportEmail}}.</p>
            <p>Best regards,<br>The {{companyName}} Team</p>
        </div>
    </div>
</body>
</html>
```

---

## 🚀 **Advanced Features Implementation**

### Email Rate Limiting Service
```java
import org.springframework.stereotype.Component;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;

@Component
public class EmailRateLimiter {
    
    private final ConcurrentHashMap<String, RateLimitInfo> rateLimits = new ConcurrentHashMap<>();
    private final int maxEmailsPerMinute = 14;
    private final int maxEmailsPerDay = 200;
    
    public boolean canSendEmail(String identifier) {
        RateLimitInfo info = rateLimits.computeIfAbsent(identifier, k -> new RateLimitInfo());
        LocalDateTime now = LocalDateTime.now();
        
        // Reset minute counter if needed
        if (ChronoUnit.MINUTES.between(info.lastMinuteReset, now) >= 1) {
            info.emailsThisMinute.set(0);
            info.lastMinuteReset = now;
        }
        
        // Reset daily counter if needed
        if (ChronoUnit.DAYS.between(info.lastDayReset, now) >= 1) {
            info.emailsToday.set(0);
            info.lastDayReset = now;
        }
        
        // Check limits
        return info.emailsThisMinute.get() < maxEmailsPerMinute && 
               info.emailsToday.get() < maxEmailsPerDay;
    }
    
    public void recordEmailSent(String identifier) {
        RateLimitInfo info = rateLimits.get(identifier);
        if (info != null) {
            info.emailsThisMinute.incrementAndGet();
            info.emailsToday.incrementAndGet();
        }
    }
    
    private static class RateLimitInfo {
        AtomicInteger emailsThisMinute = new AtomicInteger(0);
        AtomicInteger emailsToday = new AtomicInteger(0);
        LocalDateTime lastMinuteReset = LocalDateTime.now();
        LocalDateTime lastDayReset = LocalDateTime.now();
    }
}
```

### Email Event Processor
```java
import org.springframework.stereotype.Service;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

@Service
public class SESEventProcessor {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    public void processEmailEvent(String eventJson) {
        try {
            JsonNode event = objectMapper.readTree(eventJson);
            String eventType = event.get("eventType").asText();
            
            switch (eventType) {
                case "send":
                    processSendEvent(event);
                    break;
                case "bounce":
                    processBounceEvent(event);
                    break;
                case "complaint":
                    processComplaintEvent(event);
                    break;
                case "delivery":
                    processDeliveryEvent(event);
                    break;
                case "open":
                    processOpenEvent(event);
                    break;
                case "click":
                    processClickEvent(event);
                    break;
                default:
                    System.out.println("Unknown event type: " + eventType);
            }
        } catch (Exception e) {
            System.err.println("Failed to process email event: " + e.getMessage());
        }
    }
    
    private void processSendEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        String recipient = event.get("mail").get("destination").get(0).asText();
        System.out.println("Email sent - MessageID: " + messageId + ", To: " + recipient);
        // Update database, analytics, etc.
    }
    
    private void processBounceEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        String bounceType = event.get("bounce").get("bounceType").asText();
        String recipient = event.get("bounce").get("bouncedRecipients").get(0).get("emailAddress").asText();
        
        System.out.println("Email bounced - Type: " + bounceType + ", Recipient: " + recipient);
        
        // Add to suppression list if hard bounce
        if ("Permanent".equals(bounceType)) {
            addToSuppressionList(recipient);
        }
    }
    
    private void processComplaintEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        String recipient = event.get("complaint").get("complainedRecipients").get(0).get("emailAddress").asText();
        
        System.out.println("Email complaint - Recipient: " + recipient);
        
        // Add to suppression list
        addToSuppressionList(recipient);
    }
    
    private void processDeliveryEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        String recipient = event.get("delivery").get("recipients").get(0).asText();
        System.out.println("Email delivered - MessageID: " + messageId + ", To: " + recipient);
    }
    
    private void processOpenEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        System.out.println("Email opened - MessageID: " + messageId);
        // Track engagement metrics
    }
    
    private void processClickEvent(JsonNode event) {
        String messageId = event.get("mail").get("messageId").asText();
        String link = event.get("click").get("link").asText();
        System.out.println("Email link clicked - MessageID: " + messageId + ", Link: " + link);
        // Track click analytics
    }
    
    private void addToSuppressionList(String email) {
        // Add email to suppression list in database
        System.out.println("Adding to suppression list: " + email);
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

---

## 🔍 **Monitoring and Health Checks**

### SES Health Check
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

---

## 📈 **Performance Optimization Tips**

### Batch Processing
```java
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
                    
                    // Small delay to respect rate limits
                    Thread.sleep(75); // ~13 emails per second
                    
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

This comprehensive SES documentation follows the same detailed pattern as your S3 notes, providing both theoretical concepts and practical implementation examples with Spring Boot. The guide covers everything from basic email sending to advanced features like rate limiting, event processing, and analytics.