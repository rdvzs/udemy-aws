
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
