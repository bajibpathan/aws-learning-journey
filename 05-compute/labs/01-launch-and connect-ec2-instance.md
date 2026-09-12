# 🧪 Lab 01: Launch and Connect to an Amazon EC2 Instance

> Build your first Amazon EC2 virtual server, understand the components involved in an EC2 deployment, connect to the instance securely, and validate its networking, storage, and health configuration.

---

# 📖 Lab Overview

In the previous EC2 lessons, we learned that Amazon EC2 provides virtual compute capacity inside AWS.

This lab moves from theory into practice.

We will deploy a Linux EC2 instance inside a **Public Subnet** and use it as a simple:

```text
Bastion / Management Server
```

The main objective is not just to click **Launch Instance**.

The objective is to understand the components involved in deploying an EC2 instance:

* Amazon Machine Image (AMI)
* Instance Type
* Key Pair
* VPC
* Subnet
* Private IP Address
* Public IP Address
* Elastic Network Interface (ENI)
* Security Group
* EBS Storage
* IAM Role location
* User Data location
* EC2 Status Checks
* SSH Connectivity
* EC2 Instance Connect

The course explains that an EC2 deployment requires a VPC and subnet first, and that the lesson uses a public-subnet EC2 instance as a bastion/management host.

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

* Launch an Amazon EC2 instance
* Select an Amazon Machine Image
* Choose an EC2 Instance Type
* Create and use an EC2 Key Pair
* Deploy EC2 into the correct VPC and subnet
* Explain Private IP vs Public IP
* Understand the purpose of an Elastic IP
* Attach a Security Group
* Understand the EC2 ENI relationship
* Review the root EBS volume
* Identify where IAM Roles are configured
* Identify where User Data scripts are configured
* Understand EC2 System Status Checks
* Understand EC2 Instance Status Checks
* Connect to EC2 using SSH
* Connect using EC2 Instance Connect
* Explain how a bastion host can provide access toward private resources

---

# 🏢 Business Scenario

Your organization has already created a custom VPC containing:

```text
Public Subnets

and

Private Subnets
```

You need a management server that administrators can connect to remotely.

This server will be deployed inside a Public Subnet.

Architecture:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Subnet
    │
    ▼
Bastion / Management EC2
```

Later, this type of server could provide a path toward servers located in Private Subnets.

Conceptually:

```text
Administrator
      │
      ▼
Internet
      │
      ▼
Bastion Host
Public Subnet
      │
      ▼
Private EC2 Servers
```

The course uses a bastion host to demonstrate this pattern, while also noting that AWS Systems Manager Session Manager is another way to access private instances and is increasingly preferred for many real-world remote-management scenarios.

---

# 🏗️ Prerequisites

Before starting, ensure you already have:

* An AWS account
* A custom VPC
* At least one Public Subnet
* An Internet Gateway attached to the VPC
* A Route Table that makes the subnet public
* Public IP auto-assignment enabled on the selected Public Subnet

Conceptually:

```text
VPC
 │
 └── Public Subnet
       │
       ├── Route to Internet Gateway
       │
       └── Auto-Assign Public IPv4
```

The course lesson explicitly selects a pre-created VPC and public subnet and relies on public IP auto-assignment for the bastion host.

---

# 📊 EC2 Components Used in This Lab

| Component      | Purpose                                                 |
| -------------- | ------------------------------------------------------- |
| AMI            | Defines the operating system and pre-installed software |
| Instance Type  | Defines compute capacity                                |
| Key Pair       | Provides authentication for SSH access                  |
| VPC            | Provides network boundary                               |
| Subnet         | Determines network placement and Availability Zone      |
| ENI            | Provides network connectivity                           |
| Private IP     | Enables VPC communication                               |
| Public IP      | Enables Internet communication                          |
| Security Group | Controls allowed network traffic                        |
| EBS Volume     | Provides persistent block storage                       |
| IAM Role       | Gives EC2 permissions to access AWS services            |
| User Data      | Runs scripts during instance initialization             |

---

# 🖼️ Amazon Machine Image (AMI)

An EC2 instance requires an **Amazon Machine Image**.

The AMI provides the software starting point for the instance.

It can contain:

* Operating System
* Pre-installed applications
* Other software configuration

Conceptually:

```text
AMI
 │
 ├── Operating System
 │
 └── Optional Applications
       │
       ▼
   EC2 Instance
```

For this lab, use:

```text
Amazon Linux 2023
```

The lesson explains that the AMI commonly becomes the basis for the instance's root volume and that AMIs are regional resources.

---

# 🌐 EC2 Network Interface

Every EC2 instance has a primary:

```text
Elastic Network Interface
ENI
```

Conceptually:

```text
EC2 Instance
     │
     ▼
Primary ENI
     │
     ▼
Subnet
     │
     ▼
VPC
```

The instance receives at least one **Private IP Address** from the subnet CIDR.

Example:

```text
Subnet
10.0.1.0/24

       │
       ▼

EC2 Private IP
10.0.1.x
```

The course explains that EC2 receives a private IP from the subnet CIDR and can additionally receive a public IP when Internet connectivity is required.

---

# 🌍 Public IP vs Elastic IP

For this Public Subnet lab, the EC2 instance receives a Public IPv4 address.

A normal automatically assigned Public IP may change after a stop/start cycle.

An:

```text
Elastic IP
```

provides a static public IPv4 address that can remain associated with your AWS account.

For this lab, an Elastic IP is **not required**.

The lesson introduces Elastic IP as a static public address in contrast with the normal automatically assigned public address.

---

# 🔒 Phase 1: Create a Security Group

Before launching the EC2 instance, create a Security Group.

Example name:

```text
bastion-host-sg
```

Description:

```text
Security Group for EC2 Bastion Host Lab
```

Select the correct VPC.

---

## Inbound Rule

Add:

```text
Type: SSH
Protocol: TCP
Port: 22
Source: My IP
```

For learning purposes, the course demonstrates allowing SSH from the Internet but explicitly states that this is not considered best practice and suggests restricting access to the user's current IP or corporate CIDR instead.

Therefore for this lab, prefer:

```text
My IP
```

rather than:

```text
0.0.0.0/0
```

---

# 🔐 Security Group Traffic Flow

```text
Your Laptop
Public IP
    │
    │ SSH 22
    ▼
Security Group
    │
    ▼
ENI
    │
    ▼
EC2 Instance
```

Only explicitly allowed inbound traffic should reach the EC2 instance.

---

# 🚀 Phase 2: Launch the EC2 Instance

Navigate to:

```text
AWS Console
   │
   ▼
EC2
   │
   ▼
Instances
   │
   ▼
Launch Instance
```

---

# 🏷️ Instance Name

Use a descriptive name.

Example:

```text
ec2-bastion-lab
```

---

# 🖼️ Select the AMI

Choose:

```text
Amazon Linux 2023
```

The course selects Amazon Linux 2023 for this exercise.

---

# ⚙️ Select the Instance Type

Select a small instance type suitable for the lab and eligible for your account's current free-tier or low-cost options.

The course demonstrates:

```text
t2.micro
```

but always verify current eligibility and pricing in your own AWS account before launching resources. The lesson itself uses `t2.micro` as the example instance type.

---

# 🔑 Phase 3: Create a Key Pair

Create a new EC2 Key Pair.

Example name:

```text
ec2-bastion-lab-key
```

Choose:

```text
PEM
```

Download the private key.

Example:

```text
ec2-bastion-lab-key.pem
```

Do not lose the private key.

---

# 🔐 Understanding the Key Pair

The key pair consists conceptually of:

```text
Public Key
     +
Private Key
```

For Linux EC2 instances:

```text
Private Key
     │
     ▼
SSH Authentication
     │
     ▼
EC2 Instance
```

For Windows instances, the private key can be used as part of decrypting the Administrator password.

The course explains this distinction when creating the key pair.

---

# 🌐 Phase 4: Configure Networking

Select the correct:

```text
VPC
```

Then choose your:

```text
Public Subnet
```

Example:

```text
VPC
10.0.0.0/16

Public Subnet
10.0.1.0/24
```

Ensure:

```text
Auto-Assign Public IPv4
Enabled
```

---

# 🔒 Attach the Security Group

Select the existing:

```text
bastion-host-sg
```

Do not create an unnecessary duplicate Security Group.

Architecture:

```text
Internet
   │
   │ SSH 22
   ▼
Security Group
   │
   ▼
ENI
   │
   ▼
EC2
```

---

# 💾 Phase 5: Review Storage

The selected AMI includes a root EBS volume.

The course example uses:

```text
EBS Type: gp3
Capacity: 8 GiB
```

The launch wizard also allows additional volumes to be attached if needed.

For this lab:

```text
Use the default root volume
```

No additional storage is required.

---

# 🧠 Root Volume Relationship

```text
EC2 Instance
     │
     ▼
Root EBS Volume
     │
     ├── Operating System
     └── System Files
```

A later EC2 storage lab should explore EBS in more detail.

---

# 🔑 Phase 6: Review IAM Role

Expand:

```text
Advanced Details
```

Locate:

```text
IAM Instance Profile
```

For this first launch lab:

```text
Leave it unconfigured
```

unless you have already created a specific lab IAM role.

The course explains that the EC2 instance profile acts as a container for the IAM role, which can grant the instance access to other AWS services.

---

# 🧾 Phase 7: Locate User Data

Continue within:

```text
Advanced Details
```

Locate:

```text
User Data
```

Do not add a script yet.

The purpose of this step is to understand where EC2 bootstrap scripts are configured.

User Data can run scripts during instance startup.

Examples mentioned in the lesson include installing:

* Web server software
* Python
* Docker
* Other software packages

Linux instances commonly use:

```text
Bash
```

Windows instances can use:

```text
PowerShell
```

The course identifies this location but leaves User Data empty in the demonstrated launch.

---

# 🚀 Phase 8: Launch the Instance

Review:

```text
AMI
Instance Type
Key Pair
VPC
Subnet
Public IP
Security Group
Storage
```

Then:

```text
Launch Instance
```

Wait until the instance state becomes:

```text
Running
```

and the status checks pass.

---

# 🔎 Phase 9: Inspect the Instance

Open the instance details.

Identify:

```text
Instance ID

Public IPv4 Address

Private IPv4 Address

Public IPv4 DNS

Instance Type

VPC

Subnet

Availability Zone
```

The lesson reviews these values after the instance reaches the running state.

---

# 🧠 Validate the Network Placement

Record the following:

```text
VPC:
_____________________________

Subnet:
_____________________________

Availability Zone:
_____________________________

Private IP:
_____________________________

Public IP:
_____________________________
```

Then answer:

```text
Why does the EC2 instance have a Private IP?

Why does this EC2 instance also have a Public IP?

What makes the selected subnet Public?
```

---

# ❤️ Phase 10: Review EC2 Status Checks

Navigate to:

```text
Status and Alarms
```

EC2 performs two important health checks discussed in the course:

```text
System Status Check

Instance Status Check
```

---

# 🏗️ System Status Check

The **System Status Check** evaluates problems associated with the underlying AWS infrastructure supporting the instance.

Conceptually:

```text
EC2 Instance
     │
     ▼
AWS Physical Host
     │
     ▼
System Status Check
```

Examples described in the lesson include infrastructure-related issues such as networking, power, or host problems.

---

# 🖥️ Instance Status Check

The **Instance Status Check** evaluates the instance itself.

Conceptually:

```text
EC2 Instance
     │
     ▼
Operating System / Instance
     │
     ▼
Instance Status Check
```

The lesson associates failures here with issues inside the virtual machine, such as operating-system problems.

---

# 📊 Status Check Comparison

| Check                 | Focus                                       |
| --------------------- | ------------------------------------------- |
| System Status Check   | AWS infrastructure hosting the EC2 instance |
| Instance Status Check | Operating system / instance itself          |

---

# 🔑 Phase 11: Prepare the SSH Key

On macOS or Linux, navigate to the directory containing your `.pem` file.

Example:

```bash
cd ~/Downloads
```

Set restrictive permissions on the private key.

Example:

```bash
chmod 400 ec2-bastion-lab-key.pem
```

The course follows the permissions instructions presented by the EC2 Connect page before making the SSH connection.

---

# 💻 Phase 12: Connect Using SSH

Open:

```text
EC2
  │
  ▼
Instance
  │
  ▼
Connect
  │
  ▼
SSH Client
```

AWS provides the connection command.

It will look similar to:

```bash
ssh -i "ec2-bastion-lab-key.pem" ec2-user@<public-dns-name>
```

For Amazon Linux, the course uses:

```text
ec2-user
```

as the login user.

---

# ✅ Validate SSH Connectivity

Once connected:

```bash
whoami
```

Expected:

```text
ec2-user
```

Check your current directory:

```bash
pwd
```

You should be inside the EC2 user's home directory.

---

# 🧪 Phase 13: Test Operating System Connectivity

From the EC2 instance, the lesson demonstrates checking for package updates.

Use the equivalent package-manager command appropriate for your selected operating system.

The important learning objective is:

```text
EC2
 │
 ▼
Public Subnet
 │
 ▼
Internet Gateway
 │
 ▼
Internet
```

The instance should have outbound Internet connectivity because:

* It has a Public IP
* It is in a Public Subnet
* Its subnet Route Table points to the Internet Gateway

---

# 🌐 Phase 14: Connect Using EC2 Instance Connect

Return to:

```text
EC2
  │
  ▼
Connect
```

Choose:

```text
EC2 Instance Connect
```

Connect using the browser interface.

The lesson demonstrates EC2 Instance Connect as another way of reaching the public EC2 instance without using the local terminal SSH workflow.

Once connected:

```bash
pwd
```

and verify the shell works as expected.

---

# 🔄 Connection Methods Observed

This lesson introduces several possible EC2 connection methods:

```text
EC2 Connection Methods

├── SSH Client
│
├── EC2 Instance Connect
│
└── Systems Manager Session Manager
```

For this lab we use:

```text
SSH Client

and

EC2 Instance Connect
```

Session Manager requires additional configuration such as the appropriate IAM role and agent setup; the course defers that to a later lesson.

---

# 🏗️ Understanding the Bastion Pattern

The bastion host acts as an administrative entry point.

```text
Administrator
      │
      │ SSH
      ▼
Public EC2
Bastion Host
      │
      │ Private Network
      ▼
Private EC2
```

The private EC2 server does not require direct Internet exposure.

The lesson introduces this pattern and notes that private instances can instead be managed through services such as Session Manager when configured appropriately.

---

# 🔍 Phase 15: Architecture Review

You should now be able to explain this complete path:

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
Public Subnet
    │
    ▼
Security Group
    │
    ▼
Elastic Network Interface
    │
    ▼
EC2 Instance
    │
    ▼
EBS Root Volume
```

---

# 🧠 Component Relationships

A useful mental model is:

```text
EC2 Instance
     │
     ├── AMI
     │     └── Operating System
     │
     ├── Instance Type
     │     └── Compute Capacity
     │
     ├── ENI
     │     ├── Private IP
     │     ├── Public IP
     │     └── Security Group
     │
     ├── EBS
     │     └── Root Volume
     │
     ├── IAM Role
     │     └── AWS Permissions
     │
     └── User Data
           └── Startup Automation
```

This is the main architecture model you should remember from the lab.

---

# 💥 Troubleshooting Exercise 1: SSH Fails

Imagine:

```text
ssh: connect to host ... port 22: Operation timed out
```

Do not immediately terminate the server.

Check systematically:

```text
EC2 Running?
     │
     ▼
Public IP Assigned?
     │
     ▼
Public Subnet?
     │
     ▼
Route to IGW?
     │
     ▼
Security Group Allows 22?
     │
     ▼
Source IP Correct?
     │
     ▼
Correct Key?
     │
     ▼
Correct Username?
```

This begins developing a structured troubleshooting approach.

---

# 💥 Troubleshooting Exercise 2: Remove SSH Rule

Temporarily remove:

```text
TCP 22
Source: My IP
```

from the Security Group.

Try the SSH connection again.

Expected:

```text
FAIL
```

Restore the rule.

Test again.

Expected:

```text
SUCCESS
```

The objective is to prove:

```text
EC2 Running
      ≠
EC2 Reachable
```

Networking and Security Group rules still matter.

---

# 💥 Troubleshooting Exercise 3: Public IP

Observe the assigned Public IPv4 address.

The course explains that an automatically assigned public IPv4 address may change when an instance is stopped and subsequently started, whereas an Elastic IP is designed to remain static.

For this learning lab, record the current address:

```text
Before:
____________________
```

If you later experiment with stop/start behavior, compare the address afterward.

Do not rely on a normal auto-assigned public IP as a permanent endpoint.

---

# ❓ Interview Questions

### Q1. What components are required when launching an EC2 instance?

**Answer**

Common launch configuration includes:

* AMI
* Instance Type
* Key Pair
* VPC
* Subnet
* Network Interface
* Security Group
* Storage
* Optional IAM Role
* Optional User Data

---

### Q2. Why does an EC2 instance receive a Private IP address?

**Answer**

The Private IP enables the instance to communicate inside the VPC and is assigned from the selected subnet's CIDR range.

---

### Q3. Why does this bastion instance have a Public IP?

**Answer**

Because it is intentionally deployed in a Public Subnet and needs to be reachable for the remote-management exercise.

---

### Q4. What is an Elastic IP?

**Answer**

An Elastic IP is a static public IPv4 address allocated to your AWS account that can be associated with supported AWS resources.

---

### Q5. What is an AMI?

**Answer**

An Amazon Machine Image provides the software image used to launch an EC2 instance, including the operating system and potentially pre-installed applications.

---

### Q6. What is an ENI?

**Answer**

An Elastic Network Interface provides network connectivity to an EC2 instance.

---

### Q7. Where is a Security Group applied?

**Answer**

Security Groups control traffic associated with the EC2 instance's network interface.

---

### Q8. What is an EC2 Key Pair used for?

**Answer**

For Linux instances, the private key is used as part of SSH authentication.

For Windows instances, the private key can be used to decrypt the Administrator password.

---

### Q9. What is the purpose of an EBS root volume?

**Answer**

It provides block storage containing the operating system and other system data required by the EC2 instance.

---

### Q10. What is the difference between System Status Check and Instance Status Check?

**Answer**

The System Status Check focuses on the underlying AWS infrastructure.

The Instance Status Check focuses on the EC2 instance and its operating system.

---

### Q11. What is User Data?

**Answer**

User Data allows scripts to run during instance initialization and can be used to automate software installation and configuration.

---

### Q12. Why attach an IAM Role to EC2?

**Answer**

An IAM Role can provide the EC2 instance with permissions to access other AWS services and resources without embedding long-term AWS credentials on the server.

---

### Q13. Why should SSH not normally be open to `0.0.0.0/0`?

**Answer**

That would permit SSH connection attempts from anywhere on the Internet.

The lesson recommends restricting access to a specific IP or corporate CIDR instead.

---

# 🏁 Completion Criteria

Do not mark this lab complete until you can:

* [ ] Explain what an AMI is
* [ ] Explain what an Instance Type controls
* [ ] Create an EC2 Key Pair
* [ ] Explain Public vs Private IP
* [ ] Explain what an Elastic IP is
* [ ] Deploy EC2 into the correct VPC
* [ ] Select the correct subnet
* [ ] Explain why the subnet is public
* [ ] Attach a Security Group
* [ ] Explain how SG → ENI → EC2 relate
* [ ] Identify the root EBS volume
* [ ] Find the IAM Role configuration
* [ ] Find the User Data configuration
* [ ] Launch the EC2 instance
* [ ] Confirm System and Instance Status Checks pass
* [ ] Connect using SSH
* [ ] Run `whoami`
* [ ] Run `pwd`
* [ ] Connect using EC2 Instance Connect
* [ ] Break the SSH Security Group rule
* [ ] Troubleshoot and restore connectivity
* [ ] Explain the bastion-host pattern

---

# 💰 Cost Awareness

This lab can create chargeable AWS resources.

Review:

* EC2 instance runtime
* EBS storage
* Public IPv4 usage
* Elastic IP usage if created
* Data transfer

The course repeatedly emphasizes watching free-tier-related resource usage during the exercise, including the storage allocation used by EC2.

Always check the current AWS pricing and Free Tier terms associated with your own account.

---

# 🧹 Cleanup Checklist

After completing the lab:

* [ ] Terminate the EC2 instance
* [ ] Verify the EBS root volume is deleted if configured to delete on termination
* [ ] Delete the Bastion Host Security Group if no longer needed
* [ ] Delete the lab Key Pair from EC2 if no longer needed
* [ ] Securely remove the downloaded private key if the lab is finished
* [ ] Release any Elastic IP created specifically for the lab
* [ ] Verify no unnecessary EC2 resources remain running

---

# 💡 Key Takeaways

* EC2 instances require several supporting components.
* An AMI defines the software image.
* An Instance Type defines compute capacity.
* A subnet determines where the EC2 instance is deployed.
* EC2 receives a Private IP from the subnet CIDR.
* Public instances may also receive a Public IP.
* An Elastic IP provides a static public IP.
* The ENI provides network connectivity.
* Security Groups protect EC2 network interfaces.
* EBS provides block storage.
* IAM Roles can grant AWS permissions to EC2.
* User Data can automate first-boot configuration.
* EC2 Status Checks help distinguish infrastructure problems from instance problems.
* SSH and EC2 Instance Connect provide remote access options for this public-instance lab.
* Bastion hosts can provide a jump point toward private resources, although the course also introduces Systems Manager Session Manager as another management option.
* Successful EC2 deployment requires understanding networking, security, compute, storage, and access together.

> The goal of this lab is not simply to launch a virtual server. The goal is to understand every component you configured and why the EC2 instance depends on it.
