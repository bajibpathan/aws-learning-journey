# 🖥️ Amazon Elastic Compute Cloud (Amazon EC2)

> Amazon EC2 provides scalable virtual compute capacity in AWS, allowing you to deploy and manage virtual servers for application, web, management, database, migration, and other compute workloads.

---

# 📖 Overview

AWS provides multiple compute services, but one of the foundational compute services is:

```text
Amazon Elastic Compute Cloud
            │
            ▼
          Amazon EC2
```

Amazon EC2 allows you to provision:

```text
Virtual Machines
      or
Virtual Servers
```

inside the AWS Cloud.

Instead of purchasing and maintaining physical servers in an on-premises data center, EC2 allows you to provision compute capacity when required.

You can configure:

* CPU
* Memory
* Networking
* Storage
* Security
* Operating system
* Instance size

based on the requirements of your application.

---

# 🎯 Why Amazon EC2?

Traditional infrastructure requires organizations to estimate future compute requirements.

For example:

```text
Purchase Physical Servers
        │
        ▼
Install Hardware
        │
        ▼
Configure Operating System
        │
        ▼
Deploy Application
        │
        ▼
Maintain Hardware
```

The problem is that demand can change.

For example:

```text
Normal Website Traffic
        │
        ▼
Marketing Campaign
        │
        ▼
Traffic Spike
        │
        ▼
More Compute Required
```

With EC2, additional compute capacity can be added when required.

When demand decreases, compute capacity can also be reduced.

Conceptually:

```text
Demand Increases
      │
      ▼
Add EC2 Capacity

Demand Decreases
      │
      ▼
Reduce EC2 Capacity
```

This elasticity is one of the important characteristics of cloud computing.

---

# 🏗️ EC2 in a VPC Architecture

EC2 instances are deployed inside a subnet within a VPC.

A typical application network might contain:

```text
                     VPC
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
 Public Subnet    Private App   Database Subnet
                       Subnet
        │             │             │
        ▼             ▼             ▼
 Bastion /        Web / App      Database
 Management       EC2 Servers    Workloads
 Server
```

Where an EC2 instance is placed depends on its role and connectivity requirements.

---

# 🌐 Public Subnet EC2 Instances

Some EC2 instances may be deployed in public subnets for specific use cases.

Examples mentioned in the course include:

* Bastion hosts
* Management servers

A bastion host can provide an administrative entry point for managing resources in private networks.

Conceptually:

```text
Administrator
     │
     ▼
Internet
     │
     ▼
Bastion Host
Public Subnet
     │
     ▼
Private EC2
```

However, EC2 instances should not automatically be placed in public subnets.

Public placement should be based on an actual requirement.

---

# 🔒 Private Subnet EC2 Instances

Application workloads can be deployed in private subnets.

Examples include:

```text
Web Servers
Application Servers
Internal Services
```

These servers can remain inaccessible directly from the public Internet while still communicating with other resources in the VPC.

Example:

```text
Public Entry Point
       │
       ▼
Private Application EC2
       │
       ▼
Backend Services
```

The network design should determine where each EC2 workload belongs.

---

# 🗄️ EC2 as a Database Server

EC2 can also host database software.

For example:

```text
EC2
 │
 ▼
MySQL
```

or other database software installed directly on the operating system.

However, the course also notes that AWS provides other database services that may be more appropriate depending on the use case.

The key point in this section is:

> EC2 is capable of hosting database workloads, but the compute placement and service choice should depend on application requirements.

---

# 🧠 EC2 as a Virtual Server

An EC2 instance is essentially a virtual server running in AWS.

It is conceptually similar to virtual machines running on technologies such as:

```text
VMware
Hyper-V
```

Instead of running the virtual machine in your own data center, the underlying physical infrastructure is operated by AWS.

```text
AWS Physical Server
        │
        ▼
Virtualization Layer
        │
        ▼
EC2 Instance
```

---

# ⚙️ AWS Nitro System

The course introduces the AWS Nitro System as part of the underlying EC2 architecture.

Conceptually:

```text
AWS Physical Infrastructure
          │
          ▼
     AWS Nitro System
          │
          ▼
     EC2 Instances
```

The Nitro System supports AWS EC2 virtualization and provides the foundation for many modern EC2 instance families.

---

# 📍 EC2 and Availability Zones

An important concept is:

> EC2 instances are Availability Zone specific.

When launching an EC2 instance, you choose a subnet.

A subnet belongs to exactly one Availability Zone.

Therefore:

```text
EC2 Instance
     │
     ▼
Subnet
     │
     ▼
Availability Zone
```

Example:

```text
Region: ca-central-1

          ┌────────────────────┐
          │  ca-central-1a     │
          │                    │
          │  Subnet-A          │
          │      │             │
          │      ▼             │
          │    EC2-A           │
          └────────────────────┘

          ┌────────────────────┐
          │  ca-central-1b     │
          │                    │
          │  Subnet-B          │
          │      │             │
          │      ▼             │
          │    EC2-B           │
          └────────────────────┘
```

EC2-A belongs to the Availability Zone containing Subnet-A.

EC2-B belongs to the Availability Zone containing Subnet-B.

---

# 🏢 Region vs Availability Zone

It is important to distinguish these concepts.

```text
AWS Region
   │
   ├── Availability Zone A
   │       │
   │       └── EC2 Instances
   │
   └── Availability Zone B
           │
           └── EC2 Instances
```

The EC2 service is available within AWS Regions, but an individual EC2 instance runs in a specific Availability Zone.

This becomes important when designing for high availability.

---

# 🏗️ Physical Infrastructure Behind EC2

EC2 instances ultimately run on physical AWS infrastructure.

Conceptually:

```text
AWS Region
   │
   ▼
Availability Zone
   │
   ▼
AWS Data Center Infrastructure
   │
   ▼
Physical Servers
   │
   ├── CPU
   ├── Memory
   ├── Storage Capability
   └── Network Capability
        │
        ▼
Virtual EC2 Instances
```

The physical server provides resources that are used by virtual EC2 instances.

---

# 📊 EC2 Instance Types

EC2 instances are available in different configurations called:

```text
EC2 Instance Types
```

An instance type provides a particular combination of resources.

Typical characteristics include:

| Resource      | Purpose                         |
| ------------- | ------------------------------- |
| CPU           | Processing capability           |
| Memory        | Application memory requirements |
| Network       | Network performance             |
| Storage       | Storage capability              |
| Instance Size | Overall compute capacity        |

The appropriate instance type depends on workload requirements.

---

# 🎯 Choosing an EC2 Instance Type

Instance selection should be driven by application requirements.

Questions include:

```text
How much CPU is required?

How much memory is required?

What network performance is required?

What storage capability is required?

What type of workload will run?
```

Different applications require different resource profiles.

For example:

```text
Compute-Heavy Workload
        │
        ▼
More CPU Capacity

Memory-Heavy Workload
        │
        ▼
More Memory Capacity
```

Choosing the right instance type is therefore an important architecture decision.

---

# 🌐 Elastic Network Interface

EC2 instances require network connectivity.

AWS provides this through an:

```text
Elastic Network Interface
          ENI
```

Conceptually:

```text
EC2 Instance
     │
     ▼
Elastic Network Interface
     │
     ▼
Subnet
     │
     ▼
VPC Network
```

The ENI provides network connectivity to the EC2 instance.

---

# 🔒 Security Groups and EC2

Security Groups are associated with the network interfaces used by EC2 instances.

Conceptually:

```text
Network Traffic
      │
      ▼
Security Group
      │
      ▼
Elastic Network Interface
      │
      ▼
EC2 Instance
```

The Security Group determines which network traffic is permitted.

Security Groups can control:

```text
Inbound Traffic

and

Outbound Traffic
```

---

# 🧠 Relationship Between EC2, ENI and Security Group

A useful mental model is:

```text
EC2 Instance
     │
     ▼
ENI
     │
     ▼
Security Group Rules
     │
     ▼
VPC Network
```

More precisely, Security Groups are applied to network interfaces and control permitted network traffic associated with those interfaces.

---

# 🌐 Multiple Network Interfaces

The course also introduces an advanced concept.

An EC2 instance can have multiple network interfaces depending on its configuration and supported limits.

Conceptually:

```text
               EC2 Instance
                 /      \
                /        \
               ▼          ▼
            ENI-1        ENI-2
              │            │
              ▼            ▼
          Subnet-A      Subnet-B
```

For network interfaces attached to the same EC2 instance, the relevant subnets must be within the same Availability Zone as the instance.

This type of design can be used for advanced networking scenarios.

---

# 🧠 Multi-Homed EC2 Instances

An EC2 instance with multiple network interfaces can be described as a:

```text
Multi-Homed Server
```

Example:

```text
Network A
   │
   ▼
 ENI-A
   │
   ▼
 EC2
   │
   ▲
 ENI-B
   │
Network B
```

This can support specialized network architectures.

For introductory EC2 learning, however, most workloads typically begin with a simpler single-interface design.

---

# 💾 EC2 Storage

Like other virtual servers, EC2 instances require storage.

The course introduces:

```text
Amazon Elastic Block Store
           EBS
```

EBS provides block storage volumes that can be attached to EC2 instances.

Example:

```text
EC2 Instance
     │
     ▼
EBS Volume
```

The storage can contain:

* Operating system
* Applications
* Application files
* Other persistent data

---

# 📍 EBS and Availability Zones

An important relationship is:

> An EBS volume and the EC2 instance to which it is attached must be in the same Availability Zone.

Example:

```text
Availability Zone A

EC2-A
  │
  ▼
EBS-A
```

Valid:

```text
EC2: ca-central-1a
EBS: ca-central-1a
```

Not valid:

```text
EC2: ca-central-1a
EBS: ca-central-1b
```

An EBS volume cannot be directly attached to an EC2 instance located in another Availability Zone.

---

# 🧠 Why EBS Is AZ Specific

EC2 instances require low-latency access to block storage.

Therefore the EC2 instance and attached EBS volumes operate within the same Availability Zone.

Conceptually:

```text
Availability Zone
      │
      ├── EC2
      │    │
      │    ▼
      │   EBS
      │
      └── Low-Latency Connectivity
```

---

# 🖥️ Operating Systems

EC2 supports different operating systems.

The course mentions:

```text
Linux

Windows

macOS
```

The operating system selected depends on application requirements.

---

# 🏗️ Common EC2 Use Cases

EC2 can support many workloads.

Examples introduced in this lesson include:

| Use Case           | Example                              |
| ------------------ | ------------------------------------ |
| Web Server         | Host websites                        |
| Application Server | Run application workloads            |
| Database Server    | Run database software                |
| Bastion Host       | Administrative access                |
| Management Server  | Manage internal systems              |
| Migration Target   | Move existing workloads into AWS     |
| Compute Processing | Temporary heavy processing workloads |

---

# 🚚 EC2 as a Migration Target

Organizations may already operate applications on:

```text
Physical Servers

VMware Virtual Machines

Hyper-V Virtual Machines
```

EC2 can be used as a target environment when moving those workloads into AWS.

Conceptually:

```text
On-Premises Server
        │
        ▼
Migration
        │
        ▼
Amazon EC2
```

This allows traditional server-based applications to run on AWS compute infrastructure.

---

# 📈 Scaling EC2

One of the major benefits of cloud compute is the ability to change capacity as demand changes.

Conceptually:

```text
Normal Demand
     │
     ▼
2 EC2 Instances

Traffic Increase
     │
     ▼
More EC2 Instances

Traffic Decrease
     │
     ▼
Reduce EC2 Instances
```

The course introduces this idea as part of the elasticity provided by AWS compute services.

Later EC2 topics can build on this with services and features such as scaling mechanisms.

---

# 🏗️ Example Multi-Tier Architecture

EC2 instances can be distributed according to application tiers.

```text
                       Internet
                           │
                           ▼
                    Public Subnet
                           │
                     Management
                       Server
                           │
                           ▼
                 Private Web/App Tier
                    ┌──────┴──────┐
                    │             │
                  EC2-A         EC2-B
                    │             │
                    └──────┬──────┘
                           │
                           ▼
                    Database Tier
                           │
                      Database
```

The exact architecture depends on the application requirements.

---

# 🌐 EC2 Networking Relationships

The core relationships can be represented as:

```text
Region
   │
   ▼
VPC
   │
   ▼
Subnet
   │
   ▼
Availability Zone
   │
   ▼
EC2 Instance
   │
   ├── ENI
   │     │
   │     └── Security Group
   │
   └── EBS Volume
```

This is one of the most important mental models from this lesson.

---

# 📊 Component Summary

| Component         | Relationship to EC2                              |
| ----------------- | ------------------------------------------------ |
| Region            | Contains Availability Zones where EC2 can run    |
| Availability Zone | Individual EC2 instance runs in one AZ           |
| VPC               | Provides network boundary                        |
| Subnet            | Determines EC2 network placement and AZ          |
| ENI               | Provides network connectivity                    |
| Security Group    | Controls permitted ENI traffic                   |
| EBS               | Provides block storage                           |
| Instance Type     | Defines compute characteristics                  |
| Operating System  | Software environment running on the EC2 instance |

---

# 🔑 Important Architecture Relationships

Remember:

```text
EC2 Instance
      │
      └── Availability Zone Specific
```

```text
Subnet
      │
      └── Availability Zone Specific
```

```text
ENI
      │
      └── Same AZ as EC2
```

```text
EBS Volume
      │
      └── Same AZ as EC2 when attached
```

These relationships are important for both AWS architecture and troubleshooting.

---

# ✅ Best Practices

Based on the concepts introduced in this section:

* Choose EC2 placement according to workload requirements.
* Avoid placing servers in public subnets unless there is a clear requirement.
* Use Security Groups to control access to EC2 network interfaces.
* Select instance types based on CPU, memory, network, and storage requirements.
* Distribute application architecture across Availability Zones when high availability is required.
* Keep EC2 and attached EBS volumes in the same Availability Zone.
* Design VPC and subnet architecture before deploying application workloads.
* Use appropriate private network placement for internal application workloads.
* Avoid deploying infrastructure simply because EC2 makes it easy to create servers. Start with application requirements.

---

# ❓ Interview Questions

### Q1. What is Amazon EC2?

**Answer**

Amazon EC2 is an AWS compute service that provides scalable virtual server capacity in the cloud.

---

### Q2. Is an EC2 instance Region-specific or Availability Zone-specific?

**Answer**

An individual EC2 instance runs in a specific Availability Zone.

The subnet selected during deployment determines the Availability Zone.

---

### Q3. Why does an EC2 instance need a subnet?

**Answer**

The subnet determines where the EC2 instance is placed within the VPC network and determines its Availability Zone.

---

### Q4. What is an EC2 instance type?

**Answer**

An EC2 instance type defines a particular combination of compute resources such as CPU, memory, networking, and storage-related capabilities.

---

### Q5. What provides network connectivity to an EC2 instance?

**Answer**

An Elastic Network Interface provides network connectivity to the EC2 instance.

---

### Q6. Where are Security Groups applied?

**Answer**

Security Groups are associated with network interfaces and control permitted inbound and outbound traffic.

---

### Q7. Can an EC2 instance have multiple network interfaces?

**Answer**

Yes, depending on the instance configuration and supported limits.

Multiple ENIs can support more advanced networking architectures.

---

### Q8. What is Amazon EBS?

**Answer**

Amazon Elastic Block Store provides block storage volumes that can be attached to EC2 instances.

---

### Q9. Can an EBS volume in one Availability Zone be attached directly to an EC2 instance in another Availability Zone?

**Answer**

No.

The attached EBS volume and EC2 instance must be in the same Availability Zone.

---

### Q10. Why might an EC2 instance be placed in a public subnet?

**Answer**

A public subnet may be appropriate when the workload has a specific requirement for public connectivity, such as certain bastion or management-server designs.

---

### Q11. Why would application servers usually be placed in private subnets?

**Answer**

Private subnet placement can prevent application servers from being directly exposed to the Internet while still allowing required internal communication.

---

### Q12. Can EC2 run database software?

**Answer**

Yes.

Database software such as MySQL can run on EC2, although AWS also provides other database service options that may be evaluated separately.

---

### Q13. What is the relationship between a subnet and an Availability Zone?

**Answer**

A subnet belongs to a single Availability Zone.

Because an EC2 instance is launched into a subnet, the selected subnet determines the instance's Availability Zone.

---

### Q14. Why is choosing the correct EC2 instance type important?

**Answer**

The instance type determines the compute resources available to the application, including CPU, memory, networking, and other capabilities.

The choice should match workload requirements.

---

# 💡 Key Takeaways

* Amazon EC2 provides virtual server capacity in AWS.
* EC2 capacity can be increased or decreased according to workload demand.
* EC2 instances are deployed inside VPC subnets.
* An individual EC2 instance is Availability Zone specific.
* The subnet determines the EC2 instance's Availability Zone.
* Instance types provide different combinations of compute resources.
* ENIs provide EC2 network connectivity.
* Security Groups control permitted traffic associated with network interfaces.
* EC2 instances can support multiple network interfaces for advanced designs.
* EBS provides block storage for EC2.
* Attached EBS volumes must be in the same Availability Zone as the EC2 instance.
* EC2 can run Linux, Windows, and macOS workloads.
* EC2 can host web, application, management, database, and migration workloads.
* EC2 placement should be based on application and network requirements.
* High availability requires architecture across multiple Availability Zones rather than relying on a single EC2 instance.

---

# 📚 Related Topics

The next EC2 topics should build on this foundation:

```text
01 - Amazon EC2 Overview
02 - Amazon Machine Images (AMI)
03 - EC2 Instance Types
04 - EC2 Instance Lifecycle
05 - EC2 Storage and EBS
06 - EC2 Networking and ENIs
07 - EC2 Security
08 - EC2 User Data and Metadata
09 - EC2 Pricing Models
10 - EC2 Auto Scaling
11 - Elastic Load Balancing
```

---

# 📖 Source

This note is organized from the EC2 overview lesson provided in the course material.

The lesson introduces:

* EC2 as AWS virtual compute
* EC2 placement inside VPC subnets
* Availability Zone dependency
* EC2 instance types
* AWS Nitro
* Elastic Network Interfaces
* Security Groups
* EBS storage
* Common EC2 use cases
* EC2 as a migration target

> The main goal at this stage is to understand where an EC2 instance sits in the AWS architecture and what supporting networking, security, and storage components it depends on.
