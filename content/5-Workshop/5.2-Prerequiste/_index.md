---
title : "Prerequisites"
date : 2024-01-01 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### Environment Requirements

Before starting this workshop, ensure you have configured the following tools and services:

1. **AWS Account & Bedrock Access:**
   - An active AWS Account.
   - Model Access granted in **AWS Bedrock** console for **Anthropic Claude 3 (Haiku/Sonnet)** or **Amazon Nova Micro/Lite** models in your target region (e.g., `us-east-1` or `us-west-2`).

2. **Development Tools:**
   - [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) installed.
   - [Node.js (v18+)](https://nodejs.org/) and **AWS CDK CLI**:
     ```bash
     npm install -g aws-cdk
     ```
   - [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with your credentials:
     ```bash
     aws configure
     ```
   - [Unity 2022.3 LTS](https://unity.com/) (Optional, required if testing via Unity Client).

#### Required IAM Permissions

Your AWS CLI credentials must have adequate permissions to create resources via AWS CDK, including:
- `cloudformation:*`
- `lambda:*`
- `apigateway:*`
- `dynamodb:*`
- `cognito-idp:*`
- `iam:*`
- `bedrock:InvokeModel`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "lambda:*",
                "apigateway:*",
                "dynamodb:*",
                "cognito-idp:*",
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:PassRole",
                "bedrock:InvokeModel"
            ],
            "Resource": "*"
        }
    ]
}
```