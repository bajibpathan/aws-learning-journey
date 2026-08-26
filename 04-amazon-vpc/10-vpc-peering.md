# 🔗 Amazon VPC Peering

> Learn how Amazon VPC Peering connects two VPCs using private IP addresses, allowing resources to communicate securely without sending traffic over the public Internet.

---

# 📖 Overview

**Amazon VPC Peering** is a networking connection between **two VPCs** that allows resources in those VPCs to communicate using private IPv4 or IPv6 addresses.

The communication takes place using the AWS network infrastructure rather than routing traffic through the public Internet.

VPC Peering can connect VPCs:

* Within the same AWS Region.
* Across different AWS Regions.
* Within the same AWS account.
* Across different AWS accounts.

VPC Peering is useful when applications or services deployed in separate VPCs need private network connectivity.

---

# 📊 Key Components

| Component              | Purpose                                                                 |
| ---------------------- | ----------------------------------------------------------------------- |
| Requester VPC          | VPC that initiates the peering request                                  |
| Accepter VPC           | VPC that accepts the peering request                                    |
| VPC Peering Connection | Private network connection between two VPCs                             |
| CIDR Blocks            | Define the IP address ranges of each VPC                                |
| Route Tables           | Route traffic to the remote VPC through the peering connection          |
| Security Groups        | Control which traffic can reach resources across the peering connection |

---

# 🔗 What is VPC Peering?

A **VPC Peering Connection** provides direct network connectivity between **two VPCs**.

Example:

```text
VPC A
10.0.0.0/16

     │
     │
     ▼

VPC Peering Connection
      pcx-xxxxx

     │
     │
     ▼

VPC B
10.20.0.0/16
```

Resources in VPC A can communicate with resources in VPC B using their **Private IP addresses** when the required routing and security rules are configured.

---

# 🔄 Requester and Accepter

Creating a VPC Peering Connection involves two VPC roles.

### Requester VPC

The VPC that initiates the peering connection request.

### Accepter VPC

The VPC that receives and accepts the peering connection request.

Example:

```text
VPC A
Requester

   │
   │ Peering Request
   ▼

VPC B
Accepter

   │
   │ Accept Request
   ▼

Peering Connection Active
```

The peering connection must be accepted before it becomes active.

---

# 📍 CIDR Requirements

The CIDR blocks of the two VPCs **must not overlap**.

Valid example:

```text
VPC A
10.0.0.0/16

VPC B
10.20.0.0/16

✅ Peering Possible
```

Invalid example:

```text
VPC A
10.0.0.0/16

VPC B
10.0.10.0/24

❌ CIDR Overlap
```

The second network falls inside the first network's address space, so AWS cannot establish the VPC Peering Connection.

This is one reason why proper **CIDR planning** is important when designing AWS environments.

---

# 🛣️ Route Table Configuration

Creating a VPC Peering Connection alone does **not** automatically enable communication.

Route Tables must be configured on both sides.

Example:

```text
VPC A
CIDR: 10.0.0.0/16

Route Table

Destination       Target
10.0.0.0/16       Local
10.20.0.0/16      pcx-xxxxx
```

VPC B:

```text
VPC B
CIDR: 10.20.0.0/16

Route Table

Destination       Target
10.20.0.0/16      Local
10.0.0.0/16       pcx-xxxxx
```

This creates the required routing path in both directions.

---

# 🔒 Security Groups

Routing alone does not guarantee that applications can communicate.

Security Groups must also allow the required traffic.

Example:

```text
Application
VPC A
10.0.0.0/16

      │
      │ TCP 5432
      ▼

Database
VPC B
10.20.0.0/16
```

If the application needs to connect to PostgreSQL in VPC B, the Database Security Group must allow the appropriate traffic from the permitted source.

The network path therefore depends on both:

```text
Routing
   +
Security Rules
   =
Successful Communication
```

---

# 🌎 Inter-Region VPC Peering

VPC Peering can also connect VPCs located in different AWS Regions.

Example:

```text
Canada Central
ca-central-1

VPC A
10.0.0.0/16

       │
       │
       ▼

Inter-Region
VPC Peering

       │
       │
       ▼

US East
us-east-1

VPC B
10.20.0.0/16
```

Inter-Region VPC Peering traffic:

* Uses private IP addresses.
* Remains on the AWS global network.
* Does not traverse the public Internet.
* Is encrypted before leaving AWS facilities.

This can be useful for architectures requiring private connectivity between workloads deployed across AWS Regions.

---

# 🚫 No Transitive Routing

One of the most important limitations of VPC Peering is that it does **not support transitive routing**.

Consider:

```text
VPC A
   │
   │ Peering
   ▼
VPC B
   │
   │ Peering
   ▼
VPC C
```

If:

```text
VPC A ↔ VPC B
```

and:

```text
VPC B ↔ VPC C
```

this does **not** mean:

```text
VPC A ↔ VPC C
```

VPC A cannot use VPC B as a transit network to reach VPC C.

A direct peering connection is required:

```text
        VPC A
       /     \
      /       \
     ▼         ▼
   VPC B ←──→ VPC C
```

Each required communication path needs its own VPC Peering Connection.

---

# 🔢 Multiple VPC Peering Connections

A VPC Peering Connection connects only **two VPCs**.

If multiple VPCs need direct connectivity, multiple peering connections are required.

Example:

```text
VPC A ↔ VPC B
VPC A ↔ VPC C
VPC B ↔ VPC C
```

Three VPCs requiring full connectivity therefore require **three VPC Peering Connections**.

As the number of VPCs increases, managing a full-mesh peering architecture becomes increasingly complex.

For large multi-VPC environments, **AWS Transit Gateway** may provide a more scalable connectivity model.

---

# 🏢 Real-World Example

Imagine an organization separates workloads into different VPCs:

```text
Production VPC
10.10.0.0/16

Analytics VPC
10.20.0.0/16
```

The Analytics application needs access to a database or service running inside the Production VPC.

Instead of exposing the service through the public Internet:

```text
Analytics VPC
      │
      │ Private IP
      ▼
VPC Peering
      │
      │ Private IP
      ▼
Production VPC
```

The two environments can communicate privately using a VPC Peering Connection.

This keeps the communication within AWS networking infrastructure while maintaining separate VPC boundaries.

---

# 🎯 Why Use VPC Peering?

VPC Peering helps you:

* Connect resources across separate VPCs.
* Communicate using Private IP addresses.
* Avoid routing VPC-to-VPC traffic through the public Internet.
* Connect VPCs across AWS accounts.
* Connect VPCs across AWS Regions.
* Share services between separate application environments.
* Maintain separate VPC boundaries while enabling controlled communication.

---

# ✅ Best Practices

* Plan VPC CIDR ranges carefully to prevent overlaps.
* Avoid using overlapping CIDR ranges across environments that may need future connectivity.
* Configure routes only for networks that require communication.
* Apply the Principle of Least Privilege to Security Group rules.
* Avoid creating unnecessary full-mesh VPC Peering architectures.
* Monitor the number of peering connections as the environment grows.
* Consider **AWS Transit Gateway** when connecting a large number of VPCs.
* Review cross-Availability Zone and cross-Region data transfer costs.
* Accept peering requests only from trusted AWS accounts.

---

# 📊 VPC Peering vs Transit Gateway

| Feature                   | VPC Peering             | Transit Gateway              |
| ------------------------- | ----------------------- | ---------------------------- |
| Connectivity              | Point-to-Point          | Hub-and-Spoke                |
| Connects                  | Two VPCs per connection | Multiple VPCs and networks   |
| Transitive Routing        | ❌ No                    | ✅ Yes                        |
| Complexity at Small Scale | Low                     | Higher                       |
| Complexity at Large Scale | High                    | Lower                        |
| Best For                  | Small number of VPCs    | Large multi-VPC environments |

---

# ❓ Interview Questions

### Q1. What is VPC Peering?

**Answer**

VPC Peering is a networking connection between two VPCs that allows resources to communicate using Private IPv4 or IPv6 addresses without routing traffic through the public Internet.

---

### Q2. Can VPCs with overlapping CIDR ranges be peered?

**Answer**

No. AWS does not allow VPC Peering between VPCs with overlapping or matching CIDR blocks.

---

### Q3. Does VPC Peering support transitive routing?

**Answer**

No. VPC Peering does not support transitive routing.

If VPC A is peered with VPC B and VPC B is peered with VPC C, VPC A cannot communicate with VPC C through VPC B.

---

### Q4. Can VPC Peering work across AWS Regions?

**Answer**

Yes. AWS supports Inter-Region VPC Peering.

Inter-Region traffic remains on the AWS global network and is encrypted before leaving AWS facilities.

---

### Q5. Are Route Table changes required after creating a VPC Peering Connection?

**Answer**

Yes.

Each side must have appropriate routes directing traffic for the peer VPC's CIDR range to the VPC Peering Connection.

---

### Q6. Does creating a VPC Peering Connection automatically allow traffic?

**Answer**

No.

The peering connection provides the connectivity mechanism, but Route Tables and security controls such as Security Groups must also allow the required traffic.

---

### Q7. When should Transit Gateway be considered instead of VPC Peering?

**Answer**

Transit Gateway should be considered when many VPCs or networks need connectivity because VPC Peering is point-to-point and can become difficult to manage at scale.

---

# 💡 Key Takeaways

* VPC Peering provides private connectivity between **two VPCs**.
* Peered VPCs must have **non-overlapping CIDR ranges**.
* One VPC acts as the Requester and the other as the Accepter.
* Route Tables must be configured on both sides.
* Security Groups must allow the required application traffic.
* VPC Peering supports both same-Region and Inter-Region connectivity.
* Inter-Region peering traffic stays on the AWS global network and is encrypted.
* VPC Peering does **not support transitive routing**.
* Multiple VPCs require multiple point-to-point peering connections.
* Transit Gateway is generally better suited to larger multi-VPC architectures.

---

# 📚 Related Topics

* Amazon VPC
* CIDR & Subnetting
* Amazon VPC Subnets
* Route Tables
* Security Groups
* Network ACLs
* VPC Endpoints
* AWS Transit Gateway
* AWS Site-to-Site VPN
* AWS Direct Connect

---

# 📖 References

* AWS — What is VPC Peering?
* AWS — How VPC Peering Connections Work
* AWS — Update Route Tables for a VPC Peering Connection
* AWS — VPC Peering Configurations
