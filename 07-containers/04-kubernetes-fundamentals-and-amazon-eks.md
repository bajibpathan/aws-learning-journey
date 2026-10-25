# Kubernetes Fundamentals and Amazon Elastic Kubernetes Service (EKS)

> Kubernetes is an open-source container orchestration platform. Amazon Elastic Kubernetes Service (EKS) allows organizations to run Kubernetes workloads on AWS while AWS manages the Kubernetes control plane.

---

## 📖 Overview

In previous lessons, we covered:

- Containerization and Docker
- Docker images and container layers
- Amazon Elastic Container Service (ECS)
- Deploying containers using AWS Fargate

AWS provides two container orchestration solutions discussed in this course:

| Service | Description |
|---|---|
| Amazon ECS | AWS-native container orchestration service |
| Amazon EKS | Managed Kubernetes service on AWS |

Amazon EKS allows organizations to deploy Kubernetes-based applications on AWS without managing the Kubernetes control plane infrastructure.

This lesson covers Kubernetes architecture and the different Amazon EKS deployment options.

---

# Part 1: Kubernetes Fundamentals

## 1. What Is Kubernetes?

Kubernetes is an open-source container orchestration platform used to deploy, manage, and scale containerized applications.

A Kubernetes cluster consists of two major components:

1. **Control Plane:** Manages the Kubernetes cluster.
2. **Worker Nodes:** Run containerized applications inside Pods.

```text
                 Kubernetes Cluster
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Control Plane             Worker Nodes
             |                       |
             |                 +-----+-----+
             |                 |           |
             v                 v           v
       Cluster Management    Node 1      Node 2
                               |           |
                               v           v
                              Pods        Pods
                               |           |
                               v           v
                           Containers  Containers
```

The control plane manages cluster operations, while worker nodes execute application workloads.

---

## 2. Kubernetes Control Plane Components

The control plane contains several components that work together to manage the cluster.

| Component | Responsibility |
|---|---|
| Kube API Server | Exposes the Kubernetes API |
| etcd | Stores cluster configuration and state |
| Kube Scheduler | Assigns Pods to worker nodes |
| Kube Controller Manager | Runs controller processes |
| Cloud Controller Manager | Integrates Kubernetes with cloud provider APIs |

### Kube API Server

The `kube-apiserver` acts as the front end of the Kubernetes control plane.

Administrators and Kubernetes components communicate with the cluster through the API server.

```text
Administrator
      |
      v
Kubernetes API Server
      |
      v
Kubernetes Control Plane
```

### etcd

`etcd` is a key-value database that stores Kubernetes cluster data.

It acts as the backing store for the cluster's configuration and state.

### Kube Scheduler

The `kube-scheduler` determines which worker node should run a Pod.

Scheduling decisions can consider:

- CPU and memory requirements
- Hardware requirements
- Affinity and anti-affinity rules
- Policy constraints
- Data locality
- Workload requirements

### Kube Controller Manager

The `kube-controller-manager` runs controllers that manage Kubernetes resources.

Examples include node controllers and job controllers.

### Cloud Controller Manager

The Cloud Controller Manager provides integration with cloud provider APIs, such as AWS, Azure, and Google Cloud.

---

## 3. Kubernetes Worker Nodes

Worker nodes are physical or virtual machines that run application containers.

Each worker node typically contains:

- Kubelet
- Container runtime
- Kube-Proxy
- Application Pods

```text
┌─────────────────────────────────┐
│           Worker Node           │
│                                 │
│  Kubelet                        │
│  Container Runtime              │
│  Kube-Proxy                     │
│                                 │
│  ┌───────────┐ ┌───────────┐    │
│  │   Pod 1   │ │   Pod 2   │    │
│  │ Container │ │ Container │    │
│  └───────────┘ └───────────┘    │
└─────────────────────────────────┘
```

### Kubelet

The Kubelet is the agent that communicates with the Kubernetes API server and helps ensure that the assigned Pods are running.

### Container Runtime

The container runtime manages the execution and lifecycle of containers.

### Kube-Proxy

Kube-Proxy maintains networking rules that support communication with Pods.

---

## 4. Kubernetes Pods

A Pod is the basic unit used to run containers in Kubernetes.

A Pod can contain one or more containers.

```text
┌─────────────────────────────┐
│             Pod             │
│                             │
│  ┌──────────┐ ┌──────────┐  │
│  │Container │ │Container │  │
│  │    A     │ │    B     │  │
│  └──────────┘ └──────────┘  │
└─────────────────────────────┘
```

The course compares Kubernetes Pods with Amazon ECS tasks.

| Amazon ECS | Kubernetes |
|---|---|
| ECS Cluster | Kubernetes Cluster |
| ECS Task | Kubernetes Pod |
| ECS Task Definition | Pod specification (conceptual comparison) |
| ECS container instance | Kubernetes worker node |

These are conceptual similarities rather than identical implementations.

---

## 5. Ephemeral Storage and Persistent Volumes

Kubernetes Pods are temporary resources.

When a Pod is removed or replaced, data stored only in its ephemeral filesystem may be lost.

```text
Pod on Node 1
      |
      v
Ephemeral Storage
      |
      v
Pod Terminated
      |
      v
Temporary Data Lost
```

Applications should therefore avoid relying on temporary Pod storage for information that must survive replacement.

### Persistent Volumes

Kubernetes provides Persistent Volumes (PVs) for workloads requiring persistent storage.

```text
       Persistent Volume
               |
       +-------+-------+
       |               |
       v               v
    Old Pod         New Pod
```

On AWS, persistent storage can integrate with services such as:

- Amazon Elastic Block Store (EBS)
- Amazon Elastic File System (EFS)

The appropriate Kubernetes storage integration components are required.

---

# Part 2: Amazon Elastic Kubernetes Service (EKS)

## 6. What Is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service offered by AWS.

It allows organizations to run Kubernetes applications on AWS without managing the Kubernetes control plane infrastructure.

With EKS, AWS manages control plane components such as:

- Kubernetes API server
- etcd
- Scheduler
- Controller Manager

The control plane is managed by AWS and operates across multiple Availability Zones for high availability.

The deployment options differ primarily in how the worker nodes and other data plane infrastructure are managed.

---

## 7. Amazon EKS Architecture

Amazon EKS follows the same general Kubernetes architecture:

1. Control Plane
2. Data Plane

```text
                    Amazon EKS Cluster
                            |
              +-------------+-------------+
              |                           |
              v                           v
       AWS-Managed Control          Application
             Plane                   Data Plane
              |                           |
       +------+------+              +-----+------+
       |      |      |              |            |
       v      v      v              v            v
      API   etcd  Scheduler       Worker       Worker
     Server                       Node 1       Node 2
                                     |            |
                                     v            v
                                    Pods         Pods
```

### Control Plane

AWS manages the Kubernetes control plane.

This includes:

- Provisioning
- Availability
- Infrastructure maintenance
- Control plane scaling
- Supported upgrades and patching

### Data Plane

The data plane runs application workloads.

Depending on the deployment option, worker node infrastructure may be managed by the customer or by AWS.

---

# Part 3: Amazon EKS Deployment Options

## 8. Amazon EKS Standard Configuration

In a standard EKS configuration, AWS manages the Kubernetes control plane.

The customer is responsible for selecting and configuring the compute infrastructure used to run application Pods.

The course describes the data plane as running in the customer's AWS environment.

### Architecture

```text
                Amazon EKS Standard
                         |
           +-------------+-------------+
           |                           |
           v                           v
     AWS-Managed                 Customer Data
     Control Plane                  Plane
           |                           |
           v                           v
     API Server                    EC2 Nodes
     Scheduler                         |
     Controllers                       v
                                     Pods
```

### Customer Responsibilities

Depending on the node configuration, responsibilities may include:

- Provisioning worker nodes
- Selecting instance types
- Configuring VPC networking
- Selecting subnets
- Managing scaling
- Maintaining operating systems
- Configuring Kubernetes add-ons

EKS Standard provides flexibility in how the data plane is configured.

---

## 9. Amazon EKS Auto Mode

Amazon EKS Auto Mode is designed to reduce the operational work required to run Kubernetes clusters.

In this mode, AWS extends its management beyond the control plane to include major data plane infrastructure components.

### Architecture

```text
                 Amazon EKS Auto Mode
                          |
                AWS-Managed Services
                          |
           +--------------+--------------+
           |                             |
           v                             v
       Control Plane                 Data Plane
           |                             |
           v                             v
       API Server                  Managed Nodes
       Scheduler                         |
       Controllers                       v
                                      Pods
```

### Features Introduced in the Lesson

EKS Auto Mode automates several infrastructure responsibilities:

- Compute provisioning
- Node scaling
- Networking integration
- Load balancing integration
- Storage integration
- Infrastructure maintenance
- Node lifecycle management

The course also highlights DNS and GPU-related capabilities.

### Key Benefit

You focus more on defining Kubernetes application workloads and less on managing the underlying node infrastructure.

### Important Distinction

The transcript describes Auto Mode as running nodes in an AWS-managed account, with AWS managing their lifecycle.

**Technical clarification:** Auto Mode compute is AWS-managed, but it still uses networking configured for your EKS cluster, including your selected VPC and subnets. It is not accurate to assume that customers have no VPC or subnet configuration responsibilities.

Also, Auto Mode pricing should not be treated as identical to Fargate's per-Pod vCPU and memory pricing. Auto Mode has its own compute and management-related charges.

---

## 10. Amazon EKS Managed Node Groups

Managed node groups automate important parts of EC2 worker node provisioning and lifecycle management.

The EC2 instances are visible in your AWS account.

### Architecture

```text
              Amazon EKS
                  |
                  v
          Managed Node Group
                  |
          +-------+-------+
          |               |
          v               v
      EC2 Node 1      EC2 Node 2
          |               |
          v               v
         Pods            Pods
```

### Key Characteristics

- AWS assists with node provisioning and lifecycle management.
- Worker nodes are EC2 instances in your AWS account.
- You select the VPC and subnets.
- You can choose supported instance types and AMI configurations.
- You retain control over networking and security group configuration.

Managed node groups reduce operational overhead compared with fully self-managed nodes, but they do not eliminate all customer responsibilities.

---

## 11. Amazon EKS Self-Managed Nodes

Self-managed nodes provide greater control over EC2 worker instances.

The customer is responsible for provisioning, configuring, maintaining, and scaling the nodes.

### Architecture

```text
                Amazon EKS
                    |
                    v
            Self-Managed Nodes
                    |
          +---------+---------+
          |                   |
          v                   v
      EC2 Node 1          EC2 Node 2
          |                   |
          v                   v
         Pods                Pods
```

### Customer Responsibilities

- EC2 instance provisioning
- Operating system patching
- Node updates
- Node scaling
- Security configuration
- Node maintenance
- Cluster integration

### When to Consider Self-Managed Nodes

This approach may be appropriate when an organization needs extensive control over the underlying EC2 infrastructure.

The trade-off is greater operational responsibility.

---

## 12. Amazon EKS with AWS Fargate

AWS Fargate is a serverless compute option for running Kubernetes Pods on EKS.

With Fargate, you do not provision or maintain the underlying EC2 instances.

### Architecture

```text
                  Amazon EKS
                      |
                      v
                Fargate Profile
                      |
               +------+------+
               |             |
               v             v
          Fargate Pod 1  Fargate Pod 2
               |             |
               v             v
           Container     Container
```

### Key Characteristics

- No customer-managed EC2 worker nodes
- AWS manages the underlying compute infrastructure
- Pods run using Fargate
- VPC and subnet configuration remains the customer's responsibility
- A Fargate profile is required

### What Is a Fargate Profile?

A Fargate profile defines which Kubernetes Pods should run on AWS Fargate.

It uses selection criteria to determine eligible Pods.

**Important for the exam:** EKS on Fargate requires a Fargate profile.

---

# Part 4: Comparing EKS Compute Options

## 13. EKS Deployment Options Comparison

| Feature | Auto Mode | Managed Node Groups | Self-Managed Nodes | Fargate |
|---|---|---|---|---|
| AWS manages control plane | Yes | Yes | Yes | Yes |
| Customer manages EC2 hosts | No | Partially | Yes | No |
| EC2 nodes visible in customer account | AWS-managed node lifecycle | Yes | Yes | No customer-managed EC2 nodes |
| Customer selects VPC/subnets | Yes | Yes | Yes | Yes |
| Node provisioning | Automated | AWS-assisted | Customer | AWS-managed |
| OS patching responsibility | AWS-managed | Shared responsibilities | Customer | AWS-managed |
| Fargate profile required | No | No | No | Yes |
| Infrastructure control | Lower | Moderate | Highest | Lower |
| Operational overhead | Lower | Moderate | Highest | Lower |

### How to Choose

**EKS Auto Mode:** Choose when you want AWS to automate a broad range of Kubernetes infrastructure operations.

**Managed Node Groups:** Choose when you want EC2-based Kubernetes workers with AWS-assisted lifecycle management.

**Self-Managed Nodes:** Choose when you require extensive control over the EC2 worker infrastructure.

**Fargate:** Choose when you want to run eligible Pods without managing EC2 worker nodes.

---

# Part 5: Amazon EKS Add-ons

## 14. What Are EKS Add-ons?

EKS add-ons provide functionality needed to operate Kubernetes workloads on AWS.

The lesson introduces several components.

| Component | Purpose |
|---|---|
| Amazon VPC CNI | Provides VPC networking integration for Kubernetes Pods |
| Kube-Proxy | Maintains Kubernetes networking rules |
| Karpenter | Automates worker node provisioning and scaling |
| CSI Drivers | Integrate Kubernetes with persistent storage |

### Amazon VPC CNI

The Amazon VPC Container Network Interface (CNI) plugin supports networking integration between Kubernetes Pods and AWS VPC networking.

### Kube-Proxy

Kube-Proxy supports Kubernetes Service networking through network rules.

### Karpenter

Karpenter is a Kubernetes node autoscaler.

It helps provision compute capacity based on application workload requirements.

```text
New Pods Require Capacity
           |
           v
        Karpenter
           |
           v
Evaluate Compute Requirements
           |
           v
Provision Suitable Nodes
           |
           v
Schedule Pods
```

Karpenter can improve cluster efficiency by dynamically provisioning nodes that meet workload requirements.

---

# Part 6: Persistent Storage on Amazon EKS

## 15. Why Persistent Storage Matters

Kubernetes Pods are temporary resources.

If an application stores important data only in a Pod's ephemeral filesystem, that data may be lost when the Pod is replaced.

Applications requiring persistent data need an appropriate storage solution.

The lesson introduces:

- Persistent Volumes (PV)
- StorageClasses
- Container Storage Interface (CSI) drivers

---

## 16. Persistent Volumes

A Persistent Volume represents storage available for use by Kubernetes workloads.

```text
             Application Pod
                    |
                    v
          Persistent Storage
                    |
                    v
             Amazon EBS/EFS
```

The storage is managed independently of the lifecycle of an individual Pod.

---

## 17. StorageClass

A StorageClass describes how Kubernetes should provision storage.

The lesson highlights StorageClass manifests as part of configuring persistent storage for EKS workloads.

Conceptually:

```text
Application Requires Storage
             |
             v
       Storage Request
             |
             v
         StorageClass
             |
             v
         CSI Driver
             |
             v
       AWS Storage Service
```

A StorageClass is commonly used with a PersistentVolumeClaim to request dynamically provisioned storage.

---

## 18. Container Storage Interface (CSI)

CSI provides a standard way for Kubernetes to integrate with external storage systems.

For Amazon EKS, storage integrations can include:

- Amazon EBS CSI Driver
- Amazon EFS CSI Driver

### Example Architecture

```text
                 Kubernetes Pod
                       |
                       v
              PersistentVolumeClaim
                       |
                       v
                StorageClass
                       |
                       v
                  CSI Driver
                       |
              +--------+--------+
              |                 |
              v                 v
          Amazon EBS        Amazon EFS
```

**Exam point:** Persistent storage integration on EKS commonly involves Persistent Volumes, PersistentVolumeClaims, StorageClasses, and CSI drivers.

---

# Part 7: AWS Certification Exam Preparation

## 19. Scenario-Based Questions

### Scenario 1: Minimize Kubernetes Infrastructure Management

A company wants to deploy Kubernetes applications on AWS while minimizing the operational work associated with provisioning and managing worker nodes, networking integrations, and other cluster infrastructure.

**Recommended option:** Amazon EKS Auto Mode.

**Reason:** Auto Mode automates a broad range of Kubernetes infrastructure management tasks.

### Scenario 2: Use EC2 Instances with AWS-Assisted Node Management

A company wants Kubernetes worker nodes running as EC2 instances in its own AWS account but prefers AWS to assist with node provisioning and lifecycle management.

**Recommended option:** EKS Managed Node Groups.

### Scenario 3: Require Complete Control Over Worker Nodes

An organization needs to control EC2 instance configuration, patching, scaling, and node maintenance.

**Recommended option:** EKS Self-Managed Nodes.

### Scenario 4: Run Pods Without Managing EC2 Hosts

A company wants to deploy eligible Kubernetes Pods without provisioning or managing EC2 instances.

**Recommended option:** EKS with AWS Fargate.

### Scenario 5: Persist Application Data

A Kubernetes application needs its data to survive Pod replacement.

**Recommended approach:** Configure persistent storage using appropriate Kubernetes storage resources and a supported CSI driver.

---

## 20. Interview Questions

### Q1. What is Amazon EKS?

Amazon EKS is a managed Kubernetes service provided by AWS.

### Q2. What does AWS manage in a standard EKS cluster?

AWS manages the Kubernetes control plane, including its availability and underlying infrastructure.

### Q3. What is the difference between the control plane and data plane?

The control plane manages the Kubernetes cluster, while the data plane runs application Pods.

### Q4. What is EKS Auto Mode?

EKS Auto Mode automates the management of major Kubernetes data plane infrastructure components in addition to the AWS-managed control plane.

### Q5. What are EKS Managed Node Groups?

Managed node groups provide EC2 worker nodes with AWS-assisted provisioning and lifecycle management.

### Q6. What are self-managed nodes?

EC2 worker nodes that the customer provisions, configures, patches, and maintains.

### Q7. What is EKS Fargate?

A serverless compute option for running eligible Kubernetes Pods without managing EC2 worker hosts.

### Q8. What is a Fargate profile?

A configuration that determines which Kubernetes Pods are eligible to run on AWS Fargate.

### Q9. What is Karpenter?

A Kubernetes node autoscaler that provisions compute capacity based on workload requirements.

### Q10. What is Amazon VPC CNI?

A networking component that integrates Kubernetes Pod networking with AWS VPC networking.

### Q11. What is a Persistent Volume?

A Kubernetes resource representing persistent storage available to workloads.

### Q12. What is a StorageClass?

A Kubernetes resource that describes how storage should be provisioned.

### Q13. What is a CSI driver?

A component that allows Kubernetes to integrate with external storage systems.

### Q14. Which AWS storage services are discussed for EKS?

Amazon EBS and Amazon EFS.

### Q15. What is the main difference between ECS and EKS?

Amazon ECS is AWS's native container orchestration service, while Amazon EKS provides managed Kubernetes orchestration.

---

## 💡 Key Takeaways

- Kubernetes is an open-source container orchestration platform.
- Kubernetes clusters contain a control plane and worker nodes.
- Pods are the basic Kubernetes workload unit.
- Pods can contain one or more containers.
- Kubernetes Pods are temporary resources.
- Persistent storage is needed for data that must survive Pod replacement.
- Amazon EKS provides managed Kubernetes on AWS.
- AWS manages the EKS control plane.
- EKS Standard supports customer-configured worker infrastructure.
- EKS Auto Mode automates additional data plane infrastructure management.
- Managed node groups provide AWS-assisted EC2 node management.
- Self-managed nodes provide the greatest control over EC2 workers.
- AWS Fargate allows eligible Pods to run without customer-managed EC2 hosts.
- Fargate profiles determine which Pods run on Fargate.
- Karpenter helps automate node provisioning and scaling.
- Amazon VPC CNI provides networking integration.
- CSI drivers integrate Kubernetes with persistent storage services.
- Amazon EBS and EFS are storage options discussed for EKS workloads.

### Final Architecture Recap

```text
                       Amazon EKS
                           |
                    AWS Control Plane
                           |
              +------------+------------+
              |                         |
              v                         v
        EKS Standard               EKS Auto Mode
              |                         |
       +------+------+             AWS-Managed
       |      |      |             Data Plane
       v      v      v                  |
    Managed  Self-  Fargate             v
     Nodes  Managed  Pods          Application Pods
              |
              v
        Application Pods
```

### The Most Important Concept to Remember

**Amazon EKS manages the Kubernetes control plane. The compute option you choose determines how much responsibility you retain for the worker infrastructure.**

---

## 📚 Related Topics

- Kubernetes Control Plane
- Kubernetes Worker Nodes
- Kubernetes Pods
- Amazon Elastic Kubernetes Service (EKS)
- EKS Auto Mode
- EKS Managed Node Groups
- EKS Self-Managed Nodes
- AWS Fargate
- Amazon VPC CNI
- Karpenter
- Kubernetes Persistent Volumes
- Amazon EBS CSI Driver
- Amazon EFS CSI Driver

