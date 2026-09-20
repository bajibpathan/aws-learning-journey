# 🔐 Amazon EBS Encryption

> Amazon EBS encryption protects block-storage data using AWS Key Management Service (AWS KMS) while keeping encryption and decryption transparent to the EC2 operating system and applications.

---

# 📖 Overview

After learning how to create EBS volumes, attach them to EC2 instances, select the appropriate volume type, and create snapshots, the next important consideration is:

> How do I protect the data stored on those volumes?

An EBS volume may contain:

```text
Operating System

Application Files

Configuration

Database Files

Logs

Business Data

Credentials or Other Sensitive Data
```

For sensitive workloads, I do not want someone who gains unauthorized access to the underlying storage to be able to read the data directly.

Amazon EBS provides native encryption integrated with:

```text
AWS Key Management Service
        │
        ▼
      AWS KMS
```

The important thing I learned is that EBS encryption is largely transparent to the EC2 instance.

---

# 🎯 What Does EBS Encryption Protect?

When an encrypted EBS volume is attached to a supported EC2 instance, AWS encrypts:

```text
Encrypted EBS Volume
        │
        ├── Data at rest
        │
        ├── Data moving between
        │   EC2 and EBS
        │
        ├── Snapshots created
        │   from the volume
        │
        └── Volumes created
            from those snapshots
```

This means EBS encryption protects more than just the blocks sitting on the volume.

---

# 🔑 AWS KMS and EBS

Amazon EBS integrates with AWS KMS for key management.

When creating encrypted EBS resources, I can use:

```text
AWS KMS Key
│
├── AWS Managed Key
│      │
│      └── aws/ebs
│
└── Customer Managed Key
```

The AWS managed key for EBS has the alias:

```text
aws/ebs
```

AWS automatically creates this key in a Region when it is needed.

---

# 🔵 AWS Managed Key

The simplest option is the AWS managed KMS key:

```text
aws/ebs
```

AWS manages this key on my behalf.

Conceptually:

```text
EBS Volume
    │
    ▼
AWS Managed KMS Key
    │
    ▼
aws/ebs
```

This works well when I need EBS encryption but do not need detailed control over the KMS key itself.

---

# 🟢 Customer Managed Key

For greater control, I can create a:

```text
Customer Managed KMS Key
```

This gives me additional control over areas such as:

```text
Key Policies

Permissions

Key Rotation

Key Disablement

Auditing

Cross-Account Access
```

For enterprise environments, this can be important when the organization needs tighter control over who can use encryption keys.

---

# ⚠️ EBS Uses Symmetric KMS Keys

Amazon EBS supports:

```text
Symmetric KMS Keys
```

for EBS encryption.

It does not support asymmetric KMS keys for encrypting EBS volumes and snapshots.

---

# 🧠 How EBS Encryption Works

One of the most important concepts for me was understanding that the KMS key is not directly encrypting every block of application data.

Instead, EBS uses:

```text
KMS Key

and

Data Key
```

Conceptually:

```text
AWS KMS
   │
   │ KMS Key
   ▼
Generate Data Key
   │
   ▼
Unique Data Key
for EBS Volume
   │
   ▼
Encrypt Volume Data
```

For each encrypted volume, Amazon EBS asks AWS KMS to generate a unique data key.

---

# 🔑 KMS Key vs Data Key

The roles are different.

```text
KMS Key
   │
   └── Protects the data key


Data Key
   │
   └── Encrypts the actual
       EBS volume data
```

This separation is an important part of envelope encryption.

A simplified view is:

```text
             AWS KMS
                │
                ▼
             KMS Key
                │
                ▼
       Protects Data Key
                │
                ▼
         Encrypted Data Key
                │
                ▼
            EBS Volume
                │
                ▼
         Encrypted Data
```

---

# 🔐 What Happens When the Volume is Created?

When I create an encrypted EBS volume:

```text
Create Encrypted
EBS Volume
     │
     ▼
EBS Requests
Data Key from KMS
     │
     ▼
AWS KMS Generates
Unique Data Key
     │
     ▼
Data Key Protected
by KMS Key
     │
     ▼
Encrypted Data Key
Stored with Volume Information
```

The actual EBS data is encrypted using:

```text
AES-256
```

encryption.

---

# 🖥️ What Happens When the Volume is Attached?

When the encrypted EBS volume is attached to EC2:

```text
Encrypted EBS
     │
     ▼
Attach to EC2
     │
     ▼
EBS Calls AWS KMS
     │
     ▼
Decrypt Data Key
     │
     ▼
Plaintext Data Key
in Hypervisor Memory
     │
     ▼
Encrypt / Decrypt
EBS I/O
```

The plaintext data key is used in hypervisor memory to perform encryption and decryption operations.

---

# 🧠 Encryption is Transparent to the Operating System

This was one of the most important concepts for me.

EBS encryption happens below the operating system layer.

```text
Application
     │
     ▼
Operating System
     │
     ▼
EC2 / Hypervisor
     │
     ├── Encryption
     └── Decryption
     │
     ▼
Encrypted EBS
```

From the operating system's perspective, it works with the filesystem normally.

For example:

```text
Linux
  │
  └── EXT4 / XFS

Windows
  │
  └── NTFS
```

The operating system does not need to perform the EBS encryption operation itself.

---

# 🆚 EBS Encryption vs OS-Level Encryption

EBS encryption should not be confused with encryption performed inside the operating system.

For example:

```text
EBS Encryption

EC2
 │
 ▼
Hypervisor / EBS Layer
 │
 ▼
Encrypted EBS


Windows BitLocker

Application
 │
 ▼
Windows
 │
 ▼
BitLocker
 │
 ▼
Storage
```

They operate at different layers.

| EBS Encryption              | OS-Level Encryption                      |
| --------------------------- | ---------------------------------------- |
| Managed through AWS         | Managed inside OS                        |
| Integrated with AWS KMS     | Uses OS encryption technology            |
| Transparent to applications | OS participates directly                 |
| Protects EBS storage        | Can provide additional OS-level controls |

They can also be used together when a security architecture requires multiple layers of encryption.

---

# 🌎 EBS Encryption by Default

An important correction to older training material is that I should not assume:

```text
New EBS Volume
      =
Unencrypted
```

Amazon EBS provides:

```text
Encryption by Default
```

This setting is configured on a:

```text
Per-Region Basis
```

For example:

```text
AWS Account
│
├── ca-central-1
│      └── Encryption by Default: Enabled
│
└── us-east-1
       └── Encryption by Default: Disabled
```

The setting in one Region does not automatically determine the setting in another Region.

---

# 🔐 What Happens When Encryption by Default is Enabled?

If EBS encryption by default is enabled for a Region:

```text
Create New EBS Volume
          │
          ▼
Automatically Encrypted
```

It also affects snapshot copies created in that Region.

If I do not explicitly select another permitted KMS key, EBS uses the configured default encryption key for that Region.

By default, that is normally:

```text
aws/ebs
```

although the account can configure a customer managed symmetric KMS key as the default.

---

# ⚠️ Existing Resources Are Not Changed

Enabling encryption by default does not retroactively encrypt existing resources.

```text
Enable Encryption
by Default
     │
     ├── New Resources
     │      └── Encrypted
     │
     └── Existing Resources
            └── Unchanged
```

This distinction is important.

---

# ⚙️ Configuring Encryption by Default

In the EC2 console, the setting is available under the Region-specific EC2 settings.

Conceptually:

```text
EC2
 │
 ▼
Settings
 │
 ▼
Data Protection and Security
 │
 ▼
EBS Encryption
 │
 ▼
Enable Encryption by Default
```

I can also select the default KMS key.

---

# 💾 Creating an Encrypted EBS Volume

When creating a new EBS volume, encryption can be enabled and an appropriate KMS key selected.

For example:

```text
Create Volume
│
├── Volume Type: gp3
├── Size: 20 GiB
├── Availability Zone: ca-central-1a
│
└── Encryption
       │
       ├── aws/ebs
       │
       └── Customer Managed Key
```

If encryption by default is already enabled for the Region, encryption is automatically applied.

---

# 🖥️ Encrypting an EC2 Root Volume

EBS encryption is not limited to additional data volumes.

It can also protect:

```text
EC2 Root Volume
```

For example:

```text
EC2
│
├── Root EBS
│      └── Encrypted
│
└── Data EBS
       └── Encrypted
```

Both boot and data volumes can therefore use EBS encryption.

---

# 📸 EBS Encryption and Snapshots

Encryption automatically follows snapshots created from encrypted volumes.

```text
Encrypted EBS
      │
      ▼
Create Snapshot
      │
      ▼
Encrypted Snapshot
```

The snapshot uses the same KMS key as the source encrypted volume.

Similarly:

```text
Encrypted Snapshot
      │
      ▼
Create Volume
      │
      ▼
Encrypted EBS Volume
```

An encrypted snapshot cannot be used to create an unencrypted EBS volume.

---

# 🔓 What About an Unencrypted Volume?

Suppose I already have:

```text
Unencrypted EBS Volume
```

and later decide that it should be encrypted.

The encryption state of the existing volume itself cannot simply be toggled.

One established method is:

```text
Unencrypted
EBS Volume
     │
     ▼
Create Snapshot
     │
     ▼
Unencrypted Snapshot
     │
     ▼
Copy Snapshot
+ Enable Encryption
     │
     ▼
Encrypted Snapshot
     │
     ▼
Create Volume
     │
     ▼
Encrypted EBS Volume
```

This creates a new encrypted resource rather than changing the existing volume in place.

---

# 📸 Snapshot Encryption Rules

A snapshot created directly from a volume inherits the encryption state of that volume.

```text
Unencrypted Volume
        │
        ▼
Unencrypted Snapshot
```

and:

```text
Encrypted Volume
        │
        ▼
Encrypted Snapshot
```

For an encrypted source volume, the snapshot uses the same KMS key.

---

# 🔄 Encrypting an Unencrypted Snapshot

An unencrypted snapshot can be copied and encryption enabled for the copy.

```text
Unencrypted
Snapshot
    │
    ▼
Copy Snapshot
    │
    ├── Enable Encryption
    │
    └── Select KMS Key
    │
    ▼
Encrypted
Snapshot Copy
```

A volume created from that encrypted snapshot will also be encrypted.

---

# 🔑 Re-Encrypting with Another KMS Key

Snapshot copying can also be useful when I need to change the KMS key protecting the data.

For example:

```text
Encrypted Snapshot
      │
      │ KMS Key A
      ▼
Copy Snapshot
      │
      │ Select KMS Key B
      ▼
Encrypted Snapshot Copy
      │
      └── KMS Key B
```

This is useful when changing encryption-key strategies or preparing resources for certain sharing and migration scenarios.

---

# 📊 Encryption Outcomes

A simplified model is:

| Source                      | Operation                  | Result                 |
| --------------------------- | -------------------------- | ---------------------- |
| Unencrypted volume          | Create snapshot            | Unencrypted snapshot   |
| Encrypted volume            | Create snapshot            | Encrypted snapshot     |
| Unencrypted snapshot        | Copy + enable encryption   | Encrypted snapshot     |
| Encrypted snapshot          | Create volume              | Encrypted volume       |
| Encrypted snapshot          | Copy using another KMS key | Re-encrypted snapshot  |
| Existing unencrypted volume | Enable encryption directly | Not an in-place toggle |

Encryption by default can change some of these outcomes by automatically requiring encryption for newly created resources in that Region.

---

# 🔑 Unique Data Keys

Each encrypted EBS volume uses a unique data key.

Conceptually:

```text
KMS Key
│
├── Volume A
│     └── Data Key A
│
├── Volume B
│     └── Data Key B
│
└── Volume C
      └── Data Key C
```

The KMS key can therefore protect many separate EBS data keys.

This is another example of envelope encryption.

---

# 🏢 Why Use a Customer Managed KMS Key?

For a simple environment, the AWS managed:

```text
aws/ebs
```

key may be sufficient.

Enterprise environments may require greater control.

For example:

```text
Security Requirement
        │
        ▼
Customer Managed Key
        │
        ├── Control who can use it
        ├── Define key policy
        ├── Disable key
        ├── Configure rotation
        └── Audit usage
```

A customer managed key is therefore useful when the organization needs direct control over the lifecycle and permissions of the encryption key.

---

# ⚠️ KMS Permissions Matter

Encrypting an EBS volume does not mean everyone who can access EC2 automatically has permission to use every KMS key.

For customer managed keys, appropriate KMS permissions are required.

Conceptually:

```text
User / Role
    │
    ├── EC2 / EBS Permission
    │
    └── KMS Permission
             │
             ▼
       Encrypted Volume
```

This becomes especially important with:

```text
Cross-Account Sharing

Custom KMS Keys

Snapshot Copying

Disaster Recovery
```

---

# 🏗️ Example Architecture

Consider a database running on EC2:

```text
Application
     │
     ▼
EC2 Database Server
     │
     ▼
Encrypted EBS
     │
     ▼
AWS KMS
```

Snapshots inherit the encryption:

```text
Encrypted EBS
      │
      ▼
Encrypted Snapshot
      │
      ▼
Backup / Recovery
```

The application and operating system continue to use the storage normally while EBS handles encryption transparently.

---

# 🧠 Security Layers

EBS encryption protects storage, but it is only one part of a complete security architecture.

```text
Application Security
        │
        ▼
IAM / Access Control
        │
        ▼
Operating System Security
        │
        ▼
Network Security
        │
        ▼
EBS Encryption
        │
        ▼
Encrypted Storage
```

Encryption does not replace:

```text
IAM

Security Groups

Patching

Application Security

Secrets Management

Backups

Monitoring
```

It complements them.

---

# ⚠️ Common Mistakes

### Mistake 1: Assuming All New EBS Volumes Are Unencrypted

Always check:

```text
EBS Encryption by Default
```

for the Region.

---

### Mistake 2: Assuming Encryption by Default is Global

It is Region-specific.

```text
ca-central-1
     ≠
us-east-1
```

Check each Region used by the architecture.

---

### Mistake 3: Assuming Existing Volumes Become Encrypted

Enabling encryption by default does not modify existing EBS volumes or snapshots.

---

### Mistake 4: Trying to Toggle Encryption on an Existing Volume

Encryption is not simply switched on for the existing resource.

Create an encrypted copy or use the snapshot-copy workflow to create a new encrypted resource.

---

### Mistake 5: Formatting a Volume After Encryption Migration

If I create an encrypted volume from a snapshot containing existing data:

```text
Attach
  │
  ▼
Mount Existing Filesystem
```

I should not format it unless I intentionally want to erase the data.

---

### Mistake 6: Ignoring KMS Permissions

A customer managed KMS key provides more control, but workloads and administrators need the appropriate permissions to use it.

---

### Mistake 7: Thinking EBS Encryption Happens Inside Linux or Windows

Native EBS encryption operates below the guest operating system.

The operating system does not perform the EBS encryption itself.

---

# ✅ Best Practices

* Enable EBS encryption by default in Regions where organizational policy requires encrypted storage.
* Verify encryption settings in every Region being used.
* Encrypt both root and data volumes when sensitive information is involved.
* Use customer managed KMS keys when the organization requires greater control over key policies and lifecycle.
* Follow least privilege when granting KMS permissions.
* Never assume enabling encryption by default modifies existing volumes.
* Understand snapshot encryption inheritance before designing backup workflows.
* Test encrypted snapshot restoration as part of backup and disaster recovery exercises.
* Monitor KMS and EBS activity using appropriate AWS auditing and monitoring services.
* Clean up unused EBS volumes and snapshots to avoid unnecessary costs.

---

# ❓
