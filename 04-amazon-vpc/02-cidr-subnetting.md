# 🌐 CIDR, Subnet Mask & Subnetting

> Learn how IPv4 addresses are divided into networks, how subnet masks and CIDR notation work, and why subnetting is essential for designing scalable networks and AWS VPCs.

---

# 📖 Overview

Every device connected to a network requires a unique IP address. An IPv4 address contains information about both the network and the device within that network.

To efficiently utilize IP addresses and organize networks, networking uses:

- IPv4 Addressing
- Subnet Masks
- CIDR (Classless Inter-Domain Routing)
- Subnetting

These concepts form the foundation of AWS networking and are used when designing Amazon VPCs and Subnets.

---

# 📊 Key Components

| Component | Purpose |
|-----------|----------|
| IPv4 Address | Identifies a device on a network |
| Network Portion | Identifies the network |
| Host Portion | Identifies the device within the network |
| Subnet Mask | Separates the Network and Host portions |
| CIDR | Defines the network size using a prefix length |
| Subnetting | Divides a network into smaller subnetworks |

---

# 🌍 IPv4 Address

## What is an IPv4 Address?

An **IPv4 address** is a **32-bit** address made up of binary digits (0s and 1s).

Its primary purpose is to:

- Identify a device on a network.
- Identify the network the device belongs to.
- Enable communication between devices.

Example:

```text
192.168.1.10/24
```

An IPv4 address consists of two parts:

- **Network Portion** – Identifies the network.
- **Host Portion** – Identifies the device within that network.

Example:

```text
192.168.1.10/24

Network Portion : 192.168.1
Host Portion    : 10
```

---

# 🎭 Subnet Mask

## What is a Subnet Mask?

A **Subnet Mask** is a **32-bit number** that separates an IP address into the Network Portion and the Host Portion.

It consists of:

- **1s** representing the Network Portion.
- **0s** representing the Host Portion.

Common subnet masks:

| Prefix | Subnet Mask |
|---------|-------------|
| /8 | 255.0.0.0 |
| /16 | 255.255.0.0 |
| /24 | 255.255.255.0 |

Traditional class-based subnet masks:

| Class | Subnet Mask |
|--------|-------------|
| Class A | 255.0.0.0 |
| Class B | 255.255.0.0 |
| Class C | 255.255.255.0 |

---

# 📍 CIDR (Classless Inter-Domain Routing)

## What is CIDR?

CIDR (Classless Inter-Domain Routing) is a method of representing IP addresses and network sizes using a **Prefix Length** instead of traditional class-based addressing.

CIDR notation:

```text
<IP Address>/<Prefix Length>
```

Example:

```text
192.168.1.0/24
```

Where:

- `192.168.1.0` → Network Address
- `/24` → Prefix Length

---

## Prefix Length

The Prefix Length determines how many bits belong to the Network Portion.

Example:

```text
192.168.1.0/24
```

- Network Bits = 24
- Host Bits = 8

A larger prefix means:

- Smaller network
- Fewer hosts

A smaller prefix means:

- Larger network
- More hosts

---

# ✂️ Subnetting

## What is Subnetting?

Subnetting is the process of dividing a large network into multiple smaller subnetworks.

This is done by borrowing bits from the Host Portion and adding them to the Network Portion.

Benefits include:

- Better IP address utilization
- Reduced broadcast traffic
- Improved performance
- Improved security
- Easier network management

Without subnetting:

- Large broadcast domains
- Poor scalability
- Difficult network management
- Lower security

---

## Subnetting Formula

### Number of Subnets

```text
2^(Borrowed Bits)
```

### Number of IP Addresses per Subnet

```text
2^(Host Bits)
```

---

# 🚫 Reserved IP Addresses

Every subnet reserves two IP addresses:

| Address | Purpose |
|----------|----------|
| First IP | Network Address (Network ID) |
| Last IP | Broadcast Address |

These IP addresses cannot be assigned to hosts.

> **AWS Note:** In Amazon VPC, AWS reserves the first **4 IP addresses** and the last **1 IP address** in every subnet.

---

# 🎯 Why Are These Concepts Important?

These networking concepts help you:

- Design efficient IP address ranges.
- Divide large networks into smaller subnetworks.
- Reduce broadcast traffic.
- Improve network performance.
- Improve network security.
- Design scalable AWS VPC architectures.

Without understanding CIDR and subnetting, it is difficult to work with Amazon VPCs, Subnets, Route Tables, and networking services.

---

# ✅ Best Practices

- Use CIDR instead of traditional class-based addressing.
- Design subnet sizes based on current and future requirements.
- Avoid allocating unnecessarily large CIDR blocks.
- Keep public and private resources in separate subnets.
- Document CIDR allocations to prevent overlapping networks.
- Leave room for future expansion when designing networks.

---

# 📊 CIDR Quick Reference

| CIDR | Subnet Mask | Total IPs |
|------:|-------------|----------:|
| /8 | 255.0.0.0 | 16,777,216 |
| /16 | 255.255.0.0 | 65,536 |
| /24 | 255.255.255.0 | 256 |
| /25 | 255.255.255.128 | 128 |
| /26 | 255.255.255.192 | 64 |
| /27 | 255.255.255.224 | 32 |
| /28 | 255.255.255.240 | 16 |

---

# ❓ Interview Questions

### Q1. What is the purpose of a Subnet Mask?

**Answer**

A Subnet Mask separates an IP address into the Network Portion and the Host Portion, allowing devices to determine whether communication is within the same network or requires a router.

---

### Q2. What does `/24` mean in CIDR notation?

**Answer**

`/24` means the first 24 bits represent the Network Portion, while the remaining 8 bits represent the Host Portion.

---

### Q3. Why is subnetting used?

**Answer**

Subnetting improves IP address utilization, reduces broadcast traffic, improves security, and simplifies network management.

---

### Q4. What is CIDR?

**Answer**

CIDR (Classless Inter-Domain Routing) is a method of defining IP address ranges using a prefix length instead of fixed network classes.

---

### Q5. What IP addresses are reserved in every subnet?

**Answer**

Traditionally, the first IP address is reserved as the Network Address and the last IP address is reserved as the Broadcast Address. In AWS, the first four IP addresses and the last IP address of each subnet are reserved.

---

# 💡 Key Takeaways

- Every IPv4 address contains a Network Portion and a Host Portion.
- A Subnet Mask determines where the Network Portion ends and the Host Portion begins.
- CIDR provides flexible network sizing using prefix lengths.
- Subnetting divides large networks into smaller, more manageable subnetworks.
- Proper subnet design improves scalability, performance, security, and IP address utilization.
- CIDR and subnetting are fundamental concepts for designing Amazon VPCs and AWS networking architectures.

---

# 📚 Related Topics

- Network Fundamentals
- Amazon VPC
- Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs

---

# 📖 References

- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html
- https://www.rfc-editor.org/rfc/rfc4632
- https://datatracker.ietf.org/doc/html/rfc4632