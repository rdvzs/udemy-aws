# Chapter 18: AWS Account Management

## AWS Health Dashboard
-   **Service History:** Shows general health of all AWS services across regions. Provides historical data and an RSS feed.
-   **Account Health Dashboard (formerly Personal Health Dashboard - PHD):** Provides personalized alerts and remediation guidance for events directly impacting your AWS accounts and resources. Gives relevant and timely information, including notifications for scheduled maintenance. Aggregates data for your entire AWS organization. Accessible via the bell icon in the console. Shows outages directly impacting you and provides an event log for past events.
-   **Event Notifications:** Use EventBridge to react to AWS Health events. Example: receive email notifications for EC2 instance updates, or automatically delete exposed IAM keys using Lambda, or restart EC2 instances scheduled for retirement.

## AWS Organizations
-   **Global Service:** Manages multiple AWS accounts.
-   **Structure:**
    -   **Management Account:** The main account in an organization.
    -   **Member Accounts:** Other accounts that join or are created within the organization. Can only be part of one organization.
    -   **Organizational Units (OUs):** Hierarchical grouping of accounts (e.g., by business unit, environment, or project).
-   **Benefits:**
    -   **Consolidated Billing:** Single payment method for all accounts.
    -   **Pricing Benefits:** Aggregated usage leads to discounts (e.g., for EC2, S3).
    -   **Shared Reserved Instances (RIs) & Savings Plans:** Discounts apply across the entire organization. Can be enabled/disabled per account.
    -   **Automated Account Creation:** API to easily create new accounts.
    -   **Enhanced Security:** Better separation than multiple VPCs in one account.
    -   **Centralized Logging:** Enable CloudTrail/CloudWatch logs for all accounts to a central S3 bucket/logging account.
    -   **Cross-Account Roles:** Establish roles for admin purposes.
    -   **Tagging Standards:** Enforce tagging for billing and resource categorization.
    -   **`aws:PrincipalOrgID` Condition:** Use in IAM policies to grant access to IAM principals from all accounts in your organization.
-   **Service Control Policies (SCPs):**
    -   IAM policies applied to OUs or accounts to restrict what users and roles can do.
    -   Do NOT apply to the management account (always has full admin power).
    -   Require explicit `Allow` along the path from root OU to the account for an action to be permitted. `Deny` statements override `Allow` statements.
    -   Examples: Block specific services (e.g., DynamoDB, S3, EC2), or allow only specific services.
-   **Tag Policies:** Enforce tagging standards across accounts for auditing, categorization, and cost allocation. Can monitor non-compliant tags using CloudWatch Events.

## AWS Control Tower
-   **Easy Setup & Governance:** Simplifies setting up and governing a secure, compliant, multi-account AWS environment based on best practices.
-   **Automates:** Environment setup, ongoing policy management using guardrails, detection of policy violations, and remediation.
-   **Interactive Dashboard:** Monitors compliance.
-   **Built on Organizations:** Automatically sets up AWS Organizations and implements SCPs for guardrails.
-   **Landing Zone:** Creates a multi-account environment with a Home Region, Security OU (for log archive and security audit accounts), and a Sandbox OU.
-   **Guardrails:** Preventive (enforce policies, e.g., disallow deletion of log archives, public read access to logs, CloudTrail changes) and Detective (detect configuration violations).
-   **Account Factory:** Used to enroll new accounts into Control Tower.
-   **IAM Identity Center (formerly AWS SSO):** Manages user access to all accounts within Control Tower. Provides a user portal for single sign-on access to audit, log archive, and other accounts.

## AWS Billing & Cost Management Tools

### Billing Alarms
-   **Region:** Billing data is stored in `us-east-1` (N. Virginia) but represents worldwide costs.
-   **Setup:** Enable billing alerts in Billing preferences.
-   **CloudWatch:** Create alarms in CloudWatch (only visible in `us-east-1`).
-   **Metrics:** Monitor total estimated charges or specific service costs (e.g., EC2, S3).
-   **Notifications:** Trigger SNS topics for alerts (e.g., if cost exceeds a threshold).

### Cost Explorer
-   **Visualization & Management:** Visualize, understand, and manage AWS costs and usage over time.
-   **Custom Reports:** Analyze cost and usage data at high-level (total cost) or granular level (monthly, hourly, resource level).
-   **Cost Savings:** Helps choose optimal Savings Plans.
-   **Forecasting:** Forecasts usage up to 12 months based on previous usage for cost planning.
-   **Detailed Breakdown:** Provides hourly and resource-level cost information.

### AWS Budgets
-   **Granular Budgeting:** Create budgets based on actual or estimated charges, with alarms for exceeding thresholds. More granular and flexible than billing alarms.
-   **Budget Types:** Cost, Usage, Reservation, Savings Plan.
-   **Reservation Tracking:** Track utilization of Reserved Instances (EC2, ElastiCache, RDS, Redshift).
-   **Notifications:** Up to five notifications per budget. Can filter by service, linked account, tag, etc.
-   **Actions:** Automatically apply IAM policies, Service Control Policies, or stop EC2/RDS instances when budget thresholds are met. Requires an IAM role for Budgets to perform actions.
-   **Cost:** First two budgets are free, then $0.02 per day per budget.

### Cost Allocation Tags & Cost and Usage Reports (CUR)
-   **Cost Allocation Tags:** Track AWS costs at a detailed level.
    -   **AWS-Generated Tags:** Automatically applied (e.g., `aws:createdBy`). Start with `aws:`.
    -   **User-Defined Tags:** Defined by users (e.g., `user:Environment`). Start with `user:`.
    -   **Activation:** Tags must be activated in the Billing console to appear in cost reports.
-   **Cost and Usage Reports (CUR):**
    -   Most comprehensive set of AWS cost and usage data.
    -   Includes metadata on services, pricing, and reservations.
    -   Lists usage for each service with hourly/daily line items.
    -   Includes activated cost allocation tags.
    -   **Export:** Configurable for daily exports to Amazon S3.
    -   **Analysis:** Data can be analyzed using Athena, Redshift, or QuickSight.

## AWS Compute Optimizer
-   **Cost Reduction & Performance Improvement:** Recommends optimal AWS resources for workloads.
-   **Machine Learning:** Analyzes resource configuration and CloudWatch metrics to identify over/under-provisioned resources.
-   **Supported Resources:** EC2 instances, Auto Scaling groups, EBS volumes, Lambda functions.
-   **Benefits:** Can lower costs by up to 25%.
-   **Export:** Recommendations can be exported to Amazon S3.