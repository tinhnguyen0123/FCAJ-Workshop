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
   - Model Access granted in **AWS Bedrock** console for **Anthropic Claude 3 (Haiku/Sonnet)** or **Amazon Nova Micro/Lite** models in your target region (`ap-southeast-1` recommended).
   - Navigate to: *AWS Console → Bedrock → Model Access → Request Access*.

2. **Development Tools:**
   - [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) installed.
   - [Node.js (v18+)](https://nodejs.org/) and **AWS CDK CLI**:
     ```bash
     npm install -g aws-cdk
     ```
   - [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with your credentials:
     ```bash
     aws configure
     # AWS Access Key ID: <your-key>
     # AWS Secret Access Key: <your-secret>
     # Default region name: ap-southeast-1
     # Default output format: json
     ```

3. **Unity Client (Optional — for E2E testing):**
   - [Unity 2022.3 LTS](https://unity.com/) or newer (via Unity Hub).
   - Required Unity packages (already included in project):
     - **Input System** (`com.unity.inputsystem`)
     - **TextMesh Pro** (`com.unity.textmeshpro`)
   - The project is located in `Assets/` — open it in Unity Hub by adding the repository root as an existing project.

---

#### Required IAM Permissions

Your AWS CLI credentials must have adequate permissions to create resources via AWS CDK, including:
- `cloudformation:*`
- `lambda:*`
- `apigateway:*`
- `dynamodb:*`
- `cognito-idp:*`
- `iam:*`
- `bedrock:InvokeModel`
- `s3:*` *(for CDK bootstrap bucket and story template storage)*

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
                "bedrock:InvokeModel",
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```

---

#### Important Notes

> **Cognito Email Confirmation**: By default, AWS Cognito requires email confirmation before a user can log in. After registering via the game or API, you may need to confirm the user via the Cognito Console (*User Pools → Users → Confirm User*) or configure auto-confirmation in your CDK stack for testing purposes.

> **Region**: This workshop uses `ap-southeast-1` (Singapore) as the default deployment region. You may use `us-east-1` or `us-west-2` if you prefer, but ensure Bedrock model access is granted in your chosen region.