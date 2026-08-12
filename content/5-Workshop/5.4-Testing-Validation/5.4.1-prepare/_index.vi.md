---
title : "Chuẩn bị Môi trường Kiểm thử"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Cấu hình Môi trường Kiểm thử API

Trước khi thực hiện các HTTP Request tới backend vừa triển khai, hãy cấu hình công cụ kiểm thử theo cách bạn muốn.

---

#### Lựa chọn A: Postman / Insomnia / cURL

Tạo một Environment trong Postman dựa trên giá trị Output thu được từ lệnh `cdk deploy`:

- `BASE_URL`: `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod`
- `USER_POOL_ID`: `<cognito-user-pool-id>`
- `CLIENT_ID`: `<cognito-app-client-id>`
- `ID_TOKEN`: *(Tự động lưu sau khi Đăng nhập thành công)*

#### Cấu hình Request Header

Đối với tất cả các route yêu cầu xác thực (`/character`, `/story/*`, `/battle/*`, `/inventory/*`), thêm header Authorization:

```http
Authorization: Bearer {{ID_TOKEN}}
Content-Type: application/json
```

---

#### Lựa chọn B: Cấu hình Unity Client

Nếu bạn muốn kiểm thử trực tiếp qua Unity Client, thực hiện theo các bước sau:

![GameConfigSO Inspector](images/unity_gameconfig_inspector.png)

1. Mở Unity Editor và load project.

2. Trong panel **Project**, truy cập `Assets/Resources/` → chọn `GameConfig.asset`.

3. Trong panel **Inspector**, cấu hình các field sau:

   | Field | Giá trị |
   |---|---|
   | **Api Base Url** | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/` |
   | **Aws Cognito User Pool Id** | `ap-southeast-1_xxxxx` |
   | **Aws Cognito Client Id** | `<your-cognito-app-client-id>` |
   | **Aws Cognito Region** | `ap-southeast-1` |
   | **Use Mock Mode** | ☐ *Bỏ chọn* |
   | **Enable Api Logging** | ☑ *Chọn* |

4. Nhấn **Play** trong Unity. Game sẽ kết nối với backend AWS thực của bạn.

5. Theo dõi API request theo thời gian thực trong **Unity Console** — tất cả request được log với prefix `[ApiClient]`:
   ```
   [ApiClient] → POST https://xxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/auth/login
   [ApiClient] → POST https://xxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/story/start
   ```

> **Mock Mode để kiểm thử Offline**: Đặt `Use Mock Mode = true` để chạy game mà không cần backend AWS. `MockAuthService` cung cấp dữ liệu test sẵn có cho tất cả tính năng game — lý tưởng cho việc phát triển giao diện.
