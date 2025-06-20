


### 📦 `S3Utils.java` – Method Descriptions for README

| Method Name                                                 | Description                                                                           |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `validateFile(MultipartFile file)`                          | Validates the uploaded file for size, content type, filename, and allowed extensions. |
| `validateKeyName(String keyName)`                           | Ensures the S3 object key is not null, malformed, or too long.                        |
| `generateUniqueKey(String prefix, String originalFilename)` | Generates a unique S3 key using timestamp, UUID, and original file extension.         |
| `getContentType(MultipartFile file)`                        | Returns the MIME type of the file or guesses it based on the file extension.          |
| `getFileExtension(String filename)`                         | Extracts the file extension from a given filename.                                    |
| `sanitizeFilename(String filename)`                         | Cleans the filename by replacing special characters and trimming extra underscores.   |
| `generatePresignedUrlKey(String keyName)`                   | Converts spaces to '+' in key names to match presigned URL encoding format.           |
| `formatFileSize(long bytes)`                                | Converts file size in bytes to a human-readable format (e.g., MB, GB).                |
| `extractFolderPath(String key)`                             | Extracts the folder path from a given S3 key.                                         |
| `extractFilename(String key)`                               | Retrieves the file name from a given S3 key.                                          |
| `isDangerousExtension(String extension)`                    | Checks if the file extension is in the list of potentially harmful types. *(Private)* |
| `getContentTypeFromExtension(String extension)`             | Maps file extensions to standard content types. *(Private)*                           |

