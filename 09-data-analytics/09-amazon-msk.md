# Amazon MSK – Fundamentals and Architecture

**Certification:** AWS Certified Solutions Architect – Associate (SAA-C03)  
**Category:** Data Analytics  
**Topic:** Amazon Managed Streaming for Apache Kafka (Amazon MSK)  
**Difficulty:** Intermediate  
**Lesson Type:** Theory  
**Prerequisites:** Amazon VPC, Availability Zones, Streaming Data Concepts, Amazon Kinesis Data Streams

---

# 1. Introduction

Modern applications continuously generate large amounts of data.

Examples include:

- Application logs
- Financial transactions
- E-commerce events
- IoT sensor readings
- Website activity
- Customer interactions
- Database changes
- Monitoring and observability events

Organizations often need to process this information continuously rather than waiting for scheduled batch processing.

This is known as **streaming data processing**.

AWS provides several services for streaming workloads, including:

- Amazon Kinesis Data Streams
- Amazon Managed Streaming for Apache Kafka (Amazon MSK)

Both services support streaming architectures, but they use different technologies.

Amazon Kinesis Data Streams is an AWS-native streaming service.

Amazon MSK provides a managed environment for running Apache Kafka.

### Important AWS Exam Point

**Amazon MSK = Managed Apache Kafka on AWS.**

If an exam question specifically mentions an existing Apache Kafka application or migrating Kafka workloads to AWS, Amazon MSK is a strong candidate.

---

# 2. What Is Amazon MSK?

Amazon MSK stands for:

**Amazon Managed Streaming for Apache Kafka**

It is a fully managed AWS service that simplifies running Apache Kafka workloads.

Apache Kafka is an open-source distributed event-streaming platform.

Applications use Kafka to publish, store, and consume streams of events.

### Key Features

| Feature | Description |
|---|---|
| Managed Apache Kafka | AWS manages supported Kafka infrastructure operations |
| Streaming Data | Process continuously generated events |
| Kafka Compatibility | Supports Apache Kafka clients and APIs |
| Kafka Brokers | Store and serve streaming records |
| Topics | Organize streams of records |
| Partitions | Distribute data within topics |
| Replication | Replicate data across brokers |
| Multi-AZ | Improve availability and resilience |
| Standard Brokers | Configurable Kafka broker infrastructure |
| Express Brokers | Managed broker option optimized for elasticity and performance |
| MSK Provisioned | Configure and manage cluster capacity |
| MSK Serverless | Automatic capacity provisioning and scaling |

### Important Concept

Amazon MSK reduces the operational effort required to deploy and operate Apache Kafka on AWS.

However, application teams are still responsible for designing their streaming applications, producers, consumers, topics, and data-processing logic.

---

# 3. Why Do We Need Amazon MSK?

Consider an organization already using Apache Kafka in its on-premises data center.

Its applications depend on Kafka for:

- Publishing business events
- Processing financial transactions
- Streaming application logs
- Integrating microservices
- Delivering events to downstream applications

The organization decides to migrate its infrastructure to AWS.

### Option 1: Self-Managed Apache Kafka

The organization could deploy Kafka manually on EC2 instances.

```text
                  APPLICATION PRODUCERS
                           |
                           v
                    SELF-MANAGED KAFKA
                           |
              +------------+------------+
              |            |            |
              v            v            v
           EC2 NODE 1   EC2 NODE 2   EC2 NODE 3
              |            |            |
              +------------+------------+
                           |
                           v
                    KAFKA CONSUMERS
```

### Challenges

The organization must manage:

- Kafka installation
- Broker configuration
- Infrastructure provisioning
- Cluster availability
- Broker failures
- Software maintenance
- Monitoring
- Storage capacity
- Scaling

### Option 2: Amazon MSK

Instead, the organization can use Amazon MSK.

```text
                  APPLICATION PRODUCERS
                           |
                           v
                       AMAZON MSK
                           |
              +------------+------------+
              |            |            |
              v            v            v
           BROKER 1     BROKER 2     BROKER 3
              |            |            |
              +------------+------------+
                           |
                           v
                    KAFKA CONSUMERS
```

AWS manages much of the underlying Kafka infrastructure.

### Business Benefits

- Reduced infrastructure management
- Kafka compatibility
- High availability options
- Managed broker infrastructure
- Support for existing Kafka applications

---

# 4. Understanding Apache Kafka

Apache Kafka is a distributed event-streaming platform.

It allows applications to publish and consume streams of records.

### Main Kafka Components

1. Producers
2. Topics
3. Partitions
4. Brokers
5. Consumers
6. Consumer Groups

### High-Level Architecture

```text
                       DATA SOURCES
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          APPLICATIONS    IoT DEVICES   DATABASE EVENTS
              |             |             |
              +-------------+-------------+
                            |
                            v
                         PRODUCERS
                            |
                            v
                       KAFKA CLUSTER
                            |
                            v
                          TOPICS
                            |
                            v
                        PARTITIONS
                            |
                            v
                         BROKERS
                            |
                            v
                         CONSUMERS
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
           ANALYTICS      STORAGE      APPLICATIONS
```

### Important Concept

Kafka separates applications producing events from applications consuming them.

This supports loosely coupled streaming architectures.

---

# 5. Amazon MSK Architecture

The lecture explains that Amazon MSK clusters are deployed within an Amazon VPC.

For high availability, brokers are distributed across multiple Availability Zones.

### Conceptual Architecture

```text
                            AWS REGION
                                |
                                v
                           AMAZON VPC
                                |
               +----------------+----------------+
               |                |                |
               v                v                v
        AVAILABILITY ZONE A  AVAILABILITY ZONE B  AVAILABILITY ZONE C
               |                |                |
               v                v                v
            SUBNET A         SUBNET B         SUBNET C
               |                |                |
               v                v                v
           MSK BROKER 1     MSK BROKER 2     MSK BROKER 3
               |                |                |
               +----------------+----------------+
                                |
                                v
                        APACHE KAFKA TOPICS
                                |
                                v
                          KAFKA CONSUMERS
```

### Architecture Components

| Component | Responsibility |
|---|---|
| Amazon VPC | Network environment |
| Availability Zones | Infrastructure fault isolation |
| MSK Cluster | Managed Kafka environment |
| Kafka Brokers | Store and serve event records |
| Topics | Organize event streams |
| Partitions | Distribute event data |
| Producers | Publish records |
| Consumers | Read records |

### Important Clarification

The lecture describes an architecture spanning three Availability Zones.

Three-AZ deployments are a common high-availability pattern, but supported MSK configurations may use different numbers of Availability Zones.

Therefore, three Availability Zones should not be treated as a universal minimum for every MSK cluster type.

---

# 6. Understanding Kafka Brokers

A broker is a Kafka server responsible for storing and serving streaming records.

A Kafka cluster typically contains multiple brokers.

### Broker Responsibilities

- Receive records from producers
- Store records in topic partitions
- Serve records to consumers
- Participate in replication
- Support distributed streaming workloads

### Architecture

```text
                       AMAZON MSK CLUSTER
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
           BROKER 1         BROKER 2         BROKER 3
              |                |                |
              v                v                v
          PARTITION A      PARTITION B      PARTITION C
              |                |                |
              +----------------+----------------+
                               |
                               v
                          CONSUMERS
```

### Important Concept

**Kafka Broker = Server responsible for storing and serving Kafka records.**

### AWS Exam Point

When a question mentions Kafka broker management, Amazon MSK is an appropriate AWS service to consider.

---

# 7. Standard Brokers

The lecture introduces Standard brokers as one of the broker options available with Amazon MSK.

Standard brokers provide configurable infrastructure for Apache Kafka workloads.

### Characteristics

- Support different broker instance types
- Allow storage capacity configuration
- Support different throughput requirements
- Support Kafka replication
- Provide control over cluster configuration

### Architecture

```text
                      AMAZON MSK
                          |
                          v
                    STANDARD BROKERS
                          |
             +------------+------------+
             |            |            |
             v            v            v
          BROKER 1     BROKER 2     BROKER 3
             |            |            |
             v            v            v
          STORAGE      STORAGE      STORAGE
```

### When to Consider Standard Brokers

Standard brokers may be appropriate when:

- You require supported configurations specific to Standard brokers.
- You need particular storage or instance configurations.
- Your application requires greater control over broker infrastructure.
- You have an existing Kafka workload with specific compatibility requirements.

### Important Concept

Standard brokers offer configurable Kafka infrastructure.

---

# 8. Express Brokers

The lecture introduces Express brokers as another Amazon MSK broker option.

Express brokers are designed to simplify storage management and improve elasticity and performance for supported workloads.

### Characteristics

- Managed storage capacity
- Automatic storage scaling
- Reduced storage administration
- High-throughput capabilities
- Low-latency streaming
- Improved scaling characteristics

### Architecture

```text
                       AMAZON MSK
                           |
                           v
                     EXPRESS BROKERS
                           |
              +------------+------------+
              |            |            |
              v            v            v
           BROKER 1     BROKER 2     BROKER 3
              |            |            |
              +------------+------------+
                           |
                           v
                   MANAGED STORAGE
                           |
                           v
                   AUTOMATIC SCALING
```

### Performance Claims Mentioned in the Lecture

The instructor describes Express brokers as offering, under applicable comparisons:

- Up to three times the throughput per broker
- Up to twenty times faster scaling
- Approximately 90% faster recovery

These are comparative service-performance claims, not guaranteed improvements for every workload.

Actual performance depends on the broker configuration, workload, and supported service features.

---

# 9. Standard vs. Express Brokers

| Feature | Standard Brokers | Express Brokers |
|---|---|---|
| Kafka Compatibility | Yes | Yes, within supported configurations |
| Storage Management | Customer-configured capacity | Managed storage scaling |
| Broker Configuration | More configurable | More managed |
| Scaling | Configurable | Optimized for faster scaling |
| Throughput | Depends on configuration | Designed for high throughput |
| Operational Overhead | Higher | Lower for storage management |
| Suitable Workloads | Workloads requiring supported Standard configurations | Workloads benefiting from simplified operations and elasticity |

### Important Clarification

Express brokers are not automatically the best choice for every workload.

The appropriate option depends on:

- Supported Kafka versions
- Required features
- Workload throughput
- Storage requirements
- Compatibility
- Pricing
- Operational requirements

### AWS Exam Point

**Standard Brokers = Configurable Kafka infrastructure**

**Express Brokers = Simplified storage management and enhanced elasticity**

---

# 10. Understanding Kafka Topics

A Kafka topic is a logical category used to organize streaming records.

Producers publish records to topics.

Consumers subscribe to topics to retrieve records.

### Example

An e-commerce application generates three types of events:

- Customer orders
- Payment transactions
- Inventory updates

These events can be organized into separate Kafka topics.

```text
                       E-COMMERCE APPLICATION
                                 |
                                 v
                              PRODUCER
                                 |
                 +---------------+---------------+
                 |               |               |
                 v               v               v
             ORDERS TOPIC    PAYMENTS TOPIC   INVENTORY TOPIC
                 |               |               |
                 v               v               v
              CONSUMER        CONSUMER        CONSUMER
```

### Important Concept

**Kafka Topic = Logical stream of related events.**

### AWS Exam Point

Kafka uses **topics**, while Amazon Kinesis Data Streams uses **streams**.

---

# 11. Understanding Kafka Partitions

A Kafka topic is divided into partitions.

Partitions enable distributed storage and parallel processing.

### Example

Consider an `orders` topic containing customer order events.

```text
                         ORDERS TOPIC
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          PARTITION 0     PARTITION 1     PARTITION 2
              |               |               |
              v               v               v
           BROKER 1        BROKER 2        BROKER 3
```

### Why Are Partitions Important?

Partitions support:

- Parallel data processing
- Distribution across brokers
- Horizontal scalability
- Ordered records within individual partitions

### Important Concept

Kafka guarantees record ordering within a partition, not necessarily across an entire topic.

### AWS Exam Point

**Kafka Partition = Unit of data distribution and parallelism.**

---

# 12. Kafka Partitioning Example

Suppose an application publishes order events.

### Events

```text
Order 1001
Order 1002
Order 1003
Order 1004
Order 1005
Order 1006
```

Kafka distributes the events across partitions according to the producer's partitioning logic.

```text
                       ORDERS TOPIC
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
         PARTITION 0    PARTITION 1    PARTITION 2
             |              |              |
             v              v              v
          ORDER 1001     ORDER 1002     ORDER 1003
          ORDER 1004     ORDER 1005     ORDER 1006
```

### Important Clarification

The distribution above is illustrative.

Actual partition selection depends on the record key, partitioner, and producer configuration.

For example, records sharing the same customer ID can be assigned to the same partition when the producer uses that ID as the partition key.

This helps preserve ordering for that customer's events.

---

# 13. Kafka Replication

The lecture explains that Kafka brokers replicate data across Availability Zones.

Replication improves durability and availability.

### Architecture

```text
                          AMAZON MSK
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
             AZ A            AZ B            AZ C
              |               |               |
              v               v               v
           BROKER 1        BROKER 2        BROKER 3
              |               |               |
              v               v               v
          PARTITION 0     REPLICA 0       REPLICA 0
           LEADER          FOLLOWER        FOLLOWER
```

### How Replication Works

1. A producer sends a record to a Kafka topic.
2. The record is written to the partition leader.
3. Followers replicate the partition data.
4. Kafka can use another eligible replica if the leader becomes unavailable.

### Important Concept

Kafka replication supports resilience against broker failures.

### Important Clarification

The replication factor and producer acknowledgment settings influence durability.

Simply deploying brokers across multiple Availability Zones does not guarantee that every topic is configured with the desired replication factor.

---

# 14. Apache ZooKeeper

The lecture introduces Apache ZooKeeper as a distributed coordination system historically used by Kafka.

### ZooKeeper Responsibilities

Historically, ZooKeeper helped Kafka coordinate:

- Cluster metadata
- Broker membership
- Controller election
- Distributed cluster operations

### Conceptual Architecture

```text
                       KAFKA CLUSTER
                            |
                            v
                     ZOOKEEPER ENSEMBLE
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          ZOOKEEPER 1   ZOOKEEPER 2   ZOOKEEPER 3
                            |
                            v
                     CLUSTER COORDINATION
```

### Important Concept

ZooKeeper was traditionally used for Kafka cluster coordination.

---

# 15. Understanding KRaft

The lecture also introduces KRaft.

KRaft stands for:

**Kafka Raft Metadata Mode**

KRaft is Kafka's newer metadata management architecture.

It removes Kafka's dependency on ZooKeeper.

### Conceptual Architecture

```text
                       APACHE KAFKA
                            |
                            v
                     KRAFT CONTROLLERS
                            |
                            v
                     CLUSTER METADATA
                            |
                            v
                       KAFKA BROKERS
```

### ZooKeeper vs. KRaft

| Feature | ZooKeeper | KRaft |
|---|---|---|
| Purpose | External coordination system | Kafka-native metadata management |
| Kafka Dependency | Historically required | Replaces ZooKeeper |
| Metadata Management | ZooKeeper-based | Kafka controller quorum |
| Architecture | Separate coordination component | Integrated Kafka metadata architecture |

### Important Clarification

KRaft support depends on the Kafka version and the capabilities available in the selected Amazon MSK deployment.

### AWS Exam Point

For introductory exam preparation, understand that **KRaft is the newer Kafka metadata management approach that replaces ZooKeeper**.

---

# 16. Producers in Amazon MSK

A producer is an application that publishes records to Kafka topics.

### Example Producers

- Web applications
- IoT applications
- Financial transaction systems
- Application logging services
- Database change-data-capture applications

### Architecture

```text
                      DATA SOURCES
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          WEB APP       IoT DEVICE     DATABASE CDC
             |             |             |
             +-------------+-------------+
                           |
                           v
                       PRODUCERS
                           |
                           v
                       AMAZON MSK
                           |
                           v
                       KAFKA TOPIC
```

### Important Concept

Producers publish records to Kafka topics.

### AWS Exam Point

**Producer = Writes events to Kafka.**

---

# 17. Consumers in Amazon MSK

A consumer is an application that reads records from Kafka topics.

### Example Consumers

- Analytics applications
- Data processing services
- Monitoring applications
- Machine learning pipelines
- Storage applications

### Architecture

```text
                       AMAZON MSK
                           |
                           v
                       KAFKA TOPIC
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
          CONSUMER 1    CONSUMER 2    CONSUMER 3
             |             |             |
             v             v             v
           ANALYTICS      STORAGE      MONITORING
```

### Important Concept

Consumers read records from Kafka topics.

### AWS Exam Point

**Consumer = Reads events from Kafka.**

---

# 18. Kafka Consumer Groups

Kafka consumers can be organized into consumer groups.

Consumers in the same group cooperate to process records from a topic.

### Example

A topic contains three partitions.

A consumer group contains three consumers.

```text
                         ORDERS TOPIC
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          PARTITION 0     PARTITION 1     PARTITION 2
              |               |               |
              v               v               v
          CONSUMER A      CONSUMER B      CONSUMER C
              |               |               |
              +---------------+---------------+
                              |
                              v
                       CONSUMER GROUP
```

### Important Concept

Within a traditional Kafka consumer group, each partition is assigned to one consumer at a time.

This allows consumers to divide the processing workload.

### Example

If a topic has three partitions and a consumer group has five consumers, some consumers may remain idle because there are more consumers than partitions.

### AWS Exam Point

Kafka partitions influence consumer parallelism.

---

# 19. End-to-End Amazon MSK Streaming Architecture

The lecture describes a streaming pipeline involving producers, Kafka topics, and consumers.

### Architecture

```text
                         DATA SOURCES
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
         APPLICATIONS      IoT DEVICES      DATABASE EVENTS
             |                |                |
             +----------------+----------------+
                              |
                              v
                         KAFKA PRODUCERS
                              |
                              v
                          AMAZON MSK
                              |
                              v
                          KAFKA TOPICS
                              |
                              v
                           PARTITIONS
                              |
                              v
                            BROKERS
                              |
                              v
                         KAFKA CONSUMERS
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          AMAZON S3       ANALYTICS         ML PIPELINES
```

### Processing Flow

1. Applications generate events.
2. Producers publish events to Kafka topics.
3. Kafka stores records in topic partitions.
4. Brokers replicate records according to configuration.
5. Consumers read the records.
6. Downstream applications process or store the data.

### Important Concept

Amazon MSK supports decoupled, distributed event-streaming architectures.

---

# 20. Amazon MSK Control Plane

The lecture distinguishes between control plane and data plane operations.

The control plane manages the Kafka infrastructure.

### Control Plane Responsibilities

- Create clusters
- Update cluster configurations
- Delete clusters
- Manage supported infrastructure settings
- Configure cluster resources

### Architecture

```text
                       AWS MANAGEMENT
                             |
                             v
                       MSK CONTROL PLANE
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
          CREATE          UPDATE          DELETE
          CLUSTER         CLUSTER         CLUSTER
```

### Important Concept

**Control Plane = Cluster Management Operations**

---

# 21. Amazon MSK Data Plane

The data plane handles the actual streaming workload.

### Data Plane Responsibilities

- Publish records
- Read records
- Store Kafka records
- Serve Kafka topics
- Support streaming applications

### Architecture

```text
                        PRODUCER
                           |
                           v
                       MSK DATA PLANE
                           |
                           v
                        KAFKA TOPIC
                           |
                           v
                         CONSUMER
```

### Control Plane vs. Data Plane

| Feature | Control Plane | Data Plane |
|---|---|---|
| Primary Purpose | Manage cluster infrastructure | Process streaming records |
| Create Cluster | Yes | No |
| Update Cluster | Yes | No |
| Delete Cluster | Yes | No |
| Publish Records | No | Yes |
| Consume Records | No | Yes |

### AWS Exam Point

**Control Plane = Manage Kafka Clusters**

**Data Plane = Produce and Consume Kafka Records**

---

# 22. Amazon MSK Deployment Options

The lecture introduces two deployment options:

1. MSK Provisioned
2. MSK Serverless

Both support Apache Kafka workloads, but differ in how infrastructure capacity is managed.

---

# 23. Amazon MSK Provisioned

MSK Provisioned allows organizations to configure Kafka cluster capacity.

### Characteristics

- Select supported broker configurations
- Configure broker instance types
- Configure broker count
- Configure storage options
- Manage supported scaling settings
- Configure monitoring and logging
- Configure security settings

### Architecture

```text
                        AMAZON MSK
                            |
                            v
                      MSK PROVISIONED
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
           BROKER 1      BROKER 2      BROKER 3
              |             |             |
              v             v             v
          CONFIGURED    CONFIGURED    CONFIGURED
          CAPACITY      CAPACITY      CAPACITY
```

### When to Choose MSK Provisioned

Consider MSK Provisioned when:

- You require more control over broker configurations.
- You need supported Kafka features unavailable in Serverless.
- You have predictable streaming workloads.
- You want to configure broker capacity directly.

### Important Concept

**MSK Provisioned = Customer-configured Kafka cluster capacity with AWS-managed infrastructure operations.**

---

# 24. Amazon MSK Serverless

MSK Serverless allows organizations to run Apache Kafka workloads without managing broker capacity.

AWS automatically provisions and scales supported cluster resources.

### Key Features

- No broker instance selection
- Automatic capacity management
- Automatic scaling
- Reduced infrastructure administration
- Usage-based pricing
- Kafka-compatible streaming applications

### Architecture

```text
                       APPLICATIONS
                            |
                            v
                         PRODUCERS
                            |
                            v
                       MSK SERVERLESS
                            |
                            v
                    AUTOMATIC CAPACITY
                       MANAGEMENT
                            |
                            v
                        KAFKA TOPICS
                            |
                            v
                         CONSUMERS
```

### Important Concept

**MSK Serverless = Run Kafka workloads without managing broker capacity.**

### Important Clarification

Serverless does not eliminate the need to understand Kafka topics, partitions, producers, consumers, and application behavior.

It primarily reduces infrastructure capacity management.

---

# 25. MSK Provisioned vs. MSK Serverless

| Feature | MSK Provisioned | MSK Serverless |
|---|---|---|
| Broker Capacity | Customer-configured | Automatically managed |
| Instance Type Selection | Supported | Not required |
| Storage Configuration | Depends on broker type | Managed |
| Scaling | Configurable | Automatic |
| Infrastructure Control | Greater | Reduced |
| Operational Overhead | Higher | Lower |
| Kafka Compatibility | Yes | Yes, within supported features |
| Typical Use Case | Workloads requiring configuration control | Workloads prioritizing simplified operations |

### AWS Exam Point

**Need control over Kafka broker infrastructure → MSK Provisioned**

**Need Kafka without broker capacity management → MSK Serverless**

---

# 26. Amazon MSK vs. Amazon Kinesis Data Streams

This is one of the most important comparisons in the lecture.

Both services support real-time streaming data.

However, they use different technologies and architectural concepts.

| Feature | Amazon Kinesis Data Streams | Amazon MSK |
|---|---|---|
| Technology | AWS-native streaming service | Managed Apache Kafka |
| Ecosystem | AWS integrations | Apache Kafka ecosystem |
| Streaming Unit | Stream | Topic |
| Parallelism Unit | Shard | Partition |
| Producers | Kinesis producers | Kafka producers |
| Consumers | Kinesis consumers | Kafka consumers |
| Open-Source Kafka Compatibility | No | Yes |
| Capacity Options | Provisioned and on-demand | Provisioned and Serverless |
| Infrastructure Management | AWS-managed | AWS-managed with Kafka-specific configuration |
| Common Migration Scenario | AWS-native streaming adoption | Existing Kafka workloads |
| Typical Use Case | AWS-integrated event streaming | Kafka-based event streaming |

### Important Concept

**Kinesis Data Streams = AWS-native streaming**

**Amazon MSK = Apache Kafka streaming on AWS**

---

# 27. Kinesis Shards vs. Kafka Partitions

The lecture compares Kinesis shards with Kafka partitions.

Both are important for distributing streaming workloads.

### Kinesis Architecture

```text
                     KINESIS DATA STREAM
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           SHARD 1         SHARD 2         SHARD 3
              |               |               |
              v               v               v
          CONSUMERS       CONSUMERS       CONSUMERS
```

### Kafka Architecture

```text
                         KAFKA TOPIC
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          PARTITION 0     PARTITION 1     PARTITION 2
              |               |               |
              v               v               v
           BROKER 1        BROKER 2        BROKER 3
```

### Comparison

| Concept | Kinesis | Kafka |
|---|---|---|
| Logical Data Stream | Stream | Topic |
| Data Distribution | Shards | Partitions |
| Ordering | Within a shard | Within a partition |
| Parallel Processing | Shard-based | Partition-based |

### AWS Exam Memory Point

**Kinesis → Shards**

**Kafka → Topics and Partitions**

---

# 28. Scaling Kinesis vs. Kafka

The lecture highlights differences in scaling.

## Kinesis Data Streams

In provisioned-capacity mode, Kinesis supports:

- Splitting shards
- Merging shards

### Example

```text
                    ORIGINAL SHARD
                          |
                          v
                        SPLIT
                          |
                    +-----+-----+
                    |           |
                    v           v
                 SHARD A     SHARD B
```

Splitting increases shard count.

Merging reduces shard count.

## Apache Kafka

Kafka topics can have partitions added.

However, Kafka does not generally provide a simple native operation to reduce the partition count of an existing topic.

### Example

```text
                      KAFKA TOPIC
                           |
                  +--------+--------+
                  |                 |
                  v                 v
             PARTITION 0       PARTITION 1
                           |
                           v
                    ADD PARTITIONS
                           |
              +------------+------------+
              |            |            |
              v            v            v
         PARTITION 0  PARTITION 1   PARTITION 2
```

### Important Clarification

Adding partitions does not automatically guarantee improved throughput.

Performance depends on broker capacity, partition distribution, consumer parallelism, and workload characteristics.

Also, increasing the number of partitions can affect record-key mapping and ordering behavior for future records.

### AWS Exam Point

**Kinesis → Split and Merge Shards**

**Kafka → Add Partitions; reducing partition count typically requires creating a replacement topic**

---

# 29. Record Size Comparison

The lecture mentions a Kinesis record size limit of 1 MB and contrasts this with configurable Kafka message sizes.

### Important Clarification

The 1 MiB Kinesis limit cited in the lecture reflects an older commonly taught limit.

Current Kinesis Data Streams capabilities may support larger records under applicable service limits and configurations.

For exam preparation, verify the limits relevant to the current exam and AWS documentation rather than relying on a fixed historical number.

Kafka message-size limits are configurable, but practical limits depend on broker settings, producer settings, consumer settings, and available resources.

### Core Exam Concept

The important architectural distinction is not simply record size.

It is that:

- Kinesis is an AWS-native streaming platform.
- MSK supports Apache Kafka workloads.

---

# 30. Real-World Example: Financial Transaction Streaming

A financial institution generates transaction events continuously.

It wants to process these events using Apache Kafka.

### Architecture

```text
                     BANKING APPLICATIONS
                              |
                              v
                       TRANSACTION EVENTS
                              |
                              v
                         KAFKA PRODUCERS
                              |
                              v
                           AMAZON MSK
                              |
                              v
                      TRANSACTIONS TOPIC
                              |
                              v
                         KAFKA CONSUMERS
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          FRAUD ANALYSIS   DATA STORAGE    MONITORING
```

### Benefits

- Continuous event processing
- Decoupled applications
- Distributed streaming
- Kafka compatibility
- Multi-AZ resilience

---

# 31. Real-World Example: Application Monitoring

An organization collects application events from multiple systems.

It wants to analyze events continuously.

### Architecture

```text
                     APPLICATION SERVERS
                              |
                              v
                        EVENT PRODUCERS
                              |
                              v
                           AMAZON MSK
                              |
                              v
                       APPLICATION TOPIC
                              |
                              v
                         KAFKA CONSUMERS
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          ERROR ANALYSIS   EVENT STORAGE   OBSERVABILITY
```

### Example Use Cases

- Application error analysis
- Operational event processing
- Monitoring event pipelines
- Historical event storage
- Streaming analytics

### Important Concept

MSK can provide the event-streaming layer in a larger monitoring or analytics architecture.

---

# 32. Real-World Example: E-Commerce Event Streaming

An e-commerce company wants to process customer orders, payments, and inventory events.

### Architecture

```text
                     E-COMMERCE PLATFORM
                              |
                              v
                         KAFKA PRODUCERS
                              |
                              v
                           AMAZON MSK
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          ORDERS TOPIC   PAYMENTS TOPIC   INVENTORY TOPIC
              |               |               |
              v               v               v
          CONSUMER A      CONSUMER B      CONSUMER C
              |               |               |
              v               v               v
          ORDER SERVICE   PAYMENT SERVICE   STOCK SERVICE
```

### Benefits

- Event-driven processing
- Application decoupling
- Independent consumer applications
- Scalable streaming workloads

---

# 33. Amazon MSK Security Considerations

The lecture mentions enterprise-grade security, monitoring, and logging.

Important areas include:

- VPC networking
- Security groups
- Authentication
- Authorization
- Encryption
- Monitoring
- Logging

### Conceptual Architecture

```text
                        APPLICATIONS
                             |
                             v
                        AMAZON VPC
                             |
                             v
                       SECURITY GROUPS
                             |
                             v
                         AMAZON MSK
                             |
                             v
                    AUTHENTICATED CLIENTS
                             |
                             v
                       KAFKA TOPICS
```

### Important Concept

Although AWS manages the MSK infrastructure, customers still need to configure appropriate network access and supported security controls.

---

# 34. Common Mistakes

### Mistake 1: Confusing MSK with Kinesis

MSK is managed Apache Kafka.

Kinesis Data Streams is AWS-native streaming.

### Mistake 2: Assuming MSK Is a Standalone Data Warehouse

MSK is an event-streaming service.

It is not a data warehouse.

### Mistake 3: Confusing Brokers with Topics

Brokers store and serve records.

Topics organize streams of records.

### Mistake 4: Confusing Topics with Partitions

A topic is a logical event stream.

Partitions divide the topic for distributed processing.

### Mistake 5: Assuming Ordering Is Guaranteed Across an Entire Topic

Kafka ordering is guaranteed within individual partitions.

### Mistake 6: Assuming All Kafka Brokers Must Use ZooKeeper

Kafka supports KRaft-based metadata management in compatible configurations.

### Mistake 7: Assuming MSK Serverless Requires Broker Instance Selection

MSK Serverless manages broker capacity automatically.

### Mistake 8: Assuming Express Brokers Are Always Better

Broker selection depends on workload requirements and supported features.

### Mistake 9: Assuming Kafka Partitions Can Be Reduced Like Kinesis Shards

Kafka generally requires a replacement topic to reduce partition count.

### Mistake 10: Assuming MSK Eliminates All Kafka Application Management

Developers still manage application logic, producers, consumers, and relevant Kafka configurations.

---

# 35. AWS Certification Exam Scenarios

## Scenario 1 – Existing Kafka Migration

A company runs Apache Kafka on premises and wants to migrate its streaming infrastructure to AWS while retaining Kafka compatibility.

**Answer: Amazon MSK**

---

## Scenario 2 – AWS-Native Streaming

A company wants a managed streaming service with tight AWS ecosystem integration and does not require Apache Kafka compatibility.

**Answer: Amazon Kinesis Data Streams**

---

## Scenario 3 – Kafka Broker Management

A company wants AWS to manage much of the infrastructure required to operate Kafka brokers.

**Answer: Amazon MSK**

---

## Scenario 4 – High Availability

A company wants Kafka brokers distributed across multiple Availability Zones.

**Answer: Amazon MSK with an appropriate Multi-AZ cluster configuration**

---

## Scenario 5 – Kafka Data Distribution

A company needs to distribute Kafka topic records across multiple processing units.

**Answer: Kafka Partitions**

---

## Scenario 6 – Kafka Replication

A company wants Kafka records replicated across brokers.

**Answer: Configure an appropriate Kafka replication factor and broker placement.**

---

## Scenario 7 – Automatic Kafka Capacity Management

A company wants to run Kafka without selecting broker instance types or manually managing cluster capacity.

**Answer: Amazon MSK Serverless**

---

## Scenario 8 – Configurable Broker Infrastructure

A company requires supported Kafka broker configurations and direct control over provisioned cluster capacity.

**Answer: Amazon MSK Provisioned**

---

## Scenario 9 – Simplified Storage Management

A company wants a provisioned Kafka broker option with automatically managed storage.

**Answer: Amazon MSK Express Brokers**

---

## Scenario 10 – Kafka Metadata Management

A company wants to understand the newer Kafka metadata architecture that replaces ZooKeeper.

**Answer: KRaft**

---

## Scenario 11 – Kafka Consumer Parallelism

A company wants to increase processing parallelism for a Kafka consumer group.

**Answer: Evaluate topic partitions and consumer-group capacity.**

---

## Scenario 12 – Kinesis Scaling

A company uses provisioned Kinesis Data Streams and wants to increase shard count.

**Answer: Split Shards**

---

# 36. Interview Questions

### Q1. What is Amazon MSK?

Amazon MSK is a fully managed AWS service for running Apache Kafka workloads.

### Q2. What does MSK stand for?

Managed Streaming for Apache Kafka.

### Q3. What is Apache Kafka?

An open-source distributed event-streaming platform.

### Q4. What is a Kafka broker?

A server that stores and serves Kafka records.

### Q5. What is a Kafka topic?

A logical stream of related events.

### Q6. What is a Kafka partition?

A unit of data distribution and parallel processing within a topic.

### Q7. What is a Kafka producer?

An application that publishes records to Kafka topics.

### Q8. What is a Kafka consumer?

An application that reads records from Kafka topics.

### Q9. What is a consumer group?

A collection of consumers that cooperate to process records from assigned topic partitions.

### Q10. What is Kafka replication?

Copying partition data across brokers to improve resilience.

### Q11. Why deploy MSK across multiple Availability Zones?

To improve availability and resilience against infrastructure failures.

### Q12. What are Standard brokers?

Configurable Kafka broker infrastructure available with MSK Provisioned.

### Q13. What are Express brokers?

A managed broker option designed for improved elasticity, throughput, and simplified storage management.

### Q14. What is Apache ZooKeeper?

A distributed coordination system historically used by Kafka.

### Q15. What is KRaft?

Kafka's newer metadata management architecture that replaces ZooKeeper.

### Q16. What is MSK Provisioned?

A deployment option where customers configure supported Kafka cluster capacity.

### Q17. What is MSK Serverless?

A deployment option that automatically provisions and scales Kafka capacity.

### Q18. What is the difference between MSK and Kinesis?

MSK provides managed Apache Kafka, while Kinesis Data Streams is an AWS-native streaming service.

### Q19. What is the Kafka equivalent of a Kinesis shard?

A Kafka partition is the closest conceptual equivalent for distributing and parallelizing stream processing.

### Q20. Can Kafka partitions be increased?

Yes.

### Q21. Can Kafka partitions be reduced directly?

Not generally through a simple native partition-count reduction operation.

### Q22. Does MSK support Multi-AZ deployments?

Yes.

### Q23. Can MSK integrate with downstream AWS services?

Yes, using supported connectors, applications, and integrations.

### Q24. Does MSK Serverless eliminate the need to develop producers and consumers?

No.

### Q25. When should you choose Amazon MSK?

When your application requires Apache Kafka compatibility or managed Kafka streaming infrastructure.

---

# 37. Key Takeaways

1. Amazon MSK stands for Amazon Managed Streaming for Apache Kafka.
2. It is a fully managed Apache Kafka service.
3. MSK supports distributed event-streaming applications.
4. Apache Kafka uses producers and consumers.
5. Producers publish records to topics.
6. Consumers read records from topics.
7. Topics are divided into partitions.
8. Partitions support parallel processing.
9. Kafka brokers store and serve records.
10. Kafka supports replication across brokers.
11. Multi-AZ deployments improve resilience.
12. Standard brokers provide configurable infrastructure.
13. Express brokers simplify storage management.
14. ZooKeeper was historically used for Kafka coordination.
15. KRaft is Kafka's newer metadata management architecture.
16. MSK Provisioned provides greater capacity configuration control.
17. MSK Serverless automatically manages cluster capacity.
18. Kinesis uses streams and shards.
19. Kafka uses topics and partitions.
20. Amazon MSK is particularly relevant for existing Kafka workloads.

---

# 38. AWS Exam Cheat Sheet

| Requirement | AWS Service or Feature |
|---|---|
| Managed Apache Kafka | **Amazon MSK** |
| Existing Kafka Migration | **Amazon MSK** |
| AWS-Native Streaming | **Amazon Kinesis Data Streams** |
| Kafka Event Publisher | **Producer** |
| Kafka Event Reader | **Consumer** |
| Logical Kafka Event Stream | **Topic** |
| Kafka Parallelism | **Partition** |
| Kafka Data Storage Server | **Broker** |
| Kafka Data Resilience | **Replication** |
| Kafka Cluster Coordination | **ZooKeeper / KRaft** |
| Kafka-Native Metadata Management | **KRaft** |
| Configurable Kafka Capacity | **MSK Provisioned** |
| Automatically Managed Kafka Capacity | **MSK Serverless** |
| Configurable Broker Storage | **Standard Brokers** |
| Managed Broker Storage Scaling | **Express Brokers** |
| Kinesis Parallelism | **Shards** |
| Increase Provisioned Kinesis Shard Count | **Split Shards** |
| Reduce Provisioned Kinesis Shard Count | **Merge Shards** |
| Add Kafka Parallelism | **Increase Topic Partitions** |

---

# 39. Final Architecture Summary

```text
                         DATA SOURCES
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
          APPLICATIONS    IoT DEVICES    DATABASE EVENTS
              |               |               |
              +---------------+---------------+
                              |
                              v
                         KAFKA PRODUCERS
                              |
                              v
                           AMAZON MSK
                              |
                 +------------+------------+
                 |                         |
                 v                         v
            MSK PROVISIONED           MSK SERVERLESS
                 |                         |
                 v                         v
          STANDARD / EXPRESS       AUTOMATIC CAPACITY
              BROKERS                 MANAGEMENT
                 |                         |
                 +------------+------------+
                              |
                              v
                          KAFKA TOPICS
                              |
                              v
                           PARTITIONS
                              |
                              v
                       KAFKA CONSUMERS
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
           AMAZON S3       ANALYTICS       APPLICATIONS


                    HIGH AVAILABILITY
                    =================

                         AMAZON VPC
                              |
              +---------------+---------------+
              |               |               |
              v               v               v
             AZ A            AZ B            AZ C
              |               |               |
              v               v               v
           BROKER 1        BROKER 2        BROKER 3
              |               |               |
              +---------------+---------------+
                              |
                              v
                       DATA REPLICATION


                    KAFKA METADATA
                    ==============

                         APACHE KAFKA
                              |
                 +------------+------------+
                 |                         |
                 v                         v
             ZOOKEEPER                   KRAFT
              (LEGACY)               (NEWER MODEL)
```

---

# 40. Related AWS Services

- Amazon Managed Streaming for Apache Kafka (Amazon MSK)
- Amazon MSK Serverless
- Amazon Kinesis Data Streams
- Amazon Data Firehose
- Amazon S3
- Amazon VPC
- Amazon CloudWatch
- AWS IAM
- Amazon EMR
- Amazon Redshift
- Amazon SageMaker

---
