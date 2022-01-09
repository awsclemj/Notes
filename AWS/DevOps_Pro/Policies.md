- [Policies and Standards Automation](#policies-and-standards-automation)
  - [Service Catalog](#service-catalog)
  - [Trusted Advisor](#trusted-advisor)
  - [Systems Manager](#systems-manager)
  - [Organizations](#organizations)
  - [Secrets Manager](#secrets-manager)
  - [Macie](#macie)
  - [Certificate Manager](#certificate-manager)

# Policies and Standards Automation
Notes taken from ACG DevOps Pro 2020 course.

## Service Catalog
* Enables org to create and manage catalogs of products that are approved for use on AWS
* Has an API that provides programmatic control over all end-user actions as an alternative to the AWS Console
* Allows you to create your own custom interfaces and apps
* Admins can create and distribute application stacks called **products**
* Products can be grouped into folders called **portfolios**
* Users can then launch and manage products themselves without requiring access to the AWS services or AWS Console
  * Users only see products they are supposed to see

## Trusted Advisor
* A service that provides you with real-time guidance to ensure your AWS resources are provisioned and managed following AWS best practices
* Categories
  * Cost Optimization - Are you paying too much?
  * Performance - Are you under-utilizing resources?
  * Security - Is your account secure?
  * Fault tolerance - Are you ready for a possible incident?
  * Service limits - Are you close to breaching your limits?
* Basic and Developer AWS Support plans get access to 7 core TA checks
* Business and Enterprise support get all TA checks

## Systems Manager
* Management service that assists with:
  * Collecting software inventory
  * Applying OS patches
  * Creating system images
  * Configuring operating systems
  * Managing hybrid cloud systems
  * Reducing costs
* Comes with following features:
  * Run command
  * State manager
  * Inventory manager
  * Maintenance window
  * Patch manager
  * Automation 
  * Parameter store

## Organizations
* Policy-based management of multiple AWS accounts
* Enterprises often have several accounts so they can govern each account individually based on some set of criteria. Examples:
  * Environment (e.g. prod, dev, and test accounts)
  * Project (e.g. Project 1, project 2, project 3 accounts)
  * Business units (sales, support, developers)
* Features:
  * Programmatically create new accounts
  * Maintain groups of accounts
  * Set policies on grouped accounts

## Secrets Manager
* Helps you protect secrets needed to access your applications, services, and IT resources
* Encrypts secrets at rest using your own encryption keys stored in KMS
* Secrets include:
  * Database credentials
  * Passwords
  * API keys
  * Text
* Store and control access via console/CLI/API/SDK
* Hardcoded credentials in code is replaced with an API call
* Secrets can be rotated automatically based on a schedule

## Macie
* A service that uses machine learning to automatically discover, classify, and protect sensitive data in AWS
* Features:
  * Can recognize PII
  * Provides a dashboard
  * Monitors data access activity for anomalies
  * Generates detailed alerts when it detects a risk of unauthorized access or accidental data leaks
  * Protects data in S3, with more data stores planned in the future

## Certificate Manager
* Provision, manage, and deploy SSL/TLS public and private certificates
* Features:
  * Centrally manage certificates
  * Audit use of each certificate in CloudTrail
  * Private certificate authority
  * Integration with AWS frontend services like Elastic Load Balancing, CloudFront, API Gateway
  * Import third party certs from other CAs