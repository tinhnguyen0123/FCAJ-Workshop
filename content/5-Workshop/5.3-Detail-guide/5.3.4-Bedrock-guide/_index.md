---
title : "AWS Bedrock & Backend Integration"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

#### Guide to Connecting and Configuring AWS Bedrock for the AI Dungeon Master

In this adventure RPG game, **Amazon Bedrock** (utilizing LLMs such as **Anthropic Claude 3 / 3.5** or **Amazon Nova Pro**) acts as the virtual Dungeon Master. It receives player actions, analyzes the game state, and dynamically generates narrative text along with subsequent choices for the player.

This document provides a step-by-step guide on how to request model access on the **AWS Console** and how to integrate and configure Bedrock in the **Backend Source Code**.

---

### PART 1: AWS Console Configuration

Before the Backend source code can call the AWS Bedrock APIs, your AWS account must be granted access to the corresponding model.

#### Step 1.1: Access the Amazon Bedrock Service
1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).
2. Select an AWS Region that supports Bedrock and is closest to your backend services (e.g., Singapore `ap-southeast-1` or N. Virginia `us-east-1`).
3. In the search box, type **Amazon Bedrock** and select the service from the results.

![Truy cập Amazon Bedrock](images/bedrock_overview.jpg)


---

#### Step 1.2: Request Model Access
1. On the left navigation pane, scroll down to the bottom and select **Model access** (under the *Playgrounds* section).
2. You will see a list of models provided by AWS. Locate the model used by the project:
   - **Amazon Nova Pro** (`apac.amazon.nova-pro-v1:0`)
   - **Anthropic Claude 3.5 Sonnet / Claude 3** (e.g., `anthropic.claude-sonnet-4-5-20250929-v1:0` or `anthropic.claude-3-sonnet-20240229-v1:0`)
3. Click the **Manage model access** button in the upper-right corner.
4. Check the checkboxes next to the models you want to enable (e.g., the models in the *Amazon* and *Anthropic* categories).
5. Click the **Save changes** or **Request model access** button.

![Yêu cầu quyền truy cập Model](images/model_catalog.jpg)

---

#### Step 1.3: Verify Access Status (Access Granted)
1. The activation process may take 1 to 5 minutes for third-party models (such as Anthropic Claude).
2. Refresh the page and check the **Access status** column. Once the status shows a green **Access granted**, you are ready to use the API.

![Xác nhận quyền truy cập thành công](images/access_model.jpg)


---

### PART 2: Source Code Configuration and Integration

Our codebase uses the AWS SDK for .NET to communicate with Amazon Bedrock. This configuration is divided into 4 key components:

#### Step 2.1: IAM Permissions in AWS CDK (Infrastructure)
For AWS Lambda functions to invoke Bedrock APIs, they must be assigned the appropriate IAM permissions via Infrastructure as Code (AWS CDK).

- **Configuration File:** [LambdaStack.cs](AI-Dungeon-RPG-Adventure-Game/infrastructure/src/Infrastructure/Stacks/LambdaStack.cs#L162-L176)
- **Explanation:**
  In the Lambda infrastructure stack, we define a `PolicyStatement` that grants Bedrock permissions (`bedrock:InvokeModel`, `bedrock:Converse`...) on all resources (`*`), and attach it to our two main story-processing functions: `StartStoryFunction` and `StoryActionFunction`. Additionally, since LLM requests take time to process, the timeout for these functions is set to **29 seconds** with a memory limit of **1024MB**.

```csharp
// Attaching IAM Policy in LambdaStack.cs
var bedrockPolicy = new Amazon.CDK.AWS.IAM.PolicyStatement(new Amazon.CDK.AWS.IAM.PolicyStatementProps
{
    Effect = Amazon.CDK.AWS.IAM.Effect.ALLOW,
    Actions = new[] { 
        "bedrock:InvokeModel", 
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:Converse",
        "bedrock:ConverseStream"
    },
    Resources = new[] { "*" }
});
StartStoryFunction.AddToRolePolicy(bedrockPolicy);
StoryActionFunction.AddToRolePolicy(bedrockPolicy);
```

---

#### Step 2.2: Parameters Configuration in `appsettings.json`
Bedrock region and model settings are defined in the application's JSON configuration file for easy tuning without recompilation.

- **Configuration File:** [appsettings.json](AI-Dungeon-RPG-Adventure-Game/backend/src/GameBackend.Handlers/appsettings.json)
- **Explanation:**
  Add a `"Bedrock"` section to the configuration file to specify the Model ID and text generation options (Temperature, TopP, MaxTokens):

```json
{
  "Bedrock": {
    "Region": "ap-southeast-1",
    "ModelId": "anthropic.claude-sonnet-4-5-20250929-v1:0",
    "Temperature": 0.7,
    "TopP": 0.9,
    "MaxTokens": 1000
  }
}
```

> [!TIP]
> During local development, you can set the environment variable `BEDROCK_USE_MOCK=true` to force the application to use simulated **Mock Responses** defined in the code. This speeds up feature development and saves Bedrock API usage costs.

---

#### Step 2.3: Registering Dependency Injection (DI)
The application registers the Bedrock client into the .NET Core Service Collection so that other services can reuse the client connections efficiently.

- **Configuration File:** [ServiceProviderBuilder.cs](AI-Dungeon-RPG-Adventure-Game/backend/src/GameBackend.Handlers/DependencyInjection/ServiceProviderBuilder.cs#L55-L64)
- **Explanation:**
  We bind the `BedrockOptions` class to the configuration section and register the AWS Bedrock Runtime Client (`IAmazonBedrockRuntime`) using the AWS SDK package for .NET Core:

```csharp
// Registering Configuration and Services in ServiceProviderBuilder.cs
services.Configure<BedrockOptions>(configuration.GetSection("Bedrock"));
services.AddAWSService<IAmazonBedrockRuntime>();
```

---

#### Step 2.4: Implementing the Service Call in `BedrockService`
This is the core business logic implementation that communicates with the AWS Bedrock Runtime service.

- **Configuration File:** [BedrockService.cs](AI-Dungeon-RPG-Adventure-Game/backend/src/GameBackend.Core/Services/BedrockService.cs#L43-L103)
- **Explanation:**
  We use the unified `ConverseAsync` API from the latest AWS SDK to submit message payloads. The service class is equipped with robust error handling: if an exception occurs during the actual API call, it seamlessly falls back to a static Mock Response to ensure the player's gameplay session is not interrupted.

```csharp
// Creating and sending ConverseRequest in BedrockService.cs
var converseRequest = new ConverseRequest
{
    ModelId = modelId,
    InferenceConfig = new InferenceConfiguration
    {
        Temperature = _options.Temperature,
        MaxTokens = _options.MaxTokens > 0 ? _options.MaxTokens : 1000,
        TopP = _options.TopP
    },
    Messages = new List<Message>
    {
        new Message
        {
            Role = ConversationRole.User,
            Content = new List<ContentBlock>
            {
                new ContentBlock { Text = userPrompt ?? string.Empty }
            }
        }
    }
};

if (!string.IsNullOrWhiteSpace(systemPrompt))
{
    converseRequest.System = new List<SystemContentBlock>
    {
        new SystemContentBlock { Text = systemPrompt }
    };
}

var response = await _client.ConverseAsync(converseRequest);
var text = response.Output?.Message?.Content?.FirstOrDefault()?.Text;
```

