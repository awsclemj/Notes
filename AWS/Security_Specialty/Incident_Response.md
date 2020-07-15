- [Incident Response](#incident-response)
  - [Things to Know](#things-to-know)
  - [Definitions](#definitions)
  - [Threat Detection](#threat-detection)
    - [Log data inputs](#log-data-inputs)
    - [GuardDuty and Security Hub](#guardduty-and-security-hub)
    - [Triggers](#triggers)
    - [Automation](#automation)
  - [Control Tower/Landing Zone](#control-towerlanding-zone)
  - [Remediating threats](#remediating-threats)

# Incident Response
Notes taken from internal security bootcamp

## Things to Know
* **Advanced persistent actor (APT)** - an entity that is actively in an AWS account doing nefarious things
* IR teams are usually pretty small (5-10 individuals). They are defensive teams that work against threats
* The only way to be good at IR is to ask questions, be wrong, and then eventually be right a lot
  
## Definitions
* Runbooks
  * Tactical review of a situation
  * Descriptions of a situation that may occur
  * Steps to correct or enact a desired outcome
  * Contact list for situations
* Playbooks
  * Strategic review or overview of situational responses
  * Planning for future
  * Generally non-technical
  * C-level info
  * Potentially a RACI - response, accountability, consulted, informed
* Internal threat
  * Anyone inside or included in your organization
  * Intentional or unintentional
  * Malicious or accidental
  * Will get you 9 times out of 10
* External threat
  * Anyone outside of the organization; nation states, bad actors, other external forces
  * Intentional or unintentional
  * Malicious or accidental
* Security Incident Response Simulations (SIRS)
  * Practice security failure that allows you to evaluate how the company, organization, etc., will respond to specific events or scenarios
  * Two types:
    * Tabletop
      * No hands on keyboards
      * Role-play; follow incidents from start to finish
      * Suitable to involve non-technical teams
    * Technical
      * Execute technical action in an environment
      * Observe response of organizations
        * Time to detect
        * Time to respond
        * Effectiveness of response
        * Lessons learned
      * Ex: Simian Army

## Threat Detection
* What is traditional threat detection so hard?
  * Large datasets
  * Signal to noise ratio
    * GuardDuty and Config can send out a lot of notifications when they initially begin analysis
    * Generally these baselines need to be massaged for the customer based on their needs
  * Cost
* Humans and data don't mix
  * Humans are inconsistent and not repeatable
  * We can't evaluate human interactions with data

### Log data inputs
* CloudTrail - track user activity and API usage
  * Search for attack vectors based on user logins, lateral movements, and data
* VPC Flow Logs - IP traffic to/from ENIs
  * Detect attacks based on network traffic at the interface level
* CloudWatch Logs - Monitor apps using log data. Store and access log files
  * You can deliver log feeds to Lambda. You can also deliver to S3 and analyze with Athena
* DNS Logs - Log of DNS queries in a VPC when using VPC resolver

### GuardDuty and Security Hub
* GuardDuty continuously analyses CT events, VPC Flow Logs, and DNS query logs. It intelligently detects threats and creates alerts
  * Some patterns can cause "false positives" but may actually be alerting you to inappropriate uses of tokens/credentials in your organization
* Security Hub allows you to assess your high-priority security alerts in a centralized dashboard
  * Visibility to answer:
    * What data do I have in the cloud?
    * Where is it located?
    * Where does my sensitive data exist?
      * Why is it sensitive?
      * What PII/PHI is potentially exposed?
    * How is data being shared/stored?
    * How is my data being accessed?
    * How do I build workflow remediation for my security/compliance needs?

### Triggers
* Config Rules
  * Continuous recording of AWS resources vs. compliance baselines
  * When changes occur, the change is evaluated against the baseline
  * Changes can be rolled back if not compliant
* CloudWatch Events
  * Customize event triggers and react to these events with custom code via Lambda
  * Build a detective control for all of your preventative controls (in case, for example, there is a bug in your preventative control)

### Automation
* We have the opportunity to automate away some remediation tasks
* Lambda
  * Ex: Capture info about IP traffic going to and from ENIs in your VPC
* Systems Manager
  * Ex: Automate patching and proactively mitigate threats at the instance level
* Example playbook:
  * Adversary does something -> CloudWatch Events -> Step Functions -> Lambda responder

## Control Tower/Landing Zone
* Proactively set up an account structure with security controls and governance best practices
* Organizations: account management
* Log archive: security logs
* Security: security tools, AWS Config rules
* Shared services: directory, limit monitoring
* Network: DX/VPN
* Dev sandbox: Experimentation
* Dev: Development account
* Pre-prod: staging
* Prod: production
* Team shared services: data lakes, other shared services

## Remediating threats
* Workflow
  * Detection - VPC Flow Logs, WAF, Shield, Macie, GuardDuty
  * Alerting - CloudWatch/Config -> Step Functions -> Lambda -> Collaboration tools/AWS APIs
  * Remediation/Countermeasures - take action on resource based on Step Function step. Ex: take backup EBS volume
  * Forensics - analyze findings
* On EC2 instances:
  * Asynchronously execute commands via step functions
  * **DO NOT** SSH/RDP to instances - use SSM Run Commands
  * Commands/output logged