# AWS Lambda S3 Large File Upload Notification

This AWS Lambda function is designed to monitor an S3 bucket and send a notification whenever a file larger than 10GB is uploaded to the bucket. The function utilizes AWS services including S3, Lambda, and SNS to provide real-time alerts for large file uploads.

## Overview

When a new object is uploaded to an S3 bucket, this Lambda function is triggered by an S3 event. It checks the size of the uploaded object, and if the size exceeds 10GB, it sends an SNS notification to alert the operator.

### Features:
- Triggers on S3 `ObjectCreated` events (file uploads).
- Checks the size of the uploaded object.
- Sends an SNS notification if the file size is greater than 10GB.

## Prerequisites

Before deploying the Lambda function, ensure you have the following:

- **AWS Account**: With appropriate IAM permissions.
- **S3 Bucket**: The bucket where large files will be uploaded.
- **SNS Topic**: Set up an SNS topic to receive notifications (e.g., email notifications for the operator).
- **IAM Role**: Ensure the Lambda function has the necessary permissions (to access S3 and SNS).

## AWS Services Used

- **AWS Lambda**: Executes the function upon S3 upload events.
- **Amazon S3**: Stores the files being uploaded.
- **Amazon SNS**: Sends the notification to the operator.

## Lambda Function Code

The Lambda function is written in Python 3.x. It is triggered by an S3 `ObjectCreated` event and checks if the uploaded file exceeds 10GB in size. If the file is larger than 10GB, it sends an SNS notification.

### Lambda Function Code

```python
import json
import boto3
import os

# Initialize AWS clients
s3_client = boto3.client('s3')
sns_client = boto3.client('sns')

# SNS topic ARN (to notify the operator)
SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']  # Set in Lambda environment variables

def lambda_handler(event, context):
    # Extract bucket name and object key from the event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']

    try:
        # Fetch object metadata to get the size of the uploaded file
        response = s3_client.head_object(Bucket=bucket_name, Key=object_key)
        object_size = response['ContentLength']  # Size in bytes

        # Check if the file size is greater than 10GB (10GB = 10 * 1024 * 1024 * 1024 bytes)
        if object_size > 10 * 1024 * 1024 * 1024:
            # Send SNS notification if the object is larger than 10GB
            message = f"Warning: An object with size {object_size / (1024 ** 3):.2f} GB was uploaded to S3 bucket '{bucket_name}'. Object Key: {object_key}"
            sns_client.publish(
                TopicArn=SNS_TOPIC_ARN,
                Message=message,
                Subject='Large File Upload Notification'
            )
            print(f"Notification sent for large file upload: {object_key}, size: {object_size / (1024 ** 3):.2f} GB")
        else:
            print(f"Object '{object_key}' is within size limits: {object_size / (1024 ** 3):.2f} GB")
    
    except Exception as e:
        print(f"Error occurred: {str(e)}")
        raise e
