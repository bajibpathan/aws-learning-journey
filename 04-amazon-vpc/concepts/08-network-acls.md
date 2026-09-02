# 🛡️ Amazon VPC Network Access Control Lists (NACLs)

> Learn how Network Access Control Lists (NACLs) provide subnet-level security by controlling inbound and outbound traffic using stateless firewall rules.

---

# 📖 Overview

A **Network Access Control List (NACL)** is a stateless firewall that controls network traffic entering and leaving an Amazon VPC subnet.

Unlike Security Groups, which secure individual resources, NACLs secure the **entire subnet** by evaluating inbound and outbound traffic separately.

NACLs provide an additional layer of security and are commonly used to enforce network-wide security policies.

---

# 📊 Key Components

| Component | Purpose |
|-----------|----------|
| Network ACL | Controls traffic at the subnet level |
| Inbound Rules | Control traffic entering the subnet |
| Outbound Rules | Control traffic leaving the subnet |
| Rule Number | Determines the order in which rules are evaluated |
| Allow Rule | Explicitly permits traffic |
| Deny Rule | Explicitly blocks traffic |

---

# 🛡️ What is a Network ACL?

A **Network Access Control List (NACL)** is a subnet-level firewall that controls inbound and outbound traffic.

A Network ACL:

- Operates at the **Subnet** level.
- Is **Stateless**, meaning return traffic is **not automatically allowed**.
- Requires separate **Inbound** and **Outbound** rules.
- Supports both **Allow** and **Deny** rules.
- Evaluates rules in ascending numerical order.
- Stops processing once the first matching rule is found.

---

# 🌍 Default vs Custom NACL

Every Amazon VPC automatically includes a **Default Network ACL**.

## Default NACL

- Automatically associated with all new subnets unless a custom NACL is assigned.
- Allows all inbound traffic.
- Allows all outbound traffic.

## Custom NACL

- Created by the user.
- Denies all inbound and outbound traffic by default.
- Traffic is allowed only after explicitly creating rules.

---

# 🔢 Rule Evaluation

Each rule is assigned a **Rule Number**.

Rules are evaluated from the **lowest number to the highest number**.

The first matching rule is applied, and AWS stops evaluating additional rules.

Example:

```text
100  Allow HTTP (80)

200  Allow HTTPS (443)

300  Deny All
```

AWS recommends numbering rules in increments (100, 200, 300...) to make future modifications easier.

---

# 🔄 Stateless Firewall

Because NACLs are **Stateless**, return traffic must be explicitly allowed.

Example:

If an inbound HTTP request is allowed, the corresponding outbound response must also be allowed.

This differs from Security Groups, which automatically allow return traffic.

---

# 🔗 NACL Association

A subnet can be associated with **only one Network ACL**.

However, a single Network ACL can be associated with **multiple subnets**.

This allows consistent security policies to be applied across related subnets.

---

# 🎯 Why Use Network ACLs?

Network ACLs help you:

- Control traffic entering and leaving a subnet.
- Add an additional layer of network security.
- Explicitly allow or deny traffic.
- Protect multiple resources within a subnet using centralized rules.
- Enforce organization-wide network security policies.

Without Network ACLs, security would rely entirely on Security Groups attached to individual resources.

---

# ✅ Best Practices

- Use Network ACLs as an additional layer of defense alongside Security Groups.
- Keep Security Groups as the primary instance-level firewall.
- Number rules in increments (100, 200, 300...) for easier maintenance.
- Allow only the required inbound and outbound traffic.
- Use explicit **Deny** rules when required.
- Avoid creating unnecessary or overly broad rules.
- Follow the Principle of Least Privilege.

---

# 📊 Network ACL vs Security Group

| Feature | Network ACL | Security Group |
|----------|-------------|----------------|
| Applied At | Subnet | Instance / ENI |
| Stateful | ❌ No | ✅ Yes |
| Supports Allow Rules | ✅ | ✅ |
| Supports Deny Rules | ✅ | ❌ |
| Return Traffic | Must be explicitly allowed | Automatically allowed |
| Rule Evaluation | Rule Number Order | All Rules Evaluated |

---

# ❓ Interview Questions

### Q1. What is a Network ACL?

**Answer**

A Network ACL is a stateless firewall that controls inbound and outbound traffic at the subnet level.

---

### Q2. Is a Network ACL Stateful or Stateless?

**Answer**

A Network ACL is **Stateless**, meaning return traffic must be explicitly allowed.

---

### Q3. Does a Network ACL support Deny rules?

**Answer**

Yes. Network ACLs support both **Allow** and **Deny** rules.

---

### Q4. How are Network ACL rules evaluated?

**Answer**

Rules are evaluated in ascending numerical order. The first matching rule is applied.

---

### Q5. How many Network ACLs can be associated with a subnet?

**Answer**

A subnet can be associated with only **one Network ACL**, but a Network ACL can be associated with multiple subnets.

---

### Q6. What is the difference between the Default and Custom Network ACL?

**Answer**

The Default Network ACL allows all inbound and outbound traffic, whereas a Custom Network ACL denies all traffic until rules are explicitly configured.

---

# 💡 Key Takeaways

- Network ACLs provide subnet-level security.
- They are stateless, so inbound and outbound rules must both be configured.
- NACLs support both Allow and Deny rules.
- Rules are processed in ascending numerical order.
- A subnet can have only one Network ACL, while a Network ACL can protect multiple subnets.
- NACLs provide an additional layer of security alongside Security Groups.

---

# 📚 Related Topics

- Amazon VPC
- Amazon VPC Subnets
- Internet Gateway
- Route Tables
- NAT Gateway
- Security Groups
- VPC Endpoints

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html
- https://docs.aws.amazon.com/vpc/latest/userguide/default-network-acl.html
- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-rules