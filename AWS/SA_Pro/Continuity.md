- [Business Continuity](#business-continuity)
  - [Concepts](#concepts)
  - [Disaster Recovery Architectures](#disaster-recovery-architectures)
  - [Storage HA Options](#storage-ha-options)
    - [EBS](#ebs)
    - [S3](#s3)
    - [EFS](#efs)
  - [Compute HA Options](#compute-ha-options)
  - [Database HA Options](#database-ha-options)
    - [Dynamo vs. RDS](#dynamo-vs-rds)
    - [Redshift](#redshift)
    - [Memcached](#memcached)
    - [Redis](#redis)
  - [Networking HA](#networking-ha)

# Business Continuity
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Concepts
* We need to architect assuming everything will fail
* **Business Continuity (BC)** - seeks to minimize business disruption when something unexpected happens
* **Disaster Recovery (DR)** - act of responding to an event that threatens business continuity
* **High Availability (HA)** - designing redundancies to reduce the chance of impacting service levels
* **Fault Tolerance** - designing the ability to absorb problems without impacting service levels
* **Service Level Agreement (SLA)** - an agreed goal or target for a given service based on performance/availability
* **Recovery Time Objective (RTO)** - time it takes after a disruption to restore business processes to service levels
* **Recovery Point Objective (RPO)** - acceptable amount of data loss measured in time

Types of disasters:
|Category |Example|
| --- | --- |
|Harware failure|Network switch PSU fails|
|Deployment failure|Deploying a patch breaks a key ERP business process|
|Load induced|DDoS attack|
|Data induced|Ariane 5 rocket explosion|
|Cred expiration|TLS certificate expires|
|Dependency|S3 subsystem failure caused numerous AWS service failures|
|Infrastructure|Contruction crew cuts fiber cable|
|Identifier exhaustion|"We do not have sufficient capacity in the AZ you requested"|
|Human error|`sudo rm -rf /`|

## Disaster Recovery Architectures
* **Backup and restore**
  * E.g. use Snowball/Storage Gateway/VTL to sync backup data to S3
  * Pros:
    * Common entry point into AWS
    * Minimal effort to configure
  * Cons:
    * Least flexible
    * Analogous to off-site backup
* **Pilot light**
  * Spin up minimal footprint in AWS to act as a standby
  * Pros:
    * Cost-effective way to maintain a "hot site"
    * Suitable for variety of landscapes/apps
  * Cons:
    * Usually requires manual intervention for failover
    * Spinning up cloud environments could take minutes to hours
    * Must keep AMIs up-to-date with on-prem counterparts
* **Warm standby**
  * Services already up and running; ready to accept load
  * Pros:
    * All services are up and ready to accept a failover faster than pilot light (minutes to seconds)
    * Could be used as "shadow environment" for testing/staging
  * Cons:
    * Resources need to be scaled to accept prod load
    * Still requires some environment adjustments (could be scripted)
* **Multi-site**
  * Environment ready to accept load instantly at any time
  * Pros:
    * Ready to take on full prod load (effectively mirroring on-prem)
    * Fails over in seconds
    * Little to no intervention required to fail over
  * Cons:
    * Expensive
    * Could be perceived as wasteful of the resources

## Storage HA Options
### EBS
* Annual failure rate of less than 0.2%
  * Normal HDDs fail at 4%
* Availability target of 99.999%; data replicated within a single AZ
  * Vulnerable to AZ failure
* Easy to snapshot to S3, which is AZ-durable
* Copy snapshots to other regions
* Supports RAID
  * Recommended to stick with RAID0 or RAID1, as RAID5/6 parity bits can consume a lot of throughput

RAID IOPS/Throughput:
|RAID Level |Volume Size |Provisioned IOPS |Total Volume IOPS |Usable Space |Throughput |
| --- | --- | --- | --- | --- | --- |
|No RAID|(1) 1000GB|4000|1000GB|500 MB/s|
|RAID0|(2) 500GB|4000|8000|1000GB|1000 MB/s|
|RAID1|(2) 500GB|4000|4000|500GB|500 MB/s|

### S3
* Standard class - 99.99$ availability, 52.6 mins/yr
* Standard infrequent access - 99.9% availability, 8.76 hrs/yr
* One-zone infrequent access - 99.5% availability, 1.83 days/yr
* Eleven nines of durability
* Standard and standard-IA have multi-AZ durability, one-zone has single AZ

### EFS
* Managed implementation of NFS 
* True file system as opposed to block storage or object storage
* File locking, strong consistency, concurrently accessible
* Each file and metadata is stored across multiple AZs
* Can be accessed from all AZs concurrently
* Mount targets are HA

## Compute HA Options
* Up-to-date AMIs for fail-over
* AIMs can be copied to other regions for safety/DR
* Horizontally scaling architectures
  * Risk spread across multiple smaller VMs vs. one large VM
* Reserved instances is the only way to guarantee resources will be available
* Auto scaling + elastic load balancing
* R53 health checks for traffic redirection

## Database HA Options
### Dynamo vs. RDS
* Choose Dynamo over RDS if possible
* Choose Aurora if using RDS
* Choose multi-AZ RDS if you can't use Aurora
* Snapshot RDS frequently
  * Multi-AZ deployments aren't impacted by snapshots
* Regional replication an option, but not strongly consistent

### Redshift
* Doesn't support multi-AZ at this time
* Use a multi-node cluster -- supports replication and node recovery
* Single node clusters do not support replication; need to restore from snapshot

### Memcached
* Does not support replication; failure results in data loss
* Use multiple nodes in each shard to minimize data loss
* Launch noes across AZs to minimize data loss on AZ failure

### Redis
* Use multiple nodes in each shard and distribute nodes across multiple AZs
* Enable multi-AZ on the replication group to permit auto-failover if primary fails
* Schedule regular backups

## Networking HA
* Create subnets in multiple AZs
* Create at least two VPN tunnels to your VGW/TGW
* Direct Connect is not HA be default; need to establish a secondary connection or use VPN
* Route 53 health checks
* Use elastic IPs for flexibility of changing underlying infrastructure without impacting name resolution
* Create NAT gateways in multiple subnets