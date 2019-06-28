## Bootstrapping Applications
* CloudFormation can bootstrap EC2 instances for you. There are a couple ways to do this.
* Option 1: Use EC2 UserData which is an available property of `AWS::EC2::Instance`
```yaml
Resources:
  Ec2Instance:
    Type: AWS::EC2::Instance
    Properties:
      KeyName: !Ref KeyName
    SecurityGroups: !Ref InstanceSecurityGroup
    ImageId:
      'Fn::FindInMap':
        - RegionMap
        - Ref: 'AWS::Region'
        - AMI
    UserData:
      'Fn::Base64':
        'Fn::Join':
          - ''
          -
            - '#!/bin/bash -ex'
            - yum -y install gcc-c++ make mysql-devel sqlite-devel
```
* Option 2: CloudFormation provides helper scripts for deployment within your EC2 instances
    * Recommended
    * Support for yum, apt, and rpm
    * Create files with permissions
    * Create users, groups
    * Sysvinit service management
```yaml
#Linux bootstrapping
services:
  sysvinit:
    nginx:
      enabled: "true"
      ensureRunning: "true"
      files:
        - "/etc/nginx/nginx.conf"
      sources:
        - "/var/www/html"
      php-fastcgi:
        enabled: "true"
        ensureRunning: "true"
        packages:
          yum:
            - "php"
            - "spawn-fcgi"
        sendmail:
          enabled: "false"
          ensureRunning: "false"

#Windows bootstrapping
services:
  windows:
    cfn-hup:
      enabled: "true"
      ensureRunning: "true"
      files:
        - "c:\\cfn\\crn-hup.conf"
        - "c:\\cfn\\hooks.d\\cfn-auto-reloader.conf"
```

### Adding Authentication for bootstrap files
* AWS::CloudFormation::Authentication
    * HTTP basic auth
    * S3 bucket access
* Referenced by AWS::CLoudFormation::Init
```yaml
AWS::CloudFormation::Authentication:
  testBasic:
    type: "basic"
    username: !Ref UserName
    password: !Ref Password
    uris:
      - "http://www.example.com/test"
    testS3:
      type: "S3"
      accessKeyId: !Ref AccessKeyID
      secretKey: !Ref SecretAccessKeyID
      buckets:
        - "myawsbucket"
```