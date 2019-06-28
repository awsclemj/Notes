## Lambda Basics

### What is Lambda?
* **Serverless** computing platform
	* Run code without provisioning infrastructure
	* Only pay for compute  (not idle time)
	* Highly-available, fault-tolerant, scalable, elastic
* Lambda Functions are the core concept. These consist of:
	* Your code
	* Code dependencies (libraries, other resources)
* Integrates with other AWS services:
	* CloudWatch Logs
	* API Gateway (RESTful endpoints backed by Lambda)
	* KMS for managing encryption keys
	* Other services through use of AWS SDKs

### Supported Languages
* Java
* Go
* Powershell
* Node
* C#
* Python
* Ruby
* Runtime API allows use of other languages

### Lambda Triggers
* API requests through API Gateway
* CloudWatch events
* S3 file uploads
* DynamoDB Streams changes
* Invocations using the CLI/SDKs

### Programming model
* **Handler** files and functions are the entry point for Lambda invocations
* **Events** - incoming data passed to a function when triggered
* **Context** - methods and properties that provide information about the invocations, function, and environment
* **Logging** - handled via CloudWatch Logs
* **Exceptions** - success or failure is communicated to AWS