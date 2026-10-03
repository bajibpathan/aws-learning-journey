# 🧪 Lab: Deploy an Application Load Balancer Across Multiple Availability Zones

> Build a highly available web tier using two EC2 instances across two Availability Zones and distribute HTTP traffic using an internet-facing Application Load Balancer.

---

# 🎯 Lab Objective

In this lab, we will deploy:

- Two EC2 web servers
- Two Availability Zones
- Private web subnets
- A NAT Gateway for outbound internet access
- An Application Load Balancer
- A target group
- HTTP health checks

We will then:

1. Access the application through the ALB DNS name.
2. Verify traffic reaches both web servers.
3. Stop one EC2 instance.
4. Observe the ALB sending traffic only to the healthy server.
5. Restart the instance and verify it returns to service.
6. Clean up chargeable resources.

---

# 🏗️ Architecture

```text
                         Internet
                            │
                            ▼
              ┌─────────────────────────┐
              │ Application Load        │
              │ Balancer                │
              │ Internet-Facing         │
              └────────────┬────────────┘
                           │
                     HTTP : Port 80
                           │
                    Target Group
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
      Availability Zone A         Availability Zone B
             │                           │
      Private Web Subnet          Private Web Subnet
             │                           │
        Web Server 01               Web Server 02
          Apache                      Apache
             │                           │
             └─────────────┬─────────────┘
                           │
                    Outbound Access
                           │
                           ▼
                      NAT Gateway
                           │
                           ▼
                        Internet
```

The ALB itself is configured using the two **public subnets**, while the EC2 web servers are deployed into the **web/private subnets**. The application instances therefore do not require public IP addresses.

---

# 📋 Prerequisites

Before starting, the course environment already has:

- VPC
- Two public subnets
- Two web/application subnets
- Database subnets
- Route tables
- Internet connectivity for the public tier
- Load Balancer Security Group
- Web Application Security Group
- Database Security Group

The previous NAT Gateway was removed to avoid unnecessary cost, so this lab recreates it temporarily.

You will also need the web-server user data script supplied with the course.

---

# 🗺️ Lab Phases

```text
Phase 1   Restore NAT Gateway
    │
    ▼
Phase 2   Deploy Web Server 01
    │
    ▼
Phase 3   Deploy Web Server 02
    │
    ▼
Phase 4   Create Target Group
    │
    ▼
Phase 5   Create Application Load Balancer
    │
    ▼
Phase 6   Test Traffic Distribution
    │
    ▼
Phase 7   Test Instance Failure
    │
    ▼
Phase 8   Restore Instance
    │
    ▼
Phase 9   Clean Up
```

---

# Phase 1: Restore the NAT Gateway

The web servers are deployed into private web subnets and require outbound internet connectivity to install the Apache packages from the user data script.

## Step 1: Create NAT Gateway

Navigate to:

**VPC → NAT Gateways → Create NAT Gateway**

Configure:

| Setting | Value |
|---|---|
| Subnet | Public Subnet 01 |
| Connectivity | Public |
| Elastic IP | Allocate new Elastic IP |

Create the NAT Gateway.

Wait until its status becomes:

```text
Available
```

A public NAT Gateway must be placed in a public subnet and requires an Elastic IP address.

---

# Phase 2: Update the Private Route Table

Navigate to:

**VPC → Route Tables**

Select the route table associated with the web/private subnets.

Choose:

**Routes → Edit routes**

Add:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | NAT Gateway |

Save the changes.

The traffic path is now:

```text
Private Web Subnet
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
Internet
```

---

# Phase 3: Prepare the Web Server User Data

The course script performs several tasks:

- Updates the operating system.
- Installs Apache.
- Configures the web server.
- Creates an `index.html` page.
- Creates a health-check page.

Each server should display slightly different content so we can identify which server handled a request.

For example:

```text
Web Server 01
Availability Zone A
```

and:

```text
Web Server 02
Availability Zone B
```

The course script also uses different background colours to make the servers visually easy to distinguish.

Most importantly, the script creates:

```text
/health.html
```

which will later be used by the ALB target group health check.

---

# Phase 4: Deploy Web Server 01

Navigate to:

**EC2 → Instances → Launch Instance**

Configure the first instance.

| Setting | Configuration |
|---|---|
| Name | `web-server-01` |
| AMI | Amazon Linux |
| Instance Type | `t2.micro` |
| Key Pair | Proceed without key pair |
| VPC | Existing lab VPC |
| Subnet | Web Subnet 01 |
| Public IP | Disabled |
| Security Group | Web Application Security Group |

Under:

**Advanced details → User data**

paste the script configured for Web Server 01.

Make sure the page identifies the server correctly.

For example:

```text
Web Server 01
Availability Zone A
```

Launch the instance.

---

# Phase 5: Deploy Web Server 02

Repeat the same process.

| Setting | Configuration |
|---|---|
| Name | `web-server-02` |
| AMI | Amazon Linux |
| Instance Type | `t2.micro` |
| Key Pair | Proceed without key pair |
| VPC | Existing lab VPC |
| Subnet | Web Subnet 02 |
| Public IP | Disabled |
| Security Group | Web Application Security Group |

Update the user data so this server identifies itself differently.

For example:

```text
Web Server 02
Availability Zone B
```

Launch the instance.

---

# ✅ Validation Checkpoint 1

Navigate to:

**EC2 → Instances**

Confirm both instances are:

```text
Running
```

and have passed their EC2 status checks.

Expected:

| Instance | State | Status Checks |
|---|---|---|
| web-server-01 | Running | Passed |
| web-server-02 | Running | Passed |

Do not continue until both instances are operating normally.

---

# Phase 6: Create the Target Group

Navigate to:

**EC2 → Target Groups → Create target group**

Choose:

```text
Instances
```

as the target type.

Configure:

| Setting | Value |
|---|---|
| Target Type | Instances |
| Protocol | HTTP |
| Port | 80 |
| VPC | Lab VPC |
| Protocol Version | HTTP/1 |

---

# ❤️ Configure Health Checks

Set the health-check path to:

```text
/health.html
```

The health-check file exists because it was created by the EC2 user data script.

For this lab, the course reduces the health-check interval to speed up testing.

Configure:

| Setting | Value |
|---|---|
| Health Check Protocol | HTTP |
| Health Check Path | `/health.html` |
| Interval | 10 seconds |
| Success Code | `200` |

Continue to the target-registration step.

---

# Register the EC2 Instances

Select:

```text
web-server-01

web-server-02
```

and register both instances with the target group.

Create the target group.

At this stage, the target group may initially show the targets as unused because the load balancer has not yet been connected.

The target group and health-check configuration are central to the lab because the ALB should send traffic only to targets considered healthy.

---

# Phase 7: Create the Application Load Balancer

Navigate to:

**EC2 → Load Balancers → Create Load Balancer**

Choose:

```text
Application Load Balancer
```

Configure:

| Setting | Value |
|---|---|
| Scheme | Internet-facing |
| IP Address Type | IPv4 |
| VPC | Lab VPC |

---

# 🌐 Configure Network Mapping

Select both Availability Zones used by the lab.

For each Availability Zone, select its corresponding **public subnet**.

Conceptually:

```text
                    Internet
                       │
                       ▼
             Internet-Facing ALB
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Public Subnet 01     Public Subnet 02
             │                   │
             ▼                   ▼
       Web Subnet 01        Web Subnet 02
             │                   │
             ▼                   ▼
       Web Server 01        Web Server 02
```

AWS manages the underlying load-balancer nodes associated with the selected subnets.

---

# 🔐 Configure Security Group

Attach the existing:

```text
Load Balancer Security Group
```

The load balancer must be allowed to receive HTTP traffic.

The web application security group should allow HTTP traffic from the load balancer according to the security-group architecture created earlier in the course.

---

# 🎧 Configure Listener

Create an HTTP listener:

| Setting | Value |
|---|---|
| Protocol | HTTP |
| Port | 80 |
| Default Action | Forward |
| Target | Previously created Target Group |

The request flow is now:

```text
Client
   │
   │ HTTP : 80
   ▼
Application Load Balancer
   │
   ▼
Target Group
   │
   ├── Web Server 01
   └── Web Server 02
```

Create the Application Load Balancer and wait until its state becomes:

```text
Active
```

---

# Phase 8: Validate Target Health

Navigate to:

**EC2 → Target Groups → Your Target Group → Targets**

Verify:

```text
Web Server 01 → Healthy

Web Server 02 → Healthy
```

Expected:

| Target | Health |
|---|---|
| Web Server 01 | 🟢 Healthy |
| Web Server 02 | 🟢 Healthy |

If either target is unhealthy, do not continue until the problem is resolved.

---

# Phase 9: Test the Application Load Balancer

Navigate to:

**EC2 → Load Balancers → Your ALB**

Copy the:

```text
DNS Name
```

Open it in a browser.

You should see one of the web servers.

For example:

```text
Web Server 01
Availability Zone A
```

Refresh the browser.

You should eventually see:

```text
Web Server 02
Availability Zone B
```

Continue refreshing.

The course demonstrates requests reaching both servers, showing that the load balancer is distributing requests across the registered targets.

---

# 🧪 Phase 10: Failure Test

Now test one of the most important ELB capabilities:

```text
What happens when a target fails?
```

Navigate to:

**EC2 → Instances**

Select:

```text
web-server-02
```

Choose:

**Instance state → Stop instance**

Wait for the instance to stop.

---

# ❤️ Observe Target Health

Return to:

**EC2 → Target Groups → Targets**

The stopped instance should no longer be available as a healthy target.

Conceptually:

```text
Target Group

Web Server 01   🟢 Healthy
Web Server 02   🔴 Unavailable
```

The ALB should therefore stop routing traffic to Web Server 02.

---

# 🌐 Test the Application Again

Return to the ALB DNS name and refresh repeatedly.

Before failure:

```text
Request
   │
   ├──► Web Server 01
   │
   └──► Web Server 02
```

After Web Server 02 stops:

```text
Request
   │
   └──► Web Server 01
```

You should now remain on Web Server 01 because the other server cannot receive traffic.

This demonstrates the relationship between:

```text
Health Check
     │
     ▼
Target Health
     │
     ▼
Traffic Routing
```

The course validates this by stopping the second instance and observing that requests remain on the healthy server.

---

# Phase 11: Restore the Failed Instance

Return to:

**EC2 → Instances**

Select Web Server 02 and choose:

**Instance state → Start instance**

Wait until its EC2 status checks pass.

Then return to the target group.

Eventually you should see:

```text
Web Server 01 → Healthy

Web Server 02 → Healthy
```

Refresh the ALB webpage again.

Traffic should once again reach both servers.

This demonstrates automatic recovery from the load balancer's perspective:

```text
Target Fails
    │
    ▼
Health Check Fails
    │
    ▼
Remove from Traffic
    │
    ▼
Target Recovers
    │
    ▼
Health Check Passes
    │
    ▼
Receive Traffic Again
```

---

# 🔧 Troubleshooting

## Problem 1: Targets Show Unhealthy

Check:

- Is Apache running?
- Was the user data script executed successfully?
- Does `/health.html` exist?
- Is the health-check path exactly `/health.html`?
- Is the web server security group allowing HTTP traffic from the load balancer?
- Are the instances running?

---

## Problem 2: Web Server Cannot Install Apache

Check the outbound path:

```text
Private Web Subnet
        │
        ▼
Route Table
        │
        ▼
NAT Gateway
        │
        ▼
Internet Gateway
```

Verify that the private route table contains:

```text
0.0.0.0/0 → NAT Gateway
```

Also verify that the NAT Gateway is in a public subnet and is available.

---

## Problem 3: ALB DNS Name Does Not Load

Check:

- ALB state is `Active`.
- Listener exists on port 80.
- Listener forwards to the correct target group.
- ALB security group allows HTTP.
- Target group contains the correct EC2 instances.
- Targets are healthy.

---

## Problem 4: Both Servers Show the Same Page

Verify that you changed the user data content for each server.

The pages should clearly identify:

```text
Web Server 01
```

and:

```text
Web Server 02
```

Otherwise, you cannot easily observe traffic distribution.

---

# 💰 Cost Awareness

This lab creates resources that can generate charges.

Pay particular attention to:

```text
NAT Gateway

Application Load Balancer

Elastic IP

EC2 Instances
```

The course specifically removes these resources at the end to avoid unnecessary ongoing costs.

---

# 🧹 Cleanup

Do not skip cleanup.

## Step 1: Terminate EC2 Instances

Navigate to:

**EC2 → Instances**

Select:

- Web Server 01
- Web Server 02

Choose:

**Instance state → Terminate instance**

Wait for termination.

---

## Step 2: Delete Application Load Balancer

Navigate to:

**EC2 → Load Balancers**

Select the ALB and delete it.

---

## Step 3: Delete Target Group

Navigate to:

**EC2 → Target Groups**

Delete the target group created for this lab.

---

## Step 4: Delete NAT Gateway

Navigate to:

**VPC → NAT Gateways**

Delete the NAT Gateway.

Wait until deletion completes.

---

## Step 5: Release Elastic IP

Navigate to:

**VPC → Elastic IP Addresses**

Select the Elastic IP used by the NAT Gateway and release it.

---

## Step 6: Clean Up Route Table

Return to:

**VPC → Route Tables**

Select the route table used by the web subnets.

Remove the route that previously pointed to the deleted NAT Gateway.

Save the changes.

---

# ✅ Cleanup Checklist

Before finishing, confirm:

```text
[ ] Web Server 01 terminated

[ ] Web Server 02 terminated

[ ] Application Load Balancer deleted

[ ] Target Group deleted

[ ] NAT Gateway deleted

[ ] Elastic IP released

[ ] NAT Gateway route removed
```

---

# 🎯 Lab Completion Criteria

The lab is successfully completed when you have demonstrated all of the following:

```text
[✓] Two EC2 web servers deployed

[✓] Servers distributed across two Availability Zones

[✓] Servers do not require public IP addresses

[✓] Target group created

[✓] /health.html configured for health checks

[✓] Both instances become healthy

[✓] Internet-facing ALB created

[✓] HTTP listener forwards to target group

[✓] ALB DNS name successfully loads application

[✓] Requests reach both web servers

[✓] Stopping one instance removes it from traffic

[✓] Healthy instance continues serving requests

[✓] Restarted instance returns to service

[✓] Chargeable lab resources cleaned up
```

---

# ❓ Interview Questions

### Q1. Why deploy the EC2 instances across two Availability Zones?

To improve application availability. If one instance or Availability Zone experiences a failure, traffic can continue to healthy resources in another Availability Zone.

### Q2. Why don't the EC2 instances need public IP addresses?

Clients access the application through the internet-facing Application Load Balancer. The EC2 instances themselves can remain in private subnets.

### Q3. Why is the ALB associated with public subnets?

The lab creates an internet-facing ALB, so its load-balancer nodes are associated with the public-facing network tier.

### Q4. What is the purpose of the target group?

The target group contains the backend resources that receive traffic from the load balancer.

### Q5. Why do we configure `/health.html`?

The ALB uses this endpoint to determine whether each web server is healthy enough to receive traffic.

### Q6. What happens when an EC2 instance fails?

After the target is no longer considered healthy, the load balancer stops sending requests to it and continues routing traffic to healthy targets.

### Q7. What happens when the failed instance recovers?

Once it passes its health checks again, it can resume receiving traffic.

### Q8. Why was the content different on the two web servers?

It makes it easy to visually identify which backend instance handled each request.

### Q9. What does the ALB listener do?

The listener accepts incoming traffic on a configured protocol and port and forwards it according to its routing configuration.

For this lab:

```text
HTTP : 80
    │
    ▼
Target Group
```

### Q10. Why was a NAT Gateway required?

The private EC2 instances required outbound internet connectivity so the user data script could update the operating system and install Apache.

### Q11. Why doesn't the NAT Gateway make the private EC2 instances publicly accessible?

The NAT Gateway provides outbound connectivity for resources in the private subnet. It does not give those EC2 instances public IP addresses or make them directly reachable from the internet.

### Q12. Why use a Load Balancer Security Group and a Web Application Security Group separately?

It allows the architecture to control traffic between layers.

Conceptually:

```text
Internet
   │
   │ HTTP
   ▼
ALB Security Group
   │
   │ HTTP
   ▼
Web Application Security Group
   │
   ▼
EC2
```

The web tier can therefore accept application traffic from the load-balancer tier instead of being directly exposed to the internet.

---

# 💡 Key Takeaways

- A single EC2 instance creates a potential availability problem.
- Multiple EC2 instances should be distributed across Availability Zones.
- An Application Load Balancer can distribute HTTP traffic across those instances.
- Backend application instances can remain in private subnets.
- The internet-facing ALB provides the public application entry point.
- Target groups contain the resources receiving ALB traffic.
- Health checks determine whether a target should receive requests.
- An unhealthy or stopped target is removed from traffic.
- A recovered target can automatically return to service after passing health checks.
- Different content on each server makes traffic distribution easy to observe.
- Private instances can use a NAT Gateway for outbound internet connectivity.
- NAT Gateways, Elastic Load Balancers, and related resources should be removed when the lab is finished to avoid unnecessary costs.

The main architecture to remember from this lab is:

```text
                  INTERNET
                      │
                      ▼
          APPLICATION LOAD BALANCER
                      │
                Target Group
                      │
             Health Checks
                      │
           ┌──────────┴──────────┐
           ▼                     ▼
     WEB SERVER 01         WEB SERVER 02
          AZ-A                  AZ-B
           │                     │
           └──────────┬──────────┘
                      │
                Private Subnets
```

And the key behavior demonstrated is:

```text
Both Healthy
    │
    ▼
Traffic → Server 01 + Server 02


Server 02 Fails
    │
    ▼
Health Check Fails
    │
    ▼
Traffic → Server 01 Only


Server 02 Recovers
    │
    ▼
Health Check Passes
    │
    ▼
Traffic → Server 01 + Server 02
```