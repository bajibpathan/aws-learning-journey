# Amazon Simple Queue Service (Amazon SQS)

> Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables applications to communicate asynchronously, helping organizations build decoupled, scalable, and resilient distributed architectures.

---

## 📖 Overview

In traditional application architectures, frontend and backend components often communicate directly.

Although this approach works well for smaller workloads, it can introduce scalability and reliability challenges when application traffic increases.

Amazon SQS addresses these challenges by introducing a message queue between application components.

This allows applications to:

- Communicate asynchronously
- Process requests independently
- Handle sudden increases in traffic
- Reduce direct dependencies between components
- Improve application resilience
- Scale frontend and backend services independently

In this lesson, we will explore how Amazon SQS works and how it can be used to build a decoupled application architecture.

---

# 1. Understanding Application Integration

AWS provides application integration services that help organizations build distributed applications.

These services support architectural patterns such as:

- Decoupled applications
- Event-driven architectures
- Microservices
- Asynchronous processing
- Distributed systems

The objective is to reduce dependencies between application components and improve their ability to handle failures and changes in workload.

### Traditional Application Architecture

Consider a three-tier web application:

```text
                     Internet
                        |
                        v
               Application Load Balancer
                        |
                        v
                 Web / Frontend Layer
                        |
                        v
                  Application Layer
                        |
                        v
                    Database
```

The architecture contains three main layers.

| Layer | Responsibility |
|---|---|
| Web Layer | Handles user interfaces and incoming requests |
| Application Layer | Processes business logic and transactions |
| Database Layer | Stores application data |

This is a common architecture for web applications.

However, problems can occur when the number of incoming requests increases significantly.

---

# 2. The Problem with Tightly Coupled Applications

In a traditional application, the frontend sends requests directly to the backend.

For example:

1. A customer visits an e-commerce website.
2. The customer places an order.
3. The frontend sends the order request to the backend.
4. The backend processes the order.
5. The backend updates the database.

### Architecture

```text
                  Users
                    |
                    v
               Frontend
                    |
                    v
                Backend
                    |
                    v
                Database
```

This approach works when traffic is manageable.

### What Happens When Traffic Increases?

Imagine thousands or millions of users accessing the application.

The frontend may accept requests relatively quickly because its primary responsibility is handling the presentation layer and receiving user input.

However, the backend may need additional time to:

- Validate transactions
- Process orders
- Perform business logic
- Update databases
- Process uploaded files
- Perform computationally intensive operations

As traffic increases, the backend may become overloaded.

### Example

```text
                 10,000 Users
                      |
                      v
                  Frontend
                      |
                      v
              Backend Servers
                      |
                      v
                 OVERLOADED
                      |
                      v
             Application Failures
```

### Challenges

**1. Backend Overload**

The frontend may accept requests faster than the backend can process them.

**2. Increased Response Times**

Backend processing delays can affect application performance.

**3. Application Failures**

Excessive workloads may cause backend services to become unavailable.

**4. Scaling Dependencies**

The frontend and backend may have different scaling requirements.

**5. Reduced Resilience**

Failures in backend services can directly affect frontend request processing.

This is where Amazon SQS becomes useful.

---

# 3. What Is Amazon SQS?

Amazon Simple Queue Service (SQS) is a fully managed message queuing service.

It allows application components to communicate by exchanging messages through a queue.

Instead of sending requests directly from the frontend to the backend, the frontend sends messages to SQS.

The backend retrieves those messages when it is ready to process them.

### Architecture

```text
                    Users
                      |
                      v
                  Frontend
                      |
                      v
             ┌─────────────────┐
             │   Amazon SQS    │
             │                 │
             │  Message Queue  │
             └─────────────────┘
                      |
                      v
                   Backend
                      |
                      v
                   Database
```

Amazon SQS acts as a buffer between application components.

This architectural approach is known as **decoupling**.

---

# 4. What Is Decoupled Architecture?

Decoupling means reducing direct dependencies between application components.

In the context of Amazon SQS, the frontend and backend do not need to communicate directly to submit and process asynchronous work.

Instead, they communicate through a message queue.

### Without Amazon SQS

```text
Frontend ──────────────> Backend
                            |
                            v
                         Database
```

The frontend depends on the backend being available to receive requests.

### With Amazon SQS

```text
Frontend
    |
    v
Amazon SQS
    |
    v
Backend
    |
    v
Database
```

The frontend can submit a message to the queue without waiting for the backend to complete the associated processing.

### Benefits

- Reduced dependencies between components
- Improved resilience to temporary backend failures
- Better handling of traffic spikes
- Independent scaling
- Asynchronous processing

**Key Concept:** Amazon SQS separates the rate at which requests are accepted from the rate at which backend systems process them.

---

# 5. How Amazon SQS Works

Amazon SQS uses a producer-consumer messaging model.

The two main components are:

1. Producer
2. Consumer

## Producer

A producer is an application component that sends messages to an SQS queue.

In our example, the frontend acts as the producer.

## Consumer

A consumer is an application component that retrieves and processes messages from the queue.

In our example, the backend acts as the consumer.

### Producer-Consumer Architecture

```text
             Producer
             Frontend
                |
                | Send Message
                v
        ┌──────────────────┐
        │    Amazon SQS    │
        │                  │
        │   Message Queue  │
        └──────────────────┘
                |
                | Poll Messages
                v
             Consumer
              Backend
```

### Message Processing Workflow

1. A user submits a request.
2. The frontend validates the request.
3. The frontend sends a message to SQS.
4. SQS stores the message.
5. The backend polls the queue.
6. The backend retrieves the message.
7. The backend processes the request.
8. The backend updates the database or performs the required operation.

For reliable processing, the consumer should delete the message from the queue after successfully processing it.

---

# 6. Understanding Message Polling

In the architecture discussed in this lesson, backend applications retrieve messages by polling the queue.

Polling means checking SQS for available messages.

### Example

```text
                  Amazon SQS
                       |
             ┌─────────┴─────────┐
             |                   |
             v                   v
          Message 1           Message 2
             |                   |
             └─────────┬─────────┘
                       |
                       v
                Backend Polls
                       |
                       v
                Retrieves Message
                       |
                       v
                 Processes Work
```

The backend can retrieve messages according to its available processing capacity.

### Why Is This Important?

Suppose the frontend receives 1,000 requests in a short period.

The backend may only be able to process 100 requests at a time.

Instead of overwhelming the backend, SQS temporarily holds the remaining messages.

```text
Incoming Requests: 1,000
          |
          v
      Amazon SQS
          |
          v
Backend Processing Capacity
     100 Requests
          |
          v
Remaining Messages Wait
```

This buffering capability is one of the main reasons for introducing SQS into an application architecture.

---

# 7. Amazon SQS Message Size

The course introduces an SQS message size of **256 KB**.

That was a commonly documented limit, but AWS subsequently increased the maximum SQS message size to **1 MiB (1,048,576 bytes)**.

For certification preparation, distinguish the course's stated value from the current service limit.

Even with the larger limit, SQS is not designed to transport large files such as videos.

### Handling Large Files

Instead of placing the file itself in an SQS message:

1. Store the file in Amazon S3.
2. Send an SQS message containing the S3 object location.
3. Let the backend retrieve the object from S3.

### Architecture

```text
                   User
                    |
                    v
                 Frontend
                    |
          ┌─────────┴─────────┐
          |                   |
          v                   v
      Amazon S3          Amazon SQS
      Video File         S3 Reference
          |                   |
          |                   v
          |                Backend
          |                   |
          └─────────┬─────────┘
                    |
                    v
              Process Video
```

**Key Concept:** Store large objects in Amazon S3 and use SQS messages to communicate the object location and processing instructions.

---

# 8. Amazon SQS Message Retention

Amazon SQS stores messages for a configurable retention period.

The lesson highlights the following values:

| Setting | Duration |
|---|---|
| Default Message Retention | 4 days |
| Maximum Message Retention | 14 days |

### Example

Suppose the backend becomes temporarily unavailable.

Messages can remain in the queue while the backend recovers, subject to the configured retention period.

```text
Frontend
    |
    v
Amazon SQS
    |
    | Messages Waiting
    |
    v
Backend Temporarily Unavailable
    |
    v
Backend Recovers
    |
    v
Messages Processed
```

### Why Message Retention Matters

Message retention provides time for applications to recover from temporary processing delays.

However, messages must be processed before their retention period expires.

**Important:** Message retention does not guarantee that every message will be processed successfully. Applications must still implement appropriate processing, retry, and failure-handling logic.

---

# 9. Real-World Use Case: Video Processing Application

The lesson demonstrates how Amazon SQS can be used in a video-processing application.

### Business Requirement

Users upload video files through a web application.

The backend must convert those files into formats suitable for different devices.

For example:

- Desktop computers
- Mobile phones
- Tablets
- Different video resolutions

Video processing can require significant computing resources.

Therefore, the frontend and backend should scale independently.

---

# 10. Video Processing Architecture

The architecture contains:

- Users
- Frontend application servers
- Frontend Auto Scaling Group
- Amazon S3 raw video bucket
- Amazon SQS queue
- Backend processing servers
- Backend Auto Scaling Group
- Amazon S3 processed video bucket

### Architecture Diagram

```text
                         Users
                           |
                           v
                    Application Frontend
                           |
                    Auto Scaling Group
                           |
                ┌──────────┴──────────┐
                |                     |
                v                     v
         Amazon S3 Bucket        Amazon SQS
           Raw Videos           Message Queue
                |                     |
                |                     v
                |              Backend Workers
                |              Auto Scaling Group
                |                     |
                └──────────┬──────────┘
                           |
                           v
                    Video Processing
                           |
                           v
                    Amazon S3 Bucket
                    Processed Videos
                           |
                           v
                         Users
```

The architecture separates the upload process from the video-processing workload.

---

# 11. Video Processing Workflow

## Step 1: User Uploads a Video

A user accesses the application and uploads a video file.

The request is handled by the frontend application.

```text
User
 |
 v
Frontend Application
```

## Step 2: Store the Video in Amazon S3

The frontend uploads the raw video file to an S3 bucket.

```text
Frontend
    |
    v
Amazon S3
Raw Video Bucket
```

## Step 3: Send a Message to Amazon SQS

The frontend sends a message to an SQS queue.

The message identifies the uploaded video that requires processing.

```text
Frontend
    |
    v
Amazon SQS
    |
    v
Message: Video Ready for Processing
```

The message can contain information such as the S3 object location.

## Step 4: Backend Polls the Queue

Backend workers poll Amazon SQS for available messages.

```text
Amazon SQS
    |
    v
Backend Worker
    |
    v
Retrieve Message
```

## Step 5: Retrieve the Video from S3

The backend uses the information in the message to locate and retrieve the uploaded video.

```text
Backend Worker
    |
    v
Amazon S3
Raw Video Bucket
```

## Step 6: Process the Video

The backend performs video transcoding.

For example, it may generate multiple formats or resolutions.

```text
Original Video
      |
      v
Video Processing
      |
      ├── Format A
      ├── Format B
      └── Format C
```

## Step 7: Store the Processed Video

The processed files are uploaded to another S3 bucket.

```text
Backend Worker
      |
      v
Amazon S3
Processed Video Bucket
```

## Step 8: Make the Video Available to Users

The processed videos can now be delivered to users in the required formats.

### Complete Workflow

```text
User Uploads Video
        |
        v
Frontend Application
        |
        v
Store Raw Video in S3
        |
        v
Send Message to SQS
        |
        v
Backend Polls Queue
        |
        v
Retrieve Video from S3
        |
        v
Transcode Video
        |
        v
Store Processed Video in S3
        |
        v
Video Available to Users
```

---

# 12. Independent Scaling of Frontend and Backend

One of the most important benefits of this architecture is independent scaling.

The frontend and backend have different responsibilities.

Therefore, they should not necessarily scale using the same metrics.

### Frontend Scaling

The frontend is responsible for receiving user requests.

It may scale based on metrics such as:

- CPU utilization
- Incoming traffic
- Application workload

### Backend Scaling

The backend is responsible for processing messages.

It may scale based on:

- Number of messages waiting in SQS
- Age of the oldest message
- Processing demand

### Architecture

```text
                     Users
                       |
                       v
               Frontend Auto Scaling
                       |
                       v
                   Amazon SQS
                       |
                       v
               Backend Auto Scaling
                       |
                       v
                 Process Messages
```

The frontend and backend can increase or decrease their capacity independently.

---

# 13. Amazon SQS Metrics for Backend Scaling

The lesson introduces two important Amazon SQS metrics.

## ApproximateNumberOfMessagesVisible

This metric indicates the approximate number of messages available for retrieval from the queue.

### Example

Suppose the queue contains a large number of waiting messages.

```text
Amazon SQS
    |
    v
Many Messages Waiting
    |
    v
Scale Out Backend Workers
    |
    v
Process Messages Faster
```

The backend can increase processing capacity to reduce the queue backlog.

## ApproximateAgeOfOldestMessage

This metric indicates the approximate age of the oldest unprocessed message in the queue.

### Example

If messages remain in the queue for too long, the application may need additional backend processing capacity.

```text
Amazon SQS
    |
    v
Oldest Message Age Increasing
    |
    v
Backend Processing Delay
    |
    v
Increase Backend Capacity
```

### Metrics Comparison

| Metric | What It Measures | Why It Matters |
|---|---|---|
| `ApproximateNumberOfMessagesVisible` | Approximate number of available messages | Helps identify queue backlog |
| `ApproximateAgeOfOldestMessage` | Approximate age of the oldest unprocessed message | Helps identify processing delays |

These metrics can help determine when backend workers need to scale.

---

# 14. Scaling Example

Consider a video-processing application with the following workload.

### Normal Traffic

```text
Users Upload Videos
        |
        v
Amazon SQS
        |
        v
2 Backend Workers
```

The two backend workers are sufficient to process incoming messages.

### Traffic Spike

Suppose a large number of users upload videos simultaneously.

```text
Large Number of Uploads
          |
          v
      Amazon SQS
          |
          v
   Queue Backlog Increases
          |
          v
   Scale Out Backend Workers
          |
          v
      6 Backend Workers
```

The backend increases processing capacity to handle the additional workload.

### Traffic Returns to Normal

As the backlog decreases, backend capacity can be reduced.

```text
Queue Backlog Decreases
          |
          v
   Scale In Backend Workers
          |
          v
      2 Backend Workers
```

This allows the application to adjust processing capacity according to demand.

---

# 15. Benefits of Amazon SQS

| Benefit | Explanation |
|---|---|
| Decoupling | Separates frontend and backend components |
| Asynchronous Processing | Allows requests to be processed independently |
| Buffering | Temporarily stores messages during traffic spikes |
| Independent Scaling | Enables different application components to scale separately |
| Resilience | Helps applications tolerate temporary backend unavailability |
| Managed Service | AWS manages the underlying queue infrastructure |
| Distributed Architecture | Supports communication between distributed application components |

### Key Architectural Advantage

Without SQS, the frontend and backend may be directly dependent on each other's availability and processing capacity.

With SQS, the queue provides an intermediate layer that buffers work until backend consumers can process it.

---

# 16. Important SQS Concepts

The lesson introduces several foundational concepts.

| Concept | Description |
|---|---|
| Queue | Stores messages until they are processed or expire |
| Producer | Application component that sends messages |
| Consumer | Application component that retrieves and processes messages |
| Polling | Retrieving messages from the queue |
| Message Retention | Duration messages can remain in the queue |
| Decoupling | Reducing direct dependencies between application components |
| Queue Backlog | Messages waiting to be processed |
| Independent Scaling | Scaling producers and consumers separately |

These concepts form the foundation for understanding more advanced SQS capabilities.

---

# 17. Common Mistakes

### 1. Assuming Amazon SQS Processes Messages

Amazon SQS stores and delivers messages.

The backend consumer is responsible for processing them.

### 2. Confusing Producers and Consumers

A producer sends messages to the queue.

A consumer retrieves and processes messages.

### 3. Sending Large Files Directly Through SQS

SQS is intended for messages, not large media files.

Store large files in Amazon S3 and send their references through SQS.

### 4. Assuming SQS Automatically Scales Backend Servers

SQS does not directly scale EC2 instances.

Backend scaling must be configured separately using suitable scaling policies and metrics.

### 5. Assuming Messages Remain in the Queue Forever

SQS messages are retained only for the configured retention period.

### 6. Scaling Frontend and Backend Using the Same Metrics

The frontend and backend perform different workloads.

Frontend scaling may depend on CPU utilization or incoming traffic, while backend scaling may depend on queue backlog and processing delays.

### 7. Assuming Decoupling Eliminates All Failures

SQS improves resilience, but applications still need appropriate error handling and message-processing logic.

---

# 18. Amazon SQS Architecture Summary

### Without Amazon SQS

```text
                    Users
                      |
                      v
                   Frontend
                      |
                      v
                   Backend
                      |
                      v
                   Database

Problem:
Backend may become overloaded
during traffic spikes.
```

### With Amazon SQS

```text
                    Users
                      |
                      v
                   Frontend
                      |
                      v
                  Amazon SQS
                      |
                      v
                   Backend
                      |
                      v
                   Database

Benefit:
Messages are buffered and processed
according to backend capacity.
```

---

# ❓ Interview Questions

### Q1. What is Amazon SQS?

Amazon SQS is a fully managed message queuing service used to decouple distributed application components.

### Q2. What is a decoupled architecture?

A decoupled architecture reduces direct dependencies between application components, allowing them to operate and scale more independently.

### Q3. Why is Amazon SQS used in distributed applications?

It allows applications to communicate asynchronously and helps manage differences between request arrival rates and processing capacity.

### Q4. What is a producer in Amazon SQS?

A producer is an application component that sends messages to an SQS queue.

### Q5. What is a consumer in Amazon SQS?

A consumer retrieves messages from the queue and performs the required processing.

### Q6. What is message polling?

Polling is the process of retrieving available messages from an SQS queue.

### Q7. What is the default message retention period in Amazon SQS?

Four days.

### Q8. What is the maximum message retention period?

Fourteen days.

### Q9. How should large video files be processed using SQS?

Store the video files in Amazon S3 and send messages containing their object locations through SQS.

### Q10. What happens when the backend becomes temporarily unavailable?

Messages can remain in the queue until consumers become available again, subject to the configured retention period.

### Q11. Does Amazon SQS automatically scale backend EC2 instances?

No. Backend scaling must be configured using suitable scaling mechanisms and metrics.

### Q12. Which SQS metric indicates the approximate number of messages waiting to be processed?

`ApproximateNumberOfMessagesVisible`.

### Q13. Which metric helps identify how long messages have been waiting?

`ApproximateAgeOfOldestMessage`.

### Q14. Why should frontend and backend services scale independently?

Because they perform different workloads and may require different amounts of computing capacity.

### Q15. What is the role of SQS in a video-processing application?

SQS stores messages identifying uploaded videos so backend workers can retrieve and process them asynchronously.

### Q16. Can Amazon SQS improve application resilience?

Yes. It can buffer requests during temporary backend failures or processing delays.

### Q17. Does SQS store application files?

SQS stores messages. Large application files are better stored in services such as Amazon S3.

### Q18. What is the main architectural advantage of Amazon SQS?

It decouples producers from consumers and enables asynchronous processing with independent scaling.

---

# 💡 Key Takeaways

- Amazon SQS is a fully managed message queuing service.
- It helps build decoupled, distributed application architectures.
- SQS uses a producer-consumer messaging model.
- Producers send messages to queues.
- Consumers poll queues and process messages.
- SQS acts as a buffer between frontend and backend applications.
- The frontend can accept requests without waiting for backend processing to finish.
- Backend workers can retrieve messages according to their processing capacity.
- The default message retention period is four days.
- The maximum retention period is fourteen days.
- Large files should be stored in Amazon S3, with object references sent through SQS.
- SQS enables frontend and backend components to scale independently.
- Queue backlog and message age are useful metrics for backend scaling.
- SQS is useful for workloads such as order processing and video transcoding.
- Decoupling can improve application scalability and resilience.

---

## The Most Important Concept to Remember

```text
                     Users
                       |
                       v
                    Frontend
                       |
                       | Send Messages
                       v
              ┌──────────────────┐
              │    Amazon SQS    │
              │                  │
              │  Message Queue   │
              └──────────────────┘
                       |
                       | Poll Messages
                       v
                 Backend Workers
                       |
                       v
                 Process Requests
                       |
                       v
                    Database
```

**Producer = Sends messages**

**Amazon SQS = Stores and buffers messages**

**Consumer = Retrieves and processes messages**

**Decoupling = Enables independent operation and scaling**

---

# 📚 Related Topics

- AWS Application Integration Services
- Amazon Simple Queue Service (SQS)
- Decoupled Architectures
- Event-Driven Architecture
- Microservices
- Amazon EC2 Auto Scaling
- Amazon CloudWatch Metrics
- Amazon S3
- Asynchronous Processing
- Distributed Systems

