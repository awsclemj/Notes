## Security and Compliance

* **Compliance Frameworks**
	* PCI-DSS
		* Payment Card Industry Data Security Standard
		* Widely accepted set of policies and procedures intended to optimize the security of credit, debit, and cash card transactions
		* Protects cardholders against the misused of personal data
		* Requirements:
			* 1. Install and maintain firewall config to protect cardholder data
			* 2. Do not use vendor-supplied defaults for system passwords or other security parameters
			* 3. Protect stored cardholder data (encryption at rest)
			* 4. Encrypt transmission of cardholder data across open, public networks (SSL/TLS)
			* 5. Protect all systems against malware and regularly update anti-virus software or programs
			* 6. Develop and maintain secure systems and applications
			* 7. Restrict access to cardholder data by business need to know
			* 8. Identify and authenticate access to system components
			* 9. Restrict physical access to cardholder data
			* 10. Track and monitor all access to network resources and cardholder data
			* 11. Regularly test security systems and processes
			* 12. Maintain a policy that addresses InfoSec for all personnel
	* ISO
		* ISO/IEC 27001:2005 specifies requirements for establishing, implementing, operating, monitoring, reviewing, maintaining, and improving documented Information Security Management Systems.
	* FedRAMP
		* Federal Risk and Authorization Management Program
		* Government-wide program that provides standardized approach to security assessment, authorization, and continuous monitoring for cloud products/services
	* HIPAA
		* Health Insurance Portability and Accountability Act of 1996
		* Primary goal is to make it easier for people to keep healthcare-related information confidential and secure
		* Helps healthcare industry control administrative costs
		* Not all AWS services are HIPAA compliant
	* NIST
		* Framework for Improving Critical Infrastructure Cybersecurity
		* Set of industry standards and best practices to help organizations manage cybersecurity risks
	* SAS70
		* Statement of Auditing Standards No. 70
	* SOC1
		* Service Organization Controls - accounting standards
	* FISMA
		* Federal Information Security Modernization Act
	* FIPS 140-2
		* US Government computer security standard used to approve crypto modules
		* Rated level 1-4, with 4 being the highest security
		* CloudHSM meets level 3 standard

* **DDoS**
	* Whitepaper: https://d1.awsstatic.com/whitepapers/Security/DDoS_White_Paper.pdf
	* What is a DDoS?
		* Distributed Denial of Service (DDoS) is an attach that attempts to make your website or application unavailable to end users
		* Can be achieved by multiple mechanisms
			* Large packet floods
			* Reflection and amplification techniques
			* Botnets
	* Amplification/Reflection Attacks
		* Includes things like NTP, SSDP, DNS, Chargen, SNMP attacks
		* An attacker may send a third party server a request using a spoofed IP address
			* Server responds to the request with a greater payload than the initial request (usually 28-54 times larger)
			* e.g. spoofed packet is 64 bytes, server response is 3,456 bytes
			* When coordinated, attackers can send multiple large (and legitimate) packets from server to target
	* Application Attacks (L7)
		* GET request floods
		* Slowloris
			* Attacker sends many connection requests to a server at once
			* Does this in a way that keeps these connections open as long as possible
				* Sends HTTP headers, partial requests
			* Goal is to use up the maximum concurrent connection limit
	* How to Mitigate?
		* Minimize attack surface area
		* Be ready to scale to absorb attack
		* Safeguard exposed resources
		* Learn normal behavior
		* Create a plan for attacks
	* AWS Shield
		* Free service that protects AWS customers on ELB, CloudFront, and R53
		* Protects against SYN/UDP floods, reflection attacks, and other layer3/4 attacks
		* Advanced features 
			* Enhanced protection for your applications running on ELB, CloudFront, and R53 against larger and more sophisticated attacks. $3000/month
			* Always on flow-based monitoring of network traffic
			* Application monitoring to provide near real-time notification of DDoS attacks
			* DDoS Response Team (DRT) 24x7 to manage and mitigate application layer attacks
			* Protects AWS bill agains higher fees due to ELB, CloudFront, and R53 usage spikes during attack

* **IAM Policies**
	* Exam Tips
		* You can create custom policies using the visual editor or using JSON
		* You can attach roles to EC2 instances at any time using the CLI or AWS console
		* Once attached to an instance, role takes effect immediately
		* When changes are made to a policy, changes take effect immediately


* **IAM MFA and Reporting**
	* Exam tips
		* You can enable MFA using the command line and by using AWS Console
		* MFA can be enabled for the root account and user accounts
		* You can enforce the use of MFA with the CLI by using the Security Token Service (STS)
		* You can report on who is using MFA on a per-user basis by generating a credential report

* **Simple Token Service (STS)**
	* What is it?
		* Grants users temporary access to AWS resources
		* Three sources:
			* Federation (typically AD)
				* Uses SAML
				* Grants temporary access based off of the user's AD creds
				* SSO allows users to log in to the AWS Console without IAM creds
			* Federation with Mobile Apps
				* Use Facebook/Amazon/Google or other OpenID providers to log in
			* Cross-account access
				* Lets users from one AWS account access resources in another
	* Key terms
		* Federation: combining/joining list of users in one domain (such as IAM) with a list of users in another domain (such as AD, Facebook, etc.)
		* Identity Broker: a service that allows you to take an identity from point A and join it (federate it) to point B
		* Identity Store: Services like AD, Facebook, Google, etc.
		* Identities: a user of a service like Facebook, etc.
	* How STS Federates an Identity - Scenario with data reporting application
		* 1. Employee enters their username/password
		* 2. Application calls an Identity Broker. The broker captures the username/password
		* 3. Broker uses the organization's LDAP directory to validate the employee's identity
		* 4. Broker calls the GetFederationToken function using IAM creds
			* Call must include an IAM policy and duration (1 to 36 hours)
			* Also must specify a policy that grants permissions to the temporary creds
		* 5. STS confirms the policy of the IAM user making the call to GetFederationToken gives permission to create new tokens and returns for values to the application
			* An access key
			* A secret access key
			* A token
			* The duration (token's lifetime)
		* 6. Broker returns the temporary creds to the reporting application
		* 7. The data storage application uses temporary creds to make requests to S3
		* 8. S3 uses IAM to verify that the creds allow requested action
		* 9. IAM gives S3 the go-ahead to perform requested operation

* **Logging**
	* Basics
		* CloudTrail
		* Config
		* CloudWatch
		* VPC Flow Logs
	* Controlling Access
		* Recent unauthorized access:
			* IAM users, groups, roles, and policies
			* S3 bucket policies
			* MFA
		* Ensure role-based access:
			* IAM users, groups, roles, policies
			* S3 bucket policies
	* Alerts
		* Alerts when logs are created or fail:
			* CloudTrail notifications
			* AWS Config rules
		* Alerts should be specific, but don't divulge detail:
			* CloudTrail SNS notification only point to log file location
	* Managing Changes
		* Log any changes to system components:
			* AWS Config rules
			* CloudTrail
		* Controls exist to prevent modifications to logs:
			* IAM and S3 controls/policies
			* CloudTrail log file validation
			* CloudTrail log file encryption

* **Hypervisors**
	* What is a hypervisor?
		* A hypervisor or virtual machine monitor (VMM) is computer software or hardware that creates and runs virtual machines
		* A computer on which a hypervisor runs one or more virtual machines is call a host machine, and each virt is called a guest machine
	* AWS Hypervisor
		* Xen
			* Can have a guest OS running either as Paravirtualization (PV) or Hardware Virtual Machine (HVM
			* HVM guests are fully virtualized; these guests are not aware they are sharing processing with other VMS
			* PV is a lighter form of virtualization and used to be quicker
			* AWS recommends running HVM over PV
	* Isolation
		* Physical interface -> Firewall (security groups) -> virtual interfaces -> Hypervisor -> customer guest images
	* Hypervisor Access
		* Admins with business need to access management plane are required to use MFA to gain access to purpose-build admin hosts
		* These admin hosts are hardened to protect the management plane of the cloud
		* Access is logged and audited
		* Employee privilege can be revoked
	* Guest (EC2) Access
		* Completely controlled by the customer
		* Full root access and admin control over accounts, services, applications
		* AWS has no access to the guest instances
	* Memory scrubbing
		* EBS automatically resets every block of storage used by the customer so that data isn't leaked
		* Memory allocated to guests is zeroed by the hypervisor when it is unallocated from a guest 

* **Dedicated instances vs. dedicated hosts**
	* What are dedicated instances?
		* EC2 instances that run on hardware that's dedicated to a single customer
		* Physically isolated at the host hardware level from instances that belong to other accounts
		* May share hardware with other instances from the same AWS account that are not Dedicated instances
	* What are dedicated hosts?
		* Physical servers dedicated for your use
		* Important distinction: dedicated host gives you additional visibility and control over how instances are placed on a physical server
		* You can deploy your instances to the same physical server over time
		* Can use server-bound software licenses to address compliance/regulatory requirements

* **SSM EC2 Run Command**
	* What is it?
		* Manage large number of EC2 instances and on-prem systems
		* Can be used to automate common admin tasks and ad hoc config changes
			* Installing applications
			* Applying latest patches
			* Joining instances to AD
	* Exam tips
		* Commands can be applied to a group of systems based on tags or manual selection
		* SSM agent needs to be installed on your managed instances
		* Commands and parameters are defined in a Systems Manager document
		* Can use the service on EC2 and on-prem machines

* **SSM Parameter Store**
	* Scenario
		* You work for a bank and need to store confidential information (users, passwords, etc.) that can be passed to EC2 while retaining confidentiality
	* How to do this?
		* Parameter Store allows you to store secrets and allows them to be accessed via several key AWS services
			* Lambda, EC2 Run Command, CloudFormation, etc. 
		* You can store these values as plain text or encrypt them using KMS

* **S3 Pre-signed URLs**
	* Exam tips
		* You can access objects using pre-signed URLs
		* Typically done via SDK but can be done on CLI as well
		* Exist for a certain length of time in seconds; default is 1 hour
		* Can change this using "--expires-in" followed by number of seconds

* **AWS Config with S3**
	* Be aware of the following config rules for S3:
		* No public read access
		* No public write access

* **Inspector vs. Trusted Advisor**
	* Inspector
		* Automated security assessment service -- helps improve security and compliance of applications deployed on AWS
		* Assesses applications for vulnerabilities or deviations from best practice
		* Will publish a detailed list of findings prioritized by level of security 
		* How does it work?
			* Create "Assessment target"
			* Install agents of EC2 instances
			* Create "Assessment template"
			* Perform "Assessment run"
			* Review "Findings" against "Rules
	* Trusted Advisor
		* A resource to help you reduce cost, increase performance, and improve security by optimizing your AWS environment
		* Advises on cost optimization, performance, security, and fault tolerance
		* Two tiers:
			* Core checks and recommendations
			* Full TA - business or enterprise customers only

* **Shared Responsibility Model**
	* AWS's responsibilities
		* Global infra
		* Hardware, software, networking, and facilities
		* "Managed services"
		* Regions, AZs, edge locations
	* Customer's responsibility
		* Customer data
		* Platform, applications, IAM
		* OS, network, and firewall config
		* Client-side data encryption and identity authentication
		* SSE (file system or data)
		* Network traffic protection
	* Model changes for different types:
		* Infrastructure
			* Compute services (EC2, EBS, AutoScaling, and VPC)
			* You can architect and build cloud infra with these services
			* You control OS and configure an identity management system that provides access to the user layer of the virtualization stack 
		* Container
			* Services the run on separate EC2 or other infra, you don't manage the OS or platform layer (RDS, EMR, Elastic Beanstalk
			* AWS provides managed services for these application containers
			* You're responsible for setting up/managing network controls (such as firewall rules)
			* You are responsible for managing platform-level identity and access management separately from IAM
		* Abstracted
			* High-level storage, DB, and messaging services (S3, Glacier, DynamoDB, SQS, SES)
			* The services abstract the platform or management layer on which you can build and operate cloud applications
			* You can access the endpoints of these services using AWS APIs and AWS manages the underlying service components or OS which they reside

* **Other Security Aspects**
	* Security Groups
		* Stateful -- meaning if you open up an inbound port on your SG, egress will automatically be allowed
		* You should be able to read these rules 
	* Who is provisioning this stuff in my account?
		* CloudTrail will log API calls
		* You can either go through and read through the API calls manually or use Athena to analyze the CloudTrail logs for you
	* AWS Artifact
		* Provides on-demand downloads of security and compliance documents
		* You can submit the security and compliance documents (AKA audit artifacts) to your auditors or regulators to demonstrate the security and compliance of AWS infra and services in use
	* Know KMS and CloudHSM (see earlier in this doc)
	* When can you encrypt things?
		* Instant
			* Only S3
		* Encryption with Migration
			* Dynamo
			* RDS
			* S3
			* EBS