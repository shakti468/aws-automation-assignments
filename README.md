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


---

# Assignment 17: Restore EC2 Instance from Snapshot

## ✅ Objective

Automatically create a new EC2 instance from the latest snapshot of a specific volume using AWS Lambda and Boto3.

---

## 🧠 Logic Flow

1. Identify the latest snapshot of a given EBS volume
2. Create a new EBS volume from that snapshot
3. Launch a new EC2 instance using a known AMI ID
4. Attach the new volume to the new instance

---

## 🧩 Requirements

- Lambda with Python 3.12 runtime
- IAM role with `AmazonEC2FullAccess`
- An existing volume with snapshots
- A valid AMI ID (e.g., Amazon Linux 2)

---

## 🧪 Testing

- Triggered manually from Lambda Console
- Instance created: `i-06424e61e1bb35086`
- Volume used: `vol-0bd00d1e718d373bf`
- Snapshot used: `snap-0b2624e3f644491a4`

---

## 📸 Screenshots

> 🖼️ Screenshot 1: Lambda logs showing snapshot and volume creation
![image](https://github.com/user-attachments/assets/6ceec345-f822-4db3-8598-2404e11183d8)

> 🖼️ Screenshot 2: New EC2 instance in running state
![image](https://github.com/user-attachments/assets/4f4e63a2-62e5-4cd3-b634-f2b6698422b7)

> 🖼️ Screenshot 3: Code
![image](https://github.com/user-attachments/assets/166c4a35-b977-45df-9851-d41ac89453fc)

> 🖼️ Screenshot 3: Test
![image](https://github.com/user-attachments/assets/d37c0707-a00d-4a94-b179-1f755d120298)

---

## 🧼 Cleanup

- ✅ Terminate unused instances to save cost
- ✅ Optionally delete the volume created

---

## 📁 Files

- `lambda_function.py`: Boto3 script to restore from snapshot

  ### . Lambda Function Code

```python
import boto3
import datetime

ec2 = boto3.client('ec2')

def lambda_handler(event, context):
    # Replace with your volume ID
    volume_id = 'vol-0b61481bf6349739d'

    # Get latest snapshot for the volume
    snapshots = ec2.describe_snapshots(
        Filters=[
            {'Name': 'volume-id', 'Values': [volume_id]},
        ],
        OwnerIds=['self']
    )['Snapshots']

    if not snapshots:
        return {'status': 'No snapshots found.'}

    # Get latest snapshot
    latest_snapshot = sorted(snapshots, key=lambda x: x['StartTime'], reverse=True)[0]
    snapshot_id = latest_snapshot['SnapshotId']

    print(f"Using snapshot: {snapshot_id}")

    # Create a new volume from the snapshot
    availability_zone = 'ap-south-1a'  # Change as per your instance region
    new_volume = ec2.create_volume(
        SnapshotId=snapshot_id,
        AvailabilityZone=availability_zone,
        VolumeType='gp2',
        TagSpecifications=[
            {
                'ResourceType': 'volume',
                'Tags': [{'Key': 'Restored', 'Value': 'True'}]
            }
        ]
    )

    new_volume_id = new_volume['VolumeId']
    print(f"New volume created: {new_volume_id}")

    # Wait until volume is available
    waiter = ec2.get_waiter('volume_available')
    waiter.wait(VolumeIds=[new_volume_id])

    # Launch a new EC2 instance with this volume
    response = ec2.run_instances(
        ImageId='ami-0f918f7e67a3323f0',  # Amazon Linux 2 AMI (adjust as needed)
        InstanceType='t2.micro',
        MinCount=1,
        MaxCount=1,
        Placement={'AvailabilityZone': availability_zone},
        BlockDeviceMappings=[
            {
                'DeviceName': '/dev/xvda',
                'Ebs': {
                    'VolumeSize': 8,
                    'DeleteOnTermination': True,
                    'VolumeType': 'gp2',
                    'SnapshotId': snapshot_id
                }
            }
        ]
    )

    instance_id = response['Instances'][0]['InstanceId']
    print(f"New EC2 Instance launched: {instance_id}")

    return {
        'status': 'Success',
        'instance_id': instance_id,
        'volume_id': new_volume_id,
        'snapshot_id': snapshot_id
    }
```

