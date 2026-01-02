# Serverless Data Processing Pipeline on AWS

## 📌 Project Overview
In this project, I designed and implemented a serverless, event-driven data processing pipeline using AWS managed services.
The goal was to understand how cloud-native systems process data automatically without managing servers.

The pipeline reacts to file uploads in Amazon S3, processes the event using AWS Lambda, stores results in Amazon DynamoDB, and records execution logs in Amazon CloudWatch.

This project helped me gain hands-on experience with serverless architecture, event triggers, IAM roles, and AWS monitoring.

### How it works:
1. **Trigger:** A file is uploaded to an Amazon S3 bucket.
2. **Compute:** An AWS Lambda function is automatically triggered by the S3 event.
3. **Storage:** The Lambda function extracts metadata and stores the result in Amazon DynamoDB.
4. **Monitoring:** All execution logs are captured using Amazon CloudWatch.

> **Note:** This project was implemented and tested in a temporary cloud/lab environment, but the architecture is fully compatible with personal AWS accounts.

---

## 🏗️ Architecture Diagram (Logical Flow)
`S3 Bucket (Upload) ➔ S3 Event Notification ➔ AWS Lambda (Python) ➔ DynamoDB Table`
<img width="521" height="340" alt="image" src="https://github.com/user-attachments/assets/142b0e34-6682-40f3-a1e3-b12bfba955fc" />

<img width="794" height="239" alt="image" src="https://github.com/user-attachments/assets/8af6a1f1-bb2c-46e1-8f3e-d5d9c88ea8f1" />

---

## 🛠️ AWS Services Used
* **Amazon S3:** Scalable object storage for input files.
* **AWS Lambda:** Serverless compute for processing metadata.
* **Amazon DynamoDB:** NoSQL database for storing processed results.
* **Amazon CloudWatch:** Real-time logging and monitoring.
* **IAM:** Role-based access control for secure service communication.
---


## STEP 1: Create an S3 Bucket

### Go to:
AWS Console → S3 → Create bucket

### Configuration:
- **Bucket name**: `serverless-data-input-<unique-id>`
- **Region**: Same region as Lambda and DynamoDB
- **Object ownership**:  
  ✅ ACLs disabled (Bucket owner enforced)
- **Block public access**:  
  ✅ Keep all options enabled
- **Bucket versioning**: Disabled (optional)
- **Encryption**:  
  ✅ Enable SSE-S3 (Amazon S3 managed keys)

Click **Create bucket**.

📸 Screenshot:
!\[s3-bucket.](screenshots/01-s3-bucket-created.png)





---

## STEP 2: Create DynamoDB Table

### Go to:
AWS Console → DynamoDB → Tables → Create table

### Configuration:
- **Table name**: `ProcessedResults`
- **Partition key**: `file_name` (String)
- **Capacity mode**: On-demand
- **Encryption**: Default (enabled)

Click **Create table** and wait for **Status: Active**.



📸 Screenshot:

!\[02-dynamodb-table.](screenshots/02-dynamodb-table.png)



---

## STEP 3: Create Lambda Function

> ⚠️ Note: In restricted lab/sandbox environments, IAM role creation may be blocked.  
> In that case, select the **pre-configured lab execution role**.

### Go to:
AWS Console → Lambda → Create function

### Configuration:
- **Author from scratch**
- **Function name**: `serverless-data-processor`
- **Runtime**: Python 3.12
- **Execution role**:  
  ✅ Use existing role (LabRole / VocLabsRole / AWSAcademyRole)

Click **Create function**.

📸 Screenshot:



!\[03-lambda-function-role.](screenshots/03-lambda-function-role.png)





---

## STEP 4: Add Lambda Code

Open the Lambda function → Code tab  
Replace the default code with the following:

``` python

import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ProcessedResults')

def lambda_handler(event, context):
    record = event['Records'][0]
    file_name = record['s3']['object']['key']
    bucket_name = record['s3']['bucket']['name']

    table.put_item(
        Item={
            'file_name': file_name,
            'bucket': bucket_name,
            'status': 'Processed'
        }
    )

    return {
        'statusCode': 200,
        'body': json.dumps('File processed successfully')
    }
```

!\[04-lambda-code.](screenshots/04-lambda-code.png)


---

## Step 5: Configure S3 Event Trigger for Lambda

To make the pipeline event-driven, I configured Amazon S3 to automatically invoke the Lambda function whenever a new file is uploaded.

**Configuration details:**
- Trigger source: Amazon S3
- Event type: All object create events
- Bucket: Serverless input bucket created earlier
- Prefix/Suffix: Not used

This ensures that **every file upload (PUT operation)** triggers the Lambda function without any manual intervention.

**Outcome:**  
The pipeline is now fully automated and reacts in real time to incoming data.

!\[05-s3-lambda-trigger.](screenshots/05-s3-lambda-trigger.png)

---



## Step 6: Validate End-to-End Data Processing

After configuring the trigger, I validated the complete serverless workflow.

### Test Performed
1. Uploaded a test file (`test.txt`),heelo.json to the S3 bucket
2. The S3 upload generated an event automatically
3. The Lambda function was invoked without manual execution
4. Lambda processed the event and wrote metadata to DynamoDB


📸 Screenshot:

!\[06-s3-file-uploaded.](screenshots/06-s3-file-uploaded.png)

---

## Step 7: Verify Data in DynamoDB

To confirm successful processing, I checked the DynamoDB table.

**Observed result:**
- A new item was created in the table
- Stored attributes included:
  - `file_name`
  - `bucket`
  - `status = Processed`

This confirmed that the Lambda function executed successfully and persisted the processed data.

📸 Screenshot:


!\[07-dynamodb-item.](screenshots/07-dynamodb-item.png)
---


## Step 8: Monitor Execution Using CloudWatch

For observability and debugging, I reviewed the execution logs in Amazon CloudWatch.

**What I verified:**
- Lambda invocation logs
- Successful execution without errors
- Event metadata captured in logs

CloudWatch provided visibility into the serverless execution flow and confirmed reliable operation.


📸 Screenshot:

!\[08-cloudwatch-logs.](screenshots/08-cloudwatch-logs.png)



---

## Final Outcome

The serverless data processing pipeline works end-to-end:

- File uploads trigger processing automatically
- No servers are provisioned or managed
- Data is processed and stored reliably
- Logs are available for monitoring and troubleshooting

This implementation demonstrates a **production-style, event-driven serverless architecture** using AWS managed services.


## 🧠 Key Learnings


- Serverless event-driven architecture
- AWS managed services integration
- Real-time data processing
- Cloud monitoring with CloudWatch
- Cost-efficient cloud design

