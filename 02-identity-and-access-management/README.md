# 🔐 AWS Identity & Access Management (IAM)

> A comprehensive guide to AWS Identity and Access Management (IAM), covering authentication, authorization, IAM components, security best practices, and real-world implementation examples.

---

# 📖 Overview

AWS Identity and Access Management (IAM) is a foundational AWS service that enables you to securely manage access to AWS resources.

IAM helps organizations answer two fundamental security questions:

- **Who is requesting access?** *(Authentication)*
- **What are they allowed to do?** *(Authorization)*

Using IAM, administrators can create identities, assign permissions, control resource access, and enforce security best practices across AWS environments.

IAM is one of the first AWS services you should master because almost every AWS service integrates with it.

---

# 🎯 Why IAM Matters

As organizations grow, managing access becomes increasingly complex. Without a centralized identity and access management solution, organizations face risks such as:

- Excessive permissions
- Unauthorized access
- Credential sharing
- Difficulty auditing user activity
- Increased security vulnerabilities

AWS IAM addresses these challenges by enabling organizations to:

- Securely authenticate users and applications
- Control permissions using least privilege
- Manage access across multiple AWS services
- Enable temporary access using IAM Roles
- Audit credentials and permissions
- Support enterprise security and compliance requirements

---

# 🏗️ Authentication vs Authorization

Understanding the difference between **Authentication** and **Authorization** is essential for every Cloud Engineer.

| Authentication | Authorization |
|----------------|---------------|
| Verifies **who** you are | Determines **what** you are allowed to do |
| Identity verification | Permission evaluation |
| Happens first | Happens after authentication |
| Examples: Username, Password, MFA | IAM Policies, Resource Policies |

## Authentication

Authentication is the process of verifying the identity of a user, application, or AWS service before granting access.

Common authentication methods include:

- Username and Password
- Multi-Factor Authentication (MFA)
- Access Keys
- AWS IAM Identity Center
- Temporary Security Credentials (STS)

---

## Authorization

After authentication succeeds, AWS evaluates the permissions assigned to the authenticated identity.

Authorization determines:

- Which AWS services can be accessed
- Which actions are permitted
- Which resources can be accessed

Permissions are primarily controlled using:

- IAM Policies
- Resource-Based Policies
- IAM Roles

---

## Example

A developer signs in to the AWS Management Console using an IAM user account.

1. AWS verifies the username, password, and MFA.
2. Authentication succeeds.
3. IAM evaluates the attached policies.
4. The developer is allowed to access Amazon S3 but denied access to Amazon EC2.

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to do?**

---

# 🧩 IAM Components

AWS IAM consists of several building blocks that work together to provide secure access management.

| Component | Purpose |
|-----------|---------|
| IAM User | Represents an individual person or application |
| IAM Group | Collection of IAM users with common permissions |
| IAM Policy | JSON document defining permissions |
| IAM Role | Temporary identity assumed by trusted entities |
| IAM Identity Center | Centralized multi-account access management |
| Amazon Cognito | Identity service for application users |

---

# 👤 IAM Users

## What is an IAM User?

An IAM User represents an individual identity within an AWS account.

An IAM User typically belongs to:

- Administrator
- Developer
- DevOps Engineer
- Cloud Engineer
- Automation Tool

Each IAM User has unique credentials that can be used to access AWS resources.

Authentication methods include:

- Console Username & Password
- Access Keys
- MFA

---

## Why Use IAM Users?

IAM Users allow organizations to:

- Provide individual identities
- Track user activities
- Assign permissions based on job roles
- Improve auditing and accountability

Each user should have their own identity rather than sharing credentials.

---

## Best Practices

- Create one IAM User per individual.
- Enable MFA.
- Avoid using the Root User.
- Assign permissions through Groups whenever possible.
- Rotate credentials regularly.

---

# 👥 IAM Groups

## What is an IAM Group?

An IAM Group is a collection of IAM Users.

Instead of assigning permissions individually to each user, permissions are assigned once to the group.

Every member of the group inherits the same permissions.

---

## Why Use IAM Groups?

Groups simplify permission management.

Instead of updating permissions for every user individually, administrators update the group once.

This reduces administrative overhead and improves consistency.

---

## Example

Instead of assigning Amazon S3 ReadOnly permissions to 25 developers individually:

```
Developers Group
        │
        ▼
AmazonS3ReadOnlyAccess
        │
        ▼
25 Developers
```

When a new developer joins the team:

- Create IAM User
- Add user to Developers Group

No additional permission changes are required.

---

## Best Practices

- Organize groups by job function.
- Assign permissions to groups rather than users.
- Follow the Principle of Least Privilege.
- Regularly review group memberships.

---

# 📜 IAM Policies

## What is an IAM Policy?

An IAM Policy is a JSON document that defines permissions in AWS.

Policies specify:

- Which actions are allowed or denied
- Which AWS resources are affected
- Under what conditions access is granted

Policies are the foundation of AWS authorization.

---

## Types of IAM Policies

### AWS Managed Policies

Created and maintained by AWS.

Suitable for common permission requirements.

Examples:

- ReadOnlyAccess
- AdministratorAccess
- AmazonS3ReadOnlyAccess

---

### Customer Managed Policies

Created and managed by customers.

Used when custom permissions are required.

Provides better control and reusability.

---

### Inline Policies

Embedded directly within a specific IAM User, Group, or Role.

Typically used for unique permission requirements.

Because Inline Policies cannot be reused, AWS generally recommends using Customer Managed Policies instead.

---

## IAM Policy Structure

Every IAM Policy consists of one or more statements.

Example components:

- **Version** – Policy language version
- **Statement** – One or more permission blocks
- **Sid** – Optional statement identifier
- **Effect** – Allow or Deny
- **Action** – AWS API actions
- **Resource** – AWS resources affected
- **Condition** *(Optional)* – Additional access restrictions

---

## Permission Evaluation

AWS evaluates permissions in the following order:

1. **Implicit Deny** *(default)*
2. Explicit Allow
3. **Explicit Deny always overrides Allow**

This evaluation model is one of the most important IAM concepts for interviews.

---

## Best Practices

- Follow the Principle of Least Privilege.
- Prefer Customer Managed Policies over Inline Policies.
- Reuse policies whenever possible.
- Review policies regularly.
- Remove unnecessary permissions.

---

# 📌 Key Takeaways

- IAM provides centralized authentication and authorization for AWS resources.
- Authentication verifies identity, while authorization determines permissions.
- IAM Users represent individual identities.
- IAM Groups simplify permission management.
- IAM Policies define access using JSON documents.
- Following the Principle of Least Privilege is fundamental to securing AWS environments.

---


# 👑 AWS Root User

## What is the AWS Root User?

The **Root User** is the first identity created when an AWS account is established. It has unrestricted access to every AWS service and resource within the account.

Unlike IAM Users, the Root User cannot have permissions restricted through IAM policies.

Because of its elevated privileges, the Root User should only be used for a limited set of account-level administrative tasks.

---

## When Should You Use the Root User?

AWS recommends using the Root User **only when absolutely necessary**.

Examples include:

- Changing the AWS account settings
- Closing the AWS account
- Restoring IAM administrator access
- Changing support plans
- Managing payment methods
- Configuring certain account-level security settings

For all other day-to-day activities, use an IAM User or IAM Identity Center.

---

## Root User Best Practices

- Enable Multi-Factor Authentication (MFA).
- Do not create access keys for the Root User.
- Never use the Root User for daily administrative tasks.
- Store Root User credentials securely.
- Monitor Root User activity using AWS CloudTrail.

---

## Real-World Example

A Cloud Administrator needs to create an Amazon S3 bucket.

❌ Incorrect Approach

Use the Root User.

✅ Correct Approach

Create an IAM Administrator User with AdministratorAccess permissions and perform daily administration using that account.

---

# 🔑 Accessing AWS

AWS provides multiple methods to access AWS resources.

| Method | Typical Use Case |
|---------|------------------|
| AWS Management Console | Interactive web-based administration |
| AWS CLI | Command-line administration and automation |
| AWS SDKs | Programmatic access from applications |
| AWS APIs | Service integrations |

---

## AWS Management Console

The AWS Management Console provides a web-based interface for managing AWS resources.

Users authenticate using:

- IAM User credentials
- IAM Identity Center (SSO)
- Temporary credentials

---

## AWS Command Line Interface (AWS CLI)

The AWS CLI enables administrators and developers to manage AWS resources directly from the terminal.

Common use cases include:

- Automation
- Infrastructure management
- Scripting
- CI/CD pipelines

Authentication methods include:

- IAM Identity Center
- Temporary credentials
- Access Keys (when required)

---

## AWS SDKs

AWS Software Development Kits (SDKs) allow applications to communicate with AWS services programmatically.

Supported languages include:

- Python (Boto3)
- Java
- JavaScript
- .NET
- Go
- C++
- Ruby

---

## Best Practices

- Prefer IAM Identity Center for workforce authentication.
- Avoid long-term access keys whenever possible.
- Never hard-code credentials.
- Store credentials securely.
- Rotate access keys regularly.
- Enable MFA.

---

# 🎭 IAM Roles

## What is an IAM Role?

An IAM Role is an AWS identity that contains a set of permissions but **does not have permanent credentials**.

Instead, trusted entities assume the role and receive **temporary security credentials** from AWS Security Token Service (STS).

---

## Who Can Assume a Role?

IAM Roles can be assumed by:

- IAM Users
- IAM Roles
- AWS Services
- AWS Accounts
- Federated Identities

Examples of AWS Services:

- Amazon EC2
- AWS Lambda
- Amazon ECS
- AWS Glue

---

## Why Use IAM Roles?

IAM Roles provide several benefits:

- Temporary credentials
- Cross-account access
- Improved security
- No credential management
- Least Privilege access

IAM Roles eliminate the need to store AWS access keys on servers or applications.

---

## How IAM Roles Work

```
Trusted Entity
        │
        ▼
 Assume IAM Role
        │
        ▼
 AWS STS
        │
 Temporary Credentials
        │
        ▼
AWS Resources
```

---

## IAM Role Components

Every IAM Role consists of:

- Trust Policy
- IAM Permissions Policy

### Trust Policy

Defines **who is allowed to assume the role**.

### IAM Permissions Policy

Defines **what actions the role can perform** after being assumed.

---

## Real-World Example

An Amazon EC2 instance needs to upload application logs to Amazon S3.

Instead of storing AWS Access Keys on the server:

- Create an IAM Role.
- Grant Amazon S3 permissions.
- Attach the Role to the EC2 instance.
- AWS automatically provides temporary credentials.

Benefits:

- No hard-coded credentials
- Improved security
- Easier credential management

---

## Best Practices

- Grant only required permissions.
- Prefer IAM Roles over long-term access keys.
- Restrict Trust Policies.
- Review and remove unused Roles.
- Use temporary credentials whenever possible.

---

# 🌐 Cross-Account Access

Large organizations often manage multiple AWS accounts.

Instead of creating duplicate IAM Users in every account, AWS recommends using **Cross-Account IAM Roles**.

---

## Architecture

```
+--------------------------+
|      Account B           |
|--------------------------|
| IAM User                 |
+-----------+--------------+
            |
            | AssumeRole
            |
            ▼
+--------------------------+
|      Account A           |
|--------------------------|
| IAM Role                 |
| IAM Policy               |
| AWS Resources            |
+--------------------------+
```

---

## How It Works

### Account A (Target Account)

- Create an IAM Role.
- Attach the required IAM Policy.
- Configure the Trust Policy to trust Account B.

---

### Account B (Source Account)

- Create an IAM Policy allowing:

```
sts:AssumeRole
```

- Attach the policy to the IAM User.

---

### Validation

The IAM User signs in to Account B and switches to the IAM Role in Account A.

AWS STS issues temporary credentials.

The user can access only the resources allowed by the Role.

---

## Benefits

- No duplicate IAM Users
- Temporary credentials
- Better security
- Easier administration
- Supports multi-account architectures

---

## Best Practices

- Trust only required AWS accounts.
- Apply the Principle of Least Privilege.
- Use IAM Roles instead of sharing credentials.
- Monitor AssumeRole events using CloudTrail.

---

# 📌 Key Takeaways

- The Root User should only be used for account-level administrative tasks.
- AWS provides multiple secure methods to access resources.
- IAM Roles provide temporary credentials and improve security.
- Cross-Account Roles eliminate the need for duplicate IAM Users.
- AWS STS enables secure temporary access across AWS services and accounts.

---

# 🏢 AWS IAM Identity Center

## What is AWS IAM Identity Center?

AWS IAM Identity Center (formerly AWS Single Sign-On) is a centralized identity management service that enables users to securely access multiple AWS accounts and business applications using a single set of credentials.

Instead of managing IAM Users separately in each AWS account, administrators can centrally manage users, groups, and permissions from one location.

IAM Identity Center integrates with **AWS Organizations**, making it the recommended solution for enterprise multi-account environments.

---

## Why Use IAM Identity Center?

As organizations grow, managing individual IAM Users across multiple AWS accounts becomes difficult.

IAM Identity Center solves this problem by providing:

- Single Sign-On (SSO)
- Centralized user management
- Multi-account access
- Permission management through Permission Sets
- Integration with external Identity Providers (IdPs)

---

## Key Components

### Identity Source

IAM Identity Center supports multiple identity sources:

- Built-in Identity Center Directory
- Microsoft Active Directory
- External Identity Providers (Azure AD, Okta, Ping Identity, Google Workspace)

---

### Users

Represents employees or administrators who require AWS access.

---

### Groups

Users are organized into groups based on job functions.

Examples:

- Cloud Engineers
- Developers
- Security Team
- DevOps Engineers

---

### Permission Sets

Permission Sets define what users can do after signing in.

Permission Sets are similar to IAM Policies but are centrally managed and automatically provisioned across AWS accounts.

---

## How IAM Identity Center Works

```
User
   │
   ▼
IAM Identity Center
   │
Permission Set
   │
AWS Account
   │
AWS Resources
```

---

## Real-World Example

A company has:

- Development Account
- Testing Account
- Production Account

Instead of creating IAM Users in each account:

- Users sign in once using IAM Identity Center.
- Permission Sets determine their access.
- Users can switch between accounts without managing multiple credentials.

---

## Best Practices

- Use IAM Identity Center for workforce identities.
- Organize users into groups.
- Assign Permission Sets instead of individual permissions.
- Integrate with corporate Identity Providers whenever possible.
- Apply the Principle of Least Privilege.

---

# 📱 Amazon Cognito

## What is Amazon Cognito?

Amazon Cognito is an AWS service that provides authentication and authorization for **application users**.

Unlike IAM, which manages administrators and AWS users, Cognito manages end users of web and mobile applications.

Examples include:

- Banking Applications
- E-commerce Websites
- Mobile Applications
- SaaS Platforms

---

## Why Use Amazon Cognito?

Application users require secure sign-up, sign-in, and access to backend AWS resources.

Amazon Cognito provides:

- User authentication
- User management
- Social login
- Multi-Factor Authentication (MFA)
- Temporary AWS credentials

---

## Cognito Components

### User Pool

User Pools manage authentication.

Responsibilities include:

- User Registration
- Sign-In
- Password Management
- MFA
- User Profiles

---

### Identity Pool

Identity Pools provide temporary AWS credentials after authentication.

Identity Pools allow authenticated users to access AWS services such as:

- Amazon S3
- DynamoDB
- API Gateway

---

## How Cognito Works

```
Application User
        │
        ▼
User Pool
(Authentication)
        │
        ▼
Identity Pool
(AWS Credentials)
        │
        ▼
AWS Resources
```

---

## Real-World Example

A customer logs into a banking application.

- User Pool authenticates the customer.
- Identity Pool provides temporary AWS credentials.
- The customer uploads KYC documents directly to Amazon S3.

No IAM User is required.

---

## Best Practices

- Use User Pools for authentication.
- Use Identity Pools only when AWS resource access is required.
- Enable MFA.
- Integrate with social identity providers where appropriate.
- Avoid exposing AWS credentials to applications.

---

# 📦 Resource-Based Policies

## What are Resource-Based Policies?

A Resource-Based Policy is a JSON policy attached directly to an AWS resource.

Instead of granting permissions to an identity, the policy specifies **who can access the resource**.

Common AWS services supporting Resource-Based Policies include:

- Amazon S3
- Amazon SQS
- Amazon SNS
- AWS Lambda
- Amazon ECR
- AWS Secrets Manager

---

## Why Use Resource-Based Policies?

Resource-Based Policies are useful when:

- Granting cross-account access
- Sharing AWS resources
- Allowing AWS services to access resources
- Controlling access directly at the resource level

---

## Real-World Example

A company wants another AWS account to upload files into an Amazon S3 bucket.

Instead of creating IAM Users:

- Attach a Bucket Policy.
- Specify the external AWS account as the Principal.
- Grant only the required S3 permissions.

---

## Best Practices

- Apply Least Privilege.
- Grant access only to trusted principals.
- Avoid wildcard (`*`) principals whenever possible.
- Regularly review Resource Policies.

---

# 🛡️ AWS Security Tools

AWS provides tools to continuously review IAM security.

---

## Credential Report

Credential Reports provide information about IAM User credentials.

The report includes:

- Password status
- Password age
- MFA status
- Access Key age
- Last sign-in
- Last password change

Credential Reports help identify stale credentials and improve account security.

---

## IAM Access Analyzer

IAM Access Analyzer helps identify:

- External resource access
- Unused IAM permissions
- Cross-account resource sharing

This improves security by identifying unintended access.

---

## Best Practices

- Review Credential Reports regularly.
- Remove unused users.
- Rotate Access Keys.
- Enable MFA.
- Review IAM Access Analyzer findings.
- Follow the Principle of Least Privilege.

---

# 📊 Comparison Tables

## IAM User vs IAM Role

| IAM User | IAM Role |
|----------|----------|
| Permanent identity | Temporary identity |
| Long-term credentials | Temporary credentials |
| Used by people | Used by services, users, and applications |
| Credentials are managed manually | Credentials are provided automatically by AWS STS |

---

## IAM Identity Center vs IAM User

| IAM Identity Center | IAM User |
|---------------------|----------|
| Centralized access | Individual account access |
| Single Sign-On | Separate credentials |
| Multi-account management | Single AWS account |
| Recommended for enterprises | Suitable for smaller environments |

---

## User Pool vs Identity Pool

| User Pool | Identity Pool |
|------------|---------------|
| User authentication | AWS resource access |
| Sign-up / Sign-in | Temporary AWS credentials |
| User management | Authorization to AWS services |

---

## IAM Policy vs Resource-Based Policy

| IAM Policy | Resource-Based Policy |
|------------|----------------------|
| Attached to an identity | Attached to a resource |
| Defines what an identity can do | Defines who can access the resource |
| Applies to Users, Groups, and Roles | Applies to supported AWS resources |

---

# 📌 Key Takeaways

- IAM Identity Center simplifies secure access across multiple AWS accounts.
- Amazon Cognito manages authentication for application users.
- Resource-Based Policies define who can access a specific AWS resource.
- Credential Reports and IAM Access Analyzer help strengthen IAM security.
- Choosing the right identity service depends on the use case:
  - **IAM Users** → Individual AWS administrators
  - **IAM Identity Center** → Enterprise workforce access
  - **Amazon Cognito** → Application users

---


# 🏭 Production Best Practices

Following AWS security best practices helps build secure, scalable, and maintainable cloud environments.

## 1. Never Use the Root User for Daily Activities

The Root User has unrestricted access to every AWS service and resource.

Use it only for account-level administrative tasks such as:

- Closing the AWS account
- Managing payment methods
- Restoring IAM administrator access
- Changing support plans

For all other activities, use IAM Users or IAM Identity Center.

---

## 2. Enable Multi-Factor Authentication (MFA)

Always enable MFA for:

- Root User
- Administrator accounts
- Privileged IAM Users

MFA provides an additional layer of security beyond passwords.

---

## 3. Follow the Principle of Least Privilege

Grant only the permissions required to perform a specific task.

Avoid assigning broad permissions such as:

```
AdministratorAccess
```

unless absolutely necessary.

---

## 4. Prefer IAM Roles Over Access Keys

Instead of storing AWS Access Keys inside:

- EC2 Instances
- Lambda Functions
- ECS Tasks
- Applications

Use IAM Roles so AWS Security Token Service (STS) provides temporary credentials automatically.

---

## 5. Rotate Credentials Regularly

Regularly review and rotate:

- Passwords
- Access Keys
- Temporary Credentials

Remove credentials that are no longer used.

---

## 6. Use Groups Instead of Individual Permissions

Rather than assigning permissions to every user:

```
Developer Group
        │
        ▼
Developers
```

This simplifies administration and reduces errors.

---

## 7. Review IAM Permissions Regularly

Perform periodic security reviews by:

- Reviewing IAM Policies
- Removing unused users
- Removing unused roles
- Removing unnecessary permissions

---

## 8. Review IAM Access Analyzer Findings

Regularly review:

- Cross-account access
- Public resource access
- Unused permissions

---

## 9. Monitor IAM Activity

Enable:

- AWS CloudTrail
- AWS Config

to monitor security events and configuration changes.

---

## 10. Secure Secrets

Never store:

- AWS Access Keys
- Passwords
- Tokens

inside:

- Source code
- Git repositories
- Configuration files

Use:

- AWS Secrets Manager
- AWS Systems Manager Parameter Store

---

# ❌ Common Mistakes

## Using the Root User Daily

❌ Wrong

```
Root User
       │
Everything
```

✅ Correct

```
Root User
      │
Create IAM Admin User
      │
Daily Operations
```

---

## Hardcoding Access Keys

Never store:

```
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY
```

inside applications.

Use IAM Roles instead.

---

## Using AdministratorAccess Everywhere

Many beginners assign:

```
AdministratorAccess
```

to every user.

Instead:

Create smaller policies with only the required permissions.

---

## Using Wildcards

Avoid:

```
Action: *

Resource: *
```

Grant only required actions.

---

## Forgetting MFA

Administrator accounts without MFA are a significant security risk.

---

## Creating Duplicate IAM Users Across Accounts

Instead of:

```
Account A
Developer

Account B
Developer

Account C
Developer
```

Use:

IAM Identity Center

or

Cross-Account IAM Roles.

---

## Not Rotating Access Keys

Unused access keys increase the attack surface.

Rotate and remove them regularly.

---

## Leaving Old IAM Users

Remove:

- Former employees
- Temporary users
- Test accounts

---

## Ignoring IAM Access Analyzer

Review findings regularly to detect:

- Public resources
- Cross-account access
- Unused permissions

---

# 🎤 Interview Questions

## Beginner Level

### What is IAM?

IAM is AWS's identity and access management service that controls authentication and authorization for AWS resources.

---

### Authentication vs Authorization?

Authentication verifies **who** the user is.

Authorization determines **what** the user is allowed to do.

---

### What is the difference between IAM User and IAM Role?

IAM User:

- Permanent identity
- Long-term credentials

IAM Role:

- Temporary identity
- Temporary credentials
- Assumed by trusted entities

---

### Why should EC2 use IAM Roles?

Because IAM Roles eliminate the need to store Access Keys on EC2 instances.

AWS automatically provides temporary credentials.

---

### What is STS?

AWS Security Token Service (STS) issues temporary security credentials for IAM Roles.

---

### What is the Principle of Least Privilege?

Grant only the permissions required to perform a task.

---

### What is IAM Identity Center?

A centralized identity management service providing Single Sign-On (SSO) across multiple AWS accounts.

---

### What is Amazon Cognito?

Amazon Cognito manages authentication and authorization for application users.

---

### Difference between User Pool and Identity Pool?

User Pool

- Authentication

Identity Pool

- Temporary AWS credentials

---

### What is a Resource-Based Policy?

A policy attached directly to a resource that specifies who can access it.

---

### IAM Policy vs Resource Policy?

IAM Policy

- Attached to identities

Resource Policy

- Attached to resources

---

### What is Cross-Account Access?

Allows users in one AWS account to securely access resources in another account using IAM Roles.

---

### What tools improve IAM security?

- Credential Report
- IAM Access Analyzer
- CloudTrail
- AWS Config

---

# 📝 Quick Revision Cheat Sheet

| Topic | Summary |
|---------|---------|
| IAM | Authentication & Authorization |
| User | Permanent Identity |
| Group | Collection of Users |
| Policy | JSON Permissions |
| Role | Temporary Identity |
| Root User | Account Owner |
| Identity Center | Enterprise SSO |
| Cognito | Application Users |
| User Pool | Authentication |
| Identity Pool | AWS Credentials |
| STS | Temporary Credentials |
| Resource Policy | Attached to Resources |
| Access Analyzer | Reviews Resource Access |
| Credential Report | Reviews IAM Credentials |

---

# 📌 Final Summary

AWS IAM is the foundation of AWS security.

A secure AWS environment is built by:

- Using IAM Users or IAM Identity Center instead of the Root User
- Applying the Principle of Least Privilege
- Using IAM Roles instead of long-term Access Keys
- Managing permissions with IAM Policies
- Reviewing security using Credential Reports and IAM Access Analyzer
- Continuously auditing IAM resources

Mastering IAM is essential because every AWS service integrates with it.

---

# 📚 Official AWS Documentation

## AWS IAM

https://docs.aws.amazon.com/IAM/latest/UserGuide/

## IAM Best Practices

https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

## IAM Roles

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html

## IAM Policies

https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html

## IAM Identity Center

https://docs.aws.amazon.com/singlesignon/

## Amazon Cognito

https://docs.aws.amazon.com/cognito/

## IAM Access Analyzer

https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html

## AWS Security Best Practices

https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/

## AWS Well-Architected Framework

https://docs.aws.amazon.com/wellarchitected/latest/framework/

