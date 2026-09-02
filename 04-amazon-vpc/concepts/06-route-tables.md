# 🛣️ Amazon VPC Route Tables

> Learn how Amazon VPC Route Tables control network traffic, enable Internet access for Public Subnets, and isolate Private Subnets for secure cloud architectures.

---

# 📖 Overview

A **Route Table** is a set of rules (routes) that determines where network traffic is directed within an Amazon VPC.

Every subnet in a VPC must be associated with a Route Table. Based on the configured routes, resources can communicate:

- Within the VPC
- With the Internet
- With other VPCs
- With on-premises networks

Proper Route Table design is essential for building secure, scalable, and highly available AWS architectures.

---

# 📊 Key Components

| Component | Purpose |
|-----------|----------|
| Route Table | Controls how network traffic is routed |
| Route | Defines the destination and target |
| Local Route | Enables communication within the VPC |
| Internet Gateway | Provides Internet connectivity for Public Subnets |
| NAT Gateway | Provides outbound Internet access for Private Subnets |
| Route Table Association | Associates a Route Table with one or more subnets |

---

# 🛣️ What is a Route Table?

A **Route Table** contains a collection of routing rules that determine where network traffic should be sent.

Every VPC automatically includes a **Main Route Table**.

The Main Route Table contains a **Local Route**, allowing communication between all subnets within the VPC.

Example:

```text
Destination        Target

10.0.0.0/16   ---> Local
```

This Local Route is automatically created by AWS and cannot be removed.

---

# 🔗 Route Table Association

Every subnet must be associated with a Route Table.

If you do not explicitly associate a subnet with a custom Route Table, AWS automatically associates it with the **Main Route Table**.

A Route Table can be associated with multiple subnets.

---

# 🌐 Public Route Table

To allow resources in a Public Subnet to communicate with the Internet, create a Route Table containing a route to the Internet Gateway.

Example:

```text
Destination        Target

10.0.0.0/16   ---> Local
0.0.0.0/0     ---> Internet Gateway
```

The subnet associated with this Route Table becomes a **Public Subnet**.

Resources also require a **Public IP Address** or **Elastic IP Address** to communicate with the Internet.

---

# 🔒 Private Route Table

Private Subnets should not have a route to the Internet Gateway.

If outbound Internet access is required, traffic is routed through a **NAT Gateway**.

Example:

```text
Destination        Target

10.0.0.0/16   ---> Local
0.0.0.0/0     ---> NAT Gateway
```

This allows outbound Internet access while preventing inbound Internet connections.

---

# 🌍 Auto-Assign Public IP

Public Subnets can be configured to automatically assign Public IPv4 addresses to newly launched EC2 instances.

This setting simplifies deployment of Internet-facing resources.

---

# 🎯 Why Use Route Tables?

Route Tables help you:

- Control how network traffic flows.
- Enable communication between subnets.
- Provide Internet connectivity for Public Subnets.
- Keep Private Subnets isolated.
- Support hybrid networking and VPC connectivity.
- Build secure and scalable network architectures.

Without Route Tables, AWS resources would not know where to send network traffic.

---

# ✅ Best Practices

- Use separate Route Tables for Public and Private Subnets.
- Associate only Public Subnets with Route Tables that contain Internet Gateway routes.
- Keep databases and backend services in Private Subnets.
- Use NAT Gateways instead of Internet Gateways for Private Subnets requiring outbound Internet access.
- Keep Route Tables simple and easy to manage.
- Document routing configurations for easier troubleshooting.
- Follow the Principle of Least Privilege when designing network routing.

---

# 📊 Route Table Comparison

| Feature | Public Route Table | Private Route Table |
|----------|-------------------|---------------------|
| Local Route | ✅ | ✅ |
| Internet Gateway Route | ✅ | ❌ |
| NAT Gateway Route | ❌ | ✅ (Optional) |
| Internet Access | Direct | Outbound Only |
| Typical Resources | Web Servers, Bastion Hosts, Load Balancers | Application Servers, Databases |

---

# ❓ Interview Questions

### Q1. What is a Route Table?

**Answer**

A Route Table is a collection of routing rules that determines where network traffic is directed within an Amazon VPC.

---

### Q2. What is the Main Route Table?

**Answer**

Every VPC includes a Main Route Table that contains a Local Route, allowing communication between all subnets within the VPC.

---

### Q3. What makes a subnet Public?

**Answer**

A subnet becomes Public when its associated Route Table contains a route to an Internet Gateway.

---

### Q4. Can multiple subnets use the same Route Table?

**Answer**

Yes. A single Route Table can be associated with multiple subnets.

---

### Q5. Can a subnet be associated with multiple Route Tables?

**Answer**

No. A subnet can be associated with only one Route Table at a time.

---

### Q6. Is a Public IP address alone enough for Internet access?

**Answer**

No. Internet access also requires:
- An Internet Gateway attached to the VPC.
- A Route Table with a route to the Internet Gateway.
- A Public Subnet.
- A Public IP Address or Elastic IP Address.

---

# 💡 Key Takeaways

- Route Tables control how network traffic flows within a VPC.
- Every subnet must be associated with a Route Table.
- The Main Route Table contains a Local Route by default.
- Public Route Tables include a route to an Internet Gateway.
- Private Route Tables typically route Internet-bound traffic through a NAT Gateway.
- Proper Route Table design is essential for secure, scalable, and highly available AWS networking.

---

# 📚 Related Topics

- Network Fundamentals
- CIDR, Subnet Mask & Subnetting
- Amazon VPC
- Amazon VPC Subnets
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html
- https://docs.aws.amazon.com/vpc/latest/userguide/subnet-route-tables.html
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html