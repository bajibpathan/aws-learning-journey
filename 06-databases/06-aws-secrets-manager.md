# 🔐 AWS Secrets Manager

> AWS Secrets Manager is a service for securely storing, managing, retrieving, and rotating sensitive information such as database credentials, API keys, usernames, passwords, and other secrets.

---

# 📖 Overview

Applications frequently need sensitive information to communicate with other systems.

Examples include:

```text
Database Usernames

Database Passwords

API Keys

Tokens

Application Credentials
```

For example, an application connecting to an Amazon RDS database may require:

```text
RDS Endpoint

Database Name

Username

Password
```

One approach would be to place this information directly inside the application code.

```text
Application Code
      │
      ├── DB Endpoint
      ├── Username
      ├── Password
      └── Database Name
```

This creates a serious security concern.

Instead, AWS provides:

```text
AWS Secrets Manager
```

which allows applications to retrieve sensitive information dynamically when required.

---

# 🚨 The Problem: Hard-Coded Credentials

Suppose an application running on EC2 needs to connect to an RDS database.

```text
EC2 Application
      │
      ▼
  RDS Database
```

The application needs credentials to authenticate with the database.

A poor design would be:

```text
Application Source Code

DB_HOST     = "database-endpoint"
DB_USER     = "admin"
DB_PASSWORD = "mypassword"
DB_NAME     = "applicationdb"
```

The credentials now exist directly inside:

```text
Source Code
```

This becomes particularly dangerous if the source code is committed to:

```text
Git Repository
```

and especially if that repository becomes publicly accessible.

---

# ❌ Why Hard-Coding Secrets Is Dangerous

Consider:

```text
Developer
    │
    ▼
Source Code
    │
    ├── Application Logic
    └── Database Password ❌
            │
            ▼
       Git Repository
```

Anyone who gains access to the code may also gain access to the database credentials.

Instead:

```text
Application Code
      │
      ▼
Retrieve Secret
      │
      ▼
AWS Secrets Manager
```

The secret remains separate from the application source code.

---

# 🛡️ AWS Secrets Manager

AWS Secrets Manager provides a centralized location for storing secret information.

Examples include:

```text
AWS Secrets Manager
│
├── Database Credentials
├── Usernames
├── Passwords
├── API Keys
├── Tokens
└── Other Secrets
```

Applications can retrieve these secrets when they need them.

---

# 🏗️ Basic Architecture

Suppose my application architecture contains:

```text
VPC
│
├── Public Subnet
│
├── Private Subnet
│    │
│    └── EC2 Application
│
└── Data Subnet
     │
     └── RDS Database
```

Without Secrets Manager:

```text
EC2 Application
      │
      │ Hard-Coded Credentials
      ▼
RDS Database
```

With Secrets Manager:

```text
                 AWS Secrets Manager
                         │
                         │ Retrieve
                         ▼
                  EC2 Application
                         │
                         │ Authenticate
                         ▼
                    RDS Database
```

The database credentials are no longer stored directly inside the application code.

---

# 🗄️ Secrets Manager and Amazon RDS

AWS Secrets Manager integrates with Amazon RDS.

Database credentials can be stored as:

```text
Secret
```

inside Secrets Manager.

The lesson describes two approaches:

```text
RDS Configuration
       │
       ▼
Store Credentials
in Secrets Manager
```

or:

```text
Secrets Manager
       │
       ▼
Create Secret
       │
       ▼
Associate with
RDS Database
```

This makes Secrets Manager particularly useful for database credentials.

---

# 🔒 Secret Encryption

Secrets stored in Secrets Manager are encrypted using:

```text
AWS KMS
```

Conceptually:

```text
Database Credentials
        │
        ▼
AWS Secrets Manager
        │
        ▼
     Encrypted
        │
        ▼
      KMS Key
```

The lesson describes using either:

```text
AWS Managed KMS Key
```

or a customer-managed key.

The important idea is:

```text
Secret
  │
  ▼
Encrypted
  │
  ▼
AWS KMS
```

---

# 🧠 Secure Application Architecture

Now the architecture becomes:

```text
                  AWS Secrets Manager
                         │
                         │
                         ▼
                    EC2 Instance
                         │
                         ▼
                    Application
                         │
                         │ Credentials
                         ▼
                    RDS Database
```

The application dynamically obtains credentials from Secrets Manager before connecting to RDS.

---

# 🔑 How Does EC2 Access the Secret?

The EC2 instance needs permission to retrieve the secret.

AWS uses:

```text
IAM
```

for this.

Conceptually:

```text
EC2 Instance
     │
     ▼
IAM Role
     │
     ▼
Permission to
Retrieve Secret
     │
     ▼
Secrets Manager
```

The IAM role needs permission to retrieve the required secret value.

---

# 🪪 IAM Role and Instance Profile

The lesson describes:

```text
IAM Role
   │
   ├── Permissions Policy
   │
   └── Trust Policy
```

The trust policy allows:

```text
EC2 Service
```

to assume the role.

The role is then associated with the EC2 instance through:

```text
Instance Profile
```

Conceptually:

```text
EC2
 │
 ▼
Instance Profile
 │
 ▼
IAM Role
 │
 ▼
Get Secret Value
 │
 ▼
Secrets Manager
```

---

# 🔐 Least-Privilege Access

The application should only receive the permissions it needs.

Conceptually:

```text
Application EC2
      │
      ▼
IAM Role
      │
      ▼
Permission
      │
      ▼
Retrieve Required Secret
```

The goal is not to provide unnecessary access to every secret.

The application needs permission to retrieve the secret required for its database connection.

---

# 💻 Retrieving Secrets from Application Code

The application still needs a mechanism to communicate with AWS Secrets Manager.

The lesson introduces:

```text
AWS SDK
```

SDK stands for:

```text
Software Development Kit
```

AWS provides SDKs for programming languages such as:

```text
Python

Java

Other Supported Languages
```

The application uses the SDK to request the secret.

---

# 🔄 Application Credential Retrieval Flow

The basic flow is:

```text
Application
    │
    ▼
AWS SDK
    │
    ▼
Secrets Manager
    │
    ▼
Retrieve Secret
    │
    ▼
Application
    │
    ▼
Connect to RDS
```

The credentials are retrieved dynamically rather than embedded directly in the source code.

---

# 🧩 Complete Authentication Flow

Putting everything together:

```text
                AWS Secrets Manager
                        ▲
                        │
                  Get Secret Value
                        │
                     IAM Role
                        ▲
                        │
                  Instance Profile
                        │
                        │
                 EC2 Application
                        │
                        │ Credentials
                        ▼
                   RDS Database
```

Step by step:

```text
1. Application needs database access

2. Application uses AWS SDK

3. EC2 uses its IAM role

4. IAM permissions allow secret retrieval

5. Application retrieves credentials

6. Application uses credentials

7. Application connects to RDS
```

---

# 🌐 Networking Consideration

The lesson points out an important networking consideration.

Suppose the EC2 application runs in:

```text
Private Subnet
```

while Secrets Manager is accessed as an AWS service endpoint.

The EC2 instance still needs a network path to reach the service.

One possible architecture introduced in the lesson is:

```text
Private EC2
    │
    ▼
NAT Gateway
    │
    ▼
AWS Secrets Manager
```

So:

```text
Private Subnet
      │
      ▼
NAT Gateway
      │
      ▼
Secrets Manager
```

can provide a path for the application to communicate with the service.

---

# 🔗 Using a VPC Endpoint

The lesson also introduces another option:

```text
Interface VPC Endpoint
```

Instead of using:

```text
Private EC2
    │
    ▼
NAT Gateway
    │
    ▼
Secrets Manager
```

I can consider:

```text
Private EC2
    │
    ▼
Interface VPC Endpoint
    │
    ▼
Secrets Manager
```

This provides another way for resources in the VPC to communicate with Secrets Manager.

---

# 🆚 NAT Gateway vs Interface Endpoint

Conceptually:

```text
Option 1

Private EC2
    │
    ▼
NAT Gateway
    │
    ▼
Secrets Manager
```

versus:

```text
Option 2

Private EC2
    │
    ▼
VPC Interface Endpoint
    │
    ▼
Secrets Manager
```

The lesson introduces the interface endpoint as an alternative when I do not want to use the NAT Gateway path for accessing Secrets Manager.

---

# 🔄 Secret Rotation

Another major capability of Secrets Manager is:

```text
Secret Rotation
```

Database credentials do not necessarily need to remain unchanged forever.

For example:

```text
Current Password
      │
      ▼
Rotate
      │
      ▼
New Password
```

Secrets Manager can be configured to rotate credentials at scheduled intervals.

---

# 🤖 Rotation Using AWS Lambda

The lesson introduces Lambda as part of the secret rotation process.

Conceptually:

```text
Secrets Manager
      │
      ▼
Lambda Function
      │
      ▼
RDS Database
```

The Lambda function performs the required credential rotation.

---

# 🏗️ Rotation Architecture

A simplified architecture looks like:

```text
                Secrets Manager
                      │
                      ▼
               Lambda Function
                      │
                      ▼
                   ENI
                      │
                      ▼
                 RDS Database
```

The Lambda function needs to communicate with the database to update its credentials.

---

# 🪪 Lambda Execution Role

Just like EC2, Lambda requires permissions.

The Lambda function therefore needs:

```text
Execution Role
```

Conceptually:

```text
Lambda Function
      │
      ▼
Execution Role
      │
      ▼
Required Permissions
```

The execution role provides the permissions required by the function.

---

# 🌐 Lambda and the Database Subnet

The lesson describes Lambda connecting into the VPC through an:

```text
Elastic Network Interface
```

or:

```text
ENI
```

Conceptually:

```text
Lambda
   │
   ▼
ENI
   │
   ▼
Database Subnet
   │
   ▼
RDS
```

This allows the rotation function to communicate with the database.

---

# 🔐 Security Groups for Rotation

Suppose the database is MySQL.

The default database port is:

```text
TCP 3306
```

The RDS security group needs to permit the required traffic from the security group associated with the Lambda function.

Conceptually:

```text
Lambda Function
      │
      ▼
Lambda Security Group
      │
      │ TCP 3306
      ▼
RDS Security Group
      │
      ▼
RDS MySQL
```

The RDS inbound rule therefore needs to recognize the Lambda security group as an allowed source for the required database connection.

---

# 🔁 Secret Rotation Process

The rotation process introduced in the lesson can be visualized as:

```text
Secrets Manager
      │
      ▼
Rotation Schedule
      │
      ▼
Lambda Function
      │
      ▼
Change Database Credentials
      │
      ▼
RDS
      │
      ▼
Update Secret
      │
      ▼
Secrets Manager
```

Now Secrets Manager contains the updated credentials.

---

# ⏰ Rotation Schedule

The lesson describes configuring rotation at scheduled intervals.

Examples include:

```text
48 Hours

2 Days

5 Days

7 Days
```

The actual schedule should follow:

```text
Organizational Policy

Security Requirements

Governance Requirements
```

---

# 🔄 Application Behavior After Rotation

This introduces another important application design consideration.

Suppose the application retrieves the password once:

```text
Application Starts
      │
      ▼
Retrieve Password
      │
      ▼
Store / Continue Using It
```

Later:

```text
Secrets Manager
      │
      ▼
Password Rotated
```

Now the application may still be trying to use:

```text
Old Password ❌
```

and database authentication could fail.

---

# 🧠 Applications Must Account for Rotation

Applications need to be designed with credential rotation in mind.

Conceptually:

```text
Application
      │
      ▼
Retrieve Current Secret
      │
      ▼
Use Credentials
      │
      ▼
Secret Changes
      │
      ▼
Retrieve Updated Secret
      │
      ▼
Continue Database Access
```

The lesson emphasizes that developers need to design applications so that credential changes do not break database connectivity.

---

# 🧑‍💻 Example Application Logic

The lesson provides a high-level example of application code that:

```text
Secret Name
    │
    ▼
Retrieve Secret
    │
    ▼
DB Credentials
    │
    ├── Username
    ├── Password
    ├── Host
    └── Database
          │
          ▼
   Database Connection
```

The exact implementation depends on the programming language and AWS SDK being used.

The important concept is:

```text
Do Not Hard-Code
       │
       ▼
Retrieve Dynamically
```

---

# 🗃️ Types of Secrets

Secrets Manager can store more than just RDS passwords.

The lesson introduces integrations and use cases involving:

```text
Amazon RDS

Amazon DocumentDB

Amazon Redshift
```

It can also store:

```text
Other Database Credentials

API Keys

Tokens

Other Secret Values
```

Conceptually:

```text
AWS Secrets Manager
│
├── RDS Credentials
├── DocumentDB Credentials
├── Redshift Credentials
├── Other DB Credentials
├── API Keys
└── Tokens
```

---

# 🏗️ Complete Architecture

Putting the major concepts from the lesson together:

```text
                           AWS
                            │
          ┌─────────────────┴──────────────────┐
          │                                    │
          ▼                                    ▼
        VPC                           AWS Secrets Manager
          │                                    ▲
          │                                    │
          │                              Retrieve Secret
          │                                    │
          ├── Private Subnet                    │
          │      │                              │
          │      ▼                              │
          │   EC2 Application ──────────────────┘
          │      │
          │      │ DB Credentials
          │      ▼
          │   RDS Database
          │
          └── Lambda ENI
                 │
                 │ Rotate Credentials
                 ▼
             RDS Database
```

---

# 🔁 Complete Secret Lifecycle

The overall secret lifecycle can be thought of as:

```text
Create Secret
     │
     ▼
Encrypt with KMS
     │
     ▼
Store in Secrets Manager
     │
     ▼
Grant IAM Permission
     │
     ▼
Application Retrieves Secret
     │
     ▼
Application Connects to RDS
     │
     ▼
Rotate Secret
     │
     ▼
Update Database Credentials
     │
     ▼
Store New Secret
     │
     ▼
Application Retrieves
Updated Credentials
```

---

# 🆚 Hard-Coded Credentials vs Secrets Manager

| Area                       | Hard-Coded Credentials | Secrets Manager                        |
| -------------------------- | ---------------------- | -------------------------------------- |
| Credentials in Source Code | Yes                    | No                                     |
| Central Secret Storage     | No                     | Yes                                    |
| IAM-Based Access           | No                     | Yes                                    |
| Encryption                 | Depends on application | KMS-based encryption                   |
| Dynamic Retrieval          | No                     | Yes                                    |
| Credential Rotation        | Difficult              | Supported                              |
| RDS Integration            | Manual                 | Native integration described in lesson |

The better design introduced in the lesson is:

```text
Application
     │
     ▼
IAM Role
     │
     ▼
Secrets Manager
     │
     ▼
Retrieve Credentials
     │
     ▼
RDS
```

rather than:

```text
Application
     │
     ▼
Hard-Coded Password
     │
     ▼
RDS
```

---

# ⚠️ Common Mistakes

## Mistake 1: Hard-Coding Database Passwords

Avoid:

```text
password = "MyDatabasePassword"
```

inside application source code.

Instead:

```text
Application
     │
     ▼
Secrets Manager
```

---

## Mistake 2: Committing Credentials to Git

Never intentionally store database passwords, API keys, or other sensitive secrets inside code that may be committed to a repository.

```text
Source Code
    │
    ▼
Git Repository
    │
    ▼
Secret Exposure ❌
```

---

## Mistake 3: Giving EC2 No IAM Permission

Even if the secret exists:

```text
EC2
 │
 ▼
Secrets Manager
```

the instance still requires authorization.

Use:

```text
EC2
 │
 ▼
IAM Role
 │
 ▼
Permission
 │
 ▼
Secret
```

---

## Mistake 4: Forgetting Network Connectivity

An EC2 instance in a private subnet still needs a network path to access Secrets Manager.

The lesson introduces:

```text
NAT Gateway
```

or:

```text
Interface VPC Endpoint
```

as possible approaches.

---

## Mistake 5: Rotating Credentials Without Updating Application Behavior

If:

```text
Password Rotates
```

but the application continues using:

```text
Old Password
```

authentication may fail.

The application must be designed to account for credential rotation.

---

## Mistake 6: Forgetting Lambda Permissions

A rotation Lambda function requires:

```text
Execution Role
```

with the necessary permissions.

---

## Mistake 7: Forgetting Database Security Group Rules

If Lambda needs to connect to MySQL:

```text
Lambda SG
    │
    │ TCP 3306
    ▼
RDS SG
```

the RDS security group must allow the required inbound connection.

---

# ✅ Best Practices

* Do not hard-code database credentials in application source code.
* Do not commit passwords, API keys, or tokens to source repositories.
* Store sensitive credentials in AWS Secrets Manager.
* Use IAM roles to authorize applications to retrieve secrets.
* Use AWS SDKs to retrieve secrets dynamically.
* Keep permissions limited to the secrets an application requires.
* Use KMS encryption for secrets as provided by Secrets Manager.
* Consider credential rotation based on organizational requirements.
* Design applications to handle credential rotation.
* Ensure private resources have a valid network path to Secrets Manager.
* Consider an interface VPC endpoint when appropriate.
* Ensure rotation Lambda functions have the required IAM permissions.
* Configure security groups so the rotation function can communicate with the database.

---

# ❓ Interview Questions

### Q1. What is AWS Secrets Manager?

AWS Secrets Manager is a service for securely storing, managing, retrieving, and rotating sensitive information such as usernames, passwords, API keys, tokens, and database credentials.

---

### Q2. Why should database credentials not be hard-coded?

Hard-coded credentials become part of the application source code and could be exposed through source repositories or unauthorized access to the code.

---

### Q3. How can an EC2 application retrieve a secret?

Conceptually:

```text
EC2 Application
      │
      ▼
AWS SDK
      │
      ▼
IAM Role
      │
      ▼
Secrets Manager
      │
      ▼
Secret
```

---

### Q4. What gives an EC2 instance permission to retrieve a secret?

An:

```text
IAM Role
```

associated with the instance through an instance profile.

---

### Q5. What is the purpose of the IAM trust policy?

The lesson describes the trust policy as allowing the EC2 service to use the role associated with the instance.

---

### Q6. How does application code communicate with Secrets Manager?

Using:

```text
AWS SDKs
```

available for supported programming languages.

---

### Q7. How are secrets encrypted?

The lesson describes Secrets Manager secrets as being encrypted using:

```text
AWS KMS
```

---

### Q8. Does Secrets Manager integrate with Amazon RDS?

Yes.

The lesson describes native integration between Secrets Manager and Amazon RDS.

---

### Q9. How can a private EC2 instance access Secrets Manager?

The lesson introduces two approaches:

```text
NAT Gateway

or

Interface VPC Endpoint
```

---

### Q10. What is secret rotation?

Secret rotation is the process of periodically replacing existing credentials with new credentials.

---

### Q11. What AWS service is used for the database secret rotation process described in the lesson?

```text
AWS Lambda
```

---

### Q12. What permissions does the rotation Lambda require?

The Lambda function requires an execution role with the permissions needed to perform its rotation tasks.

---

### Q13. Why might the Lambda function need an ENI?

The lesson describes the Lambda function using an ENI to communicate with the RDS database inside the VPC.

---

### Q14. What security group configuration might be required for MySQL rotation?

The RDS security group needs to permit:

```text
TCP 3306
```

from the security group associated with the Lambda function.

---

### Q15. What happens after the database credentials are rotated?

The new credentials are stored in Secrets Manager so that applications can retrieve the updated values.

---

### Q16. What must developers consider when using secret rotation?

Applications need to be designed so they can obtain updated credentials when secrets change rather than relying permanently on credentials retrieved only once.

---

### Q17. What kinds of secrets can Secrets Manager store?

Examples from the lesson include:

```text
Database Credentials

Usernames

Passwords

API Keys

Tokens
```

---

### Q18. Which database services are specifically mentioned as integrations?

The lesson mentions:

```text
Amazon RDS

Amazon DocumentDB

Amazon Redshift
```

---

### Q19. What should I think of when a requirement says credentials must be stored securely and dynamically retrieved?

```text
AWS Secrets Manager
```

---

### Q20. What AWS service should I consider when credentials need regular automatic rotation?

```text
AWS Secrets Manager
```

with the rotation mechanism described in the lesson.

---

# 💡 Key Takeaways

* AWS Secrets Manager securely stores sensitive information.
* Secrets can include usernames, passwords, database credentials, API keys, and tokens.
* Database credentials should not be hard-coded in application source code.
* Secrets should not be committed to source repositories.
* Secrets Manager integrates with Amazon RDS.
* Secrets are encrypted using AWS KMS.
* EC2 applications need IAM permissions to retrieve secrets.
* IAM roles can be associated with EC2 through instance profiles.
* Applications can use AWS SDKs to retrieve credentials dynamically.
* Private EC2 instances need network connectivity to Secrets Manager.
* The lesson introduces NAT Gateway as one possible network path.
* An interface VPC endpoint can also be used to access Secrets Manager.
* Secrets Manager supports credential rotation.
* The lesson describes Lambda as part of database credential rotation.
* The rotation Lambda requires an execution role.
* Lambda may need VPC connectivity to reach the RDS database.
* Security groups must allow the required database communication.
* Applications need to account for changing credentials when rotation is enabled.
* Secrets Manager integrates with services such as RDS, DocumentDB, and Redshift.
* Read access to secrets should be controlled through IAM.

The simplest mental model is:

```text
BAD DESIGN

Application Code
      │
      ├── Username
      ├── Password
      └── API Key
            │
            ▼
      Secret Exposure ❌
```

versus:

```text
BETTER DESIGN

Application
     │
     ▼
IAM Role
     │
     ▼
AWS SDK
     │
     ▼
Secrets Manager
     │
     ▼
Retrieve Secret
     │
     ▼
RDS Database
```

And with rotation:

```text
Secrets Manager
      │
      ▼
Rotation Schedule
      │
      ▼
Lambda
      │
      ▼
RDS Credentials Updated
      │
      ▼
Secrets Manager Updated
      │
      ▼
Application Retrieves
New Credentials
```

---

# 📚 Related Topics

* AWS Secrets Manager
* Amazon RDS
* AWS Identity and Access Management (IAM)
* IAM Roles
* EC2 Instance Profiles
* AWS SDK
* AWS Key Management Service (KMS)
* AWS Lambda
* Lambda Execution Roles
* Amazon VPC
* NAT Gateway
* Interface VPC Endpoints
* Security Groups
* Amazon DocumentDB
* Amazon Redshift
* Database Credential Rotation
