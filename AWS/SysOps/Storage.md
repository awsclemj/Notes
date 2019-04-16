## Storage

* **S3**
	* What is S3?
		* Secure, durable, highly available object storage
		* For files, not operating systems/databases
		* Data stored across multiple devices and facilities
	* Basics
		* Object-based (files)
		* Can be from 0 B to 5 TB
		* Unlimited storage
		* Files stored in buckets (similar to a folder)
		* Universal namespace (buckets must be unique globally)
		* ex: https://s3-eu-west-1.amazonaws.com/acloudguru
		* HTTP 200 code if the upload is successful
	* Data Consistency Model
		* Read after write consistency of PUTS of new objects (can read right away)
		* Eventual consistency for overwrite PUTS and DELETES (can take time to propagate)
	* Simple key-value store
		* Objects consist of the following
			* Key (name of object)
			* Value (the data)
			* Version ID (important for versioning)
			* Metadata (Data about the data you're storing)
			* Subresources (bucket-specific config)
				* Bucket Policies, ACLs
				* Cross Origin Resources Sharing (CORS)
				* Transfer Acceleration
	* More Basics
		* Built for 99.99% availability
		* Amazon guarantees 99.9% availability 
		* Amazon guarantees 99.999999999% (eleven nines) durability
			* This means that you should only expect an object to be lost once every 10,000 years
		* Tiered storage
		* Lifecycle management
		* Versioning
		* Encryption
	* Storage Tiers/Classes
		* S3 - 99.99% availability, eleven nines durability, store redundantly across multiple devices and multiple facilities
			* Can sustain the loss of 2 facilities concurrently
		* S3 - IA (Infrequently Accessed) - For data that is access less frequently, but requires rapid access when needed
			* Same eleven nines durability, 99.9% availability
			* Lower fee than S3, but you get charged for retrieval
		* S3 - OneZone IA - Same as IA but data is stored in a single AZ only
			* Still eleven nines durability, but only 99.5% availability
			* Cost is 20% less than IA
		* Reduced Redundancy Storage (RRS) - provides 99.99% durability and availability of objects in a given year
			* Use for data that can be recreated if lost
			* Is being deprecated
		* Glacier
			* Very cheap, but used for archival only
			* Optimized for infrequently accessed data; takes about 3-5 hours for retrieval
	* S3 Intelligent Tiering
		* For unknown or unpredictable access patterns
		* 2 tiers
			* Frequent and infrequent access
		* Automatically moves data to most cost-effective tier based on how frequently you access each object
		* Same eleven nines durability, 99.9% availability (same as IA)
		* No fees for accessing data but monthly fee for monitoring/automation ($0.0025 per 1,000 objects)
	* Charges
		* Storage per GB
		* Requests (GET, PUT, COPY, etc)
		* Storage management pricing
			* Inventory, analytics, object tags
		* Data management
			* Data transferred out of S3
		* Transfer Acceleration

* **S3 Lifecycle Policies**
	* Basics
		* Manage your objects so they are stored with the most cost effective S3 option throughout their lifecycle
		* Configure lifecycle rules to tell S3 to transition objects to less expensive storage classes, archive them, or delete them
	* Example policies
		* Transition objects to IA storage after 90 days (log files)
		* Archive objects to Glacier after 1 year
		* Expire objects 1 year after creating them
		* e.g. buckets with server access logging

* **MFA Delete and Versioning**
	* Basics - Versioning
		* Versioning enables you to revert to older versions of S3 objects
		* Multiple versions are stored in same bucket
		* Prevents from accidental/malicious deletion
		* DELETE actions don't delete the object version, just places a delete marker
		* To permanently delete, provide the version ID of the object in the delete request
	* Basics - MFA
		* Provides additional layer of protection to versioning
		* MFA Delete enforces 2 things:
			* Need a valid code from MFA device to permanently delete an object
			* MFA needed to suspend/reactivate versioning on an S3 bucket

* **S3 Encryption**
	* In transit
		* SSL/TLS
	* At rest
		* Server-side encryption
			* S3 managed keys - SSE-S3
			* AWS Key Management Service Managed Keys - SSE-KMS
				* Additional features (access to "envelope" key, auditing)
			* SSE with Customer-provided keys - SSE-C
		* Client-side encryption
			* Encrypt before uploading to S3
	* Enforcing Encryption
		* Every time a file is uploaded, a PUT request is initiated
		* If a file is to be encrypted at upload time, the x-amz-server-side-encryption parameter will be included in request header
		* Two options:
			* x-amz-server-side-encryption: AES256 (SSE-S3)
			* x-amz-server-side-encryption: aws:kms (SSE-KMS)
		* When parameter is included in PUT request, it tells S3 to encrypt the object at time of upload
		* You can enforce this by using a Bucket Policy which denies PUT requests without x-amz-server-side-encryption in header


* **EC2 instance store vs. EBS**
	* History
		* When EC2 was first launched, all AMIs were instance store
		* This is ephemeral storage; temporary
		* Later, AWS launched EBS, which allows users to have data persistence
	* Root Devices
		* Root device volumes can be EBS or instance store volumes
		* Instance store volumes max out at 10 GB
		* EBS root device volumes can be up to 1 or 2 TB depending on OS
	* Terminating instances
		* EBS root device volumes are terminated by default when EC2 instance is terminated
			* You can stop this by unselecting "Delete on Termination" flag when creating the instance in the console or command line
		* Other EBS volumes attached to the instance will be preserved even after deleting the instance
		* Instance store devices (root or other) will all be deleted by default when the instance is terminated
	* Stopping instances
		* EBS-based instances can be stopped, instance store can only be rebooted or terminated
	* Instance store data
		* Instance store data will be retained if an instance is rebooted. However, it can be lost under the following conditions:
			* Failure of the underlying drive
			* Stopping EBS-backed instance
			* Terminating an instance
	* Upgrading EBS volume types
		* With the exception of normal Magnetic storage types, you can upgrade storage type and size of a volume after creation
			* You can do this on the fly, but it's recommended to do this while the instance is stopped
		* Volumes must be in the same AZ as the instance they're attached to
		* Volumes can be "moved" between AZs by creating a snapshot of one volume and creating a new volume from that snapshot
			* Snapshots are incremental and stored on S3
			* You can copy snapshots to other regions
			* Snapshots of encrypted volumes are encrypted automatically
		* You can create an image from snapshots which are launchable as custom AMIs
			* These can also be copied to other regions

* **Encryption and Downtime**
	* Enabling Encryption
		* For most AWS services, encryption must be enabled at creation
		* For example, if you want to encrypt an EFS file systems that already exists, you must create an encrypted EFS and migrate your data
		* For encrypting an existing RDS, you must create a new RDS and migrate your data
		* EBS volumes are similar
			* Encryption must be selected at time of creation
			* You cannot encrypt an unencrypted volume or vice versa
			* You can migrate data between encrypted and unencrypted volumes (rsync/Robocopy)
			* If you want to encrypt an existing volume, you can create a snapshot, copy the snapshot, and apply encryption
				* Restoring this snapshot will result in an encrypted volume
		* S3 is more flexible
			* S3 buckets and objects can have encryption enabled at any time

* **KMS and CloudHSM**
	* What are these services?
		* Both allow you to generate, store, and manage cryptographic keys used to protect your data in AWS
		* HSMs (Hardware Security Modules) are used to protect the confidentiality of your keys
		* Both offer a high level of security
	* KMS
		* Shared hardware, multitenant managed service
		* Allows you to generate, store, and manage your encryption keys
		* Suitable for applications for which multitenancy is not a problem
			* This can cause regulatory issues depending on industry
		* Encrypt your data stored in AWS, including EBS volumes, S3, RDS, DynamoDB, etc.
		* Uses symmetric keys only
		* Free-tier eligible
	* CloudHSM
		* Dedicated HSM instance; not multitenant (also no free-tier)
		* Allows you to generate, store, and manage your encryption keys
		* HSM is under your exclusive control within your VPC
		* FIPS 140-2 Level 3 compliance
			* US Government standard
			* Includes tamper-evident physical security mechanisms
		* Suitable for applications that have contractual or regulatory requirements for deidicated hardwae managing crypto keys
		* Use cases include:
			* DB encryption
			* DRM
			* PKI
			* Authentication/authorization
			* Document signing
			* Transaction processing
		* Symmetric or asymmetric encryption

* **Snowball and Snowball Edge**
	* Snowball
		* Physical device used for transporting many TB or PB of data in and out of AWS
		* Makes large scale data transfers fast, easy, and secure
		* Tamper-resistant enclosure
		* 256-bit encryption by default
		* Region-specific; not for transporting data from one region to another
	* How does Snowball work?
		* Connect the device to your local network
		* Client automatically encrypts and copies data you select to Snowball
		* When the transfer is complete, you ship the device back to Amazon
	* When to use Snowball
		* When you have many TB or PB of data to upload
		* You don't want to make expensive upgrades to your network for a one-off transfer
		* Frequent backlogs of data
		* Physically isolated environments (military base, area without good Internet)
		* Rule of thumb: if it takes longer than a week to upload your data, use Snowball instead
	* Snowball Edge
		* 100 TB device which features onboard compute power; it can be clustered to act as a single storage and compute pool
		* Desinged to undertake local processing / edge computing as well as data transfer
		* S3-compatible endpoint, supports NFS, and can run Lambda functions as data is copied to the device
		* S3 buckets and Lambda functions must come pre-defined
	* Use cases for Snowball Edge
		* Adds additional capability to run simple computing functions on the device
		* Use it for use cases that require local processing before returning data to AWS
		* e.g. performing data analysis before writing processed data to an S3 bucket on Snowball Edge

* **Storage Gateway**
	* What is it?
		* On-prem software appliance which connects with AWS aloud-based storage
		* Seamless and secure integration between your on-prem environment and AWS
		* Virtual appliance is installed in your DC
			* Supports VMWare ESXi or MS Hyper-V
		* On-prem systems seamlessly integrate with AWS storage (S3)
		* Three types
			* File Gateway - NFS/SMB
			* Volume Gateway (iSCSI)
				* Stored Volumes
				* Cached Volumes
			* Tape Gateway - VTL
	* File Gateway
		* Files stored as objects in S3 buckets
		* Accessed using NFS or SMB mount point
			* This makes this appear to be a network-attached storage backed by S3
		* All the benefits of S3: bucket policies, S3 versioning, lifecycle management, replication, etc.
	* Volume Gateway
		* Cloud-backed storage using iSCSI protocol
		* Two different types:
			* Stored - store all data locally and only backup to AWS
				* Apps get low latency access to entire dataset
				* Need your own storage infrastructure; all data is stored on-prem
			* Cached - use S3 as primary storage and cache frequently accessed data in Storage Gateway
				* Gateway stores all data in S3; caches only frequently accessed data locally
				* Need only enough local storage capacity for frequently accessed data
				* Applications still get low-latency access to frequently used data without investment in storage equipment
	* Tape Gateway
		* Virtual Tape Library (VTL) that provides cost-effective data archiving in the cloud using Glacier
		* Don't need to invest in tape backup infra
		* Integrates with existing tape backup infra - connect to VTL using iSCSI
			* NetBackup
			* Backup Exec
			* Veeam
		* Data stored in Glacier and accessed using VTL

* **Athena**
	* What is it?
		* Interactive query service that enables you to analyse and query data in S3 using SQL
		* Serverless; nothing to provision
		* Pay by query / per GBs called
		* No need to set up complex Extract/Transform/Load (ETL) processes
		* Works directly on data stored in S3
	* Use cases
		* Can be used to query log files stored in S3
			* ELB logs, S3 access logs, etc.
		* Generate business reports on data in S3
		* Analyze AWS cost and usage reports
		* Run queries on click-stream data