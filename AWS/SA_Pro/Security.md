# Security 
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Concepts
* **Principle of Least Privilege** - give users/services nothing more than privileges required to perform their intended function
* Important facets:
  * **Identity** - who are you? (IAM user, temp. creds, etc.)
  * **Authentication** - prove you are who you say (MFA, client-side SSL cert)
  * **Authorization** - are you allowed to do this? (IAM policies)
  * **Trust** - Do other entities that I trust say they trust you? (cross-account access, SAML/web identity federation)

### SAML vs. OAuth vs. OpenID
* **SAML 2.0**
  * Can handle authorization and authentication
  * XML-based
  * Can contain user, group membership, and other useful info
  * Assertions in the XML for authentication, attributes, or authorization
  * Best suited for enterprise SSO
* **OAuth 2.0**
  * Allows sharing of protected assets without having to send login creds
  * Handles authorization only
  * Issues token to client
  * Application validates token with Authorization Header
  * Delegate access, allowing client apps to access information on behalf of user
  * Best suited for API authorization between apps
* **OpenID Connect**
  * Identity layer on top of OAuth 2.0 adding authentication
  * Uses REST/JSON message flows
  * Supports web clients, mobile clients, and Javascript clients
  * Extensible
  * Best suited for consumer-grade SSO

### Compliance
* AWS has implemented certain processes, practices, and standards as prescribed by many national/international standards bodies
* You can find supporting documentation and compliance reports in **AWS Artifact**

## Multi-Account
* Most large orgs need multiple AWS accounts
  * Segregation of duties, cost allocation, agility
  * Need methods to manage and maintain
* Available tools:
  * **AWS Organizations** - manage accounts in a hierarchical structure
  * **Service Control Policies** - policies that can be applied to restrict access to certain actions/services in an account
  * **Tagging** - assign metadata to AWS resources
  * **Resource Groups** - create groups for resource that share commonality
  * **Consolidated Billing** - roll billing from multiple accounts up to a single payer

### Account Structures
* **Identity**
  * Manage all user accounts in one location
  * Users trust relationship from IAM roles in sub-accounts to identity account to grant temporary access
  * Variations inclue business unit, deployment environment, geography
* **Logging**
  * Centralized logging repo
  * Can be secured to be immutable
  * Can use SCPs to prevent sub-accounts from changing logging settings
* **Publishing**
  * Common repo for AMIs, containers, code
  * Permits sub-accounts to use pre-approved standardized services or assets
* **InfoSec**
  * Hybrid of consolidated security and logging
  * Allows one point of control and audit
  * Logs cannot be tampered with by sub-account users
* **Central IT**
  * IT can manage IAM users/groups while assigning to sub-account roles
  * IT can provide shared services and standardized assets (AMIs, databases, etc.)

## Network Controls
* **Security Groups**
  * Stateful virtual firewall; interface-level control of inbound and outbound traffic for IP protocol traffic
  * Port or port ranges
  * Inbound - source IP, subnet, or another SG
  * Outbound - dest. IP, subnet, or another SG
* **NACLs**
  * Stateless firewall that acts as a firewall at the subnet-level
  * Default allows all traffic
  * Further restrict traffic vs. SGs using DENY statements
  * Remember ephemeral ports

## AWS Directory Services
* **AWS Cloud Directory** - Cloud-native directory to share and control access to hierarchical data between apps
* **Amazon Cognito** - Sign-up/sign-in functionality that scales to millions of users and federated to social media services
* **AWS Directory Service for Microsoft Active Directory** - fully-managed MS AD (standard or enterprise)
* **AD Connector** - allows on-prem users to log in to AWS services with their existing AD creds. EC2 instances can also join the AD domain
* **Simple AD** - low-cost, low-scale AD implementation using Sambda

### AD Connector vs. Simple AD
* **AD Connector**
  * Must have existing AD
  * Existing AD users can access AWS assets via IAM roles
  * Supports MFA via existing RADIUS-based MFA infra

* **Simple AD**
  * Standalone AD based on Samba
  * Support user accounts, groups, group policies, and domains
  * Kerberos-based SSO
  * MFA not supported
  * No trust relationships

## Creds/Access Management
* **Security Token Service (STS)** - grant temporary access credentials to users and applications
* **Cognito** - generally used for mobile application auth

### Typical STS Flow
* App -> Identity Broker -> AD/other identity store -> STS temporary creds -> app -> AWS services
* With federation from IdP: app -> identity broker -> IdP (social media, Cognito) -> STS temporary creds -> app -> AWS

### Token Vending Machine
* Common way to issue temp creds for mobile apps
* Anonymous TVM - used as a way to provide access to AWS service only; does not store identity
* Identity TVM - used for registration, login, and authorizations
* AWS recommends using Cognito for this concept

### Secrets Manager
* Store passwords, encryption keys, API keys, SSH keys, etc.
* Alternative to storing passwords or keys in a vault
* Access secrets via API with fine-grained access control
* Automatically rotate RDS database creds from MySQL, PostgreSQL, and Aurora

## Encryption

### Key Management Service
* Key storage, management, and auditing
* Integrated with many AWS services
* Import your own keys or have KMS generate them
* Control who manages and accesses keys
* Audit use of keys with CloudTrail
* Validated by compliance frameworks (PCI DSS Level 1, FIPS 140-2 Level 2)

### CloudHSM
* Dedicated, single-tenant hardware device
* Must be within a VPC
* Does not natively integrate with AWS services; requires custom scripting
* Offload SSL from web servers, act as issuing CA, and enable TDE for Oracle databases
* Classic version
  * SafeNet Luna SA
  * Upfront cost of $5000
  * Must buy second device for HA
  * FIPS 140-2 Level 2
* Current version
  * Proprietary hardware
  * Pay per hour, no upfront cost
  * Clustered
  * FIPS 140-2 Level 3

### AWS Certificate Manager
* Managed service that lets you provision, manage, and ddeploy public/private SSL certs
* directly integrated into many AWS services like CloudFront, ELB, and API Gateway
* Free public cert to use with AWS services
* Import third-party certs if desired
* Supports wildcard domains to cover all subdomains
* Managed cert renewal
* Can create managed private CA for internal/proprietary apps

## DDoS Attacks
* Bad actors send some sort of command to many compromised machines. These machines then execute the command to flood a specific target

### Amplification/Reflection Attacks
* NTP servers have a command called MONLIST which returns the previous 600 IP addresses that it's transacted with (35,600 byte message)
* Bad actor can spoof source IP of packet to a targeted device/service; many large packets overwhelming the service

### Layer 7 Attacks
* HTTP GET Flood - bad actor floods web server with many HTTP Get requests, potentially causing downtime of the web server and potentially the backing database

### Best Practices
* Minimize attack surface by using NACLs, SGs, public/private subnets effectively
* Scale to absorb the attack by using ASGs, CloudFront, and static web content served by S3
* Safeguard exposed resources with Route 53, Shield, and WAF
* Learn normal behavior with GuardDuty and CloudWatch

## IDS/IPS
* **Intrusion Detection System** - watches network traffic and alerts on potential malicious activity
* **Intrusion Prevention System** - tries to prevent exploits by analyzing network packets for suspicious content
* Both typically comprised of a **collection/monitoring systems** and **monitoring agents** on each system
* Logs collected or analyzed in CloudWatch, S3, or third-party tools are sometimes called a **security information and event management (SIEM) system**