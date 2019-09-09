## Identity and Access Management in AWS
* Notes taken via Linux Academy Dev Associate course

### IAM Essentials
* Manage AWS users, groups, and roles
* Applies globally to all AWS regions
* By default, a new IAM user is not assigned ANY permissions to access AWS services (implicit DENY)
* Use principal of least privilege for users

#### Best practices for root accounts
* Delete root access keys
* Activate MFA
* Create individual IAM users
* Use groups to assign permissions
* Apply an IAM password policy

### IAM Policies
* Document that formally states one or more permissions
* Policies take effect immediately
* Cannot be attached directly to AWS resources
* Explicit denies **always** override explicit allows
    * Allows the use of "deny all" to quickly restrict all access to a user that may have multiple attached policies
* IAM provides pre-built policy templates to assign to users and groups
    * Examples: AdministratorAccess, PowerUserAccess (no IAM), Read-only (only view resources)
* More than one policy can be attached to a user or group at the same time

### IAM Users
* Users have unique access credentials that should not be shared with others
* User creds should **never** be stored on an EC2 instance (use roles)
* Users can have group and/or regular policies applied to them
* MFA can be configured on a per-user basis for login and/or resource access

### IAM Groups
* Allows you to assign policies to more than one user at a time
* Allows for easier access management to AWS resources

### IAM Roles
* Something another entity can assume which applies associated permissions via a temporary token
    * Consider an entity being AWS resources such as EC2/Lambda or non-AWS account holders that federate using AD or some other auth mechanism
* Roles must be used because policies cannot be attached to AWS services
* EC2 instances can only have one role attached at a time
* Other uses:
    * Users can assume a role for temporary access to AWS account and resources through AD or SSO identity access providers (Facebook, Google)
    * Creating cross-account access where a user from one account can assume a role within another account

### Security Token Service (STS)
* Allows you to create temporary creds to grant trusted users access to your AWS resources
* These creds are for short-term use only (a few minutes to several hours)
* Once expired, they can no longer be used
* When requested through an STS API call, the following creds are returned:
    * Security token
    * Access Key ID
    * Secret access key
* Key benefits:
    * No distributing or embedding long-term AWS creds in an application
    * Grant access to AWS resources without having to create an IAM identity
    * Creds don't have to be rotated or revoked
* When to use:
    * Identity federation
        * SAML allows for use of AD or other solutions
        * Web identity federation through Facebook, Google, or Amazon
    * Roles for cross-account access
    * Roles for Amazon Ec2/Lambda

#### STS API Call Examples
* AssumeRole - Cross-account delegation and federation through a custom identity broker
* AssumeRoleWithWebIdentity - Federation through a web-based identity provider
* AssumeRoleWithSAML - Federation through an enterprise identity provider compatible with SAML 2.0
* GetFederationToken - Federation through a custom identity broker
* GetSessionToken - Temporary credentials for users in untrusted environments

### IAM API Access Keys
* Required to make programmatic calls to AWS from:
    * AWS CLI
    * Tools for Windows Powershell
    * AWS SDKs
    * Direct HTTP calls using APIs for individual AWS services
* Key Facts:
    * Only available **ONE** time, when a new user is created or keys are rotated
    * AWS will not regenerate the same set of access keys
    * API creds must be associated with a user
    * Roles do not have API creds
    * You will only ever be able to see the Access Key ID from the AWS Console
    * If new API creds are required, current must be deactivated and a new set generated
    * Never store API keys on an EC2 instance

### AWS Key Management Service (KMS)
* Managed service to create and control the encryption keys used to encrypt your data
* Concepts:
    * **Customer Master Keys (CMKs)**:
        * Used to encrypt/decrypt up to 4KB of data and are the primary resource in KMS
        * CMKs generate, encrypt, and decrypt data keys that you use outside of KMS to encrypt your data
        * Two types of CMKs:
            * **Customer-managed**: CMKs you create, enable/disable, rotate, and which manage the policies that allow access to use the CMKs
            * **AWS-managed**: created, managed, and used by AWS services integrated with KMS (named like `aws/service-name` e.g. `aws/s3`)
    * **Data Keys**:
        * Encryption keys for encrypting large amounts of data or other data encryption keys
        * CMKs can generate, encrypt, or decrypt data keys
        * KMS does not manage or store your data keys -- they must be used inside your application
        * KMS cannot use data keys to encrypt data for you
    * **Envelope Encryption**:
        * Plaintext data is encrypted with data key
        * Data keys are encrypted with a **key encryption key (KEK)**
        * A KEK may be encrypted by another KEK, but eventually there is a master key (CMK) that decrypts one or more keys

#### KMS API Actions
* Encrypt - Encrypt plaintext using a CMK
* GenerateDataKey - Uses a CMK to return a plaintext and ciphertext version of a data encryption key
* Decrypt - Decrypts ciphertext that was encrypted with the Encrypt, GenerateDataKey, or GenerateDataKeyWithoutPlaintext API actions

### Amazon Inspector
* Automated security assessment service -- helps test network accessibility of instances and security state of applications running on those instances
* Can run security vulnerability assessments throughout your development pipeline or static prod systems

### Cognito Basics
* Not underneath the IAM service umbrella
* Superset of web identity federation; provides authentication, autorization, and user management for web an mobile apps
* Users can sign in directly with username and password or through a third party such as Facebook, Amazon, or Google
* Steps:
    * Users authenticate against a **user pool** to get a token
        * This is a user directory in Cognito for access via Cognito or third-party federation
    * Tokens are then exchanged for temporary AWS credentials to access AWS services. These are obtained through the **identity pool**
        * Identity pools support guest users or the following identity providers:
            * Cognito user pools
            * SSO with Facebook, Google, Amazon
            * OpenID Connect (OIDC) providers
            * SAML identity providers
            * Developer authenticated identities
* **Cognito identities**:
    * Create unique identifiers for application users
    * Authenticate identities with your own auth processes or third-parties
    * Supported unauthenticated identies (for example, signing up after using an app)
    * Save mobile user data (like mobile game data)
    * Use creds obtained to sync data with Cognito Sync
* **Cognito Sync**:
    * Sync user data across mobile devices and the web
    * Client libraries cache data locally