## Using Boto with S3
All credit for code snippets goes to Linux Academy. Notes were taken from their course content.

### Resizing Images
* In some situations, you may want to automatically resize uploaded images (for webistes, CDNs, etc)
* Lambda can be triggered by S3 PUTs of objects, generate a thumbnail with it, and PUT the object into a destination bucket
* Ensure that the Lambda function has the proper IAM execution role for S3 GETs and PUTs
* Code example:

```python
import os
import tempfile #For working with temporary directories
import boto3
from PIL import Image #Python image library (aka Pillow) for image resizing

s3 = boto3.client('s3')
DEST_BUCKET = os.environ['DEST_BUCKET']
SIZE = 128, 128

def lambda_handler(event, context):
    
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        thumb = 'thumb-' + key
        with tempfile.TemporaryDirectory() as tmpdir:
        	download_path = os.path.join(tmpdir, key)
        	upload_path = os.path.join(tmpdir, thumb)
        	s3.download_file(source_bucket, key, download_path)
        	generate_thumbnail(download_path, upload_path)
        	s3.upload_file(upload_path, DEST_BUCKET, thumb)

        print('Thumbnail image saved at {}/{}'.format(DEST_BUCKET, thumb))

def generate_thumbnail(source_path, dest_path):
	print('Generating thumbnail from:', source_path)
	with Image.open(source_path) as image:
		image.thumbnail(SIZE)
		image.save(dest_path)
```

* Note that not all libraries will always be included in Lambda's default environment. In some cases you may need to download libraries required and upload them to Lambda with your code as a zip file.

### Importing CSVs into DynamoDB
* Sometimes you may have CSV files uploaded to S3 by an application or user. You may want to process that data and store it in Dynamo
* You can set up an S3 PUT trigger to have Lambda download the CSV batch, transform it (e.g. turn into dictionary objects), and upload to Dynamo
* Ensure that your IAM execution role has the ability to GET S3 objects and allow access to the Dynamo table
* Example code:

```python
import csv
import os
import tempfile
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Movies') #Select the 'Movies' table
s3 = boto3.client('s3')

def lambda_handler(event, context):
    
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        with tempfile.TemporaryDirectory() as tmpdir:
            download_path = os.path.join(tmpdir, key)
            s3.download_file(source_bucket, key, download_path)
            items = read_csv(download_file)

            with table.batch_writer() as batch: #The batch writer will automatically upload data to Dynamo in batches. It will also retry PUTs on unprocessed items
                for item in items:
                    batch.put_item(Item=item)

def read_csv(file):
    items=[]
    with open(file) as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            data = {}
            data['Meta'] = {}
            data['Year'] = int(row['Year'])
            data['Title'] = row['Title'] or None
            data['Meta']['Length'] = int(row['Length'] or 0)
            data['Meta']['Length'] = int(row['Length'] or 0)
            data['Meta']['Subject'] = row['Subject'] or None
            data['Meta']['Actor'] = row['Actor'] or None
            data['Meta']['Actress'] = row['Actress'] or None
            data['Meta']['Director'] = row['Director'] or None
            data['Meta']['Popularity'] = row['Popularity'] or None
            data['Meta']['Awards'] = row['Awards'] == 'Yes'
            data['Meta']['Image'] = row['Image'] or None
            data['Meta'] = {k: v for k,
                            v in data['Meta'].items() if v is not None} #This will remove any keys with empty values
            items.append(data)
``` 

### Transcoding Video with S3 and Elastic Transcoder
* You can upload raw video files to S3 and have Elastic Transcoder autmoatically convert the files to play on different devices (smartphones, tablets, PCs, etc.)
* Ensure that your Lambda function has the correct execution role for S3 GETs and to interact with Elastic Transcoder
* Example code:
```python
from datetime import datetime
import json
import urllib.parse
import os
import boto3

PIPELINE_ID = os.environ['PIPELINE_ID']

transcoder = boto3.client('elastictranscoder')
s3 = boto3.resource('s3')


def lambda_handler(event, context):
    print("Received event: " + json.dumps(event))

    # Get the object from the event
    key = urllib.parse.unquote_plus(
        event['Records'][0]['s3']['object']['key'], encoding='utf-8')

    filename = os.path.splitext(key)[0]  # filename w/o extension

    # Create a job
    job = transcoder.create_job(
        PipelineId=PIPELINE_ID,
        Input={
            'Key': key
        },
        Outputs=[
            {
                'Key': filename + '-1080p.mp4',
                'ThumbnailPattern': filename + '-{resolution}-{count}',
                'PresetId': '1351620000001-000001'  # Generic 1080p
            },
            {
                'Key': filename + '-720p.mp4',
                'ThumbnailPattern': filename + '-{resolution}-{count}',
                'PresetId': '1351620000001-000010'  # Generic 720p
            }
        ]
    )

    print("start time={}".format(datetime.now().strftime("%H:%M:%S.%f")[:-3]))
    print("job={}".format(job))
    job_id = job['Job']['Id']

    # Wait for the job to complete
    waiter = transcoder.get_waiter('job_complete')
    waiter.wait(Id=job_id)
    end_time = datetime.now().strftime("%H:%M:%S.%f")[:-3]
    print("end time={}".format(end_time))
```