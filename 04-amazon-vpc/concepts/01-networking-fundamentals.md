# 🌐 Network Fundamentals

> Learn the core networking concepts required to understand cloud networking and AWS services such as Amazon VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways, and Security Groups.

---

# 📖 Overview

Networking enables devices to communicate and share resources with each other. Every device connected to a network is identified by a unique IP address, allowing data to be transmitted between systems.

Understanding networking fundamentals is essential before learning AWS networking services.

Key networking concepts include:

- IP Addressing
- Public and Private Networks
- DHCP
- Routers
- NAT (Network Address Translation)
- Port Numbers

---

# 📊 Network Components

| Component | Purpose |
|-----------|----------|
| IP Address | Uniquely identifies a device on a network |
| DHCP | Automatically assigns IP addresses |
| Router | Connects different networks |
| NAT | Allows Private IPs to access the Internet using a Public IP |
| Port Number | Identifies a specific application or service |
| Firewall | Controls inbound and outbound network traffic |

---

# 🌍 IP Address

## What is an IP Address?

An **IP (Internet Protocol) Address** uniquely identifies a device connected to a network.

No two devices on the same network can have the same IP address.

Example:

```text
192.168.1.10
```

IP addresses can be assigned in two ways:

- **Static IP**
  - Manually assigned
  - Typically used for servers and networking devices

- **Dynamic IP**
  - Automatically assigned using DHCP
  - Commonly used for client devices

---

## IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|----------|------|-------|
| Size | 32-bit | 128-bit |
| Format | 192.168.1.10 | 2001:db8::1 |
| Address Space | ~4.3 Billion | Virtually Unlimited |
| Status | Widely Used | Next Generation |

---

# 🏠 Network Types

## LAN (Local Area Network)

A LAN connects devices within a limited area such as:

- Home
- Office
- Campus

Characteristics:

- High speed
- Low latency
- Private network

---

## WAN (Wide Area Network)

A WAN connects multiple LANs across different geographic locations.

Examples:

- Internet
- Corporate WAN
- Branch office connectivity

---

# 🔀 Router

## What is a Router?

A Router connects two or more different networks.

It forwards network traffic between:

- Private Network
- Public Network (Internet)

A router typically contains:

- Internal Interface
- External Interface

The external interface usually receives a Public IP Address from an Internet Service Provider (ISP).

---

# 📍 Public vs Private IP Addresses

## Private IP Address

Private IP addresses are used within internal networks.

They cannot be accessed directly from the Internet.

Reserved Private IPv4 ranges:

| Class | IP Range |
|--------|----------|
| A | 10.0.0.0 – 10.255.255.255 |
| B | 172.16.0.0 – 172.31.255.255 |
| C | 192.168.0.0 – 192.168.255.255 |

---

## Public IP Address

Public IP addresses:

- Are globally unique
- Can communicate over the Internet
- Are assigned by an ISP

---

# 🔄 NAT (Network Address Translation)

## What is NAT?

NAT translates Private IP addresses into Public IP addresses.

It allows multiple devices inside a private network to access the Internet using a single Public IP address.

Benefits:

- Conserves IPv4 addresses
- Hides internal IP addresses
- Improves security

---

# 🚪 Port Numbers

## What is a Port Number?

A Port Number identifies the application or service running on a device.

Think of it like:

- IP Address → House Address
- Port Number → Room Number

Examples:

| Service | Port |
|----------|-----:|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |

---

## Port Ranges

| Range | Description |
|--------|-------------|
| 0–1023 | Well-Known Ports |
| 1024–49151 | Registered Ports |
| 49152–65535 | Dynamic (Ephemeral) Ports |

---

# 🎯 Why Are These Concepts Important?

Networking fundamentals help you understand:

- How devices communicate
- How applications communicate
- How cloud networking works
- How AWS VPC is designed
- Why Public and Private Subnets exist
- How Internet Gateways and NAT Gateways work

Without these fundamentals, understanding AWS networking becomes much more difficult.

---

# ✅ Best Practices

- Use DHCP to automatically assign IP addresses to client devices.
- Reserve Static IP addresses for servers and networking devices.
- Use Private IP addresses for internal resources whenever possible.
- Assign Public IP addresses only to Internet-facing resources.
- Use NAT instead of assigning Public IPs to every device.
- Allow only the required ports through firewalls or Security Groups.
- Close unused ports to reduce security risks.
- Plan IP address ranges carefully to avoid conflicts.
- Prefer IPv6 support for new environments while maintaining IPv4 compatibility.

---

# 📊 Networking Summary

| Concept | Purpose |
|----------|----------|
| IP Address | Identifies a device |
| DHCP | Assigns IP addresses automatically |
| LAN | Local communication |
| WAN | Connects multiple networks |
| Router | Connects different networks |
| Public IP | Internet communication |
| Private IP | Internal communication |
| NAT | Shares one Public IP across multiple devices |
| Port Number | Identifies an application |

---

# ❓ Interview Questions

### Q1. What is the difference between a Public IP and a Private IP?

**Answer**

A Private IP is used within an internal network and cannot be accessed directly from the Internet. A Public IP is globally unique and allows communication over the Internet.

---

### Q2. Why do we need NAT?

**Answer**

NAT allows multiple devices using Private IP addresses to access the Internet through a single Public IP address, conserving IPv4 addresses and improving security.

---

### Q3. What is the difference between Static IP and Dynamic IP?

**Answer**

A Static IP is manually assigned and typically used for servers, while a Dynamic IP is automatically assigned by a DHCP server and is commonly used for client devices.

---

### Q4. What is the difference between an IP Address and a Port Number?

**Answer**

An IP Address identifies a device on the network, while a Port Number identifies the application or service running on that device.

---

### Q5. What is the purpose of DHCP?

**Answer**

DHCP automatically assigns IP addresses to devices, simplifying network management and preventing IP conflicts.

---

# 💡 Key Takeaways

- Every device requires a unique IP address to communicate.
- DHCP automates IP address assignment.
- Routers connect different networks.
- Private IP addresses are used internally, while Public IP addresses enable Internet communication.
- NAT allows multiple devices to share a single Public IP.
- Port Numbers identify applications running on a device.
- Networking fundamentals form the foundation for AWS services such as Amazon VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways, Security Groups, and Network ACLs.

---

# 📚 Related Topics

- AWS Storage Options
- Amazon VPC
- Subnets
- Internet Gateway
- NAT Gateway
- Route Tables
- Security Groups
- Network ACLs

---

# 📖 References

- https://datatracker.ietf.org/
- https://www.rfc-editor.org/
- https://docs.aws.amazon.com/vpc/