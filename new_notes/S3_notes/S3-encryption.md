# S3 Encryption Guide

Complete guide to encrypting data in Amazon S3 using different methods.

---

## Default Upload (No Encryption)

```java
s3Client.putObject(new PutObjectRequest(bucketName, fileName, fileObj));
```

This uploads the file to S3 without any encryption. By default, files are stored in plaintext on Amazon S3.

---

## Encryption Types

Amazon S3 offers three main types of server-side encryption:

| Encryption Type | Managed By | Java SDK Option | Best Use Case |
| --- | --- | --- | --- |
| SSE-S3 | AWS | .setSSEAlgorithm("AES256") | Simple and default |
| SSE-KMS | AWS KMS | .withSSEAwsKeyManagementParams() | Audit, fine-grained control |
| SSE-C | You | .withSSECustomerKey() | You handle keys securely |
| Client-Side | You | Pre-encrypt file | Max control (e.g., sensitive data) |

---

## SSE-S3 (Server-Side Encryption with Amazon S3-Managed Keys)

Amazon S3 handles everything (key creation, storage, rotation). Easiest to implement.

### Method 1

```java
PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSEAwsKeyManagementParams(new SSEAwsKeyManagementParams());
request.withMetadata(new ObjectMetadata());

request.getMetadata().setSSEAlgorithm(ObjectMetadata.AES_256);

s3Client.putObject(request);
```

### Method 2

```java
ObjectMetadata metadata = new ObjectMetadata();
metadata.setSSEAlgorithm(ObjectMetadata.AES_256); // AES256

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj);
request.setMetadata(metadata);

s3Client.putObject(request);
```

---

## SSE-KMS (Server-Side Encryption with AWS KMS-managed keys)

You manage the key in AWS Key Management Service (KMS). More control, better audit logging.

```java
SSEAwsKeyManagementParams sseParams = new SSEAwsKeyManagementParams("your-kms-key-id");

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSEAwsKeyManagementParams(sseParams);

s3Client.putObject(request);
```

Replace "your-kms-key-id" with:

- KMS Key ID
- Alias (e.g., alias/my-key)
- Or ARN

---

## SSE-C (Server-Side Encryption with Customer-Provided Keys)

You provide the encryption key with each request. AWS never stores your key.

```java
SSECustomerKey sseKey = new SSECustomerKey("your-secret-key"); // should be 256-bit

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSECustomerKey(sseKey);

s3Client.putObject(request);
```

**Warning**: You are responsible for key management (rotation, backup).

---

## Client-Side Encryption (Before Upload)

Encrypt data in your app before sending to S3. Complete control.

- Use libraries like AWS Encryption SDK or Bouncy Castle
- You upload the already encrypted file as-is

---

## Complete Upload Examples

### Without Encryption (Default)

```java
public String uploadFile(MultipartFile file) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();
    s3Client.putObject(new PutObjectRequest(bucketName, fileName, fileObj));
    fileObj.delete();
    return "File uploaded : " + fileName;
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

Pass the KMS Key ID or Alias when calling the method:

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

The base64EncodedKey should be a 256-bit key encoded in Base64.

### Client-Side Encryption

```java
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
    return "Encrypted file uploaded (Client-Side): " + fileName;
}
```
