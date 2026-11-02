# Amazon EventBridge – Hands-On Lab: Monitoring EC2 Instance State Changes

> **Lab objective:** Build an event-driven monitoring workflow that detects Amazon EC2 instance state changes, invokes an AWS Lambda function, and records the captured events in Amazon CloudWatch Logs.

---

## 1. Lab Overview

In the previous lesson, we explored Amazon EventBridge and its core features:

- Event-driven architecture
- Event buses
- EventBridge rules
- Event patterns
- Event transformation
- Scheduled events
- EventBridge Pipes

In this hands-on lab, we will demonstrate how these concepts work using the AWS Management Console.

We will configure Amazon EventBridge to monitor an EC2 instance and capture two state changes:

- `running`
- `stopped`

When a matching state-change event occurs, EventBridge will invoke an AWS Lambda function.

The Lambda function will record the event information in Amazon CloudWatch Logs.

### Services Used

| AWS Service | Purpose |
|---|---|
| Amazon EC2 | Generates instance state-change events |
| Amazon EventBridge | Captures and routes matching events |
| AWS Lambda | Processes incoming events |
| Amazon CloudWatch Logs | Stores event information |

---

# 2. Architecture Overview

```text
                 Amazon EC2 Instance
                          |
                          |
                   State Changes
                          |
                  +-------+-------+
                  |               |
                  v               v
               Running         Stopped
                  |               |
                  +-------+-------+
                          |
                          v
                  Amazon EventBridge
                          |
                          v
                    Default Event Bus
                          |
                          v
                   EventBridge Rule
                          |
                          v
                      AWS Lambda
                          |
                          v
                   CloudWatch Logs
                          |
                          v
                  EC2 State Change Logs
```

### Event Processing Flow

1. An EC2 instance changes state.
2. Amazon EC2 generates a state-change event.
3. The event is received by the default EventBridge event bus.
4. An EventBridge rule evaluates the event.
5. If the state is `running` or `stopped`, the rule matches.
6. EventBridge invokes the Lambda function.
7. Lambda logs the event details.
8. CloudWatch Logs stores the output for investigation.

### Key Concept

**EC2 → EventBridge → Lambda → CloudWatch Logs**

This architecture demonstrates event-driven monitoring without requiring an application to continuously poll EC2 for state changes.

---

# 3. Lab Prerequisites

Before starting the lab, ensure that you have:

- An AWS account
- Access to the AWS Management Console
- Permissions to manage EC2, Lambda, EventBridge, and CloudWatch Logs
- An EC2 instance in the `running` state
- A supported Node.js runtime for Lambda

### Lab Environment

The instructor uses the following environment:

| Configuration | Value |
|---|---|
| AWS Account | Development account |
| AWS Region | US East (N. Virginia) |
| EC2 Instance Name | My Web Server |
| EC2 Operating System | Linux |
| EC2 Networking | Public subnet |
| Lambda Runtime | Node.js 22.x |
| Lambda Architecture | x86_64 |
| Lambda Timeout | 10 seconds |
| Event Bus | Default |
| Event Source | Amazon EC2 |
| Monitored States | Running and Stopped |

**Note:** The instance does not need a public IP address for EventBridge to capture EC2 state-change events. The public subnet and public IP are characteristics of the instructor's demonstration environment.

---

# 4. Step 1 – Verify the EC2 Instance

Navigate to:

**AWS Management Console → EC2 → Instances**

Locate the instance that will be monitored.

In the demonstration, the instance is named:

`My Web Server`

### Verify

Confirm that:

- The instance exists.
- The instance state is `Running`.
- The instance is in the Region where the lab will be configured.

### Expected Result

```text
Instance Name: My Web Server

Instance State: Running
```

The instance is now ready to generate state-change events.

### Screenshot

`01-ec2-instance-running.png`

Capture the EC2 instance list showing the instance in the running state.

---

# 5. Step 2 – Create an AWS Lambda Function

Navigate to:

**AWS Management Console → Lambda → Functions**

Select:

**Create function**

### Configuration

| Setting | Value |
|---|---|
| Creation Method | Author from scratch |
| Function Name | `ec2-instance-state-change-logs` |
| Runtime | Node.js 22.x |
| Architecture | x86_64 |

Select **Create function**.

### Expected Result

A new Lambda function is created.

The Lambda console displays the function's configuration and code editor.

### Screenshot

`02-lambda-function-created.png`

---

# 6. Step 3 – Configure the Lambda Function Code

The Lambda function receives the event generated by Amazon EC2 through EventBridge.

Its purpose is to log information about the state change.

The instructor uses JavaScript code provided in the course resources.

The transcript explains the code's purpose but does not include its complete contents.

The following is an **equivalent illustrative implementation**, not a verbatim copy of the instructor's code.

### Example Lambda Code

```javascript
export const handler = async (event) => {
    console.log("EC2 Instance State Change Captured");

    console.log("Event ID:", event.id);
    console.log("Event Source:", event.source);
    console.log("AWS Account:", event.account);
    console.log("AWS Region:", event.region);

    console.log(
        "Instance ID:",
        event.detail?.["instance-id"]
    );

    console.log(
        "Instance State:",
        event.detail?.state
    );

    return {
        statusCode: 200,
        body: JSON.stringify({
            message: "EC2 state change logged successfully"
        })
    };
};
```

### Code Explanation

| Code Component | Purpose |
|---|---|
| `handler` | Lambda entry point |
| `event` | Receives the EventBridge event |
| `event.id` | Identifies the event |
| `event.source` | Identifies the AWS service |
| `event.account` | AWS account where the event occurred |
| `event.region` | AWS Region associated with the event |
| `event.detail["instance-id"]` | EC2 instance ID |
| `event.detail.state` | EC2 instance state |
| `console.log()` | Writes information to Lambda logs |

### Important Concept

The Lambda function does not directly write objects into CloudWatch Logs using an application-specific API call.

Instead, the function uses `console.log()`.

The Lambda service sends the function's standard log output to CloudWatch Logs when the execution role has the necessary permissions.

### Deploy the Code

1. Open the Lambda function.
2. Navigate to the **Code** tab.
3. Replace the default code with the provided implementation.
4. Confirm the handler configuration matches the exported handler.
5. Select **Deploy**.

### Expected Result

The Lambda function code is successfully deployed.

### Screenshot

`03-lambda-function-code.png`

---

# 7. Step 4 – Configure the Lambda Timeout

The instructor increases the Lambda timeout to ten seconds.

Navigate to:

**Lambda → Configuration → General configuration**

Select **Edit**.

### Configuration

| Setting | Value |
|---|---|
| Timeout | 10 seconds |

Select **Save**.

### Why Configure a Timeout?

The timeout determines the maximum time Lambda allows a function invocation to execute.

If execution exceeds the configured timeout, Lambda terminates the invocation.

For this lab, ten seconds is sufficient because the function only logs event information.

### Screenshot

`04-lambda-timeout-configuration.png`

---

# 8. Step 5 – Create an EventBridge Rule

Navigate to:

**AWS Management Console → Amazon EventBridge**

Under **Buses**, select **Rules**.

Select **Create rule**.

### Rule Configuration

| Setting | Value |
|---|---|
| Rule Name | `ec2-instance-state-change-rule` |
| Event Bus | Default |
| Rule Type | Rule with an event pattern |

Select **Next**.

### Why Use the Default Event Bus?

Supported AWS services automatically send relevant service events to the default EventBridge event bus.

EC2 instance state-change notifications are among these supported events.

Therefore, a custom event bus is not required for this lab.

### Screenshot

`05-eventbridge-rule-configuration.png`

---

# 9. Step 6 – Configure the Event Pattern

The next step is to define which events should trigger the rule.

### Event Source Configuration

| Setting | Value |
|---|---|
| Event Source | AWS services |
| AWS Service | EC2 |
| Event Type | EC2 Instance State-change Notification |
| Specific States | Running, Stopped |
| Instance Selection | Any instance |

The instructor configures the rule to capture both running and stopped state changes.

### Example Event Pattern

```json
{
  "source": ["aws.ec2"],
  "detail-type": [
    "EC2 Instance State-change Notification"
  ],
  "detail": {
    "state": [
      "running",
      "stopped"
    ]
  }
}
```

### How the Rule Works

```text
                    EC2 State Change
                           |
                           v
                     EventBridge
                           |
                           v
                    Evaluate Pattern
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Running       Stopped      Other State
              |            |            |
              v            v            v
            Match        Match       No Match
              |            |
              +------+-----+
                     |
                     v
                Invoke Lambda
```

### Important Concept

The rule matches events based on their contents.

Only events matching the configured pattern are forwarded to the target.

### Monitoring a Specific Instance

The instructor selects **Any instance**.

However, you can restrict the rule to a particular EC2 instance.

For example:

```json
{
  "source": ["aws.ec2"],
  "detail-type": [
    "EC2 Instance State-change Notification"
  ],
  "detail": {
    "instance-id": [
      "i-0123456789abcdef0"
    ],
    "state": [
      "running",
      "stopped"
    ]
  }
}
```

Replace the example instance ID with the instance you want to monitor.

### Screenshot

`06-eventbridge-event-pattern.png`

---

# 10. Step 7 – Configure the Lambda Target

After defining the event pattern, configure the target.

### Target Configuration

| Setting | Value |
|---|---|
| Target Type | AWS service |
| Target Service | Lambda function |
| Target Location | This account |
| Function | `ec2-instance-state-change-logs` |

Select the Lambda function created earlier.

### Architecture

```text
                   EC2 State Change
                           |
                           v
                    EventBridge Rule
                           |
                           v
                      AWS Lambda
```

### Permissions

EventBridge must be authorized to invoke the Lambda function.

For a same-account Lambda target, the console can configure the required resource-based invocation permission.

Separately, the Lambda execution role needs permission to write logs to CloudWatch Logs.

These are different permission requirements.

| Permission | Purpose |
|---|---|
| Lambda resource-based permission | Allows EventBridge to invoke the function |
| Lambda execution role | Allows the function to write logs and access permitted AWS resources |

### Important Clarification

The lecture describes allowing the console to create a role during target configuration.

For a direct Lambda target, the essential EventBridge invocation authorization is normally a **Lambda resource-based policy**, rather than a general-purpose EventBridge execution role.

### Screenshot

`07-eventbridge-lambda-target.png`

---

# 11. Step 8 – Review and Create the Rule

Review the configuration.

### Expected Configuration

```text
Rule Name:
ec2-instance-state-change-rule

Event Bus:
default

Event Source:
Amazon EC2

Event Type:
EC2 Instance State-change Notification

Monitored States:
running
stopped

Target:
AWS Lambda
```

Select **Create rule**.

### Expected Result

The EventBridge rule is created successfully.

### Screenshot

`08-eventbridge-rule-created.png`

---

# 12. Step 9 – Generate an EC2 State-Change Event

Now that the EventBridge rule and Lambda function are configured, we can test the workflow.

Navigate to:

**EC2 → Instances**

Select the running EC2 instance.

Choose:

**Instance state → Stop instance**

Confirm the action.

### State Transition

```text
Running
   |
   v
Stopping
   |
   v
Stopped
```

### What Happens Behind the Scenes?

When the instance enters the stopped state:

1. EC2 generates a state-change notification.
2. The event reaches EventBridge.
3. The rule evaluates the event.
4. The event matches the `stopped` state.
5. EventBridge invokes Lambda.
6. Lambda logs the event details.
7. CloudWatch Logs stores the output.

### Expected Result

```text
Instance State: Stopped
```

### Screenshot

`09-ec2-instance-stopped.png`

---

# 13. Step 10 – Verify Events in CloudWatch Logs

Navigate to:

**AWS Management Console → CloudWatch → Logs → Log groups**

Locate the Lambda function's log group.

The log group generally follows this naming convention:

```text
/aws/lambda/<function-name>
```

For this lab:

```text
/aws/lambda/ec2-instance-state-change-logs
```

Open the log group.

Select the log stream created during the Lambda invocation.

### Expected Log Information

The Lambda logs should contain information such as:

- Event ID
- Event source
- AWS account
- AWS Region
- EC2 instance ID
- EC2 instance state

### Example Output

```text
EC2 Instance State Change Captured

Event ID: example-event-id

Event Source: aws.ec2

AWS Account: 123456789012

AWS Region: us-east-1

Instance ID: i-0123456789abcdef0

Instance State: stopped
```

The exact output depends on the Lambda code.

### Additional Lambda Logs

CloudWatch may also contain Lambda runtime information such as:

```text
START RequestId
END RequestId
REPORT RequestId
```

The `REPORT` entry can include execution duration, billed duration, and memory usage.

### Expected Result

The EC2 stopped-state event is visible in CloudWatch Logs.

### Screenshot

`10-cloudwatch-ec2-state-change-logs.png`

---

# 14. Step 11 – Test the Running State

The EventBridge rule monitors both running and stopped states.

We can perform an additional test by starting the EC2 instance again.

Navigate to:

**EC2 → Instances**

Select the stopped instance.

Choose:

**Instance state → Start instance**

### State Transition

```text
Stopped
   |
   v
Pending
   |
   v
Running
```

Once the instance enters the running state, another matching event should be generated.

### Expected Workflow

```text
EC2 Instance Running
          |
          v
     EventBridge
          |
          v
     Event Rule
          |
          v
       Lambda
          |
          v
    CloudWatch Logs
```

### Expected Result

CloudWatch Logs contains another event indicating:

```text
Instance State: running
```

### Important Note

The instructor demonstrates stopping the instance and verifying the stopped-state event.

Starting the instance again is an additional validation exercise based on the rule's configuration.

---

# 15. Understanding the Complete Event Flow

Let's examine the complete architecture.

```text
                   Amazon EC2 Instance
                            |
                            |
                   Instance State Change
                            |
                            v
                    AWS Service Event
                            |
                            v
                  EventBridge Default Bus
                            |
                            v
                    EventBridge Rule
                            |
                    Event Pattern
                            |
                  +---------+---------+
                  |                   |
                  v                   v
               Matches             No Match
                  |                   |
                  v                   v
             Lambda Target        Ignore Event
                  |
                  v
             Lambda Handler
                  |
                  v
              console.log()
                  |
                  v
             CloudWatch Logs
                  |
                  v
             Log Group
                  |
                  v
             Log Stream
                  |
                  v
           State Change Details
```

### Key Learning

This lab demonstrates how AWS services can communicate through events without requiring custom polling logic.

---

# 16. EventBridge vs CloudWatch Logs

Amazon EventBridge and Amazon CloudWatch serve different purposes.

| Feature | Amazon EventBridge | Amazon CloudWatch Logs |
|---|---|---|
| Primary Purpose | Event routing | Log collection and storage |
| Receives EC2 State Events | Yes | Through a logging integration |
| Event Filtering | Event patterns | Log filtering and queries |
| Invokes Lambda | Yes | Not its primary function |
| Stores Lambda Logs | No | Yes |
| Common Use Case | Trigger automation | Investigate application behavior |

### Important Concept

EventBridge detects and routes events.

Lambda processes the events.

CloudWatch Logs records the resulting output.

---

# 17. Real-World Applications

Although the lab uses a simple Lambda function, the same architecture can support more advanced workflows.

### Example 1: EC2 Operational Notifications

```text
EC2 State Change
       |
       v
   EventBridge
       |
       v
     Lambda
       |
       v
   Notification
```

An operations team could receive a notification when a monitored instance stops.

### Example 2: Infrastructure Automation

```text
EC2 State Change
       |
       v
   EventBridge
       |
       v
     Lambda
       |
       v
 Automated Workflow
```

A Lambda function could initiate an approved infrastructure workflow based on the event.

### Example 3: Operational Event Logging

```text
EC2 State Change
       |
       v
   EventBridge
       |
       v
     Lambda
       |
       v
  CloudWatch Logs
```

This is the architecture demonstrated in the lab.

---

# 18. Troubleshooting

## Issue 1: EventBridge Rule Is Not Triggering

### Possible Causes

- Incorrect event pattern
- Rule is disabled
- Incorrect AWS Region
- Incorrect instance ID
- Event state does not match the configured pattern

### Resolution

Verify the rule configuration and event pattern.

---

## Issue 2: Lambda Is Not Invoked

### Possible Causes

- Incorrect Lambda target
- Missing Lambda invocation permission
- Rule configuration is incorrect

### Resolution

Verify the target configuration and Lambda resource-based permissions.

---

## Issue 3: CloudWatch Log Group Is Missing

### Possible Causes

- Lambda has not been invoked
- Lambda execution role lacks logging permissions
- Incorrect AWS Region
- Incorrect log group name

### Resolution

Check Lambda invocation metrics and verify the execution role permissions.

---

## Issue 4: Lambda Is Invoked but Event Details Are Missing

### Possible Causes

- Incorrect event field references
- Incorrect Lambda code
- Unexpected event structure

### Resolution

Review the event payload and Lambda logging statements.

---

# 19. Common Mistakes

### 1. Creating the EventBridge Rule in the Wrong Region

The EC2 instance and EventBridge rule should be configured in the appropriate Region for the events being monitored.

### 2. Selecting the Wrong Event Type

For this lab, use:

`EC2 Instance State-change Notification`

### 3. Forgetting to Select Running and Stopped States

The event pattern must include the states you want to capture.

### 4. Forgetting to Deploy Lambda Code

Changes in the Lambda code editor must be deployed before they take effect.

### 5. Confusing Lambda Invocation Permissions with Execution Role Permissions

EventBridge needs permission to invoke Lambda.

Lambda needs execution-role permissions to write logs.

### 6. Looking for Logs Before Triggering an Event

The Lambda log group may not exist until the function has been invoked.

### 7. Forgetting to Clean Up Resources

Temporary lab resources should be removed when they are no longer needed.

---

# 20. AWS Certification Exam Scenarios

## Scenario 1: EC2 State Monitoring

A company wants to invoke a Lambda function whenever an EC2 instance stops.

**Question:** Which AWS service should be used?

**Answer:** Amazon EventBridge.

**Reason:** EventBridge can match EC2 state-change notifications and invoke Lambda.

---

## Scenario 2: Filter Specific States

A company wants to process only EC2 running and stopped events.

**Question:** Which feature should be configured?

**Answer:** EventBridge event pattern.

**Reason:** Event patterns identify matching events based on attributes such as instance state.

---

## Scenario 3: Serverless Event Processing

A company needs to execute custom code whenever an infrastructure event occurs.

**Question:** Which services can be combined?

**Answer:** Amazon EventBridge and AWS Lambda.

**Reason:** EventBridge routes the event, and Lambda executes the processing logic.

---

## Scenario 4: Event Logging

An operations team wants to retain information generated by a Lambda function.

**Question:** Which AWS service should be used?

**Answer:** Amazon CloudWatch Logs.

**Reason:** CloudWatch Logs collects and stores Lambda log output.

---

## Scenario 5: Monitor a Specific EC2 Instance

A company has multiple EC2 instances but wants to monitor state changes for only one instance.

**Question:** How can the EventBridge rule be configured?

**Answer:** Include the EC2 instance ID in the event pattern.

**Reason:** EventBridge can filter events using the instance ID.

---

# 21. Interview Questions

### Q1. What is the objective of this EventBridge lab?

To capture EC2 state changes, invoke Lambda, and record the event details in CloudWatch Logs.

### Q2. Which event bus is used?

The default EventBridge event bus.

### Q3. What is the event source?

Amazon EC2.

### Q4. What event type is monitored?

EC2 Instance State-change Notification.

### Q5. Which instance states are monitored?

Running and stopped.

### Q6. What is the EventBridge target?

An AWS Lambda function.

### Q7. What does the Lambda function do?

It logs information about the EC2 state-change event.

### Q8. Where are Lambda logs stored?

Amazon CloudWatch Logs.

### Q9. What is an EventBridge event pattern?

A configuration that identifies which incoming events should match a rule.

### Q10. Can an EventBridge rule monitor a specific EC2 instance?

Yes. The event pattern can include a specific instance ID.

### Q11. Does EventBridge continuously poll EC2?

No. Supported EC2 state-change events are delivered to EventBridge.

### Q12. What permissions are required?

EventBridge must be authorized to invoke Lambda, and Lambda needs permissions to write logs.

### Q13. What is the purpose of the Lambda timeout?

It limits the maximum execution duration of a function invocation.

### Q14. Can EventBridge invoke multiple targets?

Yes. An event bus rule can have multiple supported targets.

### Q15. What is the main architectural benefit of this implementation?

It provides loosely coupled, event-driven processing without custom polling.

---

# 22. Lab Cleanup

The instructor cleans up the resources after completing the demonstration.

### Cleanup Order

**Step 1 – Delete the CloudWatch Log Group**

Navigate to:

CloudWatch → Logs → Log groups

Delete:

```text
/aws/lambda/ec2-instance-state-change-logs
```

**Step 2 – Delete the Lambda Function**

Navigate to:

Lambda → Functions

Delete:

```text
ec2-instance-state-change-logs
```

**Step 3 – Delete the EventBridge Rule**

Navigate to:

EventBridge → Rules

Delete:

```text
ec2-instance-state-change-rule
```

**Step 4 – Terminate the EC2 Instance**

Navigate to:

EC2 → Instances

Select the temporary lab instance.

Choose:

Instance state → Terminate instance

### Important Warning

Terminate the EC2 instance only if it was created specifically for this lab and is no longer required.

Do not terminate an existing application instance.

### Cleanup Checklist

- [ ] CloudWatch log group deleted
- [ ] Lambda function deleted
- [ ] EventBridge rule deleted
- [ ] Temporary EC2 instance terminated
- [ ] No unnecessary lab resources remain

---

# 23. Recommended GitHub Screenshots

| # | Screenshot Filename | Description |
|---|---|---|
| 1 | `01-ec2-instance-running.png` | EC2 instance in running state |
| 2 | `02-lambda-function-created.png` | Lambda function configuration |
| 3 | `03-lambda-function-code.png` | Lambda JavaScript implementation |
| 4 | `04-lambda-timeout-configuration.png` | Lambda timeout configuration |
| 5 | `05-eventbridge-rule-configuration.png` | EventBridge rule creation |
| 6 | `06-eventbridge-event-pattern.png` | EC2 running/stopped event pattern |
| 7 | `07-eventbridge-lambda-target.png` | Lambda target selection |
| 8 | `08-eventbridge-rule-created.png` | Successfully created EventBridge rule |
| 9 | `09-ec2-instance-stopped.png` | EC2 instance stopped |
| 10 | `10-cloudwatch-ec2-state-change-logs.png` | Captured EC2 state-change event |

### Suggested GitHub Directory Structure

```text
application-integration/
│
├── 08-amazon-eventbridge-fundamentals.md
│
└── labs/
    │
    └── 02-eventbridge-ec2-state-change-monitoring.md
       
    
```

For a repository that stores screenshots separately:

```text
application-integration/
│
├── 08-amazon-eventbridge-fundamentals.md
│
├── labs/
│   └── 02-eventbridge-ec2-state-change-monitoring.md
│
└── screenshots/
    └── eventbridge-ec2-state-monitoring/
        ├── 01-ec2-instance-running.png
        ├── 02-lambda-function-created.png
        ├── 03-lambda-function-code.png
        ├── 04-lambda-timeout-configuration.png
        ├── 05-eventbridge-rule-configuration.png
        ├── 06-eventbridge-event-pattern.png
        ├── 07-eventbridge-lambda-target.png
        ├── 08-eventbridge-rule-created.png
        ├── 09-ec2-instance-stopped.png
        └── 10-cloudwatch-ec2-state-change-logs.png
```

---

# 24. Key Takeaways

- Amazon EventBridge supports event-driven infrastructure monitoring.
- EC2 instance state changes generate events.
- Supported EC2 events are delivered to the default EventBridge event bus.
- EventBridge rules match events using event patterns.
- Event patterns can filter events based on instance state.
- Rules can also target specific EC2 instances.
- AWS Lambda can be invoked when matching events occur.
- Lambda functions can log event information using `console.log()`.
- CloudWatch Logs stores Lambda log output.
- EventBridge and Lambda enable serverless automation.
- Event-driven architectures reduce the need for continuous polling.
- IAM and resource-based permissions are important for service integration.
- Temporary lab resources should be cleaned up after testing.

---

## The Most Important Concept to Remember

```text
                      AMAZON EC2
                          |
                          v
                    STATE CHANGE
                          |
                          v
                   AMAZON EVENTBRIDGE
                          |
                          v
                    DEFAULT EVENT BUS
                          |
                          v
                    EVENTBRIDGE RULE
                          |
                   +------+------+
                   |             |
                   v             v
                 MATCH         NO MATCH
                   |             |
                   v             v
                AWS LAMBDA     IGNORE
                   |
                   v
              CONSOLE.LOG()
                   |
                   v
              CLOUDWATCH LOGS
                   |
                   v
             EVENT INFORMATION
```

**EC2 = Generates the state-change event**

**EventBridge = Captures and routes the event**

**Event Pattern = Determines which events match**

**Lambda = Processes the matching event**

**CloudWatch Logs = Stores the resulting log information**

**Architecture = Event-driven, serverless monitoring**

---

# 25. Related Topics

- Amazon EventBridge Fundamentals
- Event-Driven Architecture
- EventBridge Event Buses
- EventBridge Rules
- Event Patterns
- AWS Lambda
- Amazon EC2
- Amazon CloudWatch Logs
- IAM Roles and Permissions
- Serverless Automation
- Infrastructure Monitoring

