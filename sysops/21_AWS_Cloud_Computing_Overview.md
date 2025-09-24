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