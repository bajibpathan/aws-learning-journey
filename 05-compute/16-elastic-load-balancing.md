# ⚖️ AWS Elastic Load Balancing (ELB)

> AWS Elastic Load Balancing automatically distributes incoming traffic across multiple healthy targets, helping applications achieve high availability, scalability, and fault tolerance.

---

# 📖 Overview

Consider an application running on a single EC2 instance:

```text
Users
  │
  ▼
EC2 Instance
```

If that instance fails because of an underlying infrastructure problem:

```text
Users
  │
  ▼
EC2 Instance ❌
  │
  ▼
Application Unavailable
```

Even if we quickly launch another instance, users may experience an outage.

A better architecture uses:

- Multiple instances
- Multiple Availability Zones
- An Elastic Load Balancer
- Health checks

```text
                    Users
                      │
                      ▼
             Elastic Load Balancer
                      │
              ┌───────┴───────┐
              ▼               ▼
             AZ-A             AZ-B
              │               │
          EC2 Instance    EC2 Instance
```

If one instance becomes unhealthy, the load balancer stops sending traffic to it and continues routing requests to healthy resources.

---

# 🎯 What Is Elastic Load Balancing?

Elastic Load Balancing distributes incoming traffic across multiple:

- EC2 instances
- Containers
- IP addresses
- Lambda functions

depending on the type of load balancer.

These resources are generally organized into:

```text
Target Groups
```

The basic architecture is:

```text
Clients
   │
   ▼
Load Balancer
   │
   ▼
Target Group
   │
   ├── Target 1
   ├── Target 2
   └── Target 3
```

The load balancer distributes requests only to targets considered healthy.

---

# ❤️ Health Checks

Health checks are fundamental to load balancing.

The target group performs health checks against its registered targets.

```text
Target Group
     │
     ├── Target A ✅
     ├── Target B ❌
     └── Target C ✅
```

The load balancer sends traffic to:

```text
Target A + Target C
```

but stops routing requests to:

```text
Target B
```

If Target B later becomes healthy again, it can begin receiving traffic again.

This improves both availability and the user experience.

---

# 🌐 Internet-Facing vs Internal Load Balancers

Elastic Load Balancers can be deployed for different purposes.

## Internet-Facing Load Balancer

An internet-facing load balancer accepts traffic originating from outside the VPC.

A recommended architecture described in the lesson is:

```text
Internet
   │
   ▼
Internet-Facing Load Balancer
   │
   ▼
Private EC2 Instances
```

The load balancer nodes are internet-facing while the application servers themselves can remain in private subnets without public IP addresses.

This improves the application's security posture.

---

## Internal Load Balancer

An internal load balancer distributes traffic within the VPC.

For example:

```text
Frontend Tier
     │
     ▼
Internal Load Balancer
     │
     ▼
Backend Tier
```

This is useful for load balancing internal application components that should not be directly exposed to the internet.

---

# 🏗️ Secure Multi-Tier Architecture

Instead of placing application servers directly in public subnets:

```text
Internet
   │
   ▼
Public Load Balancer
   │
   ▼
Private Frontend Servers
   │
   ▼
Internal Load Balancer
   │
   ▼
Private Backend Servers
```

Only the internet-facing load balancer is exposed externally.

The application instances remain private.

---

# 🎧 Listeners

A listener checks for incoming connection requests using a configured:

```text
Protocol + Port
```

For example:

| Listener | Typical Purpose |
|---|---|
| HTTP : 80 | HTTP web traffic |
| HTTPS : 443 | Encrypted web traffic |
| TCP : custom port | TCP application traffic |

Conceptually:

```text
Client
   │
   ▼
Listener
Protocol + Port
   │
   ▼
Routing Rule
   │
   ▼
Target Group
```

---

# 🎯 Target Groups

A target group contains the resources that receive traffic.

For example:

```text
Application Load Balancer
          │
          ▼
      Target Group
          │
    ┌─────┼─────┐
    ▼     ▼     ▼
   EC2   EC2   EC2
```

Target groups also have their own health-check configuration.

With an Application Load Balancer, targets can include:

- EC2 instances
- Containers
- IP addresses
- Lambda functions

A target can also be registered with multiple target groups.

---

# 🔀 Cross-Zone Load Balancing

Cross-zone load balancing allows traffic to be distributed across healthy targets in all enabled Availability Zones.

Consider:

```text
AZ-A                     AZ-B
8 Instances              2 Instances
```

Without cross-zone load balancing, the lesson illustrates traffic being divided equally between the Availability Zones:

```text
                Incoming Traffic
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
            50%                 50%
            AZ-A                AZ-B
         8 Instances         2 Instances
```

This creates an uneven per-instance load.

Each instance in AZ-A receives:

```text
50% ÷ 8 = 6.25%
```

while each instance in AZ-B receives:

```text
50% ÷ 2 = 25%
```

With cross-zone load balancing enabled:

```text
10 Healthy Instances
        │
        ▼
Traffic Distributed
Across All Targets
        │
        ▼
~10% per Instance
```

This reduces the need to maintain exactly the same number of instances in each Availability Zone and can improve traffic distribution.

---

# 🏷️ Types of AWS Load Balancers

The lesson covers four load balancer types:

| Load Balancer | Layer | Primary Use |
|---|---:|---|
| Classic Load Balancer (CLB) | Layer 4 & 7 | Previous-generation load balancing |
| Application Load Balancer (ALB) | Layer 7 | HTTP/HTTPS application routing |
| Network Load Balancer (NLB) | Layer 4 | TCP/UDP, static IP, high-performance networking |
| Gateway Load Balancer (GWLB) | Layer 3 | Virtual network appliances |

---

# 1️⃣ Classic Load Balancer (CLB)

The Classic Load Balancer is the previous-generation AWS load balancer.

The lesson describes it as distributing traffic across EC2 instances in multiple Availability Zones.

```text
Users
  │
  ▼
Classic Load Balancer
  │
  ├── EC2
  ├── EC2
  └── EC2
```

It supports both:

```text
Layer 4
+
Layer 7
```

and protocols such as TCP, HTTP, and HTTPS.

However, it is a previous-generation solution and newer architectures should generally use the newer load balancer types.

### Key CLB Characteristics

- Previous-generation load balancer
- Supports EC2 instances
- Supports Layer 4 and Layer 7 traffic
- Supports health checks
- Supports listeners
- Can distribute traffic across multiple Availability Zones

---

# 2️⃣ Application Load Balancer (ALB)

An Application Load Balancer operates at:

```text
Layer 7
```

of the OSI model.

It is designed primarily for:

```text
HTTP
HTTPS
```

application traffic.

```text
Client
   │
   ▼
Application Load Balancer
   │
   ▼
Listener Rules
   │
   ▼
Target Groups
```

ALB provides much more intelligent application-level routing than simple network-level traffic distribution.

---

# 🧭 ALB Listener Rules

ALB listeners use rules to determine how requests should be routed.

Each rule can contain:

```text
Priority

Conditions

Actions
```

When the conditions match, the configured action is performed.

This allows different requests to be sent to different target groups.

---

# 🛣️ Path-Based Routing

ALB supports routing based on URL paths.

For example:

```text
example.com/
      │
      ▼
Web Target Group


example.com/orders
      │
      ▼
Orders Target Group


example.com/products
      │
      ▼
Products Target Group
```

This is particularly useful for applications composed of multiple services.

---

# 🌐 Host-Based Routing

ALB can also route based on hostname.

For example:

```text
shop.example.com
       │
       ▼
Shopping Target Group


api.example.com
       │
       ▼
API Target Group
```

The lesson also mentions routing based on:

- HTTP headers
- Query parameters

This allows much more granular application routing.

---

# 📦 Multiple Applications on One Instance

ALB can support multiple applications running on the same EC2 instance using different ports.

For example:

```text
EC2 Instance
    │
    ├── Port 80
    │     └── Application A
    │
    └── Port 8080
          └── Application B
```

---

# 🎯 ALB Target Types

ALB target groups can contain:

```text
EC2 Instances

IP Addresses

Containers

Lambda Functions
```

The lesson also notes that IP targets can support resources outside the VPC.

---

# 🔐 ALB Encryption in Transit

Application Load Balancers can terminate HTTPS connections using SSL/TLS certificates.

The lesson introduces:

```text
AWS Certificate Manager
        │
        ▼
       ACM
```

for managing certificates.

Architecture:

```text
Client
   │
   │ HTTPS
   ▼
Application Load Balancer
   │
   ▼
Application
```

This provides encryption for data while it is being transmitted.

---

# 🍪 Sticky Sessions

ALB supports:

```text
Sticky Sessions
       │
       ▼
Session Affinity
```

This allows requests from a particular client to continue being routed to a specific target for the session.

Conceptually:

```text
Client A ──────► Instance 1
Client A ──────► Instance 1
Client A ──────► Instance 1
```

instead of potentially moving between different targets.

---

# 🛡️ AWS WAF Integration

The lesson also describes ALB integration with:

```text
AWS Web Application Firewall
           │
           ▼
          WAF
```

This can help protect web applications against application-layer threats such as:

- SQL injection
- Cross-site scripting

---

# 🔄 ALB Routing Algorithms

The lesson introduces two routing algorithms:

```text
Round Robin

Least Outstanding Requests
```

---

# 🔁 Round Robin

Round robin distributes requests sequentially across available targets.

For three targets:

```text
Request 1 ──► Target A
Request 2 ──► Target B
Request 3 ──► Target C
Request 4 ──► Target A
```

This works well when backend targets have similar capacity and performance characteristics.

A common example is a stateless web application.

---

# 📉 Least Outstanding Requests

The:

```text
Least Outstanding Requests
```

algorithm routes the next request to the target currently handling the fewest outstanding requests.

For example:

```text
Target A = 10 Active Requests
Target B =  3 Active Requests
Target C =  7 Active Requests

New Request
     │
     ▼
Target B
```

This can be useful when requests require different amounts of processing time.

---

# 3️⃣ Network Load Balancer (NLB)

A Network Load Balancer operates at:

```text
Layer 4
```

and supports protocols such as:

```text
TCP
UDP
```

It is designed for high-performance network traffic.

```text
Client
   │
   ▼
Network Load Balancer
   │
   ▼
Target Group
   │
   ├── EC2
   ├── IP
   └── ALB
```

---

# ⚡ When to Think About NLB

The lesson highlights NLB for scenarios involving:

```text
Layer 4 Traffic

TCP / UDP

Static IP Addresses

Very High Request Volumes

Volatile Workloads
```

It describes NLB as capable of handling millions of requests per second.

---

# 🔢 Static IP Addressing

One major NLB feature discussed in the lesson is static IP addressing.

Load balancer nodes use network interfaces that can have fixed IP addresses.

For an internet-facing NLB, an:

```text
Elastic IP Address
```

can also be associated.

This is useful when applications or firewalls require known IP addresses.

For example:

```text
Firewall
   │
   │ Allow 203.x.x.x
   ▼
Static NLB IP
   │
   ▼
Network Load Balancer
```

This makes NLB useful for:

```text
IP Whitelisting
```

and applications requiring hard-coded IP addresses.

---

# 🔀 NLB Flow Hash

The lesson explains that NLB uses flow information to select targets.

For TCP, this includes information such as:

- Protocol
- Source IP
- Source port
- Destination IP
- Destination port
- TCP sequence number

Each TCP connection is routed to one target for the lifetime of that connection.

UDP traffic also uses flow information to select the target.

---

# 🎯 NLB Target Types

The lesson identifies NLB targets as:

```text
EC2 Instances

IP Addresses

Application Load Balancer
```

The ability to use an ALB as an NLB target creates an important architecture pattern.

---

# 🔗 NLB in Front of ALB

AWS allows an Application Load Balancer to be used as a target of a Network Load Balancer.

```text
Client
   │
   ▼
Network Load Balancer
   │
   ▼
Application Load Balancer
   │
   ▼
Application Targets
```

Why would we do this?

Because the architecture can combine:

```text
NLB Features
     +
ALB Features
```

For example:

```text
NLB
 │
 ├── Static IP
 ├── Elastic IP
 └── PrivateLink
       │
       ▼
      ALB
       │
       ├── Path Routing
       ├── Host Routing
       └── Layer 7 Features
```

---

# 🎯 NLB + ALB Use Cases

The lesson identifies several reasons for this architecture.

### Static IP Requirement

ALB addresses are dynamic.

If an application requires a fixed IP:

```text
Client
   │
   ▼
Static IP
   │
   ▼
NLB
   │
   ▼
ALB
```

### Firewall Whitelisting

A static NLB address can be added to firewall allow lists.

### Legacy Applications

Some legacy clients may require hard-coded IP addresses rather than DNS-based access.

### AWS PrivateLink

The architecture can also support PrivateLink-based connectivity while retaining ALB Layer 7 capabilities.

---

# 🏗️ API Gateway + VPC Link Example

The lesson includes a real-world multi-tier application where the goal was to keep the Application Load Balancer internal.

The architecture described is:

```text
End User
   │
   ▼
API Gateway
   │
   ▼
API VPC Link
   │
   ▼
Internal NLB
   │
   ▼
Internal ALB
   │
   ▼
Backend Application
```

The purpose was to avoid exposing the backend load balancers directly to the public internet while still allowing API Gateway to reach the application through private connectivity.

The ALB then provides the Layer 7 routing required by the backend application.

---

# 4️⃣ Gateway Load Balancer (GWLB)

Gateway Load Balancer is different from ALB and NLB.

Its main purpose is to help deploy, scale, and manage:

```text
Third-Party
Virtual Network Appliances
```

Examples could include network appliances used to inspect traffic.

---

# 🔍 Traffic Inspection

Suppose incoming traffic needs to be inspected before reaching the application.

```text
Client
   │
   ▼
Gateway Load Balancer
   │
   ▼
Network Appliance
   │
   │ Inspect Traffic
   ▼
Gateway Load Balancer
   │
   ▼
Web Application
```

If the traffic is accepted:

```text
Traffic
   │
   ▼
Inspection
   │
   ▼
Clean
   │
   ▼
Web Server
```

If the traffic is rejected:

```text
Traffic
   │
   ▼
Inspection
   │
   ▼
Rejected
   │
   ▼
DROP
```

The web server itself does not need to be aware that this inspection is occurring.

---

# 🔄 Symmetric Traffic Flow

The lesson describes the response following the same path.

```text
REQUEST

Client
  │
  ▼
GWLB
  │
  ▼
Network Appliance
  │
  ▼
GWLB
  │
  ▼
Web Server


RESPONSE

Web Server
  │
  ▼
GWLB
  │
  ▼
Network Appliance
  │
  ▼
GWLB
  │
  ▼
Client
```

This keeps the network appliance transparently inserted into the traffic path.

---

# 🌐 GWLB Layer and Protocol

The lesson describes Gateway Load Balancer as operating at:

```text
Layer 3
```

and using:

```text
GENEVE
```

which stands for:

```text
Generic Network Virtualization Encapsulation
```

The protocol operates on:

```text
Port 6081
```

Gateway Load Balancer combines:

```text
Transparent Network Gateway

+

Load Balancing
```

for virtual appliances.

---

# 📊 Load Balancer Comparison

| Feature | CLB | ALB | NLB | GWLB |
|---|---|---|---|---|
| Generation | Previous | Current | Current | Current |
| OSI Layer | L4 & L7 | L7 | L4 | L3 |
| Main Traffic | TCP, HTTP/HTTPS | HTTP/HTTPS | TCP/UDP | IP packets |
| Advanced HTTP Routing | No | Yes | No | No |
| Path-Based Routing | No | Yes | No | No |
| Host-Based Routing | No | Yes | No | No |
| Static IP Focus | No | No | Yes | No |
| Lambda Target | No | Yes | No | No |
| ALB as Target | No | No | Yes | No |
| Virtual Appliances | No | No | No | Yes |
| GENEVE | No | No | No | Yes |
| Primary Purpose | Legacy ELB | Application routing | Network traffic | Traffic inspection |

---

# 🧠 Which Load Balancer Should I Choose?

A useful decision model is:

```text
What do you need?
       │
       ├── HTTP/HTTPS + Smart Routing
       │          │
       │          ▼
       │         ALB
       │
       ├── TCP/UDP + Static IP
       │          │
       │          ▼
       │         NLB
       │
       ├── Virtual Network Appliances
       │          │
       │          ▼
       │        GWLB
       │
       └── Existing Legacy ELB
                  │
                  ▼
                 CLB
```

---

# ⚠️ Common Mistakes

## Mistake 1: Putting Application Servers Directly on the Internet

A stronger architecture is:

```text
Internet
   │
   ▼
Internet-Facing ALB
   │
   ▼
Private EC2 Instances
```

The application servers do not need public IP addresses.

---

## Mistake 2: Confusing ALB and NLB

Remember:

```text
ALB
 │
 ▼
Layer 7
 │
 ▼
HTTP / HTTPS
 │
 ▼
Application Routing
```

versus:

```text
NLB
 │
 ▼
Layer 4
 │
 ▼
TCP / UDP
 │
 ▼
Network Traffic
```

---

## Mistake 3: Choosing ALB When Static IP Is Required

The lesson emphasizes NLB when static IP addressing is required.

```text
Static IP
   │
   ▼
NLB
```

---

## Mistake 4: Using NLB for Path-Based Routing

Path-based and host-based routing are ALB capabilities.

```text
/application-a
/application-b
/api
     │
     ▼
    ALB
```

---

## Mistake 5: Using ALB for Third-Party Network Appliances

For transparent traffic inspection through virtual appliances:

```text
Gateway Load Balancer
```

is the load balancer discussed in the lesson.

---

## Mistake 6: Ignoring Health Checks

Load balancers depend on health checks to determine which targets should receive traffic.

```text
Healthy
   │
   ▼
Receive Traffic


Unhealthy
   │
   ▼
Stop Traffic
```

---

# ❓ Interview Questions

### Q1. What is Elastic Load Balancing?

Elastic Load Balancing distributes incoming traffic across multiple healthy targets to improve availability, scalability, and fault tolerance.

### Q2. What is a target group?

A target group is a collection of registered resources that receive traffic from a load balancer.

### Q3. What happens when a target fails its health check?

The load balancer stops sending traffic to that target and continues using healthy targets.

### Q4. What is the difference between internet-facing and internal load balancers?

Internet-facing load balancers accept external traffic, while internal load balancers distribute traffic within the VPC.

### Q5. Why should EC2 application instances normally remain private?

The load balancer can provide the internet-facing entry point while the application servers remain unexposed in private subnets.

### Q6. What is cross-zone load balancing?

It allows traffic to be distributed across healthy targets in all enabled Availability Zones rather than being constrained to targets in a particular zone.

### Q7. Which load balancer operates at Layer 7?

```text
Application Load Balancer
```

### Q8. Which load balancer should you consider for HTTP/HTTPS routing?

```text
ALB
```

### Q9. What advanced routing capabilities does ALB provide?

The lesson discusses:

- Path-based routing
- Host-based routing
- Header-based routing
- Query parameter-based routing

### Q10. What target types does ALB support in the lesson?

- EC2 instances
- IP addresses
- Containers
- Lambda functions

### Q11. What is a sticky session?

Session affinity that keeps a client associated with a particular backend target.

### Q12. Which AWS service can provide certificates for ALB?

```text
AWS Certificate Manager
        │
        ▼
       ACM
```

### Q13. Which security service can integrate with ALB?

```text
AWS WAF
```

### Q14. What ALB routing algorithms are discussed?

```text
Round Robin

Least Outstanding Requests
```

### Q15. Which layer does NLB operate at?

```text
Layer 4
```

### Q16. Which protocols are highlighted for NLB?

```text
TCP

UDP
```

### Q17. When would you choose NLB instead of ALB?

Think about NLB when the requirement emphasizes:

- Layer 4 networking
- TCP/UDP
- Static IP addresses
- Elastic IP addresses
- Very high traffic volumes

### Q18. Can an ALB be a target of an NLB?

Yes.

The lesson describes:

```text
NLB
 │
 ▼
ALB
 │
 ▼
Application
```

### Q19. Why put an NLB in front of an ALB?

To combine NLB capabilities such as static IP addressing or PrivateLink with ALB Layer 7 application routing.

### Q20. What is Gateway Load Balancer designed for?

Deploying, scaling, and load balancing traffic through third-party virtual network appliances.

### Q21. Which layer does GWLB operate at?

```text
Layer 3
```

### Q22. Which protocol does GWLB use?

```text
GENEVE
```

### Q23. Which port does GENEVE use according to the lesson?

```text
6081
```

---

# 💡 Key Takeaways

- Elastic Load Balancing distributes traffic across healthy targets.
- Targets should be distributed across multiple Availability Zones for high availability.
- Health checks determine which targets receive traffic.
- Load balancers can be internet-facing or internal.
- Internet-facing load balancers can front application servers located in private subnets.
- Target groups organize backend resources.
- Listeners define the protocols and ports accepted by a load balancer.
- Cross-zone load balancing helps distribute traffic across targets in multiple Availability Zones.
- Classic Load Balancer is the previous-generation load balancer discussed in the lesson.
- ALB operates at Layer 7 and is designed for HTTP/HTTPS application routing.
- ALB supports path-based and host-based routing.
- ALB can integrate with ACM and AWS WAF.
- NLB operates at Layer 4 and supports TCP/UDP traffic.
- NLB is useful when static IP addressing is required.
- An ALB can be registered as an NLB target.
- NLB + ALB can combine static IP/network capabilities with Layer 7 routing.
- Gateway Load Balancer is designed for third-party virtual network appliances.
- GWLB operates at Layer 3 and uses GENEVE on port 6081 according to the lesson.

The simplest way to remember the load balancers is:

```text
ALB
=
APPLICATION
=
Layer 7
=
HTTP / HTTPS


NLB
=
NETWORK
=
Layer 4
=
TCP / UDP + Static IP


GWLB
=
GATEWAY
=
Layer 3
=
Network Appliances


CLB
=
CLASSIC
=
Previous Generation
```

---

# 📚 Related Topics

- Elastic Load Balancing
- Application Load Balancer
- Network Load Balancer
- Gateway Load Balancer
- Classic Load Balancer
- Target Groups
- Listeners
- Health Checks
- Cross-Zone Load Balancing
- AWS Certificate Manager
- AWS WAF
- AWS PrivateLink
- API Gateway VPC Link
- High Availability
- Multi-AZ Architecture