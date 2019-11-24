- [AWS Application Services](#aws-application-services)
  - [Simple Notification Service (SNS)](#simple-notification-service-sns)
    - [SNS Access Control Policies](#sns-access-control-policies)
    - [Message Data](#message-data)
    - [Mobile Push](#mobile-push)
    - [API Actions](#api-actions)
  - [Simple Queue Service (SQS)](#simple-queue-service-sqs)
    - [Resource-based Access Control Policies](#resource-based-access-control-policies)
    - [More on Polling Types](#more-on-polling-types)
    - [Message Data](#message-data-1)
    - [API Actions](#api-actions-1)
  - [Simple Workflow Service](#simple-workflow-service)
  - [Step Functions](#step-functions)
    - [State Types and Transitions](#state-types-and-transitions)
  - [API Gateway](#api-gateway)
    - [Resources and Methods](#resources-and-methods)
    - [Deployment and Stages](#deployment-and-stages)
    - [Caching and Monitoring](#caching-and-monitoring)

## AWS Application Services
Notes taken from DevAssociate course on Linux Academy

### Simple Notification Service (SNS)
* Pub/sub service for messaging; coordinates sending and delivery of messages to specific endpoints
* Integrated with many AWS services; for example can integrate with CW to send notification to admins when certain events/alarms occur
* Can be used to publish iOS/Andriod app notifications
* Components:
    * Topic - group of subscriptions that you send a message to (limited to 256 alphanumeric characters)
    * Subscription - an endpoint a message is sent to. Includes HTTP, HTTPS, email, email-JSON, SQS, mobile push notifications, Lambda, and SMS
    * Publisher - the entity that triggers the sending of a message (AWS CLI, SDKs, S3, CW Alarms, Lambda)

#### SNS Access Control Policies
* Grant access to your SNS topics to another AWS account
    * Via SNS API `AddPermission` - topic, AWS account ID(s), actions, label
* Grant access to some AWS services to publish to your SNS topic
* You can use IAM polices and access control policies at the same time

#### Message Data
* JSON-formatted key-value pairs; allows developers to parse message data
* POST to HTTPS endpoints with headers to verify message authenticity
* Content depends on the subscriber
* Body:
    * Message - message value specified when the notification is published
    * MessageId - A UUID; same value must be used for retries
    * Signature - Base64 encoded "SHA1withRSA" signature of the message, messageId, subject, type, timestamp, and TopicArn values
    * SignatureVersion - Version of SNS signature used
    * SigningCertURL - URL to cert used to sign the message
    * Subject - optional subject parameter
    * Timestamp - GMT time when notification was published
    * TopicArn - For for the topic this message was published to
    * Type - Type of message
    * UnsubscribeURL - A URL you can use to unsub from the topic
* Attributes:
    * Useful for SQS and mobile push notifications; sent along with the message
    * Needs a name, type, and value

#### Mobile Push
* Send notifications to apps on mobile devices; can appear as message alerts, badge updates, sound alerts
* Different providers allow you to send notifications to different devices:
    * Amazon Device Messaging (ADM)
    * Apple Push Notification Service (APNS)
    * Baidu Cloud Push
    * Google Cloud Messaging for Android
    * Microsoft Push Notification Service for Windows Phone
    * Windows Push Notification Services
* MPNS - SNS requires "device tokens" or "registration IDs" to send messages to devices

#### API Actions
* Common actions:
    * CreateTopic - takes the name of the topic you want to create
    * Publish - sends a message to an SNS topic or sends an SMS text message to a phone number
    * Subscribe - prepares to subscribe an endpoint by sending the endpoint a confirmation message; endpoint owner must then confirm subscription
    * Unsubscribe - deletes a subscription

### Simple Queue Service (SQS)
* Provides highly-available message queues that can be used to send and receive messages between producers and consumers
* This allows for distributed/decoupled application components
* Messages between servers are retrieved through polling
* Types of polling:
    * Long (1-20 seconds) - allows SQS to wait until a message is available in a queue before sending a response; reduces API requests
    * Short - SQS samples a subset of servers and returns messages just from those; will not return all possible messages and increases API usage
* Important facts:
    * Each messages can contain 256KB of data in any format
    * Two types of queues:
        * Standard
        * First in First Out (FIFO)
    * Highly-available, redundant, and PCI compliant
* **Producers**:
    * Any application, service, etc., that produce SQS messages and send them to a queue
    * Ex.: App on an EC2 instance, Lambda, ECS, or even applications external to AWS
* **Queues**:
    * Standard:
        * Support a nearly unlimited number of messages per second along with multiple producers and consumers
        * Guarantee ***at least once*** delivery with ***best effort ordering***
        * Applications will need to tolerate duplicate messages
        * Support 12,000 in-flight messages
    * FIFO:
        * Up to 3,000 messages per second with batching and 300 without batching (these limits can be increased)
        * Support multiple producers but only support multiple producers through use of Group IDs:
            * Used to interleave multuple ordered message groups within a FIFO queue
            * Processed in order within a group but not with respect to other groups
        * Application order of operations is critical because **FIFO queues guarantee messages are sent and processed in order**
        * 20,000 in-flight messages
    * Settings:
        * Dead-letter queues - used to deal with malformed data from consumers
        * DelaySeconds - How long messages take to be added to a queue upon creation
        * VisibilityTimeout - How long messages are invisible after being received by a consumer
* **Consumers**:
    * Any app, service, or other components that process and delete messages from an SQS queue

#### Resource-based Access Control Policies
* Grant access to your SQS queue from other AWS accounts or services
* Use the SQS `AddPermission` API to generate an access control policy to SQS

#### More on Polling Types
* Short Polling:
    * Default; queries a subset of SQS servers to determine if there are available messages
    * Occurs when the ***WaitTimeSeconds*** attribute is set to 0 for either the queue or message itself
    * If a message is assigned a ***WaitTimeSeconds*** greater than 0, it will override the queue's ***WaitTimeSeconds***
* Long Polling:
    * Occurs only when the queue or message have *WaitTimeSeconds*
    * Allows consumers to wait for messages to become available
    * Generally cheaper

#### Message Data
* Text information created by your application that is sent to SQS
* Characteristics:
    * Up to 256 KB of text and can contain up to 10 metadata attributes
    * XML, JSON, or unformatted text
    * Retainable up to 14 data or as little as one minute
    * Can be invisible up to 12 hours
    * Can be added to dead letter queues if consumers are unable to process them
* Components:
    * Body - XML, JSON, or unformatted text body of the message
    * ReceiptHandle - Attribute that allows you to delete the message after processing it
    * MessageAttributes - customer attributes you can set with your SQS message
    * VisibilityTimeout - length of time the queue will wait before making a message visible again after it has been received by a consumer
    * DelaySeconds - length of time the queue will wait before placing the message in the queue

#### API Actions
* CreateQueue - create your first queue
* GetQueueUrl - use this to return the URL you need for future actions
* SendMessage - Use the producer to send one or more messages to the queue
* ReceiveMessage - Receive messages with one or more consumers
* ChangeMessageVisibility - Consumers sometimes need more time that the default
* DeleteMessage - Consumer finishes processing the message and deletes it from the queue

### Simple Workflow Service
* Coordinates/manages execution of activities (a job) from start to finish while allowing for a distributed architecture. 
* A **workflow** allows a developer to implement distributed, asynchronous applications
    * Coordinates/manages **activities** that can be run asynchronously
* Consistent execution, guarantees the order in which tasks are received, no duplicates
* Primarily consists of a API, so on-prem applications can also use it
* Execution can last up to 1 year
* Components:
    * Workflow - aka decider, a sequence of steps required to perform a task
    * Activity - single step of a workflow
    * Task - what interacts with "workers" that are part of the workflow. Two kinds:
        * Activity task - tell a worker to perform a function
        * Decision task - tells the decider the state of the workflow execution so the decider may determine the next activity to be performed
    * Worker - responsible for receiving a task to be performed (such as an EC2 instance or a person)

### Step Functions
* Fully-managed service to coordinate components of your applications through steps
* Visual workflows and tools to coordinate components of your application; you can also use it to orchestrate and visualize application logic in serverless applications
* Components:
    * Tasks - all work in the state machine is completed by this component. Two types:
        * Activity - program code hosted on EC2, ECS, etc., that interacts with the Step Functions API (GetActivityTask, SendTaskSuccess, SendTaskFailure, SendTaskHeartbeat, etc)
        * A Lambda function - serverless functions can respond to your state machine tasks
    * State Machine - Defined using JSON ***Amazon States Language***
* Benefits:
    * Managed service that gives you a visual overview of app logic
    * Avoid coding app logic by defining it in a JSON state machine
    * Reuse components and easily edit workflows 

#### State Types and Transitions
* Step Functions coordinates tasks through state machines. Each state makes decisions based on input, performs actions, and passes output to other states
* States can do different things in your state machine:
    * **Task** states do some work
    * **Choice** states choose between different branches of your state machine
    * **Fail/Succeed** states stop execution with a failure/success
    * **Pass** states pass their input to their output and can inject fixed data
    * **Wait** states delay the process for a certain amount of time
    * **Parallel** states begin parallel executions
* Transitions:
    * When a state machine executes, it uses a StartAt field to select one of the states
    * The next step is defined by the "Next" field which references another state in the state machine
    * Some flow-control states allow choices to determine "Next" fields
    * Transitions continue until they hit a runtime error or a Fail/Succeed state
* Input/Output:
    * Every state's input is JSON. They get inputs from a previous state, or for the first state, at the execution of the state machine
    * Some states modify the input, others do not. Additionally, some states pass the input to a task which processes it, and then the state passes along that output

### API Gateway
* Acts as a "front door" for your app. Allows access to data/logic/functionality from your backend services
* Build RESTful APIs with resources, methods (GET, POST, PUT, etc.), and settings
* Deploy APIs to different stages (dev, beta, prod, etc.). Each stage can have its own throttling, caching, metering, etc.
* Create new API versions by cloning existing ones (version control). Additionally, roll back to previous deployments
* Can integrate with a custom domain name that point to specific APIs or stages
* Create and manage API keys for access/meter usage through CW Logs
* Set throttling rules based on requests per second (throttled requests get HTTP 429 response)
* Sigv4 to sign/authorize API calls
* Other benefits:
    * Cache API responses
    * DDoS protection via CloudFront
    * SDK generation for iOS, Android, JS
    * Supports Swagger
    * Request/response data transformation (ex. JSON in XML out)

#### Resources and Methods
* Resources:
    * Logical entities that are accessed via resource paths (ex. /songs, /images, etc.)
    * URL paths are used to expose resources and also include the deployment stage
        * Example URL: https://example.execute-api.us-east-1.amazonaws.com/**stage**/resource
    * Resources can also have children: https://example.execute-api.us-east-1.amazonaws.com/stage/resource/**child**
    * Resources and child resources have associated HTTP methods
* Methods:
    * HTTP methods associated with an API Gateway. Each resource can have associated methods, like:
        * GET
        * PUT
        * POST
        * DELETE
        * AWS also offers ANY as a catch-all
    * Can be configured to respond in a variety of ways:
        * Lambda functions via Lambda Proxy integration
        * Existing HTTP endpoints
        * Other AWS services

#### Deployment and Stages
* Deployments:
    * Snapshot of the API's resources and methods; must be associated with a stage in order for it to be accessed
* Stages:
    * References to the lifecycle status of the API (ex. dev, beta, prod)
    * Can be mapped to deployments in order to have a functioning API
    * Settings on per-stage basis:
        * Enable caching
        * Customize request throttling
        * Configure logging
        * Define stage variables

#### Caching and Monitoring
* Caching:
    * API Gateway can cache responses so duplicate API requests don't have to hit the backend
    * You can configure cache key TTL and API response
    * Can be set up on a per API or per stage basis
* Monitoring:
    * CW can be used to monitor API activity and usage from the API or stage level
    * Metrics include:
        * Caching
        * Latency
        * Errors
        * Throttling (if enabled)
    * Can create alarms based on these metrics