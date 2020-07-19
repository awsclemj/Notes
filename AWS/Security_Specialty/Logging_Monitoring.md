- [Logging and Monitoring](#logging-and-monitoring)
  - [Logging in AWS](#logging-in-aws)
    - [CloudTrail](#cloudtrail)
    - [CloudWatch](#cloudwatch)
    - [VPC Flow Logs](#vpc-flow-logs)
    - [AWS Config](#aws-config)
      - [Best practices](#best-practices)
    - [Amazon Inspector](#amazon-inspector)
    - [Trusted Advisor](#trusted-advisor)
    - [Service-specific Logging](#service-specific-logging)
      - [S3 Events](#s3-events)
      - [S3 Server Access Logs](#s3-server-access-logs)
      - [CloudFront Access Logs](#cloudfront-access-logs)
      - [Load Balancer Access Logs](#load-balancer-access-logs)
      - [RDS Log Files](#rds-log-files)
      - [Redshift](#redshift)
      - [Data Event Logs](#data-event-logs)
  - [Security Hub](#security-hub)
  - [Analytics](#analytics)
    - [Athena](#athena)
    - [QuickSight](#quicksight)
    - [ELK Stack](#elk-stack)
  - [Log Retention](#log-retention)
    - [Glacier with Vault Lock](#glacier-with-vault-lock)
  - [Logging at Scale](#logging-at-scale)

# Logging and Monitoring
Notes taken from internal security bootcamp and Linux Academy course

## Logging in AWS
* Logs are useful because:
  * Compliance aid
  * Visibility into activity
  * Detect data exfiltration
  * Automate security analysis
  * Troubleshoot anamolies
  * Analyze permissions
* Best practices:
  * Centralize logs
  * Enable audit and access logging capabilities
  * Use secure, durable storage
  * Implement lufecycle policies
  * Use meaningful names for log organization
  * Analyze logs
  * Create actionable reports
  * Configure appropriate alerting

### CloudTrail
* Actions taken on services (API calls) are logged via Cloudtrail
* Enabled by default on all AWS accounts (retained 90 days in **Event History**)
* Typical pattern: enable in all Org accounts and store in S3 bucket
* Integrates with AWS Organizations; all events in the org are logged
* Lifecycle management
  * Cloudtrail -> S3 -> Glacier -> Delete
* Logs tell you:
  * Who, when, where, what
* Trails:
  * Configure for single region or multi-region
  * Allow for multi-account logging
  * S3-SSE is enabled by default (can be changed to S3-KMS)
  * Have the option to enable log file validation
    * Need to have connection to AWS to validate integrity
    * Need read access to Trail bucket
    * By design, you **cannot** move/change files delivered by CloudTrail
    * Generates SHA-256 hash for each delivered log file
    * Digest files are also delivered each hour which allow you to prove whether log files have been tampered with
    * Use `validate-logs` API/CLI commands to validate the log integrity

### CloudWatch
* **Logs**:
  * Logs from many services can be centralized in CloudWatch, including logs from Cloudtrail
  * Can export to S3
  * Can stream to Lambda or Elasticsearch
  * Components:
    * Event - single record of activity by a monitored resource
    * Stream - sequence of log events from same source
    * Groups - Collection of streams with the same access control, monitoring, and retention settings
    * Metric filters - Assigned to log groups; extracts data based on a defined filter and converts it to a metric data point 
    * Retention settings - Period logs are kept. Applies to all streams in a group. 1 day to never expire
* **Metric Filters**:
  * Used to make custom metrics from log data
  * Assigned at the log group level (filters all streams in the group)
  * Filter matches are assigned to a custom CloudWatch metric with a defined namespace/name
  * Alarms are assigned to the metric filter and can trigger:
    * SNS
    * EC2/Autoscaling actions
* **Metrics**:
  * Data about the performance of systems
  * Default: 5-minute intervals, can be customized (extra charge)
* **Events**:
  * Stream of events about changes in your resources
  * Configure some sort of response using an Events Rule
  * Performs in near real-time
  * Many different configurable targets:
    * EC2, Lambda, ECS
    * Kinesis Data Streams/Firehose
    * Systems Manager Run Command/Automation
    * Code* products
    * SNS and SQS
* **Event Buses**:
  * A bus can accept events from other accounts
  * Generally a security or master account in an Org receives these events
    * Need to add permissions for specific account(s) or org ID to accept events from
  * Need event rules in sender account that are configured to send to the receiving account as a target
* **Alarms**:
  * Alarms can send out notifications based on events, metrics, or math expressions
* **Dashboard**:
  * Create customized views of metrics, alarms, etc.
  * Supports cross-account and cross-region

### VPC Flow Logs
* Captures info about IP traffic going to/from network interfaces in your VPC
* Uses:
  * Diagnose overly-resitrctive SG rules
  * Monitor traffic
  * Determine direction of traffic to/from network interfaces
* Can be stored in CW logs or S3
* Does not include packet captures
  * You can use traffic mirroring for this; only available on Nitro

### AWS Config
* Evaluate AWS resource configuration for desired settings
* Retrieve current/historical configurations of resources
* Receive notification on change
* View relationships between resources 
* Integrates with AWS Organizations
* Benefits:
  * Audit and compliance
    * Maintain history of all configuration changes
    * Verify changes do not violate policies
  * Operational governance
    * Verify configuration changes don't violate policies
    * Ensure configuration changes are tied to approved change requests
  * Security intelligence
    * Incident/breach analysis
    * Identify vulnerable resources
  * Integration with IT service management/config management DB
  * Custom rules:
    * Define custom rules via Lambda function
* Requires read-only role to enable to recorder
* (Optional) can stream changes to SNS (ex. for external config management)
  
#### Best practices
* Configure max visibility (all regions, all resources)
* Store history and snapshots in a secured S3 bucket
* Use CW Events to filter AWS Config notifications and take action
* Turn on periodic snapshots with min frequency of once per day
* Author custom rules with AWS Config Rule Development Kit (RDK)

### Amazon Inspector
* Analyze behavior of AWS resources and check for potential vulnerabilities
* Components:
  * Target: collection of AWS resources
  * Assessment template: Security rules that produce findings
  * Assessment run: Applying the assessment template to a target
* Must install an agent on the EC2 instance
* Features:
  * Configuration scanning and activity monitoring engine
  * Built-in content library
    * Rules and reports built-in to Inspector
    * Best practice, common compliance standards, and vulnerability analysis
  * Provides recommendations for resolving issues

### Trusted Advisor
* Best practice recommendation engine that provides proactive real-time guidance
* Provides details how to fix issues and links to documentation
* Example detection
    * EIPs not associated with running instnaces
    * MFA not enabled on root account
    * Vulnerable security groups
    * Low EC2 instance utilization
      * Can extend this to send a CW Event that triggers Lambda to stop instance

### Service-specific Logging

#### S3 Events
* Can send event data based on object-level actions (ex. create/delete)
* Feeds data into Lambda, SNS, or SQS
* Pushes instead of polls

#### S3 Server Access Logs
* Provides detailed records for requests being made to a bucket
  * Request-URI
  * Referer
  * User-Agent
* Especially useful for static S3 websites, or if you have sensitive information in the bucket

#### CloudFront Access Logs
* Create log files that contain detailed information about user requests
* Available for both web and RTMP distros
* Log files are aggregrated on a per-distro basis

#### Load Balancer Access Logs
* Disabled by default; enabled and configured per LB
* Stored in S3
* Log files are automatically encrypted using SSE-S3

#### RDS Log Files
* Viewable in the RDS console or accessible via AWS CLI
* Each database engine has different logging formats
* RDS logs can be published to CW Logs

#### Redshift
* Disabled by default; can be enabled and configured per cluster
* Stored in S3
* Logs the following information:
  * Connection log
  * User log
  * User activity log

#### Data Event Logs
* Provide visibility to resource operations
* High volume activities:
  * Amazon S3 object operations
  * Lambda invocations

## Security Hub
* Single pane of glass to centralize findings/results from AWS security services and third-party integrations
* Automate remediation actions via Lambda/Step Functions/etc. 
* Insights
  * Filter/consolidate findings based on things such as AMI
  * Custom actions
    * Send findings to CW Events decorated with some custom data
* Automated security checks based on industry standards
  * CIS AWS Foundations
  * PCI DSS
  * AWS Best Practices

## Analytics

### Athena
* Serverless, interactive SQL query service
* Point to an S3 bucket and start querying
* Pay per query
* Integrates with AWS Glue
* Analyze infrastructure, operation, and application logs
* Interactive analytics using popular BI tools

### QuickSight
* Supports CloudTreail analytics with minimal configuration
* Written in SQL and can be queried using Athena

### ELK Stack
* Elasticsearch, Logstash, Kibana
* If building on AWS, you're probably using Kinesis Firehose instead of Logstash
* These tools scale with your logs and allow you to visualize your various logs

## Log Retention
* S3 bucket holding your logs needs to be very well secured
* Best practices:
  * Versioning
  * Object lock
    * WORM - write once read many
  * MFA delete
  * Replication (CRR, SRR)

### Glacier with Vault Lock
* Allows you to easily set compliance controls on individual vaults
* Enforces via a lockable policy
* WORM
* Time-based retention
* Policy lockdown

## Logging at Scale
* Log archive account
  * Versioned S3 bucket
    * Restricted
    * MFA delete
  * CloudTrail logs
  * Security logs
  * Single source of truth
  * Alarm on user login
* Security account
  * Security tools/auditing
  * GuardDuty
  * Cross-account logging to log archive account
* Shared services
  * DNS
  * LDAP/AD/etc.
* SCPs
  * Deny `cloudtrail:StopLogging` from being called in your Organization