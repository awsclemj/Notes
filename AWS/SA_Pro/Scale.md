- [Architecting for Scale](#architecting-for-scale)
  - [Concepts](#concepts)
    - [Patterns](#patterns)
    - [Scaling](#scaling)
  - [AutoScaling](#autoscaling)
    - [EC2 Auto Scaling](#ec2-auto-scaling)
      - [EC2 Auto Scaling Policies](#ec2-auto-scaling-policies)
      - [Scaling Cooldown](#scaling-cooldown)
    - [Application Auto Scaling](#application-auto-scaling)
      - [Application Auto Scaling Policies](#application-auto-scaling-policies)
    - [AWS Auto Scaling](#aws-auto-scaling)
  - [Kinesis](#kinesis)
    - [Kinesis Data Streams](#kinesis-data-streams)
    - [Kinesis Data Firehose](#kinesis-data-firehose)
    - [Kinesis Data Analytics](#kinesis-data-analytics)
  - [DynamoDB Scaling](#dynamodb-scaling)
    - [Best practices](#best-practices)
    - [DynamoDB Accelerator (DAX)](#dynamodb-accelerator-dax)
# Architecting for Scale
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Concepts

### Patterns
At its core, a pattern is simply a suggested way to build architectures

**Loosely-Coupled**
* A loosely-coupled architecture has independent components that can stand on their own with little to no knowledge of one another.
* Layers of abstraction
* More flexibility
* Interchangable components
* Atomic (isolated)
* Can scale one step of a process without needing to scale the others

### Scaling
The concept of adding more compute/memory power to an application workload

|Horizontal scaling |Vertical scaling|
--- | --- |
|Add more instances as demand increases|Add more CPU/RAM as demand increases|
|No downtime required to scale up/down|Restart required to scale up/dowm|
|Automatic using AutoScaling Groups|Requires scripting to automate|
|Theoretically unlimited|Limited by instance sizes|

Some terms:
* **Scale out** - use horizontal scalling to add an instance
* **Scale up** - use vertical scaling to increase instance CPU/memory
* **Scale in** - use horizontal scaling to remove an instance
* **Scale down** - use vertical scaling to decrease instance CPU/memory

## AutoScaling

|Name |What |Why|
| --- | --- | --- |
|EC2 Auto Scaling|Focused on scaling EC2 instances|Set up ASG for instances; health checks to remove unhealthy instances|
|Application Auto Scaling|API used to control scaling for resources like Dynamo, ECS, EMR|Provides common way of using scalability with other services|
|AWS Auto Scaling|Centralized way to manage scalability for whole stacks; predictive scaling|Unified console to manage all auto scaling|

### EC2 Auto Scaling
* Provides **horizontal scaling** for EC2 instances
* Triggered by an event or scaling action
* Availability, cost, and system metrics can all factor into scaling
* Four scaling options:
  * Maintain - keep minimum number of instances running
  * Manual - use maxiumum, minimum, or specific number of instances
  * Schedule - increase/decrease based on schedule
  * Dynamic - scale based on real-time metrics of systems
* Use a **launch configuration** to specify details needed to launch instances
  * AMI, VPC, subnet, instance profile, health check grade period etc. 

#### EC2 Auto Scaling Policies
* Target tracking policy - scale based on predefined/custom metric based on a target value (e.g. CPU 70%)
* Simple scaling policy - Waits until health check/cool down period expires before evaluating new need
* Step scaling policy - Responds to scaling need with sophistication and logic

#### Scaling Cooldown
* Configurable duration that gives the system a chance to absorb load
* Default 300 Seconds; can be overridden by scaling-specific cooldown
* Automatically applies to dynamic scaling, optional for manual, not supported for scheduled

### Application Auto Scaling

#### Application Auto Scaling Policies
* Target tracking policy - initiates scaling event to try to track a given target
  * Scale my ECS hosts to stay at 70% CPU
* Step scaling policy - based on a metric, adjusts capacity given certain defined thresholds
  * Scale my EC2 spot fleet by 20% when 10,000 connections are added to my ELB
* Scheduled scaling policy - initiates scaling based on predefined date, time, etc.
  * Every Monday increase my DynamoDB RCU to 20,000

### AWS Auto Scaling
* Exposed as a console to manage the above auto scaling services
* Predictive scaling feature:
  * Dynamically scale based on learning your load and calculating expected capacity needs
  * Can opt-out of this data collection

## Kinesis
* Collection of services for processing streams of data; **transient data store** with retention between 24 hours and 7 days
* Data processed in **shards**. Each shard can ingest 1000 **records** per second
* Default limit of 500 shards (can be increased)
* **Records** consist of a partition key, sequence number, and data blob (up to 1 MB)

### Kinesis Data Streams
* Ingests and stores data streams for processing
* Send data to processing tools
  * e.g. Lambda, EMR, EC2, Kinesis Data Analytics
* Output processed data to BI tools

### Kinesis Data Firehose
* Prepares and loads data continuously to a destination of your choice
* Destinations: S3, Redshift, ElasticSearch, Splunk
* Analyze streaming data using analysis tools

### Kinesis Data Analytics
* Capture streaming data from Kinesis Data Stream or Firehose
* Run standard SQL queries against data
* Output processed data to analytics tools; create alerts and respond in real time

## DynamoDB Scaling
* Throughput - Read/Write Capacity Units
* Size - limitless amount of storage space, but items can be 400 KB max

|Term |What|
| --- | --- |
|Partition|Physical space where DynamoDB data is stored|
|Partition key|Unique identifier for each record; sometimes called **hash key**|
|Sort key|In combination with partition key, optional second part of a composite key that defines storage order; sometimes called a **range key**|

|Partition Calculations|
| --- | --- |
|Capacity|(Total RCU / 3000) + (Total WCU / 1000)|
|Size|Total size / 10 GB|
|Total partitions|Round up for the max (capacity and size)|

### Best practices
* Choose a partition key that will be unique when hashed so items can be equally distributed across partitions
* You can choose to use auto scaling to automatically scale RCU/WCU upon certain conditions
  * Uses target tracking method 
  * Does not scale down if the table consumption drops to zero
* DDB auto scaling also support global secondary indexes
  * These are like a copies of the table that are indexed/keyed differently
* **On-demand scaling** can also be used as an alternative to traditional auto scaling
  * Useful if you can't deal with scaling lag or don't know anticipated capcity requirements
  * Instantly allocated capacity as needed; no concept of provisioned capacity
  * Costs more

### DynamoDB Accelerator (DAX)
* Sits in front of a DDB table as an in-memory cache
* Microsecond response times
* Great for read-intensive applications
* If application caches locally, this is likely a waste