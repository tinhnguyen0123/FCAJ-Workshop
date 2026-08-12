---
title : "Prepare Test Environment"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Setup API Testing Environment

Before making requests to the deployed backend, configure your testing tool of choice.

---

#### Option A: Postman / Insomnia / cURL

Create an environment in Postman with the outputs received from your CDK deployment:

- `BASE_URL`: `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod`
- `USER_POOL_ID`: `<cognito-user-pool-id>`
- `CLIENT_ID`: `<cognito-app-client-id>`
- `ID_TOKEN`: *(Will be populated after Login request)*

#### Setting Up Request Headers

For all protected routes (`/character`, `/story/*`, `/battle/*`, `/inventory/*`), add the Authorization header:

```http
Authorization: Bearer {{ID_TOKEN}}
Content-Type: application/json
```

---

#### Option B: Unity Client Setup

If you prefer to test via the Unity Client directly, follow these steps:

![GameConfigSO Inspector](images/unity_gameconfig_inspector.png)

1. Open Unity Editor and load the project.

2. In the **Project** panel, go to `Assets/Resources/` → select `GameConfig.asset`.

3. In the **Inspector** panel, configure the following fields:

   | Field | Value |
   |---|---|
   | **Api Base Url** | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/` |
   | **Aws Cognito User Pool Id** | `ap-southeast-1_xxxxx` |
   | **Aws Cognito Client Id** | `<your-cognito-app-client-id>` |
   | **Aws Cognito Region** | `ap-southeast-1` |
   | **Use Mock Mode** | ☐ *Unchecked* |
   | **Enable Api Logging** | ☑ *Checked* |

4. Press **Play** in Unity. The game will connect to your live AWS backend.

5. Monitor API requests in real-time via the **Unity Console** — all requests are logged with the `[ApiClient]` prefix:
   ```
   [ApiClient] → POST https://xxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/auth/login
   [ApiClient] → POST https://xxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/story/start
   ```

> **Mock Mode for Offline Testing**: Set `Use Mock Mode = true` to run the game without any AWS backend. The `MockAuthService` provides pre-defined test data for all game features — ideal for UI development.
