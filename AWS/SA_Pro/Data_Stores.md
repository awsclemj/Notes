- [Data Stores](#data-stores)
  - [Concepts](#concepts)
    - [Data Store Types](#data-store-types)
    - [IOPS vs. Throughput](#iops-vs-throughput)
    - [Consistency Models](#consistency-models)
  - [S3](#s3)
    - [Consistency](#consistency)
    - [Security](#security)
    - [Data Protection](#data-protection)
    - [Lifecycle Management](#lifecycle-management)
    - [Other features](#other-features)
  - [Glacier](#glacier)
    - [Terms](#terms)
  - [EBS](#ebs)
    - [Snapshots](#snapshots)
    - [Data Lifecycle Manager](#data-lifecycle-manager)
  - [EFS](#efs)
  - [Storage Gateway](#storage-gateway)
    - [Modes](#modes)
  - [Databases on EC2](#databases-on-ec2)
  - [RDS](#rds)
    - [Anti-patterns](#anti-patterns)
  - [Dynamo](#dynamo)
    - [Relational vs. NoSQL](#relational-vs-nosql)
    - [Keys and Indexes](#keys-and-indexes)
  - [Redshift](#redshift)
    - [Data Lakes](#data-lakes)
  - [Neptune](#neptune)
  - [Elasticache](#elasticache)
    - [Use Cases](#use-cases)
    - [Redis vs. Memcached Use Cases](#redis-vs-memcached-use-cases)
  - [Other Database Options](#other-database-options)
    - [Athena](#athena)
    - [Quantum Ledger Database (QLDB)](#quantum-ledger-database-qldb)
    - [Amazon Managed Blockchain](#amazon-managed-blockchain)
    - [Amazon Timestream Database](#amazon-timestream-database)
    - [Amazon DocumentDB](#amazon-documentdb)
    - [ElasticSearch](#elasticsearch)

# Data Stores
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Concepts

### Data Store Types
* **Persisten**t - Data is durable; e.g. S3, DynamoDB
* **Transient** - Data is temporarily stored then passed along. e.g. SNS, SQS
* **Ephemeral** - Data is lost when stopped; e.g. EC2 instance store, Elasticache Memcached

### IOPS vs. Throughput
* **IOPS** - how fast we can read/write to device
* **Throughput** - how much data can be moved at a time

### Consistency Models
* **ACID** - e.g. RDS databases
  * Atomic - transactions are all of nothing
  * Consistent - transactions must be valid
  * Isolated - transactions can't mess with one another
  * Durable - completed transaction must stick around

* **BASE** - e.g. S3, Dynamo
  * Basic availability - values availability even if stale
  * Soft-state - might not be instantly consistent across stores
  * Eventual consistency - will achieve consistency at some point

## S3
* Object storage; max object size is 5TB, max PUT is 5GB
* Recommended to use multi-part uploads for objects larger than 100MB
* **Object storage**
  * More in common with a database than file storage
  * "Paths" are actually key names for records

### Consistency
* Read after write consistency model for PUTs of new objects (can be read immediately)
* HEAD or GET requests of the key before the object exists will result in eventual consistency
* Eventual consistency for overwrite PUTs and DELETEs
* Updates to a single key are atomic 

### Security
* Resource-based - ACLs and bucket policies
* User-based - IAM
* MFA delete

* Encryption at rest
  * SSE-S3 - AWS-managed key
  * SSE-C - use a key that you manage 
  * SSE-KMS - use a key that KMS manages
  * Client-side - encrypt data on the client-side before uploading

### Data Protection
* Versioning
  * Enables roll-back and un-delete capabilities
  * Integrates with lifecycle management
  * Old versions are billable until permanently deleted
* MFA
  * For deletions or changing versioning state on a bucket
* Cross-region replication
  * Compliance requirements
  * Durability
  * Better latency

### Lifecycle Management
  * Optimize costs
  * Data retention requirements 
  * Maintain S3 data volumes

### Other features
* Transfer acceleration - speed up data uploads using CloudFront 
* Requester pays - requester pays for data transfer
* Tags - assign metadata to objects
* Events - trigger notifications to SNS, SQS, Lambda
* Static web hosting - host a static web site 
* BitTorrent - use BitTorrent protocol to retrieve any publicly-available content using the .torrent file extension

## Glacier
* Cold storage; archival solution
* Used by AWS Storage Gateway Virtual Tape Library
* Integrated with S3 lifecycle management
* Faster retrieval speeds if you pay more

### Terms
* **Vault** - analogous to an S3 bucket
* **Archive** - analogous to an S3 object
  * Zip, tar, etc. 
  * Max size 40TB
  * Immutable 
* **Vault Lock**
  * Enforce rules like no deletes or MFA
  * Immutable
  * 24 hours to abort or complete after enabling

## EBS
* Virtual hard drives for EC2
* Tied to a single AZ
* Variety of choices for IOPS, throughput, cost
* Snapshots
* Pay for the amount of blocks even if unused 

### Snapshots
* Cost-effective backup strategy
* Share data sets with other users or accounts
* Migrate a system to another AZ or region
* Convert unencrypted volume to encrypted volume
* After initial snapshot of volume, new snapshots are incremental
* Even if a previous snapshot version is deleted, subsequent snapshots can still be used for point-in-time recovery

### Data Lifecycle Manager
* Schedule snapshots for volumes/instances every *x* hours
* Retention rules

## EFS
* Implementation of NFS
* Elastic storage capacity, pay for only what you use (different than EBS In this regard)
* Multi-AZ metadata and data storage
* Configure mount points in multiple AZs
* Can be mounted from on-prem systems
  * Ensure stable connection like DX
  * Consider encryption in transit
  * Alternatively use **Amazon DataSync**
* NFSv4 features aren't fully supported (check docs)

## Storage Gateway
* VM you can download and run on-prem with VMware/HyperV/Dell appliance
* Provides local storage backed by S3 or Glacier
* Often used for DR preparedness and/or migrations

### Modes
* File Gateway - on-prem or EC2 instances can use NFS/SMB mount point to sync file systems with S3
* Volume gateway stored mode (Gateway-stored volumes)
  * iSCSI async replication of on-prem data to S3
* Volume gateway cached mode (Gateway-cached volumes)
  * iSCSI interface
  * Primary ata stored on S3, frequently retrieved objects are cached on-prem
* Tape gateway (Gateway-virtual tape library)
  * iSCSI interface
  * Virtual media changer and tape library to be used with existing backup software

## Databases on EC2
* Can run any database on EC2; full control
* Must manage everything (backups, redundancy, patching, etc)
* Good option for DBs not supported by RDS like IBM DB2 or SAP HANA

## RDS
* Managed relational OLTP DB service for several engines
* Drop-in replacement for existing on-prem DBs
* Automates admin like backups and patching
* Push-button scaling, replication, and redundancy

### Anti-patterns
* Large binaries (blobs) - use S3, not RDS
* Automated scalability - use Dynamo
* Name-value data structure, use Dynamo
* Data not well-structured/predictable - use Dynamo
* Database not supported by RDS - EC2

## Dynamo
* Managed, multi-AZ NoSQL data store with cross-region replication option
* Defaults to eventual consistency, can request strongly consistent reads
* Priced on throughput; provision read/write capacity in anticipation of need
* Autoscale capacity adjusts per configured min/max levels
* On-demand capacity for flexible capacity at small premium
* Achieve ACID compliance with DynamoDB Transactions

### Relational vs. NoSQL
* **Relational**
  * Structured data; typically referred to as tables
  * Tables relate to one another
* **NoSQL**
  * Unstructured key-value pairs
  * Does not need to relate to any other tables in the DB

### Keys and Indexes
  * **Partition key** - a unique primary key that can be used to access data
  * **Sort key** - used in combination with a partition key as a **composite primary key** to ensure unique records. Partition keys can be the same across multiple records in this scenario. 
  * **Global secondar index** - Partition key and sort key can be different than those on the table
    * Fast query of attributes outside of the primary key (e.g. sales orders by customer number instead of order number)
  * **Local secondary index** - Same partition key as the table, but different sort key
    * Already know the partition key but want to query on some other attribute (e.g. have the order number but want to retrieve records with a specific material number)

## Redshift
* Fully-managed, clustered, petabyte-scale data warehouse for OLAP workloads
* Extremely cost-effective compared to other on-prem data w3arehouse platforms
* Postgres compatible with JDBC and ODBC drivers available; compatible with most BI tools
* Parallel processing and columnar data stores
* Option to query directly from data on S3 via Redshift Spectrum

### Data Lakes
* Query raw data without extensive pre-processing
* Lessen time from data collection to data value
* Identify correlations between disparate data sets

## Neptune
* Fully-managed graph database
* Supports open graph APIs for Gremlin and SPARQL
* Store inter-relationships between data sets

## Elasticache
* Fully-managed implementations of Redis and Memcached
* Push-button scalability for memeory, writes, and reads
* In-memory key-value store - not persistent in a traditional sense
* Billed by node size and usage hours

### Use Cases
* Web session store - use Redis to store session data. If a server is lost, the session info is not lost
* Database caching - use Memcached in front of RDS to cache popular queries
* Leaderboards - use Redis to provide a live leaderboard for millions of users
* Streaming data dashboards - provide landing spot for streaming sensor data on the factory floor

### Redis vs. Memcached Use Cases
* **Redis**
  * Need encryption
  * Need HIPAA compliance
  * Support for clustering/HA
  * Need complex data types
  * Pub/Sub capability
  * Geospacial indexing
  * Backup/restore

* **Memcached**
  * Simple, no-frills
  * Scale out and in as demand changes
  * Need to run multiple CPU cores/threads
  * Need to cache objects like DB queries

## Other Database Options

### Athena
* SQL engine overlaid on S3, based on Presto
* Query raw data objects as the sit in s3
* Use or convert data to Parquet format for big performance boost
* Similar to Reshift Specturm, but
  * Use Athena if your data lives mostly on S3 and you don't need to perform joins to other data sources
  * Use Specturm if you want to join S3 data with Redshift tables or create unions

### Quantum Ledger Database (QLDB)
* Based on blockchain 
* Provides an immutable journal as a service without having to maintain an entire blockchain
* Centralized design (as opposed to decentralized like in traditional blockchain) -- higher performance/scalability

### Amazon Managed Blockchain
* Fully managed blockchain framework supporting open source frameworks such as Hyperledger Fabric and Ethereum
* Distributed, consensus-based concept consisting of network, members, nodes, and potentially apps
* Uses QLDB ordering service to maintain history of transactions

### Amazon Timestream Database
* Fully managed time-series database solution
* Alternative to Dynamo or Redshift and include built-in analytics like interpolation and smoothing
* Useful for IoT/sensors

### Amazon DocumentDB
* Fully-managed DB compatible with MongoDB -- emulates MongoDB API and fully compatible with Mongo clients
* Redundant, scalable, HA, backups

### ElasticSearch
* Fully-managed search engine and document store
* Components are sometimes referred to as the ELK stack
  * Search/storage - ElasticSearch
  * Intake - LogStash (but also CloudWatch, Firehose, IoT)
  * Analytics - Kibana