---
title : "Kiểm thử và Xác minh"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Tổng quan

Trong phần này, bạn sẽ tiến hành kiểm thử và xác minh toàn bộ các dịch vụ Backend của **AI Dungeon RPG Adventure Game** sau khi triển khai để đảm bảo tính năng Xác thực, Lưu trữ thông tin, Sinh cốt truyện AI qua Bedrock và Xử lý logic trận đánh hoạt động chính xác.

#### Kế hoạch Kiểm thử

1. **[5.4.1 Chuẩn bị Môi trường Kiểm thử](5.4.1-prepare/)**
   Cấu hình Postman / Insomnia hoặc Unity Client với URL API Gateway và Cognito credentials vừa tạo.

2. **[5.4.2 Kiểm thử API Xác thực & Tạo Nhân vật](5.4.2-auth-character-api/)**
   Xác minh các endpoint Đăng ký, Đăng nhập, cấp JWT Token và khởi tạo thông tin Nhân vật.

3. **[5.4.3 Kiểm thử Sinh Cốt truyện AI & Trận đánh với Bedrock](5.4.3-test-endpoint/)**
   Gửi hành động chơi game tới AWS Bedrock và kiểm tra phản hồi cốt truyện cùng logic tính toán combat theo lượt trong AWS Lambda.

4. **[5.4.4 Mô phỏng Luồng chơi Game End-to-End](5.4.4-e2e-game-simulation/)**
   Thực hiện mô phỏng một phiên chơi game hoàn chỉnh từ lúc Đăng nhập, chọn hành động AI, chiến đấu với Boss đến cập nhật vật phẩm vào DynamoDB.
