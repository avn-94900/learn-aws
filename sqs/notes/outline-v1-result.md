### Amazon SQS — Key Concepts & “need‑to‑know” Notes (bullet‑point style)

---

#### 1 . What SQS is

* **Fully‑managed message‑queuing service** that decouples producers and consumers, auto‑scales, and is pay‑per‑request. Standard queues give *at‑least‑once* delivery with best‑effort ordering; FIFO queues add *exactly‑once* processing and strict order guarantees. ([docs.aws.amazon.com][1])

---

#### 2 . Queue types & throughput

* **Standard queue** → virtually unlimited TPS, at‑least‑once, best‑effort order.
* **FIFO queue** → guarantees order within a *message group* and exactly‑once delivery; throughput caps depend on mode:

  * *Classic* FIFO: up to 300 TPS per API call.
  * *High‑throughput FIFO* (2021) now supports **70 000 TPS per API action** (700 000 msg/s with batching) in some Regions (Nov 2023). ([aws.amazon.com][2], [aws.amazon.com][3])

---

#### 3 . Core queue settings & mechanics

| Concept                             | Why it matters                                                              | Key numbers / notes                                                                                |
| ----------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Visibility timeout**              | Hides message while a consumer is processing it; prevents duplicate workers | 0–12 h (default 30 s). Tune to > max processing time. ([docs.aws.amazon.com][4])                   |
| **Message retention**               | How long SQS stores un‑deleted msgs                                         | 60 s – 14 d (default 4 d).                                                                         |
| **Delay queue / per‑message delay** | Postpone delivery for *all* or individual messages                          | Up to 15 min.                                                                                      |
| **Message size**                    | Direct payload cap                                                          | 1 B – 256 KB; use **SQS Extended Client** to off‑load larger payloads to S3/DynamoDB.              |
| **Long polling**                    | Reduces empty receives & costs                                              | `WaitTimeSeconds` up to 20 s.                                                                      |
| **Batch APIs**                      | Higher efficiency, lower cost                                               | Send/receive/delete up to 10 msgs per batch; combined with high‑throughput FIFO yields huge scale. |
| **In‑flight limit**                 | Back‑pressure safety                                                        | 120 000 in‑flight msgs (standard) / 20 000 (FIFO) per queue.                                       |
| **Purging**                         | Deletes *all* msgs (irreversible)                                           | 60‑s safety cooldown after a purge.                                                                |

---

#### 4 . Reliability & error handling

* **Dead‑letter queues (DLQ)**: isolate “poison” messages after *MaxReceiveCount*; must be same queue type (standard→standard, FIFO→FIFO). ([aws.amazon.com][5])
* **Redrive to source**: since 2022 you can push DLQ messages back for re‑processing via console/API (`StartMessageMoveTask`). ([aws.amazon.com][6], [docs.aws.amazon.com][7])
* **Monitoring**: CloudWatch metrics like *ApproximateNumberOfMessages*, *AgeOfOldestMessage*; set alarms on queue length & age to auto‑scale workers. ([docs.aws.amazon.com][4])

---

#### 5 . Security & networking

* **IAM & queue policies**: fine‑grained *SendMessage*, *ReceiveMessage*, *PurgeQueue*, etc.
* **Encryption at rest**: SSE with KMS (AWS‑managed or CMK). ([aws.amazon.com][5])
* **In‑transit**: HTTPS endpoints.
* **VPC interface endpoints (PrivateLink)** remove internet traffic; **IPv6 API endpoints added Apr 2025**. ([aws.amazon.com][8])

---

#### 6 . Service integrations & patterns

* **Lambda event source mapping** (auto‑poll, batches, DLQ hand‑off).
* **SNS → SQS fan‑out** (push‑based to multiple queues).
* **EventBridge pipelines**, **Step Functions**, **Kinesis** → use for choreography vs. queuing.
* **Elastic Beanstalk worker tiers** use SQS out of the box. ([docs.aws.amazon.com][9])

---

#### 7 . Spring Boot / Java cheat‑sheet

| Area                     | How                                                                                                            |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **SDK**                  | `software.amazon.awssdk:sqs` v2; create `SqsClient`/`SqsAsyncClient`.                                          |
| **Spring Cloud AWS 3.x** | `spring-cloud-aws-starter-sqs`; annotation‑driven listeners (`@SqsListener`), auto‑DLQ, FIFO group‑id mapping. |
| **JMS compatibility**    | `amazon-sqs-java-messaging-lib` exposes JMS 1.1 API for lift‑and‑shift of legacy code.                         |
| **Local dev**            | Use *LocalStack* or AWS SAM to emulate SQS.                                                                    |

---

#### 8 . Cost model (2025 prices, *check Region*)

* **Requests**: per 1 million *API* calls (send, receive, delete, etc.).
* **Data transfer**: intra‑Region **\$0**; cross‑Region standard AWS egress fees.
* **FIFO premium**: same price per request, but high‑throughput mode incurs quota‑increase request if > 70 k TPS.

---

#### 9 . Best‑practice checklist

1. **Keep messages idempotent & small** (≤ 256 KB, ideally ≤ 64 KB for Lambda).
2. **Right‑size visibility timeout** + use *ChangeMessageVisibility* if work takes longer.
3. **Enable DLQ** and set alarms on DLQ depth.
4. **Use batching & long polling** to cut costs.
5. **Scale consumers on queue length / age** (e.g., K8s HPA or Lambda concurrency).
6. **For ordered workloads** use FIFO + *message‑group‑id*; shard groups to parallelise.
7. **Avoid tight loops** on empty receives; prefer long polling or event sources.
8. **Secure queues** with least‑privilege IAM, KMS keys, and VPC endpoints when needed.
9. **Automate queue creation** via IaC (CloudFormation, CDK, Terraform) with clear names & tags.
10. **Test DLQ redrive paths** regularly; script redrive with `StartMessageMoveTask`.

---

#### 10 . Frequently‑asked interview points

* Difference between **Standard vs FIFO** and use cases.
* **Visibility timeout** vs **message retention**.
* **Long polling** advantages.
* **DLQ workflow** & redrive.
* How **exactly‑once** is achieved in FIFO (deduplication ID + MD5).
* Handling **large messages** with S3 pointer pattern.
* **Scaling** consumers vs. SQS throughput limits.
* **Security**: SSE, IAM policies, VPC endpoints.
* **Spring Boot integration** patterns (`@SqsListener`, batching, DLQ config).

Use this list as your quick‑reference sheet when designing, coding, or explaining Amazon SQS.

[1]: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-types.html?utm_source=chatgpt.com "Amazon SQS queue types - Amazon Simple Queue Service"
[2]: https://aws.amazon.com/blogs/aws/announcing-throughput-increase-and-dead-letter-queue-redrive-support-for-amazon-sqs-fifo-queues/?utm_source=chatgpt.com "Announcing throughput increase and dead letter queue redrive ..."
[3]: https://aws.amazon.com/about-aws/whats-new/2023/11/?utm_source=chatgpt.com "November 2023 - AWS"
[4]: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/features-capabilities.html?utm_source=chatgpt.com "Amazon SQS features and capabilities - AWS Documentation"
[5]: https://aws.amazon.com/sqs/features/?utm_source=chatgpt.com "Amazon SQS Features | Message Queuing Service - AWS"
[6]: https://aws.amazon.com/blogs/compute/introducing-amazon-simple-queue-service-dead-letter-queue-redrive-to-source-queues/?utm_source=chatgpt.com "Introducing Amazon Simple Queue Service dead-letter queue ... - AWS"
[7]: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-dead-letter-queue-redrive.html?utm_source=chatgpt.com "Learn how to configure a dead-letter queue redrive in Amazon SQS"
[8]: https://aws.amazon.com/about-aws/whats-new/2025/04/amazon-sqs-internet-protocol-version-6/?utm_source=chatgpt.com "Amazon SQS now supports Internet Protocol Version 6 (IPv6) - AWS"
[9]: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options-general.html?utm_source=chatgpt.com "General options for all environments - AWS Elastic Beanstalk"
