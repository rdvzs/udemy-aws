# Chapter 17: Monitoring, Auditing, and Performance

## Amazon CloudWatch
CloudWatch is a monitoring service for AWS resources and applications.
-   **Metrics:** Provides metrics for every AWS service (e.g., CPU Utilization, NetworkIn). Metrics belong to namespaces and can have dimensions. EC2 instances send metrics every 5 minutes by default; detailed monitoring (1-minute metrics) can be enabled for a cost. Memory usage is not pushed by default and requires a custom metric.
-   **Custom Metrics:** You can define and push your own custom metrics (e.g., memory usage, disk space, login users) using `PutMetricData` API call. Metrics can have standard (1-minute) or high resolution (1, 5, 10, 30 seconds). CloudWatch accepts metrics up to 2 weeks in the past or 2 hours in the future.
-   **Dashboards:** Visualize key metrics and alarms across multiple regions and accounts. Can include various widgets (line, stacked area, number, text, logs table, alarm status). Three dashboards (up to 50 metrics) are free.
-   **Logs:** Centralized service for storing application logs.
    -   **Log Groups:** Define logical groupings for logs (e.g., per application).
    -   **Log Streams:** Represent instances, specific log files, or containers within a log group.
    -   **Log Retention:** Configurable from 1 day to indefinitely.
    -   **Log Sources:** SDK, CloudWatch Unified Agent, Elastic Beanstalk, ECS, Lambda, VPC Flow Logs, API Gateway, CloudTrail, Route 53.
    -   **Logs Insights:** Query engine for searching and analyzing log data using a purpose-built query language. Supports filtering, aggregation, sorting, and querying multiple log groups across accounts/regions.
    -   **Log Export:** Batch export to S3 (up to 12 hours to complete) using `CreateExportTask`.
    -   **Subscription Filters:** Real-time stream of log events to Kinesis Data Streams, Kinesis Data Firehose, or Lambda for processing and analysis. Can aggregate logs from different accounts/regions.
    -   **Live Tail:** Real-time view of incoming log events for debugging.
-   **Alarms:** Trigger notifications or actions based on metric thresholds.
    -   **States:** OK, INSUFFICIENT_DATA, ALARM.
    -   **Targets:** EC2 actions (stop, terminate, reboot, recover), Auto Scaling actions (scale out/in), SNS notifications (can trigger Lambda functions).
    -   **Composite Alarms:** Monitor states of multiple other alarms using AND/OR conditions to reduce noise.
    -   **EC2 Instance Recovery:** Automatically recovers an EC2 instance if status checks fail, preserving IP, metadata, and placement group.
    -   **Testing Alarms:** Use `set-alarm-state` CLI command to manually trigger an alarm for testing.

## AWS Service Quotas
A service to view and manage quotas (limits) for AWS services.
-   **Monitoring:** Monitor quota usage and set CloudWatch Alarms when usage approaches a threshold (e.g., 80% of limit).
-   **Quota Increase:** Request quota increases directly from the console.
-   **Alternative:** Trusted Advisor also checks service limits, but Service Quotas is more comprehensive.

## AWS CloudTrail
Provides governance, compliance, and audit for your AWS account by recording API calls and user activity. Enabled by default.
-   **Event Types:**
    -   **Management Events:** Operations on AWS resources (e.g., `IAM AttachRolePolicy`, `CreateSubnet`). Logged by default. Can separate Read (non-modifying) and Write (modifying) events.
    -   **Data Events:** High-volume operations on data resources (e.g., S3 object-level activity like `GetObject`, `PutObject`, Lambda function `Invoke`). Not logged by default, incurs cost.
    -   **Insights Events:** Detects unusual activity (e.g., inaccurate resource provisioning, hitting service limits, bursts of IAM actions). Requires enabling and incurs cost. Anomalies appear in console, sent to S3, and trigger EventBridge events.
-   **Event Retention:** Events stored for 90 days in CloudTrail console. For longer retention, log to S3 and analyze with Athena.
-   **Log File Integrity Validation:** Generates digest files (SHA-256 hash) to verify that log files delivered to S3 have not been tampered with.
-   **Integration with EventBridge:** CloudTrail events are sent to EventBridge, allowing rules to trigger actions (e.g., SNS notification for `DeleteTable` API call, `AssumeRole` API call, `AuthorizeSecurityGroupIngress` API call).
-   **Organization Trails:** Configure CloudTrail at the AWS Organizations level to log events for all member accounts into a central S3 bucket. Member accounts cannot modify the organization trail.

## AWS Config
Records configuration changes and assesses compliance of AWS resources against rules.
-   **Resource Recording:** Records configuration of all supported resources in a region (can include global resources like IAM). Stores configuration history in S3.
-   **Rules:**
    -   **AWS Managed Rules:** Over 75 predefined rules (e.g., `restricted-ssh`).
    -   **Custom Rules:** Defined using Lambda functions (e.g., check EBS disk type, EC2 instance type).
    -   **Triggering:** Rules can be triggered by configuration changes or at regular time intervals.
-   **Compliance:** Resources are marked compliant or non-compliant. Config rules do not prevent actions but provide auditing.
-   **Remediation:** Automatically or manually remediate non-compliant resources using SSM Automation Documents (e.g., deactivate old IAM access keys). Can configure retries.
-   **Notifications:** Can send configuration changes and compliance notifications to SNS or trigger EventBridge events.
-   **Aggregators:** Centralize configuration and compliance data from multiple accounts and regions into a single aggregator account. If using AWS Organizations, authorization is automatic.
-   **CloudFormation StackSets:** Used to deploy Config rules across multiple accounts and regions.

## AWS X-Ray
Helps developers analyze and debug distributed applications (microservices, serverless, web apps).
-   **Components:**
    -   **X-Ray SDK:** Integrated into application code to send data.
    -   **X-Ray Daemon:** Collects data from SDK and sends to X-Ray service.
    -   **X-Ray Service:** Stores and processes data.
-   **Features:**
    -   **Service Map:** Visual representation of application components and their connections.
    -   **Traces:** Detailed view of individual requests, showing latency and errors across services.
    -   **Anomalies:** Identifies errors and faults.
-   **Integration:** Integrates with EC2, Lambda, ECS, EKS, Elastic Beanstalk, API Gateway.
-   **Use Cases:** Performance analysis, bottleneck identification, error troubleshooting.

## CloudWatch Synthetics Canary
Configurable scripts (Node.js or Python) run from CloudWatch to monitor APIs, URLs, or websites.
-   **Functionality:** Reproduce customer actions (e.g., login, add to cart, checkout), check availability/latency of endpoints, store load time data, take UI screenshots.
-   **Blueprints:** Heartbeat Monitor, API Canary, Broken Link Checker, Visual Monitoring, Canary Recorder (with CloudWatch Synthetics Recorder), GUI Workflow Builder.
-   **Use Cases:** Proactive monitoring, finding issues before customers, testing user flows.