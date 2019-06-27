## Reliability Pillar
* Ability of a system to recover from failures, dynamically acquiring comupting resources, and mitigating disruptions such as misconfigs or network issues

### Design Principles
* Traditional environment:
    * Rarely test recovery procedures
    * Manually recover from failures
    * Single points of failure
    * Guessing capacity

* In the cloud:
    * Test recovery procedures
    * Automatically recover from failure
    * Scale horizontally to increase system availability
    * Stop guessing capacity
    * Manage change in automation

### Key Services
* CloudWatch is the biggest key service for reliability
* Foundations
    * IAM - control access to resources
    * VPC - isolated cloud segment
    * Trusted Advisor - real-time best practices
    * Shield - DDoS protection
* Change Management
    * CloudTrail - logs API calls as events
    * Config - inventory of AWS resources and configuration. Records changes to configurations
    * CloudWatch - alerts on metrics
    * Auto scaling - on-demand horizontal scaling 
* Failure Management
    * CloudFormation - orderly/predictable way to provision infrastructure
    * S3 - durable service to keep backups
    * Glacier - durable archives
    * KMS - managed keychain that integrates with many AWS services

### Best Practices
* Foundations
    * Ensure sufficient network bandwidth/topology
    * Physical limits and resource constraints (rate limits, service limits)
    * Ensure high availability (using AZs, ELB, Autoscaling, redundant network connectivity)
* Change Management
    * Monitor/audit to quickly identify trends
    * Send automated notifications when certain limits/constraints are reached (SNS)
    * Use CloudWatch automated responses to certain key issues (such as a server not responding)
    * Ensure you are using Autoscaling to dynamically meet demand
* Failure Managment
    * Automation can react to monitoring data
    * Backup data regularly
    * Rather than fixing broken resources, replace the broken resource and analyze the broken one out-of-band
    * Test recovery from failures based on previous analysis
    * Have a DR strategy