- [Migrations](#migrations)
  - [Strategies](#strategies)
  - [Cloud Adoption Framework](#cloud-adoption-framework)
  - [Hybrid Architecture](#hybrid-architecture)
    - [Storage Gateway](#storage-gateway)
    - [Enterprise Resource Planning](#enterprise-resource-planning)
    - [VMWare vCenter](#vmware-vcenter)
  - [Migration Tools](#migration-tools)
    - [Storage Migrations](#storage-migrations)
    - [Server Migration Service](#server-migration-service)
    - [Database Migration Service](#database-migration-service)
    - [Application Discovery Service](#application-discovery-service)
    - [AWS Migration Hub](#aws-migration-hub)
  - [Network Migrations and Cutovers](#network-migrations-and-cutovers)
  - [Snow Family](#snow-family)

# Migrations
Notes taken from Linux Academy/ACG SA Pro 2020 course.

## Strategies
* **Re-Host** - aka lift and shift. Move assets to cloud with little to no changes.
* **Re-Platform** - aka lift and reshape. Move assets but change underlying platform. 
* **Re-Purchase** - aka drop and shop. Abandon existing architecture and purchase new. 
* **Rearchitect** - redesign app to be cloud native. 
* **Retire** - drop existing applications if not needed.
* **Retain** - keep on-prem architecture as-is.

## Cloud Adoption Framework
* Enterprise architectures typically implement **TOGAF** - The Open Group Architectural Framework 
  * Not all architectures are TOGAF, but fills a vacuum where other standards don't exist. 
  * Keep in mind this is a framework, not a cookbook. Things can change depending on org. 
* Useful tool for architecting in the cloud - **Cloud Adoption Framework (CAF)**
  * Both technical and business components:
    * **Business** - Business case for cloud adoption
    * **People** - Organizational roles, skills, knowledge gaps, training options
    * **Governance** - portfolio management, program and project management, align KPIs with new business capabilities
    * **Platform** - standardization of resource provisioning, leverage cloud-native, new app dev skills, processes, and agility
    * **Security** - IAM, logging/auditing capabilities, shared responsibility model
    * **Operations** - monitoring cloud assets, performance management (scaling), business continuity/DR

## Hybrid Architecture
* At a high level, this means that a workload takes advantage of both cloud and on=prem resources
* Common first step to migrations
* Ideally, hybrid integrations are loosely-coupled 

### Storage Gateway
* Bridge between on-prem and AWS
* Service is seamless for end-users 
* Common first step; low risk and appealing economics

### Enterprise Resource Planning
* ERP software uses some middleware to push data update messages to SQS
* Worker instance polls SQS and updates a Dynamo table with data from middleware
* Example of loosely-coupled arch; ERP doesn't need to know about existence of cloud resources

### VMWare vCenter
* Plugin allows transparent migration of VMs to/from cloud
* Additional cloud-native features are available

## Migration Tools

### Storage Migrations
* Storage Gateway, Snowball, DataSync

### Server Migration Service
* Automates migration of on-prem VMware vSphere or Microsoft Hyper-V/SCVMM virtual machines
* Replicates VMs to AWS; sync volumes and create periodic AMIs
* Minimizes cutover downtime; syncs VMs incrementally
* Only supports Windows and Linux
* Server Migration Connector downloaded as a virtual appliance for on-prem vSphere or Hyper-V setup

### Database Migration Service
* Can be used along with Schema Conversion Tool (SCT) the help customers migrate to RDS or an EC2-based DB
* SCT can copy database schemas for homogenous migrations (same DB) and convert for heterogenous migrations (different DBs)
* DMS used for smaller, more simple conversions and also supports Mongo/Dynamo
* SCT is used for larger, more complex migrations like data warehouses
* DMS has replication function; on-prem to AWS, or to S3 or Snowball

### Application Discovery Service
* Gathers information about workloads running on-prem; collect config, usage, and behavior data from servers
* Can help estimate TCO for running workloads in AWS
* Run agentless (VMware) or with an agent (non-VMware)
* Supports Linux and Windows 

### AWS Migration Hub
* Each of the above services roll up to Migration Hub
* Centralized view of migration tools and statuses

## Network Migrations and Cutovers
* Ensure no IP ranges used overlap between VPC and on-prem
* VPCs support netmasks between /16 and /28
* 5 IPs are reserved per subnet 
* Most orgs start with a VPN connection; as usage grows, DX may be considered as primary link
  * Transition between VPN and DX is mostly seamless when BGP is used

## Snow Family
* Evolution of AWS Import/Export; move massive amounts of data to/from AWS
* Data transfer is as fast or as slow as you're willing to pay for a carrier (UPS, FedEx, etc.)
* Data is encrypted at rest
* Products:
  * **Import/Export** - ship a hard drive to AWS and data will be imported to S3
  * **Snowball** - Ruggedized NAS that is shipped to you by AWS. Copy over up to 80TB and send it back to AWS.
  * **Snowball Edge** - Same as Snowball, but with edge compute/onboard Lambda 
  * **Snowmobile** - shipping container full of storage. Fill with up to 100PB and send back to AWS