# Chapter 14: Advanced Storage Section

## AWS Snow Family
The AWS Snow Family consists of highly secure, portable devices for collecting, processing, and migrating data at the edge, and transferring data in and out of AWS.
-   **Snowball Edge Storage Optimized:** 210 TB storage, primarily for data migration.
-   **Snowball Edge Compute Optimized:** 28 TB storage, more compute power, for edge computing use cases (e.g., running EC2 instances or Lambda functions at locations with limited internet access).
-   **Use Cases:**
    -   **Data Migration:** For large datasets (petabytes) or when network transfer is slow, costly, or unstable (e.g., takes over a week). You receive a physical device, load data, and ship it back to AWS for import into S3.
    -   **Edge Computing:** Processing data where it's created (e.g., remote sites, vehicles) before sending it to AWS.

## Amazon FSx
Amazon FSx is a fully managed service that allows you to launch and run popular third-party high-performance file systems on AWS.
-   **FSx for Windows File Server:**
    -   Fully managed Windows File Server.
    -   Supports SMB protocol and Windows NTFS.
    -   Integrates with Microsoft Active Directory for user authentication.
    -   Can be mounted on Linux EC2 instances.
    -   Scales to tens of GB/s throughput, millions of IOPS, hundreds of PB data.
    -   Storage options: SSD (low latency) or HDD (cost-effective).
    -   Multi-AZ deployment for high availability (synchronous replication, automatic failover).
    -   Daily backups to S3.
-   **FSx for Lustre:**
    -   High-performance file system for large-scale computing (HPC, machine learning, video processing, financial modeling).
    -   Scales to hundreds of GB/s throughput, millions of IOPS, sub-millisecond latency.
    -   Storage options: SSD (low latency, IOPS intensive) or HDD (throughput intensive).
    -   Seamless integration with S3 (read S3 as file system, write computation output back to S3).
    -   Deployment options:
        -   **Scratch File System:** Temporary storage, no data replication, high burst performance (6x persistent), data lost on server failure. For short-term processing.
        -   **Persistent File System:** Long-term storage, data replicated within the same AZ, transparent replacement on server failure. For sensitive data.
    -   Can be used from on-premises via VPN/Direct Connect.
-   **FSx for NetApp ONTAP:**
    -   Managed NetApp ONTAP file system.
    -   Compatible with NFS, SMB, and iSCSI protocols.
    -   Broad compatibility (Linux, Windows, macOS, VMware Cloud, WorkSpaces, AppStream, EC2, ECS, EKS).
    -   Auto-scaling storage, replication, snapshots, low cost, data compression, data de-duplication.
    -   Point-in-time instantaneous cloning for testing.
-   **FSx for OpenZFS:**
    -   Managed OpenZFS file system.
    -   Compatible with NFS protocol.
    -   High performance (1M IOPS, <0.5ms latency).
    -   Supports snapshots, compression, low cost (no data de-duplication).
    -   Point-in-time instantaneous cloning.

## AWS Storage Gateway
A hybrid cloud storage service that bridges on-premises data with AWS cloud storage. Deployed as a virtual machine (VM) on-premises.
-   **S3 File Gateway:**
    -   Exposes S3 buckets as NFS or SMB file shares on-premises.
    -   Most recently used data is cached locally.
    -   Supports S3 Standard, S3 Standard-IA, S3 One Zone-IA, S3 Intelligent-Tiering (not Glacier directly, but can transition via lifecycle policies).
    -   Integrates with Active Directory for SMB.
    -   POSIX compliant (metadata, permissions, timestamps stored in S3 object metadata).
-   **Volume Gateway:**
    -   Provides block storage volumes to on-premises applications using iSCSI protocol.
    -   Volumes are backed by EBS snapshots in S3.
    -   **Cached Volumes:** Primary data in S3, frequently accessed data cached locally for low latency.
    -   **Stored Volumes:** Entire dataset stored locally, asynchronously backed up to S3.
    -   Monitoring: `CacheHitPercent` (high is good), `CachePercentUsed` (avoid too high).
-   **Tape Gateway:**
    -   Replaces physical tape libraries with virtual tapes in AWS.
    -   Uses iSCSI-VTL protocol.
    -   Virtual tapes stored in S3 and can be archived to Glacier/Glacier Deep Archive.
-   **Activation:** Gateway VM needs to be activated, either via CLI or web request (Port 80). Time synchronization (NTP) is crucial for activation.
-   **Rebooting:** File Gateway can be simply restarted. Volume/Tape Gateway requires stopping the service, rebooting the VM, then starting the service.

## AWS DataSync
A data transfer service for automating data movement between on-premises storage (NFS, SMB, self-managed object storage, HDFS) and AWS storage services (S3, EFS, FSx for Windows File Server).
-   Deploys a DataSync agent (VM) on-premises.
-   Creates a task defining source, destination, and transfer options (e.g., data integrity verification, metadata preservation, bandwidth limits, scheduling).
-   Fast (up to 10x faster than open-source tools), secure (encryption in transit), reliable (end-to-end data integrity), and cost-effective (incremental transfers).

## CloudEndure Migration (now AWS Application Migration Service - MGN)
A service for simplifying and expediting lift-and-shift (rehosting) cloud migrations.
-   Installs a lightweight agent on source servers (physical, virtual, or cloud).
-   Replicates data at the block level to a low-cost, low-resource EC2 staging area in AWS.
-   Automatically converts machines to run natively on AWS during cutover, minimizing downtime (seconds).
-   Provides continuous data replication for reliability.

## AWS Backup
A fully managed service that centralizes and automates data protection across various AWS services (EBS, RDS, DynamoDB, EFS, Storage Gateway volumes, EC2 instances).
-   Create backup plans defining frequency, window, retention period, and backup vault (logical container for backups).
-   Assign resources to backup plans by tags, resource type, or ID.
-   Automatically creates incremental backups.
-   Simple, minimal operational overhead, cost-effective, and secure (encryption at rest and in transit).

## AWS Elastic Disaster Recovery (DRS)
AWS's primary service for disaster recovery, minimizing downtime and data loss for physical, virtual, and cloud-based applications into AWS. It is the successor to CloudEndure Disaster Recovery.
-   Installs a lightweight agent on source servers.
-   Replicates data at the block level to a low-cost, low-resource EC2 staging area in AWS.
-   Automatically converts machines to run natively on AWS during a disaster, minimizing downtime (minutes).
-   Provides continuous data replication for reliability.