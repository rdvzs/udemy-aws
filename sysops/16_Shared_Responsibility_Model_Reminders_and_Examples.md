# Chapter 16: Databases for SysOps

## Amazon Relational Database Service (RDS)
RDS is a managed relational database service supporting SQL. It automates provisioning, OS patching, continuous backups (Point-in-Time Restore - PITR), monitoring, and scaling (vertical and horizontal via Read Replicas). It does not provide SSH access to instances.
-   **Supported Engines:** PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, IBM DB2, Aurora.
-   **Storage Auto Scaling:** Automatically increases storage when free space is low (e.g., <10% remaining, lasting >5 minutes, 6 hours since last modification). Useful for unpredictable workloads.
-   **Read Replicas:**
    -   Scale reads (up to 15 replicas).
    -   Can be within the same AZ, cross-AZ, or cross-region.
    -   Asynchronous replication (eventually consistent).
    -   Can be promoted to standalone databases.
    -   Replication traffic is free within the same region (cross-AZ), but incurs cost for cross-region.
    -   Used for offloading read-intensive workloads (e.g., reporting, analytics) from the primary database.
-   **Multi-AZ Deployments:**
    -   Primarily for Disaster Recovery (DR) and High Availability (HA).
    -   Synchronous replication to a standby instance in a different AZ.
    -   Single DNS name for the application.
    -   Automatic failover to standby in case of primary DB failure (instance, storage, network, AZ outage, OS patching, manual reboot with failover).
    -   Not used for scaling reads.
    -   Converting from Single-AZ to Multi-AZ is a zero-downtime operation.
-   **Parameter Groups:** Customize database engine settings.
    -   **Dynamic parameters:** Applied immediately.
    -   **Static parameters:** Applied after DB instance reboot.
    -   Can force SSL connections (e.g., `rds.force_ssl=1` for PostgreSQL/SQL Server, `require_secure_transport=1` for MySQL/MariaDB).
-   **Backups and Snapshots:**
    -   **Automated Backups:** Continuous, enable PITR, retention 1-35 days (cannot be disabled, setting to 0 disables). Restore creates a *new* DB instance.
    -   **Manual Snapshots:** User-initiated, take IO operations (can cause brief downtime), incremental after first full snapshot, do not expire. Restore creates a *new* DB instance.
    -   **Sharing Snapshots:** Manual snapshots can be shared across accounts. Encrypted snapshots require sharing the KMS CMK.
-   **Events and Logs:**
    -   **Events:** RDS records events (DB state changes, snapshots, parameter group changes). Can be sent to SNS topics or EventBridge for notifications.
    -   **Logs:** Database logs (general, audit, error, slow query) can be exported to CloudWatch Logs for long-term retention, analysis, and alerting (e.g., metric filters and alarms for errors).
-   **CloudWatch Metrics:** Provides basic metrics from the hypervisor (e.g., `DatabaseConnections`, `ReadIOPS`, `WriteLatency`, `FreeStorageSpace`).
-   **Enhanced Monitoring:** Gathers more detailed metrics from an agent on the DB instance (e.g., process/thread CPU/memory usage, disk I/O). Configurable granularity (up to 1 second).
-   **Performance Insights:** Visualizes database load, helps analyze performance issues by filtering by Waits (CPU, IO, locks), SQL statements, hosts, and users.

## Amazon Aurora
AWS proprietary, cloud-optimized relational database compatible with MySQL and PostgreSQL. Offers 5x MySQL performance, 3x PostgreSQL performance.
-   **Storage:** Automatically grows from 10GB to 128TB. Stores six copies of data across 3 AZs for high availability (4/6 copies for writes, 3/6 for reads). Self-healing, auto-expanding.
-   **Read Replicas:** Up to 15 read replicas, sub-10ms replica lag. Can be auto-scaled.
-   **Failover:** Instantaneous failover (much faster than RDS Multi-AZ).
-   **Endpoints:**
    -   **Writer Endpoint:** Always points to the current master instance (for writes).
    -   **Reader Endpoint:** Connects to one of the read replicas for load balancing reads.
-   **Backtracking:** Rewind database to any point in time (up to 72 hours) without creating a new cluster (in-place restore). Supports Aurora MySQL only.
-   **Database Cloning:** Creates a new DB cluster using the same DB cluster volume as the original (copy-on-write). Fast and easy for test environments.
-   **Security:**
    -   **Encryption at Rest:** Encrypted using KMS (defined at launch). Unencrypted masters cannot have encrypted replicas. To encrypt existing unencrypted DB, snapshot and restore as encrypted.
    -   **Encryption in Transit:** Supported by default using TLS root certificates.
    -   **Authentication:** Username/password or IAM roles.
    -   **Network Access:** Controlled by Security Groups.
    -   **No SSH access** (except RDS Custom).
    -   **Audit Logs:** Can be enabled and sent to CloudWatch Logs.
-   **SysOps Specifics:**
    -   **Priority Tiers:** Assign 0-15 priority to Read Replicas for failover control (lowest tier promoted first).
    -   **Migration:** Can migrate RDS MySQL snapshot to Aurora MySQL.
    -   **CloudWatch Metrics:** `AuroraReplicaLag` (lag between primary and replica), `DatabaseConnections`, `InsertLatency`.

## Amazon ElastiCache
Managed in-memory cache service (Redis, Memcached). Reduces database load for read-intensive workloads and makes applications stateless by storing user sessions. Requires application code changes.
-   **Redis:**
    -   **Cluster Mode Disabled:** One primary node, up to 5 read replicas (for HA and read scaling). Replication is asynchronous. Horizontal scaling by adding/removing replicas. Vertical scaling by changing node type (creates new node group, data replicated, DNS updated).
    -   **Cluster Mode Enabled:** Data partitioned across multiple shards (for write scaling). Each shard has a primary and up to 5 replicas. Supports auto-scaling (increase/decrease shards/replicas based on metrics like CPU utilization).
    -   **Endpoints:**
        -   **Cluster Mode Disabled:** Primary endpoint (writes), Reader endpoint (reads, load balanced), Node endpoints (individual nodes).
        -   **Cluster Mode Enabled:** Configuration Endpoint (for all read/write operations, requires compatible client).
    -   **Features:** Multi-AZ with auto-failover, read replicas, data durability (AOF persistence), backup/restore, sets/sorted sets.
    -   **Metrics:** `Evictions` (non-expired items removed), `CPUUtilization`, `SwapUsage`, `CurrentConnections`, `DatabaseMemoryUsagePercentage`, `NetworkIn/Out`, `ReplicationBytes`, `ReplicationLag`.
-   **Memcached:**
    -   Multiple nodes (up to 40) for sharding data.
    -   No built-in HA or replication (data loss on node failure).
    -   Backup/restore only for serverless version.
    -   Multi-threaded architecture.
    -   **Scaling:**
        -   **Horizontal:** Add/remove nodes (auto-discovery updates clients).
        -   **Vertical:** Create new cluster with desired node type, update application endpoints, delete old cluster (manual process, data loss).
    -   **Metrics:** Similar to Redis: `Evictions`, `CPUUtilization`, `SwapUsage`, `CurrConnections`, `FreeableMemory`.

## Amazon DynamoDB
Fully managed, highly available, scalable NoSQL (key-value and document) database. Used for low-latency, high-throughput applications (mobile, gaming, IoT).
-   **Schema:** Flexible, uses tables, items, and attributes.
-   **Primary Key:** Simple (partition key) or Composite (partition key + sort key).
-   **Capacity Modes:**
    -   **On-Demand Capacity:** Pay-per-request, for unpredictable workloads.
    -   **Provisioned Capacity:** Provision RCU/WCU, for predictable workloads, more cost-effective. Supports Auto Scaling to adjust capacity based on traffic.

## Amazon Redshift
Fully managed, petabyte-scale, columnar data warehousing service. Used for business intelligence, reporting, and analytics.
-   **Node Types:**
    -   **Dense Compute (DC):** Optimized for compute-intensive workloads (high-performance analytics).
    -   **Dense Storage (DS):** Optimized for storage-intensive workloads (large datasets).
-   **Scaling:** Scale by adding/removing nodes. Resize by changing node type or number of nodes.

## Amazon Neptune
Fully managed, highly available, scalable graph database service. Used for applications requiring relationships between data (social networking, recommendation engines, fraud detection, knowledge graphs).
-   **Schema:** Nodes and edges with properties.
-   **Query Languages:** Gremlin (graph traversal) or SPARQL (graph query).
-   **Scaling:** Scale by adding/removing instances. Resize by changing instance type or number of instances.