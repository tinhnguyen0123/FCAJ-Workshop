---
title: "Provisioning & Managing Amazon DynamoDB Database Tier"
date: 2026-08-12
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

### Provisioning & Managing Amazon DynamoDB Database Tier

#### Overview

In the **AI Dungeon RPG Adventure Game**, **Amazon DynamoDB** serves as the primary NoSQL database tier. DynamoDB provides single-digit millisecond latency, seamless automatic scaling, and pay-per-request billing, making it ideal for persistent game state storage.

To support complex game mechanics—including character progression, item management, turn-based combat, and context-aware AI story sessions—the database architecture is divided into **10 dedicated tables**:

| Table Name | Partition Key (PK) | Sort Key (SK) / GSI | Stored Attributes & Description |
|---|---|---|---|
| `GameUsers` | `userId` (String) | - | Account credentials, password hash, email, registration timestamp |
| `GameCharacters` | `characterId` (String) | GSI: `userId` | Level, EXP, HP/MP, Attack, Defense, Gold, equipped item IDs |
| `GameInventory` | `inventoryId` (String) | GSI: `characterId` | Item ID, item type, quantity, stat modifiers, `IsEquipped` status |
| `GameStorySessions` | `sessionId` (String) | GSI: `characterId` | Current chapter, location ID, current turn counter, active flags |
| `GameStoryActions` | `actionId` (String) | GSI: `sessionId` | Player choice text, generated narrative, AI choices, timestamp |
| `GameBattles` | `battleId` (String) | GSI: `characterId` | Boss ID, Boss current HP, turn history, combat state (Active/Won/Lost) |
| `GameBosses` | `bossId` (String) | - | Boss name, base stats, elemental affinities, special skill list |
| `GameBossEncounters` | `encounterId` (String) | - | Location trigger mappings, spawn probabilities, level thresholds |
| `GameLootDrops` | `lootId` (String) | - | Drop tables, item drop chances, stat randomization ranges |
| `GameDefeatedBosses` | `characterId` (String) | `bossId` (String) | Defeated boss records, completion time, first-clear achievements |

---

### Step 1: Define DynamoDB Infrastructure in AWS CDK (C#)

The DynamoDB tables are defined as Infrastructure as Code (IaC) using **AWS CDK** in C#. Below is a snippet showing `GameStorySessions` table creation with On-Demand billing mode and Global Secondary Indexes (GSI):

```csharp
using Amazon.CDK.AWS.DynamoDB;
using Constructs;

namespace Infrastructure
{
    public class GameDatabaseStack
    {
        public Table StorySessionsTable { get; private set; }

        public GameDatabaseStack(Construct scope)
        {
            // Create GameStorySessions Table
            StorySessionsTable = new Table(scope, "GameStorySessionsTable", new TableProps
            {
                TableName = "GameStorySessions",
                PartitionKey = new Attribute { Name = "sessionId", Type = AttributeType.STRING },
                BillingMode = BillingMode.PAY_PER_REQUEST,
                RemovalPolicy = Amazon.CDK.RemovalPolicy.DESTROY // For dev environment
            });

            // Add Global Secondary Index for querying sessions by characterId
            StorySessionsTable.AddGlobalSecondaryIndex(new GlobalSecondaryIndexProps
            {
                IndexName = "CharacterId-Index",
                PartitionKey = new Attribute { Name = "characterId", Type = AttributeType.STRING },
                ProjectionType = ProjectionType.ALL
            });
        }
    }
}
```

---

### Step 2: Deploy DynamoDB Tables to AWS via AWS CDK

1. Open your terminal and navigate to the `infrastructure` directory:
   ```bash
   cd infrastructure
   ```

2. Synthesize CloudFormation templates to verify DynamoDB table schemas:
   ```bash
   cdk synth
   ```

3. Deploy only the DynamoDB database stack to provision the 10 tables on AWS (Region `ap-southeast-1 Singapore`):
   ```bash
   cdk deploy GameDatabaseStack
   ```

4. Confirm the deployment prompt (`y`). AWS CDK will create the 10 tables in parallel via CloudFormation.

---

### Step 3: C# Entity Data Model (`StorySession.cs`)

The C# Lambda backend interacts with DynamoDB using the AWS SDK Document Model / Object Persistence Model:

```csharp
using Amazon.DynamoDBv2.DataModel;

namespace GameBackend.Core.Models
{
    [DynamoDBTable("GameStorySessions")]
    public class StorySession
    {
        [DynamoDBHashKey("sessionId")]
        public string SessionId { get; set; } = Guid.NewGuid().ToString();

        [DynamoDBProperty("characterId")]
        public string CharacterId { get; set; } = string.Empty;

        [DynamoDBProperty("currentLocation")]
        public string CurrentLocation { get; set; } = "prologue";

        [DynamoDBProperty("currentChapterId")]
        public string CurrentChapterId { get; set; } = "chapter_1";

        [DynamoDBProperty("turnCounter")]
        public int TurnCounter { get; set; } = 0;

        [DynamoDBProperty("createdAt")]
        public string CreatedAt { get; set; } = DateTime.UtcNow.ToString("o");
    }
}
```

---

### Step 4: Persistence Logic (How Data Gets Saved to DynamoDB from Backend)

When a player performs an action in the Unity Client (e.g. selects a narrative choice or attacks a boss), the backend persists updated game state to DynamoDB via Repository services using `IDynamoDBContext`:

```csharp
using Amazon.DynamoDBv2.DataModel;
using GameBackend.Core.Models;

namespace GameBackend.Core.Repositories
{
    public class StoryRepository : IStoryRepository
    {
        private readonly IDynamoDBContext _context;

        public StoryRepository(IDynamoDBContext context)
        {
            _context = context;
        }

        // 1. Save or update active story session state
        public async Task SaveSessionAsync(StorySession session)
        {
            session.UpdatedAt = DateTime.UtcNow;
            await _context.SaveAsync(session);
        }

        // 2. Persist turn action history (Player input + Bedrock AI Response)
        public async Task SaveActionAsync(StoryAction action)
        {
            await _context.SaveAsync(action);
        }

        // 3. Retrieve session by characterId using Global Secondary Index (GSI)
        public async Task<StorySession?> GetSessionByCharacterIdAsync(string characterId)
        {
            var config = new DynamoDBOperationConfig
            {
                IndexName = "CharacterId-Index"
            };

            var search = _context.QueryAsync<StorySession>(characterId, config);
            var results = await search.GetRemainingAsync();
            return results.FirstOrDefault();
        }
    }
}
```

#### Data Persistence Execution Flow:
1. **Player Trigger:** Unity Client POSTs choice to API Gateway → `StoryActionFunction` (AWS Lambda).
2. **State Processing:** `StoryService` processes player choice and calls Bedrock `amazon.nova-pro-v1:0` to generate narrative and new choices.
3. **Repository Execution:** `StoryService` calls `_storyRepository.SaveSessionAsync(session)` and `SaveActionAsync(action)`.
4. **DynamoDB Write:** `DynamoDBContext.SaveAsync()` serializes the C# entity models and sends an HTTP `PutItem` / `UpdateItem` request to AWS DynamoDB tables (`GameStorySessions`, `GameStoryActions`).

---

### Step 5: Verify DynamoDB Tables in AWS Management Console

1. Log in to **AWS Management Console** and navigate to **Amazon DynamoDB** (Region: `ap-southeast-1 Singapore`).
2. Click **Tables** in the left navigation panel. Verify that all 10 `Game*` tables are listed with status `Active`.

![Amazon DynamoDB Tables List](images/5.3.3-1-dynamodb-tables.png)

3. Click on the **`GameStorySessions`** table to view details.

![DynamoDB GameStorySessions Table Details](images/5.3.3-2-dynamodb-table-detail.png)

4. Click **Explore items** in the left menu, select `GameStorySessions`, and inspect the live JSON item attributes saved by the game backend.

![DynamoDB Explore Items](images/5.3.3-3-dynamodb-explore-items.png)

---

### Result

You have successfully provisioned, integrated, and verified the **Amazon DynamoDB** database tier. The 10 NoSQL tables efficiently persist all game states, item inventories, combat turns, and AI story progression with low latency!
