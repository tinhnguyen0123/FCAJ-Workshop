---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng & Triển khai Backend Game AI Dungeon RPG trên AWS

#### Tổng quan

**AI Dungeon RPG Adventure Game** là tựa game 2D RPG hiện đại kết hợp Generative AI (**AWS Bedrock**) cùng kiến trúc Serverless linh hoạt và tối ưu chi phí trên AWS (**.NET 8 Lambda**, **Amazon API Gateway**, **Amazon Cognito**, **Amazon DynamoDB**).

Trong bài workshop này, bạn sẽ học cách thiết lập, triển khai (IaC), kiểm thử và xác minh một backend Serverless hoàn chỉnh cho game tích hợp AI, sẵn sàng kết nối với Unity 2D Client.

#### Các điểm nổi bật

- **Tích hợp AWS Bedrock:** Sinh cốt truyện và xử lý trận đánh theo lượt (turn-based) thời gian thực bằng Mô hình ngôn ngữ lớn (LLM).
- **Kiến trúc Serverless:** Khả năng mở rộng tự động, chi phí theo lượng sử dụng (Pay-as-you-go) với AWS Lambda (.NET 8) và DynamoDB.
- **Bảo mật Server-Authoritative:** Xác thực người dùng qua AWS Cognito, tính toán chỉ số, máu và vật phẩm hoàn toàn ở phía Server để chống gian lận.
- **Infrastructure as Code (IaC):** Tự động hóa triển khai hạ tầng bằng AWS CDK (C#).

#### Nội dung

1. [Tổng quan về Workshop](5.1-Workshop-overview/)
2. [Các tiền đề cần chuẩn bị](5.2-Prerequiste/)
3. [Hướng dẫn triển khai chi tiết](5.3-Detail-guide/)
4. [Kiểm thử và Xác minh](5.4-Testing-Validation/)
5. [Dọn dẹp tài nguyên](5.5-Cleanup/)
6. [Tài liệu tham khảo](5.6-References/)