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