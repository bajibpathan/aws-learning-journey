# 🌐 Amazon VPC Subnets

> Learn how Amazon VPC Subnets divide a VPC into smaller networks, enabling secure workload isolation, high availability, and scalable cloud architectures.

---

# 📖 Overview

A **Subnet** is a logical subdivision of an Amazon VPC. Each subnet is associated with a single Availability Zone (AZ) and is assigned a portion of the VPC's CIDR block.

Subnets allow you to organize resources based on security, application tiers, and availability requirements.

By combining multiple subnets, route tables, and gateways, you can build secure and highly available AWS architectures.

---

# 📊 Key Components

| Component | Purpose |
|-----------|----------|
| Subnet | Divides a VPC into smaller logical networks |
| Availability Zone | A subnet resides within a single AZ |
| CIDR Block | Defines the IP address range of the subnet |
| Route Table | Controls network traffic for the subnet |
| Public Subnet | Has a route to an Internet Gateway |
| Private Subnet | Does not have direct Internet access |

---

# 🏗 What is a Subnet?

A **Subnet** is a smaller network created from a VPC's CIDR block.

It allows you to:

- Organize workloads.
- Isolate application tiers.
- Improve security.
- Build highly available architectures.

Example:

```text
VPC (10.0.0.0/16)

├── Public Subnet (10.0.1.0/24)
├── Private App Subnet (10.0.2.0/24)
├── Private DB Subnet (10.0.3.0/24)
```

---

# 🌍 Availability Zones

A subnet belongs to **only one Availability Zone**.

However, a single Availability Zone can contain **multiple subnets**.

For High Availability, production workloads should be distributed across multiple Availability Zones.

Example:

```text
VPC

├── AZ-a
│     ├── Public Subnet
│     ├── Private App Subnet
│     └── Private DB Subnet
│
├── AZ-b
│     ├── Public Subnet
│     ├── Private App Subnet
│     └── Private DB Subnet
```

---

# 📍 CIDR Blocks

Every subnet is assigned a **subset** of the VPC's CIDR block.

Example:

```text
VPC

10.0.0.0/16

Public Subnet

10.0.1.0/24

Private App Subnet

10.0.2.0/24

Private DB Subnet

10.0.3.0/24
```

Rules:

- Subnet CIDR blocks must belong to the VPC CIDR range.
- Subnet CIDR blocks must **not overlap**.
- Each subnet must have its own unique IP address range.

---

# 🔄 Route Table Association

Every subnet must be associated with a **Route Table**.

If a Route Table is not explicitly associated, AWS automatically associates the subnet with the **Main Route Table**.

Route Tables determine where network traffic is routed.

---

# 🌐 Public vs Private Subnets

## Public Subnet

A Public Subnet has a route to an **Internet Gateway (IGW)**.

Typically hosts:

- Internet-facing Load Balancers
- Bastion Hosts
- NAT Gateways
- Public Web Servers

---

## Private Subnet

A Private Subnet does **not** have a direct route to the Internet Gateway.

Typically hosts:

- Application Servers
- Databases
- Internal Services
- Backend APIs

Private resources can access the Internet through a **NAT Gateway** if required.

---

# 🚫 AWS Reserved IP Addresses

AWS reserves **5 IP addresses** in every subnet.

| Reserved Address | Purpose |
|------------------|----------|
| First IP | Network Address |
| Second IP | VPC Router |
| Third IP | Amazon DNS |
| Fourth IP | Reserved by AWS |
| Last IP | Reserved by AWS |

These addresses cannot be assigned to resources.

---

# 🔧 DHCP Options

Every VPC is associated with a **DHCP Options Set**.

You can:

- Use the AWS Default DHCP Options Set.
- Associate a Custom DHCP Options Set with the VPC.

Only one DHCP Options Set can be associated with a VPC at a time.

---

# 🎯 Why Use Subnets?

Subnets help you:

- Organize workloads into logical network boundaries.
- Separate public and private resources.
- Improve network security.
- Deploy workloads across multiple Availability Zones.
- Improve scalability and fault tolerance.

Without subnets:

- All resources would share one network.
- Security would be harder to manage.
- Network organization would become complex.

---

# ✅ Best Practices

- Design subnet CIDR blocks for future growth.
- Never create overlapping subnet CIDR ranges.
- Deploy production workloads across multiple Availability Zones.
- Place Internet-facing resources in Public Subnets.
- Keep databases and backend services in Private Subnets.
- Consider the 5 AWS-reserved IP addresses when planning subnet sizes.
- Associate the correct Route Table with every subnet.
- Follow the Principle of Least Privilege using Security Groups and Network ACLs.

---

# 📊 Public vs Private Subnets

| Feature | Public Subnet | Private Subnet |
|----------|---------------|----------------|
| Internet Gateway Route | ✅ Yes | ❌ No |
| Public IP | Usually Yes | Usually No |
| Internet Access | Direct | Via NAT Gateway |
| Typical Resources | Load Balancer, Bastion Host | App Servers, Databases |

---

# ❓ Interview Questions

### Q1. Can a subnet span multiple Availability Zones?

**Answer**

No. A subnet belongs to a single Availability Zone.

---

### Q2. Can multiple subnets exist in the same Availability Zone?

**Answer**

Yes. AWS allows multiple subnets within the same Availability Zone.

---

### Q3. Can two subnet CIDR blocks overlap?

**Answer**

No. Subnet CIDR blocks within a VPC must be unique and cannot overlap.

---

### Q4. What makes a subnet Public?

**Answer**

A subnet becomes Public when its Route Table contains a route to an Internet Gateway.

---

### Q5. What makes a subnet Private?

**Answer**

A Private Subnet does not have a route to an Internet Gateway.

---

### Q6. How many IP addresses does AWS reserve in every subnet?

**Answer**

AWS reserves five IP addresses in every subnet for networking purposes.

---

### Q7. Can resources in different subnets communicate?

**Answer**

Yes. Subnets within the same VPC can communicate by default through the VPC's local route.

---

# 💡 Key Takeaways

- A subnet is a logical subdivision of a VPC.
- Every subnet belongs to a single Availability Zone.
- Multiple subnets can exist within the same Availability Zone.
- Public and Private Subnets improve security and workload isolation.
- Every subnet must be associated with a Route Table.
- AWS reserves five IP addresses in every subnet.
- Proper subnet design is essential for building secure, scalable, and highly available AWS architectures.

---

# 📚 Related Topics

- Network Fundamentals
- CIDR, Subnet Mask & Subnetting
- Amazon VPC
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html