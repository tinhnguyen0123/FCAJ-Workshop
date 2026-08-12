---
title : "Authentication & Character APIs"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

#### Testing Authentication & Character Creation

In this step, you will test user registration, login, JWT token retrieval from Amazon Cognito, and creating a new RPG character.

---

#### 1. Register User (`POST /auth/register`)

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

> **Note**: `errorCode: "USER_NOT_CONFIRMED"` is expected behavior. AWS Cognito sends a verification email to the registered address. Confirm the user via the **Cognito Console** (*User Pools → Users → Confirm User*) before proceeding to Login.

---

#### 2. Login User (`POST /auth/login`)

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

  Save the `token` (Access Token) as `{{ID_TOKEN}}` in your Postman environment for subsequent requests.

**Unity Client Code Reference** (`Assets/Scripts/API/AuthApiService.cs`):
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

#### 3. Create RPG Character (`POST /character`)

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

  Save `characterId` — you will need it for Story and Battle API calls.

**Unity Client Code Reference** (`Assets/Scripts/API/CharacterApiService.cs`):
```csharp
public async Task<string> CreateCharacterAsync(string userId, string name, string className)
{
    var body = JsonUtility.ToJson(new CreateBody { userId = userId, name = name, className = className });
    return await ApiClient.Instance.PostRawAsync("/character", body);
}
```