# cloudwatch-agent-automation-linux
To automate CloudWatch Alarm creation, you can use AWS Lambda written in Python. The Lambda function will automatically create alarms for an EC2 instance such as CPU, Memory, and Disk usage.
Services used in this architecture:

Amazon VPC

Amazon EC2

Amazon CloudWatch

AWS Lambda

Amazon Simple Notification Service

AWS Systems Manager

AWS CloudWatch Automation Project
Automated Monitoring with Lambda

This project demonstrates:

Creating a custom AWS network

Launching EC2 instances

Automatically installing CloudWatch Agent

Automatically creating CloudWatch alarms

Sending email alerts using SNS

Step 1: Create VPC

Open the console of
Amazon VPC

Navigate:

VPC → Create VPC

Configuration:

Name: Monitoring-VPC
CIDR Block: 10.0.0.0/16
Tenancy: Default

Click Create VPC

Step 2: Create Subnet

Navigate:

VPC → Subnets → Create Subnet

Configuration:

Subnet Name: Public-Subnet
VPC: Monitoring-VPC
Availability Zone: ap-south-1a
CIDR: 10.0.1.0/24
Step 3: Create Internet Gateway

Navigate:

VPC → Internet Gateway → Create

Configuration:

Name: Monitoring-IGW

Attach it to the VPC.

Step 4: Configure Route Table

Navigate:

VPC → Route Tables

Add route:

Destination: 0.0.0.0/0
Target: Internet Gateway

Associate:

Public-Subnet
Step 5: Create Security Group

Open
Amazon EC2

Navigate:

EC2 → Security Groups → Create

Configuration:

Name: Monitoring-SG
VPC: Monitoring-VPC

Inbound rules:

SSH – Port 22 – My IP
HTTP – Port 80 – Anywhere
HTTPS – Port 443 – Anywhere

Outbound:

Allow All Traffic
Step 6: Launch EC2 Instance

Navigate:

EC2 → Launch Instance

Configuration:

Name: Monitoring-Server
AMI: Ubuntu 22.04
Instance Type: t2.micro
Key Pair: monitoring-key.pem

Networking:

VPC: Monitoring-VPC
Subnet: Public-Subnet
Auto Assign Public IP: Enable
Security Group: Monitoring-SG

Launch instance.

Step 7: Attach IAM Role to EC2

For automation using
AWS Systems Manager
the instance needs permissions.

Navigate:

IAM → Roles → Create Role

Select:

AWS Service → EC2

Attach policies:

AmazonSSMManagedInstanceCore
CloudWatchAgentServerPolicy

Role Name:

EC2-CloudWatch-Role

Attach the role to EC2:

EC2 → Instance → Actions → Security → Modify IAM Role

Select:

EC2-CloudWatch-Role
Step 8: Create SNS Alert Topic

Open
Amazon Simple Notification Service

Navigate:

SNS → Topics → Create Topic

Configuration:

Name: ServerAlert
Type: Standard

Create subscription:

Protocol: Email
Endpoint: your-email@example.com

Confirm email.

Step 9: Create Lambda Function

Open
AWS Lambda

Navigate:

Lambda → Create Function

Configuration:

Function Name: CloudWatch-Automation
Runtime: Python 3.11

Create IAM role with permissions:

AmazonSSMFullAccess
CloudWatchFullAccess
AmazonEC2ReadOnlyAccess
SNSFullAccess
Step 10: Deploy Lambda Automation Script

Paste the Lambda code.

This code performs:

1️⃣ Install CloudWatch agent using SSM
2️⃣ Start agent
3️⃣ Create CPU alarms automatically

Example code:

import boto3

ec2 = boto3.client('ec2')
ssm = boto3.client('ssm')
cloudwatch = boto3.client('cloudwatch')

SNS_TOPIC_ARN = "your-sns-topic-arn"

def install_cloudwatch_agent(instance_id):

    commands = [
        "sudo apt update",
        "sudo apt install amazon-cloudwatch-agent -y",
        "sudo systemctl start amazon-cloudwatch-agent",
        "sudo systemctl enable amazon-cloudwatch-agent"
    ]

    ssm.send_command(
        InstanceIds=[instance_id],
        DocumentName="AWS-RunShellScript",
        Parameters={'commands': commands}
    )


def create_alarm(instance_id):

    cloudwatch.put_metric_alarm(
        AlarmName=f"{instance_id}-HighCPU",
        MetricName="CPUUtilization",
        Namespace="AWS/EC2",
        Statistic="Average",
        Period=300,
        EvaluationPeriods=1,
        Threshold=80,
        ComparisonOperator="GreaterThanThreshold",
        Dimensions=[{'Name': 'InstanceId','Value': instance_id}],
        AlarmActions=[SNS_TOPIC_ARN],
        Unit="Percent"
    )


def lambda_handler(event, context):

    instances = ec2.describe_instances()

    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:

            instance_id = instance['InstanceId']

            install_cloudwatch_agent(instance_id)
            create_alarm(instance_id)

    return "CloudWatch automation completed"

Deploy function.

Step 11: Test Lambda Function

Navigate:

Lambda → Test

When executed, Lambda will:

1 Install CloudWatch Agent
2 Start Monitoring
3 Create CPU Alarm

Verify alarms in
Amazon CloudWatch

Navigate:

CloudWatch → Alarms
Step 12: Final Architecture
EC2 Instance
     ↓
Lambda Automation
     ↓
SSM installs CloudWatch Agent
     ↓
CloudWatch collects metrics
     ↓
CloudWatch Alarm
     ↓
SNS Email Alert
Step 13: Expected Output

After automation:

CloudWatch agent installed automatically

CPU metrics monitored

Alarm created automatically

Email alerts sent when CPU > 80%
