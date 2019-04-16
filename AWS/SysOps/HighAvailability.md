## High Availability

* **Elasticity and Scalability 101**
	* Elasticity
		* Stretch and contract infra based on demand
		* Pay for only what you need
		* Used during a short time period, such as hours or days
	* Scalability
		* Used to talk about building out infra long term
		* Made over longer time periods
	* EC2
		* Scalability - increase instance sizes, use reserved instances
		* Elasticity -  increase the number of EC2 instances based on autoscaling
	* DynamoDB
		* Scalability - unlimited amount of storage
		* Elasticity - increase additional IOPS for spikes in traffic, decrease after the spike
	* RDS
		* Scalability - increase instance size
		* Elasticity - not very elastic, can't scale based on demand
	* Aurora
		* Scalability - Modify instance type
		* Elasticity - Aurora Serverless

* **RDS Multi-AZ Failover**
	* Multi-AZ 101
		* Keeps a copy of prod DB in a separate AZ in case of failure/disaster
		* AWS manages failover from one AZ to another automatically
			* Planned maintenance, DB instance failure, AZ failure
		* AWS also handles replication for you. When your prod DB is written to, the write will be synced to the standby DB.
	* How Failover Works
		* Servers connect to RDS via a connection string, including an RDS endpoint name
		* RDS endpoint resolves to DNS of prod DB
		* If failure in one AZ is detected, RDS automatically updates the DNS to point at the standby DB in another AZ
		* Note that Multi-AZ RDS is not primarily used for performance enhancement. Use Read Replicas for this.
	* Replication specifics
		* MySQL, Oracle, and PostgreSQL engines
			* Synchronous physical replication
		* SQL Server engine
			* Synchronous logical replication
			* SQL Server-native mirroring tech
	* Advantages
		* High availability
		* Backups and restores are taken from the standby DB which avoids I/O suspension to primary
		* You can force failover from one AZ to another by rebooting your instance

* **RDS Read Replicas**
	* What are Read Replicas?
		* Read Replicas make it easy to take advantage of supported engines' replication functionality
		* Elastically scale out beyond the capacity constraints of a single DB instance for read-heavy workloads
		* Read-only copy of DB
		* Can create a RR with just a few clicks
		* Update on the source DB instance will be replicated asynchronously using the engine's native function
		* Can create multiple RRs and distribute application's read traffic amongst them
	* When would you use RR?
		* Scaling beyond compute or I/O capacity of single DB instance
		* For read-heavy workloads
		* RRs can be used if the source DB instances is unavailable (e.g. backup/scheduled maintenance)
		* Business reporting or data warehousing scenarios
			* May want to have business reporting queries to run against an RR instead of prod DB
	* Supported Versions
		* MySQL, PostgreSQL, MariaDB
			* Engine-native asynchronous replication
			* You can have up to 5 RRs for these engines
		* Aurora
			* SSD-backed virtualized storage layer built for DB workloads
			* Replicas share same underlying storage as source instance
				* Lowers cost and avoids the need to copy data to replica nodes
	* Creating RRs
		* When creating a new RR, AWS takes a snapshot of your DB
			* If multi-AZ not enabled
				* Snapshot is of primary DB and can cause I/O suspension
			* If multi-AZ enabled
				* Snapshot is of standby; no performance hit on primary
	* Connecting to RR
			* RR has a separate endpoint DNS name to connect to
	* RRs can be promoted
		* You can promote an RR to its own standalone DB. This breaks replication links between primary and RR.
	* Other exam tips
		* You can have RRs in different regions for all engines
		* Replication is asynchronous only
		* Can be built off of Multi-AZ DBs
		* RRs can be Multi-AZ
		* You can have RRs of RRs, but beware of replication latency
		* DB snapshots cannot be taken of RRs
		* Key metrics to looks for is replica lag

* **Elasticache**
	* In-memory cache in the cloud. 
	* Improves the performance of web applications by allowing you to retrieve information from fast, in-memory cache instead of relying solely on disk-based DBs
		* Caches most frequently accessed DB queries
	* Can be used to significantly improve latency and throughput for read-heavy or compute-extensive workloads
	* Caching improves app performance by storing critical pieces of data in memory of low-latency access
	* Types
		* Memcached
			* Widely adopted
			* Popular tools that you use today with Memcached will work seamlessly
			* Does not support Multi-AZ
		* Redis
			* Popular open-source in-memory key-value store that supports data structures such as sorted sets and lists
			* Elasticache supports Master/Slave replication and Multi-AZ which can be used to achieve cross AZ redundancy

* **Aurora 101**
	* Amazon-built HA RDS solution
	* Supports MySQL and PostgreSQL
	* Combines speed and availability of high-end relational databased with the simplicity and cost effectiveness of open-source DBs.
		* Five times better performance than PostgreSQL, 1/10th the cost of commercial DBs.
	* Scaling
		* Starts with 10GB, scales in 10GB increments up to 64TB
		* Compute can scale up to 64 vCPUs and 488 GiB of memory
		* 2 copies of your data is contained in each AZ, with a minimum of 3 AZs (6 copies)
		* Can handle the loss of up to two copies of data without affecting DB write availability
		* Can lose 3 copies without affecting read availability
		* Storage is self-healing. Disks and data blocks are continuously scanned for errors and repaired
	* Replicas
		* 2 types
			* Aurora replicas (up to 15)
			* MySQL read replicas (up to 15)
	* Aurora at 100% CPU Util?
		* If writes are the cause, scale up instance size
		* If reads are the cause, scale out number of replicas
	* Other Aurora specifics
		* Encryption at rest is turned on by default. All RRs will be encrypted
		* Failover defined by tiers. The lower the tier, the higher the priority
		* Cross region replicas - replicates entire cluster to another region
	* Aurora Serverless
		* On-demand autoscaling configuration for Aurora MySQL edition 
		* DB will automatically start up, shut down, and scale up/down capacity based on needs
		* Pay on per-second basis for DB capacity you use when DB is active
		* Can migrate between standard and server less easily in RDS console

* **Troubleshooting Autoscaling**
	* If instance is not launching into ASG:
		* Associated key pair doesn't exist
		* Security group doesn't exist
		* Autoscaling config is not working correctly
		* ASG not found
		* Instance type specified not supported in AZ
		* AZ no longer supported
		* Invalid EBS device mapping
		* Autoscaling service is not enabled in the account
		* Attempting to attach an EBS block device to and instance-store AMI