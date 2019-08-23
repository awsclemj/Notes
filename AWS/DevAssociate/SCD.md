## Storage and Content Delivery
* Notes taken via Linux Academy's DevAssociate course

### S3 Essentials
* **Buckets**
    * Main storage container for objects
    * Contain a grouping of sub namespaces similar to folders
    * Can choose the region the bucket is in for purposes of optimizing latency, costs, and addressing regulatory requirements
    * Naming conventions:
        * Globally unique
        * DNS compliant
        * Minimum 3 characters, max 63
        * All lowercase letters, numbers, periods, and hyphens
        * Periods and hyphens cannot follow each other
        * Cannot be an IP address
    * Limits:
        * 100 buckets per account
        * Ownership cannot be transferred
        * Unlimited objects
* **Objects**
    * Static files and metadata (storage type, encryption settings, permissions) uploaded to a bucket
    * Storage type defines availability, durability, and cost
    * Private by default
    * Can:
        * Be 0 bytes - 5 TB
        * Have multiple versions (if versioning enabled)
        * Be made publicly available
        * Switch between storage classes
        * Encrypted
        * Organized into folders
    * Encryption:
        * SSE (Server-side Encryption) - S3 encrypts the object at rest with AES-256 and decrypts it when downloaded
        * You may also use your own encryption keys and/or encrypt files on the client side before upload
* **Folders**
    * Folders are a means of grouping objects. Amazon does this by using key-name prefixes for objects
    * Keep in mind S3 has a flat structure; no traditional file system hierarchy
* **Read Consistency**
    * All regions support read-after-write consistency for new PUTs
    * All regions use eventual consistency for PUTs that overwrite objects and DELETEs of objects
* **Errors**
    * 404 Note Found
    * 403 Forbidden
    * 400 Bad Request
    * 500 Server Error
    * Other standard HTTP responses
* **Events**
    * Event notifications allow you to set up automated communication between S3 and other AWS services
    * Common notifications:
        * RRSObjectLost (Automates recreation of lost RRS objects)
        * ObjectCreated (for all the following APIs called):
            * PUT
            * POST
            * COPY
            * CompleteMultiPartUpload
        * Can be sent to the following services:
            * SNS
            * Lambda
            * SQS

### Uploading Data to S3
* Can use the AWS Console, CLI, or REST API to upload data to S3
* **Single Operation Upload**
    * Upload the file in one part, up to 5 GB. Any file over 100 MB should use multipart as a best practice
* **Multipart Upload**
    * Upload parts of a single file concurrently
    * Stop/resume uploads
    * If a single part fails, you can retransmit that part
    * S3 reassembles the parts after they're all uploaded
    * Required for files over 5 GB, recommended for files over 100 MB
* **Other services**
    * Storage Gateway:
        * Cached Volume:
            * Creates storage volumes and mount them as iSCSI devices on on-prem servers
            * Gateway stores data written to this volume in S3 and will cache frequently accessed data on-prem
        * Stored Volume:
            * Store all data locally in on-prem volumes
            * Gateway periodically takes snapshots of the data and stores on S3
    * Snowball/Snowmobile
    * Import/Export - mail a device you own to AWS
        * Will import to S3, EBS, or Glacier
        * Up to 16 TB per job

### S3 Performance
* By default, S3 scales to very high request rates
    * 3,500 PUT/LIST/DELETE ops per second
    * 5,500 GET requests per second
* However, it is best to follow best practices to optimize storage when reaching the following (old recommendation):
    * 100 PUT/LIST/DELETE ops per second
    * 300 GET requests per second
* **Mixed Request Workloads**
    * S3 automatically creates and maintains an index of S3 key names in each AWS region; these are stored in multiple partitions depending on the key name
    * Adding a random prefix to the first few characters of a key can help spread load across partitions
    * Random prefixes can be easily generated by taking the MD5 hash of a keyname and prepending the first four characters of the hash to the keyname
* **GET Intensive Workloads**
    * Introduce randomness as discussed previously
    * Use CloudFront to cache content instead of constantly making requests to S3

### S3 Permissions
* Buckets/objects are private by default, but the resource owner can grant access to the resource using S3 resource-based policies or traditional IAM policy
* IAM Policies:
    * Attached to users, groups, or roles, **not S3 buckets**
    * Can't grant access to anonymous users
    * JSON
* Resource-based policies:
    * Bucket policies
        * Policies attached to the S3 bucket (not objects or roles/users)
        * Applies to all objects in a bucket unless object was created by a user outside the bucket owner's account
        * Can grant access to IAM users, anonymous users, or other accounts
        * JSON
    * ACLs
        * Can be used for buckets AND objects
        * Can't deny or grant conditional permissions
        * Use cases:
            * Manage access to objects not owned by the bucket owner
            * Manage permissions at object level/permissions vary by object
            * Allow external accounts to manage policies on objects
            * Allow public read
            * aws-exec-read: allow EC2 to get AMI bundles from S3
        * XML

### S3 Encryption
* Two ways:
    * In transit/client-side
        * In transit via SSL/TLS
        * Using a client-side master key which is never uploaded to AWS (handled by SDK)
        * On upload:
            * S3 client generates data key
            * User provides a customer side master key (CSMK) to the S3 encryption client and S3 client encrypts the data key with the CSMK
            * S3 client encrypts the data using the data key
            * S3 client uploads a material description using object metadata (x-amz-meta-z-amz-key)
        * On download:
            * Client downloads the encrypted object along with metadata
            * Metadata tells the client which CSMK to use to perform decryption; decrypts data key with CSMK
            * Data key decrypts data
        * Using an AWS KMS CMK:
        * On upload:
            * Client sends a request to KMS for a key, KMS returns a plaintext key to encrypt object data and a ciphertext blob to upload to S3 as metadata
            * Plaintext key encrypts object, then client uploads to S3 with ciphertext blob
        * On download:
            * Client downloads the encrypted object and metadata from S3
            * Client sends the ciphertext blob to KMS to get the plaintext
            * Plaintext key is used to decrypt the object
    * At rest
        * S3-Managed Keys (SSE-S3)
            * Each object gets a unique key that is encrypted by a regularly rotated master key
            * Add `x-amz-server-side-encryption` request header to your upload request
            * Bucket policies can require all objects to use SSE by requiring uploads to have the `x-amz-server-side-encryption` header
        * KMS-managed Keys (SSE-KMS)
            * Similar to SSE-S3, but uses KMS-managed keys to encrypt objects
            * More permission control and better auditing of access to data
        * Customer-Provided Keys (SSE-C)
            * You manage the keys, S3 manages encryption/decryption when writing to disk/accessing objects

### Object Versioning
* A feature to manage and store all old/new/deleted versions of an object. By default, versioning is disabled. 
* Set on the bucket level and applies to all objects
* Lifecycle policies can be added to specific versions of an object

#### Enabling Versioning
* Existing objects will remain unchanged and have a null version ID
* New objects are automatically given unique version IDs and each version is billed as a new S3 object. These are not deltas of the previous version. 

#### Deleting Versioned Objects
* When a delete is made to an object (without specifying version ID), all versions of the object remain in the bucket
    * S3 inserts a delete marker at the latest version. If you try to retrieve it, S3 returns a 404 error
    * You can still get a specific version by specifying the version ID
* To permanently delete a version, make a delete request with the version ID to be deleted. 

#### Restoring Versioned Objects
* Delete the current version to fall back to a previous version. Repeat until you have the version you want
* Copy a previous version of the object to the same bucket; this adds a new object to the bucket that becomes the current version

### Storage Classes
* Standard - general purpose storage, 11 9s durability, 4 9s availability, most expensive
* RRS (not recommended) - non-critical objects, 4 9s durability and availability, less expensive than standard
* Standard-IA - for objects not frequently accessed. 11 9s durability, 99.90% availability, less expensive than standard and RRS 
* OneZone-IA - for objects not frequently accessed; only replicated in one AZ 
* Glacier - data archival; can take several hours to be changed/retrieved. 11 9s durability, very cheap
    * Three levels of retrieval:
        * Expedited - 1-5 minutes
        * Standard - 3-5 hours
        * Bulk - 5-12 hours

### Lifecycle Policies
* Set of rules to automate migration of an object's storage class to a different storage class, or deletion, based on specified rules
* Disabled by default
* Can be combined with versioning for archiving/backup solutions. For example, send non-current versions to Glacier after X days, or permanently delete them
* Objects must age for at least 30 days before moving from IA to another storage class

### Static Website Hosting
* Option for low-cost, highly-reliable hosting service for static websites. Includes content like HTML, CSS, and Javascript
* You can specify an index file and error file
* URL format: `bucket-name.s3-website(- or .)region.amazonaws.com`. You can also map Route 53 domain names to static web site buckets
* **Cross-Origin Resource Sharing**:
    * CORS is a method of allowing web applications located in one domain to access resources from another domain
    * For example, a web app hosted in one S3 bucket can access resources in another S3 bucket
    * Example:
    ```xml
    <CORSConfiguration>
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <AllowedHeader>Authorization</AllowedHeader>
        </CORSRule>
    </CORSConfiguration>
    ```

### CloudFront Basics
* Global CDN which delivers content from an origin location. This can be S3, ELB, EC2 instances, or any other custom origin
* Static objects are cached at global edge locations; this allows users to experience lower latencies and reduces load on your resources
* Can integrate with Route 53 for CNAMEs or alias records

#### Updating Cached Files
* Caching is done based off of the object name; in order to save a new version, you can:
    * Create new object with a new name
    * Invalidate the object(s) on the CloudFront distro. Note that invalidations have an associated cost.
    * Cached objects can be set with a specific expiration time, or set not to cache at all

#### Signed URLs
* Allow access to "private content" by creating a temporary URL signed with an x.509 cert and based off of the number of seconds you want it to be accessible