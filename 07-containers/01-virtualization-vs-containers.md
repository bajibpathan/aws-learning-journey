# Containers on AWS: Virtualization vs Containerization

> Containers provide a lightweight approach to running applications by sharing the host operating system kernel while keeping applications and their dependencies isolated.

---

## 📖 Overview

Virtualization transformed infrastructure management by allowing multiple virtual machines (VMs) to run on a single physical server.

However, traditional virtual machines introduce overhead because each VM requires its own guest operating system.

**Containerization takes a different approach.** Instead of creating a complete virtual machine for every application, containers share the host operating system kernel and run as isolated processes.

This makes containers more lightweight, faster to start, and efficient in their use of infrastructure resources.

## 1. Traditional Virtualization

Traditional virtualization allows multiple virtual machines to share the resources of a physical server.

Technologies such as VMware and Microsoft Hyper-V helped make this approach widely adopted.

AWS uses its **Nitro System** to support EC2 virtualization.

### How virtual machines work

A physical server provides computing resources such as CPU, memory, and storage.

A hypervisor allows multiple virtual machines to use these resources.

Each virtual machine includes:

- Its own guest operating system
- Application runtime environments
- Required libraries and dependencies
- Application code
- Allocated CPU, memory, and storage

### Architecture

```text
┌─────────────────────────────────────┐
│             Applications            │
├───────────┬────────────┬────────────┤
│  App A    │   App B    │   App C    │
│ Libraries │ Libraries  │ Libraries  │
│ Guest OS  │ Guest OS   │ Guest OS   │
├───────────┴────────────┴────────────┤
│              Hypervisor             │
├─────────────────────────────────────┤
│          Physical Hardware          │
│          CPU | RAM | Storage        │
└─────────────────────────────────────┘
```

Each VM operates independently, even though the underlying physical hardware is shared.

### Example: Application environments

An organization might run separate virtual machines for development, testing, and production.

| Environment | Guest OS | Application Dependencies |
|---|---|---|
| Development | Amazon Linux | Newer Node.js and MySQL versions |
| Testing | Amazon Linux | Dependencies under evaluation |
| Production | Amazon Linux | Stable Node.js and MySQL versions |

This isolation allows different versions of application dependencies to coexist without interfering with one another.

### Advantages of virtual machines

- Strong isolation between environments
- Independent operating systems
- Separate runtime environments and libraries
- Ability to run multiple workloads on shared physical hardware
- Better hardware utilization than deploying a dedicated physical server for every application

### Limitations of virtual machines

**1. Operating system overhead**

Every VM requires its own guest operating system, consuming CPU, memory, and storage.

**2. Resource duplication**

Multiple VMs may run identical operating systems and libraries, duplicating resources.

**3. Administrative overhead**

Each VM requires ongoing maintenance, including operating system updates, security patches, and application dependency management.

**4. Scalability challenges**

Managing a few virtual machines may be straightforward. Maintaining hundreds or thousands increases operational complexity.

These limitations create an opportunity for a more lightweight approach.

---

## 2. What Is Containerization?

Containerization is a form of **operating system-level virtualization**.

Instead of creating a complete virtual machine for every application, containers run as isolated processes that share the host operating system kernel.

A container packages the application with the dependencies required to run it.

These may include:

- Application code
- Runtime environments
- Libraries
- Utilities
- Required application services

Unlike a virtual machine, a container does not require a separate guest operating system.

### Container architecture

```text
┌─────────────────────────────────────┐
│              Containers             │
├───────────┬────────────┬────────────┤
│   App A   │   App B    │   App C    │
│ Libraries │ Libraries  │ Libraries  │
│ Runtime   │ Runtime    │ Runtime    │
├───────────┴────────────┴────────────┤
│          Container Engine           │
│                Docker               │
├─────────────────────────────────────┤
│          Host Operating System      │
│           Shared OS Kernel          │
├─────────────────────────────────────┤
│          Physical Hardware          │
│          CPU | RAM | Storage        │
└─────────────────────────────────────┘
```

The main difference is that **containers share the host OS kernel rather than running a separate guest OS for every application**.

## 3. What Is a Container Engine?

A container engine provides the functionality needed to create, run, and manage containers.

One popular example is **Docker**.

In the architecture discussed in this lesson:

1. Physical hardware provides CPU, memory, and storage.
2. The host operating system runs on the hardware.
3. Docker provides the container engine.
4. Multiple containers run on top of the container engine.
5. Each container executes its application with the required dependencies.

Docker will be covered in greater detail in the next lesson.

---

## 4. Key Benefits of Containers

### Lightweight architecture

Containers do not require a complete guest operating system for every application.

This reduces the amount of infrastructure resources needed compared with traditional VMs.

### Faster startup

A virtual machine needs to boot its guest operating system.

A container starts its application process without booting a separate guest OS.

As a result, containers generally start much faster.

### Better resource utilization

Because containers have a smaller footprint, more application workloads can run on the same underlying infrastructure.

This can improve resource efficiency and reduce unnecessary infrastructure consumption.

### Application isolation

Containers run as isolated processes with their own application environments, file systems, and dependencies.

For example, separate containers can run different application runtime versions without requiring a separate VM for each workload.

### Portability

Containers package applications and their dependencies into a consistent environment.

This makes it easier to move applications between compatible container environments.

### Support for microservices

Containers are particularly useful for microservices and cloud-native applications.

Instead of packaging an entire application into a single virtual machine, individual application components can run in separate containers.

---

## 5. Virtual Machines vs Containers

| Feature | Virtual Machines | Containers |
|---|---|---|
| Virtualization type | Hardware-level | Operating system-level |
| Isolation | Separate guest operating systems | Isolated processes sharing a kernel |
| Guest OS | Required for each VM | Not required for each container |
| Resource consumption | Higher | Lower |
| Startup time | Generally slower | Generally faster |
| Application dependencies | Installed within each VM | Packaged with each container |
| Portability | More infrastructure-dependent | Easier across compatible environments |
| Management overhead | OS and application maintenance per VM | Shared host OS with container-level dependencies |
| Typical use | Isolated operating systems and workloads | Lightweight applications and microservices |

### The fundamental difference

**Virtual machines virtualize hardware. Containers virtualize the operating system environment.**

Both approaches allow multiple applications to share physical infrastructure, but they use different architectures.

---

## 6. Example: Running Multiple Applications

Consider three applications:

- A Node.js development application
- A testing application
- A production application

### Using virtual machines

Each application runs in its own VM with its own guest operating system.

```text
Physical Server
      │
      ▼
Hypervisor
      │
      ├── VM 1 → Guest OS → Development App
      ├── VM 2 → Guest OS → Testing App
      └── VM 3 → Guest OS → Production App
```

### Using containers

Each application runs in an isolated container while sharing the host operating system kernel.

```text
Physical Server
      │
      ▼
Host Operating System
      │
      ▼
Docker Engine
      │
      ├── Container 1 → Development App
      ├── Container 2 → Testing App
      └── Container 3 → Production App
```

The container approach avoids duplicating a complete guest operating system for every application.

---

## ⚠️ Common Misconceptions

**1. Containers are the same as virtual machines.**

Both provide isolation, but containers share the host OS kernel while VMs run separate guest operating systems.

**2. Containers do not need an operating system.**

Containers rely on the host operating system kernel. They simply do not require a separate guest OS for every container.

**3. Containers include only application code.**

Containers also package the runtime libraries and dependencies required by the application.

**4. Containers completely replace virtual machines.**

Containers and VMs address different requirements. Containers are often preferable for lightweight application deployment, while VMs remain useful when separate operating systems and stronger isolation boundaries are needed.

**5. Containers can run unchanged on absolutely any host.**

Portability depends on compatibility with the container runtime, operating system kernel, and processor architecture.

---

## ❓ Interview Questions

### Q1. What is virtualization?

Virtualization allows multiple virtual machines to share physical computing resources through a hypervisor.

### Q2. What is a hypervisor?

A hypervisor enables virtual machines to run on underlying physical hardware.

### Q3. What is containerization?

Containerization is operating system-level virtualization that allows applications to run as isolated processes sharing the host OS kernel.

### Q4. What is the main difference between VMs and containers?

Virtual machines have separate guest operating systems, whereas containers share the host operating system kernel.

### Q5. Why are containers lightweight?

They do not require a complete guest operating system for every application.

### Q6. Why do containers start faster than VMs?

Containers start application processes without booting a separate guest operating system.

### Q7. What is Docker?

Docker is a container technology used to create and manage containerized applications.

### Q8. What does a container contain?

Application code, runtime dependencies, libraries, utilities, and other components needed by the application.

### Q9. Why are containers useful for microservices?

They allow individual application components to be packaged and run independently in lightweight isolated environments.

### Q10. What are the disadvantages of traditional virtual machines?

Guest operating system overhead, resource duplication, slower startup, and additional maintenance responsibilities.

---

## 💡 Key Takeaways

- Virtualization enables multiple virtual machines to run on shared physical hardware.
- Hypervisors provide the virtualization layer for virtual machines.
- Every VM includes its own guest operating system.
- Separate guest operating systems introduce resource and administrative overhead.
- Containerization provides operating system-level virtualization.
- Containers run as isolated processes sharing the host OS kernel.
- Containers package applications together with their runtime dependencies.
- Docker is a popular container engine.
- Containers generally use fewer resources and start faster than VMs.
- Containers improve portability across compatible environments.
- Containerization is particularly useful for microservices and cloud-native applications.

**The most important concept to remember:**

Virtual machines provide isolation by running separate guest operating systems, while containers provide lightweight application isolation by sharing the host operating system kernel.

---

## 📚 Related Topics

- Docker Fundamentals
- Container Images
- Container Runtime
- Amazon Elastic Container Service (ECS)
- Amazon Elastic Kubernetes Service (EKS)
- AWS Fargate
- Microservices Architecture
- Cloud-Native Applications

