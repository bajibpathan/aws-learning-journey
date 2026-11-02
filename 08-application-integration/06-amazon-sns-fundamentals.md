# Amazon SNS – Fundamentals, Architecture, Topic Types, and Message Delivery

> Amazon Simple Notification Service (SNS) is a fully managed, push-based messaging service that enables applications to publish messages to topics and automatically deliver those messages to subscribed endpoints.

---

## 📖 Overview

In the previous lessons, we explored Amazon Simple Queue Service (SQS), including:

- Producer-consumer architecture
- Standard and FIFO queues
- Message polling
- Visibility timeout
- Dead-Letter Queues
- Delay queues
- Message retention
- Asynchronous processing

Amazon SQS uses a **pull-based messaging model**.

Producers send messages to a queue, and consumers retrieve those messages when they are ready to process them.

Amazon SNS provides a different messaging approach.

Instead of consumers polling a queue, SNS uses a **push-based publish-subscribe model**.

A publisher sends a message to an SNS topic, and SNS delivers that message to the topic's subscribers.

This lesson covers:

1. Amazon SNS fundamentals
2. SNS vs SQS
3. Publish-subscribe architecture
4. SNS topics and subscriptions
5. Supported delivery protocols
6. Application-to-Person (A2P) messaging
7. Application-to-Application (A2A) messaging
8. SNS topic policies
9. Message filtering
10. Standard and FIFO topics
11. Delivery retry policies
12. Dead-Letter Queues
13. Real-world architecture examples
14. AWS certification scenarios

---

# 1. What Is Amazon SNS?

Amazon Simple Notification Service (SNS) is a fully managed messaging service that enables communication between publishers and subscribers.

SNS uses a **publish-subscribe (Pub/Sub)** messaging model.

In this model:

- Publishers send messages to an SNS topic.
- Subscribers subscribe to that topic.
- SNS delivers published messages to eligible subscribers.

### Architecture

```text
                         Publisher
                             |
                             v
                       Publish Message
                             |
                             v
                      Amazon SNS Topic
                             |
                 +-----------+-----------+
                 |           |           |
                 v           v           v
             Subscriber  Subscriber  Subscriber
                 1           2           3
```

### Key Concept

**Amazon SNS = Push-based messaging service**

Publishers do not need to know the individual subscribers.

They only need to publish messages to the appropriate SNS topic.

SNS handles message delivery to the subscribers.

---

# 2. Amazon SNS vs Amazon SQS

The most important difference between Amazon SNS and Amazon SQS is how messages are delivered.

## Amazon SQS – Pull-Based Messaging

In Amazon SQS:

1. A producer sends a message to a queue.
2. The message remains in the queue.
3. A consumer polls the queue.
4. The consumer retrieves the message.
5. The consumer processes and deletes the message.

### Architecture

```text
                       Producer
                          |
                          v
                    Amazon SQS Queue
                          |
                          |
                    Message Stored
                          |
                          v
                     Consumer
                          |
                          v
                    Poll for Message
                          |
                          v
                    Process Message
                          |
                          v
                    Delete Message
```

The consumer controls when it retrieves messages.

## Amazon SNS – Push-Based Messaging

In Amazon SNS:

1. A publisher sends a message to a topic.
2. SNS identifies eligible subscribers.
3. SNS pushes the message to their endpoints.

### Architecture

```text
                       Publisher
                          |
                          v
                     SNS Topic
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
            Email       Lambda       HTTP API
```

Subscribers do not need to poll the SNS topic.

SNS initiates delivery to the subscribed endpoints.

### Comparison

| Feature | Amazon SQS | Amazon SNS |
|---|---|---|
| Messaging Model | Queue-based | Publish-subscribe |
| Delivery Model | Pull | Push |
| Main Components | Producer, queue, consumer | Publisher, topic, subscriber |
| Message Retrieval | Consumers poll | SNS delivers to subscribers |
| Primary Purpose | Decouple workloads and buffer messages | Distribute notifications and events |
| Multiple Recipients | Competing consumers typically process different messages | A message can be delivered to multiple subscribers |
| Message Storage | Messages remain until deleted or expired | Not a general-purpose message queue |
| Typical Use Cases | Background jobs, task processing | Notifications, event distribution, fan-out |

### AWS Exam Point

**SQS = Consumers pull messages from a queue.**

**SNS = Messages are pushed to subscribers.**

---

# 3. Understanding the Publish-Subscribe Model

Amazon SNS follows the publish-subscribe architecture.

The three primary components are:

1. Publisher
2. Topic
3. Subscriber

## Publisher

A publisher is an application or AWS service that sends messages to an SNS topic.

Examples include:

- Amazon S3
- Application servers
- AWS Lambda
- Amazon CloudWatch
- Custom applications

## Topic

An SNS topic acts as a communication channel between publishers and subscribers.

Publishers send messages to the topic.

Subscribers register to receive messages published to that topic.

## Subscriber

A subscriber is an endpoint that receives messages from an SNS topic.

Examples include:

- Email addresses
- SMS recipients
- AWS Lambda functions
- Amazon SQS queues
- HTTP/HTTPS endpoints

### Architecture

```text
                      Publisher
                          |
                          |
                    Publish Event
                          |
                          v
                  +----------------+
                  |                |
                  |   SNS Topic    |
                  |                |
                  +----------------+
                          |
                +---------+---------+
                |         |         |
                v         v         v
              Email     Lambda     SQS
```

### Key Concept

SNS decouples publishers from subscribers.

The publisher does not need to communicate directly with every destination.

---

# 4. Amazon SNS Topics

An SNS topic is a logical access point used to distribute messages.

Applications publish messages to a topic using the SNS Publish API.

Subscribers receive messages based on their subscriptions.

### Example

Suppose an application creates a topic named:

```text
order-notifications
```

Multiple systems subscribe to this topic.

```text
                    E-Commerce Application
                              |
                              v
                       SNS Topic
                    order-notifications
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              Email         Lambda         SQS
           Notification   Processing     Queue
```

When an order event is published, SNS can distribute the event to multiple subscribers.

### Benefits

- Loose coupling
- Event-driven communication
- One-to-many message distribution
- Integration with multiple AWS services
- Support for distributed applications

---

# 5. Amazon SNS Supported Delivery Protocols

Amazon SNS supports multiple subscription protocols.

The lecture introduces the following destinations:

- AWS Lambda
- Amazon SQS
- HTTP
- HTTPS
- Email
- SMS
- Mobile push notifications
- Amazon Data Firehose

### Architecture

```text
                          Publisher
                              |
                              v
                         SNS Topic
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
       AWS Lambda          Amazon SQS          HTTP/HTTPS
          |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
         Email                SMS             Mobile Push
```

### Protocol Comparison

| Protocol | Destination | Common Use Case |
|---|---|---|
| Lambda | AWS Lambda function | Serverless processing |
| SQS | Amazon SQS queue | Asynchronous processing |
| HTTP | HTTP endpoint | Web application integration |
| HTTPS | HTTPS endpoint | Secure webhook delivery |
| Email | Email address | Administrator notifications |
| SMS | Mobile phone | Text notifications |
| Mobile Push | Mobile application | Push notifications |
| Firehose | Amazon Data Firehose | Streaming data delivery |

### Important Concept

A single SNS topic can distribute messages to multiple subscribers using different supported protocols.

---

# 6. Amazon SNS Messaging Categories

The lecture introduces two major messaging categories:

1. Application-to-Person (A2P)
2. Application-to-Application (A2A)

### Architecture

```text
                         Amazon SNS
                             |
                  +----------+----------+
                  |                     |
                  v                     v
          Application-to-Person  Application-to-Application
                 (A2P)                  (A2A)
                  |                     |
                  v                     v
             Email / SMS          Lambda / SQS
             Mobile Push          HTTP / HTTPS
```

These categories describe who or what receives the notification.

---

# 7. Application-to-Person (A2P) Messaging

Application-to-Person messaging allows applications to send notifications directly to individuals.

Examples include:

- Email notifications
- SMS messages
- Mobile push notifications

### Example Use Case

An administrator wants to receive an email whenever a file is uploaded to an Amazon S3 bucket.

The application can use:

- Amazon S3
- S3 Event Notifications
- Amazon SNS
- Email subscription

### Architecture

```text
                         User
                          |
                          v
                    Upload Object
                          |
                          v
                     Amazon S3
                          |
                          v
                  S3 Event Notification
                          |
                          v
                      SNS Topic
                          |
                          v
                  Email Subscription
                          |
                          v
                     Administrator
```

### Workflow

1. A user uploads an object to an S3 bucket.
2. Amazon S3 generates an event notification.
3. The event is published to an SNS topic.
4. SNS delivers the notification to the subscribed email address.
5. The administrator receives the email.

### Example Event

```text
Event: ObjectCreated

Bucket: application-uploads

Object: report.pdf
```

### Result

The administrator receives a notification when the object-upload event occurs.

---

# 8. Configuring Amazon S3 Event Notifications with SNS

The lecture uses S3 Event Notifications as an example of SNS integration.

### Step 1: Create an SNS Topic

Create a topic such as:

```text
s3-upload-notifications
```

### Step 2: Configure an S3 Event Notification

In the S3 bucket configuration, create an event notification for object creation.

Example event:

```text
s3:ObjectCreated:*
```

### Step 3: Select the SNS Topic

Configure Amazon S3 to publish the event to the SNS topic.

### Step 4: Create an Email Subscription

Subscribe the administrator's email address to the topic.

### Step 5: Confirm the Subscription

The administrator confirms the subscription through the confirmation email.

### Step 6: Test the Integration

Upload a file to the S3 bucket.

### Expected Result

SNS delivers an email notification containing information about the S3 event.

### Complete Architecture

```text
                       Amazon S3 Bucket
                              |
                              v
                       Object Uploaded
                              |
                              v
                      Event Notification
                              |
                              v
                        SNS Topic
                              |
                              v
                     Email Subscription
                              |
                              v
                      Email Confirmed
                              |
                              v
                       Administrator
```

---

# 9. SNS Subscription Confirmation

The lecture highlights that Application-to-Person subscriptions need confirmation.

For example, when an email address subscribes to an SNS topic, SNS sends a confirmation request.

The recipient must confirm the subscription before notifications can be delivered.

### Workflow

```text
                    SNS Topic
                        |
                        v
                 Email Subscription
                        |
                        v
                Confirmation Email
                        |
                        v
                 Recipient Confirms
                        |
                        v
                 Subscription Active
                        |
                        v
                Receive Notifications
```

### Why Is Confirmation Important?

Subscription confirmation helps prevent unwanted notifications from being sent to recipients who have not agreed to receive them.

### Technical Clarification

Confirmation requirements depend on the subscription protocol and endpoint.

For example, email and HTTP/HTTPS subscriptions generally require confirmation, while some AWS service integrations can be authorized through IAM and resource policies.

Therefore, subscription confirmation is not exclusively an A2P concept.

---

# 10. Amazon SNS Topic Access Policies

Amazon SNS topics support resource-based access policies.

These policies define which AWS identities or services can perform actions against a topic.

For example:

- Which application can publish messages?
- Which AWS account can access the topic?
- Which AWS service can send events?
- Which identities can manage subscriptions?

### Architecture

```text
                       AWS Service
                           |
                           v
                      Publish Event
                           |
                           v
                    SNS Topic Policy
                           |
                    Authorization
                           |
                           v
                        SNS Topic
                           |
                           v
                       Subscribers
```

### Example: Amazon S3 to SNS

Suppose an S3 bucket publishes object-upload events to SNS.

The SNS topic policy must allow the S3 service to publish messages to the topic.

### Conceptual Permission

```text
Principal: Amazon S3

Action: sns:Publish

Resource: SNS Topic ARN

Condition: Authorized S3 Bucket
```

### Why Are Topic Policies Important?

They help prevent unauthorized publishers from sending messages to a topic.

### Security Best Practice

Use least-privilege permissions.

Only authorized identities and AWS resources should be allowed to publish messages.

---

# 11. Application-to-Application (A2A) Messaging

Application-to-Application messaging allows applications and AWS services to communicate through SNS topics.

Instead of delivering messages directly to people, SNS distributes events to other systems.

### Common Subscribers

- AWS Lambda
- Amazon SQS
- HTTP/HTTPS endpoints
- Amazon Data Firehose

### Architecture

```text
                         Publisher
                             |
                             v
                         SNS Topic
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
          Lambda            SQS          HTTP/HTTPS
             |               |               |
             v               v               v
          Process          Queue          External
           Event          Message          Service
```

### Example

An e-commerce application publishes an order event.

Multiple applications need to respond.

```text
                    E-Commerce Application
                              |
                              v
                         SNS Topic
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
             Inventory      Billing     Notification
              Service       Service       Service
```

Each subscriber can independently process the event.

### Benefits

- Decoupled application components
- One-to-many event distribution
- Integration with microservices
- Event-driven architecture
- Serverless application integration

---

# 12. Amazon SNS and Amazon Data Firehose

The lecture introduces Amazon Data Firehose as another SNS integration destination.

Amazon Data Firehose is used to deliver streaming data to supported destinations.

### Architecture

```text
                       Publisher
                           |
                           v
                       SNS Topic
                           |
                           v
                    Amazon Data Firehose
                           |
                 +---------+---------+
                 |                   |
                 v                   v
              Amazon S3          Analytics
                                Destination
```

The lecture mentions destinations such as:

- Amazon S3
- Amazon Redshift
- Elasticsearch-based search services
- Third-party analytics and observability platforms

### Example

An application publishes operational events to an SNS topic.

The events are delivered to Amazon Data Firehose, which forwards them to a supported destination for storage or analysis.

### Important Concept

SNS handles event distribution.

Data Firehose handles managed delivery of streaming data to configured destinations.

---

# 13. SNS Message Filtering

Amazon SNS supports message filtering.

This allows subscribers to receive only messages that match specified criteria.

Without filtering, a topic may distribute every published message to all subscribers.

With filtering, each subscriber can define which messages it wants to receive.

### Example

An application publishes different types of events:

```text
OrderCreated
PaymentCompleted
OrderCancelled
```

Different subscribers may only need specific event types.

### Architecture

```text
                        Publisher
                            |
                            v
                        SNS Topic
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          Filter A       Filter B       Filter C
              |             |             |
              v             v             v
         OrderCreated   PaymentCompleted OrderCancelled
              |             |             |
              v             v             v
          Order SQS      Billing SQS    Refund Lambda
```

### Example Message

```json
{
  "order_id": "ORD-1001",
  "event_type": "OrderCreated"
}
```

A subscriber configured to receive `OrderCreated` events can receive this message.

A subscriber configured only for `PaymentCompleted` events would not receive it.

### Benefits

- Reduces unnecessary message delivery
- Allows selective event routing
- Supports different subscriber requirements
- Simplifies event-driven architectures

### Technical Clarification

SNS subscription filter policies can evaluate message attributes or message-body properties, depending on the configured filter policy scope.

---

# 14. Amazon SNS Topic Types

Amazon SNS supports two topic types:

1. Standard Topics
2. FIFO Topics

### Architecture

```text
                       Amazon SNS
                           |
                +----------+----------+
                |                     |
                v                     v
          Standard Topic          FIFO Topic
                |                     |
                v                     v
          High Throughput       Message Ordering
                |                     |
                v                     v
          Best-Effort           Message Grouping
            Ordering            and Deduplication
```

Each topic type supports different application requirements.

---

# 15. Amazon SNS Standard Topics

Standard topics are suitable when strict message ordering is not required.

They support high-throughput event distribution.

### Characteristics

- High throughput
- Best-effort message ordering
- At-least-once delivery
- Possible duplicate deliveries
- Broad protocol support

### Architecture

```text
                       Publisher
                           |
                           v
                    SNS Standard Topic
                           |
              +------------+------------+
              |            |            |
              v            v            v
             Email        Lambda        SQS
```

### Common Use Cases

- Administrator notifications
- Monitoring alerts
- Application events
- Email notifications
- Distributed event processing

### Important Concept

Standard topics are appropriate when message ordering is not critical.

---

# 16. Amazon SNS FIFO Topics

FIFO stands for **First-In-First-Out**.

FIFO topics support message ordering and deduplication.

The lecture introduces three important concepts:

1. Message ordering
2. Message grouping
3. Message deduplication

### Architecture

```text
                       Publisher
                           |
                           v
                      SNS FIFO Topic
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
                       SQS FIFO Queue
                           |
                           v
                         Consumer
```

### Message Ordering

FIFO topics preserve the order of messages within the same message group.

### Message Grouping

Messages are assigned a message group identifier.

For example:

```text
MessageGroupId = Customer-1001
```

Messages belonging to the same group are ordered relative to one another.

### Message Deduplication

FIFO topics support message deduplication using a deduplication identifier or supported content-based deduplication.

This helps prevent duplicate messages from being introduced during the deduplication interval.

---

# 17. SNS FIFO Topics and SQS Queues

The lecture explains that SNS FIFO topics can distribute messages to SQS FIFO queues.

This is particularly useful when applications require ordered event processing.

### Architecture

```text
                       Publisher
                           |
                           v
                       SNS FIFO Topic
                           |
              +------------+------------+
              |                         |
              v                         v
         SQS FIFO Queue            SQS FIFO Queue
              |                         |
              v                         v
          Consumer A                Consumer B
```

### Important Technical Clarification

SNS FIFO topics can also have SQS Standard queue subscribers.

However:

- SNS FIFO → SQS FIFO supports ordered message delivery.
- SNS FIFO → SQS Standard does not preserve FIFO ordering guarantees at the Standard queue consumer.

Therefore, applications requiring end-to-end FIFO behavior should use an appropriate FIFO topic and FIFO queue configuration.

---

# 18. Real-World Example: E-Commerce Order Processing

The lecture introduces an e-commerce application where order processing must preserve the order of events.

### Business Requirement

A company sells limited-stock products.

Customers place orders during a flash sale.

The system must process related order events in the correct sequence.

### Architecture

```text
                       Customers
                           |
                           v
                    E-Commerce App
                           |
                           v
                     SNS FIFO Topic
                           |
                           v
                     SQS FIFO Queue
                           |
                           v
                     Order Processor
                           |
                           v
                    Inventory Database
```

### Example Events

```text
OrderCreated
     |
     v
PaymentConfirmed
     |
     v
InventoryReserved
     |
     v
OrderCompleted
```

These events may need to be processed in sequence.

### Message Group ID

The application assigns related messages to the same message group.

Example:

```text
MessageGroupId = ORDER-1001
```

### Deduplication ID

The application may also assign a unique deduplication identifier.

Example:

```text
MessageDeduplicationId = ORDER-1001-CREATED
```

If the publisher retries the same event within the deduplication interval using the same identifier, SNS FIFO deduplication helps prevent duplicate publication effects.

### Important Design Consideration

For a flash sale requiring fairness across all orders, assigning a separate message group to each customer does **not** establish one global order across customers.

FIFO ordering applies within message groups.

The application must choose message grouping and inventory-control mechanisms that match its business requirements.

For example, ordering events by individual order ID is useful for preserving the sequence of events associated with that order.

---

# 19. SNS Standard vs FIFO Topics

| Feature | Standard Topic | FIFO Topic |
|---|---|---|
| Message Ordering | Best effort | Preserved within message groups |
| Delivery | At least once | Ordered delivery with deduplication support |
| Duplicate Messages | Possible | Deduplication supported |
| Message Group ID | Not required | Required |
| Message Deduplication ID | Not required | Supported |
| Throughput | Very high | Depends on FIFO configuration |
| Protocol Support | Broad | More restricted |
| Typical Use Case | Notifications and event distribution | Ordered business events |
| SQS Integration | Supported | Supports FIFO and Standard SQS subscribers |

### AWS Exam Point

**SNS Standard = High-throughput event distribution where ordering is not critical.**

**SNS FIFO = Ordered event distribution within message groups, with deduplication support.**

---

# 20. Amazon SNS Delivery Retry Policy

Amazon SNS supports delivery retry behavior for supported endpoint protocols.

If SNS cannot successfully deliver a message, it may retry delivery according to the applicable delivery policy.

### Example

```text
                       Publisher
                           |
                           v
                       SNS Topic
                           |
                           v
                     HTTP Endpoint
                           |
                           v
                    Delivery Failed
                           |
                           v
                      Retry Delivery
                           |
                           v
                    Delivery Successful
```

### Why Might Delivery Fail?

Examples include:

- Subscriber endpoint unavailable
- Server-side errors
- Network connectivity issues
- Temporary application failures

### Delivery Policy

A delivery policy determines how SNS handles retry attempts for supported delivery protocols.

The lecture emphasizes that SNS may stop retrying after the applicable retry policy is exhausted.

### Important Clarification

SNS retry behavior differs by subscription protocol.

Not every protocol exposes the same configurable retry settings.

---

# 21. What Happens When Delivery Retries Are Exhausted?

Suppose SNS repeatedly attempts to deliver a message to an HTTP endpoint.

The endpoint remains unavailable.

Eventually, the retry policy is exhausted.

### Without a Dead-Letter Queue

```text
                       SNS Topic
                           |
                           v
                      HTTP Endpoint
                           |
                           v
                     Delivery Failed
                           |
                           v
                      Retry Delivery
                           |
                           v
                    Retries Exhausted
                           |
                           v
                     Message Discarded
```

If no DLQ is configured, an undeliverable message may be discarded after the applicable retry policy is exhausted.

This can make troubleshooting difficult.

---

# 22. Amazon SNS Dead-Letter Queues

Amazon SNS supports Dead-Letter Queues for eligible subscriptions.

A DLQ stores messages that SNS could not successfully deliver to a subscriber.

### Architecture

```text
                        Publisher
                            |
                            v
                         SNS Topic
                            |
                            v
                        Subscriber
                            |
                            v
                      Delivery Failed
                            |
                            v
                       Retry Attempts
                            |
                            v
                      Retries Exhausted
                            |
                            v
                     Dead-Letter Queue
                            |
                            v
                       Investigation
```

### Benefits

- Preserves undeliverable messages
- Supports troubleshooting
- Helps identify subscriber failures
- Enables later message reprocessing
- Improves operational visibility

### Important Distinction

An SNS DLQ is configured for a subscription.

It captures messages that SNS could not deliver to that subscription.

This differs from an SQS source queue DLQ, which captures messages that consumers repeatedly fail to process.

---

# 23. SNS DLQ vs SQS DLQ

| Feature | SNS Dead-Letter Queue | SQS Dead-Letter Queue |
|---|---|---|
| Main Purpose | Capture failed SNS deliveries | Capture repeatedly failing SQS messages |
| Failure Occurs | During delivery to subscriber | During consumer processing |
| Configuration | SNS subscription redrive policy | SQS source queue redrive policy |
| Trigger | Delivery retries exhausted | Receive threshold exceeded |
| Destination | Amazon SQS queue | Amazon SQS queue |
| Primary Benefit | Delivery failure investigation | Processing failure investigation |

### Architecture

```text
                  SNS Delivery Failure
                           |
                           v
                      SNS Topic
                           |
                           v
                       Subscriber
                           |
                           v
                     Delivery Failed
                           |
                           v
                       SNS DLQ


                  SQS Processing Failure
                           |
                           v
                       SQS Queue
                           |
                           v
                        Consumer
                           |
                           v
                     Processing Failed
                           |
                           v
                       SQS DLQ
```

### AWS Exam Point

**SNS DLQ = Message delivery failure**

**SQS DLQ = Message processing failure**

---

# 24. Combining Amazon SNS and Amazon SQS

Amazon SNS and Amazon SQS are frequently used together.

This architecture is commonly called the **fan-out pattern**.

A publisher sends one message to an SNS topic.

SNS distributes the message to multiple SQS queues.

Each queue has its own consumers.

### Architecture

```text
                       Publisher
                           |
                           v
                        SNS Topic
                           |
              +------------+------------+
              |            |            |
              v            v            v
          SQS Queue A   SQS Queue B   SQS Queue C
              |            |            |
              v            v            v
          Consumer A   Consumer B   Consumer C
```

### Example

An e-commerce application publishes an order event.

Multiple services need to process the event independently.

```text
                    E-Commerce Application
                              |
                              v
                         SNS Topic
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
             Inventory      Billing     Notifications
               Queue         Queue         Queue
                 |            |            |
                 v            v            v
             Inventory      Billing     Notification
              Service       Service        Service
```

### Benefits

- One-to-many message distribution
- Independent application processing
- Loose coupling
- Message buffering through SQS
- Independent consumer scaling
- Better resilience when individual consumers are unavailable

### Key Concept

SNS distributes the event.

SQS buffers the event until the consumer is ready to process it.

---

# 25. Real-World Architecture: S3 Upload Notifications

Let's revisit the S3 example from the lecture.

A user uploads an object to Amazon S3.

Multiple actions may need to occur.

For example:

- Notify an administrator
- Process the uploaded file
- Record an audit event

### Architecture

```text
                          User
                           |
                           v
                     Upload Object
                           |
                           v
                       Amazon S3
                           |
                           v
                   S3 Event Notification
                           |
                           v
                        SNS Topic
                           |
              +------------+------------+
              |            |            |
              v            v            v
            Email       SQS Queue      Lambda
              |            |            |
              v            v            v
          Administrator  File Worker   Audit Processing
```

### Workflow

1. A user uploads an object.
2. Amazon S3 generates an event notification.
3. The event is published to SNS.
4. SNS distributes the event to eligible subscribers.
5. The administrator receives an email.
6. The file-processing application retrieves the event from SQS.
7. The Lambda function performs its configured operation.

This is an example of event-driven architecture using Amazon SNS.

---

# 26. Common Mistakes

### 1. Confusing SNS with SQS

SNS uses push-based delivery.

SQS uses pull-based consumption.

### 2. Assuming SNS Stores Messages Like SQS

SNS is not a general-purpose message queue.

For message buffering and consumer-controlled processing, SQS is typically more appropriate.

### 3. Forgetting Subscription Confirmation

Protocols such as email and HTTP/HTTPS generally require subscription confirmation.

### 4. Ignoring SNS Topic Policies

Publishers must have appropriate authorization to publish messages.

### 5. Assuming SNS Standard Topics Guarantee Ordering

Standard topics provide best-effort ordering.

### 6. Assuming FIFO Ordering Applies Across All Message Groups

FIFO ordering is maintained within each message group.

### 7. Assuming SNS FIFO Always Provides End-to-End FIFO Processing

End-to-end ordering depends on the downstream subscriber and application design.

### 8. Ignoring Delivery Failures

SNS delivery retries may eventually be exhausted.

DLQs can preserve undeliverable messages.

### 9. Confusing SNS DLQs with SQS DLQs

SNS DLQs handle delivery failures.

SQS DLQs handle repeated consumer-processing failures.

---

# 27. AWS Certification Exam Scenarios

## Scenario 1: Notify Multiple Applications

An application must send the same event to several independent backend services.

**Question:** Which AWS service is suitable?

**Answer:** Amazon SNS.

**Reason:** SNS supports one-to-many event distribution through topics and subscriptions.

---

## Scenario 2: Email Notification for S3 Upload

A company wants an administrator to receive an email whenever a file is uploaded to an S3 bucket.

**Question:** Which services can be used?

**Answer:** Amazon S3 Event Notifications and Amazon SNS.

**Reason:** S3 can publish object-created events to an SNS topic with an email subscription.

---

## Scenario 3: Asynchronous Background Processing

An application needs to buffer messages until backend workers have capacity to process them.

**Question:** Which service is most appropriate?

**Answer:** Amazon SQS.

**Reason:** SQS stores messages until consumers retrieve them.

---

## Scenario 4: Fan-Out Architecture

An application must distribute an event to three independent processing systems.

Each system should process the event at its own pace.

**Question:** Which architecture is suitable?

**Answer:** Amazon SNS with multiple Amazon SQS subscriptions.

**Reason:** SNS distributes the event, while SQS provides independent message buffering.

---

## Scenario 5: Ordered Event Processing

An application requires related events to be delivered in the order they were published.

**Question:** Which SNS topic type should be used?

**Answer:** SNS FIFO Topic.

**Reason:** FIFO topics preserve ordering within message groups.

---

## Scenario 6: Selective Message Delivery

An SNS topic receives order, payment, and cancellation events.

Each subscriber should receive only the events relevant to its application.

**Question:** Which SNS feature should be used?

**Answer:** Subscription filter policies.

**Reason:** Filtering allows subscribers to receive only matching messages.

---

## Scenario 7: Subscriber Delivery Failure

An SNS topic repeatedly fails to deliver messages to an HTTP endpoint.

The operations team wants to preserve undeliverable messages.

**Question:** Which feature should be configured?

**Answer:** An SNS subscription Dead-Letter Queue.

**Reason:** It stores messages that SNS could not deliver after applicable retries are exhausted.

---

## Scenario 8: Prevent Unauthorized Publishing

An S3 bucket publishes events to an SNS topic.

The company wants to ensure only the authorized bucket can publish events.

**Question:** What should be configured?

**Answer:** An appropriate SNS topic resource policy.

**Reason:** Topic policies control which identities and AWS services can publish messages.

---

# ❓ Interview Questions

### Q1. What is Amazon SNS?

Amazon SNS is a fully managed, push-based publish-subscribe messaging service.

### Q2. What does SNS stand for?

Simple Notification Service.

### Q3. What is the difference between SNS and SQS?

SNS pushes messages to subscribers, while SQS stores messages for consumers to retrieve.

### Q4. What is an SNS topic?

A logical communication channel where publishers send messages for delivery to subscribers.

### Q5. What is a publisher?

An application or AWS service that sends messages to an SNS topic.

### Q6. What is a subscriber?

An endpoint registered to receive messages from an SNS topic.

### Q7. What is Application-to-Person messaging?

Messaging where applications send notifications directly to individuals through channels such as email, SMS, or mobile push.

### Q8. What is Application-to-Application messaging?

Messaging where applications distribute events to other applications or services.

### Q9. Which protocols does SNS support?

SNS supports protocols including Lambda, SQS, HTTP/HTTPS, email, SMS, mobile push, and Data Firehose.

### Q10. Can Amazon S3 publish events to SNS?

Yes. Amazon S3 Event Notifications can publish supported events to SNS topics.

### Q11. Why are SNS topic policies important?

They control which identities and AWS services are authorized to access or publish messages to a topic.

### Q12. What are the two SNS topic types?

Standard and FIFO.

### Q13. What is the primary difference between Standard and FIFO topics?

Standard topics provide high-throughput event distribution with best-effort ordering, while FIFO topics support ordered delivery within message groups and deduplication.

### Q14. What is an SNS message group ID?

An identifier used to group related messages for FIFO ordering.

### Q15. What is message deduplication?

A mechanism that helps prevent duplicate messages from being introduced during the deduplication interval.

### Q16. What is SNS message filtering?

A feature that allows subscribers to receive only messages matching configured filter criteria.

### Q17. What is an SNS delivery policy?

A policy that controls retry behavior for supported subscriber endpoint protocols.

### Q18. What happens when SNS delivery retries are exhausted?

If delivery remains unsuccessful, SNS stops retrying. A configured DLQ can preserve the undeliverable message.

### Q19. What is an SNS Dead-Letter Queue?

An SQS queue used to store messages that SNS could not successfully deliver to a subscription.

### Q20. What is the SNS fan-out pattern?

An architecture where one SNS topic distributes messages to multiple subscribers, commonly multiple SQS queues.

### Q21. Can SNS FIFO topics deliver messages to SQS Standard queues?

Yes, but SQS Standard queues do not preserve FIFO ordering guarantees.

### Q22. What architecture is suitable for multiple applications that need to process the same event independently?

Amazon SNS with separate SQS queues for each application.

---

# 💡 Key Takeaways

- Amazon SNS is a fully managed, push-based messaging service.
- SNS uses the publish-subscribe model.
- Publishers send messages to SNS topics.
- Subscribers receive messages published to those topics.
- SNS supports one-to-many message distribution.
- SNS differs from SQS because SNS pushes messages while SQS consumers pull messages.
- SNS supports Application-to-Person and Application-to-Application messaging.
- A2P messaging includes email, SMS, and mobile push notifications.
- A2A messaging includes Lambda, SQS, HTTP/HTTPS, and Data Firehose integrations.
- Amazon S3 can publish event notifications to SNS topics.
- Email subscriptions generally require confirmation.
- SNS topic policies control authorized publishers and other access.
- Subscription filter policies support selective message delivery.
- SNS supports Standard and FIFO topics.
- Standard topics provide high-throughput messaging with best-effort ordering.
- FIFO topics support message ordering within message groups.
- FIFO topics support message deduplication.
- SNS supports delivery retries for supported subscription protocols.
- SNS subscription DLQs can preserve undeliverable messages.
- SNS and SQS can be combined using the fan-out pattern.
- SNS distributes events, while SQS provides message buffering.

---

## The Most Important Concept to Remember

```text
                         Amazon SNS
                             |
                             v
                       Publisher
                             |
                             v
                         SNS Topic
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
            Email          Lambda          SQS
              |              |              |
              v              v              v
         Notification    Event Processing  Message Queue
                                             |
                                             v
                                           Consumer
                                             |
                                             v
                                       Process Message
```

**SNS = Push-based publish-subscribe messaging**

**SQS = Pull-based message queuing**

**SNS Topic = Communication channel between publishers and subscribers**

**SNS Standard = High-throughput messaging with best-effort ordering**

**SNS FIFO = Ordered messaging within message groups with deduplication support**

**SNS Message Filtering = Selective message delivery to subscribers**

**SNS DLQ = Preserves messages that SNS could not deliver**

**SNS + SQS = Fan-out architecture with independent message buffering**

---

# 📚 Related Topics

- Amazon Simple Notification Service (SNS)
- Amazon Simple Queue Service (SQS)
- Publish-Subscribe Architecture
- Application-to-Person Messaging
- Application-to-Application Messaging
- SNS Standard Topics
- SNS FIFO Topics
- SNS Subscription Filter Policies
- SNS Topic Access Policies
- SNS Delivery Retry Policies
- SNS Dead-Letter Queues
- Amazon S3 Event Notifications
- AWS Lambda
- Amazon Data Firehose
- Event-Driven Architecture
- SNS and SQS Fan-Out Pattern
