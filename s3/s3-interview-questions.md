
## S3-related interview questions for a Spring Boot developer:



###  **Basic-Level Questions**

1. **What is Amazon S3? What are its main features?**

   * Explain S3 as an object storage service and features like durability, scalability, and access control.

2. **How do you upload a file to S3 using Spring Boot?**

   * Mention using AWS SDK (`AmazonS3`, `TransferManager`, `PutObjectRequest`, etc.) and multipart upload if applicable.

3. **How do you download a file from S3 in Spring Boot?**

   * Mention `AmazonS3.getObject()` and streaming the file to the response.

4. **How do you list all files in an S3 bucket or folder?**

   * Mention `listObjectsV2()` or `listObjects()`.

5. **What permissions or policies are required for an application to access S3?**

   * Explain IAM roles, bucket policies, and least privilege principle.

---

###  **Spring Boot Integration Specific**

6. **How do you configure AWS credentials and region in a Spring Boot application?**

   * Via `application.properties` or `application.yml` using:

     ```properties
     cloud.aws.credentials.access-key=
     cloud.aws.credentials.secret-key=
     cloud.aws.region.static=
     ```

7. **How do you define an `AmazonS3` bean in your configuration class?**

   * Using `BasicAWSCredentials` and `AmazonS3ClientBuilder`.

8. **What is the difference between using `AmazonS3` and `AmazonS3Client`?**

9. **How do you handle file size limits when uploading to S3?**

   * Multipart upload, validation at controller level.

10. **How would you upload a file to S3 using `RestController` and `MultipartFile`?**

* Upload via `MultipartFile` → convert to `InputStream` → use `AmazonS3.putObject()`.

---

###  **Intermediate to Advanced Questions**

11. **How do you secure file uploads to S3 from a Spring Boot application?**

* Mention signed URLs (pre-signed URLs), encryption, and access policies.

12. **What is a pre-signed URL and how do you generate it using AWS SDK in Spring Boot?**

13. **How do you ensure uploaded files are not overwritten in the bucket?**

* Use unique keys (UUIDs, timestamps, or folders).

14. **How do you integrate S3 with a Spring Boot microservice for file storage?**

* Show service-level abstraction with S3 client and separation of concerns.

15. **How do you handle versioning in S3 from your Spring Boot app?**

16. **How do you delete files from S3 programmatically in Spring Boot?**

17. **What are the ways to serve S3 files securely in a Spring Boot app?**

* Signed URLs, CloudFront with signed cookies/URLs.

18. **How do you monitor and handle upload failures or retries to S3?**
> https://chatgpt.com/share/684bd99e-2260-8011-b0a5-815efe8de9c8

---

###  **Scenario-Based Questions**

19. **You need to allow users to upload profile pictures. How would you implement this using S3 in Spring Boot?**

* Involves file validation, unique file naming, S3 service, and response with file URL.

20. **How would you restrict access to S3 files per user in a multi-tenant system?**

* Folder per user, pre-signed URLs, access control lists.

21. **Your application handles large files (e.g., 1GB videos). How will you handle uploading to S3 efficiently?**

* Multipart upload, progress tracking, TransferManager.

---

### **Bonus / DevOps Angle**

22. **How do you test S3 integration in a local Spring Boot development environment?**

* Use [**LocalStack**](https://github.com/localstack/localstack).

23. **What are the alternatives to storing files in S3? When would you not use S3?**

24. **How do you configure lifecycle policies on a bucket for automated cleanup or transition to Glacier?**

25. **Have you used SDK v2 vs v1? What are the differences?**

---

If you’d like, I can also provide a **sample Spring Boot file upload/download code using Amazon S3** or a **mock interview sheet with answers** to help you prepare better.
