- [Boto3 Code Snippets](#boto3-code-snippets)
  - [Stop EC2 Instances Nightly](#stop-ec2-instances-nightly)
  - [Backing up EC2 Instances](#backing-up-ec2-instances)

## Boto3 Code Snippets
All credit goes to Linux Academy. More snippets can be found [here](https://github.com/linuxacademy/content-lambda-boto3)

### Stop EC2 Instances Nightly

```python
import boto3

def lambda_handler(event, context):

	ec2_client = boto3.client('ec2')
	regions = [region['RegionName']
		for region in ec2_client.describe_regions()['Regions']] # Creates a list of all available regions

	#Iterate through each region
	for region in regions:
		ec2 = boto3.resource('ec2', region_name=region)

		print('Region name: ', region)

		#Get running instances
		instances = ec2.instances.filter(
			Filters=[{'Name': 'instance-state-name',
				'Values': ['running']}])

		#Stop the instances
		for instance in instances:
			instance.stop()
			print('Stopped instance: ', instance.id)
```

### Backing up EC2 Instances
```python
import boto3
from datetime import datetime

def lambda_handler(event, context):
    
    ec2_client = boto3.client('ec2')
    regions = [region['RegionName']
        for region in ec2_client.describe_regions()['Regions']]

    for region in regions:
        print('Instances in EC2 Region {0}:'.format(region))
        ec2 = boto3.resource('ec2', region_name=region)

        instances = ec2.instances.filter(
            Filters=[
                {'Name': 'tag:backup', 'Values': ['true']}
            ]
        )

        timestamp = datetime.utcnow().replace(microsecond=0).isoformat()

        for i in instances.all():
            for v in i.volumes.all():
                desc = 'Backup of {0}, volume {1}, created {2}'.format(
                    i.id, v.id, timestamp)
                print (desc)

                snapshot = v.create_snapshot(Description=desc)

                print("Created snapshot:", snapshot.id)
```
