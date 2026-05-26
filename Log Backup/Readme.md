# AWS EC2 Log Backup to S3 using Lambda

## Overview
This project automatically backs up Linux log files from an EC2 instance to an S3 bucket using AWS Lambda, SSM, SNS, and EventBridge.

---

# AWS Services Used

- EC2
- Lambda
- S3
- SNS
- EventBridge
- IAM
- Systems Manager (SSM)

---

# Project Workflow

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

---

# Features

- Automatic S3 bucket creation
- Linux log backup automation
- Email notification using SNS
- Scheduled execution using EventBridge
- Serverless automation

---

# Architecture

EC2 → SSM → Lambda → S3 → SNS

---

# EC2 Setup

## Install AWS CLI

```bash
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
```

---

# IAM Roles

## EC2 IAM Role

Attach these policies:

- AmazonSSMManagedInstanceCore
- AmazonS3FullAccess

---

## Lambda IAM Role

Attach these policies:

- AmazonS3FullAccess
- AmazonSSMFullAccess
- AmazonEC2FullAccess
- AmazonSNSFullAccess

---

# Lambda Function Code

```python
import boto3

def lambda_handler(event, context):

    ssm = boto3.client('ssm', region_name='ap-south-1')

    instance_id = 'i-0bf3b2df9bb42f3c3'
    bucket_name = 'garambadalii008'
    log_path = "/var/log"

    command = f"aws s3 sync {log_path} s3://{bucket_name}/ec2-logs/{instance_id}/"

    response = ssm.send_command(
        InstanceIds=[instance_id],
        DocumentName="AWS-RunShellScript",
        Parameters={
            'commands': [command]
        }
    )

    print("Logs copied to S3")

    return {
        'statusCode': 200,
        'body': response['Command']['CommandId']
    }


```

---

# EventBridge Trigger

## Schedule Expression

```text
cron(0/5 * * * ? *)
```

This trigger runs Lambda every 5 minutes.

---

# SNS Configuration

## Create Topic

Topic Name:


BackupNotification

---

## Create Subscription

Protocol:

Email

Confirm subscription from email inbox.


# S3 Backup Path


s3://atul-cha-log-backup-12345/ec2-logs/i-0ed33c7e6d6b9ece0/


---

# Linux Log Path


/var/log/


---

# Project Output
- - Automatic S3 bucket creation
- Automatic Linux log backup
- S3 storage automation
- Email notification after successful backup



