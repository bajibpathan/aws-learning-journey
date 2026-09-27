# 🛠️ Amazon RDS Custom

> Amazon RDS Custom provides a middle ground between running a self-managed database on Amazon EC2 and using the fully managed Amazon RDS service. It provides access to the underlying operating system and database environment when applications require deeper customization.

---

# 📖 Overview

Normally, I have two main approaches for running a relational database on AWS:

```text
Relational Database on AWS
          │
     ┌────┴────┐
     │         │
     ▼         ▼
    EC2       RDS
     │         │
     ▼         ▼
More         More
Control      Managed
```

These approaches involve a trade-off.

With EC2:

```text
More Control
     +
More Operational Responsibility
```

With standard RDS:

```text
Less Administrative Work
     +
Less OS-Level Control
```

But what if my application needs:

```text
OS-Level Customization

Database-Level Administrative Access

Custom Libraries

Custom Dependencies

Specific Database Configuration
```

while I still want some of the management capabilities provided by RDS?

This is where:

```text
Amazon RDS Custom
```

comes in.

---

# 🎯 The Problem RDS Custom Solves

Suppose I need to run Microsoft SQL Server.

One option is:

```text
Amazon EC2
    │
    ▼
Windows
    │
    ▼
SQL Server
```

I install and manage the database myself.

The other option is:

```text
Amazon RDS
    │
    ▼
SQL Server
```

where AWS manages much more of the environment.

These provide very different levels of control.

---

# 🖥️ Option 1: Database on Amazon EC2

When running the database on EC2:

```text
EC2 Instance
     │
     ▼
Operating System
     │
     ▼
Database Software
```

I have access to:

```text
Operating System

Database Software

File System

Database Configuration
```

This gives me significant flexibility.

However, it also creates more operational responsibility.

---

# 🧑‍💻 Responsibilities with a Database on EC2

If I run SQL Server, Oracle, or another database myself on EC2, I need to manage areas such as:

```text
Database Installation

Database Configuration

Database Backups

Backup Storage

Database Patching

Operating System Patching

Scaling

High Availability

Database Maintenance
```

AWS manages the underlying physical infrastructure.

Conceptually:

```text
AWS Responsibility
        │
        ├── Physical Servers
        ├── Hardware Lifecycle
        ├── Power
        ├── Cooling
        └── Physical Networking


My Responsibility
        │
        ├── Operating System
        ├── Database
        ├── Patching
        ├── Backups
        ├── Scaling
        └── High Availability
```

This gives me control, but also creates substantial administrative overhead.

---

# ☁️ Option 2: Standard Amazon RDS

With standard RDS:

```text
Amazon RDS
    │
    ▼
Managed Database
```

AWS takes care of much more of the database infrastructure and operational management.

This reduces the amount of administration required from my database team.

Conceptually:

```text
Application
     │
     ▼
Amazon RDS
     │
     ▼
AWS-Managed Database Environment
```

---

# 🚫 Limitation of Standard RDS

The trade-off is that standard RDS does not provide access to the underlying operating system.

Conceptually:

```text
Standard RDS
│
├── Database Access ✅
│
├── Choose Engine Version ✅
│
├── OS Access ❌
│
└── Deep OS Customization ❌
```

I cannot simply connect to the underlying server and make arbitrary operating-system changes.

The environment is managed by AWS.

---

# 🧠 Why Could This Be a Problem?

Many modern applications work perfectly well with a standard managed RDS database.

But some applications may have special requirements.

For example:

```text
Legacy Application
        │
        ▼
Requires Specific
Database Configuration
```

or:

```text
Application
    │
    ▼
Requires Additional
Libraries / Dependencies
```

or:

```text
Database
    │
    ▼
Requires OS-Level
Configuration
```

In those situations, standard RDS may not provide enough administrative control.

---

# 🤔 The Traditional Choice

Without RDS Custom, I may need to choose between:

```text
EC2
 │
 ▼
Full Control
 │
 ▼
High Operational Overhead
```

and:

```text
Standard RDS
     │
     ▼
Managed Service
     │
     ▼
Limited OS-Level Control
```

RDS Custom provides another option.

---

# 🛠️ What Is Amazon RDS Custom?

Amazon RDS Custom provides:

```text
Managed RDS Capabilities
          +
Additional Administrative Access
```

Conceptually:

```text
             RDS Custom
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
RDS Management      OS / Database
Capabilities           Access
```

This provides a middle ground between:

```text
Self-Managed EC2
```

and:

```text
Fully Managed Standard RDS
```

---

# 🏗️ RDS Custom Architecture

RDS Custom still operates inside:

```text
Amazon VPC
```

Conceptually:

```text
                    VPC
                     │
                     ▼
              RDS Custom
               Instance
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    Operating System       Database
       Access               Access
```

Unlike standard RDS, administrators can access and customize more of the environment.

---

# 🗄️ Supported Database Engines

The lesson describes RDS Custom support for:

```text
Oracle

Microsoft SQL Server
```

Conceptually:

```text
RDS Custom
│
├── Oracle
│
└── Microsoft SQL Server
```

The lesson specifically states that RDS Custom is not available for:

```text
MySQL

PostgreSQL

Amazon Aurora
```

---

# 🔓 Operating System Access

One of the major benefits of RDS Custom is:

```text
Operating System Access
```

This means an administrator can perform changes such as:

```text
Install Dependencies

Install Libraries

Configure the OS

Apply Custom Packages

Configure Application Requirements
```

For example:

```text
Application
     │
     ▼
Requires Library
     │
     ▼
RDS Custom Instance
     │
     ▼
Install Required Library
```

This type of customization is not available with the standard RDS model described in the lesson.

---

# 🗃️ Database Administrative Access

RDS Custom also provides administrative access to the database environment.

This allows the database administrator to customize the database according to application requirements.

Conceptually:

```text
DB Administrator
       │
       ▼
RDS Custom
       │
       ▼
Database
       │
       ▼
Custom Configuration
```

This can be important when legacy or specialized applications require specific database configurations.

---

# 💾 Storage and File System Access

The lesson also introduces access to the underlying storage environment.

This allows administrators to work with:

```text
Volumes

File Systems

Database Files
```

For example:

```text
RDS Custom
    │
    ▼
Underlying Volume
    │
    ▼
File System
    │
    ▼
Database Requirements
```

This provides additional flexibility compared with the standard managed RDS environment described in the lesson.

---

# 🔌 Connecting to RDS Custom

The lesson introduces several methods for accessing the custom instance.

Depending on the operating system, administrators can use:

```text
SSH

RDP

AWS Systems Manager
Session Manager
```

Conceptually:

```text
Database Administrator
          │
     ┌────┼────┐
     │    │    │
     ▼    ▼    ▼
    SSH  RDP  Session
              Manager
                 │
                 ▼
            RDS Custom
```

Once connected, the administrator can perform the required customizations.

---

# 🪟 Example: SQL Server

For Microsoft SQL Server, the architecture could conceptually look like:

```text
Database Administrator
          │
          ▼
         RDP
          │
          ▼
RDS Custom Instance
          │
          ▼
Windows
          │
          ▼
Microsoft SQL Server
```

The administrator can then perform the custom configuration required by the application.

---

# 🛠️ What Can I Customize?

Examples introduced in the lesson include:

```text
Operating System Settings

Database Settings

Database Patches

OS Patches

Custom Packages

Dependencies

Libraries

File Systems
```

For example:

```text
Legacy Application
        │
        ▼
Requires Special
Database Dependency
        │
        ▼
RDS Custom
        │
        ▼
Install Dependency
```

This is one of the main reasons RDS Custom exists.

---

# 🤖 RDS Custom Automation and Monitoring

Even though RDS Custom provides more administrative control, AWS still provides:

```text
RDS Custom
Automation and Monitoring
```

The lesson describes this as software that operates outside the DB instance and communicates with components and agents associated with the custom environment.

Conceptually:

```text
RDS Custom Automation
        │
        │ Communicates
        ▼
RDS Custom Instance
        │
        ▼
Database Environment
```

---

# 🔍 What Does the Automation System Do?

The lesson introduces responsibilities such as:

```text
Collect Metrics

Send Notifications

Monitor Instance Health

Perform Instance Recovery
```

Conceptually:

```text
RDS Custom
Automation
    │
    ├── Monitor
    │
    ├── Collect Metrics
    │
    ├── Notify
    │
    └── Recover
```

---

# 🚑 Automatic Instance Recovery

One important responsibility of the automation system is responding to problems with the underlying instance.

For example:

```text
RDS Custom Instance
        │
        ▼
Impaired / Unreachable
        │
        ▼
Automation Detects Problem
        │
        ▼
Recovery Action
```

The lesson describes recovery actions such as:

```text
Reboot Instance

or

Replace Instance
```

This allows RDS Custom to retain some managed-service capabilities even though administrators have deeper access to the environment.

---

# ⚙️ Full Automation Mode

The lesson describes the automation system as being enabled by default in:

```text
Full Automation Mode
```

Conceptually:

```text
RDS Custom
    │
    ▼
Full Automation
    │
    ├── Monitoring
    ├── Metrics
    ├── Notifications
    └── Recovery
```

Under normal operation, the automation system monitors and manages the environment.

---

# ⏸️ Pausing RDS Custom Automation

There are situations where an administrator needs to modify the environment.

For example:

```text
Install Custom Database Patch

Install OS Package

Change Database Settings

Configure File System

Install Dependency
```

Before performing these customizations, the lesson instructs administrators to:

```text
Pause Automation
```

Conceptually:

```text
RDS Custom Automation
         │
         ▼
       PAUSE
         │
         ▼
Perform Customization
```

---

# 🛠️ Customization Workflow

The workflow introduced in the lesson is:

```text
RDS Custom Running
        │
        ▼
Pause Automation
        │
        ▼
Perform Customization
        │
        ├── OS Changes
        ├── DB Changes
        ├── Install Packages
        └── Configure File Systems
        │
        ▼
Resume Automation
```

This is an important RDS Custom operational concept.

---

# ▶️ Resuming Automation

After completing the customization:

```text
Customization Complete
        │
        ▼
Resume Automation
```

The lesson describes two ways this can happen:

```text
Manually Resume

or

Pause Period Ends
```

Once resumed:

```text
RDS Custom Automation
        │
        ▼
Monitoring
        +
Instance Recovery
```

continues.

---

# 🧠 Why Pause Automation?

The important idea is that AWS automation is actively monitoring and managing the RDS Custom environment.

If I intentionally make administrative changes:

```text
Administrator
     │
     ▼
Changes OS / Database
```

I first pause the automation so that I can perform the required customization.

Then:

```text
Finish Changes
     │
     ▼
Resume Automation
```

and AWS can continue monitoring and recovery activities.

---

# ⚖️ The Middle Ground

The easiest way to understand RDS Custom is:

```text
                Database Options
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
       EC2         RDS Custom      Standard RDS
        │              │              │
        ▼              ▼              ▼
Most Control      More Control      Least OS
                                      Access
        │              │              │
        ▼              ▼              ▼
Most Management   Some Managed     Most Managed
Responsibility    Capabilities     Experience
```

RDS Custom sits between EC2 and standard RDS.

---

# 🆚 EC2 vs RDS Custom vs Standard RDS

| Capability                     | Database on EC2  | RDS Custom            | Standard RDS             |
| ------------------------------ | ---------------- | --------------------- | ------------------------ |
| OS Access                      | ✅                | ✅                     | ❌                        |
| Database Administrative Access | ✅                | ✅                     | Limited by managed model |
| Custom OS Configuration        | ✅                | ✅                     | ❌                        |
| Install Custom Dependencies    | ✅                | ✅                     | ❌                        |
| AWS Managed Capabilities       | Limited          | ✅                     | ✅                        |
| Automation & Monitoring        | Customer manages | RDS Custom automation | AWS managed              |
| Operational Responsibility     | Highest          | Middle                | Lowest                   |
| Deep Customization             | Highest          | High                  | Limited                  |

The key idea is:

```text
EC2
=
Maximum Control
+
Maximum Responsibility


Standard RDS
=
Managed Experience
+
Limited OS Control


RDS Custom
=
Customization
+
Managed Capabilities
```

---

# 🎯 When Would I Use RDS Custom?

RDS Custom makes sense when my application requires:

```text
OS-Level Access

Custom Database Configuration

Additional Libraries

Custom Packages

Application Dependencies

Specific File System Configuration
```

but I still want to retain some RDS management capabilities.

---

# 🏚️ Legacy Application Example

Suppose I have:

```text
Legacy Business Application
```

that requires:

```text
Specific SQL Server Configuration

+

Custom Windows Dependency

+

Additional Library
```

Standard RDS may not provide the required access.

Using EC2 would provide the access, but I would need to manage the entire environment.

RDS Custom provides another option:

```text
Legacy Application
        │
        ▼
Requires Custom DB Environment
        │
        ▼
RDS Custom
        │
        ├── OS Access
        ├── DB Access
        ├── Dependencies
        └── AWS Automation
```

---

# 🧩 Putting Everything Together

```text
Need Relational Database
          │
          ▼
Do I Need OS / Deep
Database Customization?
          │
      ┌───┴───┐
      │       │
     No      Yes
      │       │
      ▼       ▼
Standard    Need More
  RDS       Control
              │
        ┌─────┴─────┐
        │           │
        ▼           ▼
      EC2       RDS Custom
        │           │
        ▼           ▼
   Self-Manage   Customization
   Environment   + Automation
```

---

# ⚠️ Common Mistakes

## Mistake 1: Thinking Standard RDS Provides OS Access

It does not in the standard RDS model described in this lesson.

```text
Standard RDS
     │
     ▼
No Underlying
OS Access
```

---

## Mistake 2: Thinking RDS Custom Is the Same as EC2

RDS Custom provides deeper administrative access, but it still includes RDS Custom automation and monitoring capabilities.

```text
EC2
=
Self-Managed Database


RDS Custom
=
Customizable Database
+
RDS Automation
```

---

## Mistake 3: Assuming RDS Custom Supports Every RDS Engine

The lesson describes RDS Custom support for:

```text
Oracle

Microsoft SQL Server
```

not MySQL, PostgreSQL, or Aurora.

---

## Mistake 4: Making Custom Changes Without Considering Automation

The lesson's workflow is:

```text
Pause Automation
      │
      ▼
Make Changes
      │
      ▼
Resume Automation
```

---

## Mistake 5: Thinking RDS Custom Removes All Administrative Responsibility

RDS Custom gives me additional access and control.

With that control comes responsibility for the customizations I make to the database and operating system environment.

---

# ✅ Best Practices

* Use standard RDS when deep OS or database customization is not required.
* Consider RDS Custom when an application requires OS-level or database-level administrative access.
* Understand the operational trade-off between EC2, RDS Custom, and standard RDS.
* Use RDS Custom for supported database engines when customization is required.
* Pause RDS Custom automation before performing the customizations described in this lesson.
* Resume automation after completing the required changes.
* Use the automation and monitoring capabilities to monitor the custom environment.
* Keep customizations aligned with actual application requirements rather than modifying the environment unnecessarily.

---

# ❓ Interview Questions

### Q1. What is Amazon RDS Custom?

Amazon RDS Custom is an RDS option that provides access to the underlying operating system and database environment for workloads that require deeper customization.

---

### Q2. Why would I use RDS Custom instead of standard RDS?

I would consider RDS Custom when I need capabilities such as:

```text
OS Access

Custom Dependencies

Database Administrative Access

Custom Database Configuration

File System Configuration
```

that are not available through the standard RDS model described in the lesson.

---

### Q3. Why not simply run the database on EC2?

EC2 provides extensive control, but I would also be responsible for much more of the database and operating system administration.

RDS Custom provides a middle ground.

---

### Q4. Which database engines does the lesson identify as supported by RDS Custom?

```text
Oracle

Microsoft SQL Server
```

---

### Q5. Does the lesson describe RDS Custom support for MySQL?

No.

---

### Q6. Does the lesson describe RDS Custom support for PostgreSQL?

No.

---

### Q7. Does the lesson describe RDS Custom support for Aurora?

No.

---

### Q8. Can I access the operating system of an RDS Custom instance?

Yes.

OS-level access is one of the primary capabilities introduced in the lesson.

---

### Q9. Can I customize the database environment?

Yes.

RDS Custom provides additional administrative access to the database environment.

---

### Q10. How can administrators connect to the RDS Custom instance?

The lesson introduces:

```text
SSH

RDP

AWS Systems Manager
Session Manager
```

depending on the environment.

---

### Q11. What is RDS Custom automation and monitoring?

It is the automation capability that monitors the RDS Custom environment and performs tasks such as metrics collection, notifications, monitoring, and instance recovery.

---

### Q12. What happens if an RDS Custom instance becomes impaired or unreachable?

The lesson describes the automation system performing recovery actions such as:

```text
Reboot

or

Instance Replacement
```

---

### Q13. What should I do before installing custom patches or packages?

The workflow described in the lesson is to:

```text
Pause Automation
```

before making the customization.

---

### Q14. What should I do after completing the customization?

```text
Resume Automation
```

so that monitoring and recovery can continue.

---

### Q15. What is the easiest way to remember RDS Custom?

```text
EC2
    │
    ▼
Control
+
Management


RDS Custom
    │
    ▼
Customization
+
Automation


Standard RDS
    │
    ▼
Managed
+
Limited OS Access
```

---

# 💡 Key Takeaways

* Standard RDS provides a managed database environment.
* Standard RDS does not provide access to the underlying operating system in the model described by this lesson.
* Running a database on EC2 provides extensive control but creates more operational responsibility.
* RDS Custom provides a middle ground between EC2 and standard RDS.
* RDS Custom allows operating-system-level access.
* RDS Custom provides database administrative access for deeper customization.
* Administrators can install dependencies, libraries, patches, and packages as required.
* RDS Custom provides access needed for specific file system configurations.
* The lesson describes RDS Custom support for Oracle and Microsoft SQL Server.
* RDS Custom includes automation and monitoring capabilities.
* Automation can collect metrics and send notifications.
* Automation can perform instance recovery when problems occur.
* The lesson describes rebooting or replacing an impaired instance as examples of recovery actions.
* Administrators should pause automation before performing the customizations described in the lesson.
* Automation can be resumed manually or after the pause period ends.
* RDS Custom is particularly useful when legacy or specialized applications require database or operating system customization.

The simplest mental model is:

```text
SELF-MANAGED                                      MANAGED

EC2                 RDS CUSTOM               STANDARD RDS
 │                      │                         │
 ▼                      ▼                         ▼
Full Control       Custom Control            Managed Service
 │                      │                         │
 ▼                      ▼                         ▼
More Work          AWS Automation            Less Admin Work
```

And the RDS Custom workflow is:

```text
RDS Custom
    │
    ▼
Need Customization?
    │
    ▼
Pause Automation
    │
    ▼
Modify OS / Database
    │
    ▼
Complete Changes
    │
    ▼
Resume Automation
    │
    ▼
Monitoring + Recovery
```

---

# 📚 Related Topics

* Amazon RDS
* Amazon RDS Custom
* Amazon EC2
* Microsoft SQL Server
* Oracle Database
* AWS Shared Responsibility Model
* Database Administration
* Operating System Patching
* Database Patching
* AWS Systems Manager
* Session Manager
* RDS Monitoring
* RDS Multi-AZ
* RDS Read Replicas
