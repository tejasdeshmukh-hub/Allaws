AWS EC2 Log Backup to S3 using Lambda
Overview
This project automatically backs up Linux log files from an EC2 instance to an S3 bucket using AWS Lambda, SSM, SNS, and EventBridge.

AWS Services Used
EC2
Lambda
S3
SNS
EventBridge
IAM
Systems Manager (SSM)
Project Workflow
EventBridge Trigger
↓
Lambda Function
↓
SSM Command
↓
EC2 Instance
↓
S3 Bucket Backup
↓
SNS Email Notification

Features
Automatic S3 bucket creation
Linux log backup automation
Email notification using SNS
Scheduled execution using EventBridge
Serverless automation
Architecture
EC2 → SSM → Lambda → S3 → SNS

EC2 Setup
Install AWS CLI
sudo apt update -y

sudo apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install
Alternative way

sudo snap install aws-cli --classic


# Install SSM Agent

```bash
sudo snap install amazon-ssm-agent --classic

sudo systemctl enable snap.amazon-ssm-agent.amazon-ssm-agent.service

sudo systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service
IAM Roles
EC2 IAM Role
Attach these policies:

AmazonSSMManagedInstanceCore
AmazonS3FullAccess
Lambda IAM Role
Attach these policies:

AmazonS3FullAccess
AmazonSSMFullAccess
AmazonEC2FullAccess
AmazonSNSFullAccess
Lambda Function Code
import boto3
import time
from botocore.exceptions import ClientError

# AWS Clients
s3 = boto3.client('s3', region_name='ap-northeast-1')
ssm = boto3.client('ssm', region_name='ap-northeast-1')
sns = boto3.client('sns', region_name='ap-northeast-1')

# Wait until EC2 appears in SSM
def wait_for_ssm(instance_id):

    while True:

        response = ssm.describe_instance_information()

        ids = [
            i['InstanceId']
            for i in response['InstanceInformationList']
        ]

        if instance_id in ids:
            print("SSM Online")
            break

        print("Waiting for SSM...")
        time.sleep(15)

def lambda_handler(event, context):

    # Bucket Name
    bucket_name = 'atul-cha-log-backup-12345'

    # Region
    region = 'ap-northeast-1'

    # EC2 Instance ID
    instance_id = 'i-0ed33c7e6d6b9ece0'

    # SNS Topic ARN
    sns_topic = 'arn:aws:sns:ap-northeast-1:886274844949:BackupNotification'

    # Linux Log Path
    log_path = "/var/log/"

    # Create bucket if not exists
    try:

        s3.head_bucket(Bucket=bucket_name)

        print("Bucket already exists")

    except ClientError:

        s3.create_bucket(
            Bucket=bucket_name,
            CreateBucketConfiguration={
                'LocationConstraint': region
            }
        )

        print("Bucket created successfully")

    # Wait for SSM connection
    wait_for_ssm(instance_id)

    # Command to upload Linux logs to S3
    command = f"sudo aws s3 sync {log_path} s3://{bucket_name}/ec2-logs/{instance_id}/"

    # Send command to EC2
    response = ssm.send_command(
        InstanceIds=[instance_id],
        DocumentName="AWS-RunShellScript",
        Parameters={
            'commands': [command]
        }
    )

    print("Backup Command Sent")

    # Send SNS Email Notification
    sns.publish(
        TopicArn=sns_topic,
        Subject='Backup Success',
        Message='EC2 logs copied successfully to S3 bucket'
    )

    return {
        'statusCode': 200,
        'body': 'EC2 Logs Backup Completed Successfully'
    }
EventBridge Trigger
Schedule Expression
cron(0/5 * * * ? *)
This trigger runs Lambda every 5 minutes.

SNS Configuration
Create Topic
Topic Name:

BackupNotification

Create Subscription
Protocol:

Email

Confirm subscription from email inbox.

S3 Backup Path
s3://atul-cha-log-backup-12345/ec2-logs/i-0ed33c7e6d6b9ece0/

Linux Log Path
/var/log/

Project Output
Automatic S3 bucket creation
Automatic Linux log backup
S3 storage automation
Email notification after successful backup    give me like that step  with small changes
