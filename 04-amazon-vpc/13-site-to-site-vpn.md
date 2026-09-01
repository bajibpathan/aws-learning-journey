# 🔐 AWS Site-to-Site VPN

> Learn how AWS Site-to-Site VPN securely connects an on-premises network to AWS using encrypted IPsec tunnels over the Internet.

---

# 📖 Overview

**AWS Site-to-Site VPN** provides secure connectivity between an on-premises network and AWS using encrypted **IPsec VPN tunnels**.

It is commonly used to build **Hybrid Cloud architectures**, where applications or users in a corporate data center need private network connectivity to resources running inside AWS.

A typical architecture looks like:

```text
On-Premises Network
        │
        ▼
Customer Gateway Device
        │
        │
   IPsec VPN Tunnels
        │
        ▼
Virtual Private Gateway
        │
        ▼
      AWS VPC
```

Although the standard Site-to-Site VPN uses the public Internet as its transport network, application traffic travels through encrypted IPsec tunnels.

---

# 📊 Key Components

| Component               | Purpose                                                           |
| ----------------------- | ----------------------------------------------------------------- |
| Site-to-Site VPN        | Provides encrypted connectivity between AWS and external networks |
| Customer Gateway Device | Physical or software VPN device on the customer network           |
| Customer Gateway        | AWS resource representing the customer gateway device             |
| Virtual Private Gateway | AWS VPN endpoint attached to a VPC                                |
| Transit Gateway         | Can act as the AWS-side VPN endpoint for multi-VPC architectures  |
| IPsec Tunnel            | Encrypts traffic between AWS and the customer network             |
| BGP                     | Dynamically exchanges network routes                              |
| Route Table             | Routes traffic between the VPC and on-premises network            |

---

# 🔐 What is AWS Site-to-Site VPN?

**AWS Site-to-Site VPN** establishes encrypted network connectivity between an AWS environment and an external network.

Common external networks include:

* Corporate Data Centers
* Branch Offices
* Co-location Facilities
* Other external network environments

The VPN connection uses **IPsec encryption** to protect traffic between the networks.

Example:

```text
Corporate Network
192.168.0.0/16

       │
       ▼

Customer Gateway
       │
       │
   IPsec Tunnel
       │
       ▼

Virtual Private Gateway
       │
       ▼

AWS VPC
10.0.0.0/16
```

This allows resources in both networks to communicate using their private network addresses.

---

# 🏢 Customer Gateway

The customer side of the VPN connection requires a **Customer Gateway Device**.

This can be:

* Physical router
* Firewall
* VPN appliance
* Software-based VPN device

Examples could include network appliances from vendors such as Cisco, Fortinet, Palo Alto Networks, or other compatible devices.

AWS also requires a **Customer Gateway resource**.

The AWS Customer Gateway resource represents information about the physical or software device running in the customer network.

Conceptually:

```text
On-Premises

Customer Gateway Device
        │
        │ Represented in AWS by
        ▼
AWS Customer Gateway Resource
```

---

# 🌐 Customer Gateway Public IP

For a standard public Site-to-Site VPN, the Customer Gateway Device typically requires a static, Internet-routable IP address.

If the Customer Gateway Device is behind a NAT device:

```text
Customer Gateway
       │
       ▼
   NAT Device
       │
       ▼
Public Internet
```

the **public IP address of the NAT device** should be specified when configuring the AWS Customer Gateway.

The network must also allow the required VPN traffic, including NAT Traversal when applicable.

---

# ☁️ AWS-Side Gateway

On the AWS side, a Site-to-Site VPN can terminate on a:

* **Virtual Private Gateway (VGW)**
* **Transit Gateway (TGW)**

For a simple connection to a single VPC:

```text
On-Premises
     │
     ▼
Site-to-Site VPN
     │
     ▼
Virtual Private Gateway
     │
     ▼
    VPC
```

The Virtual Private Gateway is attached to the VPC.

For larger multi-VPC environments:

```text
On-Premises
     │
     ▼
Site-to-Site VPN
     │
     ▼
Transit Gateway
     │
 ┌───┼───┐
 ▼   ▼   ▼
VPC A VPC B VPC C
```

Transit Gateway can provide centralized connectivity between the on-premises network and multiple VPCs.

---

# 🔄 Two VPN Tunnels

Each AWS Site-to-Site VPN connection provides **two VPN tunnels**.

```text
                    ┌── Tunnel 1 ──┐
                    │              │
Customer Gateway ───┤              ├── AWS Gateway
                    │              │
                    └── Tunnel 2 ──┘
```

The tunnels terminate on separate AWS infrastructure to provide redundancy.

Both tunnels should be configured on the Customer Gateway Device.

If one tunnel becomes unavailable, traffic can fail over to the other tunnel.

---

# 🛣️ Static vs Dynamic Routing

AWS Site-to-Site VPN supports:

* Static Routing
* Dynamic Routing

---

## Static Routing

With Static Routing, network routes are manually configured.

Example:

```text
AWS VPC
10.0.0.0/16

        │
        ▼
VPN Connection
        │
        ▼

On-Premises
192.168.0.0/16
```

The administrator defines which network prefixes should be routed through the VPN.

Static Routing can be appropriate for simple environments with a small number of network routes.

---

## Dynamic Routing

Dynamic Routing uses **Border Gateway Protocol (BGP)**.

```text
On-Premises Router
       │
       │ BGP
       ▼
Site-to-Site VPN
       │
       │ BGP
       ▼
AWS Gateway
```

BGP dynamically exchanges network routing information between AWS and the customer network.

This can simplify routing management and improve failover behavior.

If the Customer Gateway Device supports BGP, AWS recommends using dynamic routing when appropriate.

---

# 📍 CIDR Planning

Network address planning is important when connecting AWS with on-premises networks.

Example:

```text
AWS VPC
10.0.0.0/16

Corporate Network
192.168.0.0/16
```

Using distinct network ranges simplifies routing and avoids ambiguous network paths.

This is why CIDR planning should be considered before building hybrid network connectivity.

---

# 🛣️ Route Table Configuration

Creating the VPN connection alone is not enough.

AWS must know which traffic should be routed toward the on-premises network.

Example:

```text
AWS VPC Route Table

Destination         Target

10.0.0.0/16         Local
192.168.0.0/16      Virtual Private Gateway
```

Routes can be:

* Manually configured.
* Propagated from the VPN connection where supported.

Successful connectivity therefore depends on:

```text
VPN Connection
      +
Routing
      +
Security Rules
      =
Successful Communication
```

---

# 🔒 IPsec Encryption

Site-to-Site VPN protects traffic using **IPsec**.

Conceptually:

```text
On-Premises
     │
     │
     ▼
══════════════════════
 Encrypted IPsec Tunnel
══════════════════════
     │
     ▼
AWS
```

The underlying Internet connection carries encrypted VPN traffic rather than exposing the application traffic directly.

---

# ⚡ VPN Bandwidth

A standard AWS Site-to-Site VPN tunnel supports bandwidth of up to:

```text
1.25 Gbps per tunnel
```

AWS also supports **Large Bandwidth Tunnels** with capacity up to:

```text
5 Gbps per tunnel
```

Large Bandwidth Tunnels are available for supported VPN connections attached to:

* AWS Transit Gateway
* AWS Cloud WAN

They are not supported with Virtual Private Gateway connections.

Actual VPN throughput can depend on factors such as:

* Packet size
* Network conditions
* Traffic type
* Customer Gateway performance
* Internet conditions

---

# 🌐 Internet Dependency

Standard AWS Site-to-Site VPN connectivity generally uses the **public Internet** as its underlying transport.

Therefore:

```text
Corporate Network
       │
       ▼
Public Internet
       │
Encrypted IPsec
       ▼
AWS
```

Although the traffic is encrypted, network performance can be affected by Internet conditions.

This means Site-to-Site VPN may provide less predictable network performance than dedicated private connectivity such as **AWS Direct Connect**.

---

# 🏢 Real-World Example

Imagine a company running its employee management system in AWS.

The application runs inside a Private Subnet:

```text
AWS VPC
10.0.0.0/16

Private Application
10.0.10.0/24
```

Employees inside the corporate network need access:

```text
Corporate Network
192.168.0.0/16
```

The company does not want to expose the application to the public Internet.

Instead:

```text
Corporate Users
      │
      ▼
Corporate Network
192.168.0.0/16
      │
      ▼
Customer Gateway
      │
      │
Encrypted IPsec VPN
      │
      ▼
Virtual Private Gateway
      │
      ▼
Private AWS Application
10.0.10.0/24
```

Employees can now access the AWS application through private network routing while the traffic between the networks is protected by the VPN tunnel.

---

# 🎯 Why Use AWS Site-to-Site VPN?

Site-to-Site VPN helps organizations:

* Build Hybrid Cloud architectures.
* Connect corporate networks with AWS.
* Secure network traffic using IPsec.
* Access AWS resources using private network addresses.
* Avoid exposing private applications directly to the Internet.
* Establish connectivity relatively quickly compared with dedicated physical connectivity.
* Provide backup connectivity for other hybrid network connections.

---

# 💰 Cost Considerations

AWS Site-to-Site VPN is a chargeable service.

Costs can include:

* VPN connection or attachment charges.
* Data transfer charges.
* Additional charges depending on the AWS networking architecture.

Pricing can vary depending on the VPN configuration and AWS Region.

Always review current AWS pricing before designing production architectures.

---

# ✅ Best Practices

* Configure **both VPN tunnels** for High Availability.
* Prefer BGP dynamic routing when supported and appropriate.
* Plan AWS and on-premises CIDR ranges carefully.
* Avoid overlapping network ranges.
* Use redundant Customer Gateway devices for critical production environments where required.
* Monitor VPN tunnel status using Amazon CloudWatch.
* Configure routing and security using the Principle of Least Privilege.
* Consider AWS Direct Connect when consistent bandwidth and predictable network performance are business requirements.
* Consider using Site-to-Site VPN as backup connectivity for Direct Connect architectures.
* Regularly test VPN failover.

---

# 📊 Site-to-Site VPN vs Direct Connect

| Feature                    | Site-to-Site VPN            | AWS Direct Connect             |
| -------------------------- | --------------------------- | ------------------------------ |
| Connectivity               | Encrypted VPN               | Dedicated network connection   |
| Transport                  | Typically Public Internet   | Dedicated connectivity         |
| Setup Time                 | Relatively Fast             | Longer                         |
| IPsec Encryption           | ✅ Yes                       | Not provided by DX itself      |
| Performance Predictability | Lower                       | Higher                         |
| Cost                       | Generally Lower             | Generally Higher               |
| Typical Use                | Hybrid connectivity, backup | Enterprise hybrid connectivity |

---

# ❓ Interview Questions

### Q1. What is AWS Site-to-Site VPN?

**Answer**

AWS Site-to-Site VPN provides secure connectivity between an on-premises or external network and AWS using encrypted IPsec VPN tunnels.

---

### Q2. What is a Customer Gateway?

**Answer**

A Customer Gateway is an AWS resource that represents the Customer Gateway Device located in the customer network.

The actual Customer Gateway Device can be a physical router, firewall, or software VPN appliance.

---

### Q3. What is a Virtual Private Gateway?

**Answer**

A Virtual Private Gateway is an AWS-side VPN endpoint that can be attached to a VPC.

---

### Q4. How many tunnels does an AWS Site-to-Site VPN connection provide?

**Answer**

Each Site-to-Site VPN connection provides **two VPN tunnels** for redundancy and High Availability.

Both tunnels should be configured on the Customer Gateway Device.

---

### Q5. Does Site-to-Site VPN require BGP?

**Answer**

No.

AWS Site-to-Site VPN supports both Static and Dynamic Routing.

Dynamic Routing uses BGP, while Static Routing can be used when the Customer Gateway Device does not support BGP.

---

### Q6. What happens if the Customer Gateway is behind NAT?

**Answer**

For a standard public IPv4 VPN, the public IP address of the NAT device is provided when creating the Customer Gateway resource, and NAT Traversal must be supported and appropriately configured.

---

### Q7. What is the bandwidth of an AWS Site-to-Site VPN?

**Answer**

A standard VPN tunnel supports up to **1.25 Gbps per tunnel**.

AWS also supports Large Bandwidth Tunnels of up to **5 Gbps per tunnel** for supported Transit Gateway and Cloud WAN VPN connections.

---

### Q8. What is the difference between Site-to-Site VPN and Direct Connect?

**Answer**

Site-to-Site VPN typically uses the public Internet with IPsec encryption, while AWS Direct Connect provides dedicated network connectivity between the customer network and AWS.

Direct Connect generally provides more predictable network performance.

---

# 💡 Key Takeaways

* AWS Site-to-Site VPN connects AWS with on-premises or external networks.
* Traffic is protected using **IPsec encrypted tunnels**.
* A VPN connection provides **two tunnels** for redundancy.
* The customer side uses a **Customer Gateway Device**.
* The AWS side can use a **Virtual Private Gateway or Transit Gateway**.
* Site-to-Site VPN supports both **Static and Dynamic (BGP) Routing**.
* Standard VPN tunnels support up to **1.25 Gbps per tunnel**.
* Large Bandwidth Tunnels can support up to **5 Gbps per tunnel** in supported architectures.
* Internet conditions can affect standard VPN performance.
* Direct Connect should be considered when more predictable dedicated connectivity is required.

---

# 📚 Related Topics

* Amazon VPC
* CIDR & Subnetting
* Route Tables
* Security Groups
* Network ACLs
* VPC Peering
* AWS Transit Gateway
* AWS Direct Connect
* Border Gateway Protocol (BGP)
* Hybrid Cloud Networking

---

# 📖 References

* AWS Site-to-Site VPN Documentation
* AWS Site-to-Site VPN Concepts
* AWS Site-to-Site VPN Routing Options
* AWS Customer Gateway Documentation
* AWS Site-to-Site VPN Tunnel Options
* AWS Site-to-Site VPN Pricing
