
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
