---
title : "Authentication & Character APIs"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Kiểm thử Authentication & Tạo Nhân vật

Trong bước này, bạn sẽ kiểm thử đăng ký người dùng, đăng nhập, lấy JWT token từ Amazon Cognito và tạo mới một nhân vật RPG.

---

#### 1. Đăng ký Người dùng (`POST /auth/register`)

- **Request Body:**
  ```json
  {
    "username": "player1",
    "password": "Password123!",
    "email": "player1@example.com"
  }
  ```
- **Expected Response (200 OK):**
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

> **Lưu ý**: `errorCode: "USER_NOT_CONFIRMED"` là hành vi bình thường. AWS Cognito gửi email xác minh đến địa chỉ đã đăng ký. Xác nhận người dùng qua **Cognito Console** (*User Pools → Users → Confirm User*) trước khi thực hiện Đăng nhập.

---

#### 2. Đăng nhập Người dùng (`POST /auth/login`)

- **Request Body:**
  ```json
  {
    "username": "player1",
    "password": "Password123!"
  }
  ```
- **Expected Response (200 OK):**
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

  Lưu giá trị `token` (Access Token) vào biến `{{ID_TOKEN}}` trong Postman environment để dùng cho các request tiếp theo.

**Tham khảo Unity Client Code** (`Assets/Scripts/API/AuthApiService.cs`):
```csharp
public async Task<LoginResponseData> LoginAsync(string username, string password)
{
    var body = new LoginRequestData { username = username, password = password };
    string json = await ApiClient.Instance.PostRawAsync("/auth/login", JsonUtility.ToJson(body));
    if (json == null) return null;
    return JsonUtility.FromJson<LoginResponseData>(json);
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
- **Expected Response (201 Created):**
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

  Lưu lại `characterId` — bạn sẽ cần dùng cho các API call Story và Battle.

**Tham khảo Unity Client Code** (`Assets/Scripts/API/CharacterApiService.cs`):
```csharp
public async Task<string> CreateCharacterAsync(string userId, string name, string className)
{
    var body = JsonUtility.ToJson(new CreateBody { userId = userId, name = name, className = className });
    return await ApiClient.Instance.PostRawAsync("/character", body);
}
```