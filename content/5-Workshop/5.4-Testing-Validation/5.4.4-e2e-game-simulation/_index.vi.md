---
title : "Mô phỏng Luồng chơi Game E2E"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### Mô phỏng Toàn bộ Vòng đời Chơi Game

Sau khi kiểm thử độc lập các endpoint API, bạn có thể thực hiện kiểm thử toàn bộ luồng chơi game hoàn chỉnh (End-to-End) trên Unity 2D Client.

---

#### Sơ đồ Tiến trình Chơi Game

```text
[1. Đăng nhập] ──► [2. Tạo Nhân vật] ──► [3. Bắt đầu Cốt truyện AI]
                                                  │
                                                  ▼
[5. Về Menu] ◄── [4. Đánh Boss & Nhận Loot] ◄───┘ (triggerBattle = true)
      │
      ├──► [Profile.unity]  — Xem chỉ số & trang bị nhân vật
      └──► [Shop.unity]     — Mua vật phẩm bằng Gold kiếm được
```

---

#### Luồng Chuyển Scene Unity

![Luồng Scene Unity](images/unity_scene_flow.png)

Game được tổ chức thành **10 Unity Scenes** với luồng điều hướng như sau:

| Từ Scene | Hành động | Đến Scene |
|---|---|---|
| `Login.unity` | Đăng nhập thành công | `Welcome.unity` |
| `Login.unity` | Nhấn "Đăng ký" | `Register.unity` |
| `Register.unity` | Đăng ký thành công | `Login.unity` |
| `Welcome.unity` | Tự động chuyển | `Menu.unity` |
| `Menu.unity` | Nhấn "Play" | `StoryScene.unity` |
| `Menu.unity` | Nhấn "Profile" | `Profile.unity` |
| `Menu.unity` | Nhấn "Shop" | `Shop.unity` |
| `StoryScene.unity` | Trigger trận đánh | `BattleScene.unity` |
| `BattleScene.unity` | Người chơi thắng | `WinBattle.unity` |
| `BattleScene.unity` | Người chơi thua | `LoseBattle.unity` |
| `WinBattle.unity` | Tiếp tục | `Menu.unity` |
| `LoseBattle.unity` | Thử lại | `Menu.unity` |

---

#### 1. Cấu hình Kết nối Unity Client

1. Mở **Unity Editor** và load project từ thư mục gốc repository.

2. Mở **Menu Scene**: `Assets/Scenes/Menu.unity`.

3. Trong panel **Project**, truy cập `Assets/Resources/` và chọn **`GameConfig.asset`**.

4. Trong panel **Inspector**, điền thông tin endpoint vừa deploy:

   | Field | Giá trị |
   |---|---|
   | **Api Base Url** | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/` |
   | **Aws Cognito User Pool Id** | `ap-southeast-1_xxxxx` |
   | **Aws Cognito Client Id** | `<your-cognito-app-client-id>` |
   | **Aws Cognito Region** | `ap-southeast-1` |
   | **Use Mock Mode** | ☐ *Bỏ chọn (OFF)* |
   | **Enable Api Logging** | ☑ *Chọn (ON)* |

5. Nhấn **Play** trong Unity Editor, bắt đầu từ `Login.unity`.

6. Đăng ký tài khoản → xác nhận qua Cognito Console → đăng nhập → khởi tạo nhân vật → bắt đầu trải nghiệm thế giới dungeon sinh động bởi AI!

---

#### 2. Các Điểm Kiểm tra Chính (Validation Criteria)

- **Thời gian Phản hồi AI:** Đoạn văn cốt truyện do AWS Bedrock sinh ra hoàn tất trong khoảng ~1.5 - 3 giây.
- **Tính chuẩn xác của Server (Server-Authoritative):** Kiểm tra chỉ số máu và vật phẩm rơi ra trên giao diện Unity khớp chính xác với dữ liệu lưu trong bảng **Amazon DynamoDB** (kiểm tra qua DynamoDB Console).
- **Kiểm tra Bảo mật:** Gửi request không kèm JWT Token hợp lệ trả về `401 Unauthorized`.
- **Kiểm thử Mock Mode:** Đặt `Use Mock Mode = true` trong `GameConfig.asset` và xác nhận game chạy offline với dữ liệu test từ `MockAuthService`.
- **Console Logs:** Với `Enable Api Logging = true`, xác nhận từng API call xuất hiện trong Unity Console:
  ```
  [ApiClient] → POST https://xxx.execute-api.ap-southeast-1.amazonaws.com/prod/story/start
  [ApiClient] → POST https://xxx.execute-api.ap-southeast-1.amazonaws.com/prod/battle/resolve
  ```