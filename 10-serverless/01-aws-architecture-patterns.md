# AWS Architecture Patterns – From Monolithic to Event-Driven Serverless

---

## 1. Introduction

Before exploring serverless services such as AWS Lambda, it is important to understand how application architectures have evolved.

The lecture discusses five architectural approaches:

1. Monolithic Architecture
2. Modular Monolithic / Three-Tier Architecture
3. Decoupled Architecture
4. Microservices Architecture
5. Event-Driven Architecture

Each architecture addresses different requirements related to scalability, availability, deployment, maintenance, and application dependencies.

### What Does Serverless Mean?

Serverless does not mean that servers do not exist.

It means AWS manages the underlying server infrastructure, allowing developers to focus more on application functionality rather than provisioning and maintaining servers.

**Key concept: Serverless reduces server-management responsibilities; it does not eliminate servers.**

---

## 2. Monolithic Architecture

A monolithic architecture packages the entire application as a single deployable unit.

The frontend, backend, business logic, and potentially the database all run together.

### Architecture

```text
                    USERS
                      |
                      v
                 WEB BROWSER
                      |
                      v
              +-------------------+
              |   EC2 INSTANCE    |
              |                   |
              |   Frontend UI     |
              |        |          |
              |   Backend Logic   |
              |        |          |
              |   Authentication  |
              |   Product Catalog |
              |   Shopping Cart   |
              |   Payment Service |
              |        |          |
              |   MySQL Database  |
              +-------------------+
```

### Example

A small WordPress website hosted on one EC2 instance, including its web server, application, and database.

### Characteristics

- Application components are tightly coupled.
- Components are typically built and deployed together.
- Scaling often requires scaling the complete application.
- Changes may require redeploying the entire application.
- Multiple development teams may need to coordinate releases.

### Advantages

- Simple initial architecture
- Easier development for small applications
- Fewer infrastructure components
- Straightforward deployment
- Suitable for small workloads

### Disadvantages

- Limited independent scalability
- Increased deployment risk
- Tight dependencies
- Potential single points of failure
- Difficult maintenance as applications grow

### Example: Payment Service Update

Suppose an e-commerce application contains:

- Authentication
- Product catalog
- Shopping cart
- Payment processing

A developer changes the payment component.

In a traditional monolithic deployment, the entire application may need to be rebuilt and redeployed.

This increases the risk of affecting unrelated components.

**Important clarification:** Monolithic applications do not inherently require downtime for every deployment. Techniques such as redundant instances and rolling deployments can reduce or avoid downtime.

### AWS Exam Point

Monolithic architectures are suitable for relatively simple applications but can become difficult to maintain and scale independently as complexity increases.

---

## 3. Modular Monolithic / Three-Tier Architecture

The lecture next introduces a modular architecture with separate application tiers.

The frontend, backend, and database are deployed on different resources.

However, the application may remain tightly coupled through synchronous communication.

### Architecture

```text
                      USERS
                        |
                        v
                   WEB BROWSER
                        |
                        v
                +----------------+
                |  FRONTEND TIER |
                |   Web Server   |
                +----------------+
                        |
                  HTTP/API Call
                        |
                        v
                +----------------+
                |  BACKEND TIER  |
                | Business Logic |
                +----------------+
                        |
                     SQL Query
                        |
                        v
                +----------------+
                | DATABASE TIER  |
                |   MySQL / RDS  |
                +----------------+
```

### Request Flow

1. A user opens the application.
2. The browser sends a request to the frontend.
3. The frontend communicates with the backend.
4. The backend processes business logic.
5. The backend retrieves or updates database records.
6. The response travels back to the user.

### Benefits

- Separation of infrastructure tiers
- Independent infrastructure scaling
- Easier database management
- Better infrastructure organization
- More flexibility than a single-server deployment

### Remaining Challenge

The frontend still depends on the backend being available.

If the backend becomes unavailable, requests requiring backend processing can fail.

This is an example of **tight runtime coupling**.

### Important Distinction

A three-tier architecture describes the separation of presentation, application, and data layers.

A modular monolith describes how application code is organized and deployed.

They are related concepts, but they are not identical. A three-tier application can contain a monolithic backend or independently deployable services.

---

## 4. Improving Availability with Auto Scaling and Load Balancing

The lecture demonstrates how AWS infrastructure services improve a three-tier application.

### AWS Services

- Application Load Balancer
- Amazon EC2
- EC2 Auto Scaling
- Internal Application Load Balancer
- Amazon RDS Multi-AZ

### Architecture

```text
                           USERS
                             |
                             v
                     INTERNET-FACING ALB
                             |
                  +----------+----------+
                  |                     |
                  v                     v
             WEB EC2 - AZ A        WEB EC2 - AZ B
                  |                     |
                  +----------+----------+
                             |
                     WEB AUTO SCALING
                             |
                             v
                       INTERNAL ALB
                             |
                  +----------+----------+
                  |                     |
                  v                     v
             APP EC2 - AZ A        APP EC2 - AZ B
                  |                     |
                  +----------+----------+
                             |
                     APP AUTO SCALING
                             |
                             v
                       AMAZON RDS
                             |
                  +----------+----------+
                  |                     |
                  v                     v
                PRIMARY              STANDBY
                  AZ A                  AZ B
```

### Improvements

**High Availability:** Multiple EC2 instances across Availability Zones reduce dependence on one server.

**Independent Scaling:** Frontend and backend Auto Scaling groups can scale separately.

**Load Distribution:** Load balancers distribute traffic among healthy instances.

**Database Resilience:** RDS Multi-AZ provides standby capacity and automatic failover for supported configurations.

### Remaining Limitation

Although the infrastructure is more resilient, synchronous dependencies remain.

If the entire backend service becomes unavailable, frontend operations requiring that service may fail.

### AWS Exam Point

Load balancing and Auto Scaling improve availability and scalability, but they do not automatically eliminate tight coupling between application components.

---

## 5. Application Deployment Strategies

The lecture introduces two deployment strategies.

### 5.1 Blue/Green Deployment

Blue/green deployment uses two application environments.

- **Blue:** Existing application version
- **Green:** New application version

```text
                       USERS
                         |
                         v
                    LOAD BALANCER
                         |
              +----------+----------+
              |                     |
              v                     v
        BLUE ENVIRONMENT      GREEN ENVIRONMENT
           Version 1             Version 2
           Current               New Release
```

### Deployment Process

1. The existing application runs in Blue.
2. Deploy the new version to Green.
3. Test the Green environment.
4. Shift traffic to Green.
5. Retain Blue temporarily for rollback if appropriate.

### Benefits

- Reduced deployment disruption
- Testing before traffic migration
- Faster rollback options
- Better release control

### 5.2 Rolling Deployment

Rolling deployment updates application instances gradually.

```text
INITIAL STATE

EC2-1: Version 1
EC2-2: Version 1
EC2-3: Version 1

FIRST BATCH

EC2-1: Version 2
EC2-2: Version 1
EC2-3: Version 1

SECOND BATCH

EC2-1: Version 2
EC2-2: Version 2
EC2-3: Version 1

FINAL STATE

EC2-1: Version 2
EC2-2: Version 2
EC2-3: Version 2
```

### Benefits

- Gradual upgrades
- Existing instances continue serving traffic
- Reduced release impact
- Supports continuous application availability when correctly configured

### Comparison

| Feature | Blue/Green | Rolling |
|---|---|---|
| Deployment Method | Separate environments | Gradual instance updates |
| Additional Capacity | Often requires more | Depends on batch strategy |
| Rollback | Often faster through traffic switching | May require reverting updated instances |
| Traffic Migration | Switch between environments | Gradually replace instances |
| Main Benefit | Release isolation | Incremental deployment |

---

## 6. Decoupled Architecture

The next architectural improvement is decoupling application components.

Instead of having the frontend directly depend on the backend, an intermediate messaging service stores requests until they can be processed.

The lecture uses **Amazon SQS**.

### Synchronous Architecture

```text
FRONTEND
    |
    | Request
    v
BACKEND
    |
    | Response
    v
FRONTEND
```

The frontend expects the backend to respond.

If the backend is unavailable, the request may fail.

### Asynchronous Architecture

```text
FRONTEND
    |
    | Submit Message
    v
AMAZON SQS QUEUE
    |
    | Poll Message
    v
BACKEND WORKERS
    |
    v
DATABASE
```

The frontend submits work to the queue.

The backend processes it when capacity becomes available.

### Benefits

- Reduced direct dependency
- Better workload buffering
- Independent worker scaling
- Improved resilience during processing spikes
- Support for background processing

### Important Concept

**Amazon SQS decouples producers from consumers.**

---

## 7. Understanding Synchronous vs. Asynchronous Communication

| Feature | Synchronous | Asynchronous |
|---|---|---|
| Communication | Direct request/response | Message or event-based |
| Backend Availability | Usually required for immediate response | Processing can be deferred |
| Caller Waiting | Usually waits | Can continue after acknowledgment |
| Coupling | Often tighter | Often looser |
| Common AWS Pattern | ALB → EC2 Backend | Application → SQS → Workers |
| Typical Use Case | Retrieve product details | Process uploaded videos |

### Example: Video Processing

A user uploads a video.

The application needs to convert it into multiple formats.

### Synchronous Design

```text
USER
 |
 v
UPLOAD VIDEO
 |
 v
BACKEND
 |
 v
CONVERT VIDEO
 |
 v
RETURN RESULT
```

The user may have to wait while the conversion completes.

### Asynchronous Design

```text
USER
 |
 v
UPLOAD VIDEO
 |
 v
STORE VIDEO IN S3
 |
 v
SEND PROCESSING MESSAGE
 |
 v
AMAZON SQS
 |
 v
PROCESSING WORKERS
 |
 v
GENERATE VIDEO FORMATS
 |
 v
STORE OUTPUT IN S3
```

The application can acknowledge the upload while processing continues separately.

### AWS Exam Point

For background tasks such as image processing, video conversion, and report generation, asynchronous processing is often a good architectural choice.

---

## 8. Amazon SQS Message Processing

The lecture explains the message-processing lifecycle.

### Architecture

```text
                   FRONTEND APPLICATION
                            |
                            v
                       AMAZON SQS
                            |
                            v
                      MESSAGE QUEUE
                            |
                            v
                       EC2 WORKER
                            |
                            v
                      PROCESS MESSAGE
                            |
                            v
                       DELETE MESSAGE
```

### Processing Steps

1. The frontend sends a message to SQS.
2. SQS stores the message.
3. A backend worker polls the queue.
4. The worker receives the message.
5. The worker processes the task.
6. After successful processing, the worker deletes the message.

### Important SQS Concepts

- Visibility timeout
- Message retention
- Long polling
- Dead-letter queues
- Standard and FIFO queues

### Important Clarification

Amazon SQS Standard queues do not guarantee strict first-in-first-out ordering.

If strict ordering is required, consider SQS FIFO queues and their ordering constraints.

Also, receiving a message does not automatically delete it. The consumer must explicitly delete the message after processing.

---

## 9. Scaling Backend Workers Based on Queue Length

The lecture introduces a useful architecture: scaling backend resources based on queued work.

### Architecture

```text
                      FRONTEND
                         |
                         v
                     AMAZON SQS
                         |
                  QUEUE BACKLOG
                         |
                         v
                  CLOUDWATCH METRIC
                         |
                         v
                   AUTO SCALING
                         |
            +------------+------------+
            |            |            |
            v            v            v
         WORKER 1     WORKER 2     WORKER 3
            |            |            |
            +------------+------------+
                         |
                         v
                    PROCESS TASKS
```

### Example Scaling Configuration

| Setting | Example |
|---|---:|
| Minimum Capacity | 0 |
| Desired Capacity | 2 |
| Maximum Capacity | 25 |

These are illustrative values from the lecture, not required settings.

### Scaling Behavior

**High Queue Backlog:** Add processing capacity.

**Low Queue Backlog:** Reduce processing capacity.

**Empty Queue:** Scale down if supported by the worker architecture and scaling configuration.

### Production Considerations

Queue length alone may not be sufficient for optimal scaling.

A useful scaling metric can be backlog per available worker.

For example:

```text
Backlog per Worker =
Visible Messages / Active Workers
```

Additional considerations include:

- Average processing time
- Acceptable processing latency
- Worker startup time
- In-flight messages
- Safe scale-in behavior

### AWS Exam Point

**Amazon SQS + EC2 Auto Scaling = Decoupled, scalable background processing.**

---

## 10. Microservices Architecture

The lecture next introduces microservices.

A microservices architecture divides an application into smaller services based on business capabilities.

Each service has a specific responsibility.

### Example: Recipe Application

The instructor uses a recipe application with multiple services.

```text
                         CLIENT
                           |
                           v
                       API LAYER
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   RECIPE SERVICE     SEARCH SERVICE     IMAGE SERVICE
        |                  |                  |
        v                  v                  v
   RECIPE DATABASE    SEARCH DATASTORE     AMAZON S3

                           |
                           v
                      EMAIL SERVICE
```

### Example Services

| Microservice | Responsibility |
|---|---|
| Recipe Service | Create and retrieve recipes |
| Search Service | Search available recipes |
| Image Service | Upload and process images |
| Email Service | Send application notifications |

### Characteristics

- Independent codebases
- Independent deployments
- Independent scaling
- Service-specific functionality
- Communication through APIs or messaging
- Potentially different programming languages

### Important Concept

**Microservices divide applications according to business capabilities rather than only technical layers.**

---

## 11. Independent Microservice Deployment

Consider an application with four microservices.

```text
RECIPE SERVICE       Version 3
SEARCH SERVICE       Version 2
IMAGE SERVICE        Version 5
EMAIL SERVICE        Version 1
```

A developer updates the image-processing service.

Only the image service may need to be redeployed.

The remaining services can continue running.

### Benefits

- Independent releases
- Smaller deployment scope
- Team autonomy
- Service-specific scaling
- Technology flexibility

### Example

The recipe service might use Python.

The search service might use Java.

The image service might use Node.js.

They communicate through defined APIs or event contracts.

### AWS Exam Point

Microservices allow independent development, deployment, and scaling when service boundaries are designed appropriately.

---

## 12. Microservices Are Not Automatically Decoupled

One of the most important concepts in the lecture is that microservices can still be tightly coupled.

### Example

```text
RECIPE SERVICE
      |
      | Synchronous API Call
      v
SEARCH SERVICE
      |
      | Synchronous API Call
      v
IMAGE SERVICE
```

If the image service becomes unavailable, the search service may fail.

That failure can propagate to the recipe service.

### Potential Problems

- Cascading failures
- Increased response latency
- Service dependencies
- Difficult troubleshooting
- Complex distributed communication

### Improved Architecture

```text
RECIPE SERVICE
      |
      v
AMAZON SQS
      |
      v
IMAGE PROCESSING SERVICE
```

The producer and consumer no longer need to be available simultaneously for the message to be accepted and later processed.

### Important Concept

**Microservices describe application boundaries. Decoupling describes how those services interact.**

A microservices architecture can use both synchronous and asynchronous communication.

---

## 13. Communication Between Microservices

The lecture discusses two communication styles.

### Synchronous Communication

Services communicate using direct API calls.

```text
SERVICE A
    |
    | HTTP Request
    v
SERVICE B
    |
    | HTTP Response
    v
SERVICE A
```

### Asynchronous Communication

Services communicate using messaging or events.

```text
SERVICE A
    |
    v
SQS / SNS / EVENTBRIDGE
    |
    v
SERVICE B
```

### AWS Services

| Service | Typical Role |
|---|---|
| Amazon SQS | Message queuing and buffering |
| Amazon SNS | Publish/subscribe notifications |
| Amazon EventBridge | Event routing using event buses and rules |
| Amazon API Gateway | Managed API entry point |
| AWS Lambda | Event-driven compute |

### AWS Exam Point

Choose the integration service based on the communication requirement.

SQS, SNS, and EventBridge are not interchangeable in every scenario.

---

## 14. Event-Driven Architecture

The lecture introduces event-driven architecture as another evolution of application design.

Instead of components continuously making direct requests to one another, applications react to events.

### What Is an Event?

An event represents something that happened.

Examples:

- A file was uploaded.
- A customer placed an order.
- A payment completed.
- An application reported an error.
- A user registered.
- An EC2 instance changed state.

### Main Components

1. Event Producer
2. Event Router
3. Event Consumer

### Architecture

```text
                       EVENT PRODUCERS
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          AMAZON S3      APPLICATIONS     AWS SERVICES
              |               |               |
              +---------------+---------------+
                              |
                              v
                         EVENT ROUTER
                              |
                              v
                        EVENTBRIDGE
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           LAMBDA A        LAMBDA B        SQS QUEUE
              |               |               |
              v               v               v
          PROCESSING      NOTIFICATION      WORKERS
```

### Important Concept

**Event-Driven Architecture = Applications react to events rather than relying entirely on direct synchronous communication.**

---

## 15. Event Producers

An event producer generates an event when something happens.

### Examples

- Amazon S3 object creation
- Application order submission
- Payment completion
- EC2 state changes
- Scheduled application events

### Example

A customer uploads an image to an S3 bucket.

```text
CUSTOMER
   |
   v
UPLOAD IMAGE
   |
   v
AMAZON S3
   |
   v
OBJECT CREATED EVENT
```

The S3 object creation becomes an event that can initiate downstream processing.

---

## 16. Event Routers

An event router directs events to interested consumers.

The lecture introduces an event bus as part of this architecture.

### Example: Amazon EventBridge

```text
                     EVENT PRODUCER
                           |
                           v
                      EVENTBRIDGE
                           |
                        EVENT BUS
                           |
                           v
                       EVENT RULES
                           |
              +------------+------------+
              |            |            |
              v            v            v
           LAMBDA         SQS          SNS
```

### Responsibilities

- Receive events
- Match events against rules
- Route events to supported targets
- Support event-based application integration

### Important Concept

Amazon EventBridge is a managed event-routing service.

---

## 17. Event Consumers

Event consumers react to relevant events.

### Example

An image upload event triggers an image-processing Lambda function.

```text
                    IMAGE UPLOADED
                          |
                          v
                       AMAZON S3
                          |
                          v
                    EVENT NOTIFICATION
                          |
                          v
                      AWS LAMBDA
                          |
                          v
                     RESIZE IMAGE
                          |
                          v
                    SAVE OUTPUT TO S3
```

### Processing Flow

1. A user uploads an image.
2. S3 generates an object creation event.
3. The event is delivered through a configured integration.
4. A Lambda function is invoked.
5. Lambda processes the image.
6. The processed image is saved.

### Important Concept

Consumers respond to events based on their configured event subscriptions or routing rules.

---

## 18. Why Event-Driven Architecture Works Well with Serverless

Traditional application servers are often provisioned to remain available for incoming requests.

For example:

```text
EC2 BACKEND
     |
     v
RUNNING CONTINUOUSLY
     |
     v
WAITING FOR REQUESTS
```

In an event-driven Lambda architecture:

```text
NO EVENT
    |
    v
NO FUNCTION INVOCATION

EVENT OCCURS
    |
    v
LAMBDA INVOKED
    |
    v
PROCESS EVENT
    |
    v
INVOCATION COMPLETES
```

### Benefits

- No need to maintain always-running application servers for suitable functions
- Automatic scaling of supported event-driven workloads
- Reduced infrastructure management
- Usage-based compute billing
- Better alignment with intermittent workloads

### Important Clarification

Event-driven architecture does not necessarily mean that no infrastructure runs continuously.

Event consumers can be EC2 instances, containers, or continuously running services.

The lecture's description of compute running only when events occur applies particularly well to serverless event consumers such as AWS Lambda.

---

## 19. AWS Lambda and Event-Driven Processing

AWS Lambda is a serverless compute service that executes code in response to invocations.

### Example

```text
                      AMAZON S3
                          |
                          v
                   OBJECT CREATED
                          |
                          v
                      AWS LAMBDA
                          |
                          v
                    PROCESS OBJECT
                          |
                          v
                      AMAZON S3
```

### Key Characteristics

- No server provisioning required
- Managed execution environment
- Automatic scaling within service limits
- Event-driven execution
- Usage-based billing

### Important Concept

Lambda does not require customers to maintain a continuously running application server to wait for incoming events.

### AWS Exam Point

**AWS Lambda = Serverless compute for event-driven workloads.**

---

## 20. Architecture Evolution Summary

The lecture demonstrates the progression from tightly coupled applications toward independently scalable, event-driven designs.

### Stage 1 – Monolithic

```text
USER
 |
 v
SINGLE APPLICATION
 |
 v
DATABASE
```

**Challenge:** All application components are packaged together.

### Stage 2 – Three-Tier

```text
USER
 |
 v
FRONTEND
 |
 v
BACKEND
 |
 v
DATABASE
```

**Improvement:** Infrastructure tiers can scale separately.

**Remaining Challenge:** Synchronous dependencies.

### Stage 3 – Decoupled

```text
FRONTEND
   |
   v
AMAZON SQS
   |
   v
BACKEND WORKERS
   |
   v
DATABASE
```

**Improvement:** Workloads can be buffered and processed asynchronously.

### Stage 4 – Microservices

```text
                    APPLICATION
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
       SERVICE A      SERVICE B      SERVICE C
```

**Improvement:** Business capabilities can be independently developed, deployed, and scaled.

### Stage 5 – Event-Driven

```text
EVENT PRODUCERS
       |
       v
EVENT ROUTER
       |
       v
EVENT CONSUMERS
       |
       v
BUSINESS ACTIONS
```

**Improvement:** Services react to events, reducing direct runtime dependencies where asynchronous communication is appropriate.

---

## 21. Architecture Comparison

| Feature | Monolithic | Three-Tier | Decoupled | Microservices | Event-Driven |
|---|---|---|---|---|---|
| Application Structure | Single deployable unit | Separate technical tiers | Producer and consumer separated | Business-oriented services | Event producers and consumers |
| Scaling | Often whole application | Per infrastructure tier | Independently scalable workers | Per service | Per consumer/workload |
| Communication | Mostly internal calls | Often synchronous | Asynchronous | Synchronous or asynchronous | Event-based |
| Deployment | Often whole application | Tier-dependent | Component-dependent | Independent services | Independent event handlers |
| Complexity | Lower initially | Moderate | Moderate | Higher | Depends on design |
| AWS Examples | EC2 | ALB, EC2, RDS | SQS, Auto Scaling | ECS, Lambda, APIs | EventBridge, Lambda |

### Important Observation

These patterns are not mutually exclusive.

A microservices application can be event-driven.

A three-tier application can use SQS.

A monolithic application can be deployed across multiple EC2 instances.

The appropriate architecture depends on business and technical requirements.

---

## 22. Real-World Example: E-Commerce Order Processing

Consider an online shopping platform.

### Traditional Architecture

```text
CUSTOMER
   |
   v
WEB APPLICATION
   |
   v
ORDER PROCESSING
   |
   v
PAYMENT PROCESSING
   |
   v
INVENTORY UPDATE
   |
   v
EMAIL NOTIFICATION
```

Each operation may depend on the previous operation.

### Event-Driven Architecture

```text
                       CUSTOMER
                          |
                          v
                      PLACE ORDER
                          |
                          v
                     ORDER SERVICE
                          |
                          v
                    ORDER CREATED EVENT
                          |
                          v
                      EVENTBRIDGE
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
      INVENTORY        ANALYTICS       NOTIFICATION
       SERVICE          SERVICE          SERVICE
```

### Benefits

- Independent downstream processing
- Reduced direct dependencies
- Independent scaling
- Easier addition of new event consumers

### Important Consideration

Not every operation should be asynchronous.

For example, an application may require immediate confirmation of payment authorization before completing an order.

Architecture design must account for business consistency and response-time requirements.

---

## 23. Real-World Example: Video Processing

The lecture specifically mentions video conversion as a suitable asynchronous workload.

### Architecture

```text
                         USER
                          |
                          v
                     UPLOAD VIDEO
                          |
                          v
                       AMAZON S3
                          |
                          v
                    PROCESSING EVENT
                          |
                          v
                      AMAZON SQS
                          |
                          v
                    PROCESSING WORKERS
                          |
                          v
                   CONVERT VIDEO FORMATS
                          |
                          v
                       AMAZON S3
                          |
                          v
                     OUTPUT VIDEOS
```

### Why Is This a Good Design?

Video conversion can take time.

The frontend does not need to wait for the entire conversion process.

Workers can process tasks based on available capacity.

### Alternative

For suitable workloads, an event-driven design could use AWS Lambda or a specialized media-processing service.

The choice depends on processing duration, compute requirements, and supported service limits.

---

## 24. Real-World Example: Image Processing

A recipe application allows users to upload images.

After uploading, the application needs to:

- Resize the image
- Generate a thumbnail
- Add a watermark
- Store processed images

### Architecture

```text
                     RECIPE APPLICATION
                             |
                             v
                       UPLOAD IMAGE
                             |
                             v
                          AMAZON S3
                             |
                             v
                     OBJECT CREATED EVENT
                             |
                             v
                         AWS LAMBDA
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
            RESIZE        THUMBNAIL       WATERMARK
              |              |              |
              +--------------+--------------+
                             |
                             v
                          AMAZON S3
```

### Important Concept

Event-driven processing allows the image-processing workflow to operate separately from the application request path.

---

## 25. Production Architecture Considerations

The lecture focuses on architecture patterns and their evolution.

The following are additional production-design considerations that build on those concepts.

### 25.1 Retry Handling

Asynchronous processing can fail.

Applications should define how failed operations are retried.

### 25.2 Dead-Letter Queues

Amazon SQS dead-letter queues can help isolate messages that repeatedly fail processing.

### 25.3 Idempotency

An idempotent consumer can safely handle repeated processing attempts without producing unintended duplicate business effects.

### 25.4 Observability

Distributed architectures require visibility into:

- Request latency
- Error rates
- Queue backlog
- Consumer failures
- Event processing time
- Service dependencies

### 25.5 Security

Services should use appropriate IAM permissions and network controls.

### 25.6 Eventual Consistency

Asynchronous workflows may not update every component immediately.

Applications should account for delayed propagation of business state.

### 25.7 Cost Management

Serverless and event-driven architectures can reduce idle compute costs, but charges still depend on requests, invocations, processing, data transfer, and service usage.

---

## 26. AWS Certification Exam Scenarios

### Scenario 1 – Small Website

A company needs to host a simple website with limited traffic and minimal architectural complexity.

**Possible Answer:** A simple monolithic deployment may be sufficient.

### Scenario 2 – Highly Available Three-Tier Application

A company needs multiple web and application servers distributed across Availability Zones.

**Answer:** Application Load Balancer + EC2 Auto Scaling + suitable Multi-AZ database architecture.

### Scenario 3 – Independent Tier Scaling

A company wants to scale its frontend and backend infrastructure separately.

**Answer:** Separate Auto Scaling groups for each tier.

### Scenario 4 – Reduce Deployment Downtime

A company wants to deploy a new application version while retaining the previous environment for rollback.

**Answer:** Blue/Green Deployment.

### Scenario 5 – Gradual Application Updates

A company wants to update instances in batches while maintaining application availability.

**Answer:** Rolling Deployment.

### Scenario 6 – Decouple Frontend and Backend

A company wants the frontend to accept processing requests even when backend workers are temporarily unavailable.

**Answer:** Amazon SQS.

### Scenario 7 – Scale Based on Pending Work

A company wants EC2 workers to scale based on the number of queued messages.

**Answer:** Amazon SQS + CloudWatch Metrics + EC2 Auto Scaling.

### Scenario 8 – Video Conversion

A company needs to process uploaded videos without making users wait for conversion.

**Answer:** Asynchronous processing using SQS and suitable workers.

### Scenario 9 – Independent Business Services

A company wants separate teams to deploy payment, catalog, and inventory functionality independently.

**Answer:** Microservices Architecture.

### Scenario 10 – Event-Based Processing

A company wants to trigger processing when a file is uploaded to S3.

**Answer:** S3 Event Notifications or EventBridge integration with an appropriate target such as Lambda.

### Scenario 11 – Event Routing

A company wants to route different application events to different consumers using event rules.

**Answer:** Amazon EventBridge.

### Scenario 12 – Serverless Compute

A company wants to execute code when events occur without maintaining application servers.

**Answer:** AWS Lambda.

---

## 27. Interview Questions

### Q1. What is a monolithic architecture?

An architecture in which application components are packaged and deployed together as a single unit.

### Q2. What is the main disadvantage of a monolithic architecture?

Tight coupling can make independent deployment and scaling difficult.

### Q3. What is a three-tier architecture?

An architecture separating presentation, application processing, and data storage.

### Q4. What AWS services can improve three-tier availability?

Elastic Load Balancing, EC2 Auto Scaling, and RDS Multi-AZ are common examples.

### Q5. What is tight coupling?

A situation where one component depends directly on another component's availability or behavior.

### Q6. What is decoupling?

Reducing direct dependencies between application components.

### Q7. How does Amazon SQS support decoupling?

It buffers messages so producers and consumers do not need to process requests simultaneously.

### Q8. What is synchronous communication?

Communication where the caller waits for a response from another component.

### Q9. What is asynchronous communication?

Communication where processing can occur separately from the original request.

### Q10. What is a microservices architecture?

An architecture that divides applications into independently deployable business services.

### Q11. Are microservices always loosely coupled?

No. Microservices using synchronous APIs can still have strong runtime dependencies.

### Q12. What is an event-driven architecture?

An architecture in which consumers react to events produced by other components.

### Q13. What is an event producer?

A component that generates an event when something happens.

### Q14. What is an event consumer?

A component that processes or responds to an event.

### Q15. What is an event router?

A component that directs events to interested consumers.

### Q16. What AWS service provides event buses and rules?

Amazon EventBridge.

### Q17. What AWS service provides managed message queues?

Amazon SQS.

### Q18. What is AWS Lambda?

A serverless compute service that executes code in response to invocations.

### Q19. What is Blue/Green Deployment?

A deployment strategy using separate existing and new application environments.

### Q20. What is Rolling Deployment?

A deployment strategy that updates application instances incrementally.

### Q21. Can backend workers scale based on SQS queue depth?

Yes, using suitable CloudWatch metrics and scaling policies.

### Q22. Does serverless mean AWS does not use servers?

No. AWS manages the underlying infrastructure.

### Q23. Does event-driven architecture require Lambda?

No. Events can be consumed by different types of compute and application services.

### Q24. Can a microservices architecture also be event-driven?

Yes.

### Q25. What is the key advantage of asynchronous processing?

It reduces direct runtime dependencies and allows work to be buffered and processed independently.

---

## 28. Key Takeaways

1. Serverless means customers do not manage the underlying servers.
2. Monolithic applications package components together.
3. Monolithic applications can be appropriate for simple workloads.
4. Three-tier architectures separate frontend, backend, and database resources.
5. Load balancers improve traffic distribution.
6. Auto Scaling improves capacity management.
7. RDS Multi-AZ improves database availability.
8. Blue/green deployment supports traffic switching between application environments.
9. Rolling deployment updates application instances incrementally.
10. Synchronous communication creates runtime dependencies.
11. Amazon SQS supports asynchronous decoupling.
12. Backend workers can scale based on queue backlog.
13. Microservices divide applications by business capability.
14. Microservices can be independently deployed and scaled.
15. Microservices can still be tightly coupled through synchronous APIs.
16. Event-driven architectures use event producers and consumers.
17. Amazon EventBridge provides event routing.
18. AWS Lambda supports event-driven serverless compute.
19. Not every workload is suitable for asynchronous processing.
20. Architecture selection depends on scalability, availability, operational complexity, and business requirements.

---

## 29. AWS Exam Cheat Sheet

| Requirement | Architecture or AWS Service |
|---|---|
| Simple Application | Monolithic Architecture |
| Separate Frontend, Backend, Database | Three-Tier Architecture |
| Distribute Web Traffic | Elastic Load Balancing |
| Automatically Scale EC2 Capacity | EC2 Auto Scaling |
| Database Failover | Amazon RDS Multi-AZ |
| Deploy New Environment Before Traffic Switch | Blue/Green Deployment |
| Update Instances in Batches | Rolling Deployment |
| Decouple Application Components | Amazon SQS |
| Scale Workers Based on Queue Backlog | SQS + CloudWatch + Auto Scaling |
| Independent Business Services | Microservices |
| Publish/Subscribe Messaging | Amazon SNS |
| Event Bus and Event Routing | Amazon EventBridge |
| Run Code in Response to Events | AWS Lambda |
| Serverless Application Processing | Event-Driven Architecture with Suitable Serverless Services |

---

## 30. Final Architecture Summary

```text
                     ARCHITECTURE EVOLUTION
                     ======================

                         MONOLITHIC
                             |
                             v
                  SINGLE APPLICATION UNIT
                             |
                             v
                         THREE-TIER
                             |
                             v
                  FRONTEND / BACKEND / DB
                             |
                             v
                       HIGH AVAILABILITY
                             |
                             v
                  ALB + ASG + RDS MULTI-AZ
                             |
                             v
                          DECOUPLED
                             |
                             v
                      AMAZON SQS QUEUE
                             |
                             v
                      BACKEND WORKERS
                             |
                             v
                        MICROSERVICES
                             |
                             v
                  INDEPENDENT BUSINESS
                         SERVICES
                             |
                             v
                        EVENT-DRIVEN
                             |
                             v
                      EVENT PRODUCERS
                             |
                             v
                        EVENTBRIDGE
                             |
                             v
                       EVENT CONSUMERS
                             |
                             v
                         AWS LAMBDA
                             |
                             v
                    SERVERLESS PROCESSING
```

---

## 31. Related AWS Services

- Amazon EC2
- Elastic Load Balancing
- EC2 Auto Scaling
- Amazon RDS
- Amazon SQS
- Amazon SNS
- Amazon EventBridge
- AWS Lambda
- Amazon S3
- Amazon API Gateway
- Amazon ECS

---
