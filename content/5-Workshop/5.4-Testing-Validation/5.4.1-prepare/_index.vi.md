---
title : "Chuẩn bị Môi trường Kiểm thử"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Cấu hình Môi trường Kiểm thử API

Trước khi thực hiện các HTTP Request tới backend vừa triển khai, hãy cấu hình công cụ kiểm thử API (Postman / Insomnia / cURL) hoặc Unity Client.

#### Khởi tạo Biến môi trường (Environment Variables)

Tạo một Environment trong Postman dựa trên giá trị Output thu được từ lệnh `cdk deploy`:

- `BASE_URL`: `https://<api-id>.execute-api.<region>.amazonaws.com/prod`
- `USER_POOL_ID`: `<cognito-user-pool-id>`
- `CLIENT_ID`: `<cognito-app-client-id>`
- `ID_TOKEN`: *(Tự động lưu sau khi Đăng nhập thành công)*

#### Cấu hình Request Header

Đối với tất cả các route yêu cầu xác thực (`/character`, `/story/*`, `/battle/*`, `/inventory/*`), thêm header Authorization:

```http
Authorization: Bearer {{ID_TOKEN}}
Content-Type: application/json
```