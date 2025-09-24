
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
