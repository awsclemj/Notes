## CloudFormation Fundamentals

### Terminology
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
