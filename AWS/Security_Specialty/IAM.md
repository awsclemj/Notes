- [Identity and Access Management](#identity-and-access-management)
  - [Exam tips](#exam-tips)
  - [Identity basics](#identity-basics)
  - [Cognito](#cognito)
  - [Organizations](#organizations)
    - [Account boundaries in AWS](#account-boundaries-in-aws)
  - [IAM policy types](#iam-policy-types)
    - [Identity policies](#identity-policies)
    - [Resource policies](#resource-policies)
      - [S3](#s3)
      - [KMS](#kms)
    - [Cross-Account](#cross-account)
  - [Policy evaluation](#policy-evaluation)
    - [Exam flow for policies](#exam-flow-for-policies)
    - [Conditions and "Nots"](#conditions-and-nots)

# Identity and Access Management
Notes taken from internal bootcamp

## Exam tips
* Be prepared to evaluate questions based on a vacuum, not reality
* Some questions are not written well. Assume that these questions are being asked by a customer
* Choose the "least wrong" answer
* Don't assume there are unknown variables. Only go off explicitly stated facts

## Identity basics
* In the beginning: users on every system. Doesn't scale well
* Then came **identity stores/user directories**
  * Organized structure of **principals** and their associated metadata
  * Active Directory, Okta, etc.
* Identity Provider (IDP):
  * System that manages identity information
  * Okta, ADFS
  * Handles SAML, OIDC, etc.
* Service provider (SP) or relaying party (RP)
  * The "server" or service that your principals are accessing which determines the identity of principals
* Who - can access - what?
  * Who - user directory
    * AWS Directory Service
    * IdP (kind of. Basic idea of users. Not true directory)
      * Cognito
      * AWS SSO
    * AWS IAM
  * Can Access
    * IdPs:
      * Amazon Cognito
      * AWS SSO
    * AWS IAM
  * What - resource management
    * AWS Resources

## Cognito
* User Pools
  * Native directory or federated principals
  * Customizable auth flows
  * Advanced features for ID management
  * Standardizes app auth through Amazon Cognito JSON Web Tokens (JWT)
    * Blobs of JSON that show you parameters about a user (stateless mechanism)
  * Normalizes user schema between IdPs
* Identity Pools
  * Credentials exchange service
  * Securely vend STS creds for end user access to AWS services
  * Auth is handled through exchanging Cognito tokens, or federation to other providers

## Organizations
* Ability to organize accounts into hierarchical Organizational Units (OU)
* Apply account-level policy guardrails (SCPs)
* Consolidate billing and ease of billing attribution
* Take advantage of AWS Organizations-aware service integrations

### Account boundaries in AWS
* AWS account
* AWS root user
  * Lock this away! It can do everything.
  * Enable MFA
  * This user can destroy things
* IAM users
  * Username/password
  * Access keys
  * These authenticate you to AWS services
* IAM groups
  * Groups of users that have a single permissions policy applied
* IAM roles
  * You should be using these all the time rather than IAM users/groups
  * Temporary STS credentials via federation

## IAM policy types
* Guardrails
  * Things that set a maximum permission
  * Can have several of these, but without a grant, requests are denied
  * Examples:
    * SCPs
    * Permission boundaries
    * Session policies
    * Endpoint policies
* Grants
  * Things that give a permission
  * Examples:
    * Identity-based policies
    * Resource-based policies
      * Role trust policies (who can assume this role?)
      * KMS key policies

### Identity policies
* Managed
  * AWS-managed and customer-managed
  * You can apply managed policies to multiple principles
  * Change permissions in the managed policy to change permissions for all principals it's attached to
* Inline
  * Specific, narrowly-scoped policy that only applies to the principal it's attached to

### Resource policies

#### S3
* S3 is special. It has block public access settings, bucket polices, and also ACLs
* ACLs
  * Uses an old construct called a canonical ID
  * Exist at bucket-level and object-level

#### KMS
* Be careful with KMS policies; they are strange.
* If the resource policy has a principal for the account root, it delegates the KMS permissions to an identity policy
* That said, if KMS has a role that you're assuming as a principal, the role will only be able to perform the permissions allowed on the resource policy

### Cross-Account
* Two ways to work with resources in another account:
  * Assume roles in another account
    * If the services doesn't have resource-based policies
    * The account has permissions for the role
    * Trust policies (who can assume this role?)
      * Users, roles
  * Cross-account resource access
    * Delegate access via resource permissions

## Policy evaluation
* It's all one policy evaluation. Stack each document together and determine access privileges
* Ex:
  * Role policy -> endpoint policy -> resource-based policy
* Be careful when you have multiple policies
  * Ex: permissions boundary + identity-based policy + resource-based policy
  * Explicit deny will always deny access
  * If there is an allow, ensure it is for a grant policy

### Exam flow for policies
1. Understand policy types and who is accessing what
2. Look for deny statements that make the question easy to answer
3. Look for the grant policy
4. Double check condition statements

### Conditions and "Nots"
* Keep a look out for conditions in policies
* Also look out for "nots"
  * NotPrincipal in explicit denies adds an explicit deny to every principal except the NotPricipals
  * NotResource - deny access to all resources except those listed in NotResources
  * NotAction - deny all actions except those listed in NotAction
