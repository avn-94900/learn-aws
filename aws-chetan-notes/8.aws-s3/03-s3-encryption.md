# 03 - S3 Encryption

S3 supports encryption both **at rest** (stored data) and **in transit** (data moving over the network).

---

## Encryption in Transit

All S3 API calls are encrypted using **HTTPS / TLS** by default.

- **HTTPS/TLS** — Encrypts all data between client and S3
- **VPC Endpoints** — Private connectivity to S3 without internet exposure
- **AWS PrivateLink** — Secure connection through the AWS backbone network

---

## Encryption at Rest

S3 offers four encryption options for data stored in buckets.

| Type        | Managed By | Best Use Case                                |
|-------------|------------|----------------------------------------------|
| SSE-S3      | AWS        | Default; simple with no additional setup     |
| SSE-KMS     | AWS KMS    | Audit logging, fine-grained key control      |
| SSE-C       | You        | You fully control and manage the keys        |
| Client-Side | You        | Maximum control; encrypt before upload       |

---

## SSE-S3 — Server-Side Encryption with S3-Managed Keys

AWS handles everything: key creation, storage, and rotation. Uses AES-256.

This is the simplest option and has no additional cost beyond S3 storage.

```java
// Upload with SSE-S3 encryption (AWS SDK v1)
ObjectMetadata metadata = new ObjectMetadata();
metadata.setSSEAlgorithm(ObjectMetadata.AES_256);

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj);
request.setMetadata(metadata);

s3Client.putObject(request);
```

---

## SSE-KMS — Server-Side Encryption with AWS KMS Keys

You create and manage encryption keys in AWS Key Management Service (KMS). This provides detailed audit logging via AWS CloudTrail and allows fine-grained access control over who can use the key.

```java
// Upload with SSE-KMS encryption (AWS SDK v1)
SSEAwsKeyManagementParams sseKmsParams = new SSEAwsKeyManagementParams("your-kms-key-id");

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSEAwsKeyManagementParams(sseKmsParams);

s3Client.putObject(request);
```

The `kmsKeyId` can be:
- A KMS Key ID
- A key alias (e.g., `alias/my-key`)
- A full key ARN

---

## SSE-C — Server-Side Encryption with Customer-Provided Keys

You provide the encryption key with each request. AWS uses your key to encrypt/decrypt the object but **never stores your key**.

> You are fully responsible for key management, rotation, and backup.

```java
// Upload with SSE-C encryption (AWS SDK v1)
byte[] decodedKey = Base64.getDecoder().decode(base64EncodedKey);
SSECustomerKey sseCustomerKey = new SSECustomerKey(decodedKey);

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSECustomerKey(sseCustomerKey);

s3Client.putObject(request);
```

The key must be a **256-bit key encoded in Base64**.

---

## Client-Side Encryption — Encrypt Before Upload

You encrypt the data in your application before sending it to S3. AWS has no knowledge of your encryption keys or the plaintext data.

```java
// Encrypt file using AES before uploading (Java)
public File encryptFile(File inputFile, SecretKey secretKey) throws Exception {
    Cipher cipher = Cipher.getInstance("AES");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);

    File encryptedFile = new File("encrypted_" + inputFile.getName());
    try (FileInputStream fis = new FileInputStream(inputFile);
         FileOutputStream fos = new FileOutputStream(encryptedFile);
         CipherOutputStream cos = new CipherOutputStream(fos, cipher)) {

        byte[] buffer = new byte[1024];
        int read;
        while ((read = fis.read(buffer)) != -1) {
            cos.write(buffer, 0, read);
        }
    }
    return encryptedFile;
}

public String uploadEncryptedFile(MultipartFile file, SecretKey secretKey) throws Exception {
    File fileObj = convertMultiPartFileToFile(file);
    File encryptedFile = encryptFile(fileObj, secretKey);
    String fileName = System.currentTimeMillis() + "_encrypted_" + file.getOriginalFilename();

    s3Client.putObject(new PutObjectRequest(bucketName, fileName, encryptedFile));

    fileObj.delete();
    encryptedFile.delete();
    return "Encrypted file uploaded: " + fileName;
}
```

---

## Upload Examples with Each Encryption Type (Spring Boot / SDK v1)

### Without Encryption (Default)

```java
public String uploadFile(MultipartFile file) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();
    s3Client.putObject(new PutObjectRequest(bucketName, fileName, fileObj));
    fileObj.delete();
    return "File uploaded: " + fileName;
}
```

### With SSE-S3

```java
public String uploadFileWithSSES3(MultipartFile file) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();

    ObjectMetadata metadata = new ObjectMetadata();
    metadata.setSSEAlgorithm(ObjectMetadata.AES_256);

    PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj);
    request.setMetadata(metadata);

    s3Client.putObject(request);
    fileObj.delete();
    return "File uploaded with SSE-S3: " + fileName;
}
```

### With SSE-KMS

```java
public String uploadFileWithSSEKMS(MultipartFile file, String kmsKeyId) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();

    SSEAwsKeyManagementParams sseKmsParams = new SSEAwsKeyManagementParams(kmsKeyId);

    PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
            .withSSEAwsKeyManagementParams(sseKmsParams);

    s3Client.putObject(request);
    fileObj.delete();
    return "File uploaded with SSE-KMS: " + fileName;
}
```

Calling it:

```java
uploadFileWithSSEKMS(file, "alias/my-kms-key");
```

### With SSE-C

```java
public String uploadFileWithSSEC(MultipartFile file, String base64EncodedKey) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();

    byte[] decodedKey = Base64.getDecoder().decode(base64EncodedKey);
    SSECustomerKey sseCustomerKey = new SSECustomerKey(decodedKey);

    PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
            .withSSECustomerKey(sseCustomerKey);

    s3Client.putObject(request);
    fileObj.delete();
    return "File uploaded with SSE-C: " + fileName;
}
```

---

## Comparison Summary

| Encryption Type | Who Manages Keys | Java SDK Method               | Best Use Case                        |
|-----------------|------------------|-------------------------------|--------------------------------------|
| SSE-S3          | AWS              | `.setSSEAlgorithm("AES256")`  | Simple default encryption            |
| SSE-KMS         | AWS KMS          | `.withSSEAwsKeyManagementParams()` | Audit logging, fine-grained control |
| SSE-C           | You              | `.withSSECustomerKey()`       | You fully control key lifecycle      |
| Client-Side     | You              | Pre-encrypt before upload     | Maximum control, highly sensitive data |
