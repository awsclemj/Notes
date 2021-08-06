- [Deployment and Ops Management](#deployment-and-ops-management)
  - [Types of Deployments](#types-of-deployments)
    - [Rolling Deployment](#rolling-deployment)
    - [A/B Deployment](#ab-deployment)
    - [Canary Release](#canary-release)
    - [Blue-Green Deployment](#blue-green-deployment)
  - [Continuous Integration and Deployment](#continuous-integration-and-deployment)
  - [Elastic Beanstalk](#elastic-beanstalk)
  - [CloudFormation](#cloudformation)
    - [Stack Policies](#stack-policies)
    - [Best Practices](#best-practices)
  - [Elastic Container Service](#elastic-container-service)
    - [Launch Types](#launch-types)
  - [API Gateway](#api-gateway)
  - [Management Tools](#management-tools)
    - [Config](#config)
    - [OpsWorks](#opsworks)
      - [OpsWorks Stacks](#opsworks-stacks)
    - [Systems Manager](#systems-manager)
  - [End-User Computing](#end-user-computing)
    - [Workspaces and AppStream](#workspaces-and-appstream)
    - [Amazon Connect and Amazon Chime](#amazon-connect-and-amazon-chime)
    - [WorkDocs and WorkMail](#workdocs-and-workmail)
    - [WorkLink and Alexa for Business](#worklink-and-alexa-for-business)
  - [Machine Learning](#machine-learning)

# Deployment and Ops Management
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Types of Deployments
* **Big Bang** - all at once rollout
  * Risky but takes less time
* **Phased rollout** - gradual release over time
  * Less risk but takes more time
* **Parallel adoption** - implement new system while still operating old
  * Low risk, but takes a lot of time

### Rolling Deployment
* Assume a basic architecture with R53, ELB, and and ASG
* Update the ASG launch config with a new AMI -- start terminating old instances

### A/B Deployment
* Have two separate versions running in parallel; use R53 weights to slowly increase traffic to new version

### Canary Release
* One instance in an ASG is deployed to prod -- wait to see if there are errors on the "canary"

### Blue-Green Deployment
* Two separate versions of the app -- identical infra, different app versions.  
* Switch traffic over to the new (green) version. 
* If there are any problems, can easily roll back to previous version. 
* Ways to do this:
  * Update Route 53 DNS to point at new ELB/instance
  * Swap ASG primed with new version to the ELB
  * Update ASG launch config with new version and terminate instances
  * Swap environment URL of Elastic Beanstalk
  * Clone stack in OpsWorks and update DNS
* Possible issues:
  * If your data store is tightly coupled with code changes
    * Make schema changes forward and backward compatible
  * Upgrade requires special upgrade routines to be run
  * COTS products may not be blue-green friendly

## Continuous Integration and Deployment
Combination of the following concepts:
* **Continuous Integration** - Merge code changes back to main branch as frequently as possible with automated testing
* **Continuous Delivery** - You have automated your release process to the point that you can deploy at the click of a button
* **Continuous Deployment** - Each code change that passes all stages of the release process is released to prod with no human intervention

Some considerations:
* Objective is to create smaller, incremental improvements and features
* Lowers deployment risk to limit negative impact
* Test automation must be strong to enable this
* Microservices lend themselves well to this pattern

## Elastic Beanstalk
* Orchestration service that makes it push-button easy to deploy scalable web apps
* Wide range of supported platforms - Docker, PHP, Java, Node.js for example
* Multiple environments within an application
* Great for ease of deployment, but not great for control/flexibility 
* Deployment options:
  * All at once - new app version is deployed to existing instanes all at once
  * Rolling - One by one; new app version is deployed to instances in batches
  * Rolling with Additional Batch - Launch new version instances prior to taking any old versions out of service
  * Immutable - Launch a full set of new version instances in separate ASG and cuts over when a health check passes
  * Traffic Splitting - Percent of client traffic is routed to new instances for canary testing
  * Blue/Green - CNAME entry changed when new version is fully up, leaving the old version in place until the new version is verified

## CloudFormation
* Infra as Code
* Use JSON or YAML to model/provision your architecture
* Repeatable, automatic deployments and rollbacks
* Nest components for reusability
* Supports over 300 resource types, with custom resources via Lambda

### Stack Policies
* Protect specific resources within your stack from being unintentionally deleted/updated
* Can be added or removed via console/CLI when creating a stack
* Can only be added via CLI for existing stacks
* Stack policies cannot be removed, but they can be modified via CLI
* By default, denies all changes

### Best Practices
* AWS provides helper scripts which can help you install software and start services on EC2 instances
* Use CloudFormation to make changes to your stack rather than changing them manually
* Make use of Change Sets to identify potential trouble spots in updates
* Use Stack Policies to explicitly protect sensitive portions of your stack
* Use version control systems to track template updates 

## Elastic Container Service
* Managed container orachestration platform -- ECS and EKS available
* ECS
  * AWS-specific platform that supports Docker containers
  * Considered simpler to learn/use
  * Leverages AWS services like R53, ALB, CloudWatch
  * Tasks - instances of containers that are run on underlying compute but essentially isolated
  * Limited extensibility
* EKS
  * Compatible with upstream k8s; easy to lift and shift from other k8s platform
  * Considered more feature-rich and complex; steep learning curve
  * Hosted k8s platform that handles many things internally
  * Pods - containers collocated with one another and can have shared access to one another
  * Extensible via a wide variety of add-ons

### Launch Types
* EC2
  * Explicitly provision EC2 instances
  * Responsible for upgrading, patching, and maintenance of EC2 pool
  * Handle cluster optimization
* Fargate
  * Control plane asks for resources and Fargate automatically provisions
  * Provisions compute as needed
  * Fargate handles cluster optimization

## API Gateway
* Managed, HA service to frontend REST APIs
* Backed with custom code via Lambda, a proxy to another AWS service, or any other HTTP API
* Regionally based, private, or edge optimized (via CloudFront)
* Support API keys and usage plans for user identification, throttling, or quotas
* Since CloudFront is leveraged, custom domains and SNI are supported

## Management Tools

### Config
* Assess, audit, and evaluate configurations of AWS resources
* Useful for config management as part of an ITIL program
* Creates baseline of various configuration settings and can track variations against that baseline
* Config Rules can check resources for certain desired conditions and if violations are found, the resources are flagged noncompliant
  * Is backup enabled on RDS?
  * Is CloudTrail enabled?

### OpsWorks
* Managed instance of Chef and Puppet
* Provide config management to deploy code, automate tasks, configure instances, perform upgrades, etc.
* Three offerings: Chef Automate, Puppet Enterprise, and Stacks

#### OpsWorks Stacks
* OpsWorks Stacks is an AWS creation that uses embedded Chef solo client installed on EC2 instances to run Chef recipes\
  * Also supports on-prem servers if the agent is installed
* Stacks are collections of resources needed to support a service or application
* Layers represent different components of the application delivery hierarchy
  * Ex. Ec2 instances, RDS instances, and ELBs are examples
* Stacks can be cloned within the same region
* OpsWorks is global, but you must specify a region for stacks

### Systems Manager
* Centralized console and toolset for a wide variety of system management tasks
* Designed to manage large fleets of systems -- tens, hundreds, thousands
* SSM agent enables SSM features and supports all OSes back to Windows Server 2003
  * Installed by default on most modern AMIs

Walkthrough of tools:
* **Inventory** - collect OS, application, and instance metadata
* **State Manager** - create states that represent a certain configuration that is applied to instances 
* **Logging** - CloudWatch Log agent and stream logs directly to CloudWatch from instances
* **Parameter Store** - shared secure storage for config data, connection strings, passwords, etc. 
* **Insights Dashboard** - account-level view of CloudTrail, Config, Trusted Advisor 
* **Resource Groups** - Groups resources through tagging
* **Maintenance Windows** - Define schedules for instances to patch, update apps, run scripts, etc. 
* **Automation** - automating routine maintenance tasks and scripts
* **Run Command** - Run commands and scripts without logging in via SSH/RDP
* **Patch Manager** - automates process of patching instances
  * Patch baselines - auto-approve certain patches based on severity level; can create custom baselines

SSM document types:
* **Command** - used with Run Command or State Manager
  * Run command uses command docs to execute commands
  * State Manager uses command docs to apply a configuration
* **Policy** - used with state manager to enforce a policy on your targets
  * Ex. define new vs. old web server states
* **Automation** - use with Automation service. Performs certain script actions on targets

## End-User Computing

### Workspaces and AppStream
* Workspaces is a desktop as a service -- access via client
* AppStream encapsulates certain applications -- allowing you to access them via web browser 
* Everything in both services lives on AWS and can be tightly managed/controlled
* Use case: remote or seasonal workers using virtual desktops or hosted apps instead of furnishing hardware

### Amazon Connect and Amazon Chime
* Connect - fully-managed cloud-based contact center solution
  * Inbound/outbound telephony
  * Interactive voice response
  * Chatbot
  * Analytics
  * Integrates with popular CRM systems
* Chime - online meeting and video conferencing
  * Supports desktop sharing, group chat, and session recording

### WorkDocs and WorkMail
* WorkDocs - online document storage and collaboration
  * Similar to Google Drive, OneDrive, etc.
  * Version management, sharing, and collaborative edits
* WorkMail - fully managed email and calendar
  * Compatible with MS Exchange, IMAP, Android, and iOS mail clients

### WorkLink and Alexa for Business
* WorkLink - secure access to internal web apps for mobile devices
  * Mobile user makes a request -> rendered on a secure machine -> image sent to mobile client
* Alexa for Business - deploy Alexa functionality and skills internally for the enterprise
  * Management functionality more appropriate for an enterprise org than individual commercial Alexa devices

## Machine Learning
* ML frameworks and infra - raw processing with custom ML models
  * Framworks:
    * mxnet
    * TensorFlow
  * Interfaces:
    * Gluon
    * Keras
  * Infra:
    * Amazon Greengrass
    * EC2
    * Deep learning AMIs
* ML Services - train and tune Amazon-provided models
  * Sagemaker - umbrella term for many tools -- managed Jupiter notebook
* AI services - easy to use, no ML experience necessary
  * Comprehend - Natural Language Processing; finds insights and relationships within text. Ex. sentiment analysis on social media posts
  * Forecast - combines time-series data with other variables to deliver highly accurate forecasts
  * Lex - build conversational interfaces that can understand intent and context of natural speech
  * Polly - test-to-speech service supporting multiple languages, accents
  * Rekognition - image and video analysis to recognize objects, people, activities, and facial expressions
  * Textract - Extract text, context, and metadata from scanned documents
  * Translate - translate text to and from many languages
  * Transcribe - speech-to-text from an audio file
  * Personalize - Recommendation engine as a service based on demographic behavioral data