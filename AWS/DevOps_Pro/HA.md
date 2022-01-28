- [High Availability, Fault Tolerance, and Disaster Recovery](#high-availability-fault-tolerance-and-disaster-recovery)
  - [AWS Single Sign-On](#aws-single-sign-on)
    - [Features](#features)
  - [CloudFront](#cloudfront)
  - [Auto Scaling and Lifecycle Hooks](#auto-scaling-and-lifecycle-hooks)
    - [Auto Scaling Lifecycle](#auto-scaling-lifecycle)
      - [How lifecycle hooks work](#how-lifecycle-hooks-work)
      - [Cooldowns](#cooldowns)
      - [Abandon or Continue](#abandon-or-continue)
      - [Spot instances](#spot-instances)
  - [Route 53](#route-53)
    - [Features](#features-1)
    - [Routing Policies](#routing-policies)
  - [Amazon RDS](#amazon-rds)
    - [Available Engines](#available-engines)
    - [Managed Administration](#managed-administration)
    - [Scaling](#scaling)
  - [Aurora](#aurora)
    - [Features](#features-2)
    - [Backups and Snapshots](#backups-and-snapshots)
      - [Backups](#backups)
      - [Snapshots](#snapshots)
    - [Failure and Fault Tolerance](#failure-and-fault-tolerance)
      - [Failure](#failure)
      - [Fault Tolerance](#fault-tolerance)
    - [Replicas](#replicas)
      - [Aurora Replicas](#aurora-replicas)
      - [MySQL Read Replicas](#mysql-read-replicas)
    - [Security](#security)
    - [Connection Management](#connection-management)
  - [DynamoDB](#dynamodb)
    - [Features](#features-3)
    - [Concepts](#concepts)
    - [Tables](#tables)
    - [Attribute types](#attribute-types)
    - [Primary Key](#primary-key)
    - [Secondary Index](#secondary-index)
    - [Streams](#streams)

# High Availability, Fault Tolerance, and Disaster Recovery
Notes taken from ACG DevOps Pro 2020 course.

## AWS Single Sign-On
* Service that makes it easy to centrally manage SSO access to multiple AWS accounts and business applications

### Features
* Integrates with AWS Organizations
* Manage access to cloud applications without third-party solutions
* Create/manage users and group membership to AWS accounts
* Use existing enterprise identity services (AD, Okta, etc.)

## CloudFront
* A fast content delivery network (CDN) that securely delivers data, videos, applications, and APIs to customers globally
* Example of how this works:

User --> Route 53 --> CloudFront --> Origin (can be static or dynamic content)

* Static content -- generally stored in S3; might have path like `images/*`, `css/*`, etc.
* Dynamic content -- generally served from your application servers and may be fronted by a load balancer. Might have paths like `default(*)`, `login.php`, `admin/*`, etc. 

## Auto Scaling and Lifecycle Hooks 
* Scale your EC2 instance capacity automatically according to your defined conditions
* Some concepts:
  * **Minimum size**: the absolutely minimum number of instances that must be running at a given time
  * **Desired capacity**: the desired amount of instances you want running
  * **Maximum size**: the most number of instances that you'd like running in the ASG

### Auto Scaling Lifecycle
* Starts when an ASG launches and instance
* Ends when the instance is terminated
* Ends when the ASG takes the instance out of service for termination
* Example:

Scale out request --> Pending state --> EC2_INSTANCE_LAUNCHING hook --> InService

* If Auto Scaling determines the instance needs to be terminated, either because of failing health checks or a scale-in event:

InService --> Terminating --> EC2_INSTANCE_TERMINATING hook --> Terminated

* There is an additional standby state for troubleshooting instances without having to take them out of the ASG

InService --> EnteringStandyby --> Standby

* You can also detach instances from the ASG:

InService --> Detaching --> Detached

* After detaching, you can reattach the instances so they are managed by the ASG again

#### How lifecycle hooks work
1. ASG responds to a scale out event by launching an instance
2. Instance goes into the Pending:Wait state
3. ASG sends a message to the notification target defined for the hook, along with information and a token
4. Waits until you tell it to continue or timeout occurs
5. By default, instance waits for an hour and will change to Pending:Proceed, then it will enter the InService state

Additional notes:
* You can change the heartbeat timeout or define it when you create the lifecycle hook in the CLI with `heartbeat-timeout` parameter
* You can call the `complete-lifecycle-action` command to tell the ASG to proceed
* You can call the `record-lifecycle-action-heartbeat` command to add more time to the timeout
* 48 hours is the max time you can keep a server in a wait state, regardless of heartbeats

#### Cooldowns
* Ensure the ASG doesn't launch/terminate more instance than needed
* Start when an instance enters InService state

#### Abandon or Continue
* At the conclusion of a lifecycle hook, an instance can result in either an ABANDON or CONTINUE state
* Abandon causes the ASG to terminate the instance and launch a new one if necessary
* Continue will put the instance into service

#### Spot instances
* You can use lifecycle hooks with spot instances, but this does not prevent an instance from terminating due to a change in spot price
* When a spot instance terminates, you must still complete the lifecycle action

## Route 53
* Highly-available and scalable DNS service

### Features
* Built on AWS's HA infra. DNS is distributed an Amazon uses reasonable efforts to keep R53 100% available.
* Interfaces directly with EC2 and S3; connect DNS records to load balancers and S3 buckets.
* Make you fault tolerant; provides multiple routing types to ensure your customers have a great experience.

### Routing Policies
* **Failover** - active/passive failover
* **Geolocation** - used when you want to route traffic based on location of users
* **Geoproximity** - must be used with traffic flow feature. Used when you want to route traffic based on location of your resources
* **Latency** - used when you want to route traffic to the region that provides best latency for the user
* **Multivalue** - used when you want to route randomly to up to eight destinations
* **Weighted** - used to route traffic to different resources using a percentage split

## Amazon RDS
* Create, run, and scale relational databases in the cloud
* Fast, cost efficient, resizable, secure, and have the option of high availability

### Available Engines
* MySQL
* MariaDB
* MS SQL
* PostgreSQL
* Oracle
* Aurora (MySQL or Postgres)

### Managed Administration
* Provision underlying infra
* Install DB software
* Automated backups
* Automatic patching
* Synchronous data replication
* Automatic failover
* You are responsible for:
  * Settings
  * Schema
  * Performance tuning

### Scaling
* Vertical:
  * Simply change the instance type
  * Smallest: db.t3.micro $0.017/hr
  * Largest: db.r5.24xlarge $11.52/hr
  * Storage can be scaled live (except MS SQL)
* Horizontal:
  * Read replicas
  * Read-only copies synchronized with primary DB
  * Can be in different regions
  * Can be promoted to primary
  * Can create replicas of replicas

## Aurora
* MySQL and Postgres-compatible database built for the cloud

### Features
* Fast, reliable, and cost-effective compared to other high-end commercial DBs
* 5x throughput of MySQL on same hardware
* Compatible with MySQL 5.7
* Storage is fault-tolerant and self healing
* Disk failures repaired in the background
* Detects crashes and restarts
* No crash recovery or cache rebuilding required
* Automatic failover to one of up to 15 read replicas
* Storage auto scaling from 10GB to 64TB

### Backups and Snapshots

#### Backups
* Automatic, continuous, and incremental backups
* Point-in-time restore within a second
* Up to 35-day retention
* Stored in S3
* No impact of DB performance

#### Snapshots
* User-initiated
* Stored in S3
* Kept until they are explicitly deleted
* Incremental

### Failure and Fault Tolerance

#### Failure
* 6 copies of your data stored across 3 AZs
* Upon failure, attempts recovery in a healthy AZ
* If this fails, PIT or snapshot restore

#### Fault Tolerance
* Data divided into 10GB segments across many disks
* Transparently handles loss
* Can lose 2 copies without affecting write
* Can lose 3 copies without affecting read
* All storage is self-healing

### Replicas

#### Aurora Replicas
* Share underlying volume of the primary instance
* Updates made by primary are immediately available to replicas
* Can have up to 15
* Low performance impact on primary
* Replica can be a failover target with no data loss

#### MySQL Read Replicas
* Primary instance data is replayed on your replica as transactions
* Supports up to 5
* High performance impact on primary
* Replica can be failover target, but can potentially have minutes of data loss

### Security
* Must be created in a VPC
* SSL/TLS AES-256 used to secure data in transit
* You can encrypt databases with AWS KMS
* Encrypted data storage:
  * Database volumes
  * Backups
  * Snapshots
  * Replicas
* You cannot encrypt an unencrypted database

### Connection Management
* **Cluster endpoint** -- connection to the current primary DB instance for the cluster
* **Reader endpoint** -- provides load-balancing support for read-only connections
* **Custom endpoint** -- represents a set of DB instances of your choice; up to 5
* **Instance endpoint** -- connect to a specific instance in the DB cluster

## DynamoDB
* Fully-managed NoSQL database that supports key-value and document data structures

### Features
* Predictable and fully manageable performance with seamless scalability
* No visible servers
* No practical storage limitation
* Fully resilient and HA
* Performance scales linearly
* Fully integrated with IAM for granular security controls

### Concepts
* DDB at its core is a collection of tables
* Tables are the highest-level structure within a DB
* You can specify read and write performance requirements per table
* **Write Capacity Units (WCU)** - number of 1KB blocks per second
* **Read Capacity Units (RCU)** - number of 4KB blocks per second
* DDB uses the performance and data quality to manage underlying resource provisioning
* Unlike SQL databases, data structures or schema is not defined at the table level

### Tables
* Tables are split into rows. In Dynamo, these are referred to as **items**
* Each **item** contains a number of elements, called **attributes**
* Each **item** can have a different number of **attributes**, or even different **attributes** (this is a distinct difference from SQL)
* However, each **item** must have a **partition key**, a unique identifier for each **item**
* Optionally, you can also have a **sort key** to more granular queries

### Attribute types
* String: `"hello"`
* Number: `123`
* Binary: `<binary blob>`
* Boolean: `true/false`
* Null
* Document (List/Map): `JSON`
* Set: (array) `["red","green","blue"]`

### Primary Key
* Must be created along with a table. Includes a partition key, and optionally a sort key.
* If used alone, the partition key must be unique to each item in the table. Ex: ProductID
* Sort keys do not need to be unique to an item, but if there are items with the same partition key, the sort key must be unique to those items. These can be used to easily query certain data sets. i.e. Type or Date
* When used in conjunction, a primary key and sort key together are referred to as a **composite primary key**
* The partition key is used as an input to a hash function that determines where data will be stored in a table

* A partition key is also known as an item's **hash attribute**
* A sort key is also known as an item's **range attribute**

### Secondary Index 
* Allows you to query the table data using an alternate key
* You can create one or more indexes on a table and they are each queryable like the table
* Two kinds:
  * **Global secondary index** -- index with a partition key and sort key that can be different from those on the table
  * **Local secondary index** -- index with the same partition key but different sort key from those on the table

### Streams
* An optional feature that captures modification events in DDB tables
* Data about the events appear in the stream in near real-time and in order
* Each event is represented by a stream record when:
  * New item is added
  * Item is updated
  * Item is deleted
* You can trigger AWS Lambda when a particular event appears in the stream