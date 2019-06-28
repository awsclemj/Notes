## CloudFormation Fundamentals
* What is it?
	* Create, manage, and automate deployment and configuration of a collection of AWS resources
	* Orderly, predictable provisioning and updating of resources
	* Infra as code with consistency, repeatability

### Terminology
* Infra as Code
	* Programmable infra
	* High-level language
	* Is not technically infra automation, but can be used to achieve it
	* Superset of server provisioning and config management
	* Has become part of the Software Development Life Cycle (SDLC)
* Templates
	* Text file formatted in JSON or YAML
	* Input into CloudFormation
	* Describes end state of infra
* Stack
	* CloudFormation orchestrates templates into stacks
	* To update the resources within a template, you update the **stack**
* Change Sets
	* Before updating a stack, you generate a **change set**
	* Allows you to see how changes will impact your running resources
	* Very important for live systems
		* For example, renaming an RDS instance will create a new RDS instance and delete the old one

### Stack Lifecycle
* Create
	* Initial creation of the stack
	* In progress/complete/failed (proceed to "Rollback")
* Rollback
	* If stack creation encounters an error
	* Common errors include permissions, limits, or missing parameters
	* In progress/complete/failed
* Update
	* Attempt to implement changes you've introduced. Typically "all or nothing"
	* Common failures include permissions, limits, missing parameters, or failing a dependency check
	* In progress/complete/failed (proceed to "Update Rollback")
* Update Rollback
	* Failed update results in a stack in its formaer deployed config
	* In progress/complete/failed
* Delete
	* When a CFN stack is deleted, it will attempt to remove all resources it was originally tasked with creating
	* In progress/complete/failed

### Template Anatomy
* Basic anatomy of a template in JSON and YAML

JSON:
```json
{
	"AWSTemplateFormatVersion" : "version date",
	"Description" : "JSON string",
	"Metadata" : { template metadata},
	"Parameters" : { set of parameters },
	"Mappings" : { set of mappings },
	"Conditions" : {set of conditions},
	"Transform" : { set of transforms },
	"Resources" : { set of resources },
	"Outputs" : { set of outputs }
}
```
YAML:
```yaml
AWSTemplateFormatVersion : "version date"
Description: String
Metadata:
	template metadata
Paramaters:
	set of parameters
Mappings:
	set of mappings
Conditions:
	set of conditions
Transform:
	set of transforms
Resources:
	set of resources
Outputs:
	set of outputs
```

* Note that the only required section is 'Resources'. All other sections are optional.

Resources section in JSON:
```json
{
	"Resources": {
		"Logical ID" : { 
			"Type" : "Resource type",
			"Properties" : {
				set of properties
			}
		}
	}
}
```

Resources section in YAML:
```yaml
Resources:
 Logical ID: #Reference this ID throughout the local template
  Type: Resource type #This is the actual resource type we will create
  Properties:
   Set of properties #Various properties can be configured for the resource
```

* **Example**: Creating a simple EC2 instance

JSON:
```json
{
	"Resources" : {
		"MyEC2Instance" : {
			"Type" : "AWS::EC2::Instance",
			"Properties" : {
				"ImageId" : "ami-43874721",
				"InstanceType": "t2.micro",
			}
		}
	}
} 
```

YAML:
```yaml
Resources:
 MyEC2Instance:
  Type: "AWS::EC2::Instance"
  Properties:
   ImageId: "ami-43874721"
   InstanceType: t2.micro
```
* **Example**: Creating an S3 bucket for a static website

```yaml
Resources:
 MyS3Bucket:
  Type: AWS::S3::Bucket
  Properties:
   BucketName: HelloWorld
   AccessControl: PublicRead
   WebsiteConfiguration:
    IndexDocument: index.html
```

* **Example**: Creating a simple Security Group

```yaml
Resources:
 MySecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
   GroupDescription: Enable SSH access via port 22
   SecurityGroupIngress:
    - IpProtocol: tcp
      FromPort: '22'
      ToPort: '22'
      CidrIp: 0.0.0.0/0
```

### Nested Templates and Stack Layering
* **Nested templates** call other templates before the completion of the master template. Not recommended as a best practice; instead use **Stack Layering**.
* This design has important implications from a DevOps and InfraCoding perspective. 
* Stack Layering uses stacks in horizontal layers that build on top of one another and are executed synchronously. 

* Why layering?
	* Separation of roles and responsibilites
	* Allows for self-service amongst teams
	* Reusability and composability
	* Overcoming template limitations
* Why not nesting?
	* Unagile antipattern
	* Broad permissions required to create a stack
	* Large blast radius
	* Testing monolithic Infracode is hard
	* Precludes "unit testing" approach

#### Example Stack Layering Order
1. VPC, subnets, NACLs
2. Route 53
3. S3, Logging
4. IAM policies
5. Shared components, instance profiles
6. Application stack

### CloudFormation StackSets
* Allows you to deploy stacks to multiple accounts at once
* Example usage: enable AWS Config, CloudTrail in all your accounts

### Best Practices
* Planning and organizing:
	* Organize stacks by lifecycle and ownership
	* Reuse templates to replicate stacks in multiple environments
	* verify quotas for all resource types
	* Limit nested stacks to reuse common template patterns or for single owner implementation
* Segment components of the stack:
	* Create single purpose independent stacks that can be layered on top of one another
	* Examples:
		* NetworkSecurityStack: VPC, IAM, and security groups
		* ApplicationStack: web, app, and database
* Creating templates:
	* Use AWS-specific parameter types (ex. AWS::EC2::KeyPair::KeyName)
	* Use parameter constraints, like min/max length, alphanumeric characters
	* Use intrinsic functions (Fn::GetAZ) instead of hardcoding or parameterizing AZs
	* Use AWS CloudFormation helper functions, AWS::CloudFormation::Init to deploy applications to EC2
	* Validate templates before using them
	* Limit condition functions that would create differences between environments
* Managing stacks:
	* Manage all stack resources through CloudFormation instead of individually via Console, API/SDK, etc. 
	* Use Stack Policies
	* Use AWS CloudTrail to log AWS CloudFormation calls
	* Use code reviews and revision controls to manage templates
	* Separate stacks based on lifesycles (Infra/Networking, IAM/Security, Application, etc.)
* Avoid:
	* Embedding creds in templates
	* A monolithic template. Use stack layering instead.
	* Hardcoding properties (such as AMI IDs) - use parameters or mappings instead