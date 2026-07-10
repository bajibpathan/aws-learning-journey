# 🧪 Lab 02 - Create IAM Users and Groups

## 📖 Lab Information

| Property | Value |
|----------|-------|
| **Category** | AWS Identity & Access Management |
| **Level** | 🟢 Beginner |
| **Estimated Time** | 15–20 Minutes |
| **AWS Services** | IAM |

---

## 🎯 Objective

Create IAM Users and IAM Groups to implement secure identity management and avoid using the AWS Root User for day-to-day administration.

---

## 📋 Prerequisites

- AWS Account
- Administrator permissions

---

## ⚙️ Implementation Steps

1. Sign in to the AWS Management Console.
2. Navigate to **IAM**.
3. Create an IAM Group.
4. Attach the required IAM Policy to the Group.
5. Create an IAM User.
6. Add the User to the IAM Group.
7. Enable AWS Management Console access.
8. Configure an initial password for the user.
9. Sign in using the newly created IAM User to verify access.

---

## ✅ Validation

Verify the following:

- The IAM User can successfully sign in to the AWS Management Console.
- The IAM User inherits permissions from the IAM Group.
- The IAM User can perform only the actions granted through the attached policy.
- The Root User is no longer required for daily administrative tasks.

---

## 🎯 Result

Successfully implemented secure user and access management using IAM Users and IAM Groups. Permissions are managed centrally through Groups, improving security and simplifying administration.

---

## 💡 Lessons Learned

- IAM Users provide individual identities for AWS access.
- IAM Groups simplify permission management by allowing permissions to be assigned once and inherited by multiple users.
- Assigning permissions to Groups is easier to manage than assigning permissions directly to individual users.
- Day-to-day administration should be performed using IAM Users instead of the Root User.

---

## 📚 References

- AWS IAM Users  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html

- AWS IAM User Groups  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html

- IAM Best Practices  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html