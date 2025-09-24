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