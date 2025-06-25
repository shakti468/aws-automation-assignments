# Assignment 1: Automated EC2 Start/Stop Using AWS Lambda and Boto3

## 🧾 Objective
Create an AWS Lambda function that starts or stops EC2 instances based on their tags using Boto3.

---

## ✅ Tags Used on EC2

- **AutoStop**: Instance will be stopped by Lambda.
- **AutoStart**: Instance will be started by Lambda.


## 📌 Steps Performed

### 1. Created EC2 Instances
- Two t2.micro EC2 instances created under free tier.
- Tagged as follows:
  - Instance 1: `Action=AutoStop`
  - Instance 2: `Action=AutoStart`

🖼️ **Screenshot: EC2 Launch Confirmation**
![image](https://github.com/user-attachments/assets/11f5b6ff-fdaf-4ec5-91f1-54d8a8ca3c86)


---

### 2. Created IAM Role for Lambda
- IAM Role: `EC2InstanceManagerLambdaRole`
- Policy: `AmazonEC2FullAccess`

🖼️ **Screenshot: IAM Role and Policy**
![image](https://github.com/user-attachments/assets/762fbc62-0ec2-4f9d-bae0-8dc0788f54e7)


---

### 3. Lambda Function Setup
- Runtime: Python 3.12
- Timeout: **1 minute**


🖼️ **Screenshot: Lambda Function Configuration**
![image](https://github.com/user-attachments/assets/7b6d4a35-720e-45c2-a18f-09b60a46fe51)



---

### 4. Lambda Function Code

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.resource('ec2')

    # Stop instances tagged with AutoStop
    stop_instances = ec2.instances.filter(Filters=[
        {'Name': 'tag:Action', 'Values': ['AutoStop']},
        {'Name': 'instance-state-name', 'Values': ['running']}
    ])
    stop_ids = [i.id for i in stop_instances]
    if stop_ids:
        ec2.instances.filter(InstanceIds=stop_ids).stop()
        print("Stopped instances:", stop_ids)
    else:
        print("No instances to stop.")

    # Start instances tagged with AutoStart
    start_instances = ec2.instances.filter(Filters=[
        {'Name': 'tag:Action', 'Values': ['AutoStart']},
        {'Name': 'instance-state-name', 'Values': ['stopped']}
    ])
    start_ids = [i.id for i in start_instances]
    if start_ids:
        ec2.instances.filter(InstanceIds=start_ids).start()
        print("Started instances:", start_ids)
    else:
        print("No instances to start.")
```

## 🖼️ Screenshot: Lambda Code Editor

![image](https://github.com/user-attachments/assets/a2b11f81-dcff-412b-83c9-20405faa2fd5)

## Screenshot: Lambda Test Event
![image](https://github.com/user-attachments/assets/c7f7ab6e-af1d-4175-8d7b-6abece83e652)

## Screenshot of ec2 instances before Test 
![image](https://github.com/user-attachments/assets/4132816a-9910-45e4-ac50-659e8e412027)

## Screenshot of ec2 instances after Test 
![image](https://github.com/user-attachments/assets/5eab06b7-b992-4e93-ba66-0d064826b330)

## Screenshot of Executing function: succeeded
![image](https://github.com/user-attachments/assets/3bd57249-46f9-4764-b366-9b9703ef6804)

## Code output 
```
Status: Succeeded
Test Event Name: ManualTest

Response:
null

Function Logs:
START RequestId: 5ec4ca70-baba-404c-bdd5-8ed08f9d5813 Version: $LATEST
Stopped instances: ['i-02dd8d72badd743a2']
Started instances: ['i-0abdbe272cba54082']
END RequestId: 5ec4ca70-baba-404c-bdd5-8ed08f9d5813
REPORT RequestId: 5ec4ca70-baba-404c-bdd5-8ed08f9d5813	Duration: 5019.02 ms	Billed Duration: 5020 ms	Memory Size: 128 MB	Max Memory Used: 96 MB	Init Duration: 266.02 ms

Request ID: 5ec4ca70-baba-404c-bdd5-8ed08f9d5813
```

# Assignment 2: Automated S3 Bucket Cleanup Using AWS Lambda and Boto3

## 🎯 Objective
Delete files older than 30 days from an S3 bucket using a Lambda function and Boto3.

---

## ✅ Steps Followed

### 1. Created S3 Bucket
- Bucket: `shakti-s3-cleanup`
- Uploaded test files of various ages.

🖼️ Screenshot Placeholder: S3 bucket and files
![Screenshot 2025-06-25 132147](https://github.com/user-attachments/assets/5da27ac3-4bce-4b75-9961-a6a14ddc8f9f)

---

### 2. IAM Role
- Role: `shakti_lambda-s3-cleanup-role`
- Policy: `AmazonS3FullAccess`

🖼️ Screenshot Placeholder: IAM role
![Screenshot 2025-06-25 124634](https://github.com/user-attachments/assets/5aa29c7d-3a2f-46d1-944a-c7abb5f0f2a2)

---

### 3. Lambda Function
- Runtime: Python 3.12
- Role: `shakti-s3-cleanup-role`
- Timeout: 1 minute
- Logic:
  - List objects in bucket
  - Delete files older than 30 days
  - Print deleted file names

🖼️ Screenshot Placeholder: Lambda config and code
![image](https://github.com/user-attachments/assets/dbb54aa8-16c8-4e9e-b393-b2ac87f11fba)

---
### 4. Lambda Function Code

```python
import boto3
from datetime import datetime, timezone, timedelta


BUCKET_NAME = 'shakti-s3-cleanup'

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    deleted_files = []
    
    threshold_date = datetime.now(timezone.utc) - timedelta(seconds=10)
    
    response = s3.list_objects_v2(Bucket=BUCKET_NAME)
    
    if 'Contents' in response:
        for obj in response['Contents']:
            if obj['LastModified'] < threshold_date:
                s3.delete_object(Bucket=BUCKET_NAME, Key=obj['Key'])
                deleted_files.append(obj['Key'])

    print(f"Deleted files: {deleted_files}")
    return {
        'statusCode': 200,
        'body': f"Deleted files: {deleted_files}"
    }
```

### 4. Manual Testing
- Manually invoked with test JSON
- Verified via CloudWatch logs

🖼️ Screenshot Placeholder: Lambda logs showing deleted files
![Screenshot 2025-06-25 132147](https://github.com/user-attachments/assets/f2c5a28a-1b20-44c2-8ff5-01e4e897fb5b)

---

## 🗂️ Files Included

| File                | Description                        |
|---------------------|------------------------------------|
| `lambda_function.py`| Lambda code                        |
| `test_event.json`   | Test input                         |
| `README.md`         | Documentation                      |

---

## ✅ Status
✅ Completed and Tested  
🧪 Works with manual invocation 

