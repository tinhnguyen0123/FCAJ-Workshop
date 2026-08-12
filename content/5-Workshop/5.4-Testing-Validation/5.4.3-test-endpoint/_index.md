---
title : "Testing AWS Bedrock AI Story & Combat APIs"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
---

#### Testing AI Dungeon Master & Combat Resolution

In this step, you will test generating real-time AI story choices via **AWS Bedrock** and resolving turn-based combat encounters.

---

#### 1. Start Story Session (`POST /story/start`)

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
      "narrativeText": "You stand at the edge of the Whispering Woods. A rusted gate creaks open in front of an ancient crypt...",
      "choices": [
        {
          "label": "Option 1",
          "description": "Light a torch and step inside",
          "nextNodeId": "node_02"
        },
        {
          "label": "Option 2",
          "description": "Inspect the stone glyphs on the gate",
          "nextNodeId": "node_03"
        }
      ],
      "character": null,
      "triggerBattle": false,
      "bossId": null,
      "bossName": null,
      "bossLevel": null,
      "debugPrompt": null,
      "storyCompleted": false
    }
  }
  ```

  Save `sessionId` for the next request.

**Unity Client Code Reference** (`Assets/Scripts/API/StoryApiService.cs`):
```csharp
public async Task<StoryActionResponse> StartStoryAsync(string characterId, string storyFileId = "prologue")
{
    var body = new StoryStartBody { characterId = characterId, storyFileId = storyFileId };
    return await ApiClient.Instance.PostAsync<StoryActionResponse>("story/start", body);
}
```

---

#### 2. Execute Custom Story Action (`POST /story/action`)

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
      "currentNodeId": "node_02",
      "currentLocation": "Dark Forest Crypt",
      "narrativeText": "Your flame illuminates damp stone walls. Red eyes gleam from the shadows as a Skeleton Warrior draws its blade!",
      "choices": [],
      "character": null,
      "triggerBattle": true,
      "bossId": "boss-skeleton-01",
      "bossName": "Skeleton Warrior",
      "bossLevel": 1,
      "debugPrompt": null,
      "storyCompleted": false
    }
  }
  ```

  When `triggerBattle: true`, the Unity Client automatically transitions to `BattleScene.unity` and spawns the boss encounter.

**Unity Client Code Reference** (`Assets/Scripts/API/StoryApiService.cs`):
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

#### 3. Resolve Battle Action (`POST /battle/resolve`)

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
      "encounterId": "encounter-8888",
      "isPlayerVictory": true,
      "playerPower": 150.0,
      "bossPower": 80.0,
      "battleScore": 70.0,
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
          {
            "itemId": "item-sun-pendant",
            "itemName": "Sun Pendant",
            "rarity": "Rare",
            "quantity": 1
          }
        ]
      },
      "updatedCharacter": {
        "characterId": "char-uuid-1234",
        "name": "Valerius the Paladin",
        "level": 2,
        "experience": 100,
        "hp": 92,
        "maxHp": 100,
        "attack": 10,
        "defense": 5,
        "criticalRate": 0.05,
        "luckyRate": 0.05,
        "gold": 100,
        "className": "Paladin",
        "status": "Alive",
        "currentLocationId": "spawn_village"
      }
    }
  }
  ```

**Unity Client Code Reference** (`Assets/Scripts/API/BattleApiService.cs`):
```csharp
public async Task<string> ResolveBattleAsync(string characterId, string encounterId)
{
    var body = JsonUtility.ToJson(new ResolveBody { characterId = characterId, encounterId = encounterId });
    return await ApiClient.Instance.PostRawAsync("/battle/resolve", body);
}
```
