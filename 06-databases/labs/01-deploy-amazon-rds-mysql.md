# 🧪 Lab: Deploy an Amazon RDS MySQL Database

> In this lab, I will deploy a MySQL database using Amazon RDS inside private database subnets in my VPC. I will create a DB Subnet Group, configure the RDS instance, review its endpoint and monitoring information, and clean up the resources after completing the lab.

---

# 🎯 Lab Objectives

By the end of this lab, I should be able to:

* Create an RDS DB Subnet Group
* Select database subnets across two Availability Zones
* Deploy a MySQL database using Amazon RDS
* Choose a DB instance class and storage configuration
* Deploy RDS inside an existing VPC
* Keep the database private
* Attach an existing database security group
* Understand the RDS endpoint and port
* Review the database configuration
* Review basic RDS monitoring information
* Understand automated backup settings
* Delete the RDS database after completing the lab

---

# 🏗️ Target Architecture

For this lab, I will use two Availability Zones.

```text
                         AWS Region
                             │
                             ▼
                            VPC
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
             AZ-A                          AZ-B
              │                             │
              ▼                             ▼
      Private Data Subnet 1        Private Data Subnet 2
              │                             │
              └──────────────┬──────────────┘
                             │
                             ▼
                      DB Subnet Group
                             │
                             ▼
                       Amazon RDS
                          MySQL
                             │
                             ▼
                         TCP 3306
```

For this exercise:

```text
Deployment
    │
    ▼
Single RDS Instance
```

I will **not configure Multi-AZ yet**.

Multi-AZ will be handled separately as a project exercise.

---

# 📋 Prerequisites

Before starting this lab, I should already have:

* An AWS account
* A VPC
* Two Availability Zones
* A private/data subnet in each AZ
* An application/web security group
* A database security group
* Access to the AWS Management Console

The database subnets should conceptually look like:

```text
VPC
│
├── AZ-A
│    └── Data Subnet 1
│
└── AZ-B
     └── Data Subnet 2
```

---

# 🔐 Database Security Group

The database security group should allow:

```text
Protocol: TCP
Port:     3306
Source:   Application/Web Security Group
```

Conceptually:

```text
Application Server
       │
       │ MySQL TCP 3306
       ▼
Database Security Group
       │
       ▼
     RDS MySQL
```

Instead of:

```text
0.0.0.0/0
     │
     ▼
TCP 3306
```

the source should be the security group associated with the application servers.

For example:

```text
Inbound Rule

Type        MySQL/Aurora
Protocol    TCP
Port        3306
Source      Web/App Security Group
```

This limits database access to resources associated with the intended application security group.

---

# 📝 Record My Environment

Before starting, record the values I will use.

| Setting                    | My Value |
| -------------------------- | -------- |
| AWS Region                 |          |
| VPC                        |          |
| Availability Zone A        |          |
| Availability Zone B        |          |
| Data Subnet 1              |          |
| Data Subnet 2              |          |
| Application Security Group |          |
| Database Security Group    |          |

This will help prevent accidentally deploying resources into the wrong VPC or subnet.

---

# 🚀 Phase 1: Verify the Database Subnets

Open:

```text
AWS Management Console
        │
        ▼
VPC
        │
        ▼
Subnets
```

Locate the two subnets that will be used for the database.

Verify that:

```text
Data Subnet 1
      │
      ▼
Availability Zone A


Data Subnet 2
      │
      ▼
Availability Zone B
```

The important requirement for this lab is:

```text
Two Database Subnets
        │
        ▼
Two Availability Zones
```

---

# ✅ Validation Checkpoint 1

Before continuing, confirm:

* [ ] Correct VPC selected
* [ ] Data Subnet 1 exists
* [ ] Data Subnet 2 exists
* [ ] Subnets are in different Availability Zones

Do not continue until these are confirmed.

---

# 🔐 Phase 2: Verify the Database Security Group

Navigate to:

```text
VPC
 │
 ▼
Security Groups
```

Locate the database security group.

Open:

```text
Inbound Rules
```

Verify that MySQL traffic is allowed.

Expected rule:

```text
Type:       MySQL/Aurora
Protocol:   TCP
Port:       3306
Source:     Application/Web Security Group
```

Architecture:

```text
Web/App SG
    │
    │ TCP 3306
    ▼
Database SG
    │
    ▼
RDS
```

---

# 💡 Why Reference a Security Group?

Instead of allowing database connections from a broad network range, I can explicitly allow connections from resources associated with the application security group.

Conceptually:

```text
Only Application Resources
           │
           ▼
       TCP 3306
           │
           ▼
         MySQL
```

This provides more controlled access to the database.

---

# ✅ Validation Checkpoint 2

Confirm:

* [ ] Database Security Group exists
* [ ] TCP 3306 is allowed
* [ ] Source is the intended Web/App Security Group
* [ ] Database is not unnecessarily open to the internet

---

# 🗂️ Phase 3: Create the DB Subnet Group

Now navigate to:

```text
AWS Management Console
        │
        ▼
Amazon RDS
```

From the RDS navigation menu, select:

```text
Subnet Groups
```

Click:

```text
Create DB subnet group
```

---

# 📝 Configure the DB Subnet Group

Provide a name.

Example:

```text
db-subnet-group
```

Provide a description.

Example:

```text
Database subnets for RDS lab
```

Select the:

```text
VPC
```

created for the lab environment.

---

# 🌎 Select Availability Zones

Select:

```text
Availability Zone A

Availability Zone B
```

Do not select an unrelated Availability Zone.

---

# 🌐 Select Subnets

Select the database/data subnet from each Availability Zone.

```text
DB Subnet Group
│
├── Data Subnet 1
│      └── AZ-A
│
└── Data Subnet 2
       └── AZ-B
```

Click:

```text
Create
```

---

# ✅ Validation Checkpoint 3

Open the newly created DB Subnet Group.

Verify:

```text
Correct VPC
     │
     ▼
Correct AZs
     │
     ▼
Correct Data Subnets
```

Checklist:

* [ ] DB Subnet Group created
* [ ] Correct VPC
* [ ] Two subnets selected
* [ ] Subnets are across two Availability Zones

---

# 🧠 Why Do We Need a DB Subnet Group?

The DB Subnet Group tells Amazon RDS:

> These are the subnets that may be used for my database deployment.

Conceptually:

```text
VPC
│
├── AZ-A
│    └── Data Subnet 1
│
└── AZ-B
     └── Data Subnet 2
          │
          ▼
     DB Subnet Group
          │
          ▼
       Amazon RDS
```

It also prepares the network architecture for features such as Multi-AZ, which will be explored separately.

---

# 🗄️ Phase 4: Create the RDS Database

Navigate to:

```text
Amazon RDS
    │
    ▼
Databases
```

Click:

```text
Create database
```

---

# ⚙️ Phase 5: Select the Database Engine

Choose:

```text
MySQL
```

For this lab, use the default engine version presented by the course environment unless there is a specific requirement to select another version.

Record the selected version:

```text
MySQL Version: __________________
```

---

# 💰 Phase 6: Select the Template

The course exercise uses:

```text
Free Tier
```

where available.

The goal of the lab is to deploy a simple single database instance without configuring Multi-AZ.

Do not enable:

```text
Multi-AZ
```

for this exercise.

---

# 📝 Phase 7: Configure Database Settings

Create a DB instance identifier.

Example:

```text
rds-lab-db
```

Record it:

```text
DB Identifier: ______________________
```

---

# 👤 Configure Credentials

For this lab, use:

```text
Self-managed credentials
```

rather than Secrets Manager.

Configure the master username and password.

Example:

```text
Master Username: admin
```

Use a password that meets the console requirements.

Do not store the real password inside the GitHub lab documentation.

Instead document:

```text
Master Username: admin
Password: <stored securely>
```

---

# 🔐 Credential Security

For this exercise:

```text
Self-Managed Credentials
```

are used to keep the lab simple.

The course will cover:

```text
AWS Secrets Manager
```

separately.

In a real environment, database credentials should not be hard-coded into:

```text
Source Code

GitHub

Scripts

Configuration Files
```

---

# 🖥️ Phase 8: Choose the DB Instance Class

Under instance configuration, select the small burstable instance used by the lab.

The course example uses:

```text
db.t3.micro
```

where available.

Conceptually:

```text
RDS
 │
 ▼
DB Instance Class
 │
 ▼
db.t3.micro
```

This determines resources such as:

```text
CPU

Memory

Network Capacity
```

---

# 💾 Phase 9: Configure Storage

The course lab uses:

```text
Storage Type:
General Purpose SSD (gp2)

Allocated Storage:
20 GiB
```

Record the values actually selected:

| Setting           | Value |
| ----------------- | ----- |
| Storage Type      |       |
| Allocated Storage |       |

---

# 📈 Storage Auto Scaling

The lab does not require:

```text
Storage Auto Scaling
```

so leave it disabled for this exercise.

Storage Auto Scaling allows RDS to increase database storage when additional capacity is required.

That feature can be explored separately.

---

# 🌐 Phase 10: Configure Connectivity

Under connectivity, configure the database to use the existing VPC environment.

For this exercise, choose not to automatically connect the database to an EC2 compute resource.

Conceptually:

```text
RDS
 │
 ▼
Existing VPC Configuration
```

---

# 🏗️ Select the VPC

Choose the VPC containing the database subnets.

Double-check this selection carefully.

The lesson highlights an important point:

```text
RDS Created
     │
     ▼
VPC Cannot Simply Be
Changed Afterwards
```

Therefore, verify the VPC before creating the database.

---

# 🗂️ Select the DB Subnet Group

Select the DB Subnet Group created earlier.

Example:

```text
db-subnet-group
```

Verify that it contains:

```text
Data Subnet 1 → AZ-A

Data Subnet 2 → AZ-B
```

---

# 🌍 Public Access

Configure:

```text
Public Access: No
```

The database is being deployed into the private database layer.

Conceptually:

```text
Internet
   │
   ✕
   │
   ▼
RDS Database
```

Instead:

```text
Application
     │
     │ TCP 3306
     ▼
RDS Database
```

---

# 🔐 Phase 11: Configure the Security Group

Choose:

```text
Existing Security Group
```

Select the:

```text
Database Security Group
```

created previously.

Remove the default security group from the configuration if it is not required for the lab.

Verify:

```text
RDS
 │
 ▼
Database Security Group
 │
 ▼
Inbound TCP 3306
from Web/App SG
```

---

# 🌎 Phase 12: Select Availability Zone

For this single-instance lab, select one of the Availability Zones used by the DB Subnet Group.

For example:

```text
Availability Zone A
```

The database will therefore be deployed in the corresponding subnet.

Conceptually:

```text
DB Subnet Group
│
├── AZ-A
│    └── Data Subnet 1
│         │
│         ▼
│       RDS
│
└── AZ-B
     └── Data Subnet 2
```

Remember:

```text
Multi-AZ = Not Enabled
```

for this lab.

---

# ⚙️ Phase 13: Advanced Configuration

Expand:

```text
Additional Configuration
```

or the equivalent advanced configuration section shown by the console.

Specify the initial database name.

Example:

```text
rds_lab_db
```

Record it:

```text
Initial Database Name: __________________
```

Do not confuse:

```text
DB Instance Identifier
```

with:

```text
Initial Database Name
```

They represent different things.

---

# 💾 Phase 14: Configure Backups

The lesson uses an automated backup retention period of:

```text
1 Day
```

Configure:

```text
Backup Retention Period: 1 day
```

for this temporary lab.

The allowed backup-retention settings shown in the lesson range from:

```text
0 to 35 days
```

For real workloads, the backup strategy should be based on actual recovery requirements rather than this temporary lab configuration.

---

# 🔐 Encryption

Leave the encryption configuration at the lab's default setting.

Record what the console shows:

```text
Encryption: __________________
```

This gives me something to verify later from the database configuration page.

---

# 🚀 Phase 15: Create the Database

Before creating the database, review the configuration.

Checklist:

* [ ] MySQL selected
* [ ] Correct DB identifier
* [ ] Credentials configured
* [ ] Correct DB instance class
* [ ] Storage configured
* [ ] Correct VPC
* [ ] Correct DB Subnet Group
* [ ] Public access disabled
* [ ] Correct Database Security Group
* [ ] Single-AZ configuration
* [ ] Initial database name configured
* [ ] Backup retention configured

Then click:

```text
Create database
```

---

# ⏳ Phase 16: Wait for RDS Provisioning

The database will take several minutes to provision.

Monitor:

```text
Amazon RDS
    │
    ▼
Databases
    │
    ▼
My Database
```

The status will eventually become:

```text
Available
```

Do not try to use the database until it reaches the available state.

---

# ✅ Validation Checkpoint 4

Confirm:

```text
Database Status
      │
      ▼
AVAILABLE
```

Checklist:

* [ ] RDS instance created successfully
* [ ] Status is Available
* [ ] No creation errors

---

# 🔎 Phase 17: Inspect Connectivity and Security

Open the database.

Navigate to:

```text
Connectivity & Security
```

Locate the:

```text
Endpoint

Port
```

For MySQL, the default port is:

```text
3306
```

Record the information:

```text
RDS Endpoint:
_______________________________________

Port:
3306
```

---

# 🧠 What Is the RDS Endpoint?

Applications normally connect to RDS using the:

```text
DNS Endpoint
```

Conceptually:

```text
Application
     │
     │ DNS Endpoint
     ▼
Amazon RDS
     │
     ▼
MySQL
```

The application connection information therefore includes:

```text
Endpoint

Port

Database Name

Username

Password
```

---

# 🧪 Example Connection Concept

Conceptually, an application needs:

```text
Host     = RDS Endpoint

Port     = 3306

Database = Initial Database Name

Username = Master Username

Password = Secure Password
```

The endpoint is the address applications use to communicate with the database.

---

# 🔐 Phase 18: Verify Security Configuration

From:

```text
Connectivity & Security
```

verify:

```text
VPC

DB Subnet Group

Security Group

Availability Zone

Public Accessibility
```

Expected architecture:

```text
Correct VPC
    │
    ▼
DB Subnet Group
    │
    ▼
Private Database Subnet
    │
    ▼
RDS MySQL
    │
    ▼
Database Security Group
```

Confirm:

```text
Publicly Accessible
        │
        ▼
       No
```

---

# ⚙️ Phase 19: Review Database Configuration

Open the:

```text
Configuration
```

section.

Review:

```text
DB Instance Class

Database Engine

Database Name

Storage

Encryption

Multi-AZ
```

For this lab:

```text
Multi-AZ
   │
   ▼
No
```

because this is intentionally a single-instance deployment.

---

# 📊 Phase 20: Review Monitoring

Open:

```text
Monitoring
```

RDS exposes monitoring information through CloudWatch.

Depending on the available metrics, I may see information related to:

```text
CPU

Connections

Storage

Database Performance
```

The detailed CloudWatch implementation will be covered later.

For now, understand:

```text
RDS
 │
 ▼
CloudWatch Metrics
 │
 ▼
Observe Database
Performance
```

---

# 📜 Phase 21: Review Logs and Events

Review the:

```text
Logs & Events
```

section.

This area provides information about database-related events and available logs.

Conceptually:

```text
RDS
 │
 ├── Events
 │
 └── Logs
       │
       ▼
Monitoring /
Troubleshooting
```

The lesson also introduces the idea that database logs can be integrated with CloudWatch for analysis and troubleshooting.

---

# 💾 Phase 22: Review Maintenance and Backups

Navigate to:

```text
Maintenance & Backups
```

Verify the backup configuration.

For this lab:

```text
Automated Backup Retention
          │
          ▼
        1 Day
```

This is a temporary lab configuration.

A real production database would require a backup strategy based on the organization's recovery requirements.

---

# 📸 Recommended Evidence

Capture screenshots for the GitHub lab documentation.

Suggested screenshots:

### Screenshot 1

DB Subnet Group

Show:

```text
Two Subnets

Two Availability Zones
```

### Screenshot 2

RDS Database Status

Show:

```text
Available
```

### Screenshot 3

Connectivity & Security

Show:

```text
Endpoint

Port

VPC

Security Group
```

Avoid exposing sensitive credentials.

### Screenshot 4

Database Configuration

Show:

```text
Engine

DB Instance Class

Storage

Multi-AZ Status
```

### Screenshot 5

Monitoring

Show available RDS monitoring information.

---

# 🧪 Validation Checklist

Before considering the lab complete:

* [ ] DB Subnet Group exists
* [ ] Two database subnets are included
* [ ] Subnets span two Availability Zones
* [ ] MySQL RDS instance created
* [ ] Correct VPC selected
* [ ] Correct DB Subnet Group selected
* [ ] Database is not publicly accessible
* [ ] Database Security Group attached
* [ ] MySQL port is 3306
* [ ] RDS status is Available
* [ ] RDS endpoint identified
* [ ] Database configuration reviewed
* [ ] Monitoring reviewed
* [ ] Logs and events reviewed
* [ ] Backup configuration reviewed
* [ ] Multi-AZ is disabled for this lab

---

# 🔧 Troubleshooting

## Problem 1: Cannot Select the Expected Subnets

Check:

```text
Correct VPC?
    │
    ▼
Correct Availability Zones?
    │
    ▼
Correct Subnets?
```

The DB Subnet Group should contain the intended database subnets.

---

## Problem 2: Wrong DB Subnet Group Appears

Verify that the DB Subnet Group was created in:

```text
The Same VPC
```

selected for the RDS database.

---

## Problem 3: Database Cannot Be Reached

Check:

```text
Application Security Group
        │
        ▼
Database SG Inbound Rule
        │
        ▼
TCP 3306
```

The database security group should permit MySQL traffic from the intended application security group.

---

## Problem 4: Database Is Publicly Accessible

Review:

```text
Connectivity
     │
     ▼
Public Access
```

For this lab it should be:

```text
No
```

---

## Problem 5: Database Remains in Creating State

RDS provisioning takes time.

Monitor:

```text
Databases
    │
    ▼
Status
```

and wait until:

```text
Available
```

before continuing.

---

## Problem 6: Cannot Find the Endpoint

Navigate to:

```text
RDS
 │
 ▼
Databases
 │
 ▼
Select Database
 │
 ▼
Connectivity & Security
```

Look for:

```text
Endpoint & Port
```

---

# 🧹 Phase 23: Cleanup

This lab creates billable AWS resources, so cleanup is important.

Navigate to:

```text
Amazon RDS
    │
    ▼
Databases
```

Select the lab database.

Choose:

```text
Actions
   │
   ▼
Delete
```

---

# 📸 Final Snapshot

The console may ask whether I want to create a final snapshot.

For this temporary lab:

```text
Final Snapshot
      │
      ▼
Do Not Create
```

because the database is no longer needed.

For a real database, this decision would be very different.

---

# 💾 Automated Backups

For this disposable exercise, do not retain automated backups after deletion if they are no longer required.

The purpose is to avoid keeping unnecessary storage resources after the lab.

---

# ⚠️ Confirm Before Deleting

Before clicking Delete, verify:

```text
Is this definitely
the lab database?
       │
       ▼
      YES
```

Then enter the required deletion confirmation and delete the database.

---

# 🧹 Optional DB Subnet Group Cleanup

After the RDS database has been deleted, the DB Subnet Group can also be removed if it was created only for this lab and will not be reused.

Do not delete shared networking resources that are required by other labs.

---

# 💰 Cost Awareness

Resources that can contribute to cost include:

```text
RDS DB Instance

Database Storage

Backup Storage

Snapshots
```

For this lab:

```text
Create
  │
  ▼
Learn
  │
  ▼
Validate
  │
  ▼
Delete
```

Do not leave the temporary RDS database running unnecessarily.

---

# 🏆 Completion Criteria

I have successfully completed this lab when I can demonstrate:

```text
1. DB Subnet Group
        │
        ▼
Two AZs


2. RDS MySQL
        │
        ▼
Private Database


3. Database Security Group
        │
        ▼
TCP 3306 from App SG


4. RDS Endpoint
        │
        ▼
Identified


5. Configuration
        │
        ▼
Verified


6. Monitoring
        │
        ▼
Reviewed


7. Cleanup
        │
        ▼
Completed
```

---

# ❓ Interview Questions

### Q1. Why does Amazon RDS need a DB Subnet Group?

The DB Subnet Group identifies the subnets that RDS can use to deploy database resources.

---

### Q2. Why does the DB Subnet Group contain subnets from multiple Availability Zones?

It provides RDS with subnet options across Availability Zones and prepares the architecture for capabilities such as Multi-AZ.

---

### Q3. Why is the database placed in a private subnet?

The application should communicate with the database through the VPC without unnecessarily exposing the database directly to the internet.

---

### Q4. Which port does MySQL use by default?

```text
TCP 3306
```

---

### Q5. Why use the application security group as the source of the database security group rule?

It restricts database access to resources associated with the intended application security group rather than allowing broad network access.

---

### Q6. What is the RDS endpoint?

It is the DNS endpoint applications use to connect to the RDS database.

---

### Q7. What information does an application need to connect to the database?

Typically:

```text
Endpoint

Port

Database Name

Username

Password
```

---

### Q8. Why did we disable public access?

Because this lab places the database in the private database layer rather than exposing it directly to the internet.

---

### Q9. Did we configure Multi-AZ?

No.

This lab intentionally deploys a single database instance.

Multi-AZ will be explored separately.

---

### Q10. What does the DB instance class determine?

It determines the compute capacity available to the database, including resources such as CPU and memory.

---

### Q11. What storage configuration was used in the course lab?

```text
General Purpose SSD (gp2)

20 GiB
```

---

### Q12. Why did we review the Monitoring tab?

To see the monitoring information available for the RDS database and understand that RDS integrates with CloudWatch metrics.

---

### Q13. Why should temporary lab databases be deleted?

Because RDS instances, storage, backups, and snapshots can contribute to AWS costs.

---

### Q14. Why did we avoid creating a final snapshot during cleanup?

Because this is a disposable lab database and its data does not need to be retained.

---

# 💡 Key Takeaways

* Amazon RDS is a managed relational database service.
* RDS databases are deployed inside a VPC.
* A DB Subnet Group defines the subnets available to RDS.
* The lab uses database subnets across two Availability Zones.
* The database itself is deployed as a single instance for this exercise.
* MySQL is used as the database engine.
* The DB instance class determines the database compute capacity.
* Compute and storage are configured separately.
* The course lab uses 20 GiB of General Purpose SSD storage.
* The database is not publicly accessible.
* A Database Security Group controls inbound database connectivity.
* MySQL uses TCP port 3306 by default.
* The application security group is used as the source for the database inbound rule.
* Applications connect to RDS using its DNS endpoint.
* RDS exposes monitoring information through CloudWatch.
* Automated backups can be configured with a retention period.
* Multi-AZ is intentionally left for a later exercise.
* Temporary RDS resources should be deleted after the lab to avoid unnecessary costs.

The complete lab flow is:

```text
Verify VPC
    │
    ▼
Verify Data Subnets
    │
    ▼
Verify Database SG
    │
    ▼
Create DB Subnet Group
    │
    ▼
Select MySQL
    │
    ▼
Choose DB Instance Class
    │
    ▼
Configure Storage
    │
    ▼
Select VPC
    │
    ▼
Select DB Subnet Group
    │
    ▼
Disable Public Access
    │
    ▼
Attach Database SG
    │
    ▼
Create RDS
    │
    ▼
Wait for Available
    │
    ▼
Find Endpoint
    │
    ▼
Review Configuration
    │
    ▼
Review Monitoring
    │
    ▼
Review Backups
    │
    ▼
Delete RDS
    │
    ▼
Verify Cleanup
```

---

# 📚 Related Topics

* Amazon RDS
* Relational Databases
* MySQL
* Amazon VPC
* Private Subnets
* Security Groups
* DB Subnet Groups
* RDS DB Instance Classes
* RDS Storage
* RDS Endpoints
* RDS Automated Backups
* Amazon CloudWatch
* RDS Multi-AZ
* RDS Read Replicas
* AWS Secrets Manager
