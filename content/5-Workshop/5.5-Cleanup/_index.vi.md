---
title : "Dọn dẹp Tài nguyên"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Hủy Hạ tầng AWS & Tối ưu Chi phí

Chúc mừng bạn đã hoàn thành bài workshop **AI Dungeon RPG Adventure Game**!

Để tránh phát sinh chi phí không mong muốn cho các tài nguyên AWS đã khởi tạo trong quá trình thực hành, hãy thực hiện các bước dọn dẹp dưới đây để gỡ bỏ toàn bộ Stack và tài nguyên liên quan.

---

#### Bước 1: Hủy CDK Infrastructure Stack

1. Mở Terminal và di chuyển tới thư mục `infrastructure`:
   ```bash
   cd infrastructure
   ```

2. Chạy lệnh hủy toàn bộ hạ tầng:
   ```bash
   cdk destroy --all
   ```

3. Xác nhận xóa bằng cách nhập `y` khi được hỏi.

---

#### Bước 2: Xóa CloudWatch Log Groups

1. Mở trang quản trị [AWS CloudWatch Console](https://console.aws.amazon.com/cloudwatch/).
2. Chọn **Log groups** ở menu bên trái.
3. Tìm kiếm các Log group có tên dạng `/aws/lambda/GameBackend-*` và `/aws/apigateway/*`.
4. Chọn các Log group tìm được, nhấn **Actions** và chọn **Delete log group(s)**.

---

#### Bước 3: Xác minh Dọn dẹp DynamoDB & Cognito Pool

1. Truy cập [Amazon DynamoDB Console](https://console.aws.amazon.com/dynamodb/) để đảm bảo các bảng (`Users`, `Characters`, `StorySessions`, `BossEncounters`) đã được xóa thành công.
2. Truy cập [Amazon Cognito Console](https://console.aws.amazon.com/cognito/) để xác nhận Cognito User Pool đã được giải phóng.