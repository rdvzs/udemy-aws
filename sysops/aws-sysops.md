
# Chapter 1: Introduction & Requirements (AWS Certified SysOps Administrator Associate)

This introductory chapter outlines the structure of the AWS Certified SysOps Administrator Associate (SOA-C02) course, introduces the instructor, and provides guidance on how to best utilize the Udemy platform for learning.

## 1.1: Course Presentation (SOA-C02)
- **Course Goal:** To prepare for the AWS Certified SysOps Administrator Associate (SOA-C02) exam.
- **Difficulty:** This certification is considered the most challenging of the AWS associate-level exams.
- **Prerequisites:** It is highly recommended that students complete either the Solutions Architect Associate (SAA) or Developer Associate (DVA) certification before taking this course, as it is structured as an advanced course requiring foundational knowledge.
- **Course Structure:**
  - The course includes videos imported from other courses by the same instructor to cover base knowledge.
  - Lectures will be tagged to indicate their origin:
    - `[CCP]`: Certified Cloud Practitioner
    - `[SAA]`: Solutions Architect Associate
    - `[DVA]`: Developer Associate
  - Students who have taken these previous courses can skip the tagged videos they have already seen.
  - Videos without tags are new and specific to the SysOps (SOA) course.

## 1.2: About Your Instructor
- **Instructor:** Stephane Maarek, an experienced instructor with all 11 AWS certifications and over seven years of teaching experience on Udemy.
- **Expertise:** His background includes roles as a data analyst, big data engineer, developer, and solutions architect, with expertise in AWS and Apache Kafka.
- **Connect with the Instructor:**
  - **LinkedIn:** For AWS news, course announcements, and celebrating student successes.
  - **Instagram:** To see stories of other students and get a chance to meet the instructor during his travels.
- **Goal Setting:** The instructor encourages students to set a clear goal before starting (e.g., finishing the course in three weeks) to stay motivated and maximize learning.

## 1.3: Important Message (Udemy Platform Features)
- **Playback Speed:** You can adjust the video playback speed (from 0.5x to 2x) to match your learning pace.
- **Subtitles & Transcripts:** The course includes professional subtitles (captions) and transcripts to aid understanding.
- **Feedback:** The instructor emphasizes the importance of student ratings and feedback for the course and encourages students to leave a rating when they feel ready.



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



# Chapter 4: AMI (Amazon Machine Image)

This chapter focuses on Amazon Machine Images (AMIs), covering their creation, management, and use in production environments. It also details EC2 Image Builder for automation and strategies for instance migration and sharing AMIs.

## 4.1: AMI Overview
- **AMI (Amazon Machine Image):** A template that contains the software configuration (operating system, application server, and applications) required to launch your instance.
- **Benefits:** Using a custom AMI results in faster boot and configuration times because all necessary software is pre-packaged.
- **AMI Sources:**
  - **Public AMIs:** Provided and maintained by AWS (e.g., Amazon Linux 2).
  - **Your own AMIs:** Custom images you create and maintain.
  - **AWS Marketplace AMIs:** Pre-configured images from third-party vendors, which may include commercial software.
- **Creation Process:** You typically launch an instance from a public AMI, customize it (install software, apply patches), and then create a new custom AMI from that instance. This process also creates underlying EBS Snapshots.

## 4.2: AMI Hands-On
- **Process:**
  1.  Launch a standard EC2 instance (e.g., Amazon Linux 2).
  2.  Use a User Data script to install and start a web server (like Apache HTTPD) but without creating a final `index.html` file. This simulates pre-installing common software.
  3.  Once the instance is running and the software is installed, create an image from it via the AWS Console (`Actions` > `Image and templates` > `Create image`).
  4.  Launch a new instance *from your custom AMI*. This new instance will boot up much faster with the web server already installed.
  5.  Provide a new User Data script to the second instance to perform final configurations, like creating the `index.html` file.
- **Result:** This demonstrates how AMIs can significantly speed up the deployment of new, pre-configured instances.

## 4.3: AMI No-Reboot Option
- **Default Behavior:** By default, when creating an AMI from a running instance, AWS will first **reboot** the instance to ensure file system integrity before taking snapshots.
- **No-Reboot Option:** You can choose to create an AMI *without* rebooting the instance. This is faster and less disruptive but carries a risk.
  - **Risk:** The file system integrity of the created image is not guaranteed, as in-memory data and OS buffers may not be flushed to disk.
- **AWS Backup:** The AWS Backup service uses the `No-Reboot` option by default when creating AMIs as part of a backup plan. If file system consistency is critical, a custom solution (e.g., a scheduled Lambda function that reboots the instance before creating the AMI) might be necessary.

## 4.4: EC2 Instance Migration and Cross-Account Sharing
- **Migrating an Instance to another AZ:** The standard method is to create an AMI from the instance, and then launch a new instance from that AMI in the desired Availability Zone.
- **Cross-Account AMI Sharing:** You can share your custom AMIs with other AWS accounts.
  - **Unencrypted AMIs:** Can be shared easily with other accounts or even made public.
  - **Encrypted AMIs:** If the AMI's root volume is encrypted with a **customer-managed KMS key (CMK)**, you must also share the KMS key with the target account to allow them to launch instances from it.
- **Cross-Account AMI Copying:** A user in another account can *copy* an AMI that has been shared with them. This creates a new, independent AMI in their own account, of which they are the owner. This is useful for isolating dependencies and managing encryption with their own keys.

## 4.5: EC2 Image Builder
- **EC2 Image Builder:** A fully managed AWS service that automates the creation, management, validation, and testing of Linux and Windows AMIs.
- **Workflow:**
  1.  **Build:** Launches a temporary EC2 instance to install and configure software based on a defined "recipe" and components.
  2.  **Test:** Launches a second instance from the newly created AMI to run tests you define, ensuring the image is secure and functional.
  3.  **Distribute:** Distributes the validated AMI to specified AWS regions.
- **Benefits:**
  - **Automation:** Simplifies the creation of "golden AMIs."
  - **Scheduling:** Can be run on a schedule (e.g., weekly) or triggered by events to keep images up-to-date with the latest patches.
  - **Cost:** The service itself is free; you only pay for the underlying resources used during the build and test process (e.g., EC2 instances, S3 storage).

## 4.6: Using AMIs in Production
- **Enforcing Approved AMIs:** You can use IAM policies to restrict users to launching EC2 instances only from specific, pre-approved AMIs. This is typically done by:
  1.  Tagging your approved AMIs with a specific tag (e.g., `Environment=Prod`).
  2.  Creating an IAM policy with a `Condition` that checks for this tag on the AMI during the `ec2:RunInstances` action.
- **Auditing with AWS Config:** You can use AWS Config rules to continuously monitor your environment and identify any running EC2 instances that were launched from non-approved AMIs, flagging them as non-compliant.

## 4.7: AMI Section Cleanup
- To avoid incurring costs after completing the hands-on labs, it's important to clean up resources:
  1.  **Terminate** all running EC2 instances.
  2.  **Deregister** any custom AMIs you created.
  3.  **Delete** the EBS snapshots associated with the deregistered AMIs.



# Chapter 5: Managing EC2 at Scale (Systems Manager - SSM)

This chapter provides a comprehensive look at AWS Systems Manager (SSM), a suite of tools designed to help you manage and automate your fleet of EC2 instances and on-premises servers at scale.

## 5.1: Systems Manager Overview
- **AWS Systems Manager (SSM):** A collection of tools for gaining operational insights and taking action on your AWS resources. It helps with patching, configuration management, and automation for EC2 and on-premises systems (making it a hybrid service).
- **Core Requirement:** The **SSM Agent** must be installed and running on any machine you want to manage. It comes pre-installed on Amazon Linux 2 and some Ubuntu AMIs. The instance also needs an appropriate **IAM Role** (`AmazonSSMManagedInstanceCore` policy) to communicate with the SSM service.
- **Key Features:** The chapter focuses on several key features, including Resource Groups, Run Command, Automation, Parameter Store, Patch Manager, and Session Manager.

## 5.2: Starting EC2 Instances with SSM Agent
- To register instances with SSM, they must have the SSM Agent installed and an IAM role with the `AmazonSSMManagedInstanceCore` policy attached.
- Once an instance is properly configured and running, it will automatically appear as a "managed node" in the **SSM Fleet Manager**.
- **Security Benefit:** Instances do not need any open inbound ports (like SSH port 22) to be managed by SSM. The agent initiates an outbound connection to the SSM service.

## 5.3: AWS Tags & SSM Resource Groups
- **Tags:** Key-value pairs used to label and organize AWS resources. They are essential for automation, security, and cost allocation.
- **Resource Groups:** A way to group AWS resources based on shared tags. This allows you to view and act on a collection of resources as a single unit.
- **Use Case:** You can create resource groups for different environments (e.g., `dev`, `prod`) or teams (e.g., `finance`, `operations`). These groups can then be targeted by SSM operations, allowing you to, for example, run a command only on your `dev` instances.

## 5.4: SSM Documents & Run Command
- **SSM Documents:** JSON or YAML files that define the actions SSM performs. AWS provides many pre-configured public documents, or you can create your own.
- **SSM Run Command:** Allows you to execute a command or an SSM Document on one or more managed instances.
  - **No SSH Needed:** The command is executed via the SSM Agent, not through an open SSH port.
  - **Targets:** Can target instances by ID, tags, or Resource Groups.
  - **Rate Control:** You can control the execution concurrency (e.g., run on one instance at a time) and set an error threshold to stop the command if too many failures occur.
  - **Output:** Command output can be viewed in the console or logged to S3 and CloudWatch Logs.

## 5.5: SSM Automations
- **SSM Automation:** Used to simplify and automate common maintenance and deployment tasks for AWS resources (not just EC2 instances).
- **Runbooks:** The SSM Documents used for automation are often called Runbooks. They can perform a series of steps, such as restarting an instance, creating an AMI, or taking an EBS snapshot.
- **Automation vs. Run Command:** Run Command executes scripts *inside* an instance, while Automation orchestrates actions *on* AWS resources (e.g., calling AWS APIs to stop/start an instance).
- **Triggers:** Automations can be triggered manually, on a schedule (via EventBridge or Maintenance Windows), or as a remediation action for an AWS Config rule.

## 5.6: SSM Parameter Store
- **SSM Parameter Store:** A service for secure, hierarchical storage for configuration data and secrets.
- **Features:**
  - **Hierarchy:** Parameters can be organized into paths (e.g., `/my-app/dev/db-password`), which simplifies management and IAM permissions.
  - **Parameter Types:** `String`, `StringList`, and `SecureString`. `SecureString` parameters are encrypted using AWS KMS.
  - **Versioning:** Tracks changes to parameters over time.
  - **Tiers:** A free **Standard** tier and a paid **Advanced** tier, which allows for larger parameter sizes and parameter policies (e.g., setting an expiration date for a password).
- **Access:** Parameters can be retrieved via the AWS CLI or SDK. The `--with-decryption` flag is needed to retrieve the plaintext value of `SecureString` parameters.

## 5.7: SSM Inventory & State Manager
- **SSM Inventory:** Collects metadata from your managed instances, such as installed applications, OS versions, network configurations, and more.
- **State Manager:** An automation service that helps you keep your managed instances in a defined state. You create an **Association** that specifies the desired state (defined by an SSM Document) and the target instances. State Manager will then automatically apply this configuration on a schedule you define.
- **Use Case:** Ensuring that a specific set of software is always installed, antivirus is running, or ports are closed.

## 5.8: SSM Patch Manager & Maintenance Windows
- **SSM Patch Manager:** Automates the process of patching managed instances for both operating systems and applications.
- **Patch Baselines:** Define which patches are approved for installation. You can use AWS-managed default baselines or create custom ones to approve or reject specific patches.
- **Patch Groups:** A way to associate a group of instances (using a specific tag) with a particular patch baseline. For example, you can have a `dev` patch group that gets patches immediately and a `prod` group that gets them after a 7-day delay.
- **Maintenance Windows:** Define a recurring schedule (e.g., every Sunday from 2 AM to 4 AM) during which you can perform disruptive actions like patching or software installations on your instances without impacting production traffic.

## 5.9: SSM Session Manager
- **SSM Session Manager:** Provides secure and auditable instance management without needing to open inbound ports (like SSH), manage bastion hosts, or handle SSH keys.
- **How it Works:** It provides a browser-based shell or a CLI-based connection to your instances through a secure tunnel via the SSM service.
- **Benefits:**
  - **Enhanced Security:** No open ports reduce the attack surface.
  - **Auditing:** All session activity can be logged to CloudWatch Logs or S3 for full auditability.
  - **IAM Control:** Access is controlled entirely through IAM policies, allowing for granular control over which users can access which instances.



# Chapter 6: EC2 High Availability and Scalability

This chapter focuses on the core concepts and services used to build scalable, elastic, and highly available applications on AWS, primarily centered around Elastic Load Balancing (ELB) and Auto Scaling Groups (ASG).

## 6.1: High Availability and Scalability Concepts
- **Scalability:** An application's ability to handle greater load.
  - **Vertical Scalability:** Increasing the size of a single instance (e.g., `t2.micro` to `t2.large`). Common for non-distributed systems like databases (RDS, ElastiCache). It has hardware limits.
  - **Horizontal Scalability (Elasticity):** Increasing the number of instances (scaling out/in). This is the cloud-native approach, ideal for web applications.
- **High Availability (HA):** Ensuring an application remains operational even if a data center fails. This is achieved by running instances across at least two Availability Zones (AZs).

## 6.2: Elastic Load Balancing (ELB) Overview
- **ELB:** A managed AWS service that distributes incoming application traffic across multiple downstream targets, such as EC2 instances.
- **Benefits:** Spreads load, provides a single point of access (DNS name), handles failures by routing traffic away from unhealthy instances, and can be used for SSL/TLS termination.
- **Types of Load Balancers:**
  - **Application Load Balancer (ALB):** (Layer 7) For HTTP/HTTPS traffic. Supports advanced, content-based routing rules.
  - **Network Load Balancer (NLB):** (Layer 4) For TCP/UDP/TLS traffic. Offers ultra-high performance and static IP addresses per AZ.
  - **Gateway Load Balancer (GWLB):** (Layer 3) For routing traffic to third-party virtual network appliances like firewalls and intrusion detection systems.
  - **Classic Load Balancer (CLB):** Old generation, now deprecated.

## 6.3: Application Load Balancer (ALB) Deep Dive
- **Target Groups:** ALBs route traffic to targets (EC2 instances, Lambda functions, IP addresses) organized into target groups.
- **Routing Rules:** ALBs can route traffic based on various conditions:
  - **Path:** `example.com/users` vs. `example.com/posts`.
  - **Hostname:** `one.example.com` vs. `two.example.com`.
  - **Query Strings & Headers:** `?platform=mobile`.
- **Use Case:** Ideal for microservices and container-based applications (ECS) due to its flexible routing capabilities.
- **Client IP:** The original client IP is not seen by the backend instances; it's passed in the `X-Forwarded-For` header.

## 6.4: Network Load Balancer (NLB) Deep Dive
- **Use Case:** Used for extreme performance requirements (millions of requests/sec), low latency, and when a static IP address is needed.
- **Static IP:** Provides one static IP per enabled AZ. You can also assign your own Elastic IPs.
- **Target Groups:** Can target EC2 instances or private IP addresses (for on-premises servers).
- **Health Checks:** Supports TCP, HTTP, and HTTPS health checks.

## 6.5: Gateway Load Balancer (GWLB) Deep Dive
- **Function:** Acts as a transparent network gateway, allowing you to insert third-party virtual appliances (firewalls, IDS/IPS) into the network path without altering the traffic flow.
- **How it Works:** All traffic to and from a VPC is routed through the GWLB, which then distributes it to a fleet of virtual appliances for inspection. It uses the GENEVE protocol on port 6081.

## 6.6: ELB Advanced Features
- **Sticky Sessions (Session Affinity):** Enables the load balancer to bind a user's session to a specific target instance. This ensures that all requests from the user during the session are sent to the same instance, which is useful for applications that store session data locally. Can be enabled for ALB, NLB, and CLB.
- **Cross-Zone Load Balancing:**
  - **ALB:** Enabled by default. Traffic is distributed evenly across all registered instances in all enabled AZs. No charge for inter-AZ data transfer.
  - **NLB/GWLB:** Disabled by default. If enabled, you are charged for inter-AZ data transfer.
- **SSL/TLS Certificates:** Load balancers can terminate SSL/TLS traffic. You can attach SSL certificates (managed via AWS Certificate Manager - ACM) to your listeners.
  - **Server Name Indication (SNI):** Allows you to host multiple SSL-secured websites behind a single load balancer (ALB/NLB) by loading multiple SSL certificates. The client specifies the target hostname in the initial SSL handshake, allowing the ELB to present the correct certificate.
- **Connection Draining (Deregistration Delay):** When an instance is being deregistered or becomes unhealthy, this feature keeps existing connections open to allow in-flight requests to complete, while routing new requests to other healthy instances. The timeout is configurable (default 300 seconds).

## 6.7: Auto Scaling Groups (ASG) Overview
- **ASG:** Automatically adjusts the number of EC2 instances in a group to meet demand.
- **Core Functions:**
  - **Scale Out/In:** Adds (scales out) or removes (scales in) instances based on load.
  - **Maintain Capacity:** Ensures a minimum number of instances are always running.
  - **Health Checks:** Replaces instances that fail EC2 or ELB health checks.
- **Configuration:** Requires a **Launch Template** (or legacy Launch Configuration) that defines the instance parameters (AMI, instance type, security groups, etc.).

## 6.8: ASG Scaling Policies
- **Dynamic Scaling:**
  - **Target Tracking:** The most common and simple policy. You set a target for a metric (e.g., average CPU utilization at 40%), and the ASG automatically adjusts capacity to maintain that target.
  - **Simple/Step Scaling:** Scales based on CloudWatch alarm thresholds (e.g., if CPU > 70%, add 2 instances).
- **Scheduled Scaling:** Changes the ASG size at predictable, scheduled times (e.g., increase capacity every Friday at 5 PM).
- **Predictive Scaling:** Uses machine learning to forecast future load and proactively schedule scaling actions.
- **Scaling Cooldown:** A period (default 300 seconds) after a scaling activity during which the ASG will not initiate another scaling action. This allows metrics to stabilize.

## 6.9: ASG for SysOps
- **Lifecycle Hooks:** Allow you to perform custom actions as instances are launched or terminated by the ASG. For example, you can pause an instance before it enters service to run a setup script, or pause it before termination to extract logs.
- **Health Checks:** ASGs can use `EC2` status checks (default) and/or `ELB` health checks to determine instance health. Unhealthy instances are terminated and replaced.
- **Custom Health Checks:** You can use the `SetInstanceHealth` API call to manually report the health of an instance to the ASG.



# Chapter 7: Elastic Beanstalk for SysOps

This chapter covers AWS Elastic Beanstalk, a Platform as a Service (PaaS) offering that simplifies the deployment and management of web applications. It automates the underlying infrastructure, allowing developers to focus on their code.

## 7.1: Elastic Beanstalk Overview
- **What is Elastic Beanstalk?** A developer-centric service for deploying and scaling web applications and services. It's a PaaS that automates the setup of an entire application stack, including EC2 instances, Auto Scaling Groups, Elastic Load Balancers, and optionally RDS databases.
- **Developer Focus:** The primary responsibility of the developer is to provide the application code. Beanstalk handles capacity provisioning, load balancing, scaling, and application health monitoring.
- **Managed Service:** While Beanstalk is a managed service, you still retain full control over the configuration of the underlying AWS resources it creates.
- **Cost:** The Beanstalk service itself is free; you only pay for the AWS resources it provisions (e.g., EC2 instances, ELB, RDS).

### Key Concepts
- **Application:** A logical collection of Beanstalk components, including environments, versions, and configurations.
- **Application Version:** A specific, labeled iteration of your application code.
- **Environment:** A running version of your application. You can have multiple environments, such as `dev`, `test`, and `prod`.
- **Environment Tiers:**
  - **Web Server Environment:** For handling HTTP(S) requests. This is the standard tier for web applications and includes a load balancer and auto-scaling group.
  - **Worker Environment:** For processing background tasks. It pulls messages from an Amazon SQS queue and processes them asynchronously.

### Deployment Modes
- **Single Instance:** Deploys the application on a single EC2 instance with an Elastic IP. Ideal for development and testing environments. It is **not** highly available.
- **High Availability (with Load Balancer):** Deploys the application across multiple EC2 instances in an Auto Scaling Group, fronted by an Application Load Balancer. This is the standard for production environments, providing scalability and fault tolerance.

## 7.2: Elastic Beanstalk Hands-On
- **Creating an Application:** The process starts by creating a new application in the Beanstalk console.
- **Creating an Environment:**
  1.  **Choose Environment Tier:** Select either a "Web server environment" or a "Worker environment".
  2.  **Application Information:** Name the application and the environment (e.g., `MyApplication-dev`).
  3.  **Platform:** Select the technology stack for your application (e.g., Node.js, Python, Go, Docker).
  4.  **Application Code:** Upload your code or use the provided sample application.
  5.  **Presets:** Choose a configuration preset, such as "Single instance" (Free Tier eligible) or "High availability".
  6.  **Service Access:** Configure IAM roles. Beanstalk requires a **Service Role** to manage resources on your behalf and an **EC2 Instance Profile** to grant permissions to the instances it launches.
- **Behind the Scenes:** When you create an environment, Elastic Beanstalk uses **AWS CloudFormation** in the background to provision all the necessary resources (EC2 instances, Security Groups, Auto Scaling Groups, etc.). You can view the events and the generated template in the CloudFormation console.
- **Accessing the Application:** Once the environment is successfully launched, Beanstalk provides a public URL (e.g., `myapplication-dev.elasticbeanstalk.com`) to access the application.
- **Management:** The Beanstalk console provides a centralized dashboard to monitor health, view logs, manage application versions, and configure the environment's underlying resources.



# Chapter 8: CloudFormation for SysOps

This chapter offers a deep dive into AWS CloudFormation from a SysOps perspective, focusing on template structure, deployment strategies, and advanced features for managing infrastructure as code (IaC).

## 8.1: CloudFormation Overview
- **Infrastructure as Code (IaC):** CloudFormation is a declarative way to model, provision, and manage your entire AWS infrastructure using code (YAML or JSON). This allows for version control, peer review, and automation.
- **Benefits:**
  - **Automation & Control:** Eliminates manual setup and ensures consistency.
  - **Productivity:** Quickly create, update, and delete entire environments (called Stacks).
  - **Cost Management:** Resources are tagged, and costs can be estimated before deployment.
- **Process:** You write a template, upload it to S3, and CloudFormation creates a **Stack** by provisioning the resources you defined in the correct order.

## 8.2: CloudFormation Template Structure
- **YAML:** The preferred format for its readability. It uses key-value pairs, indentation for nested objects, and hyphens for lists.
- **Core Sections:**
  - **`Resources` (Required):** The only mandatory section. Defines the AWS resources to be created (e.g., `AWS::EC2::Instance`, `AWS::S3::Bucket`).
  - **`Parameters`:** Defines dynamic inputs for your template, allowing for customization at deployment time (e.g., specifying an instance type).
  - **`Mappings`:** A collection of fixed key-value pairs. Useful for selecting values based on a key like the AWS Region (e.g., choosing the correct AMI ID for `us-east-1` vs. `eu-west-1`).
  - **`Outputs`:** Declares output values that you can view or import into other stacks (e.g., exporting a VPC ID).
  - **`Conditions`:** Controls whether certain resources are created or properties are assigned based on a condition (e.g., create a resource only if the environment is `prod`).

## 8.3: Intrinsic Functions (Template Helpers)
- **`!Ref`:** References the value of a parameter or the physical ID of a resource.
- **`!GetAtt`:** (Get Attribute) Fetches an attribute from a resource (e.g., `!GetAtt MyEC2Instance.PublicIp`).
- **`!FindInMap`:** Retrieves a value from a `Mappings` section.
- **`!ImportValue`:** Imports a value that has been exported from another stack's `Outputs`.
- **`!Join`:** Joins a list of values with a delimiter.
- **`!Sub`:** Substitutes variables in a string.
- **Condition Functions:** `!And`, `!Equals`, `!If`, `!Not`, `!Or`.

## 8.4: Instance Initialization (User Data & cfn-init)
- **`UserData`:** A property on an EC2 instance resource used to pass a script that runs on the instance's first boot. The script must be Base64 encoded.
- **`cfn-init` and `AWS::CloudFormation::Init`:** A more structured and powerful way to configure instances.
  - You define a configuration (packages to install, files to create, services to start) in the `Metadata` section of the EC2 resource, under `AWS::CloudFormation::Init`.
  - The `UserData` script then simply calls the `cfn-init` helper script, which reads the metadata and applies the configuration.
  - This is more readable and maintainable than a long bash script in `UserData`.
- **`cfn-signal` and `WaitCondition`:** A mechanism to signal the success or failure of an instance configuration back to CloudFormation.
  - The `UserData` script runs `cfn-init` and then uses `cfn-signal` to send the exit code (0 for success) to a `WaitCondition` resource.
  - If the `WaitCondition` does not receive the required number of success signals within a timeout period, it fails the stack creation, triggering a rollback. This ensures that a failed bootstrap process doesn't result in a "successful" but broken stack.

## 8.5: Stack Management & Policies
- **Rollbacks:**
  - **On Create Failure:** By default, if stack creation fails, all created resources are deleted (rolled back). You can disable this for troubleshooting.
  - **On Update Failure:** The stack automatically rolls back to the last known good state.
- **`DependsOn` Attribute:** Explicitly defines a creation dependency. For example, you can force an S3 bucket to be created only *after* an EC2 instance is created. CloudFormation infers most dependencies (via `!Ref`), but `DependsOn` is for explicit ordering.
- **`DeletionPolicy`:** Controls what happens to a resource when it's removed from a stack.
  - **`Delete` (Default):** The resource is deleted.
  - **`Retain`:** The resource is kept (orphaned) after the stack is deleted. Useful for protecting critical resources like databases.
  - **`Snapshot`:** For resources that support it (like EBS volumes and RDS databases), a final snapshot is taken before the resource is deleted.
- **`TerminationProtection`:** A stack-level setting that prevents the stack from being accidentally deleted.
- **Stack Policies:** A JSON document that protects stack resources from unintentional updates. When a stack policy is applied, all resources are protected by default, and you must explicitly `Allow` updates on specific resources.

## 8.6: Advanced CloudFormation Topics
- **Custom Resources:** Allows you to write custom provisioning logic in an AWS Lambda function for resources that are not natively supported by CloudFormation. A common use case is to create a custom resource that empties an S3 bucket before the bucket is deleted by CloudFormation.
- **Dynamic References:** A way to fetch values from SSM Parameter Store or AWS Secrets Manager at deployment time. This avoids hardcoding sensitive information or configuration values in your templates. The syntax is `{{resolve:service:reference-key}}`.
- **StackSets:** A powerful feature for deploying the same CloudFormation stack across multiple AWS accounts and regions in a single operation. This is ideal for setting up baseline resources, security roles, or configuration across an entire AWS Organization.



# Chapter 9: Lambda for SysOps

This chapter provides a SysOps-oriented overview of AWS Lambda, focusing on its core concepts, event-driven architecture, permissions, performance, and monitoring.

## 9.1: Lambda Overview
- **EC2 vs. Lambda:**
  - **EC2:** Virtual servers that you provision and manage. They run continuously, and you are responsible for scaling.
  - **Lambda:** A serverless compute service. You upload code as "functions" that run on demand. You don't manage any servers.
- **Key Lambda Characteristics:**
  - **Event-Driven:** Functions are invoked in response to events from other AWS services.
  - **Short Executions:** Functions have a timeout, with a maximum of 15 minutes.
  - **Automated Scaling:** Lambda automatically scales from a few requests to thousands per second.
  - **Pricing:** Pay-per-use, based on the number of requests and the compute duration (in GB-seconds).
- **Supported Runtimes:** Supports popular languages like Node.js, Python, Java, Go, and Ruby. Custom runtimes and container images are also supported.
- **Common Integrations (Event Sources):** API Gateway, S3, DynamoDB, SQS, SNS, CloudWatch Events (EventBridge), Kinesis, and more.

## 9.2: Lambda Event Triggers
- **CloudWatch Events / EventBridge:** A common way to trigger Lambda functions.
  - **Scheduled Events:** Trigger a function on a recurring schedule (e.g., every hour), creating a serverless CRON job.
  - **Event-Based Rules:** Trigger a function in response to events happening in your AWS account (e.g., an EC2 instance state change, a CodePipeline stage change).
- **S3 Event Notifications:** Trigger a Lambda function in response to S3 bucket events (e.g., `s3:ObjectCreated:*`). This is a classic pattern for use cases like automatically creating image thumbnails.
- **Invocation Types:**
  - **Asynchronous Invocation:** Services like S3 and EventBridge invoke Lambda asynchronously. If the invocation fails due to throttling or system errors, Lambda will automatically retry for up to six hours before sending the event to a Dead-Letter Queue (DLQ), if configured.

## 9.3: Lambda Permissions (IAM Roles & Resource Policies)
- **Execution Role:** Every Lambda function has an IAM role that grants it permissions to access other AWS services and resources. For example, to write logs, it needs permissions for CloudWatch Logs (`AWSLambdaBasicExecutionRole`). To read from an SQS queue, it needs SQS permissions.
- **Resource-Based Policy:** A policy attached directly to the Lambda function itself. It grants other AWS services or accounts permission to *invoke* the Lambda function. This is how services like S3 and EventBridge get permission to trigger your function. The console often sets this up for you automatically when you create a trigger.
- **Key Distinction:**
  - The **Execution Role** defines what the Lambda function *can do*.
  - The **Resource Policy** defines who *can invoke* the Lambda function.

## 9.4: Lambda Monitoring & Tracing
- **CloudWatch Logs:** All output from a Lambda function (e.g., `print` or `console.log` statements) and execution summaries are automatically sent to a dedicated log group in CloudWatch Logs.
- **CloudWatch Metrics:** Lambda automatically publishes several key metrics, including:
  - `Invocations`: Number of times the function was invoked.
  - `Duration`: The execution time of the function.
  - `Errors`: Number of failed invocations.
  - `Throttles`: Number of times an invocation was throttled due to concurrency limits.
  - `ConcurrentExecutions`: The number of function instances processing events simultaneously.
- **AWS X-Ray:** A service for tracing and analyzing requests in distributed applications. You can enable **Active Tracing** in the Lambda configuration to trace function invocations and downstream calls to other AWS services. This requires adding the `AWSXRayDaemonWriteAccess` policy to the function's execution role.

## 9.5: Lambda Performance
- **RAM:** You can configure a function's memory from 128 MB up to 10 GB. **Increasing the RAM also proportionally increases the CPU and network performance.** This is the primary way to improve the performance of a CPU-bound function.
- **Timeout:** The maximum amount of time (up to 15 minutes) a function can run before it is terminated.
- **Execution Context:** A temporary runtime environment that Lambda initializes for your function. This context, including database connections and SDK clients initialized *outside* the main handler function, can be reused for subsequent invocations. This is a critical performance optimization to avoid re-initializing resources on every call.
- **/tmp Space:** Lambda provides a temporary, 512 MB file system directory (`/tmp`) that can be used for scratch space. This space is preserved for the lifetime of the execution context, allowing data to be cached between invocations.

## 9.6: Lambda Concurrency
- **Concurrency:** The number of requests that your function is serving at any given time. By default, your AWS account has a total concurrency limit (often 1000) that is shared across all functions in a region.
- **Throttling:** If invocations exceed the concurrency limit, they are throttled.
  - **Synchronous Invocations (e.g., from API Gateway):** Return a `429` throttling error to the client.
  - **Asynchronous Invocations (e.g., from S3):** Are retried automatically by Lambda.
- **Reserved Concurrency:** You can set a specific concurrency limit for an individual function. This guarantees that the function can always scale up to that number, but it also caps its maximum concurrency, preventing it from consuming the entire account-level limit and throttling other functions.
- **Provisioned Concurrency:** A feature to keep a specified number of execution environments initialized and ready to respond instantly. This eliminates **cold starts** (the latency associated with initializing a new execution context) for latency-sensitive applications. This feature has an associated cost.



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



# Chapter 11: Amazon S3 Introduction

This chapter introduces Amazon S3 (Simple Storage Service), one of the foundational services of AWS. It covers the core concepts of S3, including buckets, objects, security, versioning, replication, and storage classes.

## 11.1: S3 Overview
- **S3 (Simple Storage Service):** An infinitely scalable object storage service.
- **Use Cases:** Backup and storage, disaster recovery, data archiving, hybrid cloud storage, static website hosting, software delivery, and serving as a data lake for analytics.
- **Buckets:** Objects are stored in buckets, which are like top-level folders. Bucket names must be **globally unique** across all AWS accounts and regions.
- **Objects:** The files you store in S3. An object is identified by a **key**, which is the full path to the file (e.g., `my-folder/my-file.txt`). The key is composed of a prefix (`my-folder/`) and an object name (`my-file.txt`).
- **Object Size:** Objects can be up to 5 TB. For objects larger than 5 GB, you must use the multi-part upload API.

## 11.2: S3 Security
- **Access Control:** S3 security is managed through a combination of user-based and resource-based policies.
  - **IAM Policies (User-Based):** Define what actions an IAM user or role can perform on S3 resources.
  - **S3 Bucket Policies (Resource-Based):** JSON documents attached to a bucket to grant permissions to other principals (users, accounts, services). This is the primary way to grant public access or cross-account access.
  - **Access Control Lists (ACLs):** A legacy mechanism for managing access to individual objects. It is now recommended to disable ACLs and use bucket policies for simpler access management.
- **Block Public Access:** A set of account-level and bucket-level settings that act as a centralized control to prevent accidental public exposure of data. These settings override any conflicting bucket policies or ACLs.

## 11.3: S3 Static Website Hosting
- **Functionality:** S3 can be configured to host a static website (containing only HTML, CSS, JavaScript, and other static files).
- **Configuration:** You enable static website hosting in the bucket properties and specify an index document (e.g., `index.html`).
- **Requirement:** For the website to be accessible, the objects in the bucket must be made public, which is typically done via an S3 bucket policy that allows `s3:GetObject` for all principals (`*`).

## 11.4: S3 Versioning
- **What is it?** A bucket-level feature that keeps a history of all versions of an object. When you overwrite an object, a new version is created instead of replacing the existing one.
- **Benefits:**
  - **Protection from accidental deletion:** Deleting an object creates a "delete marker" but does not remove the underlying versions, allowing for easy recovery.
  - **Easy rollback:** You can easily revert to a previous version of an object.
- **Note:** Versioning must be enabled to use S3 Replication.

## 11.5: S3 Replication
- **Functionality:** Asynchronously copies objects from a source bucket to a destination bucket.
- **Types:**
  - **Cross-Region Replication (CRR):** Replicates objects to a bucket in a *different* AWS Region. Used for compliance, lower latency access, and disaster recovery.
  - **Same-Region Replication (SRR):** Replicates objects to a bucket in the *same* AWS Region. Used for aggregating logs or creating a separate copy for testing.
- **Important Notes:**
  - By default, only **new** objects are replicated after the rule is created. To replicate existing objects, you must use **S3 Batch Replication**.
  - Replication of delete markers can be enabled or disabled.
  - There is no chaining of replication (if A replicates to B, and B replicates to C, objects from A do not end up in C).

## 11.6: S3 Storage Classes
- S3 offers a range of storage classes optimized for different access patterns and cost requirements. All classes offer the same high durability (11 nines).
- **For Frequently Accessed Data:**
  - **S3 Standard:** Default tier. Low latency and high throughput. Designed for frequently accessed data.
- **For Infrequently Accessed Data:**
  - **S3 Standard-IA (Infrequent Access):** Lower storage price but has a retrieval fee. Good for long-lived, less frequently accessed data like backups and disaster recovery files.
  - **S3 One Zone-IA:** Stored in a single AZ. Cheaper than Standard-IA, but data is lost if the AZ fails. Good for storing secondary backup copies or data that can be recreated.
- **For Archival Data:**
  - **S3 Glacier Instant Retrieval:** For long-term archival with millisecond retrieval. Cheaper than Standard-IA.
  - **S3 Glacier Flexible Retrieval:** Configurable retrieval times from minutes to hours. A balance between cost and access time.
  - **S3 Glacier Deep Archive:** The lowest-cost storage class. Retrieval takes 12-48 hours. For data that is rarely, if ever, accessed.
- **For Unknown/Changing Access Patterns:**
  - **S3 Intelligent-Tiering:** Automatically moves objects between access tiers (Frequent, Infrequent, and optional Archive tiers) based on usage patterns to optimize costs. Incurs a small monthly monitoring and automation fee.



# Chapter 12: Advanced Amazon S3 & Athena

This chapter explores advanced features of Amazon S3 for data management and performance optimization, and introduces Amazon Athena for serverless data analysis.

## 12.1: S3 Lifecycle Rules
- **Purpose:** To automate the transition of objects between different storage classes to optimize costs.
- **Actions:**
  - **Transition Actions:** Move objects to a different storage class after a certain number of days (e.g., move to `S3 Standard-IA` after 30 days, then to `S3 Glacier Flexible Retrieval` after 180 days).
  - **Expiration Actions:** Configure objects to expire (be deleted) after a specified period. This can be applied to current versions, previous versions, or incomplete multipart uploads.
- **Scope:** Rules can be applied to an entire bucket, or scoped to a specific prefix (folder) or object tags.
- **S3 Analytics:** A feature that can be enabled to analyze access patterns on your objects and recommend optimal lifecycle rules for transitioning data between `S3 Standard` and `S3 Standard-IA`.

## 12.2: S3 Event Notifications
- **Functionality:** S3 can trigger events when certain actions occur in your bucket (e.g., an object is created, deleted, or restored).
- **Destinations:** These event notifications can be sent to:
  - **SNS (Simple Notification Service):** To fan out notifications to multiple subscribers.
  - **SQS (Simple Queue Service):** To queue events for reliable, asynchronous processing.
  - **Lambda Function:** To directly invoke a function to process the event.
- **EventBridge Integration:** S3 events are also automatically sent to Amazon EventBridge, which allows for more advanced filtering and routing to over 18 different AWS services.
- **Use Case:** A classic example is triggering a Lambda function to automatically create a thumbnail whenever a new image is uploaded to an S3 bucket.

## 12.3: S3 Performance Optimization
- **Baseline Performance:** S3 automatically scales to provide very high request rates. By default, it supports **3,500 PUT/POST/DELETE** and **5,500 GET/HEAD** requests per second, *per prefix*.
- **Prefixes:** A prefix is the "folder path" part of an object's key (e.g., in `my-folder/my-file.txt`, the prefix is `my-folder/`). Spreading your objects across multiple prefixes allows you to scale your request rate horizontally.
- **Multi-Part Upload:** For large objects (>100 MB is recommended, >5 GB is required), this feature splits the object into smaller parts that can be uploaded in parallel. This improves upload speed, increases resilience to network errors (only failed parts need to be retried), and allows you to pause and resume uploads.
- **S3 Transfer Acceleration:** Speeds up long-distance uploads and downloads by routing traffic through a nearby AWS Edge Location over the public internet, and then over the optimized AWS global network to the S3 bucket.
- **S3 Byte-Range Fetches:** Allows you to retrieve only a specific portion of an object, rather than the whole file. This can be used to parallelize downloads or to retrieve just the header of a file for inspection.

## 12.4: S3 Batch Operations
- **Functionality:** A managed service to perform bulk operations on a large number of S3 objects with a single request.
- **Operations:**
  - Copy objects between buckets.
  - Modify object metadata, tags, or ACLs.
  - Encrypt unencrypted objects.
  - Restore objects from S3 Glacier.
  - Invoke a Lambda function for custom actions on each object.
- **Process:** You provide a manifest (a list of objects, often generated by S3 Inventory) and specify the operation. S3 Batch Operations then manages the job, including retries, tracking, and completion reports.

## 12.5: S3 Inventory
- **Functionality:** Provides a flat file output (CSV, ORC, or Parquet) listing your objects and their corresponding metadata on a daily or weekly basis.
- **Use Case:** A scalable alternative to the `ListObjects` API call for auditing and reporting on the replication and encryption status of your objects. The output can be analyzed with tools like Amazon Athena.

## 12.6: S3 Glacier
- **S3 Glacier:** A set of low-cost, long-term archival storage classes. Data is stored in **Vaults**, and each item is an **Archive**.
- **Retrieval Tiers:**
  - **Expedited:** 1-5 minutes (most expensive).
  - **Standard:** 3-5 hours.
  - **Bulk:** 5-12 hours (cheapest).
- **Glacier Vault Lock:** A compliance feature that allows you to enforce policies on a vault, such as a Write-Once-Read-Many (WORM) policy. Once a Vault Lock policy is locked, it is **immutable** and cannot be changed or deleted.

## 12.7: Amazon Athena
- **What is it?** A serverless, interactive query service that makes it easy to analyze data directly in Amazon S3 using standard SQL.
- **How it Works:** You define a table schema over your data in S3, and then you can run SQL queries against it. Athena is serverless, so there is no infrastructure to manage.
- **Pricing:** You pay per query, based on the amount of data scanned ($5 per TB).
- **Performance Optimization:**
  - Use **columnar data formats** like Apache Parquet or ORC. This allows Athena to only read the columns needed for a query, drastically reducing the amount of data scanned.
  - **Compress** your data (e.g., with GZIP or Snappy).
  - **Partition** your data in S3 based on common query filters (e.g., `/year=2023/month=09/day=25/`). This allows Athena to skip reading data in irrelevant partitions.
- **Federated Query:** Athena can use Data Source Connectors (which are Lambda functions) to run queries on data stored outside of S3, including relational databases (RDS), NoSQL databases (DynamoDB), and even on-premises data sources.


# Chapter 13: Amazon S3 Security

## S3 Encryption
S3 offers four methods for object encryption:
-   **Server-Side Encryption with S3-Managed Keys (SSE-S3):** Uses AES-256 encryption with keys managed and owned by AWS. Enabled by default for new buckets and objects. Requires `x-amz-server-side-encryption: AES256` header.
-   **Server-Side Encryption with KMS-Managed Keys (SSE-KMS):** Uses AWS Key Management Service (KMS) for key management, providing user control and CloudTrail logging of key usage. Requires `x-amz-server-side-encryption: aws:kms` header. Subject to KMS API call quotas.
-   **Server-Side Encryption with Customer-Provided Keys (SSE-C):** Encryption keys are managed outside AWS and provided by the customer with each request. AWS does not store the key. Requires HTTPS and passing the key in HTTP headers.
-   **Client-Side Encryption:** Data is encrypted by the client before sending to S3 and decrypted by the client after retrieval. The client fully manages the keys and encryption cycle.

## Encryption in Transit (SSL/TLS)
S3 buckets have HTTP (unencrypted) and HTTPS (encrypted) endpoints. It is recommended to use HTTPS for secure data transmission. Bucket policies can enforce HTTPS by denying requests where `aws:SecureTransport` is false.

## Default Encryption vs. Bucket Policies
By default, new buckets have SSE-S3 encryption. This can be changed to SSE-KMS. Bucket policies can force specific encryption types (e.g., SSE-KMS or SSE-C) by denying PUT operations without the correct encryption headers. Bucket policies are evaluated before default encryption settings.

## Cross-Origin Resource Sharing (CORS)
CORS is a web browser security mechanism that allows or denies requests to different origins (scheme, host, port) while visiting a main origin. For S3, if a client makes a cross-origin request (e.g., a website hosted on S3 trying to load assets from another S3 bucket or domain), the S3 bucket serving the assets must have a CORS configuration that allows the origin of the requesting website. This is configured in a JSON policy on the S3 bucket.

## MFA Delete
Multi-Factor Authentication (MFA) Delete is a security feature that requires an MFA code for critical S3 operations:
-   Permanently deleting an object version.
-   Suspending Versioning on a bucket.
MFA Delete must be enabled by the bucket owner (root account) using the AWS CLI (not directly from the console UI). It requires Versioning to be enabled on the bucket.

## S3 Access Logs
For auditing purposes, all access requests (authorized or denied) to an S3 bucket can be logged as files into another S3 bucket. These logs can then be analyzed using tools like Amazon Athena. The logging bucket must be in the same AWS region as the source bucket. **Important:** The logging bucket should never be the same as the source bucket to avoid an infinite logging loop.

## S3 Pre-signed URLs
Pre-signed URLs allow temporary access to private S3 objects for a limited time (up to 12 hours via console, 168 hours via CLI/SDK). The user accessing the URL inherits the permissions of the user who generated it. This is useful for sharing private files with external users without making the bucket or object public, or for allowing temporary uploads to specific locations.

## S3 Object Lock
S3 Object Lock enables a Write Once Read Many (WORM) model for objects in S3 buckets (requires Versioning enabled). It prevents object versions from being overwritten or deleted for a specified period.
-   **Retention Modes:**
    -   **Compliance Mode:** Object versions cannot be overwritten or deleted by any user, including the root user. Retention settings cannot be changed or shortened.
    -   **Governance Mode:** Most users cannot override or delete object versions or alter lock settings, but privileged users (with specific IAM permissions) can.
-   **Retention Period:** Objects are protected for a fixed period, which can be extended.
-   **Legal Hold:** Protects an object indefinitely, independent of any retention period. Requires `s3:PutObjectLegalHold` IAM permission to apply or remove.

## S3 Access Points
S3 Access Points simplify security management for S3 buckets, especially for large datasets with diverse access patterns. Each access point has its own DNS name and can have a specific access point policy (similar to a bucket policy) that grants granular permissions (e.g., read/write access to a specific prefix). This allows for managing security at scale by delegating access control from a complex bucket policy to multiple simpler access point policies.
-   **Network Origin:** Can be configured for internet access or VPC-only access (for private traffic within a VPC).
-   **VPC Endpoints:** For private access, a VPC Endpoint Gateway is used to connect to S3 access points without traversing the public internet. VPC endpoint policies can further restrict access.

## S3 Multi-Region Access Points
Provide a global endpoint that spans multiple S3 buckets across different regions. They dynamically route requests to the nearest S3 bucket for lowest latency. Requires bidirectional replication between the associated S3 buckets to ensure data consistency. Offers failover controls (active/passive or active/active setups) for high availability and disaster recovery.


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


# Chapter 15: CloudFront

## CloudFront Overview
CloudFront is a Content Delivery Network (CDN) that improves read performance by caching website content at globally distributed edge locations. This reduces latency for users worldwide and provides DDoS protection.
-   **Edge Locations:** Hundreds of points of presence globally.
-   **How it works:** Users request content from the nearest edge location. If not cached, CloudFront fetches it from the origin (e.g., S3 bucket, EC2 instance) and caches it for future requests.
-   **Origins:**
    -   **Amazon S3 buckets:** For distributing and caching files. Secured using Origin Access Control (OAC) to allow private access.
    -   **VPC origin:** For applications hosted in private subnets (e.g., ALB, NLB, EC2 instances). Provides a secure, private connection.
    -   **Custom origin:** Any public HTTP backend, inside or outside AWS.
-   **CloudFront vs. S3 Cross-Region Replication:**
    -   **CloudFront:** Global edge network (216+ PoPs), caches static content for a period (e.g., a day), ideal for static content distributed worldwide.
    -   **S3 Cross-Region Replication:** Replicates entire buckets to specific regions, near real-time updates, no caching, ideal for dynamic content needing low latency in a few regions.

## CloudFront with S3 Hands-On
Demonstrates setting up a CloudFront distribution for an S3 bucket.
-   Upload files (e.g., `beach.jpg`, `coffee.jpg`, `index.html`) to a private S3 bucket.
-   Create a CloudFront distribution, selecting the S3 bucket as the origin.
-   Enable "private S3 bucket access" (using OAC) to allow CloudFront to access the bucket without making objects public. CloudFront automatically updates the S3 bucket policy.
-   Once deployed, content is accessible via the CloudFront domain name, with caching at edge locations.

## CloudFront ALB/EC2 as an Origin
-   **VPC Origins (Recommended):** Connects CloudFront directly to applications in private VPC subnets (ALB, NLB, EC2 instances) without exposing them to the internet. This is the most secure method.
-   **Public Network (Older Method):** Required exposing the ALB/EC2 instance publicly and configuring security groups to allow traffic only from CloudFront's public IP ranges (which are dynamic and require frequent updates). Less secure and more tedious.

## CloudFront Geo Restriction
Allows restricting access to content based on the user's country.
-   **Allowlist:** Defines a list of approved countries.
-   **Blocklist:** Defines a list of banned countries.
-   Country is determined using a third-party Geo-IP database.
-   **Use Case:** Copyright laws, content licensing. Configured under distribution settings.

## CloudFront Reports, Logs, and Troubleshooting
-   **Access Logs:** CloudFront logs every request made to your distribution into a specified S3 logging bucket. Each edge location sends its log data. The logging bucket should be different from the origin bucket.
-   **Reports:** CloudFront automatically generates various reports (Cache Statistics, Popular Objects, Top Referrers, Usage, Viewers) using access log data, even if explicit access logging to S3 is not enabled.
-   **Troubleshooting:** CloudFront caches HTTP 4xx (client errors like 403 Forbidden, 404 Not Found) and 5xx (server errors) status codes from the origin. This can lead to caching of error responses.

## CloudFront Caching Deep Dive
CloudFront caching behavior can be configured based on headers, cookies, and query string parameters. The goal is to maximize cache hits and minimize requests to the origin.
-   **Headers:**
    -   **Forward All:** No caching, every request goes to origin (TTL=0).
    -   **Whitelist:** Caching based on values in specified headers.
    -   **None (Default):** Caching not based on request headers, best caching performance.
    -   **Origin Custom Headers:** Constant headers added by CloudFront to requests sent to the origin (not for caching).
    -   **Cache Behavior Headers:** Headers used for caching logic.
-   **Caching TTL (Time To Live):**
    -   Controlled by `Cache-Control: max-age` or `Expires` headers from the origin.
    -   Can be customized with minimum, maximum, and default TTL values in CloudFront settings.
-   **Cookies:**
    -   **Do Not Process (Default):** No caching based on cookies, cookies not forwarded to origin.
    -   **Whitelist:** Caching based on values in specified cookies.
    -   **Forward All:** Worst caching performance.
-   **Query String Parameters:**
    -   **Do Not Process (Default):** Not passed to origin, no caching based on query strings.
    -   **Whitelist:** Caching based on values in specified query strings.
    -   **Forward All:** Worst caching performance.
-   **Maximizing Cache Hits:**
    -   Minimize forwarded headers, cookies, and query string parameters.
    -   Use `Cache-Control: max-age` header from origin.
    -   Separate static and dynamic content into different distributions or behaviors.

## CloudFront with ALB Sticky Sessions
When using CloudFront with an ALB that has sticky sessions enabled, it's crucial to configure CloudFront to forward the session cookie (e.g., `AWSALB`) to the origin. This ensures that requests from the same user are consistently routed to the same backend EC2 instance, maintaining session affinity.


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


# Chapter 21: Identity

## IAM Security Tools
-   **IAM Credentials Report:** Account-level report listing all users and the status of their various credentials (password enabled, MFA active, access keys generated/last used/rotated). Useful for auditing and identifying security risks.
-   **IAM Access Advisor:** User-level tool showing service permissions granted to a user and when those services were last accessed. Helps implement the principle of least privilege by identifying unused permissions.

## IAM Access Analyzer
Identifies resources shared externally (outside your defined zone of trust, e.g., your AWS account or organization).
-   **Monitored Resources:** S3 buckets, IAM Roles, KMS Keys, Lambda Functions/Layers, SQS Queues, Secrets Manager Secrets.
-   **Zone of Trust:** Your AWS account(s) or organization.
-   **Findings:** Resources shared with entities outside the zone of trust are flagged as findings.
-   **Remediation:** Provides links to resources to fix unintended access. Can archive findings that are intentional.

## Identity Federation
Allows users outside AWS to assume temporary roles to access AWS resources without creating individual IAM users. User management is done outside AWS.
-   **SAML Federation:** For enterprises with SAML 2.0 compliant identity providers (e.g., Microsoft Active Directory). Users authenticate with IDP, get a SAML assertion, exchange it with AWS STS for temporary credentials, then access AWS Console/CLI.
-   **Custom Identity Broker:** For non-SAML 2.0 identity providers. Requires custom application development to programmatically exchange identity with AWS STS for temporary credentials. More complex than SAML.
-   **Cognito Identity Pools (Federated Identities):** For web and mobile application users to access AWS resources.
    -   Users log in via public providers (Google, Facebook, Amazon, Apple), Cognito User Pools, OpenID Connect, SAML, or custom developer-authenticated identities.
    -   Exchanges identity tokens for temporary AWS credentials via STS.
    -   Can allow unauthenticated (guest) users.
    -   IAM policies attached to roles define permissions. Can use policy variables for fine-grained control (e.g., user-specific S3 prefixes, DynamoDB rows).

## AWS Security Token Service (STS)
Grants limited and temporary access to AWS resources. Tokens are valid for a limited time (e.g., up to 1 hour) and must be refreshed.
-   **API Calls:**
    -   `AssumeRole`: Assume an IAM role within the same account or across accounts.
    -   `AssumeRoleWithSAML`: Exchange SAML assertion for temporary credentials.
    -   `AssumeRoleWithWebIdentity`: Exchange identity from web identity providers (e.g., Facebook, Google) for temporary credentials (now recommended to use Cognito).
    -   `GetSessionToken`: Obtain temporary credentials for MFA-authenticated users.
-   **Cross-Account Access:** A common use case for `AssumeRole` where users in one account assume a role in another account to perform actions.

## Cognito User Pools
A serverless user directory for web and mobile applications. Used for authentication (identity verification).
-   **Features:** User registration, login (username/email & password), password reset, email/phone verification, MFA, social logins (Google, Facebook, Amazon), corporate logins (SAML, OpenID Connect).
-   **Output:** Returns a JSON Web Token (JWT) upon successful authentication.
-   **Integration:** Natively integrates with API Gateway and Application Load Balancer for authentication.

## Cognito User Pools vs. Identity Pools
-   **User Pools (Authentication):** Manages user identities, provides login/registration, issues JWTs. Focus is on *who* the user is.
-   **Identity Pools (Authorization):** Exchanges identity tokens (from User Pools or other identity providers) for temporary AWS credentials. Focus is on *what* the user can do in AWS.
-   They can be used together: User Pool authenticates the user, then Identity Pool provides AWS access based on that authentication.


# Chapter 22: Networking Route 53

## What is a DNS?
DNS (Domain Name System) translates human-friendly hostnames (e.g., `www.google.com`) into IP addresses. It's the backbone of the internet, using a hierarchical naming structure.
-   **Terminology:**
    -   **Domain Registrar:** Where you register domain names (e.g., Route 53, GoDaddy).
    -   **DNS Records:** Define how to route traffic (e.g., A, AAAA, CNAME, NS).
    -   **Zone File:** Contains all DNS records for a domain.
    -   **Name Servers:** Resolve DNS queries.
    -   **TLD (Top-Level Domain):** `.com`, `.org`, etc.
    -   **Second-Level Domain:** `example.com`.
    -   **Subdomain:** `www.example.com`, `api.example.com`.
    -   **FQDN (Fully Qualified Domain Name):** Complete domain name (e.g., `api.www.example.com`).
-   **How DNS Works:** A client queries its local DNS server, which recursively queries root, TLD, and second-level domain name servers until it finds the authoritative name server (e.g., Route 53) that holds the record. The answer is cached and returned to the client.

## Route 53 Overview
Route 53 is a highly available, scalable, fully managed, and authoritative DNS service. It's also a domain registrar.
-   **Hosted Zones:** Containers for DNS records.
    -   **Public Hosted Zone:** For public domain names, answers queries from the internet.
    -   **Private Hosted Zone:** For private domain names, answers queries only from within specified VPCs.
-   **Cost:** $0.50/month per hosted zone. Domain registration costs vary (e.g., $12/year).
-   **DNS Record Types:**
    -   **A Record:** Maps hostname to IPv4 address.
    -   **AAAA Record:** Maps hostname to IPv6 address.
    -   **CNAME Record:** Maps hostname to another hostname. Cannot be used for the zone apex (root domain, e.g., `example.com`).
    -   **NS Record:** Specifies name servers for a hosted zone.
-   **TTL (Time To Live):** Amount of time a record is cached by DNS resolvers. High TTL = less Route 53 traffic, but slower record changes. Low TTL = more Route 53 traffic, faster record changes. Mandatory for most records, not for Alias records.

## CNAME vs. Alias
-   **CNAME Record:** Points a hostname to *any* other hostname. Works only for non-root domains (e.g., `app.mydomain.com`).
-   **Alias Record:** Route 53 specific. Points a hostname to a *specific AWS resource* (ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 Websites, VPC Interface Endpoints, Global Accelerator, other Route 53 records). Works for both root and non-root domains. Free of charge, automatically inherits health checks from the target resource. Cannot set TTL.

## Routing Policies
Define how Route 53 responds to DNS queries.
-   **Simple Routing:** Routes traffic to a single resource. Can return multiple values (IPs) for a single record, with clients choosing one randomly. Cannot be associated with health checks.
-   **Weighted Routing:** Routes traffic to multiple resources based on assigned weights (percentages). Useful for load balancing or A/B testing. Can be associated with health checks.
-   **Failover Routing:** Routes traffic to a primary resource if healthy, otherwise to a secondary resource. Requires health checks.
-   **Latency-Based Routing:** Routes traffic to the resource with the lowest latency for the user.
-   **Geolocation Routing:** Routes traffic based on the user's geographic location (continent, country, US state). Useful for content localization, restricting distribution. Requires a default record.
-   **Geoproximity Routing:** Routes traffic based on the geographic location of users and resources. Uses a "bias" value to shift more or less traffic to a resource. Requires Route 53 Traffic Flow.
-   **Multi-Value Answer Routing:** Returns up to 8 healthy records for a single DNS query. Clients choose one randomly. Can be associated with health checks. Not a substitute for an ELB.

## Health Checks
Monitor the health of resources.
-   **Endpoint Health Check:** Monitors public endpoints (application, server, AWS resource) via HTTP, HTTPS, or TCP. Health checkers from multiple global locations send requests.
-   **Calculated Health Check:** Combines results of multiple child health checks (up to 256) using AND/OR/NOT logic.
-   **CloudWatch Alarm Health Check:** Monitors a CloudWatch Alarm. Useful for checking private resources (e.g., EC2 instance in private subnet) by linking the alarm state to the health check.
-   **Network Configuration:** Health checkers' IP ranges must be allowed in security groups/firewalls.

## S3 Website with Route 53
To host a static website on S3 with a custom domain:
1.  Create an S3 bucket with the exact same name as the domain (e.g., `blog.example.com`).
2.  Enable static website hosting on the S3 bucket and make objects public.
3.  Create an Alias record in Route 53 pointing to the S3 website endpoint.
    -   Only works for HTTP, not HTTPS (requires CloudFront for HTTPS).

## Route 53 Resolvers (Hybrid DNS)
Connects AWS DNS with on-premises DNS.
-   **Inbound Endpoint:** Allows on-premises DNS resolvers to resolve domain names of AWS resources (e.g., private hosted zones).
-   **Outbound Endpoint:** Allows Route 53 Resolver to forward DNS queries to on-premises DNS resolvers.

## 3rd Party Domains with Route 53
You can use Route 53 as the DNS service for a domain registered with a third-party registrar (e.g., GoDaddy).
1.  Create a public hosted zone in Route 53 for your domain.
2.  Update the NS (Name Server) records on the third-party registrar's website to point to the Route 53 name servers.


# Chapter 23: Networking VPC

## What is a VPC?
A VPC (Virtual Private Cloud) is a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. It's like having your own data center in the cloud.
-   **Key Components:**
    -   **Subnets:** Subdivisions of a VPC, can be public (with Internet Gateway) or private (without Internet Gateway).
    -   **Route Tables:** Control routing for subnets.
    -   **Internet Gateway (IGW):** Allows public subnets to connect to the internet.
    -   **NAT Gateway/Instance:** Allows private subnets to connect to the internet (e.g., for updates) without being publicly accessible.
    -   **Security Groups:** Act as virtual firewalls for instances.
    -   **Network ACLs (NACLs):** Act as stateless packet filters for subnets.

## VPC Flow Logs
Capture information about the IP traffic going to and from network interfaces in your VPC.
-   **Data Captured:** Source/destination IP, port, protocol, action (ACCEPT/REJECT), bytes, packets.
-   **Destinations:** CloudWatch Logs, S3, Kinesis Data Firehose.
-   **Use Cases:** Troubleshooting network connectivity, monitoring network traffic, identifying security threats.

## VPC Peering
Connects two VPCs privately, allowing resources in one VPC to communicate with resources in another VPC as if they were in the same network.
-   **Limitations:** No transitive peering (VPC A peered with B, B with C, A cannot talk to C directly). No overlapping CIDR blocks.
-   **Use Cases:** Sharing resources between VPCs, connecting applications in different VPCs.

## VPC Endpoints
Allows private access to AWS services from within your VPC without traversing the public internet.
-   **Interface Endpoints (Powered by AWS PrivateLink):** Provides a private IP address for an AWS service (e.g., S3, DynamoDB, EC2, Lambda) within your VPC. Traffic flows through AWS's private network.
-   **Gateway Endpoints:** Specifically for S3 and DynamoDB. Creates a route table entry to direct traffic to the service. No private IP address.

## AWS Direct Connect
Establishes a dedicated private network connection from your on-premises data center to AWS.
-   **Benefits:** Reduced network costs, increased bandwidth throughput, more consistent network experience than internet-based connections.
-   **Use Cases:** Hybrid cloud environments, large data transfers, real-time applications.

## AWS Site-to-Site VPN
Establishes an encrypted connection between your on-premises network and your VPC over the public internet.
-   **Benefits:** Secure communication, cost-effective for smaller workloads or temporary connections.
-   **Components:** Customer Gateway (on-premises), Virtual Private Gateway (VPC).

## AWS Client VPN
Allows clients (e.g., remote employees) to securely access AWS resources and on-premises networks from any location using a VPN client.
-   **Benefits:** Secure remote access, integrates with Active Directory or Certificate-based authentication.

## AWS Transit Gateway
Connects thousands of VPCs and on-premises networks through a central hub.
-   **Benefits:** Simplifies network topology, reduces operational overhead, enables transitive peering.
-   **Attachments:** VPCs, VPNs, Direct Connect Gateways.
-   **Routing:** Centralized route table for all connected networks.

## AWS Global Accelerator
Improves the availability and performance of your applications for global users by directing traffic to optimal endpoints over the AWS global network.
-   **How it works:** Provides two static anycast IP addresses that act as a fixed entry point to your application. Traffic is routed to the closest AWS edge location, then over the AWS global network to the optimal endpoint.
-   **Benefits:** Improved performance (up to 60% faster), increased availability (automatic failover), simplified global traffic management.
-   **Endpoints:** ELBs, EC2 instances, Elastic IPs.

## AWS PrivateLink
Enables private connectivity between VPCs and AWS services, or between VPCs and services hosted by other AWS accounts (Service Providers).
-   **Benefits:** Eliminates exposure to the public internet, simplifies network architecture, enhances security.
-   **Components:** VPC Endpoint (Consumer), Endpoint Service (Service Provider).

## Network ACLs (NACLs) vs. Security Groups
-   **NACLs:**
    -   Operate at the subnet level.
    -   Stateless (rules apply to inbound and outbound traffic independently).
    -   Process rules in order (lowest number first).
    -   Default deny all inbound/outbound traffic (unless explicitly allowed).
-   **Security Groups:**
    -   Operate at the instance level.
    -   Stateful (rules automatically apply to return traffic).
    -   Evaluate all rules before deciding.
    -   Default deny all inbound traffic, allow all outbound traffic.

## VPC Flow Logs Hands-On
Demonstrates enabling VPC Flow Logs to capture network traffic information.
-   **Configuration:** Select VPC, subnet, or network interface. Choose destination (S3 or CloudWatch Logs). Define log format.
-   **Analysis:** Flow logs can be analyzed in CloudWatch Logs Insights or queried in S3 using Athena.

## VPC Peering Hands-On
Demonstrates connecting two VPCs using VPC Peering.
-   **Process:** Request peering connection from one VPC to another. Accept the request. Update route tables in both VPCs to allow traffic to the peered VPC's CIDR block.
-   **Verification:** Test connectivity between instances in peered VPCs.

## VPC Endpoints Hands-On
Demonstrates creating a VPC Endpoint for S3.
-   **Gateway Endpoint:** Create a Gateway Endpoint for S3. Update route tables to direct S3 traffic through the endpoint.
-   **Interface Endpoint:** Create an Interface Endpoint for S3. This provides a private IP address for S3 within your VPC.

## AWS Direct Connect Hands-On
Demonstrates the process of setting up a Direct Connect connection.
-   **Steps:** Request a Direct Connect connection. Choose location, port speed. AWS provisions the connection. You connect your router to the Direct Connect location. Configure Virtual Interfaces (VIFs) to connect to your VPC.

## AWS Site-to-Site VPN Hands-On
Demonstrates setting up a Site-to-Site VPN connection.
-   **Components:** Create a Customer Gateway (on-premises router details). Create a Virtual Private Gateway (attached to your VPC). Create a Site-to-Site VPN connection between the Customer Gateway and Virtual Private Gateway.
-   **Configuration:** Download configuration file for your on-premises router.

## AWS Client VPN Hands-On
Demonstrates setting up a Client VPN endpoint.
-   **Components:** Create a Client VPN endpoint. Configure client authentication (e.g., certificate-based). Associate target networks (subnets). Configure authorization rules.
-   **Client:** Download VPN client configuration file.

## AWS Transit Gateway Hands-On
Demonstrates setting up a Transit Gateway.
-   **Components:** Create a Transit Gateway. Create Transit Gateway attachments (VPC, VPN, Direct Connect). Configure Transit Gateway route tables.
-   **Benefits:** Centralized routing, simplified connectivity for complex networks.

## AWS Global Accelerator Hands-On
Demonstrates setting up a Global Accelerator.
-   **Components:** Create a Global Accelerator. Configure listeners (ports, protocols). Configure endpoint groups (regions, endpoints like ELBs, EC2).
-   **Benefits:** Static IP addresses, traffic routed over AWS global network for performance and availability.

## AWS PrivateLink Hands-On
Demonstrates setting up PrivateLink for a service.
-   **Service Provider:** Create an Endpoint Service (e.g., for an NLB).
-   **Consumer:** Create a VPC Endpoint (Interface Endpoint) to connect to the Endpoint Service.


# Chapter 24: Other Services

## AWS X-Ray
AWS X-Ray is a service for distributed tracing and visual analysis of applications. It helps debug performance issues, understand microservice dependencies, pinpoint service issues, review request behavior, and identify errors/exceptions.
-   **Use Cases:** Distributed tracing, troubleshooting, service graph visualization.

## AWS Amplify
A web and mobile application development tool that integrates various AWS services into a single platform.
-   **Amplify CLI:** Used to create Amplify backends (leveraging S3, Cognito, AppSync, API Gateway, SageMaker, Lex, Lambda, DynamoDB, etc.).
-   **Frontend Libraries:** Connect web and mobile applications to the Amplify backend.
-   **Amplify Console:** Used for deployment to Amplify itself and CloudFront.
-   **Analogy:** Think of it as Elastic Beanstalk for web and mobile applications.

## AWS AppSync
A fully managed service for developing GraphQL APIs.
-   **GraphQL:** A query language for APIs and a runtime for fulfilling those queries with existing data. An alternative to REST APIs.
-   **Benefits:** Flexible (single endpoint for multiple data sources), efficient (fetches only needed data), scalable, secure (multiple authentication modes).
-   **Components:**
    -   **Schema:** Defines data types and operations.
    -   **Resolvers:** Connect schema to data sources (DynamoDB, Lambda, RDS, Elasticsearch, HTTP endpoints).

## AWS IoT Core
A fully managed service for connecting billions of IoT devices to the AWS Cloud.
-   **Features:** Scalable, secure (mutual authentication, encryption), flexible (multiple protocols).
-   **Functionality:** Connects devices, routes data to other AWS services (Lambda, Kinesis, S3, DynamoDB, CloudWatch), manages devices (register, monitor, update).

## AWS Device Farm
A fully managed service for testing mobile applications on real devices in the AWS Cloud.
-   **Features:** Scalable, flexible (multiple frameworks), secure (isolated test environments).
-   **Functionality:** Upload application and tests, choose real or virtual devices, run tests, view results (screenshots, logs, performance data).

## AWS WorkSpaces
A fully managed service for provisioning cloud-based virtual desktops for users.
-   **Features:** Scalable, flexible (multiple OS), secure (isolated virtual desktops).
-   **Functionality:** Provision WorkSpaces, users connect from any device, use as regular desktops.

## AWS AppStream 2.0
A fully managed service for streaming desktop applications to any device.
-   **Features:** Scalable, flexible (multiple OS), secure (isolated application streams).
-   **Functionality:** Upload application, stream to users, users access from any device, use as regular desktop applications.

## AWS Elastic Transcoder
A fully managed service for converting media files from one format to another.
-   **Features:** Scalable, flexible (multiple formats), cost-effective (pay-as-you-go).
-   **Functionality:** Upload media to S3, create Transcoder job, convert files, deliver to users.

## AWS WorkMail
A fully managed service for hosting email and calendaring for users.
-   **Features:** Scalable, flexible (multiple clients), secure (encryption at rest and in transit).
-   **Functionality:** Create WorkMail organization, create users, users access email/calendaring from any client.

## AWS WorkDocs
A fully managed service for storing, sharing, and collaborating on documents and files.
-   **Features:** Scalable, flexible (multiple file types), secure (encryption at rest and in transit).
-   **Functionality:** Create WorkDocs site, create users, users access documents/files from any device.

## AWS WorkLink
A fully managed service for providing secure access to internal websites and web applications.
-   **Features:** Scalable, flexible (multiple browsers), secure (isolated web sessions).
-   **Functionality:** Configure WorkLink for internal websites, users access from any device, use as regular web browsers.

## AWS Cost Explorer
Used to visualize, understand, and manage AWS costs and usage over time.
-   **Reports:** Custom reports for cost and usage data (monthly, hourly, resource level).
-   **Forecasting:** Forecasts usage up to 12 months.

## AWS Budgets
Creates cost and usage alarms.
-   **Budget Types:** Cost, usage, reservation, savings plan.
-   **Alerts:** Send notifications when actual or forecasted costs/usage exceed defined thresholds.
-   **Actions:** Can trigger automated actions (e.g., attach IAM policy, stop EC2/RDS instances) when budget thresholds are breached.

## AWS Cost Allocation Tags
User-defined tags (e.g., `Environment:Production`) and AWS-generated tags (e.g., `aws:createdBy`) used to track AWS costs at a detailed level in cost and usage reports.

## AWS Cost and Usage Reports
Comprehensive reports of AWS costs and usage, including detailed metadata and cost allocation tags. Delivered to S3.


# Chapter 25: Preparing for the Exam & Practice Exam

## Exam Preparation Section Introduction
This section focuses on preparing for the AWS Certified SysOps Administrator Associate exam, covering practical tips, exam structure, and a practice exam.

## State of Learning Checkpoint: AWS Certified SysOps Administrator Associate
-   **Exam Labs:** As of March 28th, 2023, there are no exam labs. This information will be updated if they return.
-   **Exam Guide (SOA-C02):** Validates ability to manage workloads using the Well-Architected Framework, perform tasks via console/CLI, analyze logs, implement security controls, apply networking concepts, and optimize cost/performance.
-   **Recommended Experience:** At least one year of experience with deployment, management, networking, and security on AWS.
-   **Exam Content:** Multiple choice and multiple response questions. No penalty for incorrect answers.
-   **Scoring:** 50 scored questions, 15 unscored questions (unknown which are which). 130 minutes exam time. Passing score is 720 (out of 1000).
-   **Content Outline:** Covers domains like monitoring, reliability, deployments, security, networking, and cost/performance optimization.
-   **In-Scope Services:** A long list of AWS services and features are in scope. If a service is not specifically covered in the course but appears on the exam, students are encouraged to provide feedback.

## Exam Walkthrough and Signup
-   **Certification Portal:** Access via `aws.training/certification`. Requires an AWS Builder ID.
-   **Scheduling an Exam:**
    -   Select "Exam Registration" and "Schedule an exam."
    -   Choose between "in person at a test center" or "online with OnVUE."
    -   **In-person:** Requires photo ID, understanding test center rules.
    -   **Online (OnVUE):** Requires computer with webcam, good internet. Strongly recommended to run a system test beforehand. Testing space must be clear. Check-in 30 minutes before appointment.
    -   Select exam language and time zone.
    -   Complete payment (can apply vouchers).
-   **Launching Exam:** On exam day, access the Pearson VUE portal and click "Launch Exam."

## Save 50% on your AWS Exam Cost
-   **Exam Vouchers:** Successfully passing an AWS exam grants a 50% discount voucher for the next exam.
-   **Claiming Benefits:** Vouchers appear as "unclaimed" in the certification portal. Click "Claim Benefits" to get a voucher code.
-   **Applying Voucher:** Enter the voucher code during the payment step of exam registration.

## Get an Extra 30 Minutes on your AWS Exam (Non-Native English Speakers only)
-   **ESL Accommodation:** Non-native English speakers can request an extra 30 minutes for the exam.
-   **Request Process:** Go to "Exam Accommodation" in the certification portal, select "ESL +30 minutes," and request accommodation. This is a one-time approval and does not expire.
-   **Scheduling:** If an exam is already scheduled, it must be canceled and re-scheduled after the accommodation is approved to apply the extra time.

## Exam Tips and Tricks
-   **Read Carefully:** Pay close attention to question details, filtering out irrelevant information and focusing on what is being asked.
-   **Eliminate Wrong Answers:** Use process of elimination to narrow down choices.
-   **Identify Keywords:** Look for keywords that point to specific AWS services or concepts.
-   **Time Management:** Keep track of time, don't spend too long on a single question.
-   **No Penalty for Guessing:** Answer all questions, even if unsure, as there is no penalty for incorrect answers.
-   **Practice Exams:** Utilize sample questions and practice exams to familiarize yourself with the format and content.


# Chapter 26: Congratulations AWS Certified SysOps Administrator Associate

## AWS Certification Paths
This lecture provides an overview of AWS certification paths, categorized by role:
-   **Architect Path:** Solutions Architect Associate, Solutions Architect Professional. Deep dive: Security Specialty.
-   **Application Architecture Path:** Developer Associate, DevOps Engineer Professional. Deep dive: Solutions Architect Professional.
-   **Operations Path:** SysOps Administrator Associate. Deep dive: DevOps Engineer Professional.
-   **Cloud Engineer Path:** SysOps Administrator Associate, Security Specialty. Deep dive: DevOps Engineer Professional, Advanced Networking Specialty.
-   **DevOps Path:** Developer Associate, DevOps Engineer Professional.
-   **Cloud DevOps Engineer Path:** Developer Associate, SysOps Administrator Associate (optional). Deep dive: DevOps Engineer Professional.
-   **Machine Learning Engineer Path:** Machine Learning Engineer Associate. Deep dive: DevOps Engineer Professional.
-   **DevSecOps Engineer Path:** SysOps Administrator Associate, Machine Learning Associate (if AI/ML projects), DevOps Engineer Professional, Security Specialty.
-   **Cloud Security Engineer Path:** SysOps Administrator Associate, Security Specialty. Deep dive: DevOps Engineer Professional, Advanced Networking Specialty.
-   **Cloud Security Architect Path:** Solutions Architect Associate, Security Specialty. Deep dive: Solutions Architect Professional.
-   **Software Development Engineer Path:** Developer Associate, DevOps Engineer.
-   **Network Engineer Path:** Solutions Architect Associate, Advanced Networking Specialty. Deep dive: Security Specialty.
-   **Cloud Data Engineer Path:** Solutions Architect Associate, Data Engineer. Deep dive: Security Specialty.
-   **Machine Learning Engineer Path:** Machine Learning Engineer Associate. Deep dive: Data Engineer, Machine Learning Specialty.
-   **Prompt Engineer Path:** AI Practitioner Foundational, Machine Learning Engineer Associate. Deep dive: Machine Learning Specialty.
-   **Machine Learning Ops Engineer Path:** Solutions Architect Associate, Machine Learning Associate. Deep dive: Data Engineer, DevOps Engineer.
-   **Data Scientist Path:** Machine Learning Engineer Associate. Deep dive: Machine Learning Specialty.

Foundational certifications (Cloud Practitioner, AI Practitioner) are often recommended as starting points.

## Congratulations
This lecture congratulates the student on completing the course and encourages them to leave a review. It also provides instructions on how to leave a review and encourages students to share their success on LinkedIn.

## Bonus Lecture: Next Steps and Discount Links
-   **Practice Exams:** Strongly recommended to take practice exams (available on Udemy) to assess knowledge, identify weak points, and prepare for the actual exam.
-   **Other Courses:** Information on other available courses (Solutions Architect Associate, Developer Associate, DevOps Engineer Professional) with discount links.