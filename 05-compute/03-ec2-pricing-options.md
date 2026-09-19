# 💰 Amazon EC2 Purchasing Options

> Amazon EC2 provides multiple purchasing options that allow organizations to balance flexibility, cost optimization, workload interruption tolerance, capacity assurance, licensing, and compliance requirements.

---

# 📖 Overview

Amazon EC2 supports different purchasing options because workloads do not all have the same requirements.

For example:

```text
Development Environment
        │
        └── Short-term and unpredictable

Production Application
        │
        └── Long-running and predictable

Batch Processing
        │
        └── Can tolerate interruption

Licensed Enterprise Software
        │
        └── May require dedicated hardware

Critical Event
        │
        └── Capacity must be available
```

AWS provides several EC2 purchasing options to address these requirements.

| Purchasing Option               | Primary Purpose                                   |
| ------------------------------- | ------------------------------------------------- |
| On-Demand Instances             | Flexibility without long-term commitment          |
| Savings Plans                   | Reduce cost through usage commitment              |
| Reserved Instances              | Reduce EC2 cost through configuration commitment  |
| Spot Instances                  | Use spare EC2 capacity at significant discounts   |
| Dedicated Hosts                 | Dedicated physical server with host-level control |
| Dedicated Instances             | Instances running on single-tenant hardware       |
| On-Demand Capacity Reservations | Reserve EC2 capacity in a specific AZ             |

The correct option depends on the workload rather than simply selecting the option with the largest discount.

---

# 🟢 On-Demand Instances

On-Demand Instances allow you to use EC2 compute capacity without making a long-term commitment.

You control the lifecycle of the instance:

```text
Launch
  │
  ▼
Run
  │
  ▼
Stop / Hibernate / Reboot
  │
  ▼
Start
  │
  ▼
Terminate
```

For supported operating systems, AWS bills On-Demand usage per second with a minimum charge of 60 seconds.

There is no one-year or three-year commitment.

---

# 🎯 When to Use On-Demand

On-Demand is suitable for workloads that are:

* Short-term
* Irregular
* Unpredictable
* Unable to tolerate interruption
* Not ready for a long-term usage commitment

Example:

```text
New Application
      │
      ▼
Traffic Unknown
      │
      ▼
Usage Unpredictable
      │
      ▼
On-Demand
```

This makes On-Demand useful when flexibility is more important than commitment-based savings.

---

# ⚠️ EC2 Compute and Storage

EC2 compute and storage should be considered separately.

For an EBS-backed EC2 instance:

```text
EC2 Instance
    │
    ├── Compute
    │
    └── EBS Volume
```

Stopping the EC2 instance stops applicable instance compute charges, but resources such as EBS volumes can continue generating charges.

Therefore:

```text
Stopped EC2
    │
    ├── Compute Charge → Stopped
    │
    └── EBS Charge     → Can Continue
```

This is an important consideration when estimating EC2 cost.

---

# 🟢 Savings Plans

Savings Plans provide lower prices in exchange for committing to a consistent amount of eligible usage measured in:

```text
USD / Hour
```

for:

```text
1 Year

or

3 Years
```

Instead of simply asking:

```text
Which EC2 instance will I run?
```

the commitment is based on:

```text
How much eligible compute usage
will I consistently consume per hour?
```

---

# 🧠 Savings Plan Concept

```text
Predictable Compute Usage
          │
          ▼
Commit to $ / Hour
          │
          ▼
1 or 3 Years
          │
          ▼
Discounted Pricing
```

For EC2 workloads, two important Savings Plan types are:

```text
Savings Plans
     │
     ├── EC2 Instance Savings Plans
     │
     └── Compute Savings Plans
```

---

# 🔷 EC2 Instance Savings Plans

EC2 Instance Savings Plans provide discounts in exchange for committing to an EC2 instance family in a specific AWS Region.

Example:

```text
Region
ca-central-1

Instance Family
m7i
```

Within that commitment, the plan provides flexibility across characteristics such as:

* Instance size
* Operating system
* Tenancy

AWS states that EC2 Instance Savings Plans can provide savings of up to **72% compared with On-Demand pricing**.

---

# 🔄 Compute Savings Plans

Compute Savings Plans provide broader flexibility.

They can automatically apply to eligible usage across:

```text
Compute Savings Plan
        │
   ┌────┼─────────┐
   │    │         │
   ▼    ▼         ▼
 EC2  Fargate   Lambda
```

For EC2, Compute Savings Plans provide flexibility across:

* Instance family
* Instance size
* Region
* Operating system
* Tenancy

AWS states that Compute Savings Plans can provide savings of up to **66% compared with On-Demand pricing**.

---

# 📊 EC2 Instance vs Compute Savings Plans

| Feature                        | EC2 Instance Savings Plan  | Compute Savings Plan |
| ------------------------------ | -------------------------- | -------------------- |
| EC2                            | ✅                          | ✅                    |
| Fargate                        | ❌                          | ✅                    |
| Lambda                         | ❌                          | ✅                    |
| Change instance size           | ✅                          | ✅                    |
| Change operating system        | ✅                          | ✅                    |
| Change instance family         | ❌ outside committed family | ✅                    |
| Change Region                  | ❌ outside committed Region | ✅                    |
| Flexibility                    | Moderate                   | Higher               |
| Maximum advertised EC2 savings | Up to 72%                  | Up to 66%            |

The architecture trade-off is:

```text
EC2 Instance Savings Plan
        │
        ├── Higher potential savings
        └── More specific commitment


Compute Savings Plan
        │
        ├── Greater flexibility
        └── Slightly lower maximum discount
```

---

# 🔵 Reserved Instances

Reserved Instances provide a billing discount for matching EC2 usage in exchange for a commitment to a specific EC2 configuration for:

```text
1 Year

or

3 Years
```

An important concept is:

> A Reserved Instance is a billing benefit, not a separate type of EC2 virtual machine.

The discount automatically applies to eligible EC2 usage that matches the attributes of the Reserved Instance.

---

# 🧩 Reserved Instance Attributes

Reserved Instance configuration includes attributes such as:

```text
Instance Type

Platform

Scope

Tenancy

Term
```

Reserved Instances are available with different payment options:

```text
All Upfront

Partial Upfront

No Upfront
```

The appropriate option depends on the organization's financial and workload requirements.

---

# 🔷 Standard Reserved Instances

Standard Reserved Instances provide a higher discount than Convertible Reserved Instances but offer less flexibility.

Certain attributes can be modified.

However:

```text
Standard RI
     │
     ├── Can be modified in supported ways
     │
     └── Cannot be exchanged
```

Eligible Standard Reserved Instances can also be sold in the Reserved Instance Marketplace.

---

# 🔄 Convertible Reserved Instances

Convertible Reserved Instances provide more flexibility.

They can be exchanged during their term for another Convertible Reserved Instance with different attributes.

These can include:

```text
Instance Family

Instance Type

Platform

Scope

Tenancy
```

The trade-off is:

```text
Standard RI
   │
   └── Higher discount
       Less flexibility


Convertible RI
   │
   └── Lower discount
       Greater flexibility
```

---

# 📊 Standard vs Convertible Reserved Instances

| Feature                     | Standard RI              | Convertible RI |
| --------------------------- | ------------------------ | -------------- |
| Modify supported attributes | ✅                        | ✅              |
| Exchange                    | ❌                        | ✅              |
| Marketplace sale            | Eligible RIs can be sold | ❌              |
| Flexibility                 | Lower                    | Higher         |
| Discount potential          | Higher                   | Lower          |

---

# 🌎 Regional vs Zonal Reserved Instances

Reserved Instances can have different scopes.

### Regional RI

A Regional Reserved Instance provides the billing discount within the selected Region according to the matching rules.

```text
AWS Region
   │
   ├── AZ-A
   ├── AZ-B
   └── AZ-C
```

It does not provide a capacity reservation.

### Zonal RI

A Zonal Reserved Instance applies to a specific Availability Zone.

```text
AWS Region
    │
    └── AZ-A
         │
         └── Reserved Capacity
```

A Zonal Reserved Instance also provides a capacity reservation in the specified Availability Zone.

---

# 🟠 Spot Instances

Spot Instances allow you to use spare EC2 capacity at significant discounts compared with On-Demand pricing.

AWS states that Spot Instances can provide savings of:

```text
Up to 90%
```

compared with On-Demand prices.

The trade-off is interruption.

```text
Spare EC2 Capacity
        │
        ▼
Spot Instance
        │
        ▼
Lower Cost
        │
        ▼
Can Be Interrupted
```

---

# ⚠️ Spot Instance Interruptions

AWS can interrupt a Spot Instance when EC2 needs the capacity back.

When AWS interrupts a Spot Instance, it generally provides a:

```text
2-Minute
Interruption Notice
```

Applications using Spot should therefore be designed to handle interruptions.

AWS also provides **EC2 instance rebalance recommendations**, which can warn that a Spot Instance is at elevated risk of interruption before the two-minute interruption notice.

---

# 🎯 Suitable Spot Workloads

Spot is well suited to workloads that are:

* Fault tolerant
* Flexible
* Distributed
* Stateless
* Restartable
* Able to tolerate interruption

Examples include:

```text
Batch Processing

Data Processing

Media Processing

CI/CD Workers

Distributed Computing

Stateless Application Capacity
```

The architecture should not depend on a particular Spot Instance remaining available.

---

# 🏗️ Designing for Spot

Instead of:

```text
Application
     │
     ▼
One Spot Instance
     │
     ▼
Instance Interrupted
     │
     ▼
Application Down
```

design for replaceable capacity:

```text
             Application
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Instance  Instance  Instance
        │         │         │
        └──── Replaceable ───┘
```

AWS recommends designing Spot workloads to be fault tolerant and using mechanisms such as:

* EC2 Auto Scaling
* EC2 Fleet
* Multiple instance types
* Multiple Availability Zones
* Capacity Rebalancing

---

# 🧠 Spot Pricing

A common older explanation of Spot Instances focuses heavily on bidding against a changing Spot price.

Current AWS behavior is simpler.

By default:

```text
Spot Maximum Price
       │
       ▼
On-Demand Price
```

You can specify a maximum Spot price, but AWS does not recommend doing so.

The key architecture consideration should therefore be:

```text
Can my workload tolerate
Spot capacity interruption?
```

rather than attempting to predict or manage Spot bidding.

---

# 🖥️ Dedicated Hosts

A Dedicated Host is a physical EC2 server dedicated to your use.

Conceptually:

```text
Physical EC2 Host
┌───────────────────────────┐
│                           │
│     Your Workloads        │
│                           │
│   EC2    EC2    EC2       │
│                           │
└───────────────────────────┘
```

Dedicated Hosts provide visibility and control over the physical server that is not available with normal shared-tenancy EC2 instances.

---

# 🎯 Dedicated Host Use Cases

Dedicated Hosts can be useful for:

* Compliance requirements
* Server-bound software licenses
* Per-socket licensing
* Per-core licensing
* Per-VM licensing
* Host-level visibility
* Instance placement control

Example:

```text
Existing Enterprise License
           │
           ▼
License Bound to
Physical Cores / Sockets
           │
           ▼
Dedicated Host
```

Dedicated Hosts can help organizations use eligible existing software licenses while moving workloads to AWS.

---

# 💰 Dedicated Host Billing

With an On-Demand Dedicated Host:

```text
Allocate Physical Host
          │
          ▼
Pay for Active Host
```

AWS bills the active Dedicated Host regardless of how many instances are running on it.

This creates an important utilization consideration:

```text
Dedicated Host
      │
      ├── Poor Utilization → Higher effective cost
      │
      └── Good Utilization → Better cost efficiency
```

---

# 🟣 Dedicated Instances

Dedicated Instances are EC2 instances that run on hardware dedicated to a single customer.

They provide hardware isolation from other AWS customers.

However, they do not provide the same host-level visibility and placement control as Dedicated Hosts.

---

# 📊 Dedicated Host vs Dedicated Instance

| Capability                 | Dedicated Host | Dedicated Instance |
| -------------------------- | -------------- | ------------------ |
| Single-tenant hardware     | ✅              | ✅                  |
| Physical host visibility   | ✅              | ❌                  |
| Socket/core visibility     | ✅              | ❌                  |
| Instance placement control | ✅              | ❌                  |
| Server-bound licensing     | Better suited  | Limited            |
| Compliance isolation       | ✅              | ✅                  |

The decision can be simplified as:

```text
Need dedicated hardware?
        │
        ▼
       Yes
        │
        ▼
Need physical host visibility
or placement control?
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
Dedicated   Dedicated
Host        Instance
```

---

# 🟡 On-Demand Capacity Reservations

On-Demand Capacity Reservations allow you to reserve EC2 compute capacity in a specific Availability Zone.

This addresses a different problem from pricing discounts.

Consider:

```text
Application Event
       │
       ▼
Need 100 EC2 Instances
       │
       ▼
Must Be Available
in AZ-A
```

A Capacity Reservation can reserve matching EC2 capacity in that Availability Zone.

---

# 🧠 Capacity vs Cost

This distinction is critical:

```text
Savings Plan / RI
       │
       ▼
Primarily Cost Optimization


Capacity Reservation
       │
       ▼
Capacity Availability
```

Do not assume that purchasing a pricing discount automatically solves every capacity requirement.

---

# 🎯 Capacity Reservation Use Cases

Capacity Reservations can be useful when:

* Capacity must be available in a specific AZ
* A workload has strict scaling requirements
* A migration requires guaranteed EC2 capacity
* A business event requires predictable capacity
* Disaster recovery capacity must be available when required

Example:

```text
Expected Business Event
         │
         ▼
Large Capacity Requirement
         │
         ▼
Specific Availability Zone
         │
         ▼
Capacity Reservation
```

---

# 📊 Quick Comparison

| Requirement                                | Purchasing Option to Consider |
| ------------------------------------------ | ----------------------------- |
| Short-term unpredictable workload          | On-Demand                     |
| No long-term commitment                    | On-Demand                     |
| Predictable compute spend                  | Savings Plans                 |
| Predictable EC2 configuration              | Reserved Instances            |
| Interruptible workload                     | Spot                          |
| Significant savings with flexible workload | Spot                          |
| Dedicated physical host                    | Dedicated Host                |
| Host-level licensing visibility            | Dedicated Host                |
| Single-tenant hardware                     | Dedicated Instance            |
| Guaranteed EC2 capacity in an AZ           | Capacity Reservation          |
| Flexibility across EC2, Fargate and Lambda | Compute Savings Plan          |
| EC2 family commitment within a Region      | EC2 Instance Savings Plan     |

---

# 🧠 Choosing the Right Purchasing Option

Start with the workload requirements.

```text
Can the workload tolerate interruption?
              │
        ┌─────┴─────┐
        │           │
       Yes          No
        │           │
        ▼           ▼
   Consider Spot   Is usage predictable?
                         │
                   ┌─────┴─────┐
                   │           │
                  Yes          No
                   │           │
                   ▼           ▼
              Savings Plan   On-Demand
                 or RI
```

Then evaluate additional requirements:

```text
Need guaranteed AZ capacity?
          │
          ▼
Capacity Reservation


Need physical host control?
          │
          ▼
Dedicated Host


Need single-tenant hardware
without host-level control?
          │
          ▼
Dedicated Instance
```

---

# 🏗️ Architecture Examples

### Scenario 1: Development Environment

Requirements:

```text
Short Term
Unpredictable Usage
No Long-Term Commitment
```

Consider:

```text
On-Demand
```

---

### Scenario 2: Stable Production Application

Requirements:

```text
Runs 24 × 7
Predictable Usage
Long-Term Workload
```

Consider:

```text
Savings Plans

or

Reserved Instances
```

depending on the required flexibility.

---

### Scenario 3: Batch Processing

Requirements:

```text
Fault Tolerant
Restartable
Cost Sensitive
```

Consider:

```text
Spot Instances
```

---

### Scenario 4: Enterprise Licensed Software

Requirements:

```text
Existing License
      │
      ▼
Physical Socket/Core
Licensing Requirement
```

Consider:

```text
Dedicated Host
```

---

### Scenario 5: Hardware Isolation

Requirements:

```text
Single-Tenant Hardware

No Requirement for
Host-Level Control
```

Consider:

```text
Dedicated Instances
```

---

### Scenario 6: Guaranteed Capacity

Requirements:

```text
Specific AZ
      │
      ▼
Capacity Must Be Available
```

Consider:

```text
On-Demand Capacity Reservation
```

---

# ✅ Best Practices

* Start with workload requirements before selecting a purchasing option.
* Separate **cost optimization** from **capacity assurance**.
* Use On-Demand when workload requirements are unpredictable or short-lived.
* Analyze historical usage before making long-term commitments.
* Consider Savings Plans when usage is predictable but flexibility is important.
* Use Spot only for workloads designed to tolerate interruptions.
* Diversify Spot capacity across instance types and Availability Zones where appropriate.
* Use Auto Scaling or EC2 Fleet for resilient Spot architectures.
* Use Dedicated Hosts only when host-level requirements justify the additional cost and operational responsibility.
* Use Capacity Reservations when capacity availability in a specific AZ is a business requirement.
* Monitor utilization after purchasing commitments to avoid paying for unused commitments.
* Review current AWS pricing and purchasing documentation before making production financial decisions.

---

# ❓ Interview Questions

### Q1. What is an On-Demand Instance?

**Answer**

An On-Demand Instance allows you to consume EC2 compute capacity without a long-term commitment and pay for the instance while it is running according to the applicable EC2 billing rules.

---

### Q2. When should On-Demand Instances be considered?

**Answer**

For short-term, irregular, unpredictable, or non-interruptible workloads where a long-term commitment is not appropriate.

---

### Q3. What is a Savings Plan?

**Answer**

A Savings Plan provides discounted eligible compute pricing in exchange for committing to a consistent amount of usage measured in USD per hour for one or three years.

---

### Q4. What is the difference between an EC2 Instance Savings Plan and Compute Savings Plan?

**Answer**

EC2 Instance Savings Plans commit to an instance family within a specific Region and can provide higher discounts.

Compute Savings Plans provide broader flexibility across EC2 instance families, Regions, operating systems and tenancy, and can also apply to Fargate and Lambda.

---

### Q5. What is a Reserved Instance?

**Answer**

A Reserved Instance provides a billing discount for eligible matching EC2 usage in exchange for a one-year or three-year commitment to an EC2 configuration.

---

### Q6. What is the difference between Standard and Convertible Reserved Instances?

**Answer**

Standard Reserved Instances provide higher potential discounts but cannot be exchanged.

Convertible Reserved Instances provide greater flexibility because they can be exchanged for another Convertible Reserved Instance with different attributes.

---

### Q7. What is a Spot Instance?

**Answer**

A Spot Instance uses spare EC2 capacity at significantly discounted prices but can be interrupted when AWS needs the capacity back.

---

### Q8. What type of workload is suitable for Spot?

**Answer**

Fault-tolerant, flexible, distributed, stateless, or restartable workloads that can tolerate interruption.

---

### Q9. How much warning does AWS provide before a Spot interruption?

**Answer**

AWS generally provides a two-minute Spot Instance interruption notice before interrupting the instance.

A rebalance recommendation can sometimes provide an earlier signal that an instance is at elevated risk of interruption.

---

### Q10. What is a Dedicated Host?

**Answer**

A Dedicated Host is a physical EC2 server dedicated to your use that provides host-level visibility and placement control.

---

### Q11. What is the difference between a Dedicated Host and Dedicated Instance?

**Answer**

Both provide single-tenant hardware.

Dedicated Hosts additionally provide visibility and control over the physical host, which can be important for server-bound licensing and placement requirements.

---

### Q12. What is an On-Demand Capacity Reservation?

**Answer**

An On-Demand Capacity Reservation reserves EC2 compute capacity for matching instances in a specific Availability Zone.

---

### Q13. Does a Capacity Reservation automatically provide a pricing discount?

**Answer**

No.

Capacity Reservations primarily address capacity availability. Pricing discounts and capacity guarantees should be evaluated separately.

---

### Q14. What should be the first consideration when choosing an EC2 purchasing option?

**Answer**

Understand the workload:

```text
How long will it run?

Is usage predictable?

Can it be interrupted?

Does it require guaranteed capacity?

Does it require dedicated hardware?

How much flexibility is required?
```

The purchasing option should follow from those requirements.

---

# 💡 Key Takeaways

* EC2 provides multiple purchasing options because workloads have different cost, flexibility, capacity, and isolation requirements.
* On-Demand provides flexibility without long-term commitment.
* Savings Plans provide discounts based on a consistent usage commitment.
* EC2 Instance Savings Plans provide higher potential discounts with a more specific EC2 commitment.
* Compute Savings Plans provide broader flexibility across EC2, Fargate, and Lambda.
* Reserved Instances provide discounts for matching EC2 usage based on a configuration commitment.
* Spot uses spare EC2 capacity and can provide significant savings, but workloads must tolerate interruption.
* Spot architecture should focus on fault tolerance rather than relying on a single instance.
* Dedicated Hosts provide dedicated physical hardware plus host-level visibility and control.
* Dedicated Instances provide single-tenant hardware without the same host-level control.
* Capacity Reservations address guaranteed EC2 capacity in a specific Availability Zone.
* Cost optimization and capacity assurance are different architecture decisions.
* The correct purchasing option starts with understanding the workload, not simply finding the largest discount.

---

# 📚 Related Topics

* Amazon EC2
* EC2 Instance Types
* Amazon EBS
* EC2 Auto Scaling
* Elastic Load Balancing
* EC2 Spot Fleet
* EC2 Fleet
* AWS Cost Explorer
* AWS Compute Optimizer
* AWS Well-Architected Framework
* Cost Optimization Pillar

---

# 📖 References

* AWS Documentation: Amazon EC2 Billing and Purchasing Options
* AWS Documentation: Purchasing On-Demand Instances
* AWS Documentation: Reserved Instances for Amazon EC2
* AWS Documentation: Amazon EC2 Spot Instances
* AWS Documentation: Amazon EC2 Dedicated Hosts
* AWS Documentation: Amazon EC2 Dedicated Instances
* AWS Documentation: On-Demand Capacity Reservations
* AWS Documentation: Savings Plans
