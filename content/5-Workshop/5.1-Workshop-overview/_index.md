---
title : "Workshop Overview"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### Architecture Overview

The **AI Dungeon RPG Adventure Game** architecture decouples the Unity 2D Game Client from the AWS Cloud Backend to ensure security, high performance, and minimal operational overhead.

![Architecture](images/aws-architect.png)

#### Core AWS Components

1. **Unity 2D Game Client:**
   - User Interface for Authentication, Character Selection, Dynamic Story Dialogue, and Turn-based Battle.
   - Shares C# DTOs and Domain Models with the Backend via a `shared` C# class library (`GameShared.dll`).

2. **Amazon API Gateway & Amazon Cognito:**
   - API Gateway acts as the single entry point for all game endpoints.
   - Amazon Cognito manages user registration, login authentication, and issues JWT tokens.

3. **AWS Lambda (.NET 8):**
   - High-performance, serverless backend compute handling authentication, character state management, item inventory, battle resolution, and AI prompt building.

4. **AWS Bedrock:**
   - Serves as the AI Dungeon Master. Generates immersive storylines, dynamically evaluates player choices, and crafts vivid combat narratives in real-time.

5. **Amazon DynamoDB:**
   - Ultra-fast, single-digit millisecond latency NoSQL database storing Users, Characters, Items, Story Sessions, and Boss Encounters.

6. **AWS CDK (C#):**
   - Defines the entire cloud infrastructure as code in C#, enabling repeatable and reliable deployments.

---

#### Unity Client Architecture

The Unity Client is structured as a **C# Full-Stack Monorepo** sharing data models with the backend, and follows a clean **MVP (Model-View-Presenter)** architectural pattern.

![Unity Inspector - GameConfigSO](images/unity_gameconfig_inspector.png)

##### Unity Scene Structure

The game is organized into **10 Unity Scenes**, each serving a distinct purpose:

![List scenes](images/10-scene.png)

| Scene | Purpose |
|---|---|
| `Login.unity` | User login with Cognito authentication |
| `Register.unity` | New account registration |
| `Welcome.unity` | Welcome splash / loading screen |
| `Menu.unity` | Main hub — access to all game features |
| `Profile.unity` | Character stats, equipment, and history |
| `Shop.unity` | Item purchase and inventory management |
| `StoryScene.unity` | AI-driven dynamic dungeon narrative |
| `BattleScene.unity` | Turn-based boss combat |
| `WinBattle.unity` | Victory screen with loot rewards |
| `LoseBattle.unity` | Defeat screen with retry option |

##### MVP Pattern per Feature

Each major game feature follows the **Model-View-Presenter** pattern:

```text
Feature (e.g. Story)
├── StoryModel.cs       — Data structures (character state, session data)
├── StoryPresenter.cs   — Business logic, API calls, state management
└── StoryView.cs        — Unity UI components, animations, user input
```

| Feature | Model | Presenter | View |
|---|---|---|---|
| Story | `StoryModel.cs` | `StoryPresenter.cs` | `StoryView.cs` |
| Battle | `BattleModel.cs` | `BattleService.cs` | `BattleUI.cs` |
| Profile | `ProfileModel.cs` | `ProfilePresenter.cs` | `ProfileView.cs` |
| Inventory | `ItemData.cs` | `InventoryManager.cs` | `InventorySlotUI.cs` |

##### Shared Library Integration

The `GameShared` project (compiled as `.NET Standard 2.1`) is automatically synced to Unity via a PostBuild event:

```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Copy SourceFiles="$(OutputPath)GameShared.dll;$(OutputPath)GameShared.pdb"
        DestinationFolder="../Assets/Plugins"
        SkipUnchangedFiles="true" />
</Target>
```

This means Unity's `Assets/Plugins/GameShared.dll` always stays in sync with the backend's shared DTOs and Domain Models — **zero schema mismatch**.

##### Mock Mode vs Online Mode

The `GameConfigSO` ScriptableObject (`Assets/Resources/GameConfig.asset`) controls runtime behavior:

```csharp
// Assets/Scripts/Config/GameConfigSO.cs
[CreateAssetMenu(fileName = "GameConfig", menuName = "Game/GameConfig")]
public class GameConfigSO : ScriptableObject
{
    public string apiBaseUrl;           // AWS API Gateway URL
    public string awsCognitoUserPoolId; // Cognito User Pool ID
    public string awsCognitoClientId;   // Cognito App Client ID
    public string awsCognitoRegion;     // e.g. "ap-southeast-1"
    public bool   useMockMode;          // true = offline mock, false = live AWS
    public bool   enableApiLogging;     // Log all HTTP requests to Console
}
```

When `useMockMode = true`, the game uses `MockAuthService` with pre-defined test data — perfect for frontend development without a live backend.