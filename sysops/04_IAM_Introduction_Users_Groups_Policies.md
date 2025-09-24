
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
