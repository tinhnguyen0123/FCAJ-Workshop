---
title : "Kiểm thử API Xác thực & Nhân vật"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Kiểm thử Xác thực & Khởi tạo Nhân vật

Trong bước này, bạn sẽ kiểm thử đăng ký, đăng nhập người dùng, lấy JWT token từ Amazon Cognito và tạo mới một nhân vật RPG.

---

#### 1. Đăng ký Tài khoản (`POST /auth/register`)

- **Request Body:**
  ```json
  {
    "username": "player1",
    "password": "Password123!",
    "email": "player1@example.com"
  }
  ```
- **Phản hồi mong đợi (200 OK):**
  ```json
  {
    "success": true,
    "message": "Registration successful",
    "data": {
      "token": null,
      "userId": null,
      "displayName": null,
      "expiresAt": 0,
      "refreshToken": null,
      "idToken": null,
      "errorCode": "USER_NOT_CONFIRMED"
    }
  }
  ```

---

#### 2. Đăng nhập Tài khoản (`POST /auth/login`)

- **Request Body:**
  ```json
  {
    "username": "player1",
    "password": "Password123!"
  }
  ```
- **Phản hồi mong đợi (200 OK):**
  ```json
  {
    "success": true,
    "message": "Login successful",
    "data": {
      "token": "eyJraWQiOiJ...",
      "userId": "user-uuid-1234",
      "displayName": "player1",
      "expiresAt": 1735689600,
      "refreshToken": "eyJjdHkiOiJ...",
      "idToken": "eyJraWQiOiJ...",
      "errorCode": null
    }
  }
  ```

---

#### 3. Tạo Nhân vật RPG (`POST /character`)

- **Headers:** `Authorization: Bearer {{ID_TOKEN}}`
- **Request Body:**
  ```json
  {
    "name": "Valerius the Paladin",
    "className": "Paladin"
  }
  ```
- **Phản hồi mong đợi (201 Created):**
  ```json
  {
    "success": true,
    "message": "Character created",
    "data": {
      "characterId": "char-uuid-1234",
      "name": "Valerius the Paladin",
      "level": 1,
      "experience": 0,
      "hp": 100,
      "maxHp": 100,
      "attack": 10,
      "defense": 5,
      "criticalRate": 0.05,
      "luckyRate": 0.05,
      "gold": 50,
      "className": "Paladin",
      "status": "Alive",
      "currentLocationId": "spawn_village"
    }
  }
  ```