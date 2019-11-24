- [Databases in AWS](#databases-in-aws)
  - [DynamoDB Overview](#dynamodb-overview)
    - [Pricing](#pricing)
    - [Core Concepts](#core-concepts)
    - [Provisioned Throughput](#provisioned-throughput)
    - [Local and Global Secondary Indexes](#local-and-global-secondary-indexes)
    - [Common Errors and Limits](#common-errors-and-limits)
    - [DAX](#dax)
    - [DDB Throttling](#ddb-throttling)
    - [DDB Streams and Lambda](#ddb-streams-and-lambda)
    - [DDB Auto Scaling and On-Demand](#ddb-auto-scaling-and-on-demand)
  - [Other Database Products](#other-database-products)
    - [RDS](#rds)
    - [Redshift](#redshift)
    - [Elasticache](#elasticache)

## Databases in AWS
All notes taken from the DevAssociate course in Linux Academy

### DynamoDB Overview
* Managed NoSQL database; AWS scales compute for you. You only have to manage your data. 
    * Built-in monitoring
    * Schemaless key-value store
* Consistent and fast performance facilitated by SSDs
    * Control your performance with provisioned read/write capacity
    * Your data is replicated across multiple AZs; **DynamoDB Global Tables** allows replication across AWS regions
    * Optional encryption at rest
* Communicate via AWS Console, CLI, SDKs. Also easily integrates with other AWS services
* **DynamoDB Streams** - Allows for other services (such as Lambda) to take action based on changes to a table

#### Pricing
* Core Components:
    * Provisioned Throughput for Writes - monthly rate per Write Capacity Unit (WCU)
    * Provisioned Throughput for Reads - monthly rate for RCU
    * Indexed data storage - hourly rate per GB of table data
* Other factors:
    * Reserved capacity - pay upfront for minimum number of RCU/WCU at reduced cost
    * DynamoDB Auto Scaling - free; simply pay for the scaled read/write capacity
    * Global Tables - Replicated Write Capacity Units (rWCUs)
    * On-Demand Backup - pay per GB-month stored
    * On-Demand Restore - pay per GB restored
    * DynamoDB Accelerator (DAX) - pay for instances used per hour
    * DDB Streams - per 100k reads

#### Core Concepts
* **Partition Key** - required; also known as a ***hash attribute***. Must be scalar (only hold one value)
    * DDB uses partition keys in an internal hash function to determine where the partition data is stored
* **Sort Key** - optional; also known as a ***range attribute***. Must be scalar (only hold one value)
    * DDB uses this after determining in which partition data will be stored to sort data inside the partition
* **Primary Key** - uniquely identifies each item; no two items in the table can have the same key
    * Two types:
        * Simple - made up of only the partition key. All items must have a unique partition key
        * Composite - made up on a partition and sort key. Items must have a unique combination of partition and sort key. Items can have the same partition key OR sort key.
    * Primary key attributes must have a type of string, number, or binary
* **Items** - groups of attributes that are uniquely identifiable among other items in a table. Similar to rows in a relational DB
    * Example:
    ```json
    {
        "Artist": "Anthony James",
        "SongTitle": "Learning for Days",
        "AlbumTitle": "A Few of My Favorite Things",
        "Genre": "Punk Rock",
        "Price": "1.99",
        "CriticRating" : "8.1"
    }
    ```
* **Attributes** - fundamental data elements of DDB. Similar to fields and columns in other DBs.
    * Data Types:
        * Scalar - String, number, binary, boolean, null
        * Document - List, map
        * Set - Number sets, string sets, binary sets
    * Examples:
    * `MyScalarStringAttribute: "A String Value"`
    * `MyScalarNumberAttribute: 71488`
    ```
    MyDocumentAttribute: {
        "Street": "1500 Grand Avenue",
        "City": "Seattle",
        "PostalCode": "98134"
    }
    ```
* **Partitions** - SSD allocations of storage for the table; automatically replicated across AZs. Number of partitions is determined by the table's provisioned capacity and size of existing partitions

#### Provisioned Throughput
* Maximum amount of capacity an app can consume from a table or index; when the throughput is reached, requests get throttled
* You can use **DynamoDB Auto Scaling** to avoid throttling. This increases and decreases capacity as needed
* Set at the table level, but split and consumed at the partition level. This can cause certain partitions to lack necessary capacity while others are fine
* WCUs and RCUs are the number of reads and writes **per-second**
* Remember to **round up** to the nearest 1 KB (WCUs) or 4 KB (RCUs) when appropriate
* **Writes**:
    * WCUs are consumed when creating, updating, or deleting items. 1 WCU = 1 write of 1 items of 1 KB or less per second
    * Multiple items require multiple WCUs
    * Example question: A Table that needs capacity to write 120 items of 2.5 KB each per minute requires how many WCUs?
        * 120 items/min * 3 KB each (round up to nearest KB)
        * 120 items/60 sec * 3 KB each
        * 2 items/sec * 3 KB
        * 1 write * 2 items * 3 KB = 6 WCUs
    * **Atomic Counters** - Writes are applied in the order they are received; can be used to update existing values like count of website visitors.
        * This does pose the risk of updating the items twice, possibly under or over counting
    * **Conditional Writes** - These writes will only succeed if the item attributes they are writing to meet one or more conditions
* **Reads**:
    * Consumed when using `GetItem`, `Query`, and `Scan` operations. 1 RCU = 1 read of 1 item 4 KB or less per second (**strongly consistent reads**)
    * Always round up the size of single-item reads to the nearest 4 KB
    * **Eventually consistent reads** require half the RCUs (0.5 RCUs for 1 read of 1 item 4 KB or less)
    * Example: A table that needs capacity too read 10 items of 13KB each per second with strongly consistent reads requires how many RCUs?
        * 10 items * 13 KB each
        * 10 items * (16 KB/4 KB per RCU)
        * 10 items * 4 RCUs/item = 40 RCUs for strongly consistent reads
    * API Actions:
        * `GetItem` - reads a single item by providing the item's primary key
        * `BatchGetItem` - reads up to 100 items from one or more tables. 1 read per item.
        * `Query` - read item(s) with same partition key value; you can use the sort key to sort items. Treated as a single read operation
        * `Scan` - read all items in a table 
        * There is a limit of 1 MB of data returned from all of these operations. You can get more data if necessary by providing `LastEvaluatedKey`

#### Local and Global Secondary Indexes
* Allow efficient queries of non-primary key attributes; every secondary index is associated with only one table
* Tables can have multiple secondary indexes (up to 20 global)
* **Local**:
    * Gives choice of sort keys for tables; LSI primary keys are composite keys
    * Each index acts as another sort key and are automatically kept in sync with the base table
    * Read/write capacity taken from capacity of base table
    * Must be created at the same time a table is created
* **Global**:
    * Acts as another way to query information using a different partition and sort key
    * Like LSIs, automatically kept in sync with the main table
    * Cannot perform strongly consistent reads
    * Consume read/write capacity independently of the main table and have their own WCU/RCU provisioned throughput
    * Can be created after a table is created

#### Common Errors and Limits
* ThrottlingException - you are trying to delete, create, or update tables too quickly
* ProvisionedThroughputExceededException - The throughput of the table is insufficient to support the amount of read/write ops
* ResourceNotFoundException - The requested table does not exist or is too early in the CREATING state
* Errors may also be thrown because of limits:
    * Secondary Indexes
    * Provisioned Throughput
    * Number of DDB tables
    * Items and attributes
    * API limits

#### DAX
* DynamoDB Accelerator - in-memory caching service for DynamoDB
* Response times with DDB are typically calculated in milliseconds, but there are certain use cases where microseconds may be required.
* DAX delivers fast response times for accessing eventually consistent data
* Use cases:
    * Applications require fastest possible read times
    * Read a small number of items more frequently thank others
    * Read-intensive, but cost-sensitive
    * Repeated reads against a large set of data
* DAX runs as a cluster (primary + read replicas) in a VPC. Your app must have a client installed to communicate with the cluster
* Item cache - cache items or batch item requests in the DAX cluster
* Query cache - stores the result of query and scan operations

#### DDB Throttling
* Main causes:
    * Partitions in the table which receive heavy amount of requests (hot partitions)
    * Capacity limitations
* What happens when there is too much throttling?
    * Loss of data if the app doesn't retry writes
    * Slow processing if writes are retried excessively
    * Data can become outdated if writes are throttled and reads are not
* Fixes:
    * Increase provisioned capacity
    * Review capacity of global secondary indexes
    * Implement error retries and exponential backoff (logic is built-in to AWS SDK)
    * Distribute reads/writes as evenly as possible across the table. Hot partitions degrade performance
    * Caching solutions such as DAX or Elasticache
    * Enable TTL for unused items in table

#### DDB Streams and Lambda
* Example basic workflow:
    * Stream records are written to reflect table changes
    * The record triggers a Lambda function
    * The Lambda function reads the stream record and sends the record to CW Logs
* Other details:
    * Captures a time-ordered sequence of item-level modifications and stores this information for up to 24 hours
    * Apps can access this log and view the data items as seen before/after modification
    * When enabling a stream, DDB captures information about every modification to items in the table
    * Near real-time aspect enables applications to take action based on the contents
    * Separate endpoints for DDB and DDB Streams

#### DDB Auto Scaling and On-Demand
* Auto Scaling:
    * Create an Application Auto Scaling policy for DDB table; DDB publishes consumed capacity metrics to CW
    * If consumed capacity exceeds target util. for a specified length of time, CW triggers am alarm and invokes a scaling operation
    * Application Auto Scaling issues an UpdateTable request to adjust your table's provisioned throughput (increase/decrease based on alarm)
    * Due to latency in CW Alarms, it's possible that Auto Scaling will not update your capacity exactly at the time it's needed
* On-demand:
    * No capacity planning is necessary
    * Pay-per request pricing; only pay for what you use
    * Automatically ramps up/down as needed
    * Useful for unpredictable workloads
    * Consider performance vs cost; you may be able to optimize cost with provisioned capacity

### Other Database Products

#### RDS
* Fully managed relational DB; cannot access underlying OS
* Connect to it the same way you would a traditional on-prem DB (ex. MySQL command line)
* Provision/resize hardware on-demand, create multi-AZ deployments, and use read replicas
* Supported engines:
    * MySQL
    * MariaDB
    * PostgreSQL
    * MS SQL
    * Oracle
    * Aurora
        * Forked from and fully compatible with MySQL; fixe time better perf and lower price point than commercial DBs

#### Redshift
* Fully-managed petabyte-scale data warehousing service
* Generally used for big data analytics; can integrate with most popular BI tools:
    * Jaspersoft
    * Microstrategy
    * Pentaho
    * Tableau
    * Business Objects
    * Cognos

#### Elasticache
* Fully-managed in-memory cache engine used to improve DB performance by caching results of queries made to the DB
* Great for large, high-taxing queries
* Great for managing web sessions or caching dynamically-generated data
* Engines include Redis and Memcached
* Popular engines such as MySQL have plugins that support Memcached and easily work with Elasticache
* **Caching Strategies**
    * Lazy loading
        * Write data to cache when cache miss occurs; avoids filling up cache with data that won't be requested
        * Data can become stale if lazy loading is implemeted without other strategies
    * Write through
        * Cache is updated whenever a new write or update is made to the underlying DB
        * Cache data remains up-to-date, but can add wait time to write operating in your app
        * If no TTL is applied, this can result in cached data that is never read
    * Adding TTL
        * Drawbacks of the other two techniques can be helped by TTL
        * When reading an expired key, the app checks the value in the underlying DB
        * TTLs do not guarantee fresh information, but keep data from getting stale