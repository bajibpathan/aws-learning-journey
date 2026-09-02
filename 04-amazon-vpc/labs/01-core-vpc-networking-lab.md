# 🧪 Lab 01: Core Amazon VPC Networking

> Design, build, validate, break, and troubleshoot a highly available Amazon VPC network using the core AWS networking concepts covered in the VPC learning series.

---

# 📖 Lab Overview

Reading about VPCs, subnets, route tables, Internet Gateways, NAT Gateways, Security Groups, and Network ACLs helps build foundational knowledge.

This lab focuses on applying those concepts together.

The objective is not simply to create AWS resources.

The objective is to understand:

* Why each networking component exists
* How packets move through the architecture
* How public and private subnets differ
* How routing decisions are made
* How private workloads access the Internet
* How Security Groups and NACLs affect traffic
* How Availability Zone failures affect network connectivity
* How VPC Flow Logs help troubleshoot network problems

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

* Design a VPC CIDR range
* Divide a VPC into multiple subnets
* Distribute workloads across Availability Zones
* Configure public and private subnets
* Configure an Internet Gateway
* Configure public and private route tables
* Configure NAT Gateway connectivity
* Explain the VPC `local` route
* Configure Security Groups
* Understand subnet-level controls using NACLs
* Enable VPC Flow Logs
* Validate network connectivity
* Troubleshoot routing and security problems
* Explain the availability implications of NAT Gateway placement

---

# 🏢 Business Scenario

A company is deploying a new application called:

**Internal Operations Portal**

The application will run in AWS.

The network must provide isolation between Internet-facing infrastructure and private application servers.

### Requirements

| Requirement          | Description                                                                            |
| -------------------- | -------------------------------------------------------------------------------------- |
| AWS Region           | `ca-central-1`                                                                         |
| VPC                  | Custom VPC                                                                             |
| Availability         | 2 Availability Zones                                                                   |
| Public Tier          | Must support Internet-facing resources                                                 |
| Private Tier         | Application servers must not be directly accessible from the Internet                  |
| Outbound Access      | Private application servers require Internet access for patches and software downloads |
| Network Segmentation | Public and private workloads must use separate subnets                                 |
| Security             | Least-privilege network access                                                         |
| Observability        | Network traffic must be available for troubleshooting                                  |

---

# 🏗️ Network Design

## VPC

```text
VPC CIDR
10.0.0.0/16
```

This provides the address range:

```text
10.0.0.0
    ↓
10.0.255.255
```

---

# 🌐 Availability Zones

The architecture uses two Availability Zones:

```text
ca-central-1a

ca-central-1b
```

Using multiple Availability Zones allows workloads to be distributed across independent AWS infrastructure.

---

# 📦 Subnet Design

| Subnet           | Availability Zone | CIDR          | Type    |
| ---------------- | ----------------- | ------------- | ------- |
| Public Subnet A  | ca-central-1a     | `10.0.1.0/24` | Public  |
| Public Subnet B  | ca-central-1b     | `10.0.2.0/24` | Public  |
| Private Subnet A | ca-central-1a     | `10.0.3.0/24` | Private |
| Private Subnet B | ca-central-1b     | `10.0.4.0/24` | Private |

Architecture:

```text
VPC: 10.0.0.0/16

├── ca-central-1a
│
│   ├── Public-A
│   │   10.0.1.0/24
│   │
│   └── Private-A
│       10.0.3.0/24
│
└── ca-central-1b

    ├── Public-B
    │   10.0.2.0/24
    │
    └── Private-B
        10.0.4.0/24
```

The subnet CIDRs:

* Are contained within the VPC CIDR
* Do not overlap
* Provide separate address spaces for public and private workloads

---

# 🌍 Internet Gateway

Create an Internet Gateway and attach it to the VPC.

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
VPC
10.0.0.0/16
```

An Internet Gateway is attached to the **VPC**, not directly to individual subnets.

Whether a subnet can use the Internet Gateway depends on its route table.

---

# 🛣️ Public Route Table

Create a route table for the public subnets.

### Routes

```text
Destination        Target

10.0.0.0/16        local
0.0.0.0/0          Internet Gateway
```

Associate it with:

```text
Public Subnet A
Public Subnet B
```

The route:

```text
0.0.0.0/0 → Internet Gateway
```

is what makes these subnets public from a routing perspective.

Simply naming a subnet "public" does not make it public.

---

# 🔄 NAT Gateway

Private application servers require outbound IPv4 Internet connectivity for activities such as:

* Operating system updates
* Package downloads
* Software installation
* Accessing external APIs

However, these servers should not be directly reachable from the Internet.

A NAT Gateway provides this capability.

For the initial lab architecture, deploy **one NAT Gateway** in:

```text
Public Subnet A
ca-central-1a
```

The NAT Gateway requires an Elastic IP address.

Architecture:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Subnet A
    │
    ▼
NAT Gateway
    │
    ▼
Private Subnets
```

---

# 🔐 Private Route Table

Create a private route table.

### Routes

```text
Destination        Target

10.0.0.0/16        local
0.0.0.0/0          NAT Gateway
```

Associate it with:

```text
Private Subnet A
Private Subnet B
```

The two routes serve different purposes.

### Local Route

```text
10.0.0.0/16 → local
```

Allows communication between resources inside the VPC, subject to applicable security controls.

### Default Route

```text
0.0.0.0/0 → NAT Gateway
```

Sends Internet-bound IPv4 traffic from the private subnets to the NAT Gateway.

---

# 🔎 Understanding the Traffic Path

When an EC2 instance in Private Subnet B accesses the Internet:

```text
Private EC2
    │
    ▼
Private Route Table
    │
    │ 0.0.0.0/0
    ▼
NAT Gateway
    │
    ▼
Public Route Table
    │
    ▼
Internet Gateway
    │
    ▼
Internet
```

The private EC2 instance does not require a public IPv4 address.

The connection is initiated by the private instance and uses the NAT Gateway for outbound IPv4 connectivity.

---

# 🖥️ EC2 Test Instances

Launch test EC2 instances in the private subnets.

Example:

```text
Private EC2-A
Subnet: Private-A
Public IPv4: Disabled

Private EC2-B
Subnet: Private-B
Public IPv4: Disabled
```

These instances should not be directly reachable from the Internet.

For administrative access, prefer AWS Systems Manager Session Manager rather than exposing SSH to the Internet.

---

# 🔒 Security Groups

Create Security Groups according to the required traffic flow.

For example:

```text
Internet
   │
   ▼
Public Tier SG
   │
   ▼
Application SG
   │
   ▼
Private EC2
```

Apply least privilege.

Avoid opening unnecessary ports such as:

```text
0.0.0.0/0 → SSH 22
```

unless specifically required for a controlled exercise.

Security Groups are:

* Stateful
* Applied at ENI/resource level
* Allow-only
* Capable of referencing other Security Groups

---

# 🛡️ Network ACLs

NACLs provide an additional subnet-level security layer.

Remember:

```text
Security Group
    ↓
Resource / ENI level
    ↓
Stateful
    ↓
Allow rules
```

Compared with:

```text
Network ACL
    ↓
Subnet level
    ↓
Stateless
    ↓
Allow + Deny rules
```

During the troubleshooting section of this lab, NACL rules will intentionally be modified to observe how traffic is affected.

---

# 📊 VPC Flow Logs

Enable Flow Logs at the:

```text
VPC level
```

For this lab, VPC-level logging provides visibility across the complete network rather than focusing on a single ENI.

Capture:

```text
ALL
```

traffic so both accepted and rejected flows can be investigated.

Depending on the exercise, logs can be delivered to:

* CloudWatch Logs
* Amazon S3

Flow Logs will later be used to investigate:

```text
ACCEPT
```

and:

```text
REJECT
```

traffic.

Remember that VPC Flow Logs capture **traffic metadata**, not packet contents.

---

# 🏗️ Initial Lab Architecture

```text
                         Internet
                            │
                            ▼
                    Internet Gateway
                            │
              ┌─────────────┴─────────────┐
              │                           │
        ca-central-1a               ca-central-1b
              │                           │
        Public Subnet A             Public Subnet B
         10.0.1.0/24                10.0.2.0/24
              │
              ▼
         NAT Gateway
              │
        ┌─────┴─────────────────────┐
        │                           │
        ▼                           ▼
 Private Subnet A            Private Subnet B
  10.0.3.0/24                 10.0.4.0/24
        │                           │
        ▼                           ▼
     EC2-A                       EC2-B
```

---

# 🧪 Phase 1: Build the Network

Create the following resources:

* Custom VPC
* Two public subnets
* Two private subnets
* Internet Gateway
* Public route table
* Private route table
* NAT Gateway
* Elastic IP for NAT Gateway
* Security Groups
* Test EC2 instances
* VPC Flow Logs

Do not consider the phase complete simply because all resources exist.

Validate the architecture.

---

# ✅ Phase 2: Connectivity Validation

Verify the following.

### Test 1: VPC Internal Connectivity

Confirm that permitted resources can communicate using private IP addresses.

Expected path:

```text
EC2-A
  │
  │ Private IP
  ▼
VPC local route
  │
  ▼
EC2-B
```

---

### Test 2: Private EC2 Internet Access

From a private EC2 instance, test outbound connectivity.

For example:

```bash
curl https://example.com
```

The expected path is:

```text
Private EC2
    │
    ▼
NAT Gateway
    │
    ▼
Internet Gateway
    │
    ▼
Internet
```

---

### Test 3: Direct Internet Access

Verify that the private EC2 instance does **not** have a public IPv4 address and cannot be directly reached from the Internet.

This confirms that outbound Internet connectivity does not mean the resource itself is Internet-facing.

---

# 💥 Phase 3: Break the Network

A major objective of this lab is troubleshooting.

Do not stop after getting a successful connection.

Intentionally introduce failures.

---

## Failure Scenario 1: Remove the NAT Route

Remove:

```text
0.0.0.0/0 → NAT Gateway
```

from the private route table.

Then test Internet connectivity from the private EC2 instance.

### Expected Result

Internet connectivity should fail.

Internal VPC communication can still work because:

```text
10.0.0.0/16 → local
```

still exists.

### Question

Why can the instance communicate internally but not reach the Internet?

---

## Failure Scenario 2: Security Group Misconfiguration

Remove or modify a required Security Group rule.

Test the application connection again.

Investigate:

* Source
* Destination
* Protocol
* Port
* Security Group rules

Restore the correct configuration after identifying the problem.

---

## Failure Scenario 3: NACL Deny

Create a controlled NACL rule that rejects selected traffic.

Generate traffic and observe the failure.

Then inspect VPC Flow Logs.

Look for:

```text
REJECT
```

records.

Determine whether the failure originated from:

```text
Routing

Security Group

or

Network ACL
```

---

# 🔍 Troubleshooting Method

When connectivity fails, avoid randomly changing AWS resources.

Use a structured troubleshooting process.

```text
Can Source Reach Destination?
            │
            ▼
      Check IP Address
            │
            ▼
      Check Route Table
            │
            ▼
    Check Security Group
            │
            ▼
        Check NACL
            │
            ▼
     Check Flow Logs
            │
            ▼
  Check Application/Service
```

Ask:

1. What is the source?
2. What is the destination?
3. Which subnet contains the source?
4. Which route table is associated with that subnet?
5. Which route matches the destination?
6. What is the route target?
7. Does the Security Group permit the traffic?
8. Does the NACL permit traffic in both directions?
9. What do VPC Flow Logs show?
10. Is the application actually listening on the expected port?

---

# ⚠️ Architecture Weakness

The initial architecture deliberately uses only one NAT Gateway:

```text
               AZ-A                       AZ-B
                 │                          │
             Public-A                   Public-B
                 │
              NAT-A
                 │
          ┌──────┴───────────────┐
          │                      │
      Private-A              Private-B
```

Both private subnets depend on:

```text
NAT-A
```

If AZ-A becomes unavailable, Private-B may still be running in AZ-B, but it loses its NAT path for outbound IPv4 Internet access.

This introduces an availability dependency.

---

# 🏗️ Production Improvement

For a highly available production architecture, deploy a NAT Gateway in each Availability Zone.

```text
              AZ-A                       AZ-B

           Public-A                   Public-B
              │                          │
            NAT-A                      NAT-B
              │                          │
          Private-A                  Private-B
```

Each private subnet should use the NAT Gateway in its own Availability Zone.

---

# 🛣️ Production Route Table Design

This requires separate private route tables.

### Private Route Table A

Associated with:

```text
Private-A
```

Routes:

```text
10.0.0.0/16 → local
0.0.0.0/0   → NAT-A
```

### Private Route Table B

Associated with:

```text
Private-B
```

Routes:

```text
10.0.0.0/16 → local
0.0.0.0/0   → NAT-B
```

This provides:

* Independent routing control
* Better Availability Zone resilience
* AZ-local NAT routing
* Reduced unnecessary cross-AZ traffic

---

# 💡 Route Table Design Principle

A useful architecture rule is:

> Share route tables when the routing policy is identical. Separate route tables when routing behavior needs to differ.

For example:

```text
Public-A ─┐
          ├── Public Route Table
Public-B ─┘
```

because both use:

```text
0.0.0.0/0 → IGW
```

But:

```text
Private-A → Private Route Table A → NAT-A

Private-B → Private Route Table B → NAT-B
```

because their default route targets are different.

---

# 🧠 Architecture Questions

After completing the lab, you should be able to answer these without looking at documentation.

### Q1. What makes a subnet public?

**Answer**

A subnet is public when its associated route table contains a route to an Internet Gateway for Internet-bound traffic.

For IPv4:

```text
0.0.0.0/0 → IGW
```

---

### Q2. Does attaching an Internet Gateway automatically make every subnet public?

**Answer**

No.

The subnet's route table must contain a route to the Internet Gateway.

---

### Q3. Why does a NAT Gateway need to be in a public subnet?

**Answer**

A public NAT Gateway needs connectivity to the Internet Gateway so it can provide outbound Internet access for private resources.

---

### Q4. Why don't private EC2 instances require public IPv4 addresses?

**Answer**

They can initiate outbound IPv4 Internet connections through the NAT Gateway while remaining reachable only through their private addressing and permitted network paths.

---

### Q5. What is the purpose of the `local` route?

**Answer**

The local route enables routing between resources within the VPC CIDR.

Example:

```text
10.0.0.0/16 → local
```

---

### Q6. What happens if the NAT Gateway fails?

**Answer**

Private resources using that NAT Gateway lose the outbound IPv4 connectivity provided through it.

Internal VPC communication can continue if the relevant resources and network controls remain available.

---

### Q7. Why use one NAT Gateway per AZ in a highly available architecture?

**Answer**

It prevents private workloads in multiple AZs from depending on a NAT Gateway located in only one AZ and allows each AZ to use its local NAT path.

---

### Q8. Why would we create separate private route tables?

**Answer**

Each private subnet can use a different default route target.

For example:

```text
Private-A → NAT-A

Private-B → NAT-B
```

---

### Q9. Security Group vs NACL?

| Security Group                                                   | NACL                                        |
| ---------------------------------------------------------------- | ------------------------------------------- |
| Resource/ENI level                                               | Subnet level                                |
| Stateful                                                         | Stateless                                   |
| Allow rules                                                      | Allow and Deny rules                        |
| Return traffic automatically permitted for an allowed connection | Return traffic must be explicitly permitted |

---

### Q10. Why enable VPC Flow Logs?

**Answer**

VPC Flow Logs provide metadata about network traffic and can help identify accepted or rejected traffic when troubleshooting connectivity and security issues.

---

# 🏁 Completion Criteria

Do not mark this lab complete until you can:

* [ ] Explain the VPC CIDR
* [ ] Explain why each subnet CIDR was selected
* [ ] Explain why subnets are distributed across two AZs
* [ ] Explain what makes a subnet public
* [ ] Explain the purpose of the Internet Gateway
* [ ] Explain the `local` route
* [ ] Explain the public default route
* [ ] Explain the private default route
* [ ] Explain why the NAT Gateway is in a public subnet
* [ ] Explain why private EC2 instances have no public IP
* [ ] Explain Security Group behavior
* [ ] Explain NACL behavior
* [ ] Generate and investigate ACCEPT/REJECT traffic
* [ ] Use VPC Flow Logs during troubleshooting
* [ ] Diagnose a deliberately broken route
* [ ] Diagnose a deliberately broken security rule
* [ ] Explain the failure impact of using one NAT Gateway
* [ ] Explain how one NAT Gateway per AZ improves availability
* [ ] Explain why separate private route tables are useful

---

# 💰 Cost Awareness

Some resources in this lab can incur charges.

Pay particular attention to:

* NAT Gateway hourly charges
* NAT Gateway data processing
* EC2 instances
* Elastic IPv4 addresses
* CloudWatch Logs
* Data transfer

Delete lab resources when they are no longer required.

NAT Gateway should be one of the first resources checked during cleanup because leaving it running unnecessarily can continue generating charges.

---

# 🧹 Cleanup Checklist

After completing the lab:

* [ ] Terminate test EC2 instances
* [ ] Delete NAT Gateway
* [ ] Release unused Elastic IP
* [ ] Delete VPC Flow Logs if no longer required
* [ ] Delete associated CloudWatch Log Groups if appropriate
* [ ] Delete custom route tables
* [ ] Delete custom NACLs if created
* [ ] Delete Security Groups
* [ ] Detach and delete Internet Gateway
* [ ] Delete subnets
* [ ] Delete VPC

Always verify the AWS console after cleanup to ensure no chargeable resources remain.

> The goal of this lab is not to memorize AWS Console steps. The goal is to understand how traffic moves through an Amazon VPC and be able to diagnose why connectivity succeeds or fails.
