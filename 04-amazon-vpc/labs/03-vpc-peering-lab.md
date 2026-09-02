# 🧪 Lab 03: VPC-to-VPC Connectivity with VPC Peering

> Build two independent Amazon VPCs, connect them using VPC Peering, validate private communication, deliberately break the network, and explore why large AWS environments often require a transit architecture.

---

# 📖 Lab Overview

The previous labs focused primarily on networking inside a single VPC.

Real AWS environments frequently contain multiple VPCs.

Examples include:

```text
Production VPC

Development VPC

Shared Services VPC

Security VPC

Data VPC
```

These VPCs may need private communication.

One option is:

```text
VPC Peering
```

This lab introduces VPC Peering by building multiple VPCs and configuring private connectivity between them.

The lab will also demonstrate one of the most important limitations of VPC Peering:

```text
No transitive routing
```

That limitation will lead naturally into understanding why AWS Transit Gateway exists.

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

* Design multiple non-overlapping VPC CIDRs
* Create a VPC Peering connection
* Explain requester and accepter VPCs
* Configure routes on both sides
* Configure Security Groups for cross-VPC traffic
* Test communication using private IP addresses
* Troubleshoot VPC Peering connectivity
* Explain why overlapping CIDRs are a problem
* Explain why VPC Peering is non-transitive
* Understand the operational problem with full-mesh peering
* Explain when Transit Gateway becomes useful

---

# 🏢 Business Scenario

A company currently operates two environments:

```text
Application VPC
Shared Services VPC
```

The Application VPC contains application servers.

The Shared Services VPC contains internal services that applications need to access.

Requirements:

* Traffic must remain private
* VPC CIDRs must not overlap
* No public Internet path should be required for VPC-to-VPC communication
* Only required traffic should be permitted
* Connectivity should be validated using private IP addresses

---

# 🏗️ VPC Design

## VPC A: Application

```text
Name:
Application-VPC

CIDR:
10.0.0.0/16
```

Example subnet:

```text
10.0.3.0/24
```

---

## VPC B: Shared Services

```text
Name:
Shared-Services-VPC

CIDR:
10.20.0.0/16
```

Example subnet:

```text
10.20.1.0/24
```

The CIDRs do not overlap:

```text
Application VPC
10.0.0.0/16

Shared Services VPC
10.20.0.0/16
```

---

# 🔗 VPC Peering Architecture

Create a VPC Peering connection between the two VPCs.

```text
Application VPC
10.0.0.0/16
       │
       │
       ▼
VPC Peering Connection
     pcx-xxxxx
       │
       │
       ▼
Shared Services VPC
10.20.0.0/16
```

The VPC that initiates the connection is called the:

```text
Requester
```

The other VPC is the:

```text
Accepter
```

The peering connection must become:

```text
Active
```

before it can carry traffic.

---

# 🛣️ Routing Requirements

Creating the VPC Peering connection alone does not establish complete connectivity.

Routes must be configured.

---

## Application VPC Route

Add:

```text
Destination        Target

10.20.0.0/16       pcx-xxxxx
```

This tells Application VPC:

> Send traffic destined for Shared Services VPC through the VPC Peering connection.

---

## Shared Services VPC Route

Add:

```text
Destination        Target

10.0.0.0/16        pcx-xxxxx
```

This provides the return path.

---

# 🧠 Bidirectional Routing

Both sides need appropriate routes.

```text
Application EC2
10.0.3.10
     │
     │
     ▼
10.20.0.0/16
     │
     ▼
VPC Peering
     │
     ▼
Shared Service
10.20.1.10
```

Return traffic:

```text
Shared Service
10.20.1.10
     │
     ▼
10.0.0.0/16
     │
     ▼
VPC Peering
     │
     ▼
Application EC2
10.0.3.10
```

A route on only one side is not enough.

---

# 🔒 Security Groups

Routing determines where traffic can travel.

Security Groups determine whether permitted traffic can reach the resource.

For example, suppose Shared Services runs an internal application on:

```text
TCP 8080
```

The Shared Services Security Group could permit TCP 8080 from the required Application VPC source range or use supported Security Group referencing where appropriate for the peering scenario.

The goal is:

```text
Application Workload
        │
        │ TCP 8080
        ▼
Shared Service
```

Do not allow unnecessary ports.

---

# 🧪 Phase 1: Build VPC B

Retain the Application VPC from the previous labs.

Create:

```text
Shared-Services-VPC
10.20.0.0/16
```

Create at least one subnet:

```text
10.20.1.0/24
```

Launch a test EC2 instance or simple internal service.

The instance should use a private IPv4 address.

---

# 🧪 Phase 2: Establish the Baseline

Before creating VPC Peering, test connectivity from VPC A to the private address in VPC B.

Expected:

```text
FAIL
```

Why?

The VPCs are isolated networks and no connectivity path currently exists between them.

---

# 🧪 Phase 3: Create VPC Peering

Create:

```text
Application-VPC
        │
        ▼
VPC Peering
        │
        ▼
Shared-Services-VPC
```

Accept the peering request if required.

Confirm that the peering connection becomes active.

Do not test yet.

First inspect the route tables.

Ask:

> Does creating the peering connection automatically tell the subnets to use it?

The answer is:

```text
No
```

We still need routing.

---

# 🧪 Phase 4: Configure Routes

Application VPC:

```text
10.20.0.0/16 → pcx-xxxxx
```

Shared Services VPC:

```text
10.0.0.0/16 → pcx-xxxxx
```

Verify that the route is added to the route tables associated with the subnets containing the test workloads.

---

# 🧪 Phase 5: Configure Security

Allow only the required traffic.

For example:

```text
Protocol: TCP
Port:     8080
Source:   Required application network/security boundary
```

Do not simply allow:

```text
All Traffic
0.0.0.0/0
```

to make the test work.

The purpose of the lab is to understand the required network path.

---

# 🧪 Phase 6: Test Private Connectivity

From the Application EC2 instance, connect to the Shared Services EC2 private address.

For example:

```bash
curl http://10.20.1.10:8080
```

Expected:

```text
SUCCESS
```

Traffic path:

```text
Application EC2
10.0.3.10
     │
     ▼
Application Route Table
     │
10.20.0.0/16
     │
     ▼
VPC Peering
     │
     ▼
Shared Services Route Table
     │
     ▼
Shared Service EC2
10.20.1.10
```

No public IP addresses are required for this VPC-to-VPC communication.

---

# 💥 Phase 7: Break the Route

Remove:

```text
10.20.0.0/16 → pcx-xxxxx
```

from the Application VPC route table.

Test again.

Expected:

```text
FAIL
```

Restore the route.

Now remove the return route:

```text
10.0.0.0/16 → pcx-xxxxx
```

from Shared Services VPC.

Test again.

This demonstrates why network communication requires a valid forward and return path.

---

# 💥 Phase 8: Break Security

Restore routing.

Now remove the required Security Group rule from the Shared Services EC2 instance.

Test again.

Expected:

```text
FAIL
```

Ask yourself:

> Routing exists, so why does connectivity still fail?

Because:

```text
Routing
   ≠
Security Authorization
```

Both must be correct.

---

# 🔍 Troubleshooting VPC Peering

When VPC Peering connectivity fails, check:

```text
Peering Connection State
          │
          ▼
Requester Route
          │
          ▼
Accepter Route
          │
          ▼
Security Groups
          │
          ▼
NACLs
          │
          ▼
VPC Flow Logs
          │
          ▼
Application
```

Common problems include:

* Peering request not accepted
* Missing route
* Route added to wrong route table
* Missing return route
* Security Group blocking traffic
* NACL blocking traffic
* Application not listening
* Incorrect destination IP
* Overlapping CIDR design

---

# 🚫 CIDR Overlap

VPC Peering requires non-overlapping IP ranges.

Good:

```text
VPC A
10.0.0.0/16

VPC B
10.20.0.0/16
```

Problematic:

```text
VPC A
10.0.0.0/16

VPC B
10.0.0.0/16
```

This demonstrates why CIDR planning matters before networks are created.

A poorly planned CIDR strategy can make future network integration significantly more difficult.

---

# 🧪 Phase 9: Demonstrate Non-Transitive Routing

Now introduce a third VPC.

```text
VPC C
Database / Analytics VPC

CIDR:
10.30.0.0/16
```

Create peering:

```text
VPC A ←→ VPC B

VPC B ←→ VPC C
```

Architecture:

```text
Application VPC
10.0.0.0/16
      │
      │ Peering
      ▼
Shared Services VPC
10.20.0.0/16
      │
      │ Peering
      ▼
Analytics VPC
10.30.0.0/16
```

Now ask:

> Can VPC A reach VPC C through VPC B?

No.

VPC Peering does **not support transitive routing**.

VPC B cannot act as a transit router for these peering connections.

---

# 🧠 Why This Matters

Imagine:

```text
VPC A ←→ VPC B

VPC B ←→ VPC C
```

This does NOT automatically create:

```text
VPC A ←→ VPC C
```

If A and C require direct connectivity using VPC Peering, they require their own peering relationship and appropriate routes.

```text
       VPC A
       /   \
      /     \
     /       \
  VPC B ─── VPC C
```

This begins creating a **full-mesh architecture**.

---

# 📈 Scaling Problem

With only two VPCs:

```text
A ←→ B
```

Simple.

With three:

```text
A ←→ B
A ←→ C
B ←→ C
```

Still manageable.

Now imagine:

```text
10 VPCs

20 VPCs

50 VPCs

100 VPCs
```

The number of possible peer-to-peer relationships grows rapidly.

For `n` fully connected VPCs:

```text
n(n - 1)
────────
   2
```

For 10 VPCs:

```text
10 × 9
──────
  2

= 45
```

potential peering connections.

For 50 VPCs:

```text
50 × 49
───────
   2

= 1,225
```

potential connections.

This becomes operationally difficult.

---

# 🌐 Why Transit Gateway Exists

Instead of:

```text
      VPC-A
      / | \
     /  |  \
 VPC-B--|--VPC-C
     \  |  /
      \ | /
      VPC-D
```

Transit Gateway provides a hub:

```text
          VPC-A
            │
            │
VPC-B ───── TGW ───── VPC-C
            │
            │
          VPC-D
```

Each VPC connects to the Transit Gateway.

Transit Gateway provides:

* Hub-and-spoke connectivity
* Transitive routing
* Centralized routing
* Network segmentation using TGW route tables
* Easier integration with larger environments

---

# 🆚 VPC Peering vs Transit Gateway

| Feature                   | VPC Peering                          | Transit Gateway                               |
| ------------------------- | ------------------------------------ | --------------------------------------------- |
| Architecture              | Point-to-point                       | Hub-and-spoke                                 |
| Transitive Routing        | ❌ No                                 | ✅ Yes                                         |
| Centralized Routing       | ❌ No                                 | ✅ Yes                                         |
| Small number of VPCs      | Excellent                            | May be unnecessary                            |
| Large network             | Becomes complex                      | Better suited                                 |
| Per-connection simplicity | High                                 | Centralized                                   |
| Cost model                | Peering/data-transfer considerations | TGW attachment/data-processing considerations |

The correct choice depends on the architecture.

Transit Gateway should not automatically replace VPC Peering for every environment.

---

# 🧠 Architecture Scenario

Suppose a company has:

```text
Production VPC
Development VPC
Security VPC
Shared Services VPC
Data VPC
Logging VPC
```

Ask:

> Should we create many independent peering connections?

or:

> Should these networks connect through a central transit architecture?

This is the type of question a Cloud Engineer or Cloud Architect should evaluate.

---

# ❓ Interview Questions

### Q1. What is VPC Peering?

**Answer**

VPC Peering provides private IP connectivity between two VPCs using AWS networking infrastructure.

---

### Q2. Does VPC Peering require an Internet Gateway?

**Answer**

No.

Peered VPC traffic does not require the public Internet or an Internet Gateway.

---

### Q3. Do the VPC CIDRs need to be different?

**Answer**

They must not overlap.

---

### Q4. Does creating the peering connection automatically configure routing?

**Answer**

No.

The required routes must be configured in the relevant VPC route tables.

---

### Q5. Do both VPCs require routes?

**Answer**

For bidirectional communication, appropriate forward and return routes are required.

---

### Q6. Does VPC Peering support transitive routing?

**Answer**

No.

If VPC A peers with B and B peers with C, A cannot use B as a transit network to reach C through those peering connections.

---

### Q7. When would you use VPC Peering?

**Answer**

VPC Peering is useful when a relatively small number of VPCs require direct private connectivity and a centralized transit architecture is unnecessary.

---

### Q8. When might Transit Gateway be more appropriate?

**Answer**

Transit Gateway becomes attractive when many VPCs or hybrid networks require centralized routing, transitive connectivity, and easier network segmentation/management.

---

### Q9. Why is CIDR planning important?

**Answer**

Overlapping address ranges create routing ambiguity and can prevent straightforward connectivity mechanisms such as VPC Peering.

---

### Q10. Can VPC B act as a router between two peered VPCs?

**Answer**

No.

VPC Peering does not provide transitive routing.

---

# 🏁 Completion Criteria

Do not mark this lab complete until you can:

* [ ] Design non-overlapping VPC CIDRs
* [ ] Create a second VPC
* [ ] Create VPC Peering
* [ ] Explain requester and accepter
* [ ] Configure routes in VPC A
* [ ] Configure return routes in VPC B
* [ ] Configure least-privilege security
* [ ] Connect using private IP addresses
* [ ] Break and troubleshoot a route
* [ ] Break and troubleshoot a Security Group rule
* [ ] Explain why overlapping CIDRs are problematic
* [ ] Create or reason through a third VPC scenario
* [ ] Demonstrate that VPC Peering is non-transitive
* [ ] Explain the full-mesh scaling problem
* [ ] Calculate peering relationships for multiple VPCs
* [ ] Explain when Transit Gateway becomes useful
* [ ] Compare VPC Peering and Transit Gateway

---

# 💰 Cost Awareness

Monitor resources used during the lab, including:

* EC2 instances
* NAT Gateway retained from previous labs
* Elastic IPv4 addresses
* VPC Flow Logs
* CloudWatch Logs
* Data transfer
* Any Transit Gateway resources if you later choose to deploy them

There is no need to deploy a Transit Gateway merely to complete the conceptual portion of this lab.

The key objective is understanding **why it exists**.

---

# 🧹 Cleanup

After completing the lab:

* [ ] Terminate test EC2 instances
* [ ] Delete VPC Peering connections
* [ ] Delete temporary VPC C if created
* [ ] Delete Shared Services VPC resources
* [ ] Delete NAT Gateway if retained
* [ ] Release unused Elastic IP addresses
* [ ] Delete VPC Endpoints if retained
* [ ] Delete unnecessary Flow Logs
* [ ] Delete unnecessary CloudWatch Log Groups
* [ ] Delete route tables
* [ ] Delete Security Groups
* [ ] Delete subnets
* [ ] Detach/delete Internet Gateway
* [ ] Delete lab VPCs

Verify that no chargeable resources remain.

---

# 💡 Key Takeaways

* VPCs are isolated networks unless connectivity is explicitly configured.
* VPC Peering connects two VPCs privately.
* VPC Peering is point-to-point.
* Peering requires non-overlapping CIDRs.
* Routes must be configured on both sides for bidirectional communication.
* Security controls must permit the required traffic.
* Routing and security are separate concerns.
* VPC Peering does not support transitive routing.
* Full-mesh peering becomes increasingly difficult as the number of VPCs grows.
* Transit Gateway provides centralized, transitive network connectivity for larger architectures.
* Good CIDR planning today can prevent major networking problems later.

> Do not choose a networking service because it appears on an AWS architecture diagram. Understand the communication requirement first, then choose the simplest architecture that satisfies it.
