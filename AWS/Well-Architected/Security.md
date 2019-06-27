## Security Pillar
* Ability to protect information, systems, and assets, while delivering business value through risk assessments and mitigation strategies

### Design Principles
* Traditional environment:
    * Surface-level security
    * Sporadic logging (some devices don't support it)
    * Dependent on someone to respond to events
    * Managing physical security
    * Lots of manual processes dependent on people

* In the cloud:
    * Implement strong identity foundation
    * Enable traceability
    * Apply security at all layers
    * Automate security best practices
    * Protect data in transit and rest
    * Prepare for security events

### Key Services
* Most key: AWS IAM - securely control who can access what
* Identity and Access Management
    * AWS IAM
    * AWS Organizations - control what resources accounts in your organization can access
    * MFA - extra layer of protection for logins 
    * Temporary security creds (ex. STS)
* Detective Controls
    * CloudTrail - logs API calls
    * Config - define compliance and configuration standards
    * CloudWatch - monitoring
* Infra Protection
    * VPC - isolated resources in the cloud
    * Inspector - vulnerability assessment
    * Shield - DDoS protection
    * WAF - protect from OWASP top 10
* Data Protection
    * Amazon Macie - discovers, classifies, and protects sensitive data
    * KMS - cloud keychain for encryption keys
    * Amazon S3 - secure and resilient storage
    * EBS - resilient storage for your instances (encryption at rest possible)
    * ELB - offload TLS (in-trasit encryption)
    * RDS - includes encryption at rest capability
* Incident Response
    * AWS IAM - used to grant incident response teams appropriate permissions
    * Cloudformation - trusted environment for conducting investigations

### Best Practices
* IAM
    * Key and credential management
    * Enable MFA
    * Create users
    * Protect creds/access keys 
    * Do not hard code your access keys... EVER
* Cognito
    * Use identity broker to log users in via Google, Facebook, and/or Amazon creds
    * Unique users vs. devices
        * Automatically recognize unique users across devices and platforms
    * Securely access any AWS service from mobile devices
* Detective Controls
    * Lifecycle controls to establish baselines
    * Internal auditing
    * Automated alerts/responses
        * For example, setting up a CloudWatch Event to trigger Lambda code execution when a user performs a certain action
* Infrastructure Protection
    * Enforce boundary protection
        * Security groups
        * Dividing layers of the stack into separate subnets (logical isolation)
        * Bastions hosts for access
        * Host-based controls (ex. iptables, anti-malware)
    * Monitoring points of ingress/egress
    * Monitoring and logging
    * Monitoring and alerting
    * AWS service-specific access controls (ex. bucket policies/ACLs)
* Data Protection
    * Encrypt data and key management (TLS/KMS)
        * Encrypt data in transit and at rest
        * Restrict and audit access to encryption keys
    * Detailed logging
    * Versioning
    * Storage resiliency
* Incident Response
    * Ensure the IR team can easily gain access
    * Have the right tools pre-deployed for response (ex. use CloudFormation to create a "clean room")
    * Conduct game days regularly