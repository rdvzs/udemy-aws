
# Chapter 10: EC2 Storage and Data Management (EBS & EFS)

This chapter provides a deep dive into the primary storage options for EC2 instances: Elastic Block Store (EBS) and Elastic File System (EFS). It covers their types, operational management, and performance characteristics from a SysOps perspective.

## 10.1: EBS (Elastic Block Store) Overview
- **EBS:** A network-attached block storage service for use with EC2 instances. It behaves like a network-attached USB drive.
- **Key Characteristics:**
  - **Persistent:** Data on an EBS volume persists independently of the life of the instance.
  - **AZ-Locked:** An EBS volume is created in a specific Availability Zone (AZ) and can only be attached to instances within that same AZ.
  - **Single Attachment (Mostly):** A volume can only be attached to one EC2 instance at a time. The exception is EBS Multi-Attach for io1/io2 volumes.
- **Delete on Termination:** By default, the root EBS volume is deleted when its instance is terminated, but this behavior can be changed. Additional attached volumes are not deleted by default.

## 10.2: EBS Volume Types
- **SSD-Backed (Solid State Drive):**
  - **General Purpose (gp3, gp2):** Balance of price and performance. Ideal for boot volumes, virtual desktops, and dev/test environments. `gp3` is the latest generation, allowing independent provisioning of IOPS and throughput. In `gp2`, IOPS are tied to volume size.
  - **Provisioned IOPS (io2 Block Express, io1):** Highest-performance SSDs for mission-critical, low-latency, or high-throughput workloads like large databases. You provision a specific number of IOPS.
- **HDD-Backed (Hard Disk Drive):**
  - **Throughput Optimized (st1):** Low-cost HDD for frequently accessed, throughput-intensive workloads like big data and log processing.
  - **Cold HDD (sc1):** Lowest-cost HDD for less frequently accessed data where cost savings are key.
- **Boot Volumes:** Only `gp2`, `gp3`, `io1`, and `io2` can be used as boot volumes.

## 10.3: EBS Operations
- **Resizing Volumes:** You can increase the size and modify the IOPS of a volume while it is in use. After increasing the size, you must extend the file system on the OS to use the new space. **You cannot decrease a volume's size directly.**
- **Snapshots:** Point-in-time backups of EBS volumes, stored in S3. They are the primary mechanism for backing up data and migrating volumes across AZs or regions.
  - **Encryption:** You can encrypt a snapshot during the copy process. A volume created from an encrypted snapshot will also be encrypted.
  - **Fast Snapshot Restore (FSR):** A feature that fully pre-warms a volume created from a snapshot, eliminating latency on the first access to each block. It is very expensive and billed per minute.
  - **Snapshot Archive:** A low-cost storage tier for snapshots that you rarely need to access. Restoring from the archive can take 24-72 hours.
  - **Recycle Bin:** Protects snapshots from accidental deletion by retaining them for a configured period (1 day to 1 year).
- **Data Lifecycle Manager (DLM):** Automates the creation, retention, and deletion of EBS snapshots and EBS-backed AMIs based on schedules and tags.

## 10.4: EC2 Instance Store
- **What is it?** A high-performance, temporary block-level storage that is physically attached to the host computer.
- **Performance:** Offers much better I/O performance than EBS because there is no network latency.
- **Ephemeral:** The data is **lost** if the instance is stopped or terminated. It is not persistent.
- **Use Cases:** Ideal for temporary storage of information that changes frequently, such as buffers, caches, scratch data, or other temporary content.

## 10.5: EFS (Elastic File System) Overview
- **EFS:** A managed, scalable file storage for use with EC2 instances and on-premises servers. It uses the Network File System (NFS) protocol.
- **Key Characteristics:**
  - **Shared Access:** Can be mounted on hundreds or thousands of instances simultaneously, even across different AZs.
  - **Regional Service:** The file system exists across multiple AZs within a region.
  - **Linux Only:** Compatible only with Linux-based AMIs.
  - **Elastic & Pay-per-use:** Scales automatically as you add and remove files, and you pay for the storage you use.

## 10.6: EFS vs. EBS
- **EBS:** One-to-one relationship with an instance (mostly), locked to a single AZ. You provision capacity in advance.
- **EFS:** Many-to-one relationship with instances, accessible across multiple AZs. Capacity is elastic.

## 10.7: EFS Features & Operations
- **Performance Modes:**
  - **General Purpose (Default):** For latency-sensitive use cases like web serving.
  - **Max I/O:** For high-throughput, highly parallel workloads like big data analytics.
- **Throughput Modes:**
  - **Bursting (Legacy):** Throughput scales with file system size.
  - **Provisioned:** You set a specific throughput level, independent of storage size.
  - **Elastic (Recommended):** Throughput scales automatically based on your workload's needs.
- **Storage Tiers & Lifecycle Management:** EFS can automatically move files that haven't been accessed for a set period (e.g., 30 days) to a lower-cost Infrequent Access (EFS-IA) or Archive storage class.
- **EFS Access Points:** Application-specific entry points into an EFS file system that make it easy to manage application access. You can enforce a specific user/group and restrict access to a specific directory within the file system.
- **Migration/Encryption:** To encrypt an unencrypted EFS file system or change its performance mode, you must create a new file system with the desired settings and migrate the data using a service like **AWS DataSync**.

## 10.8: EFS Monitoring
- **CloudWatch Metrics:** EFS provides several key metrics:
  - **`PercentIOLimit`:** For General Purpose mode, indicates how close you are to the I/O limit. If it's consistently high, you may need to switch to Max I/O mode.
  - **`BurstCreditBalance`:** For Bursting throughput mode, shows available credits for bursting to higher throughput.
  - **`StorageBytes`:** Shows the size of the file system, broken down by storage class (Standard vs. IA).
