# Chapter 20: Security and Compliance for SysOps

## Shared Responsibility Model
AWS's responsibility is "security *of* the cloud" (infrastructure, managed services). Your responsibility is "security *in* the cloud" (OS patching, firewall config, IAM, application data encryption). Some controls are shared (e.g., patch management).

## DDoS Protection (Shield, WAF, CloudFront, Route 53)
-   **DDoS Attack:** Distributed Denial-of-Service attack, overwhelms a server with requests.
-   **AWS Shield Standard:** Free, enabled for all customers, protects against common DDoS attacks (SYN/UDP floods, reflection attacks).
-   **AWS Shield Advanced:** Optional ($3000/month/org), premium DDoS protection against sophisticated attacks, 24/7 DDoS response team, cost protection for incurred fees during attacks.
-   **AWS WAF (Web Application Firewall):** Protects web applications from common web exploits (Layer 7). Deployed on ALB, API Gateway, CloudFront. Uses Web ACLs with rules based on IP addresses, headers, body, strings, SQL injection, cross-site scripting. Supports rate-based rules.
-   **CloudFront & Route 53:** Provide DDoS protection by leveraging the global edge network and distributing traffic.
-   **Scaling:** Auto Scaling can help applications scale to handle higher demand during attacks.

## Penetration Testing on AWS
Customers are generally allowed to perform security assessments and penetration testing against their own AWS infrastructure for specific services (EC2 instances, NAT Gateways, ELB, RDS, CloudFront, Aurora, API Gateway, Lambda, Lightsail, Elastic Beanstalk) without prior approval.
-   **Prohibited Activities:** DNS zone walking (Route 53), DDoS attacks (DoS/DDoS/Simulated DoS/DDoS), port/protocol/request flooding.
-   For other activities, contact AWS security team for approval.

## Amazon Inspector
Automated security assessment service for EC2 instances, ECR container images, and Lambda functions.
-   **Functionality:** Analyzes against unintended network accessibility and known vulnerabilities (CVEs) in running OS and package dependencies. Continuous scanning.
-   **Reporting:** Reports findings to AWS Security Hub and sends events to Amazon EventBridge for automation.
-   **Risk Scoring:** Assigns a risk score to vulnerabilities for prioritization.

## AWS Artifact
A portal providing on-demand access to AWS compliance reports (e.g., ISO, PCI, SOC) and agreements (e.g., BAA, HIPAA). Used to support internal audits and compliance needs.

## AWS Certificate Manager (ACM)
Manages TLS/SSL certificates for in-flight encryption.
-   **Public Certificates:** Free, automatically renewed (if ACM-generated and DNS validated). Cannot be extracted.
-   **Private Certificates:** For internal use.
-   **Validation:** DNS validation (preferred for automation with Route 53) or email validation.
-   **Integration:** Integrates with ELB (Classic, ALB, NLB), CloudFront, API Gateway. Cannot be used directly with EC2 instances.
-   **Renewal:** ACM-generated certificates are automatically renewed 60 days before expiry. Imported certificates require manual renewal.
-   **Expiration Events:** ACM sends daily expiration events to EventBridge starting 45 days prior to expiration. Config rules (e.g., `ACM-certificate-expiration-check`) can also check for expiring certificates.

## AWS Secrets Manager
Stores and manages secrets (e.g., database credentials, API keys).
-   **Key Features:**
    -   **Automated Rotation:** Forces rotation of secrets every X days (e.g., 30 days) using Lambda functions (AWS-provided for RDS, Redshift, DocumentDB; custom for others).
    -   **Integration:** Well-integrated with RDS, Redshift, DocumentDB.
    -   **Encryption:** Secrets are encrypted using KMS (mandatory).
    -   **Multi-Region Secrets:** Replicate secrets across multiple AWS regions for disaster recovery and multi-region applications.
-   **Cost:** $0.40/secret/month + $0.05/10,000 API calls (first 30 days free).

## AWS KMS (Key Management Service)
Manages encryption keys.
-   **Key Types:**
    -   **Symmetric:** Single key for encrypt/decrypt. Used by most AWS services.
    -   **Asymmetric:** Public key (encrypt), private key (decrypt). Used for encrypt/decrypt or sign/verify operations. Public key can be downloaded.
-   **Key Ownership:**
    -   **AWS Owned Keys:** Free, managed by AWS (e.g., SSE-S3).
    -   **AWS Managed Keys:** Free, managed by AWS, recognizable by `aws/service-name` alias (e.g., `aws/rds`).
    -   **Customer Managed Keys (CMK):** User-managed, $1/month. Can be imported.
-   **Key Policies:** Control access to KMS keys (similar to S3 bucket policies). Required for cross-account access.
-   **Key Rotation:**
    -   **Automatic:** AWS-managed keys (every 1 year). Customer-managed symmetric keys (optional, configurable 90-2560 days, default 1 year).
    -   **On-Demand:** For customer-managed symmetric keys.
    -   **Manual:** For imported KMS keys.
-   **Cross-Region Copy:** To copy an encrypted EBS snapshot to another region, it must be re-encrypted with a KMS key in the target region.

## AWS Organizations
Manages multiple AWS accounts.
-   **Management Account:** Central account for billing and management.
-   **Member Accounts:** Other accounts in the organization.
-   **Consolidated Billing:** Single payment method, aggregated usage for pricing benefits, RI/Savings Plan sharing.
-   **Organizational Units (OUs):** Hierarchical grouping of accounts for management and policy application.
-   **Service Control Policies (SCPs):** IAM policies applied to OUs or accounts to restrict permissions. Apply to all users/roles in the OU/account (except management account). Require explicit allow along the path.

## AWS Control Tower
Simplifies setup and governance of secure, multi-account AWS environments based on best practices.
-   **Landing Zone:** Automates environment setup (creates OUs, shared accounts for logs/audit).
-   **Guardrails:** Automate policy management, detect violations (preventive and detective).
-   **Integration:** Runs on top of AWS Organizations, implements SCPs.

## AWS Service Catalog
Self-service portal for users to launch authorized AWS products (CloudFormation templates).
-   **Products:** CloudFormation templates.
-   **Portfolios:** Collections of products.
-   **Access Control:** IAM permissions define who can access portfolios.
-   **TagOptions:** Manage tags on provisioned products for proper resource tagging.

## AWS Compute Optimizer
Recommends optimal AWS resources (EC2, ASG, EBS, Lambda) to reduce costs and improve performance using machine learning.

## AWS Trusted Advisor
Provides high-level account assessment and advice on cost optimization, performance, security, fault tolerance, service limits, and operational excellence. Some checks require business/enterprise support plans.

## AWS Cost Explorer
Visualizes, understands, and manages AWS costs and usage over time.
-   **Reports:** Custom reports for cost and usage data (monthly, hourly, resource level).
-   **Savings Plan Recommendations:** Helps choose optimal Savings Plans.
-   **Forecasting:** Forecasts usage up to 12 months.

## AWS Budgets
Creates cost and usage alarms.
-   **Budget Types:** Cost, usage, reservation, savings plan.
-   **Alerts:** Send notifications (email, SNS, Chatbots) when actual or forecasted costs/usage exceed defined thresholds.
-   **Actions:** Can trigger automated actions (e.g., attach IAM policy, stop EC2/RDS instances) when budget thresholds are breached.

## AWS Cost Allocation Tags
User-defined tags (e.g., `Environment:Production`) and AWS-generated tags (e.g., `aws:createdBy`) used to track AWS costs at a detailed level in cost and usage reports.

## AWS Cost and Usage Reports
Comprehensive reports of AWS costs and usage, including detailed metadata and cost allocation tags. Delivered to S3.

## CloudWatch Metrics
Provides performance metrics for AWS resources and applications.

## CloudWatch Logs
Centralized service for storing application logs.

## CloudWatch Alarms
Trigger notifications or actions based on metric thresholds.

## CloudWatch Dashboards
Visualize key metrics and alarms.

## CloudWatch Synthetics Canary
Configurable scripts to monitor APIs, URLs, or websites proactively.

## Amazon EventBridge
Event-driven architecture service for routing events between AWS services and your applications.

## AWS CloudTrail
Records API calls and user activity for auditing.

## AWS DataSync
Data transfer service for automating data movement between on-premises and AWS storage.

## AWS Elastic Disaster Recovery (DRS)
AWS's primary service for disaster recovery.

## Disaster Recovery Strategies
Strategies for preparing for and recovering from IT system disasters.

## Cloud Migration Strategies
Strategies for moving applications and data to the cloud.