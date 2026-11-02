# Amazon SQS – Visibility Timeout and Message Processing

> Amazon SQS Visibility Timeout temporarily hides a message from other consumers after it has been retrieved. This allows a consumer to process the message and delete it upon successful completion. If processing fails and the message is not deleted, it becomes visible again after the visibility timeout expires.

---

## 📖 Overview

In the previous lessons, we explored:

- Amazon Simple Queue Service (SQS)
- Decoupled application architectures
- Producer-consumer messaging
- Standard and FIFO queues
- At-least-once message delivery
- Duplicate message handling
- Idempotent applications

We also learned that **consumers are responsible for deleting messages from the queue after successfully processing them**.

But why does Amazon SQS work this way?

What happens when a consumer retrieves a message?

What happens if the consumer fails before processing is complete?

How does Amazon SQS prevent other consumers from retrieving the same message while it is being processed?

These questions introduce an important Amazon SQS feature:

**Visibility Timeout**

---

# 1. Understanding Amazon SQS Message Storage

Amazon SQS is designed to provide highly available and durable message storage.

When producers send messages to an SQS queue, AWS stores the messages redundantly across multiple servers and Availability Zones.

This helps protect messages against infrastructure failures.

### High-Level Architecture

```text
                       Producers
                           |
                           v
                    Amazon SQS Queue
                           |
               +-----------+-----------+
               |           |           |
               v           v           v
          SQS Server A SQS Server B SQS Server C
               |           |           |
               v           v           v
           Message A   Message A   Message A
               |           |           |
               +-----------+-----------+
                           |
                           v
                        Consumers
```

**Note:** This diagram is a conceptual representation of SQS redundancy. AWS manages the underlying infrastructure, and customers do not manage or access individual SQS storage servers.

### Why Does Amazon SQS Store Messages Redundantly?

Consider a scenario where messages are stored on only one server.

```text
Producer
    |
    v
SQS Server
    |
    v
Message A
    |
    v
Server Failure
    |
    v
Message Unavailable
```

If that server fails, the message could become unavailable.

To reduce this risk, Amazon SQS stores messages redundantly across its distributed infrastructure.

### Benefits

- High availability
- Message durability
- Fault tolerance
- Resilience against infrastructure failures

However, this distributed architecture also introduces an important consideration for Standard queues.

**Duplicate message delivery is possible.**

---

# 2. Why Can Duplicate Messages Occur?

Amazon SQS Standard queues provide at-least-once message delivery.

This means that a message may occasionally be delivered more than once.

The lesson explains this using a scenario involving redundant message copies.

### Example

A producer sends:

```text
Message A
```

Amazon SQS stores the message redundantly.

```text
                    Amazon SQS
                        |
             +----------+----------+
             |          |          |
             v          v          v
          Server A   Server B   Server C
             |          |          |
             v          v          v
          Message A  Message A  Message A
```

Now imagine:

1. A consumer retrieves Message A.
2. The consumer successfully processes the message.
3. The consumer sends a request to delete Message A.
4. One of the servers storing a redundant copy is temporarily unavailable.
5. The unavailable server later becomes available again.
6. Message A may become available for delivery again.

### Result

The consumer may receive Message A more than once.

```text
Message A
    |
    v
Consumer Processes Message
    |
    v
Delete Message
    |
    v
Duplicate Message May Reappear
```

This is one reason applications consuming Standard queues should be designed to handle duplicate messages safely.

---

# 3. What Is Amazon SQS Visibility Timeout?

**Visibility Timeout is the period during which a message becomes temporarily invisible to other consumers after being retrieved from an SQS queue.**

When a consumer receives a message, Amazon SQS does not automatically delete it.

Instead, the message becomes temporarily hidden.

During this period, the consumer is expected to:

1. Retrieve the message.
2. Process the message.
3. Complete the required operation.
4. Delete the message from the queue.

### Architecture

```text
                     Producer
                        |
                        v
                  Amazon SQS
                        |
                        v
                     Message A
                        |
                        | ReceiveMessage
                        v
                    Consumer 1
                        |
                        v
                Visibility Timeout
                     Starts
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

### Important Concept

Retrieving a message does not remove it permanently from the queue.

The message is only temporarily hidden during the visibility timeout.

---

# 4. How Visibility Timeout Works

Let's understand the complete message lifecycle.

## Step 1: Producer Sends a Message

An application sends Message A to an SQS queue.

```text
Producer
    |
    v
Amazon SQS
    |
    v
Message A
```

At this stage, Message A is available for consumers to retrieve.

---

## Step 2: Consumer Retrieves the Message

Consumer 1 polls the queue and retrieves Message A.

```text
Amazon SQS
    |
    | ReceiveMessage
    v
Consumer 1
    |
    v
Message A
```

As soon as the message is retrieved, the visibility timeout begins.

---

## Step 3: Message Becomes Invisible

During the visibility timeout, Message A is temporarily hidden from other consumers.

```text
                    Amazon SQS
                        |
                        v
                     Message A
                        |
                        v
                Visibility Timeout
                      Active
                        |
             +----------+----------+
             |                     |
             v                     v
         Consumer 1            Consumer 2
             |                     |
             v                     v
       Processing A          Cannot Normally
                            Retrieve Message A
```

Consumer 1 continues processing the message.

Consumer 2 can retrieve other available messages, but Message A is temporarily unavailable.

**Important:** Visibility timeout reduces concurrent processing of the same message. It is not an absolute guarantee against duplicate delivery, particularly with Standard queues.

---

## Step 4: Consumer Processes the Message

The consumer performs the required business operation.

For example:

- Process a customer order
- Update a database
- Generate a report
- Process an uploaded file
- Perform a background task

### Example

```text
Consumer 1
    |
    v
Retrieve Message A
    |
    v
Process Customer Order
    |
    v
Update Database
    |
    v
Processing Complete
```

The message remains invisible while the visibility timeout is active.

---

## Step 5: Consumer Deletes the Message

After successfully processing Message A, Consumer 1 sends a delete request.

```text
Consumer 1
    |
    v
Processing Successful
    |
    v
DeleteMessage
    |
    v
Amazon SQS
    |
    v
Message A Deleted
```

Deleting the message prevents it from being retrieved again under normal circumstances.

### Why Is Deletion Necessary?

Amazon SQS cannot automatically determine whether the application's business operation completed successfully.

The consumer must explicitly acknowledge successful processing by deleting the message.

---

# 5. Visibility Timeout Timeline

Let's consider an example where the visibility timeout is configured to 30 seconds.

### Example

| Time | Event |
|---|---|
| 0 seconds | Consumer retrieves Message A |
| 0 seconds | Visibility timeout begins |
| 5 seconds | Consumer is processing Message A |
| 15 seconds | Processing continues |
| 25 seconds | Processing completes |
| 25 seconds | Consumer deletes Message A |
| 30 seconds | Original visibility timeout would have expired |

### Timeline

```text
Time
 |
 v
0s -------- 5s -------- 15s -------- 25s -------- 30s
|            |            |            |            |
Receive      Processing   Processing   Processing   Timeout
Message A    Message A    Message A    Complete     Would Expire
                                        |
                                        v
                                   DeleteMessage
                                        |
                                        v
                                   Message Removed
```

Because the consumer deletes the message before the timeout expires, the message does not become visible again through normal timeout expiration.

---

# 6. What Happens If the Consumer Fails?

Now consider a different scenario.

Consumer 1 retrieves Message A but fails before completing the processing.

### Example

```text
Producer
    |
    v
Amazon SQS
    |
    v
Message A
    |
    v
Consumer 1
    |
    v
Processing Started
    |
    v
Consumer Failure
```

Because the consumer failed, it does not send the `DeleteMessage` request.

The message remains temporarily invisible until the visibility timeout expires.

### What Happens Next?

Once the visibility timeout expires:

1. Message A becomes visible again.
2. Another consumer can retrieve it.
3. The new consumer attempts to process the message.

### Architecture

```text
                     Amazon SQS
                         |
                         v
                      Message A
                         |
                         v
                     Consumer 1
                         |
                         v
                  Processing Failed
                         |
                         v
                 Message Not Deleted
                         |
                         v
                Visibility Timeout
                     Expires
                         |
                         v
                 Message A Visible
                         |
                         v
                     Consumer 2
                         |
                         v
                  Process Message A
                         |
                         v
                    DeleteMessage
```

This behavior allows applications to retry processing when a consumer fails.

---

# 7. Consumer Failure Timeline

Let's use the same 30-second visibility timeout.

### Scenario

Consumer 1 retrieves Message A but fails after 15 seconds.

| Time | Event |
|---|---|
| 0 seconds | Consumer 1 retrieves Message A |
| 0 seconds | Visibility timeout begins |
| 5 seconds | Consumer 1 processes Message A |
| 15 seconds | Consumer 1 fails |
| 20 seconds | Message A remains invisible |
| 30 seconds | Visibility timeout expires |
| After 30 seconds | Message A becomes available for another consumer |

### Timeline

```text
0s -------- 5s -------- 15s -------- 20s -------- 30s
|            |            |            |            |
Receive      Processing   Consumer     Message      Timeout
Message A                 Failure      Invisible    Expires
                                                     |
                                                     v
                                               Message Visible
                                                     |
                                                     v
                                                 Consumer 2
                                                     |
                                                     v
                                               Retry Processing
```

### Key Concept

**A message that is not deleted becomes available again after its visibility timeout expires.**

This supports retry processing without requiring the producer to send the message again.

---

# 8. Why Doesn't SQS Delete Messages Automatically?

Consider a scenario where SQS automatically deletes a message immediately after delivering it to a consumer.

```text
Amazon SQS
    |
    v
Deliver Message
    |
    v
Automatically Delete Message
    |
    v
Consumer Starts Processing
    |
    v
Consumer Fails
    |
    v
Message Lost
```

The consumer may fail before completing the business operation.

If the message has already been deleted, another consumer cannot retrieve it for processing.

### SQS Approach

Instead, Amazon SQS separates message retrieval from message deletion.

```text
Amazon SQS
    |
    v
Deliver Message
    |
    v
Hide Message Temporarily
    |
    v
Consumer Processes Message
    |
    +----------------------+
    |                      |
    v                      v
Success                 Failure
    |                      |
    v                      v
Delete Message         No Deletion
                           |
                           v
                    Timeout Expires
                           |
                           v
                     Retry Message
```

This design supports more reliable asynchronous processing.

---

# 9. Successful Processing vs Failed Processing

| Feature | Successful Processing | Failed Processing |
|---|---|---|
| Message Retrieved | Yes | Yes |
| Visibility Timeout Starts | Yes | Yes |
| Message Temporarily Hidden | Yes | Yes |
| Consumer Completes Processing | Yes | No |
| Consumer Deletes Message | Yes | No |
| Message Becomes Visible Again | Normally no | Yes, after timeout |
| Another Consumer Can Retry | Not normally required | Yes |

The main difference is whether the consumer successfully completes processing and deletes the message.

---

# 10. Understanding Message Deletion

Amazon SQS provides the `DeleteMessage` API operation.

Consumers use this operation to remove successfully processed messages.

### Message Processing Workflow

```text
ReceiveMessage
      |
      v
Process Message
      |
      v
Processing Successful?
      |
   +--+--+
   |     |
   v     v
  Yes    No
   |     |
   v     v
Delete   Do Not Delete
Message  Message
   |     |
   v     v
Complete Wait for Timeout
              |
              v
         Retry Processing
```

### Important Technical Detail

When a consumer retrieves a message, SQS provides a receipt handle.

The consumer uses that receipt handle when calling `DeleteMessage`.

A message ID identifies the message, but the receipt handle is required for deletion.

---

# 11. Configuring Visibility Timeout

Amazon SQS allows you to configure the visibility timeout.

The appropriate value depends on how long your application needs to process a message.

### Important Settings

| Setting | Value |
|---|---|
| Default Visibility Timeout | 30 seconds |
| Minimum Visibility Timeout | 0 seconds |
| Maximum Visibility Timeout | 12 hours |

### Example

Suppose a backend worker normally takes 20 seconds to process a message.

A visibility timeout of 30 seconds may be sufficient under normal conditions.

However, if processing occasionally takes longer than 30 seconds, the timeout may expire before the consumer completes its work.

### Potential Problem

```text
Consumer 1
    |
    v
Starts Processing
    |
    v
Processing Takes 60 Seconds
    |
    v
Visibility Timeout Expires at 30 Seconds
    |
    v
Message Becomes Visible Again
    |
    v
Consumer 2 May Retrieve Message
```

Now two consumers may process the same message.

### Recommended Approach

Configure the visibility timeout to accommodate the application's processing requirements.

For operations with variable processing times, applications can extend the timeout while processing is still underway.

---

# 12. Changing Visibility Timeout

Amazon SQS provides the `ChangeMessageVisibility` API operation.

This allows a consumer to modify the visibility timeout for a message that it has retrieved.

### Example

A consumer begins processing a large file.

The original visibility timeout is 30 seconds.

However, processing takes longer than expected.

The consumer can request additional time.

```text
Consumer Retrieves Message
          |
          v
Visibility Timeout: 30 Seconds
          |
          v
Processing Still Running
          |
          v
ChangeMessageVisibility
          |
          v
Additional Processing Time
          |
          v
Processing Completes
          |
          v
DeleteMessage
```

This helps prevent a message from becoming visible before processing is complete.

**Additional AWS concept:** The lecture focuses on the visibility timeout lifecycle. The `ChangeMessageVisibility` operation is included here to connect that concept with practical application design.

---

# 13. Visibility Timeout and Duplicate Processing

Visibility timeout is designed to temporarily hide messages during processing.

However, it does not provide an absolute guarantee that only one consumer will ever receive a message.

This is particularly important for Standard queues.

### Example

```text
Amazon SQS Standard Queue
            |
            v
         Message A
            |
            v
         Consumer 1
            |
            v
      Process Message A
```

A duplicate delivery may still occur because Standard queues provide at-least-once delivery.

Therefore, applications should remain idempotent.

### Why Idempotency Matters

Suppose Message A represents a payment request.

```text
Payment ID: PAY-1001
Amount: $100
```

If the same message is processed more than once, the application should not charge the customer multiple times.

### Idempotent Processing

```text
Receive Payment Message
          |
          v
Check Payment ID
          |
          v
Already Processed?
          |
      +---+---+
      |       |
      v       v
     No      Yes
      |       |
      v       v
Process    Skip Duplicate
Payment
```

**Remember:** Visibility timeout helps coordinate message processing, while idempotency protects against unintended duplicate business operations.

---

# 14. Real-World Example: Order Processing

Consider an e-commerce application that uses Amazon SQS to process customer orders.

### Architecture

```text
                      Customer
                         |
                         v
                  Web Application
                         |
                         v
                    Amazon SQS
                         |
              +----------+----------+
              |                     |
              v                     v
          Consumer 1            Consumer 2
              |                     |
              v                     v
          Process Orders        Process Orders
              |                     |
              +----------+----------+
                         |
                         v
                       Database
```

### Successful Processing

1. A customer places an order.
2. The frontend sends an order message to SQS.
3. Consumer 1 retrieves the message.
4. Visibility timeout begins.
5. Consumer 1 processes the order.
6. The database is updated.
7. Consumer 1 deletes the message.

### Failed Processing

1. A customer places an order.
2. The frontend sends an order message to SQS.
3. Consumer 1 retrieves the message.
4. Visibility timeout begins.
5. Consumer 1 fails before completing processing.
6. The message is not deleted.
7. Visibility timeout expires.
8. Consumer 2 retrieves the message.
9. Consumer 2 retries the order processing.
10. After successful processing, Consumer 2 deletes the message.

### Benefit

A temporary failure of one backend worker does not necessarily cause the order-processing request to be lost.

---

# 15. Visibility Timeout vs Message Retention

These are two different Amazon SQS concepts.

| Feature | Visibility Timeout | Message Retention |
|---|---|---|
| Purpose | Temporarily hides retrieved messages | Determines how long messages can remain in the queue |
| Starts | When a message is received | When a message is added to the queue |
| Default | 30 seconds | 4 days |
| Maximum | 12 hours | 14 days |
| Main Benefit | Supports processing and retries | Allows messages to remain available for later processing |
| Message Deleted Automatically at Expiration? | No | Yes, when retention expires |

### Example

```text
Message Sent to SQS
        |
        v
Message Retention Period Begins
        |
        v
Consumer Retrieves Message
        |
        v
Visibility Timeout Begins
        |
        v
Consumer Processes Message
        |
        +----------------------+
        |                      |
        v                      v
     Success                 Failure
        |                      |
        v                      v
Delete Message         Timeout Expires
                               |
                               v
                         Message Visible
```

**Visibility Timeout = How long a retrieved message remains hidden**

**Message Retention = How long SQS retains a message before expiration**

---

# 16. Common Mistakes

### 1. Assuming ReceiveMessage Deletes the Message

Retrieving a message does not delete it.

The consumer must explicitly delete the message after successful processing.

### 2. Assuming Visibility Timeout Permanently Removes a Message

Visibility timeout only hides a message temporarily.

If the message is not deleted, it becomes visible again when the timeout expires.

### 3. Configuring a Timeout Shorter Than Processing Time

If processing takes longer than the visibility timeout, another consumer may retrieve the message.

### 4. Deleting Messages Before Processing Completes

If a message is deleted before processing finishes and the consumer subsequently fails, the message cannot be retried through normal SQS redelivery.

### 5. Assuming Visibility Timeout Eliminates Duplicate Messages

Visibility timeout reduces concurrent processing but does not eliminate the possibility of duplicate deliveries.

### 6. Ignoring Idempotency

Applications should handle repeated message processing safely.

---

# 17. AWS Certification Exam Scenarios

## Scenario 1: Consumer Fails During Processing

An application retrieves a message from Amazon SQS but crashes before completing processing.

The message is not deleted.

**Question:** What happens next?

**Answer:** After the visibility timeout expires, the message becomes visible again and can be retrieved by another consumer.

---

## Scenario 2: Duplicate Message Processing

An application uses an SQS Standard queue.

Occasionally, the same message is processed more than once.

**Question:** How should the application handle this?

**Answer:** Implement idempotent processing so repeated deliveries do not cause unintended duplicate operations.

---

## Scenario 3: Long-Running Processing

An application requires 10 minutes to process a message, but the queue visibility timeout is configured to 30 seconds.

**Question:** What problem could occur?

**Answer:** The message may become visible again before processing completes, allowing another consumer to retrieve it.

**Solution:** Configure an appropriate visibility timeout or extend it during processing.

---

## Scenario 4: Message Deletion

A consumer successfully processes an SQS message.

**Question:** What should happen next?

**Answer:** The consumer should call `DeleteMessage` using the appropriate receipt handle.

---

## Scenario 5: Improving Application Resilience

An application uses multiple backend workers to process SQS messages.

One worker fails while processing a message.

**Question:** How does visibility timeout help?

**Answer:** The message becomes available for another worker after the timeout expires, provided it was not deleted or expired.

---

# ❓ Interview Questions

### Q1. What is Amazon SQS Visibility Timeout?

It is the period during which a retrieved message is temporarily hidden from other consumers.

### Q2. What happens when a consumer retrieves a message?

The visibility timeout begins, and the message becomes temporarily invisible to other consumers.

### Q3. Does receiving a message automatically delete it?

No. The consumer must explicitly delete the message.

### Q4. Why does Amazon SQS require consumers to delete messages?

Because SQS cannot determine whether the application's business processing completed successfully.

### Q5. What happens if a consumer fails before deleting a message?

The message becomes visible again after the visibility timeout expires.

### Q6. What is the default visibility timeout?

30 seconds.

### Q7. What is the maximum visibility timeout?

12 hours.

### Q8. What happens if processing takes longer than the visibility timeout?

The message may become visible again and be retrieved by another consumer.

### Q9. Can visibility timeout be changed?

Yes. The `ChangeMessageVisibility` API can modify the timeout for a retrieved message.

### Q10. Does visibility timeout guarantee exactly-once processing?

No. Applications should still handle duplicate deliveries safely.

### Q11. Why can SQS Standard queues deliver duplicate messages?

Because SQS uses distributed, redundant message storage and provides at-least-once delivery.

### Q12. What is the purpose of the DeleteMessage API?

It allows consumers to delete messages after successful processing.

### Q13. What is a receipt handle?

A value returned when a message is received that is used to identify that receive operation when deleting the message.

### Q14. What is the difference between visibility timeout and message retention?

Visibility timeout controls how long a retrieved message remains hidden, while message retention controls how long SQS stores a message.

### Q15. What happens if the visibility timeout expires before the consumer deletes the message?

The message becomes visible again and may be retrieved by another consumer.

### Q16. Why is idempotency important?

It ensures repeated processing of the same message does not cause unintended additional effects.

### Q17. How does visibility timeout improve application resilience?

It allows messages that were not successfully processed to become available for retry.

### Q18. Should consumers delete messages before or after processing?

Consumers should delete messages after successful processing.

---

# 💡 Key Takeaways

- Amazon SQS stores messages redundantly across its distributed infrastructure.
- Standard queues provide at-least-once message delivery.
- Duplicate message deliveries are possible.
- Visibility timeout begins when a consumer retrieves a message.
- During the visibility timeout, the message is temporarily hidden from other consumers.
- Retrieving a message does not automatically delete it.
- Consumers must explicitly delete messages after successful processing.
- If a consumer fails, the message remains in the queue.
- When the visibility timeout expires, an undeleted message becomes visible again.
- Another consumer can retrieve and retry the message.
- The default visibility timeout is 30 seconds.
- The maximum visibility timeout is 12 hours.
- Visibility timeout should accommodate application processing time.
- Consumers can extend visibility timeout when additional processing time is required.
- Visibility timeout does not guarantee exactly-once processing.
- Idempotent application design remains important.
- Visibility timeout and message retention are different features.

---

## The Most Important Concept to Remember

```text
                       Producer
                          |
                          v
                      Amazon SQS
                          |
                          v
                       Message A
                          |
                          v
                       Consumer
                          |
                          v
                  Visibility Timeout
                       Begins
                          |
                          v
                    Process Message
                          |
                 +--------+--------+
                 |                 |
                 v                 v
              Success           Failure
                 |                 |
                 v                 v
           DeleteMessage     Message Not Deleted
                 |                 |
                 v                 v
           Message Removed   Timeout Expires
                                   |
                                   v
                             Message Visible
                                   |
                                   v
                              Retry Processing
```

**ReceiveMessage = Retrieve a message and begin its visibility timeout**

**Visibility Timeout = Temporarily hide the retrieved message**

**DeleteMessage = Remove a successfully processed message**

**Consumer Failure = Message becomes available again after timeout expiration**

---

# 📚 Related Topics

- Amazon Simple Queue Service (SQS)
- SQS Standard Queues
- SQS FIFO Queues
- Producer-Consumer Architecture
- At-Least-Once Delivery
- Idempotent Applications
- Message Polling
- Message Retention
- Amazon SQS Dead-Letter Queues
- AWS Application Integration
