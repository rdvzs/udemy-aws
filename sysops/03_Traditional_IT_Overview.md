
# Chapter 3: EC2 for SysOps

This chapter provides a SysOps-focused deep dive into Amazon EC2, covering instance management, networking, purchasing options, monitoring, and troubleshooting.

## Instance Management & Lifecycle

- **Changing Instance Type:** You can change the instance type (e.g., from `t2.micro` to `t2.small`) for vertical scaling. This requires the instance to be EBS-backed. The process is: **Stop** the instance, **change** the instance type, and then **start** the instance. The data on the EBS volume will be preserved, but the instance will likely move to new underlying hardware, resulting in a new public IP address.
- **Shutdown Behavior:** You can configure what happens when a shutdown is initiated from *within the OS*.
  - **Stop (Default):** The EC2 instance stops.
  - **Terminate:** The EC2 instance is terminated.
- **Termination Protection:** A setting that prevents accidental termination from the AWS Console or CLI. **Important:** This does *not* protect the instance if a shutdown is initiated from within the OS on an instance configured with a "terminate" shutdown behavior.
- **EC2 Hibernate:** An alternative to stopping an instance. The in-memory state (RAM) is saved to the root EBS volume. This allows for much faster instance startup, as the OS is not rebooted and applications can resume from where they left off. 
  - **Requirements:** The root EBS volume must be encrypted and large enough to store the RAM content. Hibernation is supported for up to 60 days.

## Networking & Placement

- **Enhanced Networking (ENA & EFA):**
  - **Elastic Network Adapter (ENA):** Provides higher bandwidth, higher packets per second (PPS), and lower latency for newer generation instances (e.g., T3 vs. T2). It can support up to 100 Gbps.
  - **Elastic Fabric Adapter (EFA):** An improved version of ENA specifically for High-Performance Computing (HPC) and tightly-coupled applications (Linux only). It bypasses the OS to provide lower latency and is ideal for inter-node communication (e.g., using MPI).
- **Placement Groups:** Control the physical placement of EC2 instances to influence performance and fault tolerance.
  - **Cluster:** Groups instances close together on a single rack within one AZ. Provides the lowest latency and highest throughput. Use for HPC and applications needing high network performance. *Risk: A single hardware failure can impact all instances.*
  - **Spread:** Spreads instances across distinct underlying hardware (different racks). Minimizes simultaneous failures. Use for critical applications needing high availability. *Limitation: Max 7 instances per AZ per placement group.*
  - **Partition:** Spreads instances across multiple partitions (logical groups of racks) within one or more AZs. A failure in one partition does not affect others. Allows for hundreds of instances. Use for large, distributed, and partition-aware workloads like Hadoop, Cassandra, and Kafka.

## Purchasing Options & Cost Optimization

- **On-Demand:** Pay-as-you-go, highest cost, no commitment. For short-term, unpredictable workloads.
- **Reserved Instances (RI):** 1 or 3-year commitment for a specific instance type. Significant discount. For steady-state workloads like databases.
- **Savings Plans:** 1 or 3-year commitment to a certain amount of spend ($/hour). More flexible than RIs.
- **Spot Instances:** Up to 90% discount, but can be terminated with a 2-minute warning if AWS needs the capacity back. Ideal for fault-tolerant, non-critical workloads like batch jobs and data analysis.
  - **Spot Fleet:** A collection of Spot Instances (and optionally On-Demand) to meet a target capacity. Strategies like `lowestPrice` or `diversified` help optimize for cost or availability.
  - **Termination:** To permanently terminate Spot Instances, you must **first cancel the Spot Request**, and then terminate the instances.
- **Dedicated Hosts:** A physical server dedicated to your use. For compliance or complex licensing models (BYOL).
- **Capacity Reservations:** Reserve On-Demand capacity in a specific AZ for any duration. Ensures you can launch instances when needed, but you pay for it whether you use it or not.

## Monitoring & Troubleshooting

- **CloudWatch Metrics for EC2:** AWS provides default metrics every 5 minutes (or 1 minute with Detailed Monitoring).
  - **Key Metrics:** CPU Utilization, Network In/Out, Status Checks, Disk I/O (for Instance Store only).
  - **RAM is NOT a default metric.** It must be collected as a custom metric.
- **Unified CloudWatch Agent:** An agent installed on EC2 or on-premises servers to collect additional system-level metrics (like RAM, disk space, individual processes via `procstat`) and custom application logs, and send them to CloudWatch.
- **Instance Status Checks:**
  - **System Status Check:** Monitors the underlying AWS hardware and software. A failure requires an instance **stop/start** to move it to new hardware. A CloudWatch alarm can be set to automatically **recover** the instance.
  - **Instance Status Check:** Monitors the software and network configuration on your specific instance. A failure may require a **reboot** or OS-level troubleshooting.
- **Troubleshooting Launch Issues:**
  - **Instance Limit Exceeded:** You've reached the vCPU limit for your account in a region. Request a limit increase or launch in a different region.
  - **Insufficient Instance Capacity:** AWS does not have enough On-Demand capacity in the chosen AZ. Wait, try a different instance type, or try a different AZ.
  - **Instance Terminates Immediately:** Could be due to EBS volume limits, a corrupt EBS snapshot, or KMS key permission issues for an encrypted root volume.
- **Troubleshooting SSH Issues:**
  - **Connection Timed Out:** A network issue. Check Security Group rules (is port 22 open?), Network ACLs, and route tables. Also, ensure the instance has a public IP and isn't overloaded (CPU at 100%).
  - **Permission Denied / Unprotected Private Key:** Incorrect file permissions on your `.pem` key file (use `chmod 400`) or using the wrong username for the OS (e.g., `ec2-user` for Amazon Linux, `ubuntu` for Ubuntu).

## IP & Other Features

- **Elastic IPs (EIP):** A fixed, public IPv4 address you own. You can attach it to an EC2 instance to provide a static public IP. Useful for remapping an IP to a new instance during a failure. You are charged for EIPs when they are *not* attached to a running instance.
- **Burstable Instances (T-family):** These instances have a baseline CPU performance and can "burst" to a higher performance level using CPU credits. If credits are depleted, performance is throttled to the baseline. `T2/T3 Unlimited` mode allows sustained high performance but may incur extra charges.
