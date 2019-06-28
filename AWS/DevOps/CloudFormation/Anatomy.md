## Template Anatomy in Further Detail

### Parameters
* Used to pass user input to variables when launching a stack
* You can use the "Ref" function later in the template to reference these variables

```yaml
Parameters:
  InstanceTypeParameter: #logical/variable name
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro #Your default value needs to be in the list or it will not work
      - m1.small
      - m1.large
    Description: Enter t2.micro, m1.small, or m1.large. Default is t2.micro.

#Using 'Ref'
Resources:
  Ec2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceTypeParameter
      ImageId: ami-123xyz
```

* Allowed Pattern - Java Regex:
```yaml
Paramaeters:
  AdminUserAccount:
    Default: admin
    Description: The admin account user name
    Type: String
    MinLength: 1
    MaxLength: 16
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
```

### Resources
* Resources actually interact with the AWS API, so you need the correct permissions and regional limits in place to execute this section
* Resources include but are not limited to:
    * IAM policies, users, roles, groups
    * EC2 instances, ASGs
    * RDS DBs, S3 buckets
    * ELBs
    * CW alarms
    * Lambda functions
    * Logging via CloudTrail, CW Logs
```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      UserData: #For bootstrapping
        "Fn::Base64":
          !Sub |
            Queue=${MyQueue}
      AvailabilityZone: us-east-1a
      ImageId: ami-123-xyz
  MyQueue:
    Type: AWS::SQS::Queue
    Properties: {}
```

#### Custom Resources
* Used when AWS CFN doesn't support a needed API call
* CFN hands a payload to send to Lambda, which then executes some arbitrary code
* The function needs to return output to CFN template before it is allowed to proceed
    * The stack will hang if it doesn't receive a response
```yaml
MyCustomResource:
  Type: Custom::TestLambdaCrossStackRef
  Properties:
    ServiceToken:
      !Sub |
        arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:${LambdaFunctionName}
    StackName:
      Ref: "NetworkStackName"
```

### Mappings
* Set of custom name-value pairs
* Use for setting values based on different possible conditions (ex. regions)
* Commonly used for mapping AMI IDs to make template reusable accross regions
```yaml
RegionMap:
  us-east-1:
    '32': 'ami-123'
    '64': 'ami-456'
  us-west-1:
    '32': 'ami-789'
    '64': 'ami-1011'
#etc
```
* Use `Fn::FindInMap` when referencing the mapped values:
`AMIForDeploy: !FindInMap [RegionMap, !Ref "AWS::Region", 32]`

### Conditions
* Use condition functions to make some resources optional

```yaml
MyCondition: !And
  - !Equals ["sg-abc123", !Ref ASecurityGroup]
  - !Condition SomeOtherCondition
```
* This example evaluates to true if the referenced security group name is equal to "sg-abc123" AND if "SomeOtherCondition" evaluates to true

### Outputs/Exports
* Outputs allow you to pass values back to your user
* Export allows you to publish values to a cross-stack referenceable variable

Outputs/exports example:
```yaml
Outputs:
  StackVPC:
    Description: The ID of the VPC
    Value: !Ref MyVPC
    Export:
      Name: !Sub "${AWS::StackName}-VPCID"
```