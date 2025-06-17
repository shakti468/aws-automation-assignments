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

