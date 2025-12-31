# Event-Driven Data Processing Pipeline

A fully automated, serverless data processing pipeline built on AWS using Infrastructure as Code (Terraform) and CI/CD (GitHub Actions).

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [Deployment](#deployment)
- [Usage](#usage)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

This project implements an event-driven data processing pipeline that:
- Automatically processes data files uploaded to S3
- Transforms and validates data using AWS Lambda
- Stores processed data and metadata in S3 and DynamoDB
- Generates automated daily summary reports
- Sends email notifications with report summaries

## 🏗️ Architecture

```
External Sources → S3 (Raw) → SQS → Lambda (Processor) → S3 (Processed) + DynamoDB
                                                               ↓
EventBridge (Daily) → Lambda (Reporter) → S3 (Reports) → SNS (Email)
```

### Key Components

- **Amazon S3**: Storage for raw data, processed data, and reports
- **Amazon SQS**: Message queue for reliable event handling
- **AWS Lambda**: Serverless compute for data processing and reporting
- **Amazon DynamoDB**: NoSQL database for metadata and tracking
- **EventBridge**: Scheduled trigger for daily reports
- **Amazon SNS**: Email notifications
- **CloudWatch**: Logging and monitoring

## ✨ Features

### Data Processing
- ✅ Event-driven architecture (processes files as they arrive)
- ✅ Automatic scaling based on workload
- ✅ Data validation and transformation
- ✅ Error handling with Dead Letter Queue
- ✅ Metadata tracking in DynamoDB

### Reporting
- ✅ Automated daily summary reports
- ✅ JSON and HTML report formats
- ✅ Email notifications via SNS
- ✅ Historical report storage

### Infrastructure
- ✅ 100% Infrastructure as Code (Terraform)
- ✅ Multi-environment support (dev, staging, prod)
- ✅ Automated CI/CD pipeline (GitHub Actions)
- ✅ Security scanning (Checkov, TruffleHog)

### Monitoring
- ✅ CloudWatch Logs for all Lambda functions
- ✅ CloudWatch Metrics and Alarms
- ✅ X-Ray tracing
- ✅ Custom CloudWatch Dashboard

## 📦 Prerequisites

- **AWS Account** with appropriate permissions
- **Terraform** >= 1.0
- **Python** >= 3.11
- **Git** and **GitHub** account
- **AWS CLI** configured with credentials

## 📁 Project Structure

```
event-driven-pipeline/
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline configuration
├── lambda/
│   ├── processor/
│   │   ├── index.py           # Data processor Lambda function
│   │   └── requirements.txt
│   └── reporter/
│       ├── index.py           # Report generator Lambda function
│       └── requirements.txt
├── tests/
│   ├── test_processor.py      # Unit tests for processor
│   ├── test_reporter.py       # Unit tests for reporter
│   └── smoke_test.py          # Post-deployment smoke tests
├── terraform/
│   ├── main.tf                # Main Terraform configuration
│   ├── variables.tf           # Input variables
│   ├── outputs.tf             # Output values
│   ├── providers.tf           # Provider configuration (optional)
│   └── terraform.tfvars       # Variable values (gitignored)
├── docs/
│   ├── architecture.md        # Detailed architecture documentation
│   └── deployment-guide.md    # Step-by-step deployment guide
├── .gitignore
├── README.md
└── LICENSE
```

## 🚀 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/event-driven-pipeline.git
cd event-driven-pipeline
```

### 2. Configure AWS Credentials

```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and Region
```

### 3. Create terraform.tfvars

Create a file `terraform.tfvars` with your configuration:

```hcl
aws_region         = "us-east-1"
environment        = "dev"
owner              = "Your Name"
notification_email = "your-email@example.com"
```

### 4. Initialize Terraform

```bash
terraform init
```

### 5. Review Infrastructure Plan

```bash
terraform plan
```

### 6. Deploy Infrastructure

```bash
terraform apply
```

Type `yes` when prompted to confirm deployment.

## 🔄 Deployment

### Manual Deployment

```bash
# Initialize Terraform
terraform init

# Plan changes
terraform plan -out=tfplan

# Apply changes
terraform apply tfplan
```

### CI/CD Deployment

The project includes a GitHub Actions workflow that automatically:

1. **On Pull Request:**
   - Lints code
   - Runs security scans
   - Executes unit tests
   - Creates Terraform plan
   - Comments plan on PR

2. **On Merge to Main:**
   - Deploys infrastructure to production
   - Runs post-deployment tests
   - Sends notifications

#### Setting Up CI/CD

1. **Fork the Repository**

2. **Configure GitHub Secrets:**
   Go to Settings → Secrets → Actions and add:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `NOTIFICATION_EMAIL`

3. **Push to Main Branch:**
   ```bash
   git push origin main
   ```

## 📊 Usage

### Uploading Data Files

Upload CSV files to the raw data bucket:

```bash
aws s3 cp your-data.csv s3://event-pipeline-raw-data-dev/uploads/
```

The pipeline will automatically:
1. Detect the new file
2. Send event to SQS queue
3. Process file with Lambda
4. Store results in processed bucket
5. Update metadata in DynamoDB

### Monitoring Processing

Check CloudWatch Logs:

```bash
# Processor logs
aws logs tail /aws/lambda/event-pipeline-processor-dev --follow

# Reporter logs
aws logs tail /aws/lambda/event-pipeline-reporter-dev --follow
```

### Viewing Reports

Reports are stored in S3:

```bash
aws s3 ls s3://event-pipeline-reports-dev/daily-reports/ --recursive
```

Download a report:

```bash
aws s3 cp s3://event-pipeline-reports-dev/daily-reports/2025/12/31/report_20251231_060000.html .
```

### Manual Report Generation

Trigger report generation manually:

```bash
aws lambda invoke \\
  --function-name event-pipeline-reporter-dev \\
  --payload '{}' \\
  response.json
```

## 📈 Monitoring

### CloudWatch Dashboard

Access the dashboard:
1. Go to AWS Console → CloudWatch
2. Select "Dashboards"
3. Open `event-pipeline-dev`

### CloudWatch Alarms

The following alarms are configured:
- **Lambda Errors**: Alerts when processor has > 5 errors in 5 minutes
- **DLQ Messages**: Alerts when messages appear in Dead Letter Queue

### Metrics to Monitor

- Lambda invocations and errors
- Lambda duration and memory usage
- SQS queue depth
- DynamoDB read/write capacity
- S3 bucket size

### Accessing Logs

```bash
# View recent processor logs
aws logs tail /aws/lambda/event-pipeline-processor-dev --since 1h

# View recent reporter logs
aws logs tail /aws/lambda/event-pipeline-reporter-dev --since 1h

# Search logs for errors
aws logs filter-log-events \\
  --log-group-name /aws/lambda/event-pipeline-processor-dev \\
  --filter-pattern "ERROR"
```

## 🐛 Troubleshooting

### Files Not Processing

1. **Check SQS Queue:**
   ```bash
   aws sqs get-queue-attributes \\
     --queue-url $(terraform output -raw sqs_queue_url) \\
     --attribute-names ApproximateNumberOfMessages
   ```

2. **Check Lambda Logs:**
   ```bash
   aws logs tail /aws/lambda/event-pipeline-processor-dev --follow
   ```

3. **Check Dead Letter Queue:**
   ```bash
   aws sqs receive-message \\
     --queue-url $(terraform output -raw dlq_url)
   ```

### Lambda Timeout Issues

Increase timeout in `variables.tf`:

```hcl
variable "lambda_timeout" {
  default = 600  # 10 minutes
}
```

Then reapply:

```bash
terraform apply
```

### Out of Memory Errors

Increase memory in `variables.tf`:

```hcl
variable "lambda_memory_size" {
  default = 1024  # 1 GB
}
```

### Reports Not Generating

1. **Verify EventBridge Rule:**
   ```bash
   aws events list-rules --name-prefix event-pipeline-daily-report
   ```

2. **Manually Trigger Report:**
   ```bash
   aws lambda invoke \\
     --function-name event-pipeline-reporter-dev \\
     --payload '{}' \\
     response.json
   ```

3. **Check Reporter Logs:**
   ```bash
   aws logs tail /aws/lambda/event-pipeline-reporter-dev --follow
   ```

## 🔐 Security Best Practices

- ✅ All S3 buckets have versioning enabled
- ✅ Lambda functions use IAM roles (no hardcoded credentials)
- ✅ Secrets managed via AWS Secrets Manager
- ✅ All data encrypted at rest (S3, DynamoDB)
- ✅ All data encrypted in transit (HTTPS)
- ✅ Security scanning in CI/CD pipeline
- ✅ CloudTrail enabled for audit logging

## 💰 Cost Optimization

- Lambda uses reserved concurrency to prevent runaway costs
- S3 lifecycle policies move old reports to Glacier after 90 days
- DynamoDB uses on-demand billing (pay per request)
- CloudWatch Logs retention set to 30 days

**Estimated Monthly Cost (dev environment):**
- Lambda: $5-10
- S3: $1-5
- DynamoDB: $1-5
- SQS: $0-1
- **Total: $7-21/month** (assuming moderate usage)

## 🧹 Cleanup

To destroy all resources:

```bash
# Empty S3 buckets first
aws s3 rm s3://event-pipeline-raw-data-dev --recursive
aws s3 rm s3://event-pipeline-processed-data-dev --recursive
aws s3 rm s3://event-pipeline-reports-dev --recursive

# Destroy infrastructure
terraform destroy
```

## 📚 Additional Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## 📝 License

MIT License - See LICENSE file for details

## 👥 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📧 Support

For issues or questions:
- Open a GitHub issue
- Email: your-email@example.com

---

**Built with ❤️ for MegaMinds IT Services**


# just for testing 1 2