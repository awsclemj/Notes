## Using Lambda/Boto3 with SQS
All credit for code snippets goes to Linux Academy. Notes were taken from their course content.

### Triggering Lambda from SQS
* A common use case for invoking Lambda is when a message enters an SQS queue
* Whenever a message is placed in the queue, the Lambda function reads the contents of that message and inserts contents to Dynamo
* Steps:
	1. Create SQS queue (standard is fine. FIFO is for more specific use cases)
	2. Create DynamoDB table
	3. Create Lambda function with permissions for Dynamo and SQS
	4. Select your SQS ARN as the trigger

```python
import json
import os
from datetime import datetime

import boto3

QUEUE_NAME = os.environ['QUEUE_NAME']
MAX_QUEUE_MESSAGES = os.environ['MAX_QUEUE_MESSAGES']
DYNAMODB_TABLE = os.environ['DYNAMODB_TABLE']

sqs = boto3.resource('sqs')
dynamodb = boto3.resource('dynamodb')


def lambda_handler(event, context):

    # Receive messages from SQS queue
    queue = sqs.get_queue_by_name(QueueName=QUEUE_NAME)

    print("ApproximateNumberOfMessages:",
          queue.attributes.get('ApproximateNumberOfMessages'))

    for message in queue.receive_messages(
            MaxNumberOfMessages=int(MAX_QUEUE_MESSAGES)):

        print(message)

        # Write message to DynamoDB
        table = dynamodb.Table(DYNAMODB_TABLE)

        response = table.put_item(
            Item={
                'MessageId': message.message_id,
                'Body': message.body,
                'Timestamp': datetime.now().isoformat()
            }
        )
        print("Wrote message to DynamoDB:", json.dumps(response))

        # Delete SQS message
        message.delete()
        print("Deleted message:", message.message_id)
```
