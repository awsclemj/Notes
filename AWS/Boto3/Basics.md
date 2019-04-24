## Boto3 - Basics

### What is Boto3?
* AWS SDK for Python
	* Provides easy-to-use object-oriented API as well as low-level access to AWS resources
* Built atop a library called Botocore, which AWS CLI uses
	* Provides low-level clients, session, and credential configuration
* Boto3 builds atop Botocore with its own session, resources, and collection

### Boto3 Clients
* Exposes Botocore clients to devs
* Generated from JSON service descriptions
* Low-level service access
* Maps 1:1 with service API
* Method names snake-cased
	* `ListBuckets -> list_buckets`

### Boto3 Resources
* Generated from JSON resource descriptions
* Object-oriented API
* Identifiers and attributes
* Actions
* References
* Sub-resources
* Collections

### Definitions
* **Session** - manages state about a configuration. Created when needed, but sometimes recommended to manage yourself
	* Credentials
	* Region
	* Other configurations
* **Configuration** - Credentials and non-credential values. Boto3 uses these as sources:
	* Explicitly passed as config parameter when creating a client
	* Environment variables
	* The ~/.aws/config file
* **Resources** - an object-oriented interface to AWS. High-level abstraction to low-level calls made by clients
	* Example:
```python
	import boto3
	
	sqs = boto3.resource('sqs')
	s3 = boto3.resource('s3')
```
* **Clients** - low-level interface to AWS services. All service operations are supported by clients
	* Example:
```python
	import boto3
	
	sqs = boto3.client('sqs')
```


### More on Botocore
* Provides low-level interface for:
	* Providing access to all available services
	* All operations within a service
	* **Serializes** parameters for a particular operation
	* Signs the request with correct authentication signature
	* Receives response and returns data in native Python data structures (**deserialization**)
* Goal is not to provide high-level abstrations. That is left to the application layer
* Simply handles all lower-level details of making requests and receiving responses

### Code Examples
* Resource:
```python
import boto3

s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
	print(bucket) #Outputs all buckets in my configured account
```
* Client:
```python
import boto3

ec2 = boto3.client('ec2')
response = ec2.run_instances(
	ImageId='ami-b70554c8',
	InstanceType='t2.micro',
	KeyName='VPC Key',
	MinCount=1,
	MaxCount=1
)

response #Returns JSON-formatted output of response
```
