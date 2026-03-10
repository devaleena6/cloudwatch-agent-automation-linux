# cloudwatch-agent-automation-linux
To automate CloudWatch Alarm creation, you can use AWS Lambda written in Python. The Lambda function will automatically create alarms for an EC2 instance such as CPU, Memory, and Disk usage.
This automation works with:
AWS Lambda
Amazon CloudWatch
Amazon EC2
Amazon Simple Notification Service

1.Prerequisites

Before running the Lambda function:

Create an SNS Topic for alerts.

Note the SNS Topic ARN.

Ensure the EC2 instance already has CloudWatch Agent installed (for memory & disk metrics).

Create a Lambda IAM role with permissions:

CloudWatchFullAccess

AmazonEC2ReadOnlyAccess

SNSFullAccess
