# 📊 Amazon VPC Flow Logs

> Learn how Amazon VPC Flow Logs capture information about IP traffic flowing to and from network interfaces in your VPC for monitoring, troubleshooting, security analysis, and auditing.

---

# 📖 Overview

**Amazon VPC Flow Logs** is a VPC feature that captures information about **IP traffic flowing to and from network interfaces** in your AWS environment.

Flow Logs help answer questions such as:

* Which source IP connected to my resource?
* Which destination IP was contacted?
* Which port was used?
* Was the traffic accepted or rejected?
* Which network interface handled the traffic?
* Why might communication between two resources be failing?

Flow Logs can be created at three main levels:

```text
VPC
 │
 ├── Subnet
 │
 └── Network Interface (ENI)
```

Flow log data can be published to destinations including:

* Amazon CloudWatch Logs
* Amazon S3
* Amazon Data Firehose

---

# 📊 Key Components

| Component            | Purpose                                                    |
| -------------------- | ---------------------------------------------------------- |
| VPC Flow Log         | Captures metadata about IP traffic                         |
| VPC                  | Can capture traffic across network interfaces in the VPC   |
| Subnet               | Can capture traffic for network interfaces within a subnet |
| ENI                  | Can capture traffic for a specific network interface       |
| Flow Log Record      | Contains information about observed network traffic        |
| CloudWatch Logs      | Stores logs for monitoring and analysis                    |
| Amazon S3            | Stores logs for longer-term analysis and retention         |
| Amazon Data Firehose | Delivers flow log records to supported destinations        |

---

# 🔍 What are VPC Flow Logs?

VPC Flow Logs capture information about network traffic associated with Elastic Network Interfaces.

Consider:

```text
EC2-A
10.0.10.10
   │
   │ TCP 443
   ▼
EC2-B
10.0.20.20
```

A Flow Log record can contain information such as:

```text
Source IP       10.0.10.10
Destination IP  10.0.20.20
Source Port     49152
Destination     443
Protocol        TCP
Action          ACCEPT
```

This provides visibility into the communication without capturing the actual contents of the packets.

---

# 🚫 Flow Logs are Not Packet Capture

An important distinction is:

**VPC Flow Logs do not capture the complete contents of network packets.**

They capture **metadata about network flows**.

For example:

```text
VPC Flow Logs

Source IP
Destination IP
Source Port
Destination Port
Protocol
Bytes
Packets
Action
Timestamps
```

They do not provide:

```text
HTTP Request Body
Passwords
Application Payload
Complete Packet Contents
```

Think of Flow Logs as:

```text
Who communicated with whom?
        +
Which port/protocol?
        +
How much traffic?
        +
Was it accepted or rejected?
```

rather than:

```text
What exactly was inside every packet?
```

---

# 🏗️ Flow Log Levels

VPC Flow Logs can be created for:

1. VPC
2. Subnet
3. Network Interface

---

# 1️⃣ VPC-Level Flow Logs

A Flow Log can be created for an entire VPC.

```text
             VPC
              │
      ┌───────┴───────┐
      │               │
 Public Subnet    Private Subnet
      │               │
     ENI             ENI
      │               │
     EC2             EC2

      Flow Log
         │
         ▼
   Entire VPC Scope
```

This provides broad network visibility across network interfaces in the VPC.

It can be useful for:

* Central network monitoring
* Security analysis
* Troubleshooting
* Auditing

---

# 2️⃣ Subnet-Level Flow Logs

A Flow Log can also be created for a specific subnet.

```text
VPC
 │
 ├── Public Subnet
 │
 └── Private Subnet
          │
          ▼
       Flow Log
```

The Flow Log captures traffic associated with network interfaces within that subnet.

This can be useful when monitoring a specific application tier.

For example:

```text
VPC

Web Subnet
   │
   ▼
Flow Logs

App Subnet

Database Subnet
```

You could monitor only the Web Subnet if that is the area being investigated.

---

# 3️⃣ ENI-Level Flow Logs

Flow Logs can also be created for a specific **Elastic Network Interface (ENI)**.

```text
EC2 Instance
     │
     ▼
    ENI
     │
     ▼
 VPC Flow Log
```

This provides more focused visibility for a particular network interface.

For example, if one EC2 instance is experiencing connectivity problems, you could analyze traffic associated with its ENI.

---

# 🚦 Accepted vs Rejected Traffic

One of the most useful Flow Log fields is the **action**.

Traffic can be recorded as:

```text
ACCEPT
```

or:

```text
REJECT
```

---

## ACCEPT

`ACCEPT` means the traffic was permitted by applicable VPC network controls such as Security Groups and Network ACLs.

Example:

```text
Source
10.0.10.10
    │
    │ TCP 443
    ▼
Security Controls
    │
    │ ACCEPT
    ▼
Destination
10.0.20.20
```

Flow Log:

```text
10.0.10.10 → 10.0.20.20 → TCP 443 → ACCEPT
```

---

## REJECT

`REJECT` means the traffic was rejected.

Example:

```text
Source
10.0.10.10
    │
    │ TCP 22
    ▼
Security Controls
    │
    ✖ REJECT
    │
Destination
10.0.20.20
```

Flow Log:

```text
10.0.10.10 → 10.0.20.20 → TCP 22 → REJECT
```

Rejected traffic is particularly useful when troubleshooting Security Group or Network ACL configurations.

---

# 🎯 Traffic Types

When creating a Flow Log, you can choose which type of traffic to capture.

The options are:

```text
ACCEPT
REJECT
ALL
```

### ACCEPT

Records accepted traffic.

### REJECT

Records rejected traffic.

### ALL

Records both accepted and rejected traffic.

Example:

```text
Flow Log
   │
   ├── ACCEPT
   │
   └── REJECT
```

The correct choice depends on monitoring and troubleshooting requirements.

---

# 📝 Flow Log Records

A Flow Log record can contain fields such as:

```text
version
account-id
interface-id
srcaddr
dstaddr
srcport
dstport
protocol
packets
bytes
start
end
action
log-status
```

For example:

```text
2 123456789012 eni-abc123
10.0.10.10 10.0.20.20
49152 443 6
10 5000
ACCEPT OK
```

This could indicate communication from:

```text
10.0.10.10
     │
     │ TCP 443
     ▼
10.0.20.20

Action: ACCEPT
```

AWS also supports additional Flow Log fields beyond the default format.

---

# 🔢 Protocol Numbers

Flow Logs may represent protocols using their protocol numbers.

Common examples include:

| Protocol | Number |
| -------- | -----: |
| ICMP     |      1 |
| TCP      |      6 |
| UDP      |     17 |
| ICMPv6   |     58 |

For example:

```text
protocol = 6
```

means:

```text
TCP
```

This is useful when interpreting raw Flow Log records.

---

# ☁️ CloudWatch Logs Destination

Flow Logs can be published to **Amazon CloudWatch Logs**.

Architecture:

```text
VPC / Subnet / ENI
        │
        ▼
    Flow Logs
        │
        ▼
CloudWatch Logs
```

CloudWatch Logs can be useful for:

* Searching network events
* Troubleshooting
* Monitoring rejected traffic
* Creating metrics and alarms
* Analyzing recent network behavior

For example, you might investigate repeated rejected SSH connections:

```text
Destination Port = 22
Action = REJECT
```

---

# 🪣 Amazon S3 Destination

Flow Logs can also be delivered to **Amazon S3**.

```text
VPC
 │
 ▼
Flow Logs
 │
 ▼
Amazon S3
```

S3 can be useful for:

* Long-term retention
* Historical analysis
* Security investigations
* Compliance requirements
* Large-scale analytics

Flow Logs stored in S3 can also be analyzed using services such as **Amazon Athena**.

Example:

```text
VPC Flow Logs
      │
      ▼
Amazon S3
      │
      ▼
Amazon Athena
      │
      ▼
SQL Analysis
```

---

# 🔥 Amazon Data Firehose Destination

VPC Flow Logs can also publish records to **Amazon Data Firehose**.

Conceptually:

```text
VPC Flow Logs
      │
      ▼
Amazon Data Firehose
      │
      ▼
Destination
```

This can be useful when building centralized log-processing and analytics architectures.

---

# 🕵️ Troubleshooting with VPC Flow Logs

Suppose:

```text
Web Server
10.0.10.10
     │
     │ TCP 3306
     ▼
Database
10.0.30.10
```

The application cannot connect to the database.

Without Flow Logs, you might start checking multiple components manually.

With Flow Logs, you could search for:

```text
Source IP:
10.0.10.10

Destination IP:
10.0.30.10

Destination Port:
3306
```

If you see:

```text
REJECT
```

you know the traffic is being rejected by applicable network controls and can investigate Security Groups or Network ACLs.

If you see:

```text
ACCEPT
```

the network controls permitted the traffic, so you may need to investigate other causes such as:

* Application configuration
* Service availability
* Operating system firewall
* Incorrect listening port
* Application-level failure

Flow Logs therefore help narrow down the troubleshooting scope.

---

# 🔒 Security Monitoring

VPC Flow Logs can also help identify suspicious network activity.

For example:

```text
Unknown Internet IP
       │
       ├── Port 22
       ├── Port 3389
       ├── Port 3306
       └── Port 5432
```

Repeated connection attempts across multiple ports may indicate scanning or unwanted activity.

Flow Logs can provide information such as:

```text
Source IP
Destination IP
Port
Protocol
Action
Packets
Bytes
```

Security teams can use this information as part of network investigations.

---

# 🏢 Real-World Example

Consider a three-tier application:

```text
Internet
   │
   ▼
Application Load Balancer
   │
   ▼
Web Tier
   │
   ▼
Application Tier
   │
   ▼
Database Tier
```

An application suddenly reports database connectivity problems.

Flow Logs might show:

```text
App Server
10.0.20.15

      │
      │ TCP 5432
      ▼

Database
10.0.30.25

Action: REJECT
```

This immediately gives the engineer useful information:

```text
Source       = 10.0.20.15
Destination  = 10.0.30.25
Port         = 5432
Action       = REJECT
```

The engineer can then investigate the relevant Security Group or Network ACL instead of guessing where the problem exists.

---

# 🔍 Flow Logs and Security Groups

VPC Flow Logs are particularly useful when troubleshooting Security Groups.

Consider:

```text
Application Server
      │
      │ TCP 3306
      ▼
Database Security Group
      │
      ✖
      ▼
Database
```

The Flow Log might show:

```text
srcaddr   = 10.0.20.10
dstaddr   = 10.0.30.10
dstport   = 3306
protocol  = 6
action    = REJECT
```

This gives you evidence that the network traffic was rejected.

You can then verify whether the database Security Group permits the expected source and port.

---

# 🔍 Flow Logs and Network ACLs

Flow Logs can also help investigate Network ACL issues.

Remember:

```text
Security Group
Stateful
Allow Rules Only

Network ACL
Stateless
Allow + Deny Rules
```

A poorly configured Network ACL may reject traffic even when Security Group rules appear correct.

Flow Logs can help identify these rejected network flows.

---

# ⚠️ Flow Log Limitations

VPC Flow Logs are extremely useful, but they are not a replacement for full packet capture or application logging.

Flow Logs do not capture every type of traffic in every situation.

Some traffic is not logged, and Flow Logs generally provide network metadata rather than application payloads.

Therefore:

```text
VPC Flow Logs
      │
      ▼
Network Visibility

Application Logs
      │
      ▼
Application Visibility

Packet Capture
      │
      ▼
Detailed Packet Analysis
```

Each serves a different troubleshooting purpose.

---

# 🎯 Why Use VPC Flow Logs?

VPC Flow Logs help organizations:

* Monitor network traffic.
* Troubleshoot connectivity problems.
* Identify accepted and rejected traffic.
* Investigate Security Group issues.
* Investigate Network ACL issues.
* Analyze communication between resources.
* Support security investigations.
* Detect suspicious network behavior.
* Maintain historical network records.
* Support compliance and auditing requirements.

---

# 💰 Cost Considerations

Creating VPC Flow Logs can generate additional costs depending on the selected destination and volume of log data.

Potential costs can include:

* CloudWatch Logs ingestion
* CloudWatch Logs storage
* Amazon S3 storage
* Data processing
* Amazon Data Firehose
* Analytics services such as Amazon Athena

High-traffic VPCs can generate significant amounts of Flow Log data.

Therefore, logging strategy and retention should be planned carefully.

---

# ✅ Best Practices

* Enable VPC Flow Logs for important production networks.
* Choose the appropriate scope: VPC, Subnet, or ENI.
* Capture `REJECT` traffic when troubleshooting connectivity and security issues.
* Use `ALL` when broader network visibility is required.
* Use CloudWatch Logs when operational searching and monitoring are important.
* Consider Amazon S3 for long-term retention and large-scale analysis.
* Use Amazon Athena to analyze large volumes of Flow Logs stored in S3.
* Define appropriate log retention policies.
* Monitor logging costs in high-traffic environments.
* Use Flow Logs together with Security Groups, Network ACLs, application logs, and other monitoring tools.
* Remember that Flow Logs provide traffic metadata, not complete packet contents.

---

# 📊 VPC vs Subnet vs ENI Flow Logs

| Level  | Scope                          | Typical Use                         |
| ------ | ------------------------------ | ----------------------------------- |
| VPC    | Broad VPC visibility           | Central network monitoring          |
| Subnet | Network interfaces in a subnet | Monitor a specific application tier |
| ENI    | Individual network interface   | Troubleshoot a specific resource    |

---

# 📊 CloudWatch Logs vs S3

| Feature                         | CloudWatch Logs                | Amazon S3                    |
| ------------------------------- | ------------------------------ | ---------------------------- |
| Operational Troubleshooting     | ✅ Excellent                    | Possible                     |
| Search Recent Logs              | ✅                              | Requires analytics tools     |
| Long-Term Storage               | Possible                       | ✅ Excellent                  |
| Large-Scale Historical Analysis | Possible                       | ✅                            |
| Athena Integration              | Not direct S3-style analysis   | ✅                            |
| Monitoring / Alarms             | ✅                              | Requires additional services |
| Typical Use                     | Operations and troubleshooting | Archive and analytics        |

---

# ❓ Interview Questions

### Q1. What are Amazon VPC Flow Logs?

**Answer**

VPC Flow Logs capture information about IP traffic flowing to and from network interfaces in an Amazon VPC.

They can help with network monitoring, troubleshooting, security investigations, and auditing.

---

### Q2. At what levels can VPC Flow Logs be created?

**Answer**

Flow Logs can be created at:

* VPC level
* Subnet level
* Network Interface level

---

### Q3. Where can VPC Flow Logs be stored?

**Answer**

VPC Flow Logs can publish records to:

* Amazon CloudWatch Logs
* Amazon S3
* Amazon Data Firehose

---

### Q4. Do VPC Flow Logs capture actual network packets?

**Answer**

No.

VPC Flow Logs capture metadata about network traffic, such as source and destination addresses, ports, protocol, bytes, packets, and whether the traffic was accepted or rejected.

They do not provide the complete packet payload.

---

### Q5. What does ACCEPT mean in a VPC Flow Log?

**Answer**

`ACCEPT` indicates that the recorded traffic was permitted by applicable VPC network controls such as Security Groups and Network ACLs.

---

### Q6. What does REJECT mean in a VPC Flow Log?

**Answer**

`REJECT` indicates that the recorded traffic was rejected.

This can be useful when investigating Security Group or Network ACL configuration problems.

---

### Q7. Can VPC Flow Logs help troubleshoot Security Groups?

**Answer**

Yes.

Flow Logs can show whether traffic was accepted or rejected, helping engineers investigate whether Security Group or Network ACL rules may be responsible for a connectivity problem.

---

### Q8. Can VPC Flow Logs be used for security monitoring?

**Answer**

Yes.

Flow Logs can help identify suspicious source addresses, unexpected ports, rejected connections, unusual communication patterns, and other network activity that may require investigation.

---

### Q9. When would you store Flow Logs in S3 instead of CloudWatch Logs?

**Answer**

Amazon S3 is often appropriate when long-term retention, cost-efficient storage, or large-scale historical analysis is required.

Services such as Amazon Athena can then query Flow Logs stored in S3.

CloudWatch Logs is often more convenient for operational troubleshooting, searching, monitoring, and alerting.

---

### Q10. Are VPC Flow Logs the same as packet capture?

**Answer**

No.

Flow Logs provide metadata about network flows.

Packet capture provides much deeper visibility into individual packets and potentially their contents.

They solve different troubleshooting and monitoring requirements.

---

# 💡 Key Takeaways

* VPC Flow Logs capture **metadata about IP traffic**, not complete packets.
* Flow Logs can be enabled at the **VPC, Subnet, or ENI level**.
* Traffic can be recorded as **ACCEPT, REJECT, or ALL**.
* Flow Logs are extremely useful for network troubleshooting.
* They can help investigate Security Group and Network ACL issues.
* Logs can be published to **CloudWatch Logs, Amazon S3, or Amazon Data Firehose**.
* CloudWatch Logs is useful for operational troubleshooting and monitoring.
* S3 is useful for long-term retention and large-scale analysis.
* Amazon Athena can analyze Flow Logs stored in S3.
* Flow Logs can support network security investigations and auditing.

---

# 📚 Related Topics

* Amazon VPC
* Amazon VPC Subnets
* Elastic Network Interfaces
* Route Tables
* Security Groups
* Network ACLs
* Internet Gateway
* NAT Gateway
* VPC Endpoints
* Amazon CloudWatch Logs
* Amazon S3
* Amazon Athena
* Network Troubleshooting

---

# 📖 References

* AWS VPC Flow Logs Documentation
* AWS VPC Flow Log Records
* AWS VPC Flow Log Limitations
* AWS VPC Flow Logs to CloudWatch Logs
* AWS VPC Flow Logs to Amazon S3
* AWS VPC Flow Logs to Amazon Data Firehose
