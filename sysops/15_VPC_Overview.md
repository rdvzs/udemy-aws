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