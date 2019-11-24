- [Server-based Compute](#server-based-compute)
  - [EC2 Fundamentals](#ec2-fundamentals)
    - [Amazon Machine Images (AMIs)](#amazon-machine-images-amis)
    - [Purchasing Options](#purchasing-options)
  - [EC2 Instance Configuration](#ec2-instance-configuration)
    - [Bootstrapping and User-data/Metadata](#bootstrapping-and-user-datametadata)
    - [EC2 Key Pairs](#ec2-key-pairs)
  - [Storage Basics](#storage-basics)
    - [Elastic Block Store (EBS)](#elastic-block-store-ebs)
      - [Snapshots](#snapshots)
    - [Instance Store](#instance-store)
    - [Elastic File System (EFS)](#elastic-file-system-efs)
  - [Elastic Load Balancers](#elastic-load-balancers)
    - [Maintaing Session State](#maintaing-session-state)
  - [Elastic Container Service](#elastic-container-service)
    - [Why Use ECS?](#why-use-ecs)
    - [Working with ECS](#working-with-ecs)
    - [Docker Essentials](#docker-essentials)
    - [IAM Roles for ECS Tasks](#iam-roles-for-ecs-tasks)
  - [Elastic Beanstalk](#elastic-beanstalk)

## Server-based Compute
* Notes taken from the AWS Developer Associate course on Linux Academy

### EC2 Fundamentals
* Scalable virtual servers in the cloud
* Can be made up of several different instance types and sizes
* Most commonly run Linux or Windows operating systems
* Designed to mimic on-premise servers, but can be commissioned or decommissioned on demand
* Comprised of the following:
    * **Amazon Machine Image (AMI)** - operating system and other settings
    * **Instance type** - Hardware compute power, RAM, network bandwidth, etc.
    * **Network interface** - public, private, or elastic IP addresses and other settings
    * **Storage** - storage drive for the instance:
        * Elastic Block Store (EBS) - network persistent storage
        * Elastic File System (EFS) - scalable, elastic file storage
        * Instance store - epehmeral storage
* Other important facts:
    * A security group must be assigned to the instance during the creation process
    * Each instance must be placed into an existing VPC subnet
    * Bootstrapping can be passed to the instance during launch via "user data" scripts
    * Tags can be used to help name and organize instances
    * Keypairs are used to manage login authentication
    * There are account-level limits to the amount of instances you can have running in a region at any time

#### Amazon Machine Images (AMIs)
* Preconfigured package required to launch an EC2 instance. Includes an OS, software packages, and other settings
* AMIs are used with Auto Scaling to quickly launch new servers on demand, and quickly recover from disaster
* You can create your own AMIs, choose from a list of AWS/community AMIs, or choose one from the marketplace
* Three main categories:
    * Community - free to use, generally you get to select the OS you want
    * Marketplace - pay to use, comes packaged with additional licensed software
    * My AMIs - AMIs that you have created yourself

#### Purchasing Options
* On-demand - lets you choose instance type and provision/terminate at any time
    * Most expensive
    * Most flexible
    * Only charged while instance is running
* Reserved Instances (RIs) - Allows you to purchase an instance for a set period of time of one to threee years
    * Significant price discount over on-demand
    * Pay upfront, partial upfront, or none upfront
    * Responsible for the entire price regardless of how often it's used
    * AZ-specific RIs provide capacity reservation, regional reservations do not
* Spot - "Bid" on an instance type, only pay for the instance when it is at or below your bid price
    * Amazon sells the use of unused instances for short amounts of time at a substantial discount
    * Spot prices fluctuate based on supply and demand
    * Charges per second (with some conditions)
    * An instance is provisioned for you when the spot price is equal to or less than your bid price
    * Automatically terminate when spot price is greater than your bid price
    * Generally used for non-prod applications
* Dedicated hosts - dedicated physical machine that you have full control over. Save on license fees/regulatory compliance

### EC2 Instance Configuration
* EC2 IP addresses:
    * Private - all EC2 instances are automatically provisioned with a private IP. Used for internal communication
    * Public - have the option to enable or auto-assign a public IP for the instance to have direct Internet connectivity. Auto-assign's default is based on the subnet's setting
    * Elastic - Static public IPv4 address; can be used to mask failures or persist the public IP after stop/starts
* IAM Roles - attach permissions to an instance to allow it to take actions/have access to your other services
* Shutdown behavior - can choose whether stopping an instance only temporarily stops it or permanently terminates it
* Prevent accidental termination enable/disable
* Detailed CloudWatch Monitoring
* Tenancy - what kind of hardware we're running the instance on
* T2 Unlimited - allows your T2 to burst past the typical threshold
* User data - Can be used to bootstrap your instance on startup

#### Bootstrapping and User-data/Metadata
* Bootstrapping - a self-starting process or set of commands without external input. We can bootstrap the instance during creation process with custom commands (software packages, updates, configs)
* User-data - Step during instance launch to include your own commands via a script
* Metadata - you can view instance metadata and user-data using the following commands:
`curl http://169.254.169.254/latest/user-data/`
`curl http://169.254.169.254/latest/meta-data/`

#### EC2 Key Pairs
* Two cryptographically secure keys that are used by AWS for login authentication of a client (public/private key pair)
* Public key is stored on the instance; you are responsible for storing the private key
* Linux instances will not use a password, and instead will use the key pair for logging in
* Windows instances use a key pair to obtain the administrator password for logging in via RDP
* During creation, you are prompted to either create a new key pair or use an existing one
* Ensure to change pemissions on your private key file to make it read-only for your user (chmod 400)

### Storage Basics
* Outlines the various storage options for EC2 instances

#### Elastic Block Store (EBS)
* Persistent, meaning they can live beyond the life of an instance
* Network attached storage, meaning they can be attached/detached to/from various EC2 instances
* Can only be attached to one instance at a time
* Can be backed up into snapshots, which can be restored into a new volume
* Replicated within the AZ
* Usually mounted to the file system at /dev/sda1 or /dev/xvda

* EBS Performance
    * Measure input/output operations in units called IOPS
        * IOPS are input/output ops per second
        * Measured in 256KB chunks
        * Operations greater than 256KB are separated into 256KB chunks
        * Ex. a 512KB operation would be 2 IOPS
    * The type od EBS volume influences the I/O performance
    * Sometimes even provisioned IOPS may not produce the performance you'd expect, in which case an EBS optimized instance is required

* EBS initialization
    * Volumes created from snapshots must be initialized
    * Initializing occurs the first time a storage block on the volume is read - perf impact can be up to 50%
    * You can avoid this in prod by manually reading all blocks

* EBS Types
    * General Purpose SSD
        * Dev/test environments and small DB instances
        * Performance of 3 IOPS/GiB of storage (burstable with baseline performance)
        * Volume size of 1GiB to 16TiB
    * Provisioned IOPS SSD
        * Mission critical applications and large DB workloadfs
        * Valume size 4GiB to 16TiB
        * Performs at provisioned level and can provision up to 32k IOPS per volume
    * Throughput Optimized and Cold HDD
        * Cheaper than SSD options, but less performant
        * Cold HDD is designed for infrequent access
        * Volume size of 500GiB - 16 TiB
        * Cannot be a boot volume
    * EBS Magnetic (Previous gen)
        * Low storage cost
        * Used for infrequently accessed workloads
        * Volume size 1GiB - 1TiB

##### Snapshots
* Point-in-time backups of EBS volumes stored in S3
* Incremental in nature; only stores changes made since the most recent snapshot
* If the original snapshot is deleted, all data is still available in other snapshots
* Can be used to create fully restored EBS volumes
* Some important notes:
    * Frequent snapshots increases data durability
    * Taking snapshots can degrade performance, so snapshots should occur during non-peak hours

#### Instance Store
* Virtual devices whose underlying hardware is directly attached to the host running the instance
* ***Ephemeral data***, meaning the volumes only exist during the life of the instance
* If the instance is stopped or shutdown, the data is erased. The instance can be rebooted without data loss
* Not all instance types can use instance store

#### Elastic File System (EFS)
* Fully managed storage option for EC2 that allows for scalable, elastic storage
* Capcity will increase or decrease as files are added and removed
* Applications running on EC2 using EFS will always have the storage they need without having to provision and attach larger devices
* Supports NFS version 4.0 and 4.1 protocols for mounting
* Best performance when using Linux kernel 4.0 or newer
* Other benefits:
    * Can be accessed by one or more EC2 instances at the same time
    * Can be mounted to on-prem servers
    * Can scale to petabytes in size while maintening low latency and high throughput
    * Only pay for what you're using
* Security:
    * Control file system access through POSIX permissions
    * VPC for network access control and IAM for API access control
    * Encrypt data at rest with KMS
* When to use:
    * Big data/analytics
    * Media processing
    * Web serving and content management

### Elastic Load Balancers
* EC2 service that automates process of evenly distributing traffic to all instances associated with the ELB
* Can load balance traffic to multiple EC2 instances across multiple AZs
* Help reduce compute power via SSL/TLS offloading

#### Maintaing Session State
* ELB Option 1 - load balancer generated cookie stickiness
    * LB issues cookie if the user doesn't have one
    * Sends users to specific instance based on the cookie
* ELB Option 2 - Application generated cookie stickiness
    * LB uses application-generated cookie to associate a session with an instance
    * Sends users to specific instance based on the application-generated cookie
* Non-ELB Option (Recommended) - use caching service such as ElastiCache
    * ELB requests are evenly distributed to instances
    * Instances check ElastiCache for the session
    * If no session exists, the isntance can check a backing database

### Elastic Container Service
* Docker-compatible container management service provided by AWS. Allows for easy container deployment on fleets of EC2 instances. Highly available/fault tolerant.
* **Fargate** allows you to use containers while AWS manages EC2 instances for you

#### Why Use ECS?
* Create distributed apps and microservices architectures
    * Architectures comprised of independent tasks/processes
    * Can have separate containers for various components of your application:
        * Web server
        * App server
        * Message queue
        * Backend servers
    * Allows you to start, stop, manage, monitor, and scale each container independently
* Batch and ETL jobs:
    * Package batch and ETL jobs into containers and deploy them into shared EC2 cluster(s)
    * Run different versions of the same job or multiple jobs on the same cluster
    * Share cluster capacity with other processes and/or grow jobs dynamically on-demand
* CI/CD:
    * By using Docker's image versioning, you can use containers for continuous integration and deployment
    * Build processes can pull, build, and create Docker images for your containers

#### Working with ECS
* Dockerfile - A plain text file (script) that specifies all the components that are included in the container
* Docker image - Built from the Dockerfile. Contains all the downloaded software, code, runtime, system tools, and libraries as specified in the Dockerfile script
* Container registry - repo where container images are stored and accessed when needed
    * Can be located on AWS via Elastic Container Registry (ECR), 3rd party repo like Docker Hub, or a self-hosted registry
* Task definition - JSON-formatted text file that contains "blueprint" for the application, including:
    * Which container image to use
    * The repo the image is located in
    * Which ports should be open on the container instance
    * What data volumes should be used with the containers
* Task - actual representation of the Task Definition on the EC2 instances inside of the container cluster
    * ECS Agent will start/stop these tasks based on instruction/schedule
* ECS Agent - runs on the EC2 instance in the ECS cluster. Communicates information such as running tasks and resource util. Also starts/stops tasks.

#### Docker Essentials
* Software platform that allows you to build, test, and deploy applications quickly. Packages software into standardized units called containers that have everything they software needs to run:
    * Libraries
    * System tools
    * Code
    * Runtime
* Docker lets you ship code faster, standardize application ops, seamlessly move code, and save money by improving resource util. 
* Docker allows you to decouple applications into multiple containers and all containers are highly portable between difference OSes

#### IAM Roles for ECS Tasks
* ECS tasks can assume IAM roles to perform actions on your AWS resources
* Benefits:
    * Tasks can only perform actions based on the policy attached to the role
    * You can isolate permissions based on task
    * Good for auditing purposes
* Set IAM role when creating a Task Definition

### Elastic Beanstalk
* Service that allows you to easily scale/deploy your web application
* Supports a variety of programming languages and technologies like Docker, Apache, and Nginx
* Designed for less complex applications; reduces the management required for building/deploying apps
* Builds single-tier applications that take advantage of services such as:
    * EC2
    * Auto Scaling
    * ELB
    * RDS
    * SQS
    * CloudFront
* Why use Beanstalk?
    * Quickly provision an AWS environment that requires little management
    * Application fits within parameters of the Beanstalk service
    * Deploy from repos or uploaded files
    * Update by uploading new code files or requesting a pull from a repo
* Supported platforms:
    * Docker
    * Packer
    * Java
    * .NET
    * Node.js
    * PHP
    * Python
    * Ruby
    * Go