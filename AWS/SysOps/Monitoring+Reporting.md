## Monitoring and Reporting

* **Cloudwatch vs. Cloudtrail**
	* CW monitors performance 
		* Network perf, CPU perf, etc.
	* CTrail monitors API calls in the AWS platform (auditing)
		* Who provisioned/deleted what?

* **Cloudwatch - EC2**
	* Standard monitoring - 5 minute intervals
	* Detailed monitoring - 1 minute intervals
		* Costs extra
	* CW is for performance monitoring
		* CPU util, network throughput, disk usage by default
		* Can use custom metrics to report other things such as memory util. 
	* What can we do with CW?
		* Dashboards - create dashboards for your important resources for monitoring
		* Alarms - will notify you when your thresholds are hit
		* Events - helps respond to state changes for you AWS resources
		* Logs - allows you to aggregate, monitor, and store logs (even from outside resources)

* **Cloudwatch - EBS**
	* General EBS
		* Storage types:
			* General purpose (SSD)
			* Provisioned IOPS (SSD)
			* Throughput optimized hard disk
			* Cold hard disk
			* Magnetic (legacy)
		* Performance:
			* Key IOPS metrics
				* You can burst up to 3000 IOPS
				* If you need more than 3000 but less than 10,000 increase the volume size
				* Anything over 10,000 IOPS move to PIOPS

* **Monitoring ELB**
	* ELB basics
		* Application Load Balancer
			* Application (L7) aware
		* Network Load Balancer
			* Very high network throughput prod environments
		* Classic Load balancer
			* Either L4 or L7 aware, but getting deprecated
	* 4 ways to monitor ELB
		* CW metrics
		* Access logs
		* Request tracing (ALB only)
		* CloudTrail logs
	* CW metrics
		* ELB published data points to CW for your load balancers and targets
		* Can retrieve stats about those data points as an ordered set of time-series data (AKA metrics)
		* A metric is like a variable to monitor, and the data points are the values of that variable over time
		* Several metrics are automatically created and populated as soon as you spin up an ELB
	* Access Logs
		* Capture detailed information about requests sent to your load balancer
		* Contains information such as the time request was received, client's IP address, latencies, request paths, and server responses.
		* Can be used to analyze traffic patterns and troubleshoot issues
		* Optional; disabled by default on CLB
		* Enabling them requires you to specify an S3 bucket. Logs are stored as compressed files
		* Can still store data after EC2 instances have been deleted.
			* Ex: fleet of EC2 instances behind ASG. For some reason, application experiences many 5XX errors which is only reported days after the event
			* If the EC2 instance logs aren't stored anywhere persistent, it is still possible to trace the 5XX errors using Access Logs
	* Request Tracing
		* Can be used to track HTTP requests from clients to targets or other services. 
		* When LB receives a request from a client, it add or updated the X-Amzn-Trace-Id header before sending to request to the target
		* Any services or applications between the LB and target can also add or updated this header
		* Available for ALB only
	* CloudTrail
		* Can be used to capture detailed information about calls made to ELB API and store them as files in S3
		* Can use these logs to determine:
			* Which calls were made
			* Source IP
			* Who made the call
			* When the call was made

* **Monitoring Elasticache**
	* Elasticache basics
		* Consists of two engines:
			* Memcached
			* Redis
	* What do we monitor in caching engines?
		* CPU Util.
		* Swap usage
		* Evictions 
		* Concurrent connections
	* CPU util.
		* Memcached
			* Multi-threaded
			* Can handle loads of up to 90%. Add more nodes to the cluster if this is exceeded
		* Redis
			* Not multi-threaded. To determine when to scale, take 90 and divide by number of cores
			* Ex: Suppose you are using cache.m1.xlarge node (4 cores)
				* Threshold for CPU Util. would be (90/4), or 22.5%
	* Swap usage
		* Amount of the swap file that is being used
		* Swap file (paging file) is the amount of disk storage space that is reserved on disk if your server runs out of memory
		* Typically size of swap file = size of memory
		* Memcached
			* Should be around 0 most of the time, should not exceed 50 MB
			* If exceeding 50 MB, increase memcached_connections_overhead parameter
				* Defines amount of memory to be reserved for memcached connections and other misc. overhead
		* Redis
			* No SwapUsage metric; instead use reserved-memory
	* Evictions
		* Like tenants in an apartment; there are a number of empty apartments that slowly fills up
		* Eviction occurs when a new item is added and an old item must be removed due to lack of free space
		* Memcached
			* No recommended setting; choose threshold based on application
				* Scale up (increase memory of existing nodes)
				* Scale out (add more nodes)
			* Redis
				* Choose threshold based off application
					* Only scale out (add read replicas)
		* Concurrent connections
			* Memcached and Redis
				* No recommended threshold
				* If there is a large and sustained spike in number, this can either mean a traffic spike or your application is not releasing connections as it should be
				* Remember to set an alarm on this value

* **CloudWatch Dashboards**
	* Custom dashboards can help visualize several metrics in a single pane of glass
	* You can use this to show several aspects about your EC2 instances, ELB, prod environment, etc. 
	* Adding widgets to a dashboard is region-specific, however you can view all of your custom dashboards in any region
		* To add widgets from multiple regions to a single dashboard, ensure to change your region first

* **AWS Organizations**
	* Main features
		* Centrally manage policies across multiple accounts
		* Control access to AWS services
		* Automate account creation and management
		* Consolidate billing across multiple accounts
	* Central management
		* Allows you to manage multiple accounts at ones
		* Create groups of accounts then attach policies to groups
	* Control access
		* Create Service Control Policies (SCPs) that centrally control AWS service use across accounts 
		* Allow/deny individual services
			* Even if IAM in account allows it, SCP overrides
	* Automate account creation
		* APIs allow you to create new accounts programmatically and add these new accounts to a group
		* Policies attached to the group are automatically added to the new account
	* Consolidated billing
		* Single payment method for all accounts in organization (consolidated billing)
		* You can see a combined view of charges incurred by all accounts
		* Can take advantage of some pricing benefits from aggregated usage
			* Volume discounts from EC2 and S3

* **Tagging and Resource Groups**
	* What are tags?
		* Key-value pairs attached to AWS resources
		* Metadata
		* Tags can be inherited
			* Autoscaling, CloudFormation, Elastic Beanstalk, for ex.
	* What are resource groups?
		* Make it easy to group you resources using tags assigned to them
		* Resource groups contain info such as:
			* Region
			* Name
			* Health checks
		* Specific info:
			* For EC2 - public and private IP
			* For ELB - port configurations
			* For RDS - Database engine
	* Classic resource groups
		* Global resources
	* Systems Manager
		* Per region basis
		* Can run commands against these resources

* **EC2 Pricing**
	* On-demand
		* Flexibility of EC2 without upfront payment or long-term commitment
		* Applications with spiky, short term, or unpredictable workloads that can’t be interrupted
	* Reserved
		* Applications with steady, predictable usage
		* Apps that require reserved capacity
		* Users able to make upfront payments to reduce compute costs
			* Standard - up to 75% off on-demand
			* Convertible - up to 54% off on-demand
				* Capability to change attributes of RI as long as it results in a new RI of equal or greater value
			* Scheduled - available to launch within reserved time windows
				* Match a capacity reservation to predictable recurring schedule
	* Spot
		* Applications that have flexible start and end times
		* Applications only feasible for very low compute prices
		* Users with urgent computing needs for large amounts of additional capacity
	* Dedicated hosts
		* Useful for regulatory requirements
		* No multi-tenancy
		* Great for licensing that doesn’t support multi-tenancy
		* Can be purchased on-demand
		* Can be purchased as reservation for up to 70% off on-demand price

* **AWS Config**
	* Fully managed service that provides you with and AWS resource inventory, config history, and config change notifications to enable security and governance
	* Enables
		* Compliance auditing
		* Security analysis
		* Resource tracking
	* Provides
		* Configuration snapshots and logs config changes of resources
		* Automated compliance checking
		* Example: will notify you if port 22 is open on a server
	* Components:
		* Config dashboard
		* Config rules
			* Managed
			* Custom
		* Resources
		* Settings
	* How does it work?
		* Similarly to CloudTrail
		* Example: Make a change to security group or S3 bucket ACL
			* This fires off an event that goes to AWS Config
			* AWS Config logs this to S3
			* You can trigger a Lambda function depending on the event, or schedule a Lambda function to check Config for compliance
			* Lambda will say whether or not a rule is being broken
				* If a rule is being broken, AWS Config triggers an SNS topic that informs subscribed individuals of compliance issue
	* Terminology
		* Configuration Items
			* Point-in-time attributes of a resource
		* Configuration snapshots
			* Collection of config items
			* Can be configured to take a snapshot every 1 hour, 3 hours, 6 hours, 12 hours, or 24 hours
		* Configuration stream
			* Stream of changed config items
		* Configuration history
			* Collection of config items for a resource over time
		* Configuration recorder
			* The configuration of Config that records and stores config items
		* Recorder setup:
			* Logs config for account in region
			* Stores in S3
			* Notifies SNS
	* What we can see:
		* Resource type
		* Resource ID
		* Compliance
		* Timeline
			* Configuration details
			* Relationships
			* Changes 
			* CloudTrail events
	* Compliance checks:
		* Trigger
			* Periodic
			* Configuration snapshot delivery (filterable)
		* Managed rules
			* About 40 (may increase)
			* Basic, but fundamental
	* Permissions needed:
		* IAM role
			* Read-only permissions to the recorded resources
			* Write access to an S3 logging bucket
			* Publish access to SNS
	* Restricting Access:
		* Users need to be authenticated with AWS and have the appropriate permissions set via IAM policies
		* Only admins needing to set up and manage Config require full access
		* Provide read-only permissions for Config day-to-day use
	* Monitoring with Config
		* Use CloudTrail with Config to provide deeper insight into resources
		* Use CloudTrail to monitor access to Config, such as someone stopping the Config Recorder

* **Health Dashboards**
	* Service Health Dashboard
		* Shows health of each AWS service as a whole per region
		* status.aws.amazon.com
	* Personal Health Dashboard
		* Provides alters, remediation guidance, etc., when AWS is experiencing event that may impact you