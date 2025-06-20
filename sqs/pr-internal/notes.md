

## ✅ Core SQS Operations (Common Across Sync & Async)

### 🔹 Message Producer Operations

1. **Send a single message**

   * Used to send one message to a queue.
   * Parameters: `queueUrl`, `messageBody`, optional attributes.

2. **Send batch of messages**

   * Send up to **10 messages at once**.
   * Useful for reducing network overhead and improving throughput.

---

### 🔹 Message Consumer Operations

3. **Receive messages (single or batch)**

   * Pull up to **10 messages** per call.
   * `waitTimeSeconds` used for long polling.
   * Messages stay "invisible" during the visibility timeout.

4. **Delete a message**

   * Done after successful processing.
   * Requires `receiptHandle` of the message.

5. **Delete batch of messages**

   * Efficient way to delete multiple messages after batch processing.

---

### 🔹 Other Useful Operations

6. **Change message visibility**

   * Extend or reduce the visibility timeout of a specific message.

7. **Purge queue**

   * Deletes **all** messages in a queue.
   * Dangerous in production, used in dev/test setups.

8. **Get queue attributes**

   * Useful for getting queue size, delay settings, DLQ config, etc.

9. **List queues**

   * List all queues in your account/region.

---

## 🔁 Sync vs Async — Summary

| Feature              | Sync (e.g., `SqsClient`)         | Async (e.g., `SqsAsyncClient`)             |
| -------------------- | -------------------------------- | ------------------------------------------ |
| Execution Style      | Blocking                         | Non-blocking (returns `CompletableFuture`) |
| Thread Usage         | Your thread is blocked           | AWS manages threads under the hood         |
| Throughput           | Moderate (good for simpler apps) | High throughput & scale                    |
| Use Cases            | Simpler apps, quick demos        | Reactive apps, high concurrency            |
| Operations Supported | Same operations as async         | Same as sync                               |

---

## 📌 Example Mapping (Operation Summary)

| Operation Type       | Sync Method                 | Async Method                     |
| -------------------- | --------------------------- | -------------------------------- |
| Send message         | `sendMessage()`             | `sendMessageAsync()`             |
| Send batch           | `sendMessageBatch()`        | `sendMessageBatchAsync()`        |
| Receive message      | `receiveMessage()`          | `receiveMessageAsync()`          |
| Delete message       | `deleteMessage()`           | `deleteMessageAsync()`           |
| Delete batch         | `deleteMessageBatch()`      | `deleteMessageBatchAsync()`      |
| Change visibility    | `changeMessageVisibility()` | `changeMessageVisibilityAsync()` |
| Get queue attributes | `getQueueAttributes()`      | `getQueueAttributesAsync()`      |

---
