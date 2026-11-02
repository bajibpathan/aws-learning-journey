# Amazon SQS – Standard vs FIFO Queues

> Amazon Simple Queue Service (SQS) supports two types of queues: Standard and First-In-First-Out (FIFO). Both help decouple application components, but they differ in message ordering, delivery guarantees, and throughput.

---

## 📖 Overview

In the previous lesson, we learned how Amazon SQS helps build decoupled, scalable, and resilient applications.

We explored:

- Producer-consumer architecture
- Asynchronous message processing
- Message queues and polling
- Independent frontend and backend scaling
- Message retention
- Video-processing applications using Amazon SQS and Amazon S3

Now that we understand the fundamentals of Amazon SQS, let's explore the two queue types.

Amazon SQS supports:

1. **Standard Queues**
2. **FIFO (First-In-First-Out) Queues**

The primary differences involve:

- Message ordering
- Message delivery guarantees
- Throughput
- Duplicate message handling
- Application use cases

Understanding these differences is important when designing distributed applications and preparing for the AWS Certified Solutions Architect – Associate examination.

---

# 1. Amazon SQS Queue Types

Amazon SQS provides two queue types for different application requirements.

```text
                    Amazon SQS
                        |
              +---------+---------+
              |                   |
              v                   v
         Standard Queue       FIFO Queue
              |                   |
              v                   v
        High Throughput      Strict Ordering
              |                   |
              v                   v
       At-Least-Once       FIFO Delivery with
          Delivery          Deduplication
```

The appropriate queue type depends on the application's requirements.

For example:

- If message processing order is not critical, a Standard queue may be suitable.
- If messages must be processed in a particular sequence, a FIFO queue may be more appropriate.

---

# 2. Amazon SQS Standard Queue

A **Standard queue** is the default queue type in Amazon SQS.

It is designed to support highly scalable message processing.

Standard queues offer very high throughput and can support a nearly unlimited number of API requests per second.

Common API operations include:

- `SendMessage`
- `ReceiveMessage`
- `DeleteMessage`

### Architecture

```text
                    Producers
                        |
                        v
              ┌───────────────────┐
              │  SQS Standard     │
              │      Queue        │
              │                   │
              │   Message A       │
              │   Message B       │
              │   Message C       │
              └───────────────────┘
                        |
                        v
                    Consumers
                        |
                        v
                 Process Messages
```

Standard queues are suitable for applications that require high throughput but do not require strict message ordering.

---

# 3. Standard Queue Message Ordering

One important characteristic of Standard queues is that **message ordering is not guaranteed**.

Messages may occasionally be delivered in a different order from the order in which they were sent.

### Example

Suppose a producer sends three messages:

```text
Producer Sends Messages

Message 1
    |
    v
Message 2
    |
    v
Message 3
```

The expected processing order might be:

```text
Message 1 → Message 2 → Message 3
```

However, a Standard queue may deliver them as:

```text
Message 2 → Message 1 → Message 3
```

### Why Does This Happen?

Amazon SQS is a distributed service.

Messages are stored across multiple servers to provide redundancy and high availability.

Because of this distributed architecture, messages may occasionally be delivered out of order.

### Important Point

If your application requires messages to be processed in the exact order in which they were sent, a Standard queue may not be appropriate.

---

# 4. Standard Queue Delivery Guarantee

Amazon SQS Standard queues provide **at-least-once message delivery**.

This means a message is delivered at least once, but duplicate deliveries are possible.

### Example

A producer sends a message:

```text
Order ID: 1001
```

The consumer may receive:

```text
Message 1 → Order ID: 1001
```

However, under certain circumstances, the same message may be delivered again.

```text
Message 1 → Order ID: 1001

Message 1 → Order ID: 1001
```

This is known as duplicate message delivery.

### Why Can Duplicate Messages Occur?

Amazon SQS stores redundant copies of messages across multiple servers.

Consider the following scenario:

1. A producer sends a message to an SQS Standard queue.
2. Amazon SQS stores redundant copies of the message.
3. A consumer retrieves and processes the message.
4. The consumer deletes the message.
5. One of the servers storing a copy is temporarily unavailable during deletion.
6. When that server becomes available again, the message may be delivered another time.

### Architecture

```text
                 Amazon SQS
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Server A   Server B   Server C
          |          |          |
          |          |          |
          +----------+----------+
                     |
                     v
                  Consumer
                     |
                     v
               Process Message
                     |
                     v
                Delete Message
```

If one server is temporarily unavailable during the deletion process, a redundant copy may remain.

This is one reason Standard queues provide at-least-once delivery rather than exactly-once delivery.

---

# 5. Understanding Idempotency

Because Standard queues can deliver duplicate messages, applications should be designed to be **idempotent**.

### What Is Idempotency?

Idempotency means that processing the same operation multiple times does not produce unintended additional effects.

### Example: Order Processing

Suppose a customer places an order.

```text
Order ID: 1001
Amount: $100
```

The backend receives the message and processes the order.

If the same message is delivered again, the application should recognize that the order has already been processed.

### Without Idempotency

```text
Message Received
       |
       v
Process Order
       |
       v
Charge Customer $100
       |
       v
Duplicate Message Received
       |
       v
Charge Customer Another $100
```

**Result:** The customer may be charged twice.

### With Idempotency

```text
Message Received
       |
       v
Check Order ID
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
Process   Skip Duplicate
Order
```

The application can use a unique identifier to recognize previously processed requests.

### Key Concept

**Idempotent applications can safely handle duplicate message deliveries without producing unintended duplicate business operations.**

This is especially important for:

- Payment processing
- Order management
- Inventory updates
- Database transactions

---

# 6. Standard Queue Use Cases

The lesson introduces several common use cases for Standard queues.

## Use Case 1: Background Processing

A frontend application accepts requests and sends time-consuming operations to a backend service.

```text
Users
  |
  v
Frontend Application
  |
  v
SQS Standard Queue
  |
  v
Backend Workers
  |
  v
Process Requests
```

For example, users may upload files that require additional processing.

The frontend does not need to wait for the processing to finish.

## Use Case 2: Distributing Tasks Across Multiple Workers

Standard queues can distribute work among multiple consumers.

```text
                  SQS Standard Queue
                          |
               +----------+----------+
               |          |          |
               v          v          v
            Worker 1   Worker 2   Worker 3
```

Each worker retrieves available messages and processes them.

This supports parallel processing.

## Use Case 3: Batch Processing

Applications can place messages in a queue for processing later.

```text
Application
     |
     v
SQS Standard Queue
     |
     v
Batch Processing Workers
```

### Standard Queue Summary

| Feature | Standard Queue |
|---|---|
| Default Queue Type | Yes |
| Throughput | Very high |
| Message Ordering | Best effort |
| Delivery Guarantee | At least once |
| Duplicate Messages | Possible |
| Idempotency | Recommended |
| Common Use Cases | Background processing, distributed workloads, batch processing |

---

# 7. Amazon SQS FIFO Queue

FIFO stands for **First-In-First-Out**.

FIFO queues are designed for applications where message ordering and duplicate prevention are important.

The lesson highlights two key features:

1. Messages are delivered in order.
2. Duplicate messages are prevented through FIFO deduplication.

### Architecture

```text
                    Producer
                       |
                       v
                SQS FIFO Queue
                       |
                       v
                   Message 1
                       |
                       v
                   Message 2
                       |
                       v
                   Message 3
                       |
                       v
                    Consumer
```

Unlike Standard queues, FIFO queues preserve message ordering within a message group.

---

# 8. FIFO Message Ordering

FIFO queues preserve the order of messages within the same message group.

### Example

Suppose an application sends three messages:

```text
Message 1 → Register Account
Message 2 → Enroll in Course
Message 3 → Make Payment
```

The messages must be processed in the correct sequence.

```text
Register Account
       |
       v
Enroll in Course
       |
       v
Make Payment
```

Processing these operations out of order could cause application errors.

### Incorrect Processing Order

```text
Enroll in Course
       |
       v
Register Account
       |
       v
Make Payment
```

The application may attempt to enroll a student before the account exists.

### FIFO Solution

A FIFO queue helps preserve the required sequence for messages belonging to the same message group.

**Important:** FIFO ordering is guaranteed within a message group. Messages in different groups can be processed independently.

---

# 9. FIFO Exactly-Once Processing

The lecture describes FIFO queues as providing exactly-once processing.

For technical accuracy, it is important to distinguish **exactly-once message processing** from **FIFO deduplication**.

FIFO queues provide exactly-once processing support by preventing duplicate messages from being introduced into the queue during the deduplication interval.

However, consumers may still receive a message again if processing fails or the message is not deleted successfully.

Therefore, applications should still be designed to handle retries safely.

### Example

A producer sends:

```text
Order ID: 1001
```

If the same message is submitted again with the same deduplication identifier within the deduplication interval, FIFO deduplication prevents it from being added as a separate message.

### Conceptual Architecture

```text
Producer
   |
   v
Send Message
   |
   v
SQS FIFO Queue
   |
   v
Check Deduplication
   |
   +-------------------+
   |                   |
   v                   v
New Message        Duplicate Message
   |                   |
   v                   v
Accept Message     Suppress Duplicate
```

### Key Concept

FIFO queues provide deduplication and ordered delivery, but application-level idempotency remains a useful safeguard.

---

# 10. FIFO Queue Throughput

The lesson introduces the following FIFO throughput limits:

| Operation | Throughput |
|---|---|
| Without Batching | 300 transactions per second |
| With Batching | 3,000 messages per second |

These figures describe the traditional FIFO throughput model discussed in the course.

Modern SQS FIFO queues also support high-throughput configurations, with quotas that depend on the AWS Region and configuration.

### Why Is FIFO Different?

FIFO queues provide additional ordering and deduplication capabilities.

These capabilities introduce constraints that are different from those of Standard queues.

### Standard vs FIFO

```text
                Amazon SQS
                    |
          +---------+---------+
          |                   |
          v                   v
     Standard Queue       FIFO Queue
          |                   |
          v                   v
     Very High           Ordering and
     Throughput          Deduplication
```

For certification preparation, understand the trade-off between ordering requirements and throughput characteristics.

---

# 11. FIFO Queue Use Cases

## Use Case 1: E-Commerce Order Management

An e-commerce application may need to process order-related events in sequence.

For example:

```text
Order Created
      |
      v
Payment Confirmed
      |
      v
Inventory Updated
      |
      v
Order Shipped
```

If these operations must be processed in order for a particular order, FIFO messaging can help maintain that sequence.

## Use Case 2: Student Registration

Consider an educational application.

A student must register an account before enrolling in a course.

```text
Register Student Account
          |
          v
Enroll Student in Course
          |
          v
Confirm Enrollment
```

A FIFO queue can preserve the order of messages associated with the student's registration workflow.

## Use Case 3: Ordered User Inputs

Some applications require user actions to be processed in the order they were submitted.

For example:

```text
Update 1
   |
   v
Update 2
   |
   v
Update 3
```

A FIFO queue helps maintain this order within the relevant message group.

---

# 12. Standard vs FIFO Queues

The following table summarizes the main differences.

| Feature | Standard Queue | FIFO Queue |
|---|---|---|
| Default Queue Type | Yes | No |
| Message Ordering | Best effort | Preserved within message groups |
| Delivery Guarantee | At least once | Ordered delivery with deduplication |
| Duplicate Delivery | Possible | Deduplication supported; retries remain possible |
| Throughput | Very high | Depends on FIFO configuration |
| Message Groups | Not required | Required |
| Idempotent Consumers | Recommended | Still recommended |
| Background Processing | Excellent | Supported |
| Order-Sensitive Workloads | Not ideal | Recommended |
| Typical Use Cases | Video processing, batch jobs, background tasks | Order management, sequential workflows |

---

# 13. Choosing Between Standard and FIFO

The decision depends primarily on whether message ordering is important.

### Choose Standard Queues When:

- Message ordering is not critical.
- Very high throughput is required.
- Duplicate messages can be handled safely.
- Tasks can be processed independently.
- Multiple workers need to process messages concurrently.

### Choose FIFO Queues When:

- Message ordering is critical.
- Related operations must follow a specific sequence.
- Message deduplication is required.
- Business workflows depend on ordered events.

### Decision Flow

```text
                Application Requirement
                         |
                         v
              Is Message Order Critical?
                         |
                  +------+------+
                  |             |
                  v             v
                 No            Yes
                  |             |
                  v             v
           Standard Queue    FIFO Queue
                  |             |
                  v             v
           High Throughput   Ordered Messages
```

---

# 14. Real-World Architecture Examples

## Example 1: Video Processing with Standard SQS

Video-processing tasks are often independent.

A video uploaded by one user does not necessarily need to be processed before a video uploaded by another user.

```text
                       Users
                         |
                         v
                    Frontend
                         |
                         v
                    Amazon S3
                         |
                         v
                  Standard SQS
                         |
              +----------+----------+
              |          |          |
              v          v          v
           Worker 1   Worker 2   Worker 3
              |          |          |
              +----------+----------+
                         |
                         v
                 Processed Videos
```

**Recommended Queue:** Standard

**Reason:** The workload benefits from high throughput and parallel processing without requiring strict message ordering.

## Example 2: Order Processing with FIFO SQS

An order-processing application may need to preserve the sequence of events for each order.

```text
                    E-Commerce App
                           |
                           v
                       FIFO SQS
                           |
                           v
                     Order Created
                           |
                           v
                    Payment Confirmed
                           |
                           v
                    Inventory Updated
                           |
                           v
                       Order Shipped
```

**Recommended Queue:** FIFO

**Reason:** Related order events must be processed in the correct sequence.

---

# 15. Common Mistakes

### 1. Assuming Standard Queues Preserve Message Order

Standard queues provide best-effort ordering.

Messages may occasionally arrive out of order.

### 2. Assuming Standard Queues Deliver Each Message Only Once

Standard queues provide at-least-once delivery.

Duplicate deliveries are possible.

### 3. Ignoring Idempotency

Applications should handle duplicate messages safely, particularly when processing payments or database updates.

### 4. Assuming FIFO Guarantees Exactly-Once Business Transactions

FIFO deduplication reduces duplicate message insertion, but application failures and retries can still occur.

Consumers should still handle retries safely.

### 5. Assuming FIFO Ordering Applies Across All Message Groups

FIFO ordering is maintained within each message group.

Different groups can be processed independently.

### 6. Selecting FIFO for Every Application

FIFO provides useful ordering capabilities, but not every application requires them.

Standard queues are often suitable for independent background-processing workloads.

---

# 16. AWS Certification Exam Scenarios

## Scenario 1: High-Throughput Background Processing

A company needs to process millions of independent image-processing requests.

Message ordering is not important.

**Recommended Solution:** Amazon SQS Standard Queue.

**Reason:** Standard queues support very high throughput and independent message processing.

## Scenario 2: Order-Sensitive Transactions

An application must process events for each customer order in the same sequence in which they were submitted.

**Recommended Solution:** Amazon SQS FIFO Queue.

**Reason:** FIFO queues preserve message ordering within message groups.

## Scenario 3: Duplicate Payment Processing

An application uses an SQS Standard queue for payment-processing requests.

Occasionally, the same message is delivered more than once.

**Recommended Solution:** Implement idempotent processing.

**Reason:** Standard queues provide at-least-once delivery, so duplicate messages must be handled safely.

## Scenario 4: Student Registration Workflow

An application must ensure that student account creation occurs before course enrollment.

**Recommended Solution:** Amazon SQS FIFO Queue.

**Reason:** The workflow requires ordered message processing.

## Scenario 5: Parallel Processing Across Multiple Workers

An application distributes independent background jobs across multiple EC2 workers.

Strict ordering is not required.

**Recommended Solution:** Amazon SQS Standard Queue.

**Reason:** Standard queues support high-throughput message distribution among multiple consumers.

---

# ❓ Interview Questions

### Q1. What are the two types of Amazon SQS queues?

Standard queues and FIFO queues.

### Q2. Which queue type is the default in Amazon SQS?

Standard queue.

### Q3. What is the main difference between Standard and FIFO queues?

Standard queues prioritize high throughput with best-effort ordering, while FIFO queues preserve ordering within message groups and support message deduplication.

### Q4. What delivery guarantee does a Standard queue provide?

At-least-once delivery.

### Q5. Can Standard queues deliver duplicate messages?

Yes. Duplicate message delivery is possible.

### Q6. Do Standard queues guarantee message ordering?

No. Messages may occasionally be delivered out of order.

### Q7. What is idempotency?

Idempotency means that processing the same operation multiple times does not cause unintended additional effects.

### Q8. Why is idempotency important for Standard queues?

Because Standard queues can deliver the same message more than once.

### Q9. What does FIFO stand for?

First-In-First-Out.

### Q10. What is the primary advantage of FIFO queues?

They preserve message ordering within message groups and support deduplication.

### Q11. What throughput figures are introduced in the lecture for FIFO queues?

300 transactions per second without batching and 3,000 messages per second with batching.

### Q12. Are those FIFO throughput limits universal?

No. Modern SQS FIFO throughput depends on the queue configuration and AWS Region.

### Q13. What is a suitable use case for Standard queues?

High-throughput background processing where message ordering is not critical.

### Q14. What is a suitable use case for FIFO queues?

Order-sensitive workflows such as e-commerce order management.

### Q15. Can FIFO queues completely eliminate the need for idempotent consumers?

No. Consumers should still handle retries and potential duplicate processing safely.

### Q16. Which queue would you choose for video transcoding?

A Standard queue is generally suitable when video-processing tasks are independent.

### Q17. Which queue would you choose for sequential account-registration events?

A FIFO queue is suitable when the events must be processed in order.

### Q18. What is the purpose of message deduplication in FIFO queues?

It helps prevent duplicate messages from being inserted into the queue within the deduplication interval.

---

# 💡 Key Takeaways

- Amazon SQS supports Standard and FIFO queues.
- Standard is the default queue type.
- Standard queues provide very high throughput.
- Standard queues provide at-least-once message delivery.
- Duplicate messages are possible with Standard queues.
- Standard queues do not guarantee strict message ordering.
- Applications consuming Standard queues should be idempotent.
- FIFO stands for First-In-First-Out.
- FIFO queues preserve message ordering within message groups.
- FIFO queues support message deduplication.
- The lecture introduces traditional FIFO throughput figures of 300 transactions per second without batching and 3,000 messages per second with batching.
- Modern FIFO throughput depends on configuration and AWS Region.
- Standard queues are suitable for independent background-processing workloads.
- FIFO queues are suitable for order-sensitive workflows.
- Queue selection should be based on application requirements rather than choosing FIFO by default.

---

## The Most Important Concept to Remember

```text
                         Amazon SQS
                             |
                  +----------+----------+
                  |                     |
                  v                     v
             Standard Queue         FIFO Queue
                  |                     |
                  v                     v
             At-Least-Once        Ordered Delivery
                Delivery          Within Message Groups
                  |                     |
                  v                     v
             Best-Effort            Deduplication
               Ordering               Support
                  |                     |
                  v                     v
             High-Throughput       Order-Sensitive
                Workloads             Workloads
```

**Standard Queue = High throughput + At-least-once delivery + Best-effort ordering**

**FIFO Queue = Ordered delivery + Message deduplication**

**Idempotency = Safe handling of repeated message processing**

---

# 📚 Related Topics

- Amazon Simple Queue Service (SQS)
- Producer-Consumer Architecture
- Decoupled Applications
- Message Ordering
- At-Least-Once Delivery
- Message Deduplication
- Idempotent Applications
- Amazon SQS Message Groups
- Asynchronous Processing
- Distributed Systems

**Next topic:** Amazon SQS Visibility Timeout, Message Retention, and Dead-Letter Queues