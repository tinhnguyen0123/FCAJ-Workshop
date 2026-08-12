---
title : "Mô phỏng Luồng chơi Game E2E"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### Mô phỏng Toàn bộ Vòng đời Chơi Game

Sau khi kiểm thử độc lập các endpoint API, bạn có thể thực hiện kiểm thử toàn bộ luồng chơi game hoàn chỉnh (End-to-End) trên Unity 2D Client hoặc thông qua kịch bản kiểm thử tự động.

---

#### Sơ đồ Tiến trình Chơi Game

```text
[1. Đăng nhập] ──► [2. Tạo Nhân vật] ──► [3. Bắt đầu Cốt truyện AI]
                                                │
                                                ▼
[5. Cập nhật Unity UI] ◄── [4. Đánh Boss & Nhận Loot] ◄───┘ (Hành động gọi Boss)
```

---

#### 1. Cấu hình Kết nối Unity Client

1. Mở Unity Editor với project game (`Assets/`).
2. Mở Scene `Assets/Scenes/DemoMenu.unity`.
3. Mở file `GameConfig.json` trong thư mục `Assets/Resources/` và điền thông tin endpoint vừa deploy:
   ```json
   {
     "ApiBaseUrl": "https://<api-id>.execute-api.us-east-1.amazonaws.com/prod",
     "UserPoolId": "us-east-1_xxxxx",
     "ClientId": "1h2j3k4l5m6n7o8p..."
   }
   ```
4. Nhấn **Play** trong Unity.
5. Đăng ký tài khoản, đăng nhập, khởi tạo nhân vật và bắt đầu trải nghiệm thế giới dungeon sinh động bởi AI!

---

#### 2. Các Đểm Kiểm tra Chính (Validation Criteria)

- **Thời gian Phản hồi AI:** Đoạn văn cốt truyện do AWS Bedrock sinh ra hoàn tất trong khoảng ~1.5 - 3 giây.
- **Tính chuẩn xác của Server (Server-Authoritative):** Kiểm tra chỉ số máu và vật phẩm rơi ra trên giao diện Unity khớp chính xác với dữ liệu lưu trong bảng **Amazon DynamoDB**.
- **Kiểm tra Bảo mật:** Gửi request không kèm JWT Token hợp lệ trả về lại câu trả lời `401 Unauthorized`.