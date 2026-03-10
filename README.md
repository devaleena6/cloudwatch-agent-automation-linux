# CloudWatch Agent Automation for Linux

This project automates **CloudWatch monitoring setup for EC2 instances** using **AWS Lambda and AWS Systems Manager (SSM)**.

The automation performs the following tasks:

- Creates a custom AWS network
- Launches an EC2 instance
- Automatically installs the CloudWatch Agent
- Automatically creates CloudWatch alarms
- Sends email alerts using SNS

---

# AWS Services Used

- Amazon VPC
- Amazon EC2
- Amazon CloudWatch
- AWS Lambda
- Amazon SNS (Simple Notification Service)
- AWS Systems Manager (SSM)

---

# Project Architecture

```
EC2 Instance
     │
     ▼
Lambda Automation
     │
     ▼
SSM installs CloudWatch Agent
     │
     ▼
CloudWatch collects metrics
     │
     ▼
CloudWatch Alarm triggers
     │
     ▼
SNS sends Email Alert
```

---

# Step 1: Create VPC

Navigate to:

```
VPC → Create VPC
```

Configuration:

| Setting | Value |
|-------|------|
| Name | Monitoring-VPC |
| CIDR Block | 10.0.0.0/16 |
| Tenancy | Default |

Click **Create VPC**.

---

# Step 2: Create Subnet

Navigate to:

```
VPC → Subnets → Create Subnet
```

Configuration:

| Setting | Value |
|-------|------|
| Subnet Name | Public-Subnet |
| VPC | Monitoring-VPC |
| Availability Zone | ap-south-1a |
| CIDR | 10.0.1.0/24 |

---

# Step 3: Create Internet Gateway

Navigate to:

```
VPC → Internet Gateway → Create
```

Configuration:

| Setting | Value |
|-------|------|
| Name | Monitoring-IGW |

Attach the Internet Gateway to **Monitoring-VPC**.

---

# Step 4: Configure Route Table

Navigate to:

```
VPC → Route Tables
```

Add Route:

| Destination | Target |
|-------------|--------|
| 0.0.0.0/0 | Internet Gateway |

Associate Route Table with **Public-Subnet**.

---

# Step 5: Create Security Group

Navigate to:

```
EC2 → Security Groups → Create
```

Configuration:

| Setting | Value |
|-------|------|
| Name | Monitoring-SG |
| VPC | Monitoring-VPC |

Inbound Rules:

| Type | Port | Source |
|-----|------|--------|
| SSH | 22 | My IP |
| HTTP | 80 | Anywhere |
| HTTPS | 443 | Anywhere |

Outbound Rules:

Allow **All Traffic**

---

# Step 6: Launch EC2 Instance

Navigate to:

```
EC2 → Launch Instance
```

Configuration:

| Setting | Value |
|-------|------|
| Name | Monitoring-Server |
| AMI | Ubuntu 22.04 |
| Instance Type | t2.micro |
| Key Pair | monitoring-key.pem |

Networking:

| Setting | Value |
|-------|------|
| VPC | Monitoring-VPC |
| Subnet | Public-Subnet |
| Auto Assign Public IP | Enable |
| Security Group | Monitoring-SG |

Launch the instance.

---

# Step 7: Attach IAM Role to EC2

Navigate to:

```
IAM → Roles → Create Role
```

Select:

```
AWS Service → EC2
```

Attach policies:

```
AmazonSSMManagedInstanceCore
CloudWatchAgentServerPolicy
```

Role Name:

```
EC2-CloudWatch-Role
```

Attach the role:

```
EC2 → Instance → Actions → Security → Modify IAM Role
```

Select **EC2-CloudWatch-Role**.

---

# Step 8: Create SNS Alert Topic

Navigate to:

```
SNS → Topics → Create Topic
```

Configuration:

| Setting | Value |
|-------|------|
| Name | ServerAlert |
| Type | Standard |

Create Subscription:

| Setting | Value |
|-------|------|
| Protocol | Email |
| Endpoint | your-email@example.com |

Confirm the email subscription.

---

# Step 9: Create Lambda Function

Navigate to:

```
Lambda → Create Function
```

Configuration:

| Setting | Value |
|-------|------|
| Function Name | CloudWatch-Automation |
| Runtime | Python 3.11 |

Attach IAM policies:

```
AmazonSSMFullAccess
CloudWatchFullAccess
AmazonEC2ReadOnlyAccess
SNSFullAccess
```

---

# Step 10: Deploy Lambda Automation Script

Paste the following Python code in the Lambda function.

```python
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
        Dimensions=[
            {
                'Name': 'InstanceId',
                'Value': instance_id
            }
        ],
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
```

---

# Step 11: Test Lambda Function

Navigate to:

```
Lambda → Test
```

When executed, Lambda will:

- Install CloudWatch Agent
- Start monitoring
- Create CPU alarms

---

# Step 12: Verify CloudWatch Alarms

Navigate to:

```
CloudWatch → Alarms
```

You should see alarms like:

```
i-xxxxxxxxxxxx-HighCPU
```

---

# Expected Output

After automation:

- CloudWatch Agent installed automatically
- EC2 metrics monitored
- CPU alarms created automatically
- Email alerts sent when CPU usage exceeds 80%

---

# Future Improvements

- Add Memory monitoring
- Add Disk monitoring
- Automatically detect new EC2 instances
- Trigger Lambda using EventBridge

---

# Author

DevOps Cloud Automation Project
