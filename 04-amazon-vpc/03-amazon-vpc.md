# 🌐 Amazon Virtual Private Cloud (Amazon VPC)

> Learn what an Amazon Virtual Private Cloud (VPC) is, why it is the foundation of AWS networking, and how to design secure, scalable, and isolated cloud environments.

---

# 📖 Overview

An **Amazon Virtual Private Cloud (Amazon VPC)** is a logically isolated virtual network within AWS where you can launch AWS resources such as Amazon EC2, Amazon RDS, and Amazon ElastiCache.

It provides complete control over your networking environment, including:

- IP Address Ranges
- Subnets
- Route Tables
- Internet Connectivity
- Security
- DNS Settings

Unlike the Default VPC, a Custom VPC allows you to design your network based on your application's security, scalability, and compliance requirements.

---

# 📊 VPC Types

| VPC Type | Description | Best For |
|----------|-------------|----------|
| **Default VPC** | Automatically created by AWS with internet connectivity | Learning, Testing |
| **Custom VPC** | User-created VPC with custom networking configuration | Production Workloads |

---

# 🏗 What is a Custom VPC?

A **Custom VPC** is a Virtual Private Cloud created by the user with custom networking configurations instead of using the AWS-created Default VPC.

A Custom VPC provides complete control over:

- IP Address Planning
- Subnet Design
- Routing
- Internet Connectivity
- Security
- Hybrid Connectivity

Unlike the Default VPC, a Custom VPC **does not provide Internet access by default**.

---

# 🌍 Key Characteristics

### Regional Resource

- A VPC is a **Regional resource**.
- It spans all Availability Zones (AZs) within the selected AWS Region.
- You can create subnets in one or more Availability Zones.

---

### Default Router

Every VPC automatically includes a **Virtual Router**.

The router enables communication between subnets within the VPC without requiring additional configuration.

---

### Internet Connectivity

A Custom VPC does **not** have Internet connectivity by default.

To enable Internet access, you must configure:

- Internet Gateway (IGW)
- Route Tables
- Public Subnets

---

### Hybrid Networking

A Custom VPC can securely connect to:

- On-Premises Data Centers
- Other AWS VPCs
- Multi-Cloud Environments

Using services such as:

- AWS Site-to-Site VPN
- AWS Direct Connect
- VPC Peering
- AWS Transit Gateway

---

### Instance Tenancy

When creating a VPC, you can choose one of two tenancy options.

| Tenancy | Description |
|----------|-------------|
| **Default** | Instances share the underlying AWS hardware with other customers (Recommended) |
| **Dedicated** | Instances run on hardware dedicated to a single AWS customer |

> **Recommendation:** Use **Default Tenancy** unless Dedicated Tenancy is required for compliance or licensing reasons.

---

### DNS Settings

Two important DNS settings are available within a VPC.

#### enableDnsSupport

- Enables DNS resolution within the VPC.
- Allows resources to use the Amazon-provided DNS server.

#### enableDnsHostnames

- Assigns Public DNS hostnames to EC2 instances that have Public IP addresses.
- Required for publicly accessible workloads.

---

### Reserved IP Addresses

AWS reserves **5 IP addresses in every subnet**.

These addresses are used for AWS networking purposes and cannot be assigned to resources.

---

# 🎯 Why Use a Custom VPC?

A Custom VPC allows you to:

- Build isolated environments.
- Design secure network boundaries.
- Meet compliance and regulatory requirements.
- Separate Development, Testing, and Production workloads.
- Build highly available and scalable architectures.
- Connect AWS with on-premises environments.

---

# ✅ Best Practices

- Use **Custom VPCs** for production workloads instead of the Default VPC.
- Plan the VPC CIDR block carefully to avoid overlapping IP ranges.
- Design Public and Private Subnets based on application requirements.
- Consider the **5 AWS-reserved IP addresses** when planning subnet sizes.
- Use **Default Tenancy** unless Dedicated Tenancy is specifically required.
- Enable DNS Support and DNS Hostnames when applications require DNS resolution.
- Design for High Availability by distributing workloads across multiple Availability Zones.

---

# 📊 Default VPC vs Custom VPC

| Feature | Default VPC | Custom VPC |
|----------|-------------|------------|
| Created By | AWS | User |
| Internet Access | Yes | No (Requires Internet Gateway) |
| Public Subnets | Created Automatically | User Creates |
| Public IP Assignment | Enabled by Default | Configurable |
| Production Ready | Not Recommended | Recommended |
| Networking Control | Limited | Full Control |

---

# ❓ Interview Questions

### Q1. What is an Amazon VPC?

**Answer**

An Amazon VPC is a logically isolated virtual network in AWS where you can securely deploy AWS resources with complete control over networking, security, and routing.

---

### Q2. Is a VPC Regional or Availability Zone specific?

**Answer**

A VPC is a **Regional resource** and spans all Availability Zones within that Region.

---

### Q3. Does a Custom VPC have Internet access by default?

**Answer**

No. A Custom VPC requires an Internet Gateway and appropriate Route Tables to enable Internet connectivity.

---

### Q4. What is the difference between Default Tenancy and Dedicated Tenancy?

**Answer**

Default Tenancy allows instances to run on shared AWS hardware, while Dedicated Tenancy runs instances on hardware dedicated to a single AWS customer.

---

### Q5. How many IP addresses does AWS reserve in every subnet?

**Answer**

AWS reserves **5 IP addresses** in every subnet for networking purposes.

---

# 💡 Key Takeaways

- Amazon VPC is the foundation of AWS networking.
- A Custom VPC provides complete control over networking, routing, security, and Internet connectivity.
- A VPC is a Regional resource that spans multiple Availability Zones.
- Custom VPCs do not provide Internet access by default.
- Every VPC includes a Virtual Router for internal routing.
- AWS reserves 5 IP addresses in every subnet.
- A well-designed Custom VPC improves security, scalability, and availability.

---

# 📚 Related Topics

- Network Fundamentals
- CIDR, Subnet Mask & Subnetting
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs
- VPC Peering
- AWS Transit Gateway

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html