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