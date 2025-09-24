# Chapter 19: Disaster Recovery

## AWS DataSync
AWS DataSync is a data transfer service for automating data movement between on-premises storage (NFS, SMB, HDFS, self-managed object storage) and AWS storage services (S3, EFS, FSx). It can also synchronize data between different AWS storage services.
-   **Agent:** Requires a DataSync agent (VM) installed on-premises or in another cloud. AWS Snowcone devices come with the DataSync agent pre-installed.
-   **Synchronization:** Scheduled tasks (hourly, daily, weekly), not continuous.
-   **Metadata Preservation:** Preserves file permissions and metadata (POSIX compliant for NFS, SMB permissions). This is a key differentiator for exams.
-   **Bandwidth:** Can utilize up to 10 Gbps per agent, with optional bandwidth limits.
-   **Direction:** Supports bidirectional synchronization (on-premises to AWS, and AWS to on-premises).

## AWS Backup
AWS Backup is a fully managed service for centralizing and automating backups across various AWS services (EC2, EBS, S3, RDS, Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx, Storage Gateway volumes).
-   **Backup Plans:** Define backup frequency (e.g., daily, monthly), backup window, transition to cold storage (e.g., Glacier), and retention period.
-   **Resource Assignment:** Assign resources to backup plans by tags or specific resource types.
-   **Cross-Region/Cross-Account Backups:** Supports copying backups to other regions or accounts for disaster recovery.
-   **Point-in-Time Recovery (PITR):** Supported for certain services (e.g., Aurora).
-   **Vault Lock:** Enforces a WORM (Write Once Read Many) policy on backup vaults, preventing accidental or malicious deletion/modification of backups, even by the root user.

## AWS Elastic Disaster Recovery (DRS)
AWS Elastic Disaster Recovery (DRS) is AWS's primary service for disaster recovery, minimizing downtime and data loss for physical, virtual, and cloud-based applications into AWS. It is the successor to CloudEndure Disaster Recovery.
-   **Agent:** Installs a lightweight agent on source servers.
-   **Replication:** Replicates data at the block level to a low-cost, low-resource EC2 staging area in AWS.
-   **Recovery:** Automatically converts machines to run natively on AWS during a disaster, minimizing downtime (minutes).
-   **Continuous Data Protection:** Provides continuous data replication for reliability.

## Disaster Recovery Strategies
Strategies for preparing for and recovering from IT system disasters, focusing on minimizing downtime and data loss.
-   **Recovery Time Objective (RTO):** Maximum acceptable delay between service interruption and restoration.
-   **Recovery Point Objective (RPO):** Maximum acceptable amount of data loss (measured in time).
-   **Four Main Strategies:**
    -   **Backup and Restore:** Cheapest, easiest. Back up data to S3, restore in case of disaster. Higher RTO/RPO.
    -   **Pilot Light:** Minimal version of application running in DR region. Faster RTO/RPO than backup/restore.
    -   **Warm Standby:** Scaled-down version of application running in DR region. Faster RTO/RPO than pilot light.
    -   **Multi-Site Active/Active:** Most expensive, most complex. Application runs simultaneously in multiple regions. Lowest RTO/RPO.

## Cloud Migration Strategies (The 7 Rs)
Strategies for moving applications and data from on-premises to the cloud.
-   **Rehost (Lift and Shift):** Move applications as is to the cloud without changes.
-   **Replatform (Lift, Tinker, and Shift):** Minor changes to optimize applications for the cloud.
-   **Refactor (Re-architect):** Re-architect applications to take full advantage of cloud-native features.
-   **Repurchase (Drop and Shop):** Replace existing applications with a SaaS solution.
-   **Retire:** Decommission applications no longer needed.
-   **Retain:** Keep applications on-premises due to specific requirements.
-   **Relocate:** Move applications to the cloud without new hardware or rewriting code (e.g., VMware Cloud on AWS).