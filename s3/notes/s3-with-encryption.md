
<br/>
<br/>

```java
s3Client.putObject(new PutObjectRequest(bucketName, fileName, fileObj));
```

**uploads the file to S3 without any encryption.** By default, this means your files are stored in plaintext on Amazon S3.

---

### 🔐 To store files **with encryption**, Amazon S3 offers three main types:

---

###  **1. SSE-S3 (Server-Side Encryption with Amazon S3-Managed Keys)**

Amazon S3 handles everything (key creation, storage, rotation). Easiest to implement.

```java
PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSEAwsKeyManagementParams(new SSEAwsKeyManagementParams()); // Optional for SSE-S3
request.withMetadata(new ObjectMetadata()); // optional

request.getMetadata().setSSEAlgorithm(ObjectMetadata.AES_256); // enable SSE-S3

s3Client.putObject(request);
```

OR just:

```java
ObjectMetadata metadata = new ObjectMetadata();
metadata.setSSEAlgorithm(ObjectMetadata.AES_256); // AES256

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj);
request.setMetadata(metadata);

s3Client.putObject(request);
```

---

###  **2. SSE-KMS (Server-Side Encryption with AWS KMS-managed keys)**

You manage the key in AWS Key Management Service (KMS). More control, better audit logging.

```java
SSEAwsKeyManagementParams sseParams = new SSEAwsKeyManagementParams("your-kms-key-id");

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSEAwsKeyManagementParams(sseParams);

s3Client.putObject(request);
```

Replace `"your-kms-key-id"` with:

* KMS Key ID
* Alias (e.g., `alias/my-key`)
* Or ARN

---

###  **3. SSE-C (Server-Side Encryption with Customer-Provided Keys)**

You provide the encryption key with each request. AWS never stores your key.

```java
SSECustomerKey sseKey = new SSECustomerKey("your-secret-key"); // should be 256-bit

PutObjectRequest request = new PutObjectRequest(bucketName, fileName, fileObj)
        .withSSECustomerKey(sseKey);

s3Client.putObject(request);
```

🔒⚠️ You are responsible for key management (rotation, backup).

---

###  **4. Client-Side Encryption (Before Upload)**

Encrypt data in your app before sending to S3. Complete control.

* Use libraries like [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html) or Bouncy Castle.
* You upload the already encrypted file as-is.

Example using AWS Encryption SDK (Java):

```java
// Not simple; needs AWS Encryption SDK setup
```

---

###  Summary:

| Encryption Type | Managed By | Java SDK Option                    | Best Use Case                      |
| --------------- | ---------- | ---------------------------------- | ---------------------------------- |
| SSE-S3          | AWS        | `.setSSEAlgorithm("AES256")`       | Simple and default                 |
| SSE-KMS         | AWS KMS    | `.withSSEAwsKeyManagementParams()` | Audit, fine-grained control        |
| SSE-C           | You        | `.withSSECustomerKey()`            | You handle keys securely           |
| Client-Side     | You        | Pre-encrypt file                   | Max control (e.g., sensitive data) |

---
<br/>
<br/>


### 1. **Without Encryption (Default – as you have now)**

```java
public String uploadFile(MultipartFile file) {
    File fileObj = convertMultiPartFileToFile(file);
    String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();
    s3Client.putObject(new PutObjectRequest(bucketName, fileName, fileObj));
    fileObj.delete();
    return "File uploaded : " + fileName;
}
```

---

### 2. **SSE-S3 (Server-Side Encryption with Amazon S3-Managed Keys)**

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

---

### 3. **SSE-KMS (Server-Side Encryption with AWS KMS-Managed Keys)**

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

---

### 4. **SSE-C (Server-Side Encryption with Customer-Provided Keys)**

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

The `base64EncodedKey` should be a 256-bit key encoded in Base64.

---

### 5. **Client-Side Encryption (Encrypt Before Upload)**

This approach requires an encryption library. Here's a conceptual example using Java's built-in AES encryption:

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