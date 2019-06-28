## Intrinsic Functions
* Built-in functions that help you manage the stack
* [List of Intrinsic Functions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

### Join
* Appends a set of values into a single value

JSON syntax and example:
```json
{"Fn::Join":["delimiter",[comma-delimited list of values]]}

{"Fn::Join" : [ ":", [ "a", "b", "c" ] ]}
```
YAML:
```yaml
!Join [delimiter,[comma-delimited list of values]]

!Join [ ":", [ a, b, c ] ]
```

* Both of the above would output the value `"a:b:c"`

### Split
* Split a string into a list based on a 'needle'

Example:
```yaml
!Split [delimiter, [string to split]]

!Split ["|", ["a|b|c"]]
```
* This outputs `["a","b","c"]`

### Sub
* Complex token substitution of text within a large block of text. Commonly used for user data scripts.

Exmaple:
```yaml
Name: !Sub
  - www.${Domain}
  - {Domain: !Ref RootDomainName}
```

### Select
* Select an item from a list based on its numerical 0-based index

Example:
```yaml
!Select ["1",["apples","grapes","oranges","mangoes"]]
```
* This would output `"grapes"`

### Base64
* Used to encode a string to Base64. Commonly used to pass user data to an EC2 instance.

### FindInMap
* Select a value from the template's mappings section

### Ref
* Refer to another logical resource in the stack:
```yaml
ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", 32]
```

### GetAtt
* Get an attribute of another logical resource in the stack:
```yaml
!GetAtt myELB.DNSName
```
* This would output the DNS name of the load balancer with logical name myELB

### GetAZs
* Get a list of Availability Zones available to the account in the current region
```yaml
Fn::GetAZs: us-east-1
```
* This outputs `["us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d", "us-east-1e", "us-east-1f"]`