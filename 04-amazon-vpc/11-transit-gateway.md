# 🌐 AWS Transit Gateway

> Learn how AWS Transit Gateway acts as a central network hub for connecting multiple VPCs and on-premises networks using a scalable hub-and-spoke architecture.

---

# 📖 Overview

**AWS Transit Gateway (TGW)** is a managed network transit service that acts as a **central hub** for connecting multiple Amazon VPCs and on-premises networks.

Instead of creating individual point-to-point connections between every VPC, Transit Gateway provides a centralized connectivity model.

This simplifies large AWS network architectures and reduces the administrative overhead associated with managing many VPC Peering connections.

Transit Gateway can connect:

- Multiple Amazon VPCs
- AWS accounts
- On-premises networks
- Site-to-Site VPN connections
- AWS Direct Connect through a Direct Connect Gateway
- Transit Gateways in other AWS Regions

---

# 📊 Key Components

| Component | Purpose |
|-----------|---------|
| Transit Gateway | Central networking hub |
| VPC Attachment | Connects a VPC to the Transit Gateway |
| VPN Attachment | Connects on-premises networks using Site-to-Site VPN |
| Transit Gateway Peering | Connects Transit Gateways across AWS Regions |
| Transit Gateway Route Table | Controls traffic between connected networks |
| AWS RAM | Shares Transit Gateway across AWS accounts |

---

# 🌐 What is AWS Transit Gateway?

**AWS Transit Gateway** is a Regional network transit hub that connects multiple VPCs and external networks through a centralized architecture.

Without Transit Gateway, connecting several VPCs using VPC Peering could require many individual connections.

Example:

```text
Without Transit Gateway

       VPC A
      /     \
     /       \
 VPC B ───── VPC C
     \       /
      \     /
       VPC D

Multiple Peering Connections
```

As the number of VPCs increases, the number of connections becomes increasingly difficult to manage.

Transit Gateway simplifies this using a **Hub-and-Spoke model**:

```text
             VPC A
               │
               │
               ▼
VPC B ───► Transit Gateway ◄─── VPC C
               ▲
               │
               │
             VPC D
```

Each VPC connects to the Transit Gateway instead of directly connecting to every other VPC.

---

# 🔗 Transit Gateway Attachments

Networks connect to a Transit Gateway using **Attachments**.

Common attachment types include:

```text
                   Transit Gateway
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
     VPC Attachment  VPN Attachment  TGW Peering
```

For example, when connecting a VPC to a Transit Gateway, a **VPC Attachment** is created.

The Transit Gateway then uses its routing configuration to determine whether traffic can travel between the attached networks.

---

# 🔄 Transitive Routing

One of the major differences between **VPC Peering** and **Transit Gateway** is support for **Transitive Routing**.

VPC Peering does not support transitive routing.

Example:

```text
VPC A ←→ VPC B ←→ VPC C

VPC A cannot automatically reach VPC C
through VPC B.
```

Transit Gateway acts as a routing hub.

Example:

```text
VPC A
   │
   ▼
Transit Gateway
   │
   ▼
VPC C
```

Multiple attached networks can communicate through the Transit Gateway when the appropriate routes and security controls are configured.

This makes Transit Gateway suitable for large multi-VPC environments.

---

# 🛣️ Transit Gateway Route Tables

Transit Gateway uses **Transit Gateway Route Tables** to control communication between attached networks.

These are separate from the VPC Route Tables you have already learned.

Example:

```text
VPC A
10.10.0.0/16
      │
      ▼
Transit Gateway
      │
      ▼
VPC B
10.20.0.0/16
```

The VPC Route Table must direct traffic toward the Transit Gateway:

```text
Destination        Target

10.20.0.0/16  ---> Transit Gateway
```

The Transit Gateway Route Table must also contain the appropriate routing information for the destination attachment.

Therefore, successful communication depends on both:

```text
VPC Route Table
       +
Transit Gateway Route Table
       +
Security Rules
       =
Successful Communication
```

---

# 🏢 Multi-Account Connectivity

Large organizations commonly use multiple AWS accounts.

For example:

```text
Organization

├── Production Account
│      └── Production VPC
│
├── Development Account
│      └── Development VPC
│
├── Security Account
│      └── Security VPC
│
└── Shared Services Account
       └── Shared Services VPC
```

Instead of deploying separate Transit Gateways in every account, organizations can centrally manage network connectivity.

A Transit Gateway can be shared with other AWS accounts using **AWS Resource Access Manager (AWS RAM)**.

Example:

```text
                 Transit Gateway
                 Network Account
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Production    Development    Security
       Account       Account       Account
```

This provides centralized network connectivity while allowing workloads to remain in separate AWS accounts.

---

# 🏢 Connecting On-Premises Networks

Transit Gateway can also act as a central hub between AWS and on-premises environments.

Example:

```text
                On-Premises
                 Data Center
                      │
                      │
             VPN / Direct Connect
                      │
                      ▼
               Transit Gateway
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       VPC A        VPC B       VPC C
```

This is useful in **Hybrid Cloud architectures** where multiple AWS VPCs need connectivity to corporate data centers.

---

# 🌎 Inter-Region Transit Gateway Peering

Transit Gateways can be peered across AWS Regions.

Example:

```text
Canada Central
ca-central-1

VPC A
  │
  ▼
Transit Gateway
  │
  │ TGW Peering
  ▼
Transit Gateway
  │
  ▼
VPC B

US East
us-east-1
```

This allows organizations to build private multi-Region network architectures using the AWS global network.

---

# 🏗️ Real-World Example

Imagine an enterprise with:

```text
Production VPC
Development VPC
Security VPC
Shared Services VPC
Analytics VPC
On-Premises Data Center
```

Using only VPC Peering could require many individual connections.

Instead:

```text
                    On-Premises
                         │
                         ▼
                  Transit Gateway
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Production        Development       Security
      VPC               VPC              VPC
        │
        ├───────────────┐
        ▼               ▼
 Shared Services     Analytics
      VPC               VPC
```

Each network connects to the centralized Transit Gateway.

This makes the architecture easier to:

- Scale
- Manage
- Route
- Troubleshoot
- Govern

---

# 🎯 Why Use AWS Transit Gateway?

AWS Transit Gateway helps you:

- Connect multiple VPCs through a central networking hub.
- Connect AWS networks with on-premises environments.
- Support transitive routing.
- Reduce the number of point-to-point VPC Peering connections.
- Centralize network routing.
- Connect workloads across multiple AWS accounts.
- Build scalable multi-VPC architectures.
- Build multi-Region network architectures using Transit Gateway Peering.

---

# ✅ Best Practices

- Use Transit Gateway when managing connectivity between many VPCs.
- Use a dedicated networking account for centralized network management in large multi-account environments.
- Share Transit Gateway across AWS accounts using **AWS Resource Access Manager (AWS RAM)**.
- Design Transit Gateway Route Tables based on required network segmentation.
- Avoid allowing every attached network to communicate with every other network unless required.
- Follow the Principle of Least Privilege when designing network routes.
- Plan VPC CIDR ranges carefully to avoid overlapping networks.
- Monitor Transit Gateway traffic and routing configurations.
- Consider data processing and cross-Region transfer costs when designing the architecture.

---

# 📊 VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|---------|-------------|-----------------|
| Architecture | Point-to-Point | Hub-and-Spoke |
| Connectivity | Two VPCs per connection | Multiple VPCs and networks |
| Transitive Routing | ❌ No | ✅ Yes |
| Centralized Routing | ❌ No | ✅ Yes |
| Multi-Account | ✅ Yes | ✅ Yes |
| Inter-Region | ✅ Yes | ✅ Through TGW Peering |
| On-Premises Connectivity | Not directly | ✅ Yes |
| Management at Large Scale | Complex | Simplified |
| Best For | Small number of VPCs | Large multi-VPC environments |

---

# ❓ Interview Questions

### Q1. What is AWS Transit Gateway?

**Answer**

AWS Transit Gateway is a managed network transit service that acts as a central hub for connecting multiple VPCs and on-premises networks.

---

### Q2. Does Transit Gateway support transitive routing?

**Answer**

Yes. Transit Gateway supports transitive routing between attached networks when the appropriate Transit Gateway Route Tables and VPC routes are configured.

---

### Q3. What is the main difference between VPC Peering and Transit Gateway?

**Answer**

VPC Peering provides point-to-point connectivity between two VPCs and does not support transitive routing.

Transit Gateway provides centralized hub-and-spoke connectivity for multiple VPCs and networks and supports transitive routing.

---

### Q4. Can Transit Gateway be shared across AWS accounts?

**Answer**

Yes. Transit Gateway can be shared with other AWS accounts using **AWS Resource Access Manager (AWS RAM)**.

---

### Q5. Can Transit Gateway connect to an on-premises data center?

**Answer**

Yes.

On-premises networks can connect to Transit Gateway using services such as:

- AWS Site-to-Site VPN
- AWS Direct Connect through a Direct Connect Gateway

---

### Q6. Is Transit Gateway a Regional service?

**Answer**

Yes. AWS Transit Gateway is a Regional resource.

Transit Gateways in different Regions can be connected using **Inter-Region Transit Gateway Peering**.

---

### Q7. Does attaching a VPC to Transit Gateway automatically allow communication?

**Answer**

No.

The required routes must be configured in the VPC Route Tables and Transit Gateway Route Tables. Security controls such as Security Groups and Network ACLs must also permit the required traffic.

---

### Q8. When should you choose Transit Gateway instead of VPC Peering?

**Answer**

Transit Gateway is generally preferred when many VPCs, AWS accounts, or on-premises networks require centralized connectivity.

For a small number of VPCs requiring simple point-to-point connectivity, VPC Peering may be sufficient.

---

# 💡 Key Takeaways

- AWS Transit Gateway acts as a **central network transit hub**.
- It connects multiple VPCs and on-premises networks.
- Transit Gateway supports **transitive routing**.
- It uses a **hub-and-spoke architecture** instead of multiple point-to-point connections.
- Transit Gateway Route Tables control communication between attached networks.
- Transit Gateway can be shared across AWS accounts using **AWS RAM**.
- Transit Gateway can connect to on-premises networks using VPN and Direct Connect architectures.
- Transit Gateways can be peered across AWS Regions.
- Transit Gateway significantly simplifies network management as the number of VPCs grows.

---

# 📚 Related Topics

- Amazon VPC
- CIDR & Subnetting
- Amazon VPC Subnets
- Route Tables
- VPC Peering
- AWS Resource Access Manager (AWS RAM)
- AWS Site-to-Site VPN
- AWS Direct Connect
- Transit Gateway Peering
- VPC Flow Logs

---

# 📖 References

- AWS Transit Gateway Documentation
- AWS Transit Gateway Architecture and Concepts
- AWS Transit Gateway Route Tables
- AWS Resource Access Manager
- AWS Transit Gateway Peering