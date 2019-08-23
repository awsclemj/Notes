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