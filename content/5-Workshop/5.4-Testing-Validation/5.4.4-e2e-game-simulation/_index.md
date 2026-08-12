---
title : "End-to-End Game Flow Simulation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### Simulating Complete Game Gameplay

Now that all isolated endpoints have been verified, you can test the full game loop end-to-end either via Unity 2D Client or automated scripts.

---

#### Game Flow Execution Diagram

```text
[1. User Auth] ──► [2. Create Character] ──► [3. Start AI Dungeon Story]
                                                     │
                                                     ▼
[5. Sync Unity UI] ◄── [4. Win Boss Battle & Loot] ◄───┘ (Action Triggers Boss)
```

---

#### 1. Unity Client Integration Setup

1. Open Unity Editor with the project loaded (`Assets/` folder).
2. Open `Assets/Scenes/DemoMenu.unity`.
3. Locate `GameConfig.json` under `Assets/Resources/` and populate your deployed endpoint details:
   ```json
   {
     "ApiBaseUrl": "https://<api-id>.execute-api.us-east-1.amazonaws.com/prod",
     "UserPoolId": "us-east-1_xxxxx",
     "ClientId": "1h2j3k4l5m6n7o8p..."
   }
   ```
4. Press **Play** in Unity.
5. Register a new user, log in, create a character, and start playing the AI dungeon story!

---

#### 2. Verification Checkpoints

- **Latency Check:** AWS Bedrock dynamic text response should complete within ~1.5 - 3 seconds.
- **Server-Authoritative Combat:** Validate that health changes and loot items in Unity match the exact records stored in **Amazon DynamoDB**.
- **Security Check:** Attempting requests without a valid JWT Token returns `401 Unauthorized`.