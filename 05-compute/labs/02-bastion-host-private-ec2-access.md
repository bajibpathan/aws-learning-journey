# 🧪 Lab 02: Access a Private EC2 Instance Through a Bastion Host

> Deploy EC2 instances across public and private subnets and securely access the private instance through a bastion host without exposing the private EC2 instance directly to the Internet.

---

# 📖 Lab Overview

In the previous EC2 lab, we launched an EC2 instance inside a public subnet and connected to it using SSH.

In this lab, we extend that architecture by deploying:

```text
1 EC2 Instance
Public Subnet

+

1 EC2 Instance
Private Subnet
```

The objective is to access the private EC2 instance **without giving it direct inbound Internet exposure**.

Instead of:

```text
Internet
    │
    │ SSH 22
    ▼
Private EC2
```

we use:

```text
Administrator
     │
     │ SSH
     ▼
Bastion Host
Public Subnet
     │
     │ SSH
     ▼
Private EC2
Private Subnet
```

The public EC2 instance acts as a:

```text
Bastion Host
    or
Jump Host
```

This provides an administrative entry point into the VPC.

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

* Deploy EC2 instances in public and private subnets
* Explain why a private EC2 instance does not need a public IP
* Understand the bastion/jump-host pattern
* Configure different Security Groups for public and private EC2 instances
* Restrict SSH access to trusted sources
* Understand Security Group referencing
* Connect to a public EC2 instance using SSH
* Connect from the bastion host to a private EC2 instance
* Use private IP addresses for internal VPC communication
* Explain why the private EC2 instance is not directly Internet-accessible
* Compare subnet CIDR-based access with Security Group-based access
* Understand why Systems Manager Session Manager can be a better operational alternative

---

# 🏢 Business Scenario

An organization has application servers running inside private subnets.

Administrators occasionally need operating-system-level access to these servers.

The application servers should **not** have:

```text
Public IPv4 Addresses
```

and should **not** accept SSH connections directly from the Internet.

The organization therefore introduces a management server in a public subnet.

Architecture:

```text
                    Internet
                        │
                        │ SSH 22
                        ▼
                ┌─────────────────┐
                │  Bastion Host   │
                │  Public Subnet  │
                └────────┬────────┘
                         │
                         │ SSH 22
                         │ Private IP
                         ▼
                ┌─────────────────┐
                │   Private EC2   │
                │ Private Subnet  │
                └─────────────────┘
```

---

# 🏗️ Starting Architecture

This lab assumes the VPC networking from the previous labs already exists.

Example:

```text
VPC
10.0.0.0/16

├── Public Subnet A
│   10.0.1.0/24
│
└── Private Subnet A
    10.0.3.0/24
```

The public subnet has:

```text
0.0.0.0/0 → Internet Gateway
```

The private subnet does **not** have a direct route to the Internet Gateway.

If outbound Internet connectivity is required from the private subnet, it can use a NAT Gateway:

```text
0.0.0.0/0 → NAT Gateway
```

---

# 🏗️ Target Architecture

```text
                         Internet
                             │
                             ▼
                    Internet Gateway
                             │
                             ▼
                    Public Route Table
                             │
                             ▼
                ┌──────────────────────┐
                │    Public Subnet     │
                │     10.0.1.0/24      │
                │                      │
                │    Bastion EC2       │
                │    Public IP ✓       │
                │    Private IP ✓      │
                └──────────┬───────────┘
                           │
                           │ SSH 22
                           │
                           ▼
                ┌──────────────────────┐
                │    Private Subnet    │
                │     10.0.3.0/24      │
                │                      │
                │    Private EC2       │
                │    Public IP ✗       │
                │    Private IP ✓      │
                └──────────────────────┘
```

---

# 🔐 Security Design

We will use two separate Security Groups.

```text
Bastion Host
     │
     ▼
Bastion-SG

Private EC2
     │
     ▼
Private-EC2-SG
```

Each Security Group has a different responsibility.

---

# 🔒 Security Group 1: Bastion Host

Create:

```text
Name:
Bastion-SG
```

Inbound:

```text
Protocol: TCP
Port:     22
Source:   My IP
```

Example:

```text
Your Public IP
      │
      │ TCP 22
      ▼
  Bastion-SG
      │
      ▼
 Bastion EC2
```

Avoid:

```text
SSH 22
Source: 0.0.0.0/0
```

when it is not required.

The bastion should only accept SSH from trusted administrator locations.

---

# 🔒 Security Group 2: Private EC2

During the initial exercise, SSH can be permitted from the Public Subnet CIDR.

Example:

```text
Protocol: TCP
Port:     22
Source:   10.0.1.0/24
```

Architecture:

```text
Public Subnet
10.0.1.0/24
      │
      │ SSH 22
      ▼
Private-EC2-SG
      │
      ▼
Private EC2
```

This is already significantly different from:

```text
0.0.0.0/0
```

because the private instance accepts SSH only from the specified internal network range.

However, we can improve this further.

---

# ⭐ Improved Security Design

Instead of trusting every resource in:

```text
10.0.1.0/24
```

configure the private EC2 Security Group to trust:

```text
Bastion-SG
```

The inbound rule becomes conceptually:

```text
Type:     SSH
Protocol: TCP
Port:     22
Source:   Bastion-SG
```

Architecture:

```text
Administrator
     │
     │ SSH 22
     ▼
┌─────────────────┐
│   Bastion EC2   │
│                 │
│   Bastion-SG    │
└────────┬────────┘
         │
         │ SSH 22
         ▼
┌─────────────────┐
│   Private EC2   │
│                 │
│ Private-EC2-SG  │
│                 │
│ Source:         │
│ Bastion-SG      │
└─────────────────┘
```

This is an important improvement.

---

# 🧠 CIDR vs Security Group Reference

### Initial Approach

```text
Private EC2 SG

SSH 22
Source: 10.0.1.0/24
```

This means:

> Allow SSH from resources using addresses within this permitted network range.

### Improved Approach

```text
Private EC2 SG

SSH 22
Source: Bastion-SG
```

This means:

> Allow SSH traffic when the source network interface is associated with the trusted Bastion Security Group, subject to AWS Security Group referencing rules.

---

# 📊 Comparison

| Approach               | Trust Based On             | Scope                  |
| ---------------------- | -------------------------- | ---------------------- |
| `0.0.0.0/0`            | Anywhere                   | Very broad             |
| Public Subnet CIDR     | IP range                   | More restricted        |
| Bastion Security Group | Security Group association | More workload-specific |

For this architecture, Security Group referencing provides a cleaner trust relationship.

---

# 🧪 Phase 1: Deploy the Bastion Host

Launch the first EC2 instance.

Configure:

```text
Name:
ec2-bastion

AMI:
Amazon Linux

Subnet:
Public Subnet A

Public IPv4:
Enabled

Security Group:
Bastion-SG
```

The bastion needs both:

```text
Private IP
+
Public IP
```

The public address allows the administrator to reach it.

The private address allows it to communicate with resources inside the VPC.

---

# 🧪 Phase 2: Deploy the Private EC2 Instance

Launch another EC2 instance.

Configure:

```text
Name:
ec2-private

AMI:
Amazon Linux

Subnet:
Private Subnet A

Public IPv4:
Disabled

Security Group:
Private-EC2-SG
```

The important difference is:

```text
Public IPv4
Disabled
```

Record the private IP:

```text
Private EC2 IP:

________________________
```

Example:

```text
10.0.3.25
```

---

# 🔎 Phase 3: Compare the Instances

Open the EC2 console and compare both servers.

| Property                   | Bastion EC2             | Private EC2 |
| -------------------------- | ----------------------- | ----------- |
| Public Subnet              | ✅                       | ❌           |
| Private Subnet             | ❌                       | ✅           |
| Private IP                 | ✅                       | ✅           |
| Public IP                  | ✅                       | ❌           |
| Direct SSH from Internet   | Restricted but possible | ❌           |
| Internal VPC communication | ✅                       | ✅           |

This distinction is fundamental.

---

# 🧪 Phase 4: Prove Direct Access Does Not Work

From your local computer, try to determine how you would directly SSH to the private instance.

The private instance has:

```text
Private IP
```

but no:

```text
Public IP
```

Your laptop on the Internet cannot simply connect to:

```text
10.0.3.x
```

through the Internet.

Expected direct path:

```text
Laptop
   │
   ▼
Internet
   │
   ✗
   │
Private EC2
```

There is no direct Internet path to the private EC2 instance.

That is intentional.

---

# 🧪 Phase 5: Connect to the Bastion Host

From your local terminal:

```bash
ssh -i <key.pem> ec2-user@<bastion-public-ip>
```

Architecture:

```text
Your Laptop
     │
     │ SSH
     ▼
Internet
     │
     ▼
Bastion Public IP
     │
     ▼
Bastion EC2
```

Validate:

```bash
whoami
```

Expected:

```text
ec2-user
```

Check:

```bash
hostname
```

and:

```bash
ip addr
```

You are now inside the AWS VPC through the bastion host.

---

# 🧪 Phase 6: Test Private Connectivity

From the bastion host, identify the private EC2 address.

Example:

```text
10.0.3.25
```

Test basic connectivity where permitted by your configuration.

Then prepare to establish SSH:

```text
Bastion
   │
   │ TCP 22
   ▼
10.0.3.25
```

The connection now originates from inside the VPC rather than directly from the Internet.

---

# 🔑 Phase 7: SSH Authentication

The private EC2 instance still requires authentication.

Your SSH credentials must therefore be available to the SSH client performing the authentication.

Avoid casually copying private key files onto the bastion server.

A better SSH workflow can use:

```text
SSH Agent Forwarding
```

or other appropriately managed access methods.

For a simple learning environment, understand the authentication requirement separately from the networking requirement:

```text
Networking
    │
    └── Can Bastion Reach Private EC2:22?

Authentication
    │
    └── Can Administrator Authenticate?
```

Both must succeed.

---

# 🧪 Phase 8: Connect to the Private EC2

After the required SSH authentication mechanism is available, connect from the bastion path to the private EC2 instance.

Conceptually:

```bash
ssh ec2-user@<private-ip>
```

Example:

```bash
ssh ec2-user@10.0.3.25
```

Traffic path:

```text
Laptop
  │
  │ SSH
  ▼
Bastion EC2
Public Subnet
  │
  │ SSH using Private IP
  ▼
Private EC2
Private Subnet
```

You have now reached an EC2 instance that has:

```text
No Public IP
```

without exposing SSH on that instance directly to the Internet.

---

# ✅ Phase 9: Validate the Private Instance

Once connected:

```bash
whoami
```

Check:

```bash
hostname
```

and:

```bash
pwd
```

You should confirm that you are now on:

```text
ec2-private
```

rather than:

```text
ec2-bastion
```

This is an important habit when moving through multiple servers.

---

# 🧠 Understanding the Complete Connection

The full path is:

```text
Administrator Laptop
        │
        │ Internet
        ▼
Bastion Public IP
        │
        ▼
Bastion-SG
SSH from Administrator IP
        │
        ▼
Bastion EC2
Public Subnet
        │
        │ VPC Private Network
        │ TCP 22
        ▼
Private-EC2-SG
Source: Bastion-SG
        │
        ▼
Private EC2
Private Subnet
```

Notice what is **not** present:

```text
Internet
   │
   ✗
   ▼
Private EC2
```

The private EC2 server has no direct Internet-facing SSH path.

---

# 🛣️ Why Can the Two Instances Communicate?

Both subnets belong to the same VPC.

The VPC route table contains the local route.

Example:

```text
10.0.0.0/16 → local
```

Therefore:

```text
Public EC2
10.0.1.x
     │
     ▼
VPC Local Route
     │
     ▼
Private EC2
10.0.3.x
```

The Internet Gateway is **not** required for this communication.

The NAT Gateway is **not** required for this communication.

The traffic remains within the VPC networking environment.

---

# 🧠 Routing vs Security

This lab demonstrates an important networking principle:

```text
Routing
   ≠
Security
```

The VPC local route may provide a network path:

```text
10.0.0.0/16 → local
```

but the Security Group must still permit the traffic.

Therefore successful SSH requires:

```text
Valid Route
    +
Security Group Permission
    +
SSH Authentication
    +
SSH Service Running
```

---

# 💥 Phase 10: Break the Security Group

Now deliberately break the architecture.

Remove:

```text
SSH 22
Source: Bastion-SG
```

from:

```text
Private-EC2-SG
```

Try connecting again.

Expected:

```text
FAIL
```

Restore:

```text
SSH 22
Source: Bastion-SG
```

Try again.

Expected:

```text
SUCCESS
```

This proves that simply being inside the same VPC does not automatically give one server unrestricted access to another.

---

# 💥 Phase 11: Test the Trust Boundary

This is an important architecture exercise.

Imagine another EC2 instance exists in the Public Subnet:

```text
Public Subnet

├── Bastion EC2
│   └── Bastion-SG
│
└── Random EC2
    └── Different-SG
```

If the Private Security Group allows:

```text
10.0.1.0/24
```

both machines may satisfy the source network condition for SSH.

But if the rule uses:

```text
Source: Bastion-SG
```

the architecture expresses a more specific trust relationship:

```text
Bastion-SG
     │
     │ Trusted
     ▼
Private-EC2-SG
```

This is one reason Security Group references are powerful in multi-tier AWS architectures.

---

# 🔐 Least-Privilege Design

The objective should not be:

```text
Make SSH Work
```

The objective should be:

```text
Allow only the required source
            │
            ▼
Allow only the required protocol
            │
            ▼
Allow only the required port
            │
            ▼
Allow only the required destination
```

For this lab:

```text
Administrator IP
       │
       │ SSH 22
       ▼
Bastion
       │
       │ SSH 22
       ▼
Private EC2
```

---

# ⭐ Better Operational Alternative: Session Manager

The bastion pattern is useful to understand because it teaches:

* Public vs private network placement
* VPC routing
* Security Group relationships
* SSH
* Trust boundaries
* Jump-host architecture

However, AWS Systems Manager Session Manager can eliminate the need for traditional inbound SSH access in many management scenarios.

Conceptually:

```text
Administrator
      │
      ▼
AWS Systems Manager
Session Manager
      │
      ▼
Private EC2
```

This can avoid maintaining:

```text
Public Bastion Host
Public IP
Inbound SSH Rule
SSH Key Distribution
```

The exact Session Manager architecture and requirements should be covered in a separate lab rather than mixed into this exercise.

---

# 📊 Bastion Host vs Session Manager

| Characteristic                 | Bastion Host                           | Session Manager                                   |
| ------------------------------ | -------------------------------------- | ------------------------------------------------- |
| Jump server required           | Yes                                    | No traditional bastion required                   |
| Public IP potentially required | For this public bastion design         | Not on managed private instance                   |
| Inbound SSH required           | Yes for this design                    | No                                                |
| SSH key management             | Usually required                       | Not for Session Manager sessions                  |
| IAM integration                | Indirect/optional depending on design  | Core access-control mechanism                     |
| Infrastructure management      | Bastion must be managed                | AWS-managed service handles session control plane |
| Learning value                 | Excellent for understanding networking | Excellent modern management pattern               |

Understanding both approaches is valuable.

---

# 🔍 Troubleshooting Method

If the bastion can be reached but the private server cannot, troubleshoot systematically.

```text
Can Laptop Reach Bastion?
          │
          ▼
Is Bastion Running?
          │
          ▼
Can Bastion Resolve/Reach Private IP?
          │
          ▼
Check VPC Route
          │
          ▼
Check Private EC2 Security Group
          │
          ▼
Check NACL
          │
          ▼
Check SSH Authentication
          │
          ▼
Check SSH Service
```

Do not randomly change Security Group rules.

---

# 🔍 Troubleshooting Scenario

Suppose:

```text
Laptop → Bastion
SUCCESS

Bastion → Private EC2
FAIL
```

What should you investigate?

### 1. Destination

Are you using the correct:

```text
Private IP
```

of the private EC2 instance?

### 2. Routing

Does the VPC have:

```text
10.0.0.0/16 → local
```

or the appropriate local VPC route?

### 3. Security Group

Does:

```text
Private-EC2-SG
```

permit:

```text
TCP 22
Source: Bastion-SG
```

### 4. NACL

Does the subnet NACL permit the required traffic and return traffic?

### 5. Authentication

Are you using the correct:

```text
Username
Key / Authentication Method
```

### 6. Service

Is SSH actually running on the destination?

---

# ❓ Interview Questions

### Q1. What is a bastion host?

**Answer**

A bastion host is a server used as a controlled administrative entry point for accessing resources that are not directly accessible from an external network.

---

### Q2. Why is the bastion host in a public subnet?

**Answer**

In this architecture, administrators need to reach the bastion from the Internet, so it is placed in a subnet with the required Internet Gateway route and given a public address.

---

### Q3. Why is the application server in a private subnet?

**Answer**

The server does not require direct inbound Internet connectivity and can therefore remain privately addressed.

---

### Q4. Does the private EC2 instance require a Public IP for the bastion to reach it?

**Answer**

No.

The bastion communicates with the private EC2 instance using its private IP through VPC routing.

---

### Q5. Does traffic between the bastion and private EC2 go through the Internet Gateway?

**Answer**

No.

When communicating using their private addresses within the same VPC, the traffic uses VPC-local routing.

---

### Q6. Does the NAT Gateway enable the bastion-to-private SSH connection?

**Answer**

No.

A NAT Gateway is not required for communication between these instances using their private IP addresses within the VPC.

---

### Q7. Why not allow SSH to the private instance from `0.0.0.0/0`?

**Answer**

The private instance does not need to accept SSH from arbitrary Internet sources.

Access should be restricted to the required trusted administrative path.

---

### Q8. Why is a Security Group reference better than allowing the entire public subnet CIDR?

**Answer**

A subnet CIDR rule trusts addresses within that network range.

A Security Group reference allows the architecture to express that traffic should come from resources associated with the specified trusted Security Group, making the rule more closely aligned with workload identity.

---

### Q9. Can two instances in the same VPC automatically communicate on every port?

**Answer**

No.

VPC routing may provide the network path, but Security Groups and NACLs must still permit the required traffic.

---

### Q10. What is the difference between routing and security?

**Answer**

Routing determines where traffic should go.

Security controls determine whether that traffic is permitted.

Both must be correctly configured.

---

### Q11. What alternatives exist to using a bastion host?

**Answer**

AWS Systems Manager Session Manager can provide administrative access to appropriately configured managed instances without requiring traditional inbound SSH access.

---

### Q12. Should private SSH keys normally be copied onto a bastion host?

**Answer**

Avoid storing private SSH keys on intermediate servers when possible.

Use an appropriate secure authentication mechanism, such as carefully configured SSH agent forwarding, certificate-based approaches, or AWS management services such as Session Manager depending on the architecture.

---

# 🏁 Completion Criteria

Do not mark this lab complete until you can:

* [ ] Deploy an EC2 instance in a Public Subnet
* [ ] Deploy an EC2 instance in a Private Subnet
* [ ] Explain why the bastion has a Public IP
* [ ] Explain why the private instance does not
* [ ] Create separate Security Groups
* [ ] Restrict bastion SSH to your trusted IP
* [ ] Initially understand subnet CIDR-based SSH access
* [ ] Replace the subnet CIDR rule with a Bastion Security Group reference
* [ ] Connect from your laptop to the bastion
* [ ] Connect from the bastion path to the private EC2 instance
* [ ] Verify the private connection uses the Private IP
* [ ] Explain the VPC local route
* [ ] Explain why IGW is not involved in bastion-to-private communication
* [ ] Explain why NAT Gateway is not involved
* [ ] Break the private Security Group rule
* [ ] Diagnose the failed SSH connection
* [ ] Restore connectivity
* [ ] Explain routing vs security
* [ ] Explain least privilege
* [ ] Explain the bastion-host pattern
* [ ] Explain why Session Manager can be a better operational alternative

---

# 💰 Cost Awareness

This lab may create costs for:

* EC2 instances
* EBS volumes
* Public IPv4 address usage
* NAT Gateway if retained from previous networking labs
* Data transfer where applicable

Because this lab uses two EC2 instances, terminate resources when the exercise is complete unless they are required for the next lab.

---

# 🧹 Cleanup Checklist

After completing the lab:

* [ ] Terminate Bastion EC2
* [ ] Terminate Private EC2
* [ ] Verify unnecessary EBS volumes are removed
* [ ] Delete Bastion-SG if no longer required
* [ ] Delete Private-EC2-SG if no longer required
* [ ] Remove unused EC2 Key Pairs
* [ ] Securely remove unused local private-key files
* [ ] Release unused Elastic IPs if any were created
* [ ] Verify no unnecessary EC2 resources remain

If the underlying VPC is being reused for future labs, keep the VPC networking resources.

---

# 💡 Key Takeaways

* Private EC2 instances do not need to be directly exposed to the Internet for administrative access.
* A bastion host can act as a controlled jump point into a private network.
* The bastion host can have both a Public IP and a Private IP.
* The private EC2 instance needs only a Private IP for communication with the bastion.
* Communication between the bastion and private EC2 uses VPC-local networking.
* An Internet Gateway is not used for private-IP communication between these instances.
* A NAT Gateway is not required for bastion-to-private SSH communication.
* Routing determines whether a network path exists.
* Security Groups determine whether the required traffic is permitted.
* Allowing the Public Subnet CIDR is more restrictive than allowing the entire Internet.
* Referencing the Bastion Security Group provides a more precise trust relationship than trusting every resource in the public subnet.
* Least privilege means allowing only the source, destination, protocol, and port actually required.
* Bastion hosts are important to understand even when newer operational approaches are available.
* Systems Manager Session Manager can remove the need for traditional inbound SSH and public bastion infrastructure in many management scenarios.
* Understanding both patterns helps explain how AWS networking and secure administrative access actually work.

> The important lesson is not simply how to SSH from one server to another. It is how to design a controlled trust path from a public entry point to a private workload without exposing the private server directly to the Internet.
