# 🌐 AWS Egress-Only Internet Gateway

> Learn how an AWS Egress-Only Internet Gateway allows resources using IPv6 addresses to initiate outbound Internet connections while preventing unsolicited inbound IPv6 connections from the Internet.

---

# 📖 Overview

An **Egress-Only Internet Gateway (EIGW)** is an AWS VPC networking component that provides **outbound-only Internet connectivity for IPv6 traffic**.

It allows resources inside a VPC to initiate connections to the Internet while preventing the Internet from initiating new connections to those resources through the Egress-Only Internet Gateway.

A simplified architecture looks like:

```text
IPv6 EC2 Instance
Private Subnet
       │
       │ Outbound IPv6
       ▼
Egress-Only Internet Gateway
       │
       ▼
    Internet
```

Return traffic for connections initiated by the instance is allowed back through the Egress-Only Internet Gateway.

---

# 📊 Key Components

| Component                    | Purpose                                                          |
| ---------------------------- | ---------------------------------------------------------------- |
| Egress-Only Internet Gateway | Provides outbound-only IPv6 Internet connectivity                |
| IPv6 Address                 | Globally unique IPv6 address assigned to the resource            |
| Route Table                  | Routes IPv6 Internet traffic to the Egress-Only Internet Gateway |
| `::/0` Route                 | Represents the default route for IPv6 traffic                    |
| Security Group               | Controls inbound and outbound traffic at the resource level      |
| Network ACL                  | Controls traffic at the subnet level                             |

---

# 🌐 What is an Egress-Only Internet Gateway?

An **Egress-Only Internet Gateway** allows IPv6-enabled resources inside a VPC to initiate connections to the Internet.

For example:

```text
EC2 Instance
IPv6 Address
     │
     │ Initiates HTTPS Connection
     ▼
Egress-Only Internet Gateway
     │
     ▼
Internet
```

The response traffic is allowed to return:

```text
Internet
    │
    │ Response
    ▼
Egress-Only Internet Gateway
    │
    ▼
EC2 Instance
```

However, an external Internet host cannot use the Egress-Only Internet Gateway to initiate a new connection toward the EC2 instance.

---

# 🔄 Outbound vs Inbound Traffic

The primary purpose of an Egress-Only Internet Gateway is to allow **outbound-initiated IPv6 communication**.

Allowed flow:

```text
AWS Resource
     │
     │ Initiates Connection
     ▼
Egress-Only Internet Gateway
     │
     ▼
Internet
     │
     │ Response Traffic
     ▼
Egress-Only Internet Gateway
     │
     ▼
AWS Resource

✅ Allowed
```

Unsolicited inbound connection:

```text
Internet
    │
    │ New Connection
    ▼
Egress-Only Internet Gateway
    │
    ✖
    │
AWS Resource

❌ Blocked
```

This makes EIGW useful when IPv6-enabled workloads require Internet access but should not accept Internet-initiated connections.

---

# 🛣️ Route Table Configuration

Creating an Egress-Only Internet Gateway alone does not provide Internet connectivity.

The subnet's Route Table must direct IPv6 Internet traffic toward the EIGW.

Example:

```text
Destination              Target

10.0.0.0/16              Local
2001:db8:1234::/56        Local
::/0                      eigw-xxxxxxxx
```

The important route is:

```text
::/0 → Egress-Only Internet Gateway
```

`::/0` is the IPv6 equivalent of the IPv4 default route:

```text
IPv4
0.0.0.0/0

IPv6
::/0
```

---

# 🧠 Why Do We Need Egress-Only Internet Gateway?

With IPv4, private resources commonly use private IPv4 addresses.

Example:

```text
Private EC2
10.0.10.50
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

The NAT Gateway translates the private IPv4 address to a public IPv4 address.

IPv6 works differently.

IPv6 addresses assigned to VPC resources can be globally unique and publicly routable.

Therefore, NAT is generally not required simply to translate private addresses into public addresses.

But this creates another requirement:

> How can an IPv6 resource access the Internet without allowing the Internet to initiate connections back to it?

AWS provides the **Egress-Only Internet Gateway** for this requirement.

---

# 🔄 IPv4 NAT vs IPv6 Egress-Only Gateway

Consider an application server that needs to download software updates.

For IPv4:

```text
Private EC2
Private IPv4
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

For IPv6:

```text
EC2
IPv6 Address
     │
     ▼
Egress-Only
Internet Gateway
     │
     ▼
Internet
```

The architectural goal is similar:

```text
Workload can initiate
Internet connections

        BUT

Internet should not initiate
connections to the workload
```

The implementation is different because IPv4 and IPv6 addressing work differently.

---

# 🚫 Egress-Only Internet Gateway Does Not Perform NAT

An important distinction is:

**Egress-Only Internet Gateway does not perform Network Address Translation.**

The IPv6 source address is not translated into another IPv6 address.

Conceptually:

```text
IPv6 Instance
2001:db8:1234::10

        │
        ▼

Egress-Only Internet Gateway

        │
        ▼

Internet

Source IPv6 remains IPv6
```

This differs from the common IPv4 NAT Gateway architecture.

---

# 🌐 Internet Gateway vs Egress-Only Internet Gateway

A normal **Internet Gateway (IGW)** can provide Internet connectivity for both IPv4 and IPv6 when routing and security controls permit it.

For IPv6:

```text
IPv6 EC2
    │
    ▼
Internet Gateway
    │
    ↕
Internet

Potential inbound and outbound connectivity
subject to security controls
```

An Egress-Only Internet Gateway is specifically designed for outbound-initiated IPv6 connectivity:

```text
IPv6 EC2
    │
    ▼
Egress-Only Internet Gateway
    │
    ▼
Internet

Outbound initiated connectivity only
```

This distinction is important when designing private IPv6 workloads.

---

# 🔒 Security Groups Still Matter

An Egress-Only Internet Gateway does not replace Security Groups.

Security Groups continue to control traffic at the resource's Elastic Network Interface.

Example:

```text
EC2 Instance
     │
     ▼
Security Group
     │
     ▼
Route Table
     │
     ▼
Egress-Only Internet Gateway
     │
     ▼
Internet
```

The Security Group must allow the required outbound traffic.

For example:

```text
HTTPS
TCP 443
```

Security Groups remain an important layer of network security.

---

# 🛡️ Network ACLs Still Matter

Network ACLs also continue to apply at the subnet level.

Remember that Network ACLs are **stateless**.

Therefore, both the outbound connection and the required return traffic must be permitted.

Conceptually:

```text
EC2
 │
 ▼
Security Group
 │
 ▼
Network ACL
 │
 ▼
Route Table
 │
 ▼
Egress-Only Internet Gateway
 │
 ▼
Internet
```

An Egress-Only Internet Gateway does not bypass existing VPC security controls.

---

# 🏢 Real-World Example

Imagine an application server running inside an IPv6-enabled subnet.

The server needs to:

* Download operating system updates.
* Access external APIs.
* Download software packages.
* Contact external monitoring services.

However, the organization does not want Internet users initiating connections to the server.

The architecture could be:

```text
Private Application Server
IPv6 Enabled
       │
       │ HTTPS Request
       ▼
Route Table
::/0 → EIGW
       │
       ▼
Egress-Only Internet Gateway
       │
       ▼
Internet
```

The application can initiate outbound Internet connections.

But:

```text
Internet
    │
    │ New inbound connection
    ▼
Egress-Only Internet Gateway
    │
    ✖
    │
Application Server
```

The unsolicited inbound connection is blocked.

---

# 🎯 Why Use an Egress-Only Internet Gateway?

Egress-Only Internet Gateway helps you:

* Provide outbound IPv6 Internet connectivity.
* Prevent unsolicited inbound IPv6 connections.
* Keep IPv6 workloads from being directly reachable through inbound Internet-initiated connections.
* Avoid using NAT simply for IPv6 address translation.
* Build IPv6-enabled private application architectures.
* Maintain outbound Internet access for software updates and external APIs.

---

# 💰 Cost Considerations

An Egress-Only Internet Gateway does not have the same hourly and data-processing pricing model as an IPv4 NAT Gateway.

This can make IPv6 architectures attractive in scenarios where workloads require outbound Internet connectivity.

However, normal AWS data transfer charges and charges for other services involved in the architecture may still apply.

Always review current AWS pricing when designing production architectures.

---

# ✅ Best Practices

* Use an Egress-Only Internet Gateway when IPv6 workloads require outbound Internet connectivity but should not accept unsolicited inbound Internet connections.
* Configure the IPv6 default route as `::/0` toward the Egress-Only Internet Gateway.
* Continue using Security Groups to control resource-level traffic.
* Continue using Network ACLs when subnet-level controls are required.
* Do not treat an Egress-Only Internet Gateway as an IPv6 NAT Gateway.
* Apply the Principle of Least Privilege to outbound Security Group rules where appropriate.
* Verify both IPv4 and IPv6 routing when operating dual-stack workloads.
* Avoid assuming that an IPv4 private subnet design automatically behaves the same way with IPv6.

---

# 📊 Internet Gateway vs NAT Gateway vs Egress-Only Internet Gateway

| Feature                               | Internet Gateway                       | NAT Gateway                              | Egress-Only Internet Gateway    |
| ------------------------------------- | -------------------------------------- | ---------------------------------------- | ------------------------------- |
| IPv4                                  | ✅                                      | ✅                                        | ❌                               |
| IPv6                                  | ✅                                      | NAT64 scenarios possible                 | ✅                               |
| Primary Purpose                       | Internet connectivity                  | IPv4 outbound connectivity / translation | IPv6 outbound-only connectivity |
| NAT                                   | IPv4 public/private mapping behavior   | ✅                                        | ❌                               |
| Allows Internet-Initiated Connections | Possible with correct routing/security | ❌                                        | ❌                               |
| Typical Workload                      | Public resources                       | Private IPv4 resources                   | Private-style IPv6 resources    |
| Default Route                         | `0.0.0.0/0` or `::/0`                  | `0.0.0.0/0`                              | `::/0`                          |

---

# ❓ Interview Questions

### Q1. What is an Egress-Only Internet Gateway?

**Answer**

An Egress-Only Internet Gateway is an AWS VPC component that allows IPv6-enabled resources to initiate outbound Internet connections while preventing unsolicited inbound IPv6 connections from the Internet.

---

### Q2. Is an Egress-Only Internet Gateway used for IPv4?

**Answer**

No.

Egress-Only Internet Gateway is designed specifically for **IPv6 traffic**.

For private IPv4 workloads requiring outbound Internet connectivity, a NAT Gateway is commonly used.

---

### Q3. What route is required for an Egress-Only Internet Gateway?

**Answer**

The Route Table typically contains:

```text
::/0 → Egress-Only Internet Gateway
```

`::/0` represents the default IPv6 route.

---

### Q4. Does an Egress-Only Internet Gateway perform NAT?

**Answer**

No.

Egress-Only Internet Gateway does not perform Network Address Translation.

IPv6 does not generally require the same private-to-public address translation model commonly used with IPv4.

---

### Q5. What is the difference between an Internet Gateway and an Egress-Only Internet Gateway?

**Answer**

An Internet Gateway can provide inbound and outbound Internet connectivity when routing and security rules allow it.

An Egress-Only Internet Gateway allows IPv6 resources to initiate outbound connections while preventing unsolicited inbound connections from the Internet.

---

### Q6. What is the IPv6 equivalent of `0.0.0.0/0`?

**Answer**

The IPv6 default route is:

```text
::/0
```

---

### Q7. Does an Egress-Only Internet Gateway replace Security Groups?

**Answer**

No.

Security Groups and Network ACLs continue to control network traffic.

The Egress-Only Internet Gateway controls the Internet connectivity path for outbound-initiated IPv6 traffic.

---

### Q8. Why doesn't IPv6 normally require NAT Gateway for Internet access?

**Answer**

IPv6 provides a much larger address space, allowing resources to use globally unique IPv6 addresses.

Therefore, private-to-public address translation is generally not required simply to provide IPv6 Internet connectivity.

An Egress-Only Internet Gateway can instead control whether Internet connections can be initiated toward those resources.

---

# 💡 Key Takeaways

* Egress-Only Internet Gateway is specifically designed for **IPv6**.
* It allows workloads to **initiate outbound Internet connections**.
* It prevents **unsolicited inbound Internet connections**.
* The IPv6 default route is `::/0`.
* The Route Table points `::/0` toward the Egress-Only Internet Gateway.
* Egress-Only Internet Gateway does **not perform NAT**.
* Security Groups and Network ACLs continue to apply.
* NAT Gateway is commonly associated with private IPv4 Internet access.
* Egress-Only Internet Gateway provides a similar outbound-only architectural goal for IPv6, but without address translation.

---

# 📚 Related Topics

* Amazon VPC
* IPv4 and IPv6
* CIDR & Subnetting
* Amazon VPC Subnets
* Route Tables
* Internet Gateway
* NAT Gateway
* Security Groups
* Network ACLs
* VPC Endpoints
* Dual-Stack Networking

---

# 📖 References

* AWS VPC Egress-Only Internet Gateway Documentation
* AWS IPv6 VPC Documentation
* AWS VPC Route Tables
* AWS Internet Gateway Documentation
* AWS NAT Gateway Documentation
