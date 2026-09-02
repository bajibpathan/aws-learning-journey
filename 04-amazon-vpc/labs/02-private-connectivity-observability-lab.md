# 🧪 Lab 02: Private AWS Connectivity & Network Observability

> Extend the Core VPC Networking lab by introducing private connectivity to AWS services using VPC Endpoints and troubleshooting network traffic using VPC Flow Logs.

---

# 📖 Lab Overview

In Lab 01, we built the core VPC networking architecture:

* Custom VPC
* Public and private subnets
* Multiple Availability Zones
* Internet Gateway
* NAT Gateway
* Route tables
* Security Groups
* Network ACLs
* VPC Flow Logs

Private EC2 instances were able to reach the Internet through a NAT Gateway.

But what happens when a private EC2 instance only needs to communicate with an AWS service such as Amazon S3?

Should the traffic travel through:

```text
Private EC2
    │
    ▼
NAT Gateway
    │
    ▼
Internet Gateway
    │
    ▼
Amazon S3
```

Or can we provide a more direct private path?

This lab explores that question using an **Amazon S3 Gateway VPC Endpoint**.

We will also use **VPC Flow Logs** to investigate network traffic and troubleshoot deliberately introduced failures.

---

# 🎯 Learning Objectives

By completing this lab, you should be able to:

* Explain why VPC Endpoints exist
* Create an S3 Gateway Endpoint
* Explain how Gateway Endpoints modify routing
* Access Amazon S3 privately from a VPC
* Compare NAT Gateway and VPC Endpoint traffic paths
* Explain AWS-managed prefix lists
* Understand endpoint policies
* Use VPC Flow Logs
* Identify `ACCEPT` and `REJECT` traffic
* Troubleshoot routing and security problems
* Understand when NAT Gateway is still required

---

# 🏗️ Starting Architecture

This lab builds on Lab 01.

```text
VPC: 10.0.0.0/16

                Internet
                    │
                    ▼
                   IGW
                    │
        ┌───────────┴───────────┐
        │                       │
      AZ-A                    AZ-B
        │                       │
    Public-A                Public-B
  10.0.1.0/24             10.0.2.0/24
        │
     NAT Gateway
        │
   ┌────┴──────────────────────┐
   │                           │
Private-A                   Private-B
10.0.3.0/24                10.0.4.0/24
   │                           │
 EC2-A                       EC2-B
```

Private route table:

```text
Destination        Target

10.0.0.0/16        local
0.0.0.0/0          NAT Gateway
```

---

# 🧩 The Problem

Assume EC2-A needs to download an object from Amazon S3.

Without a VPC Endpoint, its path can use the NAT architecture:

```text
EC2-A
   │
   ▼
Private Route Table
   │
   ▼
NAT Gateway
   │
   ▼
Internet Gateway
   │
   ▼
Amazon S3
```

The EC2 instance remains private, but NAT Gateway is involved.

AWS provides another option:

```text
Gateway VPC Endpoint
```

For Amazon S3 and DynamoDB, a Gateway Endpoint can provide private connectivity from the VPC without requiring NAT Gateway or Internet Gateway for that service traffic.

---

# 🔐 Gateway VPC Endpoint

Create an Amazon S3 Gateway Endpoint.

Conceptually:

```text
Private EC2
     │
     ▼
Private Route Table
     │
     ▼
S3 Gateway Endpoint
     │
     ▼
Amazon S3
```

Gateway Endpoints are:

* Route-table based
* Private
* Highly available
* AWS managed
* Available for Amazon S3 and DynamoDB
* Not implemented using ENIs in your subnet
* Not associated with Security Groups

---

# 🛣️ Route Table Changes

When the S3 Gateway Endpoint is associated with a route table, AWS adds a route using an AWS-managed prefix list.

Conceptually:

```text
Destination          Target

10.0.0.0/16          local
S3 Prefix List       vpce-xxxxx
0.0.0.0/0            NAT Gateway
```

The S3 route is more specific than the default route.

Therefore S3 traffic uses:

```text
S3 Prefix List
      │
      ▼
VPC Endpoint
```

instead of:

```text
0.0.0.0/0
     │
     ▼
NAT Gateway
```

---

# 🧠 Important Routing Concept

AWS routing uses **longest prefix match**.

Suppose the route table contains:

```text
0.0.0.0/0 → NAT Gateway

S3 Prefix List → VPC Endpoint
```

Traffic matching the S3 prefix list uses the VPC Endpoint because that route is more specific than the default route.

This allows the same private EC2 instance to use:

```text
Amazon S3
     │
     ▼
VPC Endpoint
```

while other Internet destinations continue using:

```text
Internet
    │
    ▼
NAT Gateway
```

---

# 🪣 S3 Test Bucket

Create a temporary S3 bucket for this lab.

Upload a simple object such as:

```text
network-test.txt
```

Example contents:

```text
Private S3 connectivity test successful.
```

The bucket does not need to be publicly accessible.

Keep S3 Block Public Access enabled.

---

# 🔑 EC2 Permissions

The EC2 instance needs appropriate IAM permissions to access the test S3 bucket.

Prefer an IAM role attached to the EC2 instance rather than storing AWS credentials on the server.

The role should provide only the S3 permissions required for the exercise.

Example requirement:

```text
EC2-A
   │
IAM Role
   │
   ▼
Read Test S3 Bucket
```

---

# 🧪 Phase 1: Establish the Baseline

Before creating the VPC Endpoint, verify that the private EC2 instance can access S3.

For example:

```bash
aws s3 ls
```

or:

```bash
aws s3 cp s3://<bucket-name>/network-test.txt .
```

The command should succeed if:

* IAM permissions are correct
* DNS is working
* NAT connectivity is working
* S3 access is permitted

This establishes the baseline.

---

# 🧪 Phase 2: Create the S3 Gateway Endpoint

Create a Gateway VPC Endpoint for:

```text
Amazon S3
```

Associate the endpoint with the private route table.

Inspect the route table afterward.

Identify the new route.

You should see a route conceptually similar to:

```text
S3 Prefix List → VPC Endpoint
```

Do not simply accept that AWS created the route.

Understand why it exists.

---

# 🧪 Phase 3: Validate Private S3 Access

From the private EC2 instance, access the S3 object again.

```bash
aws s3 cp s3://<bucket-name>/network-test.txt .
```

Confirm that the operation succeeds.

Now examine the architecture.

Before:

```text
EC2
 │
 ▼
NAT Gateway
 │
 ▼
Internet Gateway
 │
 ▼
S3
```

After:

```text
EC2
 │
 ▼
Gateway Endpoint
 │
 ▼
S3
```

The VPC Endpoint provides the private network path for S3 traffic.

---

# 💥 Phase 4: Prove NAT Is Not Required for S3

Now perform an important experiment.

Temporarily remove the private route:

```text
0.0.0.0/0 → NAT Gateway
```

Your private EC2 instance should lose general IPv4 Internet connectivity.

Test:

```bash
curl https://example.com
```

Expected:

```text
FAIL
```

Now test S3:

```bash
aws s3 cp s3://<bucket-name>/network-test.txt .
```

Expected:

```text
SUCCESS
```

Why?

Because S3 traffic has a separate route:

```text
S3 Prefix List → VPC Endpoint
```

This demonstrates that a VPC Endpoint and NAT Gateway solve different connectivity requirements.

Restore the NAT route after the experiment if it is still required.

---

# 🔐 Endpoint Policy

A Gateway Endpoint can have an endpoint policy controlling which resources or operations can be accessed through the endpoint.

For the lab, examine the endpoint policy.

Then consider how it could be restricted to:

```text
Only the lab S3 bucket
```

instead of providing broader S3 access.

This introduces another layer of access control:

```text
Network Path
     +
IAM Permissions
     +
Endpoint Policy
     +
S3 Bucket Policy
```

Successful access may depend on multiple layers.

---

# 📊 Phase 5: VPC Flow Logs

Use the VPC Flow Logs configured during Lab 01.

For this lab, capture:

```text
ALL
```

traffic.

This allows analysis of:

```text
ACCEPT
```

and:

```text
REJECT
```

records.

Remember:

> VPC Flow Logs contain network flow metadata. They do not contain packet payloads.

---

# 🔍 Understanding Flow Log Records

A Flow Log record can contain fields such as:

```text
srcaddr
dstaddr
srcport
dstport
protocol
packets
bytes
action
```

Example:

```text
10.0.3.10
      │
      │ TCP 443
      ▼
Destination
      │
      ▼
ACCEPT
```

or:

```text
10.0.3.10
      │
      │ TCP 443
      ▼
Destination
      │
      ▼
REJECT
```

---

# 💥 Phase 6: Generate REJECT Traffic

Intentionally create a controlled network failure.

For example:

* Modify a NACL rule
* Attempt access to a blocked destination/port
* Restore the rule after the test

Generate the traffic and inspect VPC Flow Logs.

Search for:

```text
REJECT
```

Identify:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
Action
```

---

# 🔍 Troubleshooting Exercise

Imagine an application team reports:

> "EC2-A cannot reach a required service."

Do not immediately change Security Groups.

Follow the packet path.

```text
EC2-A
  │
  ▼
Destination?
  │
  ▼
Route Table
  │
  ▼
Route Target
  │
  ▼
Security Controls
  │
  ▼
Flow Logs
  │
  ▼
Application / Service
```

Ask:

1. What is the destination IP?
2. What protocol is being used?
3. What destination port is being used?
4. Which route matches the destination?
5. What is the route target?
6. Does the Security Group permit the traffic?
7. Does the NACL permit both directions?
8. What does the Flow Log action show?
9. Is IAM involved?
10. Is an endpoint policy involved?
11. Is the application/service itself healthy?

---

# 🆚 NAT Gateway vs VPC Endpoint

| Feature                             | NAT Gateway                        | VPC Endpoint                                           |
| ----------------------------------- | ---------------------------------- | ------------------------------------------------------ |
| Purpose                             | General outbound IPv4 connectivity | Private access to supported services                   |
| Internet Gateway involved           | Yes for public NAT Internet access | No for endpoint service traffic                        |
| Public Internet required            | For Internet destinations          | No                                                     |
| Works with arbitrary Internet sites | Yes                                | No                                                     |
| AWS service-specific                | No                                 | Yes                                                    |
| Route table involved                | Yes                                | Gateway Endpoint: Yes                                  |
| Security Groups                     | NAT Gateway does not use SGs       | Interface Endpoint uses SGs, Gateway Endpoint does not |
| Typical example                     | Linux package downloads            | Private S3 access                                      |

---

# 🧠 Architecture Decision

Suppose a private server requires:

```text
Amazon S3
Amazon DynamoDB
External GitHub API
Linux package repositories
```

A possible design is:

```text
S3
 │
 ▼
Gateway Endpoint

DynamoDB
 │
 ▼
Gateway Endpoint

External APIs
 │
 ▼
NAT Gateway

Package Repositories
 │
 ▼
NAT Gateway
```

The goal is not:

> Replace every NAT Gateway with a VPC Endpoint.

The correct question is:

> What destinations does the workload actually need to reach?

Then choose the appropriate connectivity mechanism.

---

# ❓ Interview Questions

### Q1. What problem does a VPC Endpoint solve?

**Answer**

It allows supported services/resources to be accessed privately from a VPC without requiring the traffic to use an Internet Gateway, NAT Gateway, public IP address, or public Internet path.

---

### Q2. Which AWS services support Gateway Endpoints?

**Answer**

Amazon S3 and Amazon DynamoDB.

---

### Q3. Does a Gateway Endpoint create an ENI in the subnet?

**Answer**

No.

Gateway Endpoints are route-table based.

Interface Endpoints use ENIs.

---

### Q4. Does an S3 Gateway Endpoint use a Security Group?

**Answer**

No.

Gateway Endpoints do not use Security Groups.

Interface Endpoints do.

---

### Q5. Why can S3 continue working after removing the NAT default route?

**Answer**

Because S3 traffic matches the more specific S3 prefix-list route pointing to the Gateway Endpoint.

---

### Q6. What does `REJECT` in VPC Flow Logs mean?

**Answer**

The recorded traffic was rejected by applicable VPC network controls, commonly because of Security Group or NACL rules.

---

### Q7. Are VPC Flow Logs packet captures?

**Answer**

No.

They capture network flow metadata rather than complete packet payloads.

---

### Q8. Can VPC Endpoints completely replace NAT Gateway?

**Answer**

Not generally.

Endpoints provide private access only for supported endpoint services. Workloads that need arbitrary outbound IPv4 Internet connectivity may still require NAT or another egress architecture.

---

# 🏁 Completion Criteria

Do not mark this lab complete until you can:

* [ ] Explain why VPC Endpoints exist
* [ ] Create an S3 Gateway Endpoint
* [ ] Identify the S3 prefix-list route
* [ ] Explain longest prefix matching
* [ ] Access S3 from private EC2
* [ ] Prove S3 works without the NAT default route
* [ ] Explain why general Internet access fails without NAT
* [ ] Explain endpoint policies
* [ ] Explain the difference between IAM and network connectivity
* [ ] Generate ACCEPT traffic
* [ ] Generate REJECT traffic
* [ ] Find the traffic in VPC Flow Logs
* [ ] Explain the important Flow Log fields
* [ ] Troubleshoot a deliberately broken connection
* [ ] Explain NAT Gateway vs VPC Endpoint

---

# 💰 Cost Awareness

Monitor resources such as:

* NAT Gateway
* EC2
* CloudWatch Logs
* Data transfer

Gateway Endpoints for S3 and DynamoDB do not have the hourly endpoint pricing associated with Interface Endpoints.

Always review current AWS pricing before using services in production.

---

# 🧹 Cleanup

If Lab 03 will be performed immediately after this lab, the original VPC can be retained.

Otherwise:

* [ ] Terminate test EC2 instances
* [ ] Delete temporary S3 objects/bucket if no longer needed
* [ ] Delete VPC Endpoint
* [ ] Delete NAT Gateway
* [ ] Release unused Elastic IP
* [ ] Remove VPC Flow Logs if no longer required
* [ ] Delete unnecessary CloudWatch Log Groups
* [ ] Delete remaining lab networking resources

---

# 💡 Key Takeaways

* Private workloads do not always need NAT Gateway to reach AWS services.
* Gateway Endpoints provide private connectivity to S3 and DynamoDB.
* Gateway Endpoints are implemented through route tables.
* AWS-managed prefix lists identify service network ranges.
* More-specific routes take precedence over default routes.
* NAT Gateway remains useful for general outbound IPv4 Internet connectivity.
* VPC Flow Logs provide network metadata useful for troubleshooting.
* Network troubleshooting should follow the traffic path rather than relying on random configuration changes.
* Connectivity, IAM authorization, endpoint policies, and resource policies are separate concerns.

> The architecture should be based on where the workload needs to communicate, not simply on which AWS networking services are available.
