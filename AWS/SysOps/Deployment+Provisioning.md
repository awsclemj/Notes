## Deployment and Provisioning

* **EC2 Launch Issues**
	* Common reasons:
		* InstanceLimitExceeded error
			* Reached the limit of the number of instances you can launch in a region
			* AWS sets a default limit on the number of instances you can run on a per-region basis
			* 20 is the default
			* You can request a limit increase
		* InsufficientInstanceCapacity errors
			* AWS doesn't have enough on-demand capacity to serve request
			* Options to resolve this error:
				* Wait a few minutes and try again
				* Request fewer instances
				* Select a different instance type
				* Try purchasing reserved instances instead
				* Submit a new request without specifying an AZ
					* This allows AWS to provision the instance in an AZ without capacity issues

* **EBS Volumes and IOPS**
	* General information
		* Elastic Block Store (EBS) allows you to create storage volumes and attach them to your EC2 instances
			* Can be used to create a file system, run a DB, or run an OS
			* We'll be focusing on SSD-backed storage
		* IOPS - Input/output operations per second
			* Used to benchmark performance of SSD volumes
			* Dependent on the size of volume
				* gp2 - minimum of 100 IOPS, 3 IOPS/GB afterwards up to a maximum of 16,000 IOPS
				* io1 - 50 IOPS/GB, max of 64,000 IOPS
	* 2 different SSD types:
		* gp2 - General purpose - boot volumes
		* io1 - Provisioned IOPS - I/O intensive workloads, NoSQL/Relational DBs, latency-sensitive workloads
	* Hitting IOPS limit of volume
		* What happens if you hit the limit of gp2 volume?
			* You'll start to get I/O requests queued
			* Depending on your application's sensitivity to IOPS/latency, you may start to see your application become slow
		* What can we do about this?
			* 2 approaches:
				* For gp2, you can increase the size of your volume
					* However, if your volume is already 5.2 TB or more, you will have reached the 16,000 IOPS limit for gp2 volumes
				* If you need more than 16,000 IOPS, change to Provisioned IOPS

* **What is a Bastion Host?**
	* A bastion host is located in your public subnet (Internet access)
	* Allows you to connect to EC2 instances using SSH or RDP
	* You can log into this host over the Internet
	* You then use the Bastion host to initiate an SSH/RDP session to your EC2 instances in the private subnet
	* Also called a jump host
	* Safely administer EC2 instances without exposing them to the Internet
	* Best practice: lock down your Bastion as much as possible (known IP ranges only/only ports needed)

* **ELB 101**
	* What is a Load Balancer?
		* Equally balance incoming requests to multiple backend web servers
		* Should allow for a consistent experience for clients, as no single web server should be bogged down with requests
	* Types of Load Balancers
		* Application Load Balancer
			* Operates at L7 (Application layer)
			* Has the ability to inspect packets and make routing decisions based on HTTP headers
			* Content-based routing
				* Ex: requests for acloudguru.com/sales and acloudguru.com/marketing go to different backend web servers
		* Network Load Balancers
			* Operates at L4 (Transport layer)
			* Use if you want super-fast performance (prod environments or latency-sensitive applications)
		* Classic Load Balancer
			* L4 or L7
			* Legacy, no longer recommended
			* Limited L7 capabilities compared to ALB
	* A deeper look
		* ALB
			* Best suited for load balancing of HTTP and HTTPS traffic
			* Operate at Layer 7 and are application-aware
			* Intelligent, can create advanced request routing
				* Specific requests to specific web servers
		* NLB
			* Best suited for load balancing of TCP traffic where extreme performance is required
			* Operate at Layer 4
			* Capable of handling millions of requests per second while maintaining ultra-low latencies
			* Most costly of all three options
		* CLB
			* Legacy ELB
			* Can load balance HTTP/HTTPS and use some L7 features like X-Forwarded-For and sticky sessions
				* No intelligent content-based routing
			* Can also do L4 load balancing
	* Pre-warming load balancers
		* Imagine running a busy e-commerce website. Black Friday is coming up and it's expected traffic may increase by 10x
			* ELB can scale based on gradual increases in traffic, but may not be able to handle a spike of this magnitude
			* There is a chance that this sudden increase in traffic may cause your ELB to become overloaded and be unable to handle all the requests
			* You can contact AWS and request a pre-warm for your ELB
		* Pre-warming configures the ELB to the appropriate level of capacity based on expected traffic
		* AWS needs to know:
			* Start and end dates
			* Expected request rate per second
			* Total size of a typical request
	* Load Balancers and static IP addresses
		* ALBs scale automatically to adapt to your workload
			* This has the effect of changing the IP address your clients connect to as new ALB nodes are brought into service
		* NLBs solve this by creating a static IP address in each subnet you enable
			* Keeps firewall rules simple -- clients only need to enable access to a single IP address per subnet
		* You can get the best of both worlds by putting an ALB behind an NLB


* **ELB Error Messages**
	* Load Balancer Errors 4XX and 5XX
		* Successful requests for CLB and ALB will be 200 OK
		* Unsuccessful requests will generate either 4XX or 5XX error message
	* 4XX message
		* indicates something has gone wrong on the client side
			* Most likely an issue with the request. Identify, fix, and retry
		* 400 - Bad/malformed request (e.g. heder malformed)
		* 401 - Unauthorized - user access denied
		* 403 - Forbidden - request is blocked by WAF ACL
		* 460 - Client closed connection before LB could respond (client timeout period too short
		* 463 - LB recieved and X-Forwarded-For request header with >30 IP addresses - similar to malformed request
	* 5XX message
		* indicates server-side error
			* Most likely some issue on the server/application. Identify and fix the problem.
		* 500 - Internal server error - errors on the LB
		* 502 - bad gateway - application server closed connection or sent back a malformed response
		* 503 - Service unavailable - no registered targets
		* 504 - Gateway timeout - application is not responding - problem with web server, application server, or database
		* 561 - Unauthorized - received an error code from the ID provider when trying to authenticate a user


* **ELB CloudWatch Metrics**
	* General:
		* ELBs publish metric to CW for the LB itself and for backend instances
		* Helps to verify system is performing as expected
		* You can create a CW alarm to perform a specific action
		* e.g. send an email when a metric reaches a defined limit/range
		* Gathered at 60-second intervals
	* Metrics - overall health
		* BackendConnectionErrors - number of unsucessful connections to backend instances
		* HealhtyHostCount - number of healthy instances registered
		* UnHealthyHostCount - number of unhealthy instances
		* HTTPCode_Backend_2XX,3XX,4XX,5XX - HTTP codes from backend
	* Metrics - performance
		* Latency - number of seconds taken for a registered instance to respond
		* RequestCount - number of requests completed / connections made during specified interval (1 or 5 mins)
		* SurgeQueueLength -  (CLB only) number of pending requests, max queue size is 1024, additional requests will be rejected
		* SpilloverCount - (CLB only) number of reuqests being rejected because the surge queue is full

* **Systems Manager**
	* What is Systems Manager?
		* Systems Manager (SSM) is a management tool which gives you visibility and control over your AWS infra
		* Integrates with CW allowing you to view your dashboards, operational data, and detect problems
		* Includes Run Command which automates operational tasks such as patching and package installs
		* Organize inventory, grouping resources together by application or environment, including on-prem systems
	* Run Command
		* Allows you to run pre-defined commands on one or more EC2 instances
		* Stop, restart, terminate, or re-size instances
		* Attach/detach EBS volumes
		* Create snapshots, backup DynamoDB tables
		* Apply patches and updates
		* Run Ansible playbook
		* Run shell scripts
	* EC2 SSM Role and Resource Groups
		* EC2 instances need SSM role attached to them to interact with SSM
		* Ensure to tag instances so you can group them together in SSM resource groups
	* Insights with SSM
		* Config - auditing and compliance
		* CloudTrail - can check API calls for your resources in resource groups
		* Personal Health Dashboard - notifies you of any operational issues within the region
		* Trusted Advisor - reports and provides recommended actions, service limits, cost optimization, and more
		* CloudWatch dashboards
		* Inventory of instances 
		* Compliance - check how many of your instances have been patched, etc.
	* Actions with SSM
		* There are several pre-defined actions that can be taken on your resources within SSM
		* You can also create and execute your own automation scripts
		* Run Command
			* You can run several different pre-defined commands on your instances
				* e.g. Apply patches, configure Docker, configure CloudWatch Logs, etc.
			* You can also run your own scripts
			* You can send this output to an S3 bucket or view it directly within SSM
		* Patch Manager
			* Set a patch baseline for your systems in SSM
		* Maintenance Windows
			* Automate and schedule tasks to run in a specific time window
		* State Manager
			* Configuration management tool that ensures your resources stay in a consistent state
			* Define a specific baseline configuration. If any manual changes are made, state manager will bring the configuration back to the baseline
	* Shared resources
		* Managed instances
			* All off the EC2 instances configured in SSM
		* Activations
			* Can register on-prem systems from SSM
			* Systems need to have SSM Agent installed
		* Documents
			* Where documents are defined. You can create your own documents here
		* Parameter Store
			* You can store secrets such as passwords, database strings, etc. here


