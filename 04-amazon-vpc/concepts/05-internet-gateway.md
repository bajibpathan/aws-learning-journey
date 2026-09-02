# 🌍 Amazon Internet Gateway (IGW)

> Learn what an Internet Gateway (IGW) is, how it enables Internet connectivity for Amazon VPC resources, and the requirements for building Internet-facing applications on AWS.

---

# 📖 Overview

An **Internet Gateway (IGW)** is a highly available and fully managed AWS service that enables communication between an Amazon VPC and the Internet.

By attaching an Internet Gateway to a VPC and configuring the appropriate Route Tables, resources in Public Subnets can securely send and receive Internet traffic.

Internet Gateways are commonly used for:

- Public Web Applications
- Bastion Hosts
- Internet-facing Load Balancers
- Resources that require outbound Internet access

---

# 📊 Key Components

| Component | Purpose |
|-----------|----------|
| Internet Gateway | Connects a VPC to the Internet |
| Public Subnet | Contains resources that require Internet access |
| Route Table | Directs Internet traffic to the Internet Gateway |
| Public IP Address | Allows a resource to communicate with the Internet |
| Elastic IP Address | Static Public IP Address for AWS resources |

---

# 🌐 What is an Internet Gateway?

An **Internet Gateway (IGW)** is a VPC component that enables communication between a VPC and the Internet.

An Internet Gateway:

- Is a fully managed AWS service.
- Is highly available and horizontally scalable.
- Must be attached to a VPC.
- Enables inbound and outbound Internet connectivity.
- Supports both IPv4 and IPv6 traffic.

An Internet Gateway itself does **not** make a subnet public.

Additional networking components are required.

---

# ⚙️ Requirements for Internet Access

For an EC2 instance to communicate with the Internet, all of the following must exist:

- An Internet Gateway attached to the VPC.
- A Route Table containing a route to the Internet Gateway.
- A Public Subnet associated with that Route Table.
- A Public IPv4 Address or Elastic IP Address assigned to the EC2 instance.

Example Route:

```text
Destination     Target
0.0.0.0/0  ---> Internet Gateway
```

Without any one of these components, Internet connectivity will not work.

---

# 🌍 How Internet Gateway Works

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2 Instance (Public IP)
```

Traffic flows through the Internet Gateway to reach Internet-facing resources inside the VPC.

---

# 🎯 Why Use an Internet Gateway?

An Internet Gateway is required when resources need to communicate with the Internet.

Common use cases include:

- Hosting public websites.
- Downloading software updates.
- Accessing third-party APIs.
- Allowing users to access Internet-facing applications.
- Managing EC2 instances using Public IP addresses.

Without an Internet Gateway, resources inside a VPC cannot directly communicate with the Internet.

---

# ✅ Best Practices

- Attach only one Internet Gateway to a VPC.
- Use Public Subnets only for Internet-facing resources.
- Keep application servers and databases in Private Subnets.
- Restrict inbound traffic using Security Groups and Network ACLs.
- Assign Public IP addresses only to resources that require Internet access.
- Use Elastic IP addresses only when static Public IPs are required.
- Follow the Principle of Least Privilege for network access.

---

# 📊 Internet Gateway Checklist

| Requirement | Required |
|-------------|----------|
| Internet Gateway attached to VPC | ✅ |
| Route Table contains `0.0.0.0/0 → IGW` | ✅ |
| Public Subnet | ✅ |
| Public IP or Elastic IP | ✅ |

---

# ❓ Interview Questions

### Q1. What is an Internet Gateway?

**Answer**

An Internet Gateway is a highly available and fully managed AWS service that enables communication between an Amazon VPC and the Internet.

---

### Q2. Does attaching an Internet Gateway automatically provide Internet access?

**Answer**

No. Internet access also requires a Route Table pointing to the Internet Gateway, a Public Subnet, and a Public IP address.

---

### Q3. Can a Private Subnet use an Internet Gateway directly?

**Answer**

No. A Private Subnet does not have a route to the Internet Gateway. It typically accesses the Internet through a NAT Gateway.

---

### Q4. How many Internet Gateways can be attached to a VPC?

**Answer**

Only one Internet Gateway can be attached to a VPC at a time.

---

### Q5. Does an Internet Gateway perform Network Address Translation (NAT)?

**Answer**

Yes. It performs one-to-one NAT for instances with Public IPv4 addresses, allowing them to communicate with the Internet.

---

# 💡 Key Takeaways

- An Internet Gateway connects an Amazon VPC to the Internet.
- It must be attached to a VPC before it can be used.
- Internet connectivity requires an Internet Gateway, Route Table, Public Subnet, and Public IP address.
- Internet Gateways are used only for Internet-facing resources.
- Private resources should remain in Private Subnets and use a NAT Gateway when outbound Internet access is required.

---

# 📚 Related Topics

- Amazon VPC
- Amazon VPC Subnets
- Route Tables
- NAT Gateway
- Security Groups
- Network ACLs
- Elastic IP Address

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario1.html
- https://docs.aws.amazon.com/vpc/latest/userguide/working-with-igw.html