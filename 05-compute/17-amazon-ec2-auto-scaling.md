# 📈 Amazon EC2 Auto Scaling

> Amazon EC2 Auto Scaling automatically adjusts the number of EC2 instances based on application demand, helping maintain availability while avoiding unnecessary capacity and cost.

---

# 📖 Overview

Traditional on-premises environments often used **peak load provisioning**.

If an application required its highest capacity on Tuesday, enough infrastructure had to be purchased to handle that peak even if most of that capacity was unused during the rest of the week.

```text
Capacity
   │
   │       Tuesday Peak
   │          ███
   │       ███████
   │    ███████████
   │ █████████████████
   └──────────────────────► Time
```

This creates two problems:

- Overprovisioning wastes resources and money.
- Underprovisioning can cause poor application performance during peak demand.

AWS Auto Scaling addresses this by dynamically adjusting capacity according to demand.

---

# 🎯 Why Auto Scaling?

Instead of permanently provisioning infrastructure for peak usage:

```text
Low Demand
    │
    ▼
Scale In
Remove Unneeded Capacity


High Demand
    │
    ▼
Scale Out
Add Capacity
```

The objective is to provide:

- Enough capacity to handle demand
- Better application availability
- Improved resource utilization
- Reduced unnecessary cost
- Automated capacity management

---

# 📈 Scale Out vs Scale In

Two important terms are:

### Scale Out

Add more instances when demand increases.

```text
2 Instances
     │
     │ Demand ↑
     ▼
4 Instances
```

### Scale In

Remove unnecessary instances when demand decreases.

```text
4 Instances
     │
     │ Demand ↓
     ▼
2 Instances
```

The goal is to match available compute capacity with application demand.

---

# 🏗️ Types of Auto Scaling

The lesson introduces two broad categories:

## 1. Amazon EC2 Auto Scaling

Used to automatically adjust the number of EC2 instances.

```text
Demand ↑
   │
   ▼
EC2 Auto Scaling
   │
   ▼
Launch More Instances
```

When demand decreases:

```text
Demand ↓
   │
   ▼
EC2 Auto Scaling
   │
   ▼
Terminate Unneeded Instances
```

Scaling can be driven by policies or schedules.

---

## 2. Application Auto Scaling

Application Auto Scaling provides scaling capabilities for supported AWS services.

Examples discussed in the lesson include:

### Amazon ECS

Scale the number of container tasks:

```text
Demand ↑
   │
   ▼
Additional ECS Tasks
```

### Amazon DynamoDB

Scale:

- Read capacity
- Write capacity
- Capacity associated with global secondary indexes

### Amazon Aurora

Scale the number of Aurora read replicas as read demand changes.

The key distinction is:

| Service | What Can Scale |
|---|---|
| EC2 Auto Scaling | EC2 instances |
| Amazon ECS | Tasks |
| DynamoDB | Read/write capacity |
| Aurora | Read replicas |

---

# 🔢 Minimum, Desired, and Maximum Capacity

An Auto Scaling Group requires three important capacity settings:

```text
Minimum Capacity

Desired Capacity

Maximum Capacity
```

Example:

```text
Minimum = 2
Desired = 4
Maximum = 8
```

---

# Minimum Capacity

Minimum capacity defines the lowest number of instances that the Auto Scaling Group should maintain.

```text
Minimum = 2
```

Even during low demand:

```text
ASG
├── EC2
└── EC2
```

the group should not scale below two instances.

This helps maintain baseline application capacity.

---

# Desired Capacity

Desired capacity represents the number of instances the Auto Scaling Group currently tries to maintain.

Example:

```text
Desired = 4
```

The group aims to have:

```text
ASG
├── EC2
├── EC2
├── EC2
└── EC2
```

Desired capacity can change as scaling activities occur, but remains between the minimum and maximum limits.

---

# Maximum Capacity

Maximum capacity defines the upper scaling limit.

Example:

```text
Maximum = 8
```

Even if demand continues increasing, the Auto Scaling Group will not exceed the configured maximum.

The lesson highlights maximum capacity as useful for:

- Cost control
- Preventing excessive scaling
- Protecting against misconfiguration
- Limiting the impact of application behavior that might otherwise trigger repeated scaling

---

# 📊 Capacity Example

Suppose:

| Setting | Instances |
|---|---:|
| Minimum | 2 |
| Desired | 4 |
| Maximum | 8 |

Then:

```text
                 Auto Scaling Group

Minimum                                      Maximum
   │                                             │
   ▼                                             ▼
   2 ─────── 3 ─────── 4 ─────── 5 ... ─────── 8
                       ▲
                       │
                    Desired
```

Under normal conditions:

```text
4 Instances
```

During low demand, the group can scale down, but not below:

```text
2 Instances
```

During high demand, it can scale out, but not above:

```text
8 Instances
```

These three values define the operating boundaries of the Auto Scaling Group.

---

# 🏗️ Typical Auto Scaling Architecture

The lesson describes an architecture with:

- VPC
- Two Availability Zones
- EC2 Auto Scaling Group
- Elastic Load Balancer
- CloudWatch monitoring

```text
                       Users
                         │
                         ▼
                 Elastic Load Balancer
                         │
                ┌────────┴────────┐
                ▼                 ▼
               AZ-A              AZ-B
                │                 │
             ┌──┴──┐           ┌──┴──┐
             │     │           │     │
            EC2   EC2         EC2   EC2
             │     │           │     │
             └─────┴─────┬─────┴─────┘
                         │
                         ▼
                Auto Scaling Group
                         │
                         ▼
                    CloudWatch
```

The instances are distributed across multiple Availability Zones, while the load balancer distributes incoming application traffic.

---

# 🔗 Auto Scaling + Elastic Load Balancing

EC2 Auto Scaling can integrate with an Elastic Load Balancer.

When Auto Scaling launches a new instance:

```text
Demand Increases
      │
      ▼
Auto Scaling
      │
      ▼
Launch EC2
      │
      ▼
Register Instance
      │
      ▼
Load Balancer
      │
      ▼
Instance Receives Traffic
```

When an instance is removed, it no longer participates in serving application traffic.

This combination provides:

```text
Elastic Load Balancing
        +
EC2 Auto Scaling
        =
Traffic Distribution
        +
Dynamic Capacity
```

---

# 📊 CloudWatch and Auto Scaling

Amazon CloudWatch can monitor metrics used to make scaling decisions.

Examples discussed include:

- CPU utilization
- Network traffic
- Load balancer request counts

Suppose CPU utilization rises significantly.

```text
Users ↑
   │
   ▼
CPU Utilization ↑
   │
   ▼
CloudWatch Metric
   │
   ▼
Alarm
   │
   ▼
Auto Scaling Action
   │
   ▼
Launch More EC2 Instances
```

The lesson uses an example of CPU utilization reaching approximately **80% for 15 minutes** as a possible trigger.

Once additional instances are available, workload can be distributed across more capacity.

---

# 📉 Scaling In

The opposite process can occur when demand decreases.

```text
Users ↓
   │
   ▼
CPU Utilization ↓
   │
   ▼
CloudWatch
   │
   ▼
Scaling Action
   │
   ▼
Terminate Unneeded EC2
```

The lesson uses an example of CPU utilization dropping below approximately **20%** as a possible scale-in condition.

These percentages are examples used to explain the concept rather than universal thresholds.

---

# ❤️ Auto Scaling Health Checks

Health checking is another important responsibility of the Auto Scaling Group.

The lesson discusses two health-check perspectives:

```text
EC2 Health Checks

+

ELB Health Checks
```

---

# EC2 Status Checks

EC2 health checks determine whether the EC2 instance itself is operating.

Conceptually:

```text
EC2 Instance
     │
     ▼
Is the Instance Healthy?
```

This focuses on the health of the compute resource.

---

# ELB Health Checks

Load balancer health checks can verify whether the application is responding correctly.

For example:

```text
EC2 Running
     │
     ▼
Application
     │
     ▼
HTTP Health Check
     │
     ▼
200 OK?
```

An instance can therefore be running while its application is not functioning correctly.

This is why application-level health checks are valuable.

---

# 🆚 EC2 vs ELB Health Checks

| Check | What It Helps Determine |
|---|---|
| EC2 Health Check | Whether the instance itself is healthy |
| ELB Health Check | Whether the application is responding correctly |

Using both gives a broader view:

```text
Is the Server Healthy?
        +
Is the Application Healthy?
```

---

# 🔄 What Happens When an Instance Becomes Unhealthy?

The lesson highlights an important difference between the load balancer and Auto Scaling.

### Load Balancer

If a target becomes unhealthy:

```text
Unhealthy Target
      │
      ▼
Stop Sending Traffic
```

### Auto Scaling Group

If Auto Scaling determines an instance is unhealthy:

```text
Unhealthy EC2
      │
      ▼
Auto Scaling
      │
      ├── Terminate Unhealthy Instance
      │
      └── Launch Replacement
```

The goal is to return the group to its required capacity.

For example:

```text
Desired Capacity = 4

4 Instances
    │
    ▼
1 Becomes Unhealthy
    │
    ▼
Replace Instance
    │
    ▼
4 Healthy Instances
```

The lesson specifically describes Auto Scaling replacing unhealthy instances to maintain desired capacity.

---

# ⚙️ Auto Scaling Policies

The lesson covers several ways of controlling scaling:

1. Manual Scaling
2. Dynamic Scaling
   - Target Tracking
   - Step Scaling
3. Scheduled Scaling
4. Predictive Scaling

---

# 1️⃣ Manual Scaling

Manual scaling means manually changing the number of resources in the Auto Scaling Group.

```text
Administrator
      │
      ▼
Change Capacity
      │
      ▼
Auto Scaling Group
```

This can be useful for one-time capacity changes.

However, it does not provide automatic response to changing demand.

---

# 2️⃣ Dynamic Scaling

Dynamic scaling adjusts capacity according to monitored metrics.

Examples include:

```text
CPU Utilization

Network Traffic

Load Balancer Requests
```

The general flow is:

```text
CloudWatch Metric
      │
      ▼
Threshold Reached
      │
      ▼
Alarm
      │
      ▼
Scaling Action
```

The lesson discusses two dynamic scaling approaches:

- Target tracking
- Step scaling

---

# 🎯 Target Tracking Scaling

Target tracking works similarly to a thermostat.

Suppose the target is:

```text
Average CPU Utilization = 30%
```

If CPU rises:

```text
CPU > 30%
    │
    ▼
Increase Capacity
```

If CPU falls sufficiently:

```text
CPU < Target
    │
    ▼
Reduce Capacity
```

The goal is to keep the monitored metric around the configured target value.

The thermostat analogy is useful:

```text
Thermostat
    │
    ▼
Maintain 18°C


Target Tracking
    │
    ▼
Maintain Target Metric
```

---

# 📶 Step Scaling

Step scaling changes capacity by different amounts depending on the magnitude of the metric.

The examples in the lesson are:

| CPU Utilization | Example Scaling Action |
|---|---|
| 60% | Add 10 instances |
| 75% | Add 30 instances |
| 85% | Add 40 instances |

Conceptually:

```text
Moderate Load
     │
     ▼
Small Scaling Action


Higher Load
     │
     ▼
Larger Scaling Action
```

The greater the threshold breach, the larger the configured scaling adjustment can be.

---

# 🗓️ Scheduled Scaling

Scheduled scaling is useful when demand follows a known schedule.

For example:

```text
Wednesday Morning
       │
       ▼
Increase Desired Capacity


Friday Evening
       │
       ▼
Decrease Desired Capacity
```

This works well when demand is predictable based on time.

---

# 🔮 Predictive Scaling

Predictive scaling uses historical load patterns to forecast future capacity requirements.

The lesson describes patterns such as:

```text
Daily Patterns

Weekly Patterns
```

The conceptual flow is:

```text
Historical Metrics
       │
       ▼
Analyze Usage Patterns
       │
       ▼
Forecast Demand
       │
       ▼
Increase Capacity
Before Demand Arrives
```

Instead of waiting for load to increase before reacting, predictive scaling attempts to prepare capacity in advance.

---

# 📊 Predictive Scaling Details from the Lesson

The lesson describes predictive scaling as requiring at least:

```text
24 Hours
```

of metric data before forecasting can begin.

It can analyze up to:

```text
14 Days
```

of historical metric data to identify patterns.

The lesson further describes:

```text
Forecast Window = Next 48 Hours

Forecast Granularity = Hourly

Forecast Updated = Every 6 Hours
```

As new CloudWatch data becomes available, the forecast can continue to improve.

---

# 📊 Scaling Policy Comparison

| Scaling Method | Best Fit |
|---|---|
| Manual | One-time manual capacity changes |
| Target Tracking | Maintain a target metric |
| Step Scaling | Different scaling actions for different thresholds |
| Scheduled | Predictable time-based demand |
| Predictive | Recurring demand patterns based on historical data |

A simple way to distinguish them:

```text
"I'll change it myself"
        │
        ▼
      Manual


"Keep CPU around 30%"
        │
        ▼
 Target Tracking


"Add more as CPU gets higher"
        │
        ▼
   Step Scaling


"Every Wednesday morning"
        │
        ▼
 Scheduled Scaling


"Learn the historical pattern"
        │
        ▼
 Predictive Scaling
```

---

# 🚀 Launch Templates

Before creating an EC2 Auto Scaling Group, the lesson introduces a:

```text
Launch Template
```

The launch template defines how new EC2 instances should be configured.

It can contain settings such as:

- AMI
- Instance type
- EBS volumes
- Security groups
- IAM instance profile
- User data
- Key pair
- Termination protection
- Other EC2 configuration

Think of it as the blueprint for new instances:

```text
Launch Template
      │
      ├── AMI
      ├── Instance Type
      ├── Storage
      ├── Security Group
      ├── IAM Role
      └── User Data
             │
             ▼
         New EC2
```

Every time the Auto Scaling Group needs additional capacity, the launch template provides the configuration required to create the new instances.

---

# 🏗️ Creating an Auto Scaling Group

Once the launch template exists, create the:

```text
Auto Scaling Group
```

The lesson identifies several important configuration areas.

### 1. VPC and Subnets

Define where the EC2 instances should be launched.

For high availability:

```text
Auto Scaling Group
       │
   ┌───┴───┐
   ▼       ▼
  AZ-A    AZ-B
```

---

### 2. Load Balancer Integration

Optionally attach an Elastic Load Balancer.

New instances can then be registered so they can receive application traffic.

---

### 3. Health Checks

Configure:

```text
EC2 Health Checks
```

or:

```text
EC2 + ELB Health Checks
```

depending on the architecture.

---

### 4. Capacity

Define:

```text
Minimum

Desired

Maximum
```

capacity.

---

### 5. Scaling Policies

Configure the required scaling behavior, such as:

- Target tracking
- Step scaling
- Scheduled scaling
- Predictive scaling

---

# 🔄 Complete Auto Scaling Flow

Putting the concepts together:

```text
                     USERS
                       │
                       ▼
              ELASTIC LOAD BALANCER
                       │
                       ▼
                AUTO SCALING GROUP
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
            AZ-A                AZ-B
             │                   │
            EC2                 EC2
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
                   CLOUDWATCH
                       │
                 Monitor Metrics
                       │
              ┌────────┴────────┐
              ▼                 ▼
         Demand Rises      Demand Falls
              │                 │
              ▼                 ▼
          SCALE OUT          SCALE IN
```

The launch template defines how replacement or additional instances are created.

---

# ⚠️ Common Mistakes

## Mistake 1: Confusing Scale Up with Scale Out

The lesson focuses on adding and removing EC2 instances:

```text
Add Instances
=
Scale Out

Remove Instances
=
Scale In
```

---

## Mistake 2: Thinking Desired Capacity Is Always Fixed

Desired capacity is the target number of instances the group currently tries to maintain and can change as scaling activities occur.

---

## Mistake 3: Forgetting Maximum Capacity

Even if demand continues rising, Auto Scaling does not scale beyond the configured maximum.

```text
Maximum = 8

Demand ↑ ↑ ↑

Still Maximum = 8
```

---

## Mistake 4: Thinking a Load Balancer Replaces an Unhealthy Instance

The load balancer:

```text
Stops Sending Traffic
```

to an unhealthy target.

The Auto Scaling Group can:

```text
Replace the Unhealthy Instance
```

when configured to use the relevant health information.

---

## Mistake 5: Confusing Scheduled and Predictive Scaling

Scheduled scaling:

```text
Known Time
→ Change Capacity
```

Predictive scaling:

```text
Historical Patterns
→ Forecast Demand
→ Prepare Capacity
```

---

## Mistake 6: Forgetting the Launch Template

The Auto Scaling Group needs to know how to create new EC2 instances.

That configuration comes from the:

```text
Launch Template
```

---

# ❓ Interview Questions

### Q1. What is Amazon EC2 Auto Scaling?

Amazon EC2 Auto Scaling automatically adjusts the number of EC2 instances to help match application capacity with demand.

### Q2. What is scaling out?

Adding EC2 instances when more capacity is required.

### Q3. What is scaling in?

Removing unnecessary EC2 instances when demand decreases.

### Q4. What are minimum, desired, and maximum capacity?

- **Minimum:** Lowest number of instances the group maintains.
- **Desired:** Number of instances the group currently tries to maintain.
- **Maximum:** Highest number of instances the group can provision.

### Q5. Why use Auto Scaling with a load balancer?

The Auto Scaling Group adjusts application capacity while the load balancer distributes traffic across the available healthy instances.

### Q6. What role does CloudWatch play?

CloudWatch monitors metrics that can be used to trigger scaling actions.

### Q7. What happens when an Auto Scaling instance becomes unhealthy?

The lesson describes Auto Scaling terminating the unhealthy instance and launching a replacement to maintain desired capacity.

### Q8. What is target tracking scaling?

A dynamic scaling method that adjusts capacity to keep a selected metric near a configured target value.

### Q9. What is step scaling?

A scaling method where different threshold levels trigger different scaling adjustments.

### Q10. What is scheduled scaling?

Scaling based on a known schedule.

### Q11. What is predictive scaling?

Scaling that analyzes historical usage patterns and forecasts future capacity requirements.

### Q12. What is a launch template?

A template defining how Auto Scaling should configure newly launched EC2 instances.

### Q13. What can a launch template contain?

The lesson mentions:

- AMI
- Instance type
- EBS volumes
- Security groups
- IAM instance profile
- User data
- Key pairs
- Termination protection

### Q14. Why use multiple Availability Zones?

To distribute application capacity across multiple locations and improve availability.

### Q15. What is the difference between ELB health checks and Auto Scaling health handling?

The load balancer stops sending traffic to an unhealthy target. Auto Scaling can replace an unhealthy instance to restore the required group capacity.

---

# 💡 Key Takeaways

- Traditional peak-load provisioning can create large amounts of unused capacity.
- Auto Scaling adjusts capacity according to application demand.
- **Scale out** means adding instances.
- **Scale in** means removing instances.
- EC2 Auto Scaling adjusts EC2 instance capacity.
- Application Auto Scaling can scale supported services such as ECS, DynamoDB, and Aurora.
- Every Auto Scaling Group has minimum, desired, and maximum capacity settings.
- Maximum capacity helps control scaling and cost.
- CloudWatch metrics can drive automatic scaling decisions.
- Auto Scaling can integrate with Elastic Load Balancing.
- Load balancers distribute traffic while Auto Scaling manages capacity.
- Health checks can help Auto Scaling identify instances that need replacement.
- Target tracking attempts to maintain a target metric value.
- Step scaling performs different adjustments based on threshold levels.
- Scheduled scaling handles known time-based demand.
- Predictive scaling uses historical patterns to forecast future demand.
- Launch templates define how new EC2 instances should be created.
- Auto Scaling Groups can distribute instances across multiple Availability Zones.

The core architecture to remember is:

```text
Users
  │
  ▼
Load Balancer
  │
  ▼
Auto Scaling Group
  │
  ├── AZ-A → EC2 EC2
  │
  └── AZ-B → EC2 EC2
  │
  ▼
CloudWatch Metrics
  │
  ├── Demand ↑ → Scale Out
  │
  └── Demand ↓ → Scale In
```

And the three capacity values are:

```text
MINIMUM
   │
   │  Never go below
   ▼

DESIRED
   │
   │  Try to maintain
   ▼

MAXIMUM
   │
   │  Never go above
   ▼
```

---

# 📚 Related Topics

- Amazon EC2 Auto Scaling
- Auto Scaling Groups
- Launch Templates
- Elastic Load Balancing
- Amazon CloudWatch
- EC2 Health Checks
- ELB Health Checks
- Target Tracking Scaling
- Step Scaling
- Scheduled Scaling
- Predictive Scaling
- Application Auto Scaling
- High Availability
- Multi-AZ Architecture