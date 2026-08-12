---
title : "Testing AWS Bedrock AI Story & Combat APIs"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Kiểm thử AI Dungeon Master & Tính toán Trận đánh

Trong bước này, bạn sẽ kiểm thử tạo lựa chọn cốt truyện AI theo thời gian thực qua **AWS Bedrock** và giải quyết các encounter turn-based.

---

#### 1. Bắt đầu Session Cốt truyện (`POST /story/start`)

- **Headers:** `Authorization: Bearer {{ID_TOKEN}}`
- **Request Body:**
  ```json
  {
    "characterId": "char-uuid-1234",
    "storyFileId": "chapter_1"
  }
  ```
- **Expected Response (200 OK):**
  ```json
  {
    "success": true,
    "message": "Success",
    "data": {
      "sessionId": "session-5566",
      "currentNodeId": "node_01",
      "currentLocation": "Dark Forest Crypt",
      "narrativeText": "You stand at the edge of the Whispering Woods...",
      "choices": [
        { "label": "Option 1", "description": "Light a torch and step inside", "nextNodeId": "node_02" },
        { "label": "Option 2", "description": "Inspect the stone glyphs on the gate", "nextNodeId": "node_03" }
      ],
      "triggerBattle": false,
      "storyCompleted": false
    }
  }
  ```

  Lưu lại `sessionId` để dùng cho request tiếp theo.

**Tham khảo Unity Client Code** (`Assets/Scripts/API/StoryApiService.cs`):
```csharp
public async Task<StoryActionResponse> StartStoryAsync(string characterId, string storyFileId = "prologue")
{
    var body = new StoryStartBody { characterId = characterId, storyFileId = storyFileId };
    return await ApiClient.Instance.PostAsync<StoryActionResponse>("story/start", body);
}
```

---

#### 2. Gửi Hành động Cốt truyện (`POST /story/action`)

- **Headers:** `Authorization: Bearer {{ID_TOKEN}}`
- **Request Body:**
  ```json
  {
    "characterId": "char-uuid-1234",
    "sessionId": "session-5566",
    "playerInput": "I light my torch and call out into the darkness."
  }
  ```
- **Expected Response (200 OK):**
  ```json
  {
    "success": true,
    "message": "Success",
    "data": {
      "sessionId": "session-5566",
      "narrativeText": "Your flame illuminates damp stone walls. Red eyes gleam from the shadows!",
      "choices": [],
      "triggerBattle": true,
      "bossId": "boss-skeleton-01",
      "bossName": "Skeleton Warrior",
      "bossLevel": 1,
      "storyCompleted": false
    }
  }
  ```

  Khi `triggerBattle: true`, Unity Client tự động chuyển sang `BattleScene.unity` và spawn boss encounter.

**Tham khảo Unity Client Code** (`Assets/Scripts/API/StoryApiService.cs`):
```csharp
public async Task<StoryActionResponse> SendActionAsync(string characterId, string sessionId,
    int choiceIndex, string playerInput)
{
    var body = new StoryActionBody
    {
        characterId = characterId,
        sessionId = sessionId,
        choiceIndex = choiceIndex,
        playerInput = playerInput
    };
    return await ApiClient.Instance.PostAsync<StoryActionResponse>("story/action", body);
}
```

---

#### 3. Giải quyết Trận đánh (`POST /battle/resolve`)

- **Headers:** `Authorization: Bearer {{ID_TOKEN}}`
- **Request Body:**
  ```json
  {
    "characterId": "char-uuid-1234",
    "encounterId": "encounter-8888"
  }
  ```
- **Expected Response (200 OK):**
  ```json
  {
    "success": true,
    "message": "Success",
    "data": {
      "battleId": "battle-uuid-9999",
      "isPlayerVictory": true,
      "playerPower": 150.0,
      "bossPower": 80.0,
      "luckyEffects": ["Critical Hit"],
      "turns": [
        {
          "attackerName": "Valerius",
          "logMessage": "Valerius deals 35 damage!",
          "damage": 35,
          "playerHpRemaining": 92,
          "bossHpRemaining": 0,
          "isCritical": true
        }
      ],
      "rewards": {
        "goldEarned": 50,
        "expEarned": 100,
        "lootItems": [
          { "itemId": "item-sun-pendant", "itemName": "Sun Pendant", "rarity": "Rare", "quantity": 1 }
        ]
      },
      "updatedCharacter": {
        "characterId": "char-uuid-1234",
        "level": 2,
        "experience": 100,
        "hp": 92,
        "gold": 100
      }
    }
  }
  ```

**Tham khảo Unity Client Code** (`Assets/Scripts/API/BattleApiService.cs`):
```csharp
public async Task<string> ResolveBattleAsync(string characterId, string encounterId)
{
    var body = JsonUtility.ToJson(new ResolveBody { characterId = characterId, encounterId = encounterId });
    return await ApiClient.Instance.PostRawAsync("/battle/resolve", body);
}
```
