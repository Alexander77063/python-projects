# S3 File Organizer Lambda Function

This project provides a Lambda function that automatically organizes files uploaded to an S3 bucket into folders based on the file's creation date. The files are moved to a folder named `yyyy-mm-dd/filename`, where `yyyy-mm-dd` is the date the file was created in the S3 bucket.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [How It Works](#how-it-works)
- [Deployment](#deployment)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)

## Overview
When a client uploads files to an S3 bucket, this Lambda function is triggered. It checks the creation date of the file and moves it to a folder named after the date (`yyyy-mm-dd`). This helps in organizing files chronologically.

## Prerequisites
- An AWS account with access to S3 and Lambda.
- Python 3.x installed locally (for testing and deployment).
- AWS CLI configured with appropriate permissions.
- Boto3 library (AWS SDK for Python).

## Setup
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Alexander77063/python-projects.git
   cd python-projects/s3-file-organizer
   ```

2. **Install Dependencies**:
   Ensure you have the `boto3` library installed:
   ```bash
   pip install boto3
   ```

3. **Configure AWS CLI**:
   If not already configured, set up your AWS CLI with the necessary credentials:
   ```bash
   aws configure
   ```

## How It Works
1. The Lambda function is triggered whenever a new file is uploaded to the specified S3 bucket.
2. It retrieves the file's creation date using the `LastModified` attribute.
3. It constructs a new key (path) in the format `yyyy-mm-dd/filename`.
4. The file is moved to the new location using the `copy_object` and `delete_object` S3 operations.

### Lambda Function Code
Here’s a snippet of the Lambda function:
```python
import boto3
from datetime import datetime

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get the bucket and object key from the event
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']
    
    # Get the object's metadata
    response = s3.head_object(Bucket=bucket_name, Key=object_key)
    last_modified = response['LastModified']
    
    # Format the date as yyyy-mm-dd
    folder_name = last_modified.strftime('%Y-%m-%d')
    
    # Construct the new key
    new_key = f"{folder_name}/{object_key.split('/')[-1]}"
    
    # Copy the object to the new location
    s3.copy_object(
        Bucket=bucket_name,
        CopySource={'Bucket': bucket_name, 'Key': object_key},
        Key=new_key
    )
    
    # Delete the original object
    s3.delete_object(Bucket=bucket_name, Key=object_key)
    
    return {
        'statusCode': 200,
        'body': f"File moved to {new_key}"
    }
```

## Deployment
1. **Package the Lambda Function**:
   Zip the Lambda function code:
   ```bash
   zip lambda_function.zip lambda_function.py
   ```

2. **Deploy to AWS Lambda**:
   Use the AWS CLI to create or update the Lambda function:
   ```bash
   aws lambda create-function \
       --function-name s3-file-organizer \
       --runtime python3.8 \
       --handler lambda_function.lambda_handler \
       --role arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_LAMBDA_ROLE \
       --zip-file fileb://lambda_function.zip
   ```

3. **Configure S3 Trigger**:
   Set up an S3 trigger to invoke the Lambda function whenever a new file is uploaded to the bucket.

## Testing
- Upload a file to the S3 bucket.
- Verify that the file is moved to the correct `yyyy-mm-dd` folder.

### Notes:
- Replace `YOUR_ACCOUNT_ID` and `YOUR_LAMBDA_ROLE` with your actual AWS account ID and Lambda execution role ARN.
- Ensure the Lambda function has the necessary permissions to read and write to the S3 bucke
