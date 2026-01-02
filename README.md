# Serverless Data Processing Pipeline on AWS

## 📌 Project Overview
This project demonstrates how to build a **serverless, event-driven data processing pipeline** using AWS managed services. 

### How it works:
1. **Trigger:** A file is uploaded to an Amazon S3 bucket.
2. **Compute:** An AWS Lambda function is automatically triggered by the S3 event.
3. **Storage:** The Lambda function extracts metadata and stores the result in Amazon DynamoDB.
4. **Monitoring:** All execution logs are captured using Amazon CloudWatch.

> **Note:** This project was implemented and tested in a temporary cloud/lab environment, but the architecture is fully compatible with personal AWS accounts.

---

## 🏗️ Architecture Diagram (Logical Flow)
`S3 Bucket (Upload) ➔ S3 Event Notification ➔ AWS Lambda (Python) ➔ DynamoDB Table`

---


\## 🎯 What You Will Learn

\- Event-driven serverless architecture

\- S3 event notifications

\- Lambda function creation and execution

\- DynamoDB data storage

\- CloudWatch log monitoring

\- Secure default AWS configurations



---



\## 🧱 Step-by-Step Implementation Guide



---



\## STEP 1: Create an S3 Bucket



\### Go to:

AWS Console → S3 → Create bucket



\### Configuration:

\- \*\*Bucket name\*\*: `serverless-data-input-<unique-id>`

\- \*\*Region\*\*: Same region as Lambda and DynamoDB

\- \*\*Object ownership\*\*:  

&nbsp; ✅ ACLs disabled (Bucket owner enforced)

\- \*\*Block public access\*\*:  

&nbsp; ✅ Keep all options enabled

\- \*\*Bucket versioning\*\*: Disabled (optional)

\- \*\*Encryption\*\*:  

&nbsp; ✅ Enable SSE-S3 (Amazon S3 managed keys)



Click \*\*Create bucket\*\*.



📸 Screenshot:

!\[s3-bucket.](screenshots/01-s3-bucket-created.png)





---



\## STEP 2: Create DynamoDB Table



\### Go to:

AWS Console → DynamoDB → Tables → Create table



\### Configuration:

\- \*\*Table name\*\*: `ProcessedResults`

\- \*\*Partition key\*\*: `file\_name` (String)

\- \*\*Capacity mode\*\*: On-demand

\- \*\*Encryption\*\*: Default (enabled)



Click \*\*Create table\*\* and wait for \*\*Status: Active\*\*.



📸 Screenshot:

!\[02-dynamodb-table.](screenshots/02-dynamodb-table.png)





---



\## STEP 3: Create Lambda Function



> ⚠️ Note: In restricted lab/sandbox environments, IAM role creation may be blocked.  

> In that case, select the \*\*pre-configured lab execution role\*\*.



\### Go to:

AWS Console → Lambda → Create function



\### Configuration:

\- \*\*Author from scratch\*\*

\- \*\*Function name\*\*: `serverless-data-processor`

\- \*\*Runtime\*\*: Python 3.12

\- \*\*Execution role\*\*:  

&nbsp; ✅ Use existing role (LabRole / VocLabsRole / AWSAcademyRole)



Click \*\*Create function\*\*.



📸 Screenshot:



!\[03-lambda-function-role.](screenshots/03-lambda-function-role.png)





---



\## STEP 4: Add Lambda Code



Open the Lambda function → Code tab  

Replace the default code with the following:



```python

import json

import boto3



dynamodb = boto3.resource('dynamodb')

table = dynamodb.Table('ProcessedResults')



def lambda\_handler(event, context):

&nbsp;   record = event\['Records']\[0]

&nbsp;   file\_name = record\['s3']\['object']\['key']

&nbsp;   bucket\_name = record\['s3']\['bucket']\['name']



&nbsp;   table.put\_item(

&nbsp;       Item={

&nbsp;           'file\_name': file\_name,

&nbsp;           'bucket': bucket\_name,

&nbsp;           'status': 'Processed'

&nbsp;       }

&nbsp;   )



&nbsp;   return {

&nbsp;       'statusCode': 200,

&nbsp;       'body': json.dumps('File processed successfully')

&nbsp;   }





!\[04-lambda-code.](screenshots/04-lambda-code.png)



\## STEP 5: Add S3 Trigger to Lambda



Inside Lambda → Function overview → Add trigger



\# Configuration:

Trigger source: S3

Bucket: Select the bucket created in Step 1

Event type:

✅ All object create events

Prefix / Suffix: Leave empty

Acknowledge recursive warning



!\[05-s3-lambda-trigger.](screenshots/05-s3-lambda-trigger.png)



STEP 6: Test the Pipeline (Final Proof)

Upload a File



Go to:

S3 → Your bucket → Upload



Upload any file:



test.txt



sample.csv



hello.json



📸 Screenshot:

!\[06-s3-file-uploaded.](screenshots/06-s3-file-uploaded.png)



Verify DynamoDB Output



Go to:

DynamoDB → Tables → ProcessedResults → Explore table items



You should see:



file\_name: uploaded file name



bucket: S3 bucket name



status: Processed



📸 Screenshot:



!\[07-dynamodb-item.](screenshots/07-dynamodb-item.png)



Verify CloudWatch Logs



Go to:

CloudWatch → Logs → Log groups → /aws/lambda/serverless-data-processor



📸 Screenshot:

!\[08-cloudwatch-logs.](screenshots/08-cloudwatch-logs.png)



🔐 Security Best Practices Followed



S3 bucket is private by default



No public access enabled



Server-side encryption enabled



IAM permissions handled via execution role



No credentials hard-coded



🧠 Key Learnings



Serverless event-driven architecture



AWS managed services integration



Real-time data processing



Cloud monitoring with CloudWatch



Cost-efficient cloud design

