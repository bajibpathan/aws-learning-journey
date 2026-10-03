# 🧪 Lab: EC2 Auto Scaling with Application Load Balancer

> Build an EC2 Auto Scaling Group across two Availability Zones, integrate it with an Application Load Balancer, test automatic instance replacement, and trigger scale-out and scale-in using a target tracking scaling policy.

---

# 🎯 Lab Objectives

In this lab, we will:

1. Create a target group.
2. Create an internet-facing Application Load Balancer.
3. Recreate the NAT Gateway for private-subnet internet access.
4. Create an IAM role for Systems Manager Session Manager.
5. Create an EC2 launch template.
6. Create an Auto Scaling Group across two Availability Zones.
7. Configure minimum, desired, and maximum capacity.
8. Attach the Auto Scaling Group to the load balancer.
9. Enable Elastic Load Balancing health checks.
10. Configure a target tracking scaling policy using average CPU utilization.
11. Verify traffic distribution across the EC2 instances.
12. Test automatic replacement of a terminated instance.
13. Simulate an application health-check failure.
14. Generate CPU load to trigger scale-out.
15. Stop the CPU load and observe scale-in.
16. Clean up all chargeable resources.

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
                                 ▼
                           Target Group
                         /health.html
                                 │
                  ┌──────────────┴──────────────┐
                  │                             │
                  ▼                             ▼
          Availability Zone A          Availability Zone B
                  │                             │
          Private Web Subnet           Private Web Subnet
                  │                             │
                EC2                           EC2
                  └──────────────┬──────────────┘
                                 │
                                 ▼
                       Auto Scaling Group
                                 │
                       Minimum: 2
                       Desired: 2
                       Maximum: 4
                                 │
                                 ▼
                     Target Tracking Policy
                                 │
                       Average CPU = 40%
                                 │
                                 ▼
                            CloudWatch
```

The Auto Scaling Group starts with two EC2 instances distributed across the two web subnets and can scale out to a maximum of four instances.

---

# 📋 Prerequisites

This lab assumes the previous networking and load-balancing exercises have already created:

- VPC
- Two public subnets
- Two private web subnets
- Route tables
- Internet Gateway
- Load Balancer Security Group
- Web Application/Auto Scaling Security Group

You will also need the course-provided user data script.

The script:

- Installs Apache.
- Configures the web server.
- Creates the application page.
- Retrieves Availability Zone information from instance metadata.
- Displays that information on the web page.
- Creates the `health.html` file used by the load balancer health check.

---

# 🗺️ Lab Workflow

```text
Create Target Group
        │
        ▼
Create Application Load Balancer
        │
        ▼
Create NAT Gateway
        │
        ▼
Update Private Route Table
        │
        ▼
Create Session Manager IAM Role
        │
        ▼
Create Launch Template
        │
        ▼
Create Auto Scaling Group
        │
        ▼
Configure Target Tracking
        │
        ▼
Validate Load Balancing
        │
        ▼
Test Instance Replacement
        │
        ▼
Test Application Health Failure
        │
        ▼
Generate CPU Load
        │
        ▼
Observe Scale Out
        │
        ▼
Stop CPU Load
        │
        ▼
Observe Scale In
        │
        ▼
Cleanup
```

---

# Phase 1: Create the Target Group

Navigate to:

**EC2 → Target Groups → Create target group**

Configure:

| Setting | Value |
|---|---|
| Target Type | Instances |
| Protocol | HTTP |
| Port | 80 |
| VPC | Lab VPC |

## Configure Health Check

Use:

```text
/health.html
```

The user data script will create this file when Auto Scaling launches the EC2 instances.

The lab also reduces the health-check interval and configures:

- Healthy threshold: `3`
- Unhealthy threshold: `2`

This means multiple successful or failed checks are required before changing the health status.

At this point, do **not** register any EC2 instances.

There are no instances yet because the Auto Scaling Group will create them later.

Create the target group.

---

# Phase 2: Create the Application Load Balancer

Navigate to:

**EC2 → Load Balancers → Create Load Balancer**

Choose:

**Application Load Balancer**

Configure:

| Setting | Value |
|---|---|
| Scheme | Internet-facing |
| IP Address Type | IPv4 |
| VPC | Lab VPC |
| Availability Zones | Both lab AZs |
| Subnets | Public Subnet 01 and Public Subnet 02 |
| Security Group | Load Balancer Security Group |
| Listener | HTTP : 80 |
| Default Action | Forward to target group |

The architecture at this stage is:

```text
Internet
    │
    ▼
Application Load Balancer
    │
    ▼
Target Group
    │
    ▼
No Targets Yet
```

Create the load balancer and wait until its state becomes:

```text
Active
```

---

# Phase 3: Create the NAT Gateway

The EC2 instances will be deployed into private subnets.

The lab therefore recreates the NAT Gateway to provide the required outbound connectivity.

Navigate to:

**VPC → NAT Gateways → Create NAT Gateway**

Configure:

| Setting | Value |
|---|---|
| Subnet | Public Subnet 01 |
| Connectivity | Public |
| Elastic IP | Allocate new Elastic IP |

Create the NAT Gateway.

Wait until:

```text
State = Available
```

---

# Phase 4: Update the Private Route Table

Navigate to:

**VPC → Route Tables**

Select the route table associated with the private web subnets.

Choose:

**Routes → Edit routes**

Add:

| Destination | Target |
|---|---|
| `0.0.0.0/0` | NAT Gateway |

Save the changes.

The outbound path is now:

```text
Private EC2
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

# Phase 5: Create IAM Role for Session Manager

The EC2 instances will have private IP addresses and the lab uses **AWS Systems Manager Session Manager** instead of a bastion host or SSH connection.

Navigate to:

**IAM → Roles → Create role**

Select:

```text
Trusted Entity: AWS Service

Service: EC2
```

Attach the Systems Manager managed-instance permission shown in the course.

The transcript identifies:

```text
AmazonSSMManagedInstanceCore
```

as the permission required for the EC2 instances to communicate with Systems Manager.

Give the role a name such as:

```text
EC2-IAM-Master-Role
```

Create the role.

The lab notes that the Amazon Linux 2023 AMI being used already includes the Systems Manager agent.

---

# Phase 6: Create the Launch Template

Navigate to:

**EC2 → Launch Templates → Create launch template**

The launch template defines how Auto Scaling creates each EC2 instance.

Configure:

| Setting | Configuration |
|---|---|
| AMI | Amazon Linux 2023 |
| Instance Type | `t2.micro` |
| Key Pair | None |
| Security Group | Web Server / ASG Security Group |
| Root Volume | 8 GB |
| IAM Instance Profile | EC2 IAM role created earlier |
| User Data | Course web-server script |

Do not configure the subnets in the launch template.

The subnets will be selected when creating the Auto Scaling Group.

---

# 👤 Add the IAM Instance Profile

Under:

**Advanced details → IAM instance profile**

select:

```text
EC2-IAM-Master-Role
```

This allows the instances to communicate with Systems Manager so they can be accessed through Session Manager.

---

# 📜 Add User Data

Under:

**Advanced details → User data**

paste the course-provided script.

The script should:

```text
Launch EC2
    │
    ▼
Install Apache
    │
    ▼
Configure Web Server
    │
    ▼
Read Instance Metadata
    │
    ▼
Identify Availability Zone
    │
    ▼
Generate index.html
    │
    └── Generate health.html
```

Create the launch template.

---

# Phase 7: Create the Auto Scaling Group

Navigate to:

**EC2 → Auto Scaling Groups → Create Auto Scaling Group**

Select the launch template created in the previous phase.

Configure the VPC and select both private web subnets:

```text
Web Subnet 01

Web Subnet 02
```

This allows the Auto Scaling Group to distribute EC2 instances across both Availability Zones.

---

# Phase 8: Attach the Existing Load Balancer

During Auto Scaling Group creation, configure load balancing.

Choose the existing target group created earlier.

Conceptually:

```text
Auto Scaling Group
       │
       ▼
Launch EC2
       │
       ▼
Register with Target Group
       │
       ▼
Application Load Balancer
       │
       ▼
Receive Traffic
```

---

# ❤️ Enable ELB Health Checks

Turn on:

```text
Elastic Load Balancing Health Checks
```

The load balancer will use:

```text
/health.html
```

to determine whether the application on each instance is responding correctly.

---

# Phase 9: Configure Capacity

Configure:

| Capacity | Value |
|---|---:|
| Minimum | 2 |
| Desired | 2 |
| Maximum | 4 |

Therefore:

```text
Minimum       Desired                 Maximum
   │             │                       │
   ▼             ▼                       ▼
   2 ─────────── 2 ─────── 3 ────────── 4
```

The Auto Scaling Group should initially create two EC2 instances.

It can later scale out to a maximum of four.

---

# Phase 10: Configure Target Tracking Scaling

Configure a target tracking scaling policy.

Use:

```text
Metric:
Average CPU Utilization

Target:
40%
```

Conceptually:

```text
Average CPU
    │
    ▼
Target = 40%
    │
    ├── CPU rises above target
    │       │
    │       ▼
    │    Scale Out
    │
    └── CPU falls sufficiently
            │
            ▼
         Scale In
```

The lab uses 40% as the target value for demonstrating target tracking.

---

# Phase 11: Add Tags

Add a `Name` tag so that Auto Scaling-created instances can be easily identified.

For example:

```text
Name = RR Auto Scaling Srvs
```

Review the configuration and create the Auto Scaling Group.

---

# Phase 12: Verify EC2 Deployment

Navigate to:

**EC2 → Instances**

Auto Scaling should begin creating two EC2 instances automatically.

Wait until both instances are:

```text
Running
```

and their status checks have passed.

Expected:

| Instance | Subnet/AZ | State |
|---|---|---|
| EC2 01 | AZ-A | Running |
| EC2 02 | AZ-B | Running |

The instances should have private IP addresses.

---

# Phase 13: Verify Target Health

Navigate to:

**EC2 → Target Groups → Targets**

Verify both instances become:

```text
Healthy
```

Expected:

```text
Target Group

EC2 01     🟢 Healthy
EC2 02     🟢 Healthy
```

---

# Phase 14: Test the Application Load Balancer

Navigate to:

**EC2 → Load Balancers**

Copy the ALB DNS name and open it in a browser.

Refresh the page several times.

Because the user data displays Availability Zone information, you should see requests being handled by instances in different Availability Zones.

```text
Request
   │
   ▼
Application Load Balancer
   │
   ├──► EC2 in AZ-A
   │
   └──► EC2 in AZ-B
```

This confirms:

- Auto Scaling created the instances.
- The instances registered with the target group.
- The health checks are passing.
- The ALB is distributing traffic.

The transcript demonstrates browser refreshes alternating between the two Availability Zones.

---

# 🧪 Phase 15: Test Automatic Instance Replacement

Now test whether Auto Scaling maintains the desired capacity.

Navigate to:

**EC2 → Instances**

Select one of the Auto Scaling-created instances.

Choose:

**Instance state → Terminate instance**

Initially:

```text
Desired Capacity = 2

EC2 01     Running
EC2 02     Running
```

After termination:

```text
EC2 01     Terminated
EC2 02     Running
```

But the desired capacity is still:

```text
2
```

Auto Scaling detects the difference.

```text
Actual Capacity = 1

Desired Capacity = 2
        │
        ▼
Launch Replacement
        │
        ▼
Actual Capacity = 2
```

---

# 🔍 Verify Auto Scaling Activity

Navigate to:

**EC2 → Auto Scaling Groups → Your ASG → Activity**

The activity history should show that an instance was removed and another instance launched.

Return to:

**EC2 → Instances**

Verify a replacement instance has appeared.

The lab demonstrates Auto Scaling restoring the fleet after an instance is manually terminated.

---

# 🧪 Phase 16: Test Application Health Failure

The previous test terminated the entire EC2 instance.

Now test what happens when:

```text
EC2 is running
```

but:

```text
Application health check fails
```

---

# Connect Using Session Manager

Navigate to:

**EC2 → Instances**

Select one of the running instances.

Choose:

**Connect → Session Manager → Connect**

This provides shell access without using a bastion host or SSH key.

---

# Locate the Health File

Navigate to the Apache HTML directory.

Verify that the web files include:

```text
index.html

health.html
```

Delete:

```text
health.html
```

The instance itself remains running, but the ALB can no longer successfully request its configured health-check page.

---

# Observe the Failed Health Check

Navigate to:

**EC2 → Target Groups → Targets**

One target should eventually become:

```text
Unhealthy
```

The transcript shows the health check returning:

```text
404
```

because `/health.html` no longer exists.

The load balancer stops sending application traffic to that target.

```text
Before

ALB
 ├──► AZ-A
 └──► AZ-B


After Health Failure

ALB
 ├──► AZ-A
 └──X AZ-B
```

Refresh the application.

Traffic should now go only to the remaining healthy instance.

---

# 🔄 Observe Auto Scaling Replacement

Return to:

**Auto Scaling Group → Activity**

Because ELB health checks were enabled for the Auto Scaling Group, the unhealthy instance is replaced.

Conceptually:

```text
EC2 Running
    │
    ▼
health.html Missing
    │
    ▼
ELB Health Check Fails
    │
    ▼
Target Unhealthy
    │
    ▼
Auto Scaling
    │
    ▼
Terminate Instance
    │
    ▼
Launch Replacement
```

After the replacement becomes healthy, refresh the application again.

Traffic should once again reach instances across both Availability Zones.

---

# 🧪 Phase 17: Test Target Tracking Scale-Out

Now test the scaling policy itself.

Current capacity should return to:

```text
Desired = 2

Running = 2

Maximum = 4
```

We will deliberately increase CPU utilization.

---

# Connect to an EC2 Instance

Select one of the instances.

Choose:

**Connect → Session Manager**

Elevate privileges as shown in the course.

Install the stress-test utility used in the lab.

The transcript uses:

```bash
yum install stress -y
```

Then run the CPU stress test:

```bash
stress -c 4
```

This deliberately increases CPU utilization on the selected instance.

---

# Phase 18: Monitor CPU Utilization

Navigate to:

**Auto Scaling Group → Monitoring → EC2**

Watch average CPU utilization.

The target tracking policy is configured for:

```text
40%
```

The transcript demonstrates the average CPU increasing above this target.

Example from the lab:

```text
Average CPU ≈ 51%
```

---

# 📊 Observe CloudWatch Alarms

Navigate to:

**CloudWatch → Alarms**

The lab shows two automatically associated alarm conditions around the target-tracking policy.

The demonstrated high condition is:

```text
CPU > 40%

3 data points

within 3 minutes
```

Once the high alarm triggers:

```text
CPU > Target
     │
     ▼
CloudWatch Alarm
     │
     ▼
Auto Scaling
     │
     ▼
Increase Capacity
```

---

# Phase 19: Observe Scale-Out

Navigate to:

**Auto Scaling Group → Activity**

The lab first shows capacity changing from:

```text
2 → 3
```

A third EC2 instance is launched.

Return to:

**EC2 → Instances**

You should see the new instance initializing.

---

# 📈 Why Can It Scale Again?

The stress process continues running on the original server.

If average CPU across all three instances remains above the target, Auto Scaling can scale again.

```text
2 Instances
     │
     │ CPU > Target
     ▼
3 Instances
     │
     │ CPU Still > Target
     ▼
4 Instances
```

Because the configured maximum is:

```text
Maximum Capacity = 4
```

the group cannot scale beyond four instances.

The transcript demonstrates the group eventually reaching four running instances, with two instances in each Availability Zone.

---

# ✅ Validate Scale-Out

Expected state:

```text
Auto Scaling Group

AZ-A
├── EC2
└── EC2

AZ-B
├── EC2
└── EC2
```

Total:

```text
Running Instances = 4
```

This demonstrates:

```text
High CPU
    │
    ▼
Target Tracking
    │
    ▼
Scale Out
    │
    ▼
2 → 3 → 4
```

---

# 🧪 Phase 20: Stop the Stress Test

Return to the Session Manager session where the stress test is running.

Stop the process.

CPU utilization should begin falling.

```text
Stress Test Stops
       │
       ▼
CPU Utilization ↓
       │
       ▼
Average Fleet CPU ↓
```

---

# Phase 21: Observe Scale-In

Return to:

**CloudWatch → Alarms**

The transcript demonstrates the lower target-tracking alarm becoming active after average CPU remains below its lower threshold.

The lab shows:

```text
CPU < 28%

15 data points

within 15 minutes
```

The Auto Scaling Group then begins reducing capacity.

---

# 📉 Observe Capacity Reduction

Navigate to:

**Auto Scaling Group → Activity**

The group scales from:

```text
4
│
▼
3
│
▼
2
```

It stops at:

```text
Desired Capacity = 2
```

The resulting environment returns to:

```text
AZ-A                    AZ-B
 │                       │
EC2                     EC2
```

---

# 🎯 Target Tracking Behavior Demonstrated

The complete test demonstrates:

```text
Normal Load

2 Instances
     │
     ▼

CPU Increases Above Target
     │
     ▼

Scale Out

2 → 3 → 4
     │
     ▼

CPU Load Removed
     │
     ▼

CPU Drops
     │
     ▼

Scale In

4 → 3 → 2
```

The Auto Scaling Group remains within:

```text
Minimum = 2

Maximum = 4
```

while target tracking adjusts capacity according to CPU utilization.

---

# 🔧 Troubleshooting

## Problem 1: Instances Are Not Launching

Check:

- Launch template
- AMI
- Instance type
- Security group
- IAM instance profile
- Selected VPC
- Selected subnets
- Auto Scaling activity history

---

## Problem 2: Targets Are Unhealthy

Check:

- Apache is running.
- `health.html` exists.
- Target group health-check path is `/health.html`.
- Security groups permit the required HTTP traffic.
- User data completed successfully.

---

## Problem 3: Session Manager Connect Is Unavailable

Check:

- IAM instance profile is attached.
- Required Systems Manager permission is present.
- Systems Manager agent is available.
- Instance has the required connectivity to reach the Systems Manager service.

---

## Problem 4: Load Balancer Page Does Not Open

Check:

- ALB state is `Active`.
- Listener is configured on HTTP port 80.
- Listener forwards to the correct target group.
- ALB security group allows HTTP.
- Targets are healthy.

---

## Problem 5: Auto Scaling Does Not Replace the Failed Application Instance

Verify that:

```text
Elastic Load Balancing Health Checks
```

were enabled on the Auto Scaling Group.

---

## Problem 6: Scale-Out Does Not Occur Immediately

Scaling is not instantaneous.

The lab demonstrates that CloudWatch needs enough metric data to satisfy the configured alarm condition before the scaling activity begins.

Check:

**CloudWatch → Alarms**

and:

**Auto Scaling Group → Activity**

---

# 💰 Cost Awareness

This lab creates several potentially chargeable resources.

Pay particular attention to:

- NAT Gateway
- Elastic IP
- Application Load Balancer
- EC2 instances

The transcript explicitly cleans up these resources after completing the tests to avoid unnecessary charges.

---

# 🧹 Cleanup

Cleanup order matters.

Do **not** simply terminate the EC2 instances first.

If the Auto Scaling Group still exists and its desired capacity is two, it will launch replacement instances.

---

# Step 1: Delete the Auto Scaling Group

Navigate to:

**EC2 → Auto Scaling Groups**

Select the Auto Scaling Group.

Choose:

**Actions → Delete**

Confirm the deletion.

This removes the group and its Auto Scaling-managed instances.

---

# Step 2: Delete the Application Load Balancer

Navigate to:

**EC2 → Load Balancers**

Select the ALB and delete it.

---

# Step 3: Delete the Target Group

Navigate to:

**EC2 → Target Groups**

Delete the target group used by this lab.

---

# Step 4: Delete the Launch Template

Navigate to:

**EC2 → Launch Templates**

Select the lab launch template and delete it.

---

# Step 5: Delete the NAT Gateway

Navigate to:

**VPC → NAT Gateways**

Delete the NAT Gateway.

Wait for deletion to complete.

---

# Step 6: Release the Elastic IP

Navigate to:

**VPC → Elastic IP Addresses**

Release the Elastic IP associated with the NAT Gateway.

---

# Step 7: Remove the NAT Route

Navigate to:

**VPC → Route Tables**

Select the private/main route table used in the lab.

Remove:

```text
0.0.0.0/0 → Deleted NAT Gateway
```

Save the route table.

---

# ✅ Cleanup Checklist

```text
[ ] Auto Scaling Group deleted

[ ] Auto Scaling EC2 instances terminated

[ ] Application Load Balancer deleted

[ ] Target Group deleted

[ ] Launch Template deleted

[ ] NAT Gateway deleted

[ ] Elastic IP released

[ ] NAT Gateway route removed
```

---

# ✅ Lab Completion Criteria

You have successfully completed the lab when you can verify:

```text
[✓] Target Group created

[✓] Application Load Balancer created

[✓] NAT Gateway configured

[✓] Session Manager IAM role created

[✓] Launch Template created

[✓] Auto Scaling Group created

[✓] EC2 instances deployed into private subnets

[✓] Two Availability Zones used

[✓] Minimum capacity = 2

[✓] Desired capacity = 2

[✓] Maximum capacity = 4

[✓] Target tracking configured at 40% CPU

[✓] ALB distributes traffic between instances

[✓] Terminated EC2 instance is automatically replaced

[✓] Failed application health check causes replacement

[✓] CPU stress triggers scale-out

[✓] Capacity increases from 2 toward 4

[✓] Removing CPU stress triggers scale-in

[✓] Capacity returns to 2

[✓] All chargeable lab resources cleaned up
```

---

# ❓ Interview Questions

### Q1. What is the purpose of the launch template?

It defines how EC2 instances created by the Auto Scaling Group should be configured, including the AMI, instance type, security group, IAM instance profile, storage, and user data.

### Q2. Why did we not manually create the EC2 instances?

The purpose of the lab is to allow the Auto Scaling Group to create and manage the EC2 fleet automatically.

### Q3. Why are two private subnets selected for the Auto Scaling Group?

They allow instances to be distributed across two Availability Zones.

### Q4. What were the capacity settings?

```text
Minimum = 2
Desired = 2
Maximum = 4
```

### Q5. What is the target tracking value in this lab?

Average CPU utilization of:

```text
40%
```

### Q6. What happens when CPU utilization rises above the target?

The target tracking policy can cause Auto Scaling to increase capacity.

### Q7. Why doesn't Auto Scaling create more than four instances?

The maximum capacity is configured as four.

### Q8. What happens after CPU utilization decreases?

Auto Scaling can reduce the number of instances until the required capacity is restored.

### Q9. What happens if someone manually terminates an Auto Scaling-managed instance?

The Auto Scaling Group detects that actual capacity is below desired capacity and launches a replacement.

### Q10. What is the difference between the EC2 termination test and the `health.html` test?

The first test removes the entire EC2 instance.

The second keeps the EC2 instance running but deliberately causes the application health check to fail.

### Q11. Why enable ELB health checks in the Auto Scaling Group?

It allows application health information from the load balancer to contribute to determining whether an instance should remain in service.

### Q12. Why use Session Manager?

In this lab it provides shell access to the private EC2 instances without using a bastion host, opening SSH access, or using SSH keys.

### Q13. Why is a NAT Gateway required?

The EC2 instances reside in private subnets and need outbound connectivity for activities such as installing packages.

### Q14. What role does CloudWatch play?

CloudWatch monitors the CPU metric used by the target tracking scaling policy and its associated alarm conditions.

### Q15. Why should the Auto Scaling Group be deleted before manually terminating the instances during cleanup?

If the Auto Scaling Group still exists and its desired capacity is two, it can launch replacement instances.

### Q16. What is the relationship between ELB and Auto Scaling?

The load balancer distributes incoming traffic among healthy targets, while the Auto Scaling Group manages how many EC2 instances are running and replaces unhealthy instances.

---

# 💡 Key Takeaways

- A launch template defines how Auto Scaling-created EC2 instances are configured.
- An Auto Scaling Group can deploy instances across multiple Availability Zones.
- Minimum, desired, and maximum capacity define the fleet boundaries.
- In this lab, the group starts with two instances and can scale to four.
- Auto Scaling can automatically register instances with an existing load balancer target group.
- ELB health checks can detect application-level failures.
- Auto Scaling can replace an unhealthy instance to restore desired capacity.
- Manually terminating an Auto Scaling-managed instance does not permanently reduce the fleet because the group restores its desired capacity.
- Target tracking uses CloudWatch metrics to adjust capacity.
- The lab uses average CPU utilization of 40% as its target.
- Artificial CPU load demonstrates scale-out from two toward four instances.
- Removing the load demonstrates scale-in back toward the desired capacity.
- Session Manager allows the lab to access private EC2 instances without relying on a bastion host or SSH keys.
- Auto Scaling and Elastic Load Balancing work together to provide dynamic capacity and traffic distribution.
- Chargeable resources should be removed after completing the lab.

The most important workflow from this lab is:

```text
                         USERS
                           │
                           ▼
                APPLICATION LOAD BALANCER
                           │
                           ▼
                     TARGET GROUP
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
           EC2 AZ-A                    EC2 AZ-B
             │                           │
             └─────────────┬─────────────┘
                           │
                    AUTO SCALING GROUP
                           │
                 Min 2 / Desired 2 / Max 4
                           │
                           ▼
                 TARGET TRACKING POLICY
                           │
                     CPU Target 40%
                           │
                           ▼
                       CLOUDWATCH
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
           CPU High                   CPU Low
              │                         │
              ▼                         ▼
          SCALE OUT                  SCALE IN
           2 → 4                      4 → 2
```

The second behavior to remember is **self-healing**:

```text
Instance Failure
       │
       ▼
Health Check Fails
       │
       ▼
Instance Unhealthy
       │
       ▼
Auto Scaling Replaces Instance
       │
       ▼
Desired Capacity Restored
```