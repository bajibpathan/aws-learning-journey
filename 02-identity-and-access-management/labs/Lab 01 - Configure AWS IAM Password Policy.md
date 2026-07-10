# 🧪 Lab 01 - Configure AWS IAM Password Policy

## 📖 Lab Information

| Property | Value |
|----------|-------|
| **Category** | AWS Identity & Access Management |
| **Level** | 🟢 Beginner |
| **Estimated Time** | 10–15 Minutes |
| **AWS Services** | IAM |

---

## 🎯 Objective

Configure an AWS IAM Password Policy to enforce strong password requirements and improve the security of IAM users.

---

## 📋 Prerequisites

- AWS Account
- Administrator permissions

---

## ⚙️ Implementation Steps

1. Sign in to the AWS Management Console.
2. Navigate to **IAM**.
3. Select **Account Settings**.
4. Edit the Password Policy.
5. Configure the following settings:
   - Minimum password length
   - Require uppercase letters
   - Require lowercase letters
   - Require numbers
   - Require special characters
   - Prevent password reuse
6. Save the password policy.

---

## ✅ Validation

Verify the following:

- Passwords must meet the configured complexity requirements.
- Weak passwords are rejected.
- Previously used passwords cannot be reused (if password history is enabled).

---

## 🎯 Result

Successfully configured an IAM Password Policy that enforces strong password requirements for all IAM users in the AWS account.

---

## 💡 Lessons Learned

- IAM Password Policies are configured at the AWS account level.
- Strong password requirements improve account security.
- Password policies should be combined with Multi-Factor Authentication (MFA) for better protection.

---

## 📚 References

- AWS IAM Password Policy  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html

- AWS IAM Best Practices  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html