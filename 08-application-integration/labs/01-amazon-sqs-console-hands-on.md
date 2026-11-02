# Amazon SQS – Hands-On Lab: Creating and Testing an SQS Standard Queue

> **Lab Objective:** Create an Amazon SQS Standard queue using the AWS Management Console and understand how messages are sent, received, temporarily hidden during visibility timeout, and deleted after processing.

---

## 📖 Overview

In the previous lessons, we explored the fundamental concepts of Amazon Simple Queue Service (SQS), including:

- Standard and FIFO queues
- Producer-consumer architecture
- Message polling
- Visibility timeout
- Message retention
- Dead-Letter Queues (DLQs)
- Redrive policies
- Delay queues

In this hands-on lab, we will use the AWS Management Console to demonstrate how Amazon SQS works.

We will:

1. Create an SQS Standard queue.
2. Review the available queue configuration options.
3. Send a test message.
4. Poll the queue to retrieve the message.
5. Observe visibility timeout behavior.
6. Retrieve the message again after the timeout expires.
7. Observe changes in the message receive count.
8. Delete the message.
9. Verify that the queue no longer contains the message.

This lab uses the AWS Management Console rather than actual producer and consumer applications.

---

# 1. Lab Architecture

In a production environment, applications communicate with Amazon SQS using AWS SDKs or APIs.

For this demonstration, we will use the AWS Management Console to simulate producer and consumer activities.

### Architecture

```text
                    AWS Management Console
                              |
                 +------------+------------+
                 |                         |
                 v                         v
          Send Message                Poll Messages
                 |                         |
                 v                         |
           Producer Action                 |
                 |                         |
                 v                         |
          +--------------------+           |
          |                    |           |
          |     Amazon SQS     |<----------+
          |                    |
          |   Standard Queue   |
          |                    |
          +--------------------+
                    |
                    v
              Receive Message
                    |
                    v
             Visibility Timeout
                    |
                    v
              Delete Message
```

### Lab Components

| Component | Description |
|---|---|
| AWS Region | US East (N. Virginia) |
| AWS Service | Amazon SQS |
| Queue Type | Standard |
| Queue Name | `ritual-roast-order-queue` |
| Visibility Timeout | 30 seconds |
| Delivery Delay | 0 seconds |
| Message Retention | 4 days |
| Test Message | `Hello ritual roast coffee` |
| Producer | AWS Management Console |
| Consumer | AWS Management Console |

**Note:** The queue name above is a descriptive normalization of the name spoken in the demonstration. Use the exact name shown in your AWS Console if it differs.

---

# 2. Prerequisites

Before starting this lab, ensure that you have:

- An AWS account
- Access to the AWS Management Console
- Permissions to create and manage SQS queues
- Basic knowledge of Amazon SQS

### Required Permissions

The AWS identity performing this lab needs permissions for operations such as:

```text
sqs:CreateQueue
sqs:GetQueueAttributes
sqs:SendMessage
sqs:ReceiveMessage
sqs:DeleteMessage
sqs:DeleteQueue
```

The exact permissions depend on your organization's IAM configuration.

---

# 3. Step 1: Open Amazon SQS

1. Sign in to the AWS Management Console.
2. Select the AWS Region.
3. Search for **SQS**.
4. Open **Amazon Simple Queue Service**.
5. Navigate to the SQS console.

### Region Used in the Demonstration

```text
US East (N. Virginia)
```

### Navigation

```text
AWS Management Console
          |
          v
      Search: SQS
          |
          v
      Amazon SQS
          |
          v
       Queues Page
```

The SQS console allows you to create queues and manage their configuration.

---

# 4. Step 2: Create an SQS Queue

From the Amazon SQS console:

1. Click **Create queue**.
2. Select the queue type.
3. Enter a queue name.
4. Review the configuration settings.
5. Create the queue.

### Queue Type

Select:

**Standard**

Standard queues provide:

- At-least-once message delivery
- Best-effort message ordering
- Very high throughput

### Queue Name

For this lab, use:

```text
ritual-roast-order-queue
```

### Configuration

| Setting | Value |
|---|---|
| Queue Type | Standard |
| Queue Name | `ritual-roast-order-queue` |
| Visibility Timeout | 30 seconds |
| Delivery Delay | 0 seconds |
| Message Retention | 4 days |

These settings are sufficient for the basic demonstration.

---

# 5. Step 3: Review Visibility Timeout

During queue creation, locate the **Visibility timeout** setting.

### Default Value

```text
30 seconds
```

### Maximum Value

```text
12 hours
```

Visibility timeout determines how long a retrieved message remains temporarily hidden from other consumers.

### Example

```text
Amazon SQS
    |
    v
Message A Available
    |
    v
Consumer Retrieves Message
    |
    v
Visibility Timeout: 30 Seconds
    |
    v
Message Temporarily Hidden
```

If the consumer does not delete the message before the timeout expires, it becomes visible again.

### Lab Configuration

Keep the default value:

```text
VisibilityTimeout = 30 seconds
```

We will observe this behavior later in the demonstration.

---

# 6. Step 4: Review Delivery Delay

The next setting is **Delivery delay**.

This controls how long newly added messages remain unavailable to consumers.

### Configuration Range

| Setting | Value |
|---|---|
| Minimum Delay | 0 seconds |
| Maximum Delay | 15 minutes |

### Lab Configuration

```text
DeliveryDelay = 0 seconds
```

This means messages become eligible for retrieval immediately after being sent.

### Architecture

```text
Producer
    |
    v
Amazon SQS
    |
    v
Delivery Delay: 0 Seconds
    |
    v
Message Available
```

We are not enabling delayed delivery in this lab.

---

# 7. Step 5: Review Message Retention

The next setting is **Message retention period**.

Message retention determines how long Amazon SQS stores messages before they expire.

### Configuration

| Setting | Value |
|---|---|
| Default Retention | 4 days |
| Maximum Retention | 14 days |

### Lab Configuration

```text
MessageRetentionPeriod = 4 days
```

### Example

```text
Producer Sends Message
          |
          v
      Amazon SQS
          |
          v
Message Retained in Queue
          |
          v
Maximum Configured Retention
          |
          v
Message Expires if Not Deleted
```

For this lab, the default four-day retention period is sufficient.

---

# 8. Step 6: Review Message Size

The instructor discusses the maximum size of messages that can be sent to Amazon SQS.

### Course Demonstration

The lecture describes the message size limit as:

```text
256 KB
```

### Current AWS Service Note

Amazon SQS subsequently increased its maximum message size to **1 MiB**.

This is an updated service detail beyond the limit described in the demonstration.

### Handling Large Files

Amazon SQS is not intended to store large objects such as video files.

Instead:

1. Upload the file to Amazon S3.
2. Send a message containing the S3 object location.
3. Let the consumer retrieve the file from S3.

### Architecture

```text
                    Producer
                       |
             +---------+---------+
             |                   |
             v                   v
         Amazon S3           Amazon SQS
         Large File          S3 Reference
             |                   |
             |                   v
             |                Consumer
             |                   |
             +---------+---------+
                       |
                       v
                 Process File
```

This approach is commonly used for large-file processing applications.

---

# 9. Step 7: Review Encryption Settings

Amazon SQS supports server-side encryption.

The instructor demonstrates that encryption is enabled by default in the console configuration.

### Encryption Options

Amazon SQS supports:

- SQS-managed encryption keys
- AWS KMS-managed encryption keys

### Comparison

| Encryption Option | Description |
|---|---|
| SQS-managed encryption | Amazon SQS manages encryption keys |
| AWS KMS encryption | Uses AWS Key Management Service |

### Lab Configuration

Keep the default encryption configuration shown in the console.

### Important Concept

Encryption helps protect messages stored in Amazon SQS.

For production workloads, encryption requirements should align with organizational security policies.

---

# 10. Step 8: Review Access Policies

Access policies determine which identities can interact with an SQS queue.

For example:

- Who can send messages?
- Who can receive messages?
- Who can delete messages?
- Which AWS accounts can access the queue?

### Example Architecture

```text
                 AWS IAM
                    |
          +---------+---------+
          |                   |
          v                   v
     Producer Role       Consumer Role
          |                   |
          v                   v
    sqs:SendMessage     sqs:ReceiveMessage
                              |
                              v
                        sqs:DeleteMessage
          |                   |
          +---------+---------+
                    |
                    v
                Amazon SQS
```

### Production Considerations

Applications should receive only the permissions they require.

For example:

**Producer Permissions**

```text
sqs:SendMessage
```

**Consumer Permissions**

```text
sqs:ReceiveMessage
sqs:DeleteMessage
```

### Security Best Practice

Use the principle of least privilege.

Avoid unnecessarily broad permissions in production environments.

The instructor leaves the demonstration settings unchanged because this is a basic console lab.

---

# 11. Step 9: Review Redrive Policy and DLQ Settings

The SQS queue creation page also provides configuration options related to Dead-Letter Queues.

### Redrive Policy

A redrive policy defines when messages that repeatedly fail processing should be moved to a DLQ.

Important parameters include:

```text
deadLetterTargetArn
maxReceiveCount
```

### Example

```text
Source SQS Queue
        |
        v
Consumer Processing
        |
        v
Repeated Failures
        |
        v
Receive Count Exceeded
        |
        v
Dead-Letter Queue
```

### Lab Configuration

For this demonstration:

```text
Dead-Letter Queue = Not Configured
```

We are focusing on message delivery, visibility timeout, and deletion.

### Important Distinction

The console also includes a **redrive allow policy**, which controls which source queues may use a queue as a DLQ.

This differs from the source queue's **redrive policy**, which identifies the DLQ and receive threshold.

---

# 12. Step 10: Create the Queue

After reviewing the configuration:

1. Scroll to the bottom of the page.
2. Click **Create queue**.
3. Wait for the queue to be created.

### Expected Result

The queue should appear in the SQS console.

```text
Queue Name:
ritual-roast-order-queue

Queue Type:
Standard

Status:
Created Successfully
```

### Architecture

```text
AWS Management Console
          |
          v
      Create Queue
          |
          v
     Standard Queue
          |
          v
ritual-roast-order-queue
          |
          v
    Queue Created
```

The queue is now ready to receive messages.

---

# 13. Step 11: Open Send and Receive Messages

After creating the queue:

1. Open the queue details page.
2. Click **Send and receive messages**.
3. Locate the message body field.

This page allows you to simulate producer and consumer operations.

### Console Workflow

```text
Amazon SQS
    |
    v
Select Queue
    |
    v
Send and Receive Messages
    |
    +-------------------+
    |                   |
    v                   v
Send Message       Poll Messages
```

---

# 14. Step 12: Send a Test Message

Enter the following message body:

```text
Hello ritual roast coffee
```

Click:

**Send message**

### Producer Workflow

```text
AWS Management Console
          |
          v
      SendMessage
          |
          v
Message Body:
"Hello ritual roast coffee"
          |
          v
      Amazon SQS
```

### Expected Result

The message is successfully sent to the queue.

The console should indicate that a message is available.

### Message Status

```text
Messages Available: 1
```

This confirms that the queue has received the message.

---

# 15. Step 13: Poll the Queue

Now we will simulate a consumer retrieving the message.

### Steps

1. Navigate to the **Receive messages** section.
2. Click **Poll for messages**.
3. Wait for the polling operation to complete.

### Expected Result

The console retrieves the message.

```text
Message Body:
Hello ritual roast coffee
```

### Architecture

```text
                  Amazon SQS
                      |
                      v
                Message Available
                      |
                      v
                 Poll Messages
                      |
                      v
                 ReceiveMessage
                      |
                      v
               Message Retrieved
```

### Important Concept

The message has been retrieved, but it has not been deleted.

This means it still exists in the queue.

---

# 16. Step 14: Observe Visibility Timeout

After retrieving the message, the visibility timeout begins.

For this lab:

```text
VisibilityTimeout = 30 seconds
```

During this period, the message is temporarily unavailable for another retrieval.

### Architecture

```text
Amazon SQS
    |
    v
Message Retrieved
    |
    v
Visibility Timeout Begins
    |
    v
Message Temporarily Hidden
    |
    v
Messages Available: 0
```

### What Happens If We Poll Again?

If another polling request occurs while the visibility timeout is active, the same message is normally not returned.

### Expected Result

```text
Messages Available: 0
```

The message is not permanently deleted.

It is temporarily hidden.

---

# 17. Step 15: Wait for Visibility Timeout to Expire

The instructor waits for the 30-second visibility timeout to expire.

### Timeline

```text
0s -------- 10s -------- 20s -------- 30s
|            |            |            |
Message      Message      Message      Timeout
Retrieved    Hidden       Hidden       Expires
                                          |
                                          v
                                   Message Visible
```

Once the timeout expires, the message becomes available again.

### Steps

1. Wait approximately 30 seconds after the message was retrieved.
2. Click **Poll for messages** again.
3. Observe the returned message.

### Expected Result

The same message becomes available again.

```text
Message Body:
Hello ritual roast coffee
```

### Why?

The message was retrieved but never deleted.

Therefore, SQS makes it available again after the visibility timeout expires.

---

# 18. Step 16: Observe the Receive Count

Amazon SQS tracks how many times a message has been received.

This is called the receive count.

The instructor demonstrates that the receive count increases each time the same message is retrieved.

### First Retrieval

```text
Receive Count = 1
```

### Second Retrieval

```text
Receive Count = 2
```

### Third Retrieval

```text
Receive Count = 3
```

### Timeline

| Attempt | Action | Receive Count |
|---|---|---|
| 1 | First message retrieval | 1 |
| 2 | Message retrieved after timeout | 2 |
| 3 | Message retrieved again after timeout | 3 |

### Architecture

```text
                    Amazon SQS
                        |
                        v
                     Message A
                        |
                        v
                   Receive Count 1
                        |
                        v
                Visibility Timeout
                        |
                        v
                   Receive Count 2
                        |
                        v
                Visibility Timeout
                        |
                        v
                   Receive Count 3
```

### Important Concept

The receive count increases when the message is received again.

It does not mean the message has been processed successfully.

This concept is also important when configuring Dead-Letter Queues.

---

# 19. Step 17: Delete the Message

In a real-world application, the consumer should delete the message after successful processing.

For this lab, we will delete the message manually.

### Steps

1. Poll the queue and retrieve the message.
2. Select the message.
3. Click **Delete**.
4. Confirm the deletion.

### Architecture

```text
Amazon SQS
    |
    v
Receive Message
    |
    v
Process Message
    |
    v
DeleteMessage
    |
    v
Message Removed
```

### Expected Result

The console confirms that the message was deleted successfully.

### Important Concept

**Receiving a message does not delete it.**

A separate delete operation is required.

---

# 20. Step 18: Verify Message Deletion

After deleting the message:

1. Wait for any current polling operation to finish.
2. Refresh the queue details if necessary.
3. Click **Poll for messages** again.

### Expected Result

No messages should be returned.

```text
Messages Available: 0

Messages Retrieved: 0
```

### Architecture

```text
Amazon SQS
    |
    v
Message Deleted
    |
    v
Poll for Messages
    |
    v
No Messages Returned
```

This confirms that the message is no longer available in the queue.

---

# 21. Complete Lab Workflow

```text
                     AWS Console
                         |
                         v
                    Amazon SQS
                         |
                         v
                  Create Standard Queue
                         |
                         v
                Configure Queue Settings
                         |
                         v
                    Create Queue
                         |
                         v
                 Send Test Message
                         |
                         v
                  Message Available
                         |
                         v
                   Poll for Message
                         |
                         v
                  Message Retrieved
                         |
                         v
                Visibility Timeout
                         |
                         v
                 Message Temporarily
                       Hidden
                         |
                         v
                 Wait for 30 Seconds
                         |
                         v
                  Message Available
                         |
                         v
                   Poll Again
                         |
                         v
                Receive Count Increases
                         |
                         v
                   Delete Message
                         |
                         v
                  Poll for Messages
                         |
                         v
                 No Messages Returned
```

---

# 22. Lab Validation Checklist

| Validation Step | Expected Result |
|---|---|
| Create Standard queue | Queue created successfully |
| Send test message | Message accepted |
| Check available messages | One message available |
| Poll queue | Message retrieved |
| Check message body | Correct message displayed |
| Poll during visibility timeout | Message temporarily unavailable |
| Wait for timeout | Message becomes visible again |
| Retrieve message again | Receive count increases |
| Delete message | Message deleted successfully |
| Poll after deletion | No messages returned |

### Lab Result

The demonstration successfully illustrates the Amazon SQS message lifecycle.

---

# 23. Understanding the Message Lifecycle

The key operations in this lab are:

1. `SendMessage`
2. `ReceiveMessage`
3. Visibility timeout
4. `DeleteMessage`

### Architecture

```text
Producer
    |
    | SendMessage
    v
Amazon SQS
    |
    | ReceiveMessage
    v
Consumer
    |
    v
Visibility Timeout
    |
    v
Process Message
    |
    | DeleteMessage
    v
Amazon SQS
    |
    v
Message Removed
```

### Message States

| State | Description |
|---|---|
| Available | Message can be retrieved |
| In Flight | Message has been retrieved and is temporarily hidden |
| Available Again | Visibility timeout expired without deletion |
| Deleted | Message removed after successful processing |

---

# 24. Important AWS Certification Exam Points

Remember the following:

1. Amazon SQS supports Standard and FIFO queues.
2. Standard queues provide at-least-once delivery.
3. Standard queues provide best-effort ordering.
4. The default visibility timeout is 30 seconds.
5. The maximum visibility timeout is 12 hours.
6. The default delivery delay is zero seconds.
7. The maximum delivery delay is 15 minutes.
8. The default message retention period is four days.
9. The maximum message retention period is fourteen days.
10. Retrieving a message does not automatically delete it.
11. Messages become temporarily invisible during visibility timeout.
12. Undeleted messages become visible again after the timeout expires.
13. Receive count increases when messages are retrieved again.
14. Consumers must explicitly delete messages after successful processing.
15. Dead-Letter Queues can isolate repeatedly failing messages.

---

# 25. Common Mistakes

### 1. Assuming Message Retrieval Means Successful Processing

Retrieving a message only means the consumer has received it.

The application must still process it.

### 2. Assuming Messages Are Automatically Deleted

Amazon SQS does not automatically delete messages after retrieval.

The consumer must explicitly delete them.

### 3. Confusing Visibility Timeout with Message Retention

Visibility timeout temporarily hides messages after retrieval.

Message retention determines how long SQS stores messages.

### 4. Assuming Zero Available Messages Means the Queue Is Empty

A message may be temporarily hidden during visibility timeout.

Therefore, zero available messages does not necessarily mean all messages have been deleted.

### 5. Ignoring Receive Count

Receive count helps identify messages that have been retrieved multiple times.

This is useful for understanding retries and configuring Dead-Letter Queues.

### 6. Using Broad Access Policies in Production

Production applications should use appropriately scoped IAM permissions and queue policies.

---

# 26. AWS Certification Exam Scenarios

## Scenario 1: Message Reappears After 30 Seconds

An application retrieves a message from an SQS queue but does not delete it.

The queue uses the default visibility timeout.

**Question:** What happens after approximately 30 seconds?

**Answer:** The message becomes visible again and can be retrieved by another consumer.

---

## Scenario 2: Receive Count Increases

An application retrieves the same message multiple times without deleting it.

**Question:** What happens to the receive count?

**Answer:** The receive count increases each time the message is received.

---

## Scenario 3: Message Successfully Processed

A consumer successfully processes an SQS message.

**Question:** What should the consumer do next?

**Answer:** Call `DeleteMessage` using the appropriate receipt handle.

---

## Scenario 4: Queue Shows Zero Available Messages

An administrator notices that an SQS queue shows zero available messages immediately after a consumer retrieves a message.

**Question:** Does this mean the message has been deleted?

**Answer:** No. The message may be temporarily hidden during its visibility timeout.

---

## Scenario 5: Long-Running Processing

A consumer needs several minutes to process a message.

The visibility timeout is configured to 30 seconds.

**Question:** What should be done?

**Answer:** Configure an appropriate visibility timeout or extend it during processing.

---

# ❓ Interview Questions

### Q1. What is the purpose of Amazon SQS?

Amazon SQS is a fully managed message queuing service used to decouple application components and support asynchronous processing.

### Q2. What are the two types of SQS queues?

Standard and FIFO.

### Q3. Which queue type was used in this lab?

Standard.

### Q4. What delivery guarantee does a Standard queue provide?

At-least-once delivery.

### Q5. What is the default visibility timeout?

30 seconds.

### Q6. What happens when a consumer retrieves a message?

The visibility timeout begins, and the message becomes temporarily hidden.

### Q7. Does receiving a message automatically delete it?

No.

### Q8. What happens if a message is not deleted?

It becomes visible again after the visibility timeout expires.

### Q9. What is the purpose of DeleteMessage?

It removes a message from the queue after successful processing.

### Q10. What is the receive count?

It indicates how many times a message has been received.

### Q11. What happens to the receive count when the same message is retrieved again?

The count increases.

### Q12. What is the default message retention period?

Four days.

### Q13. What is the maximum message retention period?

Fourteen days.

### Q14. What is the maximum delivery delay?

15 minutes.

### Q15. What is the maximum visibility timeout?

12 hours.

### Q16. Can messages be encrypted in Amazon SQS?

Yes. Amazon SQS supports server-side encryption.

### Q17. How can access to an SQS queue be controlled?

Using IAM permissions and SQS queue access policies.

### Q18. Why is visibility timeout important?

It temporarily hides retrieved messages while consumers process them and supports retry processing if messages are not deleted.

---

# 💡 Key Takeaways

- Amazon SQS can be configured and tested through the AWS Management Console.
- Standard queues provide at-least-once delivery and best-effort ordering.
- Queue settings include visibility timeout, delivery delay, and message retention.
- The default visibility timeout is 30 seconds.
- The default delivery delay is zero seconds.
- The default message retention period is four days.
- Producers send messages using `SendMessage`.
- Consumers retrieve messages using `ReceiveMessage`.
- Retrieved messages become temporarily invisible during visibility timeout.
- Messages that are not deleted become visible again after the timeout expires.
- The receive count increases when messages are retrieved multiple times.
- Consumers should delete messages after successful processing.
- `DeleteMessage` removes messages from the queue.
- Queue access should be controlled using appropriate IAM permissions and policies.
- Amazon SQS supports server-side encryption.
- Dead-Letter Queues can be configured to isolate repeatedly failing messages.

---

## The Most Important Concept to Remember

```text
                         Producer
                            |
                            v
                       SendMessage
                            |
                            v
                       Amazon SQS
                            |
                            v
                     Message Available
                            |
                            v
                       ReceiveMessage
                            |
                            v
                         Consumer
                            |
                            v
                     Visibility Timeout
                            |
                    +-------+-------+
                    |               |
                    v               v
                 Success          Failure
                    |               |
                    v               v
              DeleteMessage     No Deletion
                    |               |
                    v               v
              Message Removed   Timeout Expires
                                    |
                                    v
                              Message Available
                                    |
                                    v
                              Receive Count +1
```

**SendMessage = Producer sends a message to the queue**

**ReceiveMessage = Consumer retrieves a message**

**Visibility Timeout = Temporarily hides the retrieved message**

**Receive Count = Tracks how many times a message has been received**

**DeleteMessage = Permanently removes a successfully processed message**

---

# 📚 Related Topics

- Amazon Simple Queue Service (SQS)
- SQS Standard Queues
- SQS FIFO Queues
- Visibility Timeout
- Message Retention
- Delay Queues
- Dead-Letter Queues
- Redrive Policies
- IAM Permissions
- SQS Queue Policies
- Amazon CloudWatch

