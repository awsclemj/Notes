- [Infrastructure Security](#infrastructure-security)
  - [Exam tips](#exam-tips)
  - [Shared Responsibility Model](#shared-responsibility-model)
  - [Access Restrictions](#access-restrictions)
    - [S3](#s3)
      - [S3 Pre-signed URLS](#s3-pre-signed-urls)
      - [S3 Cross-region Replication](#s3-cross-region-replication)
    - [Cloudfront](#cloudfront)
      - [Signed URLs and Cookies](#signed-urls-and-cookies)
      - [Geo Restrictions](#geo-restrictions)
    - [Other Geo Restrictions](#other-geo-restrictions)
      - [WAF](#waf)
      - [Route 53 Geolocation Routing](#route-53-geolocation-routing)
  - [DDoS Resiliency](#ddos-resiliency)
    - [Attack Types with Mitigation](#attack-types-with-mitigation)
  - [Network Address Translation (NAT)](#network-address-translation-nat)
  - [Endpoints](#endpoints)
    - [Gateway Endpoints](#gateway-endpoints)
    - [Interface Endpoints](#interface-endpoints)
  - [Systems Manager](#systems-manager)
    - [State Manager](#state-manager)
    - [Actions](#actions)
    - [Other SSM Services](#other-ssm-services)

# Infrastructure Security
Notes taken from internal bootcamp

## Exam tips
* Subdomains for exam:
  * Edge security
  * Design/implement secure network infra (endpoints, NAT, VPC, peering, SG/NACL)
  * Troubleshoot secure network infrastructure
  * Design/implement host-based security (SSM, proxies, IDS/IPS)
* Good to have hands-on experience in this domain

## Shared Responsibility Model
![AWS Shared Responsibility Model](https://d1.awsstatic.com/security-center/Shared_Responsibility_Model_V2.59d1eccec334b366627e9295b304202faf7b899b.jpg)

## Access Restrictions

### S3

#### S3 Pre-signed URLS
* `aws s3 presign <object name>`
* User who accesses the resource will have permissions of the entity that generates the URL
* Default expiry time is 1 hour; can be customized
* You can give a subset of permissions to the accessing user

#### S3 Cross-region Replication
* Not retroactive
* Unidirectional
* Replicates unencrypted and SSE-S3 objects by default
* SSE-C is not supported. SSE-KMS is but requires extra configuration
* By default, ownership and ACLs are maintained. These can be modified
* Storage class is maintained 
* Lifecycle events are not replicated
* Objects are not replicated if the bucket owner does not have permissions

### Cloudfront 
* Supports SNI (server name identifier)
  * Free, but browsers must support it
* Dedicated IP SSL is supported by all browsers but costs extra
* **Viewer Protocol Policy** - allows redirection of HTTP -> HTTPS
* **Origin Protocol Policy** - protocol for CloudFront to communicate with origin
* Integrates with AWS WAF
* Access control via signed URLs/cookies
* Basic white/blacklist geo-resrtiction per distro
* Field-level encryption 
* Lambda @ Edge

#### Signed URLs and Cookies
* Behaviors -> Restrict Viewer Access (creates private distro)
* Web distributions - both signed cookies and URLs
* RTMP distributions - only signed URLs
* Signed URLs - single object access
* Signed cookies - granular access
  * ex: Restrict access only to jpeg files
* Use a canned or custom policy
* Note: if S3 isn't properly locked down, users can bypass CloudFront and directly access your object
  * Hint: use **Origin Access Identities** and lock your bucket down to only allow reads from that OAI

#### Geo Restrictions
* Built-in
  * Per-distribution basis
  * Whitelist or blacklist (either/or) -- based on country
  * Very simple; cannot filter on user agents, cookies, etc. 
* Third-party
  * Anything beyond country - state, region, latitude/longitude, user agent, etc.

### Other Geo Restrictions

#### WAF
* More fine-grained than Cloudfront
  * Block POST requests coming from Australia
  * Block access to http://foo.com/bar.html from Germany
* Block (blacklist)/Allow(whitelist)/Count(monitor)

#### Route 53 Geolocation Routing
* Redirect traffic coming from Europe to ELB in Frankfurt
* Route all the traffic coming from China to a static error page

## DDoS Resiliency
* Architecting for resiliency:
  * VPC:
    * Use Auto Scaling groups
    * Use load balancers
    * Leverage SGs/NACLs to lock down source traffic
  * Use CloudFront/Route 53 -- distributed across edge locations
  * Use WAF to protect from Layer 7 attacks
    * Rate-limiting rules
* Shield standard is included for free (protects from layer3/4 attacks)
* Shield advanced adds additional DDoS mitigation features/dashboards
  * Support from DDoS response team
  * Cost protection during DDoS attacks (don't need to pay for scaling events, etc.)

### Attack Types with Mitigation
* Layer 3/4 such as SYN flood, UDP reflection
  * Mitigated by Shield standard
* HTTP flood
  * WAF rate based rules with baseline threshold
* Slowloris attacks
  * Opening many connection in the target server until max concurrent connections
  * CloudFront has built-in protection by timing out connections

## Network Address Translation (NAT)
* NAT instance
  * instance in private subnet initiates outbound connections to the Internet, but prevent inbound connections
  * Disable source/dest. check
  * Private subnet route table 0.0.0.0/0 -> nat instance
* NAT gateway
  * Highly available
  * Not supported for IPv6 traffic -- use egress-only IGW instead
  * Deploy in public subnet
  * Private subnet route table 0.0.0.0/0 -> nat gateway

## Endpoints

### Gateway Endpoints
* S3 and DynamoDB
* Service is identified by an AWS-managed prefix list
  * Prefix ID: pl-xxxxx
  * Prefix list name: com.amazonaws.region.service
* Endpoint policies restrict access to specific buckets and actions 
* No changes to the application; just ensure DNS resolution is enabled in the VPC

### Interface Endpoints
* Include AWS services and services hosted by customers and partners
* Creates an ENI, so you can apply security groups/reference other security groups
* For HA, put them in each AZ in which your VPC resides
* Multiple DNS names
  * Default DNS - resolves to public service endpoints
  * Regional DNS - will resolve to one of the interface endpoints in your VPC
  * Zonal DNS - resolve interface endpoint for a particular AZ
  * Enable private DNS names
    * Default DNS name will resolve to interface endpoint IP
    * VPC DNS hostnames and DNS support must be set to true 

## Systems Manager
* System management tool that is used to view and control your infrastructure on AWS
* Managed instances:
  * SSM agent must be installed
  * Have necessary permissions
  * EC2 instances, on-prem servers, VMs in other clouds

### State Manager
* Inventory
  * Periodically scan managed instances and collect details such as installed apps, network config, Windows updates, running services, etc.
* Compliance
  * Compare against a baseline
  * Provide compliant or non-compliant state of a resource

### Actions
* Automation
  * Reboot an EC2 instnace
  * Run a script
* Run Command
  * Execute an action in a managed instance
  * Action as a document
* Patch Manager
  * Evaluate the instance against a patch baseline; if non-compliant, remediate
* State Manager
  * Desired state as document 

### Other SSM Services
* Session Manager
  * Secure remote login and interactive sessions
* Parameter Store
  * Store confidential info such as users, passwords, licenses