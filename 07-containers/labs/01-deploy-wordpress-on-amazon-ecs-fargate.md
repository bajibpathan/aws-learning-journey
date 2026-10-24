# Lab: Deploy a WordPress Container Using Amazon ECS and AWS Fargate

> Deploy a WordPress application as a single Amazon ECS task using AWS Fargate, without provisioning or managing EC2 instances.

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

1. Prepare a VPC and security group for an ECS application.
2. Create an Amazon ECS cluster using AWS Fargate.
3. Identify the official WordPress container image on Docker Hub.
4. Create an ECS task definition.
5. Configure CPU, memory, IAM roles, and container networking.
6. Deploy a standalone ECS task.
7. Access the WordPress application using its public IP address.
8. Understand how ECS services can support scaling and load balancing.
9. Clean up ECS resources to avoid unnecessary charges.

---

## 1. Architecture Overview

The lab deploys one WordPress container using AWS Fargate.

```text
                         Internet
                            |
                            |
                         HTTP:80
                            |
                            v
                 ┌─────────────────────┐
                 │       AWS VPC       │
                 │    us-east-1        │
                 │                     │
                 │   Public Subnet     │
                 │                     │
                 │  ┌───────────────┐  │
                 │  │ Fargate Task  │  │
                 │  │               │  │
                 │  │ WordPress     │  │
                 │  │ Container     │  │
                 │  │ Port 80       │  │
                 │  └───────────────┘  │
                 └─────────────────────┘
                            |
                            v
                   AWS-Managed Compute
```

### Architecture Components

| Component | Purpose |
|---|---|
| Amazon ECS | Container orchestration |
| AWS Fargate | Serverless container compute |
| ECS Cluster | Logical grouping of ECS workloads |
| Task Definition | Configuration blueprint for the WordPress container |
| ECS Task | Running instance of the task definition |
| Docker Hub | Source of the WordPress container image |
| Default VPC | Network for the task |
| Public Subnet | Provides a network location for the publicly accessible task |
| Security Group | Controls inbound and outbound traffic |
| Public IP | Allows access to the WordPress application |

**Important:** This lab runs a standalone task, not an ECS service. It does not configure automatic task replacement, scaling, or an Application Load Balancer.

---

## 2. Prerequisites

Before starting, ensure that you have:

- An AWS account with permissions to manage ECS, IAM, VPC, and security groups.
- Access to the AWS Management Console.
- A default VPC in the selected Region.
- At least one public subnet with internet connectivity.
- A security group that permits HTTP traffic.
- An ECS task execution role, or permission to create one.

### AWS Region

The instructor uses:

```text
US East (N. Virginia)
us-east-1
```

The lesson deliberately uses this Region to keep the exercise separate from other projects.

### Security Group

Navigate to:

**VPC → Security Groups → Create security group**

Configure:

| Setting | Value |
|---|---|
| Name | `wordpress-ecs-demo-sg` |
| VPC | Default VPC |
| Inbound protocol | TCP |
| Inbound port | 80 |
| Source | `0.0.0.0/0` for this public demonstration |

The instructor's existing security group also allows SSH port 22, but explicitly notes that only port 80 is required for this lab.

There is no need to open SSH access to a Fargate task.

---

# Phase 1: Create the ECS Cluster

## Step 1: Open Amazon ECS

1. Sign in to the AWS Management Console.
2. Select the **N. Virginia (`us-east-1`)** Region.
3. Search for **Elastic Container Service**.
4. Open the Amazon ECS console.
5. Select **Clusters**.
6. Click **Create cluster**.

## Step 2: Configure the Cluster

Use the following configuration:

| Setting | Value |
|---|---|
| Cluster Name | `ritual-wordpress` |
| Infrastructure | AWS Fargate |

Fargate is selected because AWS manages the underlying compute infrastructure.

You do not need to create EC2 instances, install an ECS agent on those instances, or maintain the host operating system.

## Step 3: Create the Cluster

Click **Create**.

Wait until the cluster is successfully created.

### Validation

Open the cluster and confirm that it exists.

Initially, it should have no running tasks.

```text
ECS Cluster: ritual-wordpress

Running Tasks: 0
```

---

# Phase 2: Identify the WordPress Container Image

The next step is to identify the container image that ECS will run.

## Step 1: Open Docker Hub

Visit:

https://hub.docker.com/_/wordpress

The instructor uses the official WordPress image available on Docker Hub.

## Step 2: Identify the Image Name

For the lesson, the image is referenced as:

```text
wordpress
```

This is the container image that will be specified in the ECS task definition.

### Image Workflow

```text
Docker Hub
    |
    v
WordPress Image
    |
    v
ECS Task Definition
    |
    v
Fargate Task
    |
    v
WordPress Container
```

**Note:** The lab does not build a custom Docker image or use Amazon ECR. Those activities are reserved for a later capstone project.

---

# Phase 3: Create an ECS Task Definition

An ECS task definition describes how ECS should run the WordPress container.

## Step 1: Open Task Definitions

Navigate to:

**Amazon ECS → Task Definitions → Create new task definition**

## Step 2: Configure Task Definition Details

| Setting | Value |
|---|---|
| Task Definition Family | `ritual-wordpress-td` |
| Launch Type | AWS Fargate |
| Operating System | Linux |
| Network Mode | `awsvpc` |
| CPU | 0.5 vCPU |
| Memory | 1 GB |

### Why `awsvpc`?

For AWS Fargate tasks, the networking mode is `awsvpc`.

This provides the task with its own network interface and IP-based connectivity within the selected VPC.

### Why 0.5 vCPU and 1 GB Memory?

The instructor chooses a relatively small task size to demonstrate container deployment while keeping resource consumption modest.

This is a demonstration configuration, not a production sizing recommendation.

---

# Phase 4: Configure IAM Roles

The task definition includes IAM role configuration.

Two role types are relevant.

## Task Role

A task role grants permissions to application code running inside the containers.

For example, an application may require permission to access AWS services.

The instructor mentions Amazon EFS as a possible future integration.

No additional application-specific task role is configured for this simple WordPress demonstration.

## Task Execution Role

The task execution role allows ECS to perform supported actions required to start and operate the task.

Examples include retrieving supported container images and sending container logs to CloudWatch.

The lesson uses the managed policy:

```text
AmazonECSTaskExecutionRolePolicy
```

### Configuration

In the task definition:

1. Locate **Task execution role**.
2. Select the existing ECS task execution role.
3. If the role is unavailable, use the console option to create one.

The role is separate from the application's task role.

---

# Phase 5: Configure the WordPress Container

In the task definition, configure the container.

| Setting | Value |
|---|---|
| Container Name | `ritual-wordpress` |
| Image URI | `wordpress` |
| Essential Container | Yes |
| Container Port | 80 |
| Protocol | TCP |
| Application Protocol | HTTP |

### Container Configuration

```text
Task Definition
       |
       v
WordPress Container
       |
       +-- Image: wordpress
       +-- Port: 80
       +-- Protocol: TCP
       +-- Application Protocol: HTTP
```

## Environment Variables

The instructor does not configure database environment variables for this demonstration.

The transcript explains that an externally hosted database would require connection details.

For example, a WordPress deployment may need:

```text
WORDPRESS_DB_HOST
WORDPRESS_DB_USER
WORDPRESS_DB_PASSWORD
WORDPRESS_DB_NAME
```

These are examples of the database settings WordPress can use, not values supplied in the course.

### Important Database Limitation

The instructor describes the exercise as a simple monolithic demonstration, but the task definition shown contains only the WordPress container. It does **not** explicitly configure or launch a MySQL database.

Therefore, this lab demonstrates launching the WordPress container and reaching its installation/configuration screen. It does not demonstrate a completed, database-backed WordPress website.

## Logging

The task definition also provides options for container logging through Amazon CloudWatch.

The instructor introduces these settings but does not perform a detailed logging configuration exercise.

## Create the Task Definition

Review the configuration and click **Create**.

### Validation

Confirm that the task definition appears under:

**ECS → Task Definitions**

Expected family:

```text
ritual-wordpress-td
```

---

# Phase 6: Deploy the ECS Task

Now that the cluster and task definition exist, launch the WordPress container.

## Step 1: Run the Task

Open the WordPress task definition.

Choose **Run task**.

## Step 2: Configure Deployment

| Setting | Value |
|---|---|
| Existing Cluster | `ritual-wordpress` |
| Launch Type | Fargate |
| Platform Version | Latest |
| Number of Tasks | 1 |
| VPC | Default VPC |
| Subnet | One public subnet in `us-east-1a` |
| Security Group | `wordpress-ecs-demo-sg` |
| Public IP | Enabled |

The instructor deploys a single task into one Availability Zone.

### Why Enable Public IP?

The purpose of this demonstration is to access the WordPress container directly from a browser without creating an Application Load Balancer.

Assigning a public IP allows the task to be reached through the internet, provided the subnet routing and security group are correctly configured.

## Step 3: Launch the Task

Click **Create** or **Run task**, depending on the console workflow.

ECS begins provisioning the Fargate task.

---

# Phase 7: Verify Task Deployment

Navigate to:

**Amazon ECS → Clusters → ritual-wordpress → Tasks**

Wait for the task to reach:

```text
Last Status: RUNNING
```

Open the task details.

Verify:

| Property | Expected |
|---|---|
| Launch Type | FARGATE |
| Task Definition | `ritual-wordpress-td` |
| Desired Status | RUNNING |
| Last Status | RUNNING |
| Container | WordPress |

The task should also have networking information, including its assigned IP address.

---

# Phase 8: Access the WordPress Application

## Step 1: Find the Public IP

Open the running task.

Navigate to its **Networking** section.

Locate the public IP address associated with the task.

## Step 2: Open WordPress

In a browser, enter:

```text
http://<TASK_PUBLIC_IP>
```

Replace `<TASK_PUBLIC_IP>` with the actual public IP assigned to your task.

Use HTTP for this demonstration because the task is configured on port 80 and no HTTPS listener or certificate has been configured.

## Expected Result

The instructor reaches the WordPress setup/configuration screen.

This confirms that:

- ECS successfully launched the Fargate task.
- The WordPress image was retrieved.
- The container started.
- The task has network connectivity.
- HTTP traffic reaches the WordPress container.

### WordPress Database Setup

The WordPress installation screen requests database information such as:

| Field | Purpose |
|---|---|
| Database Name | Name of the WordPress database |
| Username | Database username |
| Password | Database password |
| Database Host | Database server address |
| Table Prefix | Prefix for WordPress database tables |

The instructor stops at this stage and does not complete the WordPress installation.

**Lab success criteria:** The WordPress setup page is accessible through the Fargate task's public IP.

---

# Phase 9: Understand ECS Services and Scaling

The lab launches a single standalone ECS task.

The instructor briefly explains that a more complete deployment could use an ECS service.

An ECS service can maintain a desired number of running tasks.

For example:

```text
ECS Service
     |
     +-- Task 1
     +-- Task 2
     +-- Task 3
     +-- Task 4
```

A service can also integrate with an Application Load Balancer.

```text
                   Internet
                      |
                      v
           Application Load Balancer
                      |
                      v
                 Target Group
                      |
             +--------+--------+
             |                 |
             v                 v
        Fargate Task 1    Fargate Task 2
             |                 |
             v                 v
          WordPress          WordPress
```

These capabilities are discussed but not configured in this lab.

The instructor notes that a later ECS capstone project will cover a more complete deployment, including building a custom Docker image.

---

# Phase 10: Cleanup

Fargate tasks consume billable compute resources while running.

After completing the demonstration, remove the resources created specifically for this lab.

## Step 1: Stop the ECS Task

Navigate to:

**ECS → Clusters → ritual-wordpress → Tasks**

Select the running WordPress task.

Choose **Stop task**.

Confirm that the task stops.

## Step 2: Delete the ECS Cluster

Navigate to:

**ECS → Clusters**

Select:

```text
ritual-wordpress
```

Choose **Delete cluster**.

Confirm the deletion.

Wait for the cluster to disappear from the cluster list.

## Step 3: Deregister the Task Definition

Navigate to:

**ECS → Task Definitions**

Select:

```text
ritual-wordpress-td
```

Choose **Actions → Deregister**.

This removes the active task definition revision from normal deployment selection.

## Step 4: Review the IAM Execution Role

Navigate to:

**IAM → Roles**

Locate the task execution role used for the lab.

The instructor deletes the role during cleanup.

However, delete it only if it was created exclusively for this exercise and is not required by another ECS workload.

## Step 5: Delete the Security Group

Navigate to:

**VPC → Security Groups**

Select:

```text
wordpress-ecs-demo-sg
```

Delete the security group after confirming it is no longer attached to any resources.

---

## ✅ Cleanup Checklist

- [ ] WordPress ECS task stopped
- [ ] ECS cluster deleted
- [ ] Task definition deregistered
- [ ] Lab-specific IAM role reviewed
- [ ] Lab-specific security group deleted
- [ ] No unintended Fargate tasks remain running

---

## 🔧 Troubleshooting

### Issue 1: Task Remains in PENDING

Check:

- Task definition configuration
- Fargate CPU and memory settings
- Subnet selection
- Network connectivity
- ECS task execution role

### Issue 2: Task Stops Immediately

Open the stopped task and inspect:

- Stopped reason
- Container exit code
- Container reason
- CloudWatch logs, if configured

These details help identify why the container could not remain running.

### Issue 3: Cannot Access the Public IP

Verify:

- Task status is `RUNNING`.
- A public IP was assigned.
- The selected subnet is public and has a route to an Internet Gateway.
- The security group permits inbound TCP port 80.
- The browser uses `http://` rather than `https://`.

### Issue 4: WordPress Displays a Database Setup Page

This is expected for the demonstration.

The lab does not configure a database backend or complete the WordPress installation.

### Issue 5: Container Image Cannot Be Retrieved

Verify that the image reference is correct and that the task has outbound network connectivity.

---

## ❓ Interview Questions

### Q1. What is the objective of this lab?

To deploy a WordPress container as a standalone ECS task using AWS Fargate.

### Q2. Which ECS launch type is used?

AWS Fargate.

### Q3. Why is Fargate selected?

Because AWS manages the underlying compute infrastructure, eliminating the need to provision and maintain EC2 container hosts.

### Q4. What is the purpose of the ECS cluster?

It logically groups the ECS tasks and services associated with the application.

### Q5. What is a task definition?

A blueprint that describes how ECS should run one or more containers.

### Q6. What is an ECS task?

A running instance of a task definition.

### Q7. Where is the WordPress image obtained?

Docker Hub.

### Q8. What image is used?

The official WordPress image, referenced as `wordpress` in the lesson.

### Q9. What CPU and memory settings are used?

0.5 vCPU and 1 GB memory.

### Q10. What network mode is required for Fargate?

`awsvpc`.

### Q11. Why is a public IP assigned?

To allow the browser to access the task directly over the internet for this demonstration.

### Q12. Which port is used for the WordPress application?

HTTP port 80.

### Q13. What is the purpose of the ECS task execution role?

It grants ECS the permissions required for supported task startup and operation activities, such as image retrieval and CloudWatch logging.

### Q14. What is the difference between a task role and a task execution role?

A task role provides permissions to application code inside the containers.

A task execution role provides permissions used by ECS to start and operate the task.

### Q15. Does this lab deploy an ECS service?

No. It launches one standalone ECS task.

### Q16. Does this lab configure Auto Scaling?

No. The instructor discusses ECS services and scaling but does not configure them.

### Q17. Does this lab complete WordPress installation?

No. The demonstration stops after accessing the WordPress configuration page.

### Q18. Why should the ECS task be stopped after the lab?

To avoid continued Fargate compute charges.

---

## 💡 Key Takeaways

- Amazon ECS can deploy containers using AWS Fargate without customer-managed EC2 hosts.
- An ECS cluster logically groups container workloads.
- A task definition specifies the configuration required to run containers.
- ECS can retrieve public container images from Docker Hub.
- Fargate tasks use `awsvpc` networking.
- The lab deploys one WordPress container with 0.5 vCPU and 1 GB memory.
- A public subnet, public IP, and HTTP security group rule allow direct browser access.
- The ECS task execution role is separate from the application's task role.
- A standalone ECS task does not provide the same desired-count maintenance behavior as an ECS service.
- ECS services can later be used for task scaling and load-balancer integration.
- The WordPress setup page confirms the container is accessible, but does not establish that a complete WordPress website has been configured.
- Lab-specific resources should be cleaned up after testing.

### Final Architecture Recap

```text
Docker Hub
    |
    v
WordPress Container Image
    |
    v
ECS Task Definition
    |
    v
Amazon ECS Cluster
    |
    v
AWS Fargate
    |
    v
Single ECS Task
    |
    v
WordPress Container :80
    |
    v
Public IP Address
    |
    v
Browser → WordPress Setup Page
```

---

## 📚 Related Topics

- Amazon Elastic Container Service (ECS)
- AWS Fargate
- Docker Hub
- Amazon Elastic Container Registry (ECR)
- ECS Task Definitions
- ECS Task Execution Roles
- ECS Networking
- ECS Services
- Application Load Balancers
- ECS Service Auto Scaling

**Next topic:** ECS Capstone Project — Deploying a Complete Containerized Application