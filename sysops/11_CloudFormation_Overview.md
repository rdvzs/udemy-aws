
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
