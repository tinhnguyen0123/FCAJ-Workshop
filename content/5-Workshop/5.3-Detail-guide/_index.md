---
title : "Detailed Deployment Guide"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
alwaysopen : true
---

### Detailed Infrastructure & Backend Deployment Guide

#### Overview

Section **5.3 Detailed Deployment Guide** provides end-to-end instructions for compiling shared assets, building C# Lambda functions, and deploying the entire serverless infrastructure for the **AI Dungeon RPG Adventure Game** using AWS CDK.

The detailed guide is organized into **5 sub-modules**:

| Sub-Module | Title & Focus | Key Services & Artifacts |
|---|---|---|
| **[5.3.1](5.3.1-cognito-setup/)** | **[Setting Up AWS Cognito](5.3.1-cognito-setup/)** | User Pool, App Client, JWT Tokens, Auth Flow |
| **[5.3.2](5.3.2-lambda/)** | **[Building & Deploying AWS Lambda Backend (.NET 8)](5.3.2-lambda/)** | 13 C# Lambda Functions, `dotnet lambda package`, `GameLambdaStack` |
| **[5.3.3](5.3.3-dynamodb/)** | **[Provisioning & Managing Amazon DynamoDB Database Tier](5.3.3-dynamodb/)** | 10 NoSQL Tables, C# Entity Models, `SaveAsync` Persistence, `GameDatabaseStack` |
| **[5.3.4](5.3.4-Bedrock-guide/)** | **[AWS Bedrock & Backend Integration](5.3.4-Bedrock-guide/)** | `amazon.nova-pro-v1:0`, Model Access, Prompt Builder, DI Registration |
| **[5.3.5](5.3.5-Deployment/)** | **[Deploy Game to Web (WebGL, S3 & CloudFront)](5.3.5-Deployment/)** | Unity WebGL Build, S3 Bucket Hosting, CloudFront CDN Distribution |

---

### Step-by-Step Infrastructure & Backend Deployment

Follow these quick execution steps to compile the monorepo shared libraries, package the .NET 8 Lambda functions, and deploy the entire serverless infrastructure using AWS CDK (C#).

---

#### Step 1: Clone Repository & Build Shared Assets

1. Navigate to the workspace directory:
   ```bash
   cd AI-Dungeon-RPG-Adventure-Game
   ```

2. Build the shared C# class library (`GameShared`):
   ```bash
   dotnet build shared/GameShared.csproj -c Release
   ```
   *Note: This automatically syncs `GameShared.dll` and `GameShared.pdb` into `Assets/Plugins/` for Unity Client usage via the PostBuild event configured in the project file.*

---

#### Step 2: Build & Publish .NET 8 Lambda Backend

The backend is split into two projects:
- `GameBackend.Core` — Business logic, services, repositories
- `GameBackend.Handlers` — AWS Lambda entry points (depends on Core)

1. Navigate to the Handlers project folder:
   ```bash
   cd backend/src/GameBackend.Handlers
   ```

2. Restore dependencies and publish the Lambda deployment package:
   ```bash
   dotnet lambda package --configuration Release --output-package bin/Release/net8.0/deploy-package.zip
   ```
   *This command automatically builds `GameBackend.Core` as a dependency before packaging.*

---

#### Step 3: Bootstrap AWS CDK & Synthesize Stack

1. Navigate to the infrastructure folder:
   ```bash
   cd infrastructure
   ```

2. Bootstrap your target AWS environment (only required once per region/account):
   ```bash
   cdk bootstrap aws://<YOUR_ACCOUNT_ID>/ap-southeast-1
   ```

3. Synthesize CloudFormation templates to verify the stack definition:
   ```bash
   cdk synth
   ```

---

#### Step 4: Deploy Infrastructure Stack

1. Execute the CDK deployment command:
   ```bash
   cdk deploy --all
   ```

2. Once deployment completes, AWS CDK will output key resources in your terminal:
   - `CognitoUserPoolId` (e.g., `ap-southeast-1_xxxxx`)
   - `CognitoAppClientId` (e.g., `1h2j3k4l5m6n7o8p...`)
   - `ApiGatewayUrl` (e.g., `https://xxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/`)

3. Save these outputs — you will need them for configuring your Unity Client and API testing tools.

---

#### Step 5: Configure Unity Client

After deploying, connect the Unity Client to your live AWS backend:

![GameConfigSO Inspector](5.3.1-cognito-setup/images/unity_gameconfig_inspector.png)

1. Open **Unity Editor** and load the project (the `Assets/` folder of this repository).

2. In the **Project** panel, navigate to `Assets/Resources/` and select **`GameConfig.asset`**.

3. In the **Inspector** panel, fill in the values from your CDK deployment output:

   | Field | Value |
   |---|---|
   | **Api Base Url** | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/` |
   | **Api Timeout Seconds** | `30` |
   | **Aws Cognito User Pool Id** | `ap-southeast-1_xxxxx` |
   | **Aws Cognito Client Id** | `<your-cognito-app-client-id>` |
   | **Aws Cognito Region** | `ap-southeast-1` |
   | **Use Mock Mode** | ☐ *Unchecked (OFF)* |
   | **Enable Api Logging** | ☑ *Checked (ON) — recommended for testing* |

4. Press **Ctrl+S** to save the asset. The Unity Client is now connected to your live AWS backend.

5. Press **Play** in Unity Editor to launch the game and test the full flow.

> **Tip**: The `ApiClient.cs` script automatically loads `GameConfig.asset` on startup and attaches the JWT token to all API requests. If the asset is missing or `apiBaseUrl` is empty, it falls back to the hardcoded default URL defined in the script.