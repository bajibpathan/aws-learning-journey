# Amazon Elastic Container Service (Amazon ECS)

> Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that helps deploy, manage, and scale containerized applications on AWS.

---

## 📖 Overview

In the previous lessons, we learned about containers, Dockerfiles, Docker images, and container layers.

Once a container image has been built, we need a platform to deploy and manage containers.

This is where **Amazon Elastic Container Service (Amazon ECS)** comes in.

Amazon ECS enables organizations to run containerized applications on AWS without having to build their own container orchestration platform.

ECS supports two primary launch types discussed in this lesson:

- **EC2 launch type:** Run containers on EC2 instances that you manage.
- **AWS Fargate launch type:** Run containers without managing the underlying EC2 infrastructure.

Both approaches use common ECS components, but they differ in how the infrastructure is managed.

---

## 1. How Amazon ECS Works

Before deploying containers with ECS, we need to understand the main components.

The basic workflow is:

```text
Dockerfile
    │
    ▼
Build Container Image
    │
    ▼
Container Registry
(Docker Hub / Amazon ECR)
    │
    ▼
ECS Task Definition
    │
    ▼
ECS Cluster
    │
    ▼
ECS Task / Service
    │
    ▼
Running Containers
```

Each component plays a different role in deploying and managing the application.

---

## 2. Container Image Repositories

Once a Docker image has been built, it needs to be stored somewhere so that ECS can retrieve it.

The lesson introduces two commonly used container registries.

### Docker Hub

Docker Hub is a container registry maintained by Docker.

It allows developers and organizations to store and share container images.

Docker Hub supports:

- Public repositories
- Private repositories
- Sharing images with teams
- Distributing container images to deployment environments

### Amazon Elastic Container Registry (Amazon ECR)

Amazon ECR is an AWS-managed container registry.

Like Docker Hub, ECR stores container images that can later be used to launch containers.

The lesson highlights its integration with AWS services, including:

- AWS Identity and Access Management (IAM)
- AWS Key Management Service (KMS)
- Amazon ECS

### Docker Hub vs Amazon ECR

| Feature | Docker Hub | Amazon ECR |
|---|---|---|
| Provider | Docker | AWS |
| Purpose | Store container images | Store container images |
| Public repositories | Supported | Supported |
| Private repositories | Supported | Supported |
| AWS IAM integration | Not native in the same way | Native |
| AWS KMS integration | Not native in the same way | Supported |

Both can be used as image sources for ECS applications.

---

## 3. ECS Task Definition

A **task definition** is the blueprint that describes how ECS should run an application.

It can be represented in JSON format and includes the configuration for one or more containers.

A task definition can specify:

- Container image
- Container definitions
- IAM roles
- Networking mode
- CPU allocation
- Memory allocation
- Storage volumes
- Container ports

### Example structure

```text
Task Definition
      │
      ├── Container Image
      ├── Container Definitions
      ├── IAM Roles
      ├── Networking Mode
      ├── CPU
      ├── Memory
      ├── Storage
      └── Port Configuration
```

For example, a task definition may specify an application image stored in Amazon ECR and the CPU, memory, and port settings required to run it.

### Container definitions

A task definition can contain one or more container definitions.

Each container definition describes an individual container that forms part of the task.

```text
Task Definition
       │
       ├── Container Definition A
       │
       └── Container Definition B
```

This allows multiple related containers to be configured together as one task.

---

## 4. What Is an ECS Task?

An **ECS task** is a running instance of a task definition.

The task definition describes what should run.

The task represents that definition being executed.

```text
Task Definition
       │
       ▼
    Run Task
       │
       ▼
    ECS Task
       │
       ├── Container A
       └── Container B
```

An important distinction is that a task can contain one or more containers.

For example, if a task definition contains two container definitions, running that task can launch both containers together.

---

## 5. What Is an ECS Cluster?

An ECS cluster is a logical grouping of ECS tasks and services.

It provides the organizational boundary within which ECS workloads are managed.

```text
ECS Cluster
     │
     ├── ECS Service A
     │       ├── Task 1
     │       └── Task 2
     │
     └── ECS Service B
             ├── Task 1
             └── Task 2
```

The cluster contains the tasks and services that make up the containerized application environment.

The underlying compute infrastructure depends on whether the workloads use EC2 or Fargate.

---

## 6. What Is an ECS Service?

An ECS service helps maintain and manage a specified number of running tasks.

The lesson compares this with the behavior of EC2 Auto Scaling Groups.

For example:

```text
ECS Service
Desired Tasks = 2
       │
       ├── Task 1
       └── Task 2
```

If one task fails, the ECS service can launch a replacement to maintain the required number of tasks.

```text
Task 1 Fails
      │
      ▼
ECS Service Detects Failure
      │
      ▼
Launch Replacement Task
      │
      ▼
Desired Task Count Restored
```

### ECS Service Auto Scaling

ECS services can also use scaling policies to increase or decrease the number of tasks.

The lesson discusses:

- Target tracking scaling
- Step scaling

Scaling can use metrics such as:

- CPU utilization
- Memory utilization
- Application Load Balancer request count per target

### Example

```text
Normal Demand
     │
     ▼
2 ECS Tasks
     │
     │ Demand Increases
     ▼
4 ECS Tasks
     │
     │ Demand Decreases
     ▼
2 ECS Tasks
```

This allows container capacity to adjust as application demand changes.

---

## 7. ECS Task Placement Strategies

For ECS workloads running on EC2 instances, task placement strategies help determine where tasks should be deployed.

The lesson introduces three strategies.

| Strategy | Purpose |
|---|---|
| Binpack | Place tasks to reduce resource fragmentation and potentially use fewer instances |
| Spread | Distribute tasks across infrastructure, such as Availability Zones |
| Random | Place tasks using random selection among eligible instances |

### Binpack

```text
EC2 Instance A
├── Task 1
├── Task 2
└── Task 3

EC2 Instance B
└── Available Capacity
```

Binpack aims to use available instance capacity efficiently.

### Spread

```text
Availability Zone A
├── Task 1
└── Task 3

Availability Zone B
├── Task 2
└── Task 4
```

Spread distributes tasks across selected infrastructure attributes.

The lesson introduces these strategies at a high level without going into detailed placement configuration.

---

## 8. ECS Anywhere and AWS Copilot

The lesson also introduces two additional capabilities.

### ECS Anywhere

ECS Anywhere allows ECS workloads to be managed on external infrastructure, including on-premises servers.

The lesson highlights that this requires:

- Connectivity between the external infrastructure and AWS
- Required access and permissions
- ECS agents installed on the external instances

### AWS Copilot

AWS Copilot is a command-line tool designed to simplify building, releasing, and operating containerized applications on ECS.

It helps developers work with ECS applications from their local development environment.

---

# 9. ECS Launch Types

The lesson focuses on two ECS launch types:

1. Amazon EC2
2. AWS Fargate

The main difference is **who manages the underlying compute infrastructure**.

---

## 10. ECS EC2 Launch Type

With the EC2 launch type, containers run on EC2 instances associated with an ECS cluster.

You are responsible for managing those EC2 instances.

This includes activities such as:

- Selecting EC2 instance types
- Configuring the instances
- Applying operating system updates
- Patching the instances
- Managing compute capacity

### Architecture

```text
Container Registry
(Docker Hub / ECR)
         │
         ▼
   Task Definition
         │
         ▼
      ECS Service
         │
         ▼
      ECS Cluster
         │
    ┌────┴────┐
    ▼         ▼
   AZ-A      AZ-B
    │         │
    ▼         ▼
 EC2 Host  EC2 Host
    │         │
    ▼         ▼
 ECS Agent  ECS Agent
    │         │
    ▼         ▼
Containers Containers
```

The ECS agent on the EC2 instances allows ECS to manage tasks running on those hosts.

---

## 11. Networking for ECS on EC2

The lesson describes creating a VPC with subnets across multiple Availability Zones.

A typical setup includes:

- VPC
- Public subnets
- Private subnets
- Multiple Availability Zones
- ECS cluster
- EC2 instances running the ECS agent

The objective is to provide an appropriate network environment for running containerized applications.

```text
VPC
 │
 ├── Availability Zone A
 │      ├── Public Subnet
 │      └── Private Subnet
 │             └── EC2 + Containers
 │
 └── Availability Zone B
        ├── Public Subnet
        └── Private Subnet
               └── EC2 + Containers
```

Deploying workloads across Availability Zones helps support high availability.

---

## 12. Scaling ECS on EC2

With the EC2 launch type, there are two different capacity considerations.

### EC2 instance capacity

An EC2 Auto Scaling Group can manage the number of EC2 instances available to host containers.

### ECS task capacity

An ECS service can manage the number of application tasks running on those instances.

```text
                  ECS Cluster
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
    EC2 Auto Scaling         ECS Service Scaling
           │                       │
           ▼                       ▼
    Scale EC2 Hosts           Scale ECS Tasks
```

For example:

- EC2 Auto Scaling increases the number of available container hosts.
- ECS Service Auto Scaling increases the number of running application tasks.

The lesson also introduces Auto Scaling Group capacity providers as a way of managing EC2 capacity for ECS workloads.

**Key distinction:** EC2 capacity scaling and ECS task scaling are related, but they are not the same operation.

---

## 13. When to Use ECS EC2 Launch Type

The lesson identifies the EC2 launch type as suitable for large, consistent workloads.

Reasons include:

- Control over the underlying EC2 instances
- Ability to choose instance types
- Support for EC2 purchasing options
- Opportunity to use Spot Instances
- Flexibility in managing compute capacity

However, the organization remains responsible for managing the underlying instances.

---

# 14. AWS Fargate Launch Type

AWS Fargate provides a different approach.

With Fargate, AWS manages the underlying compute infrastructure used to run containers.

You do not need to provision and maintain EC2 instances for your ECS tasks.

The application still uses:

- Container images
- Container registries
- Task definitions
- ECS clusters
- ECS services
- VPC networking

But the underlying hosts are managed by AWS.

### Architecture

```text
Container Registry
(Docker Hub / ECR)
         │
         ▼
   Task Definition
         │
         ▼
      ECS Service
         │
         ▼
      ECS Cluster
         │
         ▼
     AWS Fargate
         │
    ┌────┴────┐
    ▼         ▼
  Task 1    Task 2
    │         │
    ▼         ▼
Containers Containers
```

The application runs without requiring you to manage the underlying EC2 hosts.

---

## 15. How Fargate Works

The lesson describes Fargate as running tasks on AWS-managed infrastructure.

The process is:

1. Build the container image.
2. Store the image in a container registry.
3. Create the task definition.
4. Create an ECS cluster.
5. Configure an ECS service.
6. Launch tasks using Fargate.
7. Access the application through the configured network.

```text
Docker Image
     │
     ▼
Amazon ECR
     │
     ▼
Task Definition
     │
     ▼
ECS Service
     │
     ▼
Fargate Tasks
     │
     ▼
Running Application
```

AWS manages the underlying compute infrastructure.

The lesson emphasizes that Fargate tasks are isolated from other customers' tasks.

---

## 16. Fargate Networking and Load Balancing

Fargate tasks can communicate using networking associated with your VPC.

The lesson explains that applications running in Fargate can be accessed through private IP addresses.

An Application Load Balancer can distribute traffic to the tasks.

### Architecture

```text
                 Internet
                    │
                    ▼
           Application Load Balancer
                    │
                    ▼
                Target Group
               Target Type: IP
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Fargate Task 1      Fargate Task 2
          │                   │
          ▼                   ▼
       Container           Container
```

For the Fargate configuration discussed in the lesson, the Application Load Balancer target group uses **IP addresses** rather than EC2 instance targets.

---

## 17. Benefits of AWS Fargate

### No EC2 instance management

AWS handles the underlying compute infrastructure.

You do not need to manage the EC2 hosts running your containers.

### Reduced administrative overhead

AWS handles host-level responsibilities such as infrastructure maintenance and patching.

### Task isolation

Fargate tasks run in isolated environments.

### Integration with ECS services

Fargate tasks can be deployed and managed using ECS services.

### Usage-based compute pricing

The lesson emphasizes that Fargate charges are associated with the container resources used rather than billing you for separately managed EC2 container hosts.

---

## 18. EC2 vs Fargate Launch Types

| Feature | EC2 Launch Type | Fargate Launch Type |
|---|---|---|
| Underlying infrastructure | Customer-managed EC2 | AWS-managed |
| EC2 provisioning | Required | Not required |
| Host OS maintenance | Customer responsibility | AWS responsibility |
| Container orchestration | ECS | ECS |
| Task definitions | Required | Required |
| ECS services | Supported | Supported |
| Scaling tasks | Supported | Supported |
| Scaling hosts | EC2 Auto Scaling | Managed by AWS |
| Compute control | Greater | Less |
| EC2 Spot purchasing | Supported | Not applicable to customer-managed hosts |
| Administrative overhead | Higher | Lower |
| Lesson's suggested fit | Large, consistent workloads | Smaller workloads and simpler operations |

The lesson presents Fargate as a simpler option when you do not want to manage EC2 infrastructure.

---

## 19. Choosing Between EC2 and Fargate

The primary decision is whether you want to manage the underlying compute infrastructure.

### Choose EC2 when:

- You need greater control over the EC2 hosts.
- You are comfortable managing instance patching and maintenance.
- You want to manage EC2 capacity and purchasing options.
- Your workload is large and relatively consistent.

### Choose Fargate when:

- You want AWS to manage the underlying compute infrastructure.
- You want to reduce host administration.
- You prefer focusing on containers and application deployment.
- You want to run ECS tasks without provisioning EC2 hosts.

The appropriate choice depends on workload characteristics, operational requirements, and cost considerations.

---

## ⚠️ Common Mistakes

### 1. Confusing a task definition with a task

A task definition is the configuration blueprint.

A task is a running instance of that definition.

### 2. Assuming one ECS task always contains one container

An ECS task can contain one or more containers.

### 3. Confusing an ECS cluster with an ECS service

A cluster logically groups tasks and services.

A service maintains and manages running tasks.

### 4. Assuming ECS always manages EC2 instances

With the EC2 launch type, you manage the underlying EC2 infrastructure.

With Fargate, AWS manages the underlying compute infrastructure.

### 5. Confusing EC2 scaling with task scaling

EC2 Auto Scaling adjusts the number of container hosts.

ECS Service Auto Scaling adjusts the number of application tasks.

### 6. Forgetting the container registry

ECS requires access to the container images specified in the task definition.

---

## ❓ Interview Questions

### Q1. What is Amazon ECS?

Amazon ECS is a managed container orchestration service used to deploy, manage, and scale containerized applications on AWS.

### Q2. What is Amazon ECR?

Amazon ECR is an AWS-managed container registry used to store container images.

### Q3. What is an ECS task definition?

A task definition describes how ECS should run one or more containers, including image, CPU, memory, networking, and other configuration.

### Q4. What is an ECS task?

A task is a running instance of a task definition.

### Q5. Can an ECS task contain multiple containers?

Yes. A task definition can describe multiple containers that run together as one task.

### Q6. What is an ECS cluster?

A logical grouping of ECS tasks and services.

### Q7. What is an ECS service?

An ECS service manages running tasks and helps maintain the required number of tasks.

### Q8. What happens if an ECS service task fails?

The service can launch a replacement task to maintain its desired task count.

### Q9. What are the two ECS launch types discussed in this lesson?

EC2 and AWS Fargate.

### Q10. What is the main difference between EC2 and Fargate?

With EC2, you manage the underlying instances. With Fargate, AWS manages the underlying compute infrastructure.

### Q11. What is an ECS capacity provider?

The lesson introduces Auto Scaling Group capacity providers as a mechanism for managing EC2 capacity used by ECS workloads.

### Q12. What is the difference between ECS Service Auto Scaling and EC2 Auto Scaling?

ECS Service Auto Scaling changes the number of tasks. EC2 Auto Scaling changes the number of EC2 instances.

### Q13. What metrics can be used for ECS Service Auto Scaling?

The lesson discusses CPU utilization, memory utilization, and Application Load Balancer request count per target.

### Q14. What is the purpose of task placement strategies?

They determine how ECS tasks are distributed across available EC2 infrastructure.

### Q15. What are binpack, spread, and random placement strategies?

Binpack attempts to consolidate tasks, spread distributes tasks across infrastructure, and random selects eligible placement locations randomly.

### Q16. Why does Fargate use IP targets with an Application Load Balancer?

The Fargate configuration described in the lesson uses task networking with private IP addresses, so the load balancer targets the tasks by IP.

### Q17. What is ECS Anywhere?

A capability that allows ECS workloads to be managed on external infrastructure, including on-premises environments.

### Q18. What is AWS Copilot?

A command-line tool that simplifies building, releasing, and operating containerized applications on ECS.

---

## 💡 Key Takeaways

- Amazon ECS is a managed container orchestration service.
- Docker images are built from Dockerfiles and stored in container registries.
- Docker Hub and Amazon ECR are common container registry options.
- Task definitions describe how containers should run.
- ECS tasks are running instances of task definitions.
- ECS clusters logically group tasks and services.
- ECS services maintain and scale running tasks.
- ECS can replace failed tasks to maintain the required task count.
- ECS supports task placement strategies such as binpack, spread, and random for EC2 workloads.
- The EC2 launch type runs containers on customer-managed EC2 instances.
- The Fargate launch type runs containers on AWS-managed infrastructure.
- EC2 Auto Scaling manages container-host capacity.
- ECS Service Auto Scaling manages application-task capacity.
- Fargate reduces the operational overhead associated with managing EC2 hosts.
- Application Load Balancers can distribute traffic to Fargate tasks using IP targets.

### The most important concept to remember

```text
                  Amazon ECS
                      │
             ┌────────┴────────┐
             ▼                 ▼
        EC2 Launch          Fargate
             │                 │
             ▼                 ▼
      Manage EC2 Hosts    AWS Manages Hosts
             │                 │
             └────────┬────────┘
                      │
                      ▼
               ECS Task Definition
                      │
                      ▼
                 ECS Service
                      │
                      ▼
                 Running Tasks
                      │
                      ▼
                  Containers
```

---

## 📚 Related Topics

- Docker Images and Container Layers
- Amazon Elastic Container Registry (ECR)
- ECS Task Definitions
- ECS Tasks and Services
- ECS Clusters
- ECS Capacity Providers
- ECS Service Auto Scaling
- AWS Fargate
- Application Load Balancers
- ECS Anywhere
- AWS Copilot

**Next topic:** Deploying a Multi-Tier Application with Amazon ECS