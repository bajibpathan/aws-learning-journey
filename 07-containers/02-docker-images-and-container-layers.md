# Docker Images and Container Layers

> Docker images provide the foundation for creating containers. They are built using Dockerfile instructions and consist of multiple read-only layers, allowing Docker to reuse unchanged components and build images efficiently.

---

## 📖 Overview

In the previous lesson, we learned that containers are lightweight, isolated environments used to run applications and their dependencies.

But how are containers actually created?

Containers are created from **container images**, such as Docker images.

Docker uses a file called a **Dockerfile** to define how an image should be built.

The basic workflow is:

```text
Dockerfile
    │
    ▼
Build Image
    │
    ▼
Docker Image
    │
    ▼
Run Container
```

The Dockerfile contains the instructions required to prepare the application environment. Docker uses those instructions to build an image, which can then be used to launch one or more containers.

---

## 1. What Is a Dockerfile?

A Dockerfile is a text file containing instructions that describe how to build a Docker image.

It defines the components required by an application, such as:

- Base image
- Application runtime
- Required libraries and extensions
- Operating system packages and updates
- Application source code

Docker processes these instructions to create the final image.

### Example Dockerfile

The lesson describes an application using PHP, Apache, and MySQL extensions.

A simplified example of that structure is:

```dockerfile
FROM php:8.2-apache

RUN docker-php-ext-install mysqli pdo_mysql

RUN apt-get update

COPY . /var/www/html/
```

This is an illustrative Dockerfile based on the components discussed in the lesson, not the exact source file shown by the instructor.

### Understanding the instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Selects the parent or base image |
| `RUN` | Executes commands during the image build |
| `COPY` | Copies application files into the image |

The first instruction starts with a PHP and Apache parent image.

Subsequent instructions add MySQL extensions, update package information, and copy application code into the web server directory.

---

## 2. What Is a Docker Image?

A Docker image is a packaged collection of the components required to run an application.

It can contain:

- Application code
- Runtime environments
- Required libraries
- Dependencies
- Utilities
- Filesystem contents
- Environment configuration

Once built, the image can be used to launch containers.

```text
Docker Image
     │
     ├── Application Code
     ├── Runtime
     ├── Libraries
     ├── Dependencies
     └── Configuration
              │
              ▼
         Container
```

An important characteristic of Docker images is that they are built using **layers**.

---

## 3. Understanding Docker Image Layers

Docker images are composed of multiple layers stacked on top of one another.

The lesson explains that Dockerfile instructions can introduce new layers containing changes to the image filesystem.

For example:

```text
┌────────────────────────────────┐
│ Layer 4: Application Code      │
│ COPY application files         │
├────────────────────────────────┤
│ Layer 3: Package Updates       │
│ RUN apt-get update             │
├────────────────────────────────┤
│ Layer 2: MySQL Extensions      │
│ RUN install extensions         │
├────────────────────────────────┤
│ Layer 1: Parent Image          │
│ PHP + Apache                   │
└────────────────────────────────┘
```

Each filesystem layer represents changes relative to the layers beneath it.

Together, these layers form the final Docker image.

### How layers are created

**Layer 1: Parent image**

The build starts with a parent image that provides the foundation for the application.

In this example, the parent image contains PHP and Apache.

**Layer 2: Install MySQL extensions**

The next instruction adds the extensions required for PHP to communicate with MySQL.

**Layer 3: Perform updates**

Another instruction updates package information or installs required components.

**Layer 4: Copy application code**

The application source files are copied into the image.

The resulting layers are combined to produce the final image.

---

## 4. What Is a Parent Image?

A parent image is the starting point for building another Docker image.

For example:

```dockerfile
FROM php:8.2-apache
```

This tells Docker to use an existing PHP and Apache image as the foundation.

Additional instructions customize that image for the application.

```text
Parent Image
PHP + Apache
     │
     ▼
Add MySQL Extensions
     │
     ▼
Add Required Packages
     │
     ▼
Copy Application Code
     │
     ▼
Final Docker Image
```

Using a parent image avoids having to build the entire application environment from scratch.

---

## 5. What Is Docker Layer Caching?

One of the important advantages of Docker's layered architecture is **layer caching**.

Docker can reuse previously built layers when their inputs have not changed.

This means that rebuilding an image does not always require repeating every build operation.

### Example

Suppose an image contains four layers:

```text
Layer 4: Application Code
Layer 3: Package Updates
Layer 2: MySQL Extensions
Layer 1: PHP + Apache
```

Now imagine that only the application source code changes.

The lower layers may remain unchanged.

Docker can reuse the cached results of those earlier build steps and rebuild the affected portion of the image.

```text
┌────────────────────────────────┐
│ Application Code               │
│ Rebuild Required               │
├────────────────────────────────┤
│ Package Updates                │
│ Cached                         │
├────────────────────────────────┤
│ MySQL Extensions               │
│ Cached                         │
├────────────────────────────────┤
│ PHP + Apache                   │
│ Cached                         │
└────────────────────────────────┘
```

This can significantly reduce build time.

### Why instruction order matters

The lesson emphasizes that components that change frequently should generally be placed later in the Dockerfile.

For example:

- Base images change relatively infrequently.
- Application dependencies may change occasionally.
- Application source code may change frequently.

A useful general ordering is:

```text
Less Frequently Changed
         │
         ▼
    Base Image
         │
         ▼
    Dependencies
         │
         ▼
  Application Code
         │
         ▼
More Frequently Changed
```

**Key principle:** Place frequently changing build steps later where practical so that earlier cached layers can be reused.

---

## 6. Building the Final Docker Image

After Docker processes the Dockerfile instructions, it produces the final image.

```text
Dockerfile
    │
    ├── Parent Image
    ├── Install Extensions
    ├── Perform Updates
    └── Copy Application Code
             │
             ▼
       Docker Build
             │
             ▼
       Final Docker Image
```

The image contains the filesystem and configuration required to launch the application as a container.

The lesson also introduces the concept of storing images in a **repository**.

An image repository provides a location where built images can be stored and retrieved for deployment.

The next AWS container services topics build on this concept.

---

## 7. Docker Images vs Running Containers

A Docker image and a Docker container are not the same thing.

A Docker image is the packaged template used to create containers.

A container is a running instance created from that image.

```text
                  Docker Image
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
        Container 1 Container 2 Container 3
```

Multiple containers can be created from the same image.

Each container runs as an isolated process while using the application components provided by the image.

---

## 8. Read-Only Image Layers

The lesson highlights that Docker image filesystem layers are **read-only**.

Once an image has been built, its layers provide the filesystem foundation used when containers are launched.

```text
┌────────────────────────────────┐
│ Application Code               │
│ Read-Only                      │
├────────────────────────────────┤
│ Dependencies                   │
│ Read-Only                      │
├────────────────────────────────┤
│ Runtime                        │
│ Read-Only                      │
├────────────────────────────────┤
│ Base Image                     │
│ Read-Only                      │
└────────────────────────────────┘
```

However, applications running inside containers may need to create or modify files.

This is where the container's writable layer becomes important.

---

## 9. Container Read-Write Layer

When Docker creates a container from an image, it adds a thin **read-write layer** on top of the image layers.

```text
┌────────────────────────────────┐
│ Container Read-Write Layer     │
│ Writable                       │
├────────────────────────────────┤
│ Application Code               │
│ Read-Only                      │
├────────────────────────────────┤
│ Dependencies                   │
│ Read-Only                      │
├────────────────────────────────┤
│ Runtime                        │
│ Read-Only                      │
├────────────────────────────────┤
│ Base Image                     │
│ Read-Only                      │
└────────────────────────────────┘
```

This writable layer allows the running container to make filesystem changes.

Examples include:

- Creating new files
- Modifying existing files
- Deleting files

These changes are recorded in the container's writable layer rather than modifying the underlying image layers.

### Example

Suppose a container creates a new file:

```text
/app/output.txt
```

That file is created in the container's writable filesystem.

The original Docker image remains unchanged.

This separation allows multiple containers to use the same image while maintaining their own filesystem changes.

---

## 10. Image Layers vs Container Writable Layer

| Feature | Image Layers | Container Writable Layer |
|---|---|---|
| Created during | Image build | Container creation |
| Access | Read-only | Read-write |
| Purpose | Store image filesystem contents | Store runtime filesystem changes |
| Shared between containers | Can be reused | Unique to each container |
| Example | Application code and dependencies | Files created or modified at runtime |

The important distinction is:

**Docker images provide the read-only foundation, while running containers have their own writable layer.**

---

## 11. Complete Docker Image Lifecycle

The entire process discussed in the lesson can be summarized as:

```text
              Dockerfile
                  │
                  ▼
          Docker Build Process
                  │
       ┌──────────┴──────────┐
       │                     │
       ▼                     ▼
  Parent Image         Instructions
       │                     │
       └──────────┬──────────┘
                  │
                  ▼
           Image Layers
                  │
                  ▼
          Final Docker Image
                  │
                  ▼
          Image Repository
                  │
                  ▼
           Run Container
                  │
                  ▼
       Add Read-Write Layer
                  │
                  ▼
        Running Application
```

This is the foundation for understanding how container platforms deploy applications from images.

---

## ⚠️ Common Mistakes

### 1. Confusing Docker images with containers

An image is used to create containers.

A container is a running instance of an image.

### 2. Assuming every container needs a separate image

Multiple containers can be launched from the same Docker image.

### 3. Ignoring Dockerfile instruction order

Frequently changing instructions placed too early can reduce the benefits of layer caching.

### 4. Assuming image layers are writable

Docker image filesystem layers are read-only.

Runtime filesystem changes are written to the container's writable layer.

### 5. Assuming container changes modify the original image

Changes made inside a running container do not directly modify the underlying Docker image.

---

## ❓ Interview Questions

### Q1. What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build a Docker image.

### Q2. What is a Docker image?

A Docker image is a packaged collection of application code, runtime components, libraries, dependencies, and configuration used to create containers.

### Q3. What is a parent image?

A parent image provides the starting foundation for building a new Docker image.

### Q4. What are Docker image layers?

Image layers represent filesystem changes introduced during the image build process.

### Q5. Why does Docker use layers?

Layers allow image components to be organized and reused, helping make image builds more efficient.

### Q6. What is Docker layer caching?

Layer caching allows Docker to reuse previously built layers when their inputs have not changed.

### Q7. Why should frequently changing instructions be placed later in a Dockerfile?

Doing so can improve cache reuse and reduce unnecessary rebuild operations.

### Q8. Are Docker image layers writable?

No. Docker image filesystem layers are read-only.

### Q9. How can a running container modify files?

Docker adds a writable layer on top of the image layers when the container is created.

### Q10. What happens when a container creates a new file?

The new file is stored in the container's writable filesystem layer.

### Q11. Can multiple containers use the same Docker image?

Yes. Multiple containers can be created from the same image, each with its own writable layer.

### Q12. What is the difference between a Docker image and a container?

A Docker image is the template used to create containers. A container is a running instance created from that image.

---

## 💡 Key Takeaways

- Containers are created from container images.
- Dockerfiles define the instructions used to build Docker images.
- Docker images begin with a parent or base image.
- Image build instructions can introduce additional filesystem layers.
- Docker image filesystem layers are read-only.
- Docker layer caching allows unchanged build results to be reused.
- Dockerfile instruction order affects caching efficiency.
- Frequently changing application code should generally be copied later in the build process.
- A final Docker image contains the application and its required dependencies.
- Docker images can be stored in image repositories.
- Multiple containers can be launched from the same image.
- Every running container has its own writable layer.
- Filesystem changes made by a running container do not modify the original image.

### The most important concept to remember

```text
Dockerfile
    │
    ▼
Docker Image
    │
    ├── Read-Only Layer 1
    ├── Read-Only Layer 2
    ├── Read-Only Layer 3
    └── Read-Only Layer 4
                │
                ▼
         Run Container
                │
                ▼
       Read-Write Layer
                │
                ▼
       Running Application
```

---

## 📚 Related Topics

- Virtualization vs Containerization
- Docker Fundamentals
- Dockerfile Instructions
- Docker Image Layers
- Docker Layer Caching
- Container Writable Layers
- Container Image Repositories
- Amazon Elastic Container Service (ECS)

