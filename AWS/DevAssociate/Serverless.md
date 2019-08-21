## AWS Serverless Compute Services
* All notes taken from Linux Academy Dev Associate course.

### Lambda Essentials
* Runs code without you having to manage server infra
* Upload your code and AWS handles everything else to run and scale your functions
* Only pay for the time you consume (in millisenconds)
* Integrates with many other AWS services:
    * CloudWatch logs (monitoring)
    * API Gateway (REST endpoints backed by Lambda)
    * KMS (encryption keys)
    * Many more via AWS SDKs and IAM role assigned to function
* Use cases:
    * Event-driven workloads
    * Stream-processing and ETL processes
    * IOT/mobile backends
    * Web app APIs

### Functions and Events
* Event Sources:
    * A web/mobile app may use API Gateway, which would then send an event to Lambda in the backend
    * AWS services such as S3, DynamoDB, and CloudWatch Events can also trigger events to send to Lambda
* Lambda Events:
    * Lambda Functions are triggered by *events*. These events can be a variety of things:
        * API requests through API Gateway
        * CloudWatch Events
        * S3 file uploads
        * DynamoDB Streams changes
        * Direct invocation
        * Others

* Lambda Functions:
    * Consist of application code, dependencies, and anything else your code requires such as libraries or custom configuration

**Programming Model:**
All Lambda runtimes deal with similar concepts, including:
* Handler files and functions - entry point for invocations
* Events - incoming data passed to Lambda function when triggered
* Context - a way to get runtime information (e.g. time remaining)
* Logging/exceptions - always handled through CloudWatch Logs and functions communicate success/failure to AWS
* Runtime-specific concepts - each runtime has a few differences and have unique concepts such as Node.js callback (return info to caller)

**Invocation Types:**
* Synchronous - wait for the return value and return it
* Asynchronous - don't wait for the return value; discard it

### Function Configuration
* Lambda requires significantly less configuration and management than traditional server infra, but there are some configurable options:
    * Language runtime
    * Handler file and function (filename.functionname)
    * Memory
        * 128 MB - 3008 MB
        * CPU allocation scales with memory
    * Execution duration
        * Max 900 seconds (1 second increments)
    * Permissions
        * IAM roles
        * Resource-based access control policies allow other services and external accounts to invoke/take action on functions
    * Environment variables
        * Key-value pairs available at runtime for all functions
        * Can be encrypted with KMS
    * VPC networking - grant function access to VPC resources
    * Dead Letter Queues - configure SQS or SNS to accept data from failed function executions
    * Concurrency - Maximum number of concurrent invocations of your function
    * Tags - key-value pairs used to help organize functions

### Function Packages
* Function packages, aka deployment packages, are zip packages of all the code and dependencies required by your Lambda function at runtime
* Include:
    * Handler file
    * Custom libraries/packages
    * Other application code that integrates with handler
    * Third-party packages from providers like npm/pip
        * Python - pip install libraries in the same directory as your function code
        * Node.js - include node_modules folder with npm dependencies

### Versions and Aliases
* Versions
    * Distinct versions of functions stored inside of AWS each with unique ARNs
    * Versions are either the $LATEST version (mutable) or a numbered version (immutable)
        * Allows you to manage functions for different environments (like dev, test, prod, etc.)
* Aliases
    * Act as a pointer to a specific Lambda version
    * You can invoke a function with the alias without having to know which version is being referenced
    * Static ARNs that can point to any version of a function
    * Can be used to split traffic between versions
* Benefits
    * Easier development workflow and management of stages
    * Avoid having to reconfigure event sources
    * Rolling back to an earlier version becomes as easy as updating the alias
    * Traffic splitting between versions can help test new versions in prod (blue/green deployments)

### API Actions
* AddPermission - Adds a permission to a resource policy associated with the specified Lambda function
* CreateFunction - Create a new Lambda function
    * Metadata is created from request parameters, code is provided in a .zip file in the request body
* Invoke - Invokes a specified Lambda function. Can also specify invocation type (synchronous/asynchronous)
* CreateEventSourceMapping - Identifies a stream as an event source for a Lambda function (DynamoDB or Kinesis stream)
    * The association between a stream source and Lambda function is called the event source mapping