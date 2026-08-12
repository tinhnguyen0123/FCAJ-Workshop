---
title : "Tổng quan về Workshop"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Tổng quan Kiến trúc

Kiến trúc của **AI Dungeon RPG Adventure Game** tách biệt hoàn toàn giữa Unity 2D Client và AWS Cloud Backend nhằm đảm bảo tính bảo mật, hiệu năng cao và tối ưu chi phí vận hành.

```text
[ Unity 2D Client ]
       │ (REST APIs + JWT Token)
       ▼
[ Amazon API Gateway ] ◄───► [ Amazon Cognito (User Pool) ]
       │
       ▼
[ AWS Lambda Functions (.NET 8) ]
   ├── Auth & Character Service
   ├── Story & Prompt Builder Service ◄───► [ AWS Bedrock (Claude / Nova) ]
   ├── Battle & Boss Combat Resolver
   └── Inventory & Item Management
       │
       ▼
[ Amazon DynamoDB Tables ]
```

#### Các thành phần chính

1. **Unity 2D Game Client:**
   - Giao diện người dùng cho Đăng nhập, Tạo/Chọn Nhân vật, Hội thoại Cốt truyện Động và Trận đánh theo lượt.
   - Chia sẻ C# DTOs và Domain Models với Backend thông qua thư viện `shared` (.NET Standard 2.1).

2. **Amazon API Gateway & Amazon Cognito:**
   - API Gateway đóng vai trò là điểm tiếp nhận duy nhất cho toàn bộ các endpoint game.
   - Amazon Cognito quản lý đăng ký, đăng nhập và cấp phát JWT token để bảo mật API.

3. **AWS Lambda (.NET 8):**
   - Xử lý logic Serverless hiệu năng cao cho quản lý nhân vật, túi đồ, tính toán sát thương trận đánh và xây dựng prompt cho AI.

4. **AWS Bedrock:**
   - Đóng vai trò là "Dungeon Master AI". Sinh cốt truyện sinh động, phân tích lựa chọn của người chơi và tạo diễn biến trận đánh theo thời gian thực.

5. **Amazon DynamoDB:**
   - Cơ sở dữ liệu NoSQL độ trễ thấp (vài miligiây) lưu trữ thông tin Người dùng, Nhân vật, Vật phẩm, Session Cốt truyện và Boss.

6. **AWS CDK (C#):**
   - Khai báo toàn bộ hạ tầng AWS dưới dạng mã nguồn (IaC) bằng C#, giúp triển khai nhanh chóng và nhất quán.