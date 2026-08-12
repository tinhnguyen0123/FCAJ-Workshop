---
title : "Các tiền đề cần chuẩn bị"
date : 2024-01-01 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### Yêu cầu Môi trường

Trước khi bắt đầu workshop, hãy đảm bảo bạn đã cài đặt và cấu hình các công cụ cũng như dịch vụ sau:

1. **Tài khoản AWS & Quyền truy cập Bedrock:**
   - Một tài khoản AWS đang hoạt động.
   - Đã kích hoạt Model Access trong trang quản trị **AWS Bedrock** cho các mô hình **Anthropic Claude 3 (Haiku/Sonnet)** hoặc **Amazon Nova Micro/Lite** tại Region bạn chọn (ví dụ: `us-east-1` hoặc `us-west-2`).

2. **Công cụ Phát triển:**
   - Cài đặt [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0).
   - Cài đặt [Node.js (v18+)](https://nodejs.org/) và **AWS CDK CLI**:
     ```bash
     npm install -g aws-cdk
     ```
   - Cài đặt và cấu hình [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) với IAM user/credentials của bạn:
     ```bash
     aws configure
     ```
   - Cài đặt [Unity 2022.3 LTS](https://unity.com/) (Tùy chọn, cần thiết nếu muốn test trực tiếp từ Unity Client).

#### Quyền IAM cần thiết

Tài khoản IAM sử dụng AWS CLI cần có quyền tối thiểu để khởi tạo tài nguyên thông qua AWS CDK:
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