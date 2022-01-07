- [Configuration Management and Infrastructure as Code](#configuration-management-and-infrastructure-as-code)
  - [CloudFormation](#cloudformation)
    - [What is CloudFormation?](#what-is-cloudformation)
    - [Terms](#terms)
    - [Template Anatomy](#template-anatomy)
    - [Use cases](#use-cases)
    - [Intrinsic Functions](#intrinsic-functions)
      - [Examples](#examples)
    - [Condition Functions](#condition-functions)
    - [Wait Conditions](#wait-conditions)
      - [Wait Conditions and Handlers](#wait-conditions-and-handlers)
    - [Nested Stacks](#nested-stacks)
    - [Deletion Policies](#deletion-policies)
    - [Stack Updates](#stack-updates)
      - [Stack Policies](#stack-policies)
      - [Resource Impacts](#resource-impacts)
    - [Change Sets](#change-sets)
    - [Custom Resources](#custom-resources)
  - [Elastic Beanstalk](#elastic-beanstalk)
    - [.ebextensions](#ebextensions)
  - [AWS Config](#aws-config)
  - [Amazon ECS](#amazon-ecs)
  - [AWS Lambda](#aws-lambda)
  - [Step Functions](#step-functions)
  - [OpsWorks](#opsworks)
    - [OpsWorks Stacks](#opsworks-stacks)
      - [Chef](#chef)

# Configuration Management and Infrastructure as Code
Notes taken from ACG DevOps Pro 2020 course.

## CloudFormation

### What is CloudFormation?
* Gives you building blocks to describe the infrastructure you want in AWS
* Text files written in JSON or YAML; version and track changes like code
* Free; only pay for infra it provisions

### Terms
* **Stack** - collection of AWS resources that you manage as a single unit. Created when you give CloudFormation your template.
* **Template** - document that describes how to create and configure resources. Can be used to either create or update a stack.
* **Stack Policy** - IAM-style policy to governs what can be changed and who can change it. 

### Template Anatomy
* **Parameters** - allows passing of variables into the template
* **Mappings** - allow processing of hashes (arrays of key-value pairs) by the template
* **Resources** - declare your resources 
* **Outputs** - results from the template

### Use cases
* Deploy infrastructure in an automated manner rather than manually
* Create a repeatable, patterned environment and reuse when necessary
* Run automated testing for CI/CD environments by creating a dedicated, clean environment for running tests. Delete the environment when completed, all with no human intervention
* Define an environment all at once and deploy to any region in AWS without reconfiguration
* Manage infrastructure configuration using software development-style versioning and code repositories


### Intrinsic Functions
CloudFormation provides functions that assist in assigning values to template properties that are not available until runtime. 

#### Examples
* **FindInMap**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: 'ami-1234xyz'

Resources:
  rEc2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [ RegionMap, !Ref AWS::Region, HVM64 ]
```
* **Base64**
  * Returns Base64 representation of an input string
  * Used for defining EC2 UserData
```yaml
!Base64 inputString
```
* **Cidr**
  * Returns an array of CIDR block addresses
  * Number of block returns depends on *count* value from 1 to 256
```yaml
!Cidr [ ipBlock, count, cidrBits ]
```
* **GetAtt**
  * Returns the value of an attribute from a resource in the template
```yaml
!GetAtt logicalResourceName.attributeName
```
* **GetAZs**
  * Returns an array of the available AZs in a region
```yaml
!GetAZs region
```
* **ImportValue**
  * Used to import a value of an output exported from another stack
  * Used to create cross-stack references
```yaml
!ImportValue nameOfExport 
```
* **Join**
  * Appends a set of values into a single value, separated by a specified delimiter
```yaml
!Join [ delimiter, [ list, of, values ]]
```
* **Select**
  * Returns a single object from a list of objects based on index
```yaml
!Select [ index, listOfObjects ]
```
* **Split**
  * Splits a string into a list of string values so that you may select an element from the result
```yaml
!Split [ delimiter, sourceString ]
```
* **Sub**
  * Substitutes variables in an input string with values you specify
```yaml
!Sub 
  - source string
  - {var1Name: var1Value}
```
* **Transform**
  * Specifies a macro to perform custom processing on part of a template
```yaml
!Transform {'Name': macroName, 'Parameters':{key:value}}
```
* **Ref**
  * Returns the value of a parameter or resource 
```yaml
!Ref logicalName
```

### Condition Functions
* Use these to conditionally create stack resources or configurations
* Evaluated based on input parameters declared when creating or updating a stack
* Functions:
  * Fn::And
  * Fn::Or
  * Fn::If
  * Fn::Not
  * Fn::Equals

### Wait Conditions
* **DependsOn** - used for controlling resource creation order within CloudFormation
```yaml
Resources:
  rEc2:
    Type: AWS::EC2::Instance
    DependsOn: rIgw # Must wait to be created until after this resource
    Properties:
      ImageId: 'ami-1234xyz'
```
* **Creation Policy** - prevent a resource from reaching create complete until CloudFormation received a specified number of success signals or the timeout period is exceeded
```yaml
CreationPolicy:
  ResourceSignal:
    Count: 3
    Timeout: PT15M
```

#### Wait Conditions and Handlers
* Allows you to coordinate stack resource creation with other configuration actions external to the stack
* Allows you to track the status of a configuration process
* Wait condition handlers are a resource with no properties that generate a signed URL which can be used to communicate a SUCCESS or FAILURE
* Components:
  * DependsOn for the resource(s) you are waiting on
  * Handle property references the handle mentioned above
  * Response timeout
  * Count (default is 1)
```yaml
rWaitHandle:
  Type: AWS::CloudFormation::WaitConditionHandle # Creates the signed URL

rWaitCondition:
  Type: AWS::CloudFormation::WaitCondition
  DependsOn: rWebServerGroup
  Properties:
    Handle: !Ref rWaitHandle
    Timeout: 300
    Count: !Ref pWebServerCapacity 

  rWebServerGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      LaunchConfiguration: !Ref rLaunchConfig
      MinSize: 1
      MaxSize: 5
      DesiredCapacity: !Ref pWebServerCapacity
      LoadBalancerNames:
        - !Ref rAlb
```
* How are wait conditions and handlers different than creation policies?
  * You can implement a complex order of conditions; wait condition can depend on many resources and have many resources depend on it
  * You can influence the order in which this are built by using `DependsOn`
  * Additional data can be passed back via the signed URL which can then be accessed by the template

### Nested Stacks
* With nesting, a whole other stack can be a resource within a stack
* Nested stacks can have nested stacks
* Use cases:
  * Allows a large set of infra to be split into multiple templates
  * Limits to stacks -- 200 resources, 60 outputs, and 60 parameters
  * Allows more effective IaC reuse

### Deletion Policies
* Setting associated with each resource in a CloudFormation template; controls what happens to a resource when the stack is deleted
* Can be one of:
  * Delete (default) - delete the resource
  * Retain - resource is retained
  * Snapshot - exists on only a few services like EBS volumes, RDS databases, and Redshift. Creates a snapshot of the volume before deleting

### Stack Updates
* What happens when we update a stack?
  * Stack policy is checked
  * Changes are orchestrated based on changes to the template

#### Stack Policies
* The absence of a stack policy allows all updates
* Once a stack policy is applied:
  * It cannot be deleted
  * All objects are protected and `Update:*` is denied, unless explicitly allowed in the policy

#### Resource Impacts
* The change a resource undergoes during an update is dependent on the resource or property
* An update can impact a resource in the following ways:
  * No interruption - no impact to the resource
  * Some interruption - service my be impacted briefly (i.e. an instance reboot)
  * Replacement - the entire resource must be replaced
  * Delete - the resource is deleted entirely

### Change Sets
* Preview how changes will impact your stack and resources
* See if changes will delete or replace critical resources
* Let you make changes only when you execute the change set

### Custom Resources
* Enable you to write custom provisioning logic in templates; you can also extend CloudFormation beyond AWS
* How do they work?
  * Template developer creates a template with a custom resource type
  * A service token as well as any input to the custom resource is defined
  * A custom resource provider owns the custom resource and knows how to handle/respond to requests from CloudFormation. The resource owner provides the service token to the developer.
  * CloudFormation sends a request to the service token specified and waits for a response before proceeding with additional stack operations

## Elastic Beanstalk
* An orchestration service offered by AWS that is used to deploy and scale web applications and services
* Supports Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker
* You write the code, Beanstalk does the rest -- deployment, capacity provisions, load balancing, autoscaling, and health monitoring
* Retain full control over the AWS resources that are provisioned. Additionally, you only pay for the resources provisioned

### .ebextensions
* Allows advanced environment customization with YAML or JSON configuration files
* Placed in a folder called `.ebextensions`
* Allows develops to configure the systems being deployed by Beanstalk

```yaml
files:
  'c:/temp/configureUpdates.ps1':
    content: |
      Invoke-Expression "reg add 'HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update' /v.."
      Invoke-Expression "sc config wuauserv start=auto"
      Invoke-Expression "net start wuauserv"
container-commands:
  01-execute-config-script:
    command: Powershell.exe -ExecutionPolicy Bypass -File c:\\temp\\configureUpdates.ps1
    ignoreErrors: true
    waitAfterCompletion: 0
```

## AWS Config
* Continuous monitoring of resource configurations; notifications can be configured for non-compliant configuration changes
* Continuous assessment of resources to keep configurations aligned with organizational policies
* Troubleshooting configuration issues by comparing resource configurations from one point in time to another
* Compliance status across all AWS accounts
* Change management

## Amazon ECS
* Highly scalable service that orchestrates Docker containers across a cluster
* Supports API calls and scheduling of tasks

## AWS Lambda
* Run code without servers. Basis of serverless computing model.
* 1 million requests for free per month
* Triggering via services or in response to events from other services, e.g.:
  * S3
  * DynamoDB
  * Kinesis Streams
  * SNS notifications
* Cost savings due to only have to pay for compute cycles used. No "always on" infrastructure
* Less management overhead

## Step Functions
* A service that allows you to orchestrate Lambda functions
* Reliable way to step through functions that power your application in a particular order
* Presents you with graphical view of application components

## OpsWorks
* **OpsWorks for Chef Automate** - fully-managed config management service that hosts Chef Automate
* **OpsWorks for Puppet Automate** - fully managed config management service that hosts Puppet Enterprise
* **OpsWorks Stacks** - application and server management service; used to be called just "OpsWorks." Allows application deployment in stacks with configuration using Chef Solo

### OpsWorks Stacks
* Manage applications on AWS and on-prem
* Design layers that perform different functions
* Supports autoscaling ad scheduled scaling
* Implements Chef Solo for config management 

#### Chef
* OpsWorks Stacks/Chef is a declarative state engine
* State what you want to happen and OpsWorks Stacks/Chef handles the how
* OpsWorks Stacks has resources it can use, such as packages to install, services to control, and configuration files to update
* Recipes tell OpsWorks Stacks/Chef what you want the end result to be

```ruby
package "httpd" do
  action :install
end

service "httpd" do
  action [:enable, :start]
  supports :restart => :true
end

template "/var/www/html/index.html" do
  source "index.html.erb"
  owner "apache"
  group "apache"
  mode "0644"
  notifies :restart, "service[httpd]"
end
```