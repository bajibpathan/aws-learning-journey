# 💽 Amazon EC2 Instance Store

> EC2 Instance Store provides temporary block-level storage using disks that are physically attached to the host computer running the EC2 instance.

---

# 📖 Overview

In the previous topic, I learned about **Amazon EBS**, which provides persistent block storage for EC2 instances.

EBS storage is separate from the physical host running the EC2 instance.

There is another storage option called:

```text
EC2 Instance Store
```

The main difference I learned is where the storage comes from.

```text
Amazon EBS
    │
    └── Remote block storage

EC2 Instance Store
    │
    └── Storage physically attached
        to the EC2 host computer
```

Instance Store provides temporary block storage directly from disks attached to the physical server hosting the EC2 instance.

This makes Instance Store useful when an application needs very fast temporary storage and the data does not need to survive the lifetime of the instance.

---

# 🏗️ How Instance Store Works

An EC2 instance ultimately runs on physical infrastructure managed by AWS.

Some EC2 instance types have local storage devices physically attached to that host.

```text
Physical EC2 Host
┌─────────────────────────────┐
│                             │
│      EC2 Instance           │
│           │                 │
│           ▼                 │
│     Instance Store          │
│                             │
│    Local SSD / NVMe         │
│                             │
└─────────────────────────────┘
```

AWS exposes this storage to the EC2 instance as block devices.

The operating system can then format and mount those devices and use them for temporary data.

---

# 🧱 Instance Store is Block Storage

Just like EBS, Instance Store is presented to the operating system as block-level storage.

Conceptually:

```text
Application
     │
     ▼
File System
     │
     ▼
Operating System
     │
     ▼
Block Device
     │
     ▼
Instance Store
```

The important difference is not that one is block storage and the other is not.

Both provide block storage.

The important difference is **where the storage lives and how long the data persists**.

---

# 🆚 EBS vs Instance Store Architecture

### Amazon EBS

```text
EC2 Host
┌───────────────┐
│ EC2 Instance  │
└───────┬───────┘
        │
        │ AWS storage infrastructure
        ▼
┌───────────────┐
│  EBS Volume   │
└───────────────┘
```

### Instance Store

```text
EC2 Host
┌─────────────────────────────┐
│                             │
│ EC2 Instance                │
│      │                      │
│      ▼                      │
│ Instance Store              │
│                             │
└─────────────────────────────┘
```

Instance Store storage is physically attached to the host computer.

EBS is remote block storage.

---

# 🖥️ Instance Type Determines Instance Store

Not every EC2 instance type provides Instance Store.

The available:

* Number of devices
* Storage capacity
* Device type

depend on the selected EC2 instance type and size.

Therefore:

```text
Choose EC2 Instance Type
          │
          ▼
Does it support Instance Store?
          │
     ┌────┴────┐
     │         │
    Yes        No
     │         │
     ▼         ▼
Local       No Instance
Storage       Store
Available
```

This means Instance Store capacity cannot simply be selected independently like an EBS volume.

The storage characteristics come with the EC2 instance type.

---

# 🔗 Instance Store is Attached at Launch

Another important difference from EBS is when the storage becomes available.

EBS volumes can be created and attached to a compatible running or stopped EC2 instance later.

Instance Store volumes cannot be added after the EC2 instance has been launched.

```text
Instance Launch
      │
      ▼
Instance Store
Made Available
      │
      ▼
EC2 Running
```

For modern instance types with NVMe Instance Store, the supported Instance Store volumes are automatically attached when the instance launches.

They may still need to be:

```text
Formatted
    +
Mounted
```

before applications can use them.

---

# ⚡ Why Use Instance Store?

One of the main reasons to use Instance Store is its direct-attached nature.

```text
EC2 Instance
      │
      ▼
Local Instance Store
```

This makes it suitable for workloads requiring high-performance temporary storage.

However, performance alone is not enough to decide between EBS and Instance Store.

The first question should be:

> Can I afford to lose this data?

If the answer is no, Instance Store should not be the only place where that data exists.

---

# ⏳ Instance Store is Temporary Storage

The most important concept I learned is:

```text
Instance Store
      =
Ephemeral Storage
```

Instance Store should be treated as temporary storage.

AWS recommends it for information that changes frequently or can be recreated.

Examples include:

```text
Buffers

Caches

Scratch Data

Temporary Processing Data

Replicated Temporary Data
```

The data should not be treated as the only persistent copy of important information.

---

# 🔄 Instance Store Lifecycle

The lifetime of Instance Store data is closely connected to the lifecycle of the EC2 instance.

```text
EC2 Instance
     │
     ▼
Instance Store
     │
     ▼
Temporary Data
```

The important state transitions are:

| EC2 Action | Instance Store Data |
| ---------- | ------------------- |
| Reboot     | ✅ Preserved         |
| Stop       | ❌ Lost              |
| Hibernate  | ❌ Lost              |
| Terminate  | ❌ Lost              |

This is one of the most important differences between Instance Store and EBS.

---

# 🔁 Reboot Does NOT Delete Instance Store Data

A reboot behaves similarly to rebooting an operating system.

```text
EC2 Running
     │
     ▼
Reboot
     │
     ▼
EC2 Running
```

During a normal EC2 reboot, the instance remains on the same host computer.

Therefore:

```text
Instance Store Data
        │
        ▼
     Preserved
```

This is an important point:

> Rebooting an EC2 instance does not erase its Instance Store data.

---

# 🛑 Stop/Start is Different from Reboot

Stopping an EBS-backed EC2 instance is different from rebooting it.

```text
Reboot
   │
   └── Same host
       Instance Store preserved


Stop
   │
   └── Instance Store data erased
```

When an EC2 instance with Instance Store volumes is stopped, AWS erases the Instance Store data.

When the instance starts again, it should be treated as receiving fresh Instance Store storage.

Therefore:

```text
Stop
  +
Start
  ≠
Reboot
```

from an Instance Store data-persistence perspective.

---

# ❄️ Hibernate Also Removes Instance Store Data

Hibernation preserves memory contents by writing RAM to the EBS root volume.

However, it does **not** preserve Instance Store data.

```text
Hibernate
    │
    ├── RAM → EBS Root Volume
    │
    └── Instance Store → Data Lost
```

Therefore, applications should not depend on Instance Store data surviving hibernation.

---

# 🗑️ Termination Removes Instance Store Data

When an EC2 instance is terminated:

```text
EC2 Instance
     │
     ▼
Terminate
     │
     ▼
Instance Store Data
     │
     ▼
Permanently Lost
```

The data cannot be recovered from the Instance Store after termination.

Important data must therefore be copied elsewhere before terminating the instance.

---

# ⚠️ Host Failure

Instance Store storage is tied to the physical host.

If the underlying storage device or host fails, Instance Store data can be lost.

Therefore, Instance Store should never be the only location for valuable long-term data.

Instead:

```text
Temporary Processing
       │
       ▼
Instance Store
       │
       ▼
Final / Important Data
       │
       ▼
Persistent Storage
```

Persistent destinations could include:

```text
Amazon EBS

Amazon S3

Amazon EFS

Database / Other Persistent Service
```

depending on the workload.

---

# 🎯 Good Use Cases for Instance Store

Instance Store is useful when data is:

```text
Temporary

Replaceable

Re-creatable

Replicated Elsewhere

Performance Sensitive
```

Common examples include:

### Caches

```text
Application
     │
     ▼
Instance Store
     │
     ▼
Temporary Cache
```

If the cache disappears, the application rebuilds it.

---

### Scratch Space

Applications performing temporary processing may need fast working storage.

```text
Input Data
    │
    ▼
Instance Store
    │
    ▼
Processing
    │
    ▼
Result
    │
    ▼
Persistent Storage
```

The temporary intermediate files do not need to survive after processing completes.

---

### Buffers

Instance Store can hold temporary buffered data before it is processed or transferred elsewhere.

```text
Incoming Data
      │
      ▼
Temporary Buffer
Instance Store
      │
      ▼
Processing
      │
      ▼
Persistent Destination
```

---

### Temporary Data Processing

For example:

```text
S3
 │
 │ Input
 ▼
EC2
 │
 ▼
Instance Store
 │
 │ Fast Temporary Processing
 ▼
Result
 │
 ▼
S3
```

If the EC2 instance fails, the processing job can be restarted using the original persistent data.

This is a good example of designing around ephemeral storage.

---

# ❌ Poor Use Cases for Instance Store

Instance Store should not be the only storage location for data that must survive instance loss.

Examples include:

```text
Critical Business Data

Only Copy of Customer Data

Long-Term Application Data

Important Database Data
without replication

Files that cannot be recreated
```

The question to ask is:

```text
If this EC2 instance disappears,
can I recreate this data?
```

If:

```text
YES
 │
 ▼
Instance Store may be suitable
```

If:

```text
NO
 │
 ▼
Use persistent storage
or replicate the data elsewhere
```

---

# 💾 Amazon EBS vs Instance Store

The difference became much clearer to me by comparing them directly.

| Feature                             | Amazon EBS                        | EC2 Instance Store              |
| ----------------------------------- | --------------------------------- | ------------------------------- |
| Storage type                        | Block                             | Block                           |
| Storage location                    | Remote AWS storage infrastructure | Physically attached to EC2 host |
| Persistent                          | ✅ Yes                             | ❌ Temporary                     |
| Survives reboot                     | ✅                                 | ✅                               |
| Survives stop/start                 | ✅                                 | ❌                               |
| Survives hibernate                  | EBS data persists                 | ❌                               |
| Survives instance termination       | Depends on `DeleteOnTermination`  | ❌                               |
| Attach after launch                 | ✅                                 | ❌                               |
| Detach and move to another instance | ✅ Same-AZ rules apply             | ❌                               |
| Available on every instance type    | EBS supported broadly             | ❌ Depends on instance type      |
| Temporary cache/scratch data        | Possible                          | ✅ Strong use case               |
| Persistent application data         | ✅                                 | ❌ Not as sole copy              |

---

# 🧠 The Key Architecture Difference

The simplest way for me to remember the difference is:

```text
EBS
 │
 ▼
Storage exists independently
from the EC2 host


Instance Store
 │
 ▼
Storage is tied to
the EC2 host
```

This explains most of the lifecycle differences.

---

# 🧠 Compute Should Be Replaceable

Instance Store also reinforces an important cloud design principle.

Instead of designing:

```text
EC2 Instance
     │
     ▼
Critical Data
     │
     ▼
If EC2 dies
everything is lost
```

a better architecture separates temporary compute data from persistent data:

```text
Persistent Input
      │
      ▼
Replaceable EC2
      │
      ▼
Temporary Instance Store
      │
      ▼
Processing
      │
      ▼
Persistent Output
```

The EC2 instance can then be replaced without losing the authoritative copy of the data.

---

# ⚠️ Common Mistakes

### Mistake 1: Treating Instance Store as Persistent Storage

```text
Instance Store
      ≠
Persistent Storage
```

Do not store the only copy of important data there.

---

### Mistake 2: Assuming Stop/Start is the Same as Reboot

It is not.

```text
Reboot
  │
  └── Instance Store preserved


Stop / Start
  │
  └── Instance Store data lost
```

---

### Mistake 3: Assuming Instance Store Can Be Added Later

Unlike EBS:

```text
Launch EC2
    │
    ▼
Later attach Instance Store

❌
```

Instance Store availability is determined at launch and by the selected instance type.

---

### Mistake 4: Assuming Every EC2 Instance Has Instance Store

Not every EC2 instance type provides local Instance Store.

Always check the specifications of the selected instance type.

---

### Mistake 5: Trying to Move Instance Store to Another EC2 Instance

Instance Store cannot be detached from one EC2 instance and attached to another.

For movable persistent block storage, Amazon EBS is the appropriate model.

---

### Mistake 6: Keeping Important Results Only on Instance Store

For temporary processing:

```text
Input
  │
  ▼
Instance Store
  │
  ▼
Process
  │
  ▼
Important Result
```

move the important result to persistent storage.

For example:

```text
Important Result
      │
      ▼
Amazon S3 / EBS / EFS
```

before the EC2 instance is stopped or terminated.

---

# ✅ Best Practices

* Treat Instance Store as ephemeral storage.
* Use it for caches, buffers, scratch data, and temporary processing.
* Never keep the only copy of critical data on Instance Store.
* Replicate or copy important data to persistent storage.
* Design applications so temporary data can be recreated.
* Verify that the selected EC2 instance type supports Instance Store.
* Understand the number, size, and type of Instance Store devices provided by the selected instance type.
* Remember that Instance Store volumes cannot be added after launch.
* Format and mount Instance Store devices when required before using them.
* Understand EC2 lifecycle behavior before using Instance Store.
* Do not confuse reboot with stop/start.
* Design workloads using Instance Store so that EC2 instances remain replaceable.

---

# ❓ Interview Questions

### Q1. What is EC2 Instance Store?

**Answer**

EC2 Instance Store provides temporary block-level storage using disks that are physically attached to the host computer running the EC2 instance.

---

### Q2. What is the main difference between EBS and Instance Store?

**Answer**

EBS provides persistent block storage that exists independently of the EC2 host.

Instance Store provides temporary block storage from disks physically attached to the host running the EC2 instance.

---

### Q3. Does every EC2 instance type provide Instance Store?

**Answer**

No.

Instance Store availability, number of devices, size, and device type depend on the EC2 instance type and size.

---

### Q4. Can Instance Store be attached after launching an EC2 instance?

**Answer**

No.

Instance Store volumes are attached only when the instance is launched.

---

### Q5. Can an Instance Store volume be detached and attached to another EC2 instance?

**Answer**

No.

Instance Store is tied to the EC2 instance and its underlying host.

---

### Q6. Does Instance Store data survive an EC2 reboot?

**Answer**

Yes.

During a reboot, the EC2 instance remains on the same host and Instance Store data is preserved.

---

### Q7. Does Instance Store data survive stop/start?

**Answer**

No.

When an instance with Instance Store volumes is stopped, the data on those volumes is erased.

---

### Q8. Does Instance Store survive hibernation?

**Answer**

No.

Instance Store data is lost when the instance is hibernated.

---

### Q9. Does Instance Store survive instance termination?

**Answer**

No.

Instance Store data is permanently lost when the instance is terminated.

---

### Q10. What happens if the underlying Instance Store drive fails?

**Answer**

Data on the affected Instance Store can be lost.

Important data should therefore be replicated or copied to persistent storage.

---

### Q11. What are good workloads for Instance Store?

**Answer**

Examples include:

* Caches
* Buffers
* Scratch data
* Temporary processing data
* Replicated temporary data

The workload should be able to tolerate or recover from loss of the Instance Store data.

---

### Q12. Should I store a database on Instance Store?

**Answer**

It depends on the database architecture.

Instance Store should not contain the only persistent copy of important database data. Some specialized distributed or replicated database architectures can use local ephemeral storage safely because the data exists elsewhere and nodes are replaceable.

The important question is whether the architecture can recover if the Instance Store data disappears.

---

### Q13. Why can Instance Store provide high storage performance?

**Answer**

The storage devices are physically attached to the host computer running the EC2 instance, avoiding the remote-storage path used by network-attached block storage.

Actual performance still depends on the selected EC2 instance type and its Instance Store specifications.

---

### Q14. How should important data generated on Instance Store be protected?

**Answer**

Copy or replicate it to persistent storage such as:

```text
Amazon EBS

Amazon S3

Amazon EFS

or another persistent data service
```

before the Instance Store data can be lost.

---

# 💡 Key Takeaways

* EC2 Instance Store provides temporary block-level storage.
* The underlying storage devices are physically attached to the EC2 host computer.
* Instance Store availability depends on the EC2 instance type and size.
* Not every EC2 instance type provides Instance Store.
* Instance Store volumes are available only at instance launch and cannot be attached later.
* Instance Store cannot be detached and moved to another EC2 instance.
* Instance Store is designed for temporary or replaceable data.
* Data survives an EC2 reboot.
* Data does not survive stop, hibernate, or terminate operations.
* Host or storage-device failures can also result in data loss.
* Good use cases include caches, buffers, scratch space, and temporary processing.
* Critical data should be copied or replicated to persistent storage.
* EBS is persistent block storage, while Instance Store is ephemeral block storage.
* The most important design question is: **Can the application recover if this data disappears?**

---

# 📚 Related Topics

* Amazon EC2
* Amazon EBS
* EBS Volume Types
* EBS Snapshots
* EC2 Instance Types
* EC2 Instance Lifecycle
* Amazon S3
* Amazon EFS
* EC2 Auto Scaling
* Stateless Application Design

---

# 📖 References

* AWS Documentation: EC2 Instance Store
* AWS Documentation: Add Instance Store Volumes to an EC2 Instance
* AWS Documentation: Data Persistence for Instance Store Volumes
* AWS Documentation: EC2 Instance State Changes
* AWS Documentation: Storage Options for Amazon EC2
