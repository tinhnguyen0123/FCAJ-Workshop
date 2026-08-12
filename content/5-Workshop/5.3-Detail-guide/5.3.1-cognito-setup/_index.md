---
title : "Setting Up AWS Cognito"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

#### What is AWS Cognito and Why Does This Game Use It?

Think of **Amazon Cognito** as the game's security guard — it handles everything related to **who you are** before you're allowed to touch any game data. When a player opens the app and types in their username and password, Cognito is the one that checks those credentials, and if everything checks out, hands back a **JWT token** (a signed digital pass). From that point on, every single API call the Unity Client makes carries that token, and API Gateway validates it automatically before forwarding the request to Lambda.

Without Cognito, you'd either have to build your own authentication system from scratch (risky) or store passwords in DynamoDB yourself (very bad practice). Cognito handles it all — password hashing, token expiry, refresh tokens, email verification — securely and at scale.

In this project, Cognito is **created automatically by the CDK stack** (`ApiStack.cs`). You don't need to click anything manually. But understanding what it creates and how it works is essential for debugging and configuring the Unity Client correctly.

---

#### What CDK Creates for You

When you run `cdk deploy GameLambdaStack`, the infrastructure stack automatically provisions:

| Resource | Name / Format | Purpose |
|---|---|---|
| **User Pool** | `GameUserPool` | The main user directory — stores all registered player accounts |
| **App Client** | `GameUserPoolClient` | The "key" your app uses to communicate with the User Pool — no client secret (suitable for mobile/game clients) |
| **User Pool Domain** | *(optional)* | Hosted UI for OAuth flows (not used in this project — we use direct API auth) |

After `cdk deploy`, your terminal will print:

```
Outputs:
GameStack.CognitoUserPoolId     = ap-southeast-1_AbCdEfGhI
GameStack.CognitoAppClientId    = 1a2b3c4d5e6f7g8h9i0j1k2l3m
GameStack.ApiGatewayUrl         = https://ne6hi09ope.execute-api.ap-southeast-1.amazonaws.com/prod/
```

**Save these three values** — you will paste them into `GameConfig.asset` in Unity.

---

#### How the Authentication Flow Works

Here's the full picture of what happens when a player logs in:

![Cognito Flow](images/cognito-flow.png)

---

#### The USER_NOT_CONFIRMED Situation

When you register a new account (`POST /auth/register`), Cognito creates the user but **puts them in a "UNCONFIRMED" state by default**. The user won't be able to log in until their account is confirmed.

By default, Cognito sends a **verification email** to the registered address. Once the user clicks the link (or enters the code), the account becomes `CONFIRMED` and login works.

![Cognito Register](images/cognito-register.png)
![Cognito Verification](images/cognito-verification.png)
![Cognito Status](images/cognito-status.png)

**For testing purposes**, you have two options to bypass this:

---

##### Option 1: Manually Confirm via Cognito Console *(Quickest for testing)*

1. Open the [Amazon Cognito Console](https://console.aws.amazon.com/cognito/).
2. Click on **User Pools** in the left sidebar.
3. Select your User Pool (named `GameUserPool` or similar).
![status0](images/cognito-change-status-0.png)
4. Click the **Users** tab.
5. Find the user you registered (search by username or email).
![status1](images/cognito-change-status-1.png)
6. Click the username to open the user detail page.
![status2](images/cognito-change-status-2.png)
7. Click **Actions** → **Confirm user**.

The user's status changes from `UNCONFIRMED` to `CONFIRMED` and they can now log in.

---

##### Option 2: Confirm via AWS CLI *(Faster if you're comfortable with CLI)*

```bash
aws cognito-idp admin-confirm-sign-up \
  --user-pool-id ap-southeast-1_AbCdEfGhI \
  --username player1 \
  --region ap-southeast-1
```

Replace `ap-southeast-1_AbCdEfGhI` with your actual `CognitoUserPoolId` from the CDK output.

---

##### Option 3: Auto-Confirm in CDK Stack *(For development environments only)*

You can add a **Pre Sign-up Lambda trigger** in your CDK stack that automatically confirms every new user:

```csharp
// In ApiStack.cs — add a pre sign-up trigger to auto-confirm users
var autoConfirmFunction = new Function(this, "AutoConfirmFunction", new FunctionProps {
    Runtime = Runtime.DOTNET_8,
    Handler = "GameBackend.Handlers::AutoConfirm::Handler",
    Code = Code.FromAsset("path-to-lambda")
});

userPool.AddTrigger(UserPoolOperation.PRE_SIGN_UP, autoConfirmFunction);
```

The trigger Lambda simply sets `event.response.autoConfirmUser = true` and `event.response.autoVerifyEmail = true`.

> **Warning**: Only use auto-confirmation in `dev` or `test` environments. In production, always require email or phone verification to prevent account abuse.

---

#### Understanding the JWT Tokens

Cognito returns three tokens on successful login. Here's what each one does in this project:

![token](images/cognito-access-token.png)

| Token | Lifespan | Used For |
|---|---|---|
| **Access Token** | 1 hour | Sent in `Authorization: Bearer` header for API calls. Validated by API Gateway's Cognito Authorizer. |
| **ID Token** | 1 hour | Contains user claims (username, email, sub/userId). The `LoginHandler` extracts the `sub` claim to use as `userId`. |
| **Refresh Token** | 30 days | Used to obtain new Access/ID tokens when they expire — without requiring the user to re-enter their password. |

In the Unity Client (`RealAuthService.cs`), the Access Token is stored in `ApiClient` via `ApiClient.Instance.SetAuth(token)` after a successful login. Every subsequent API call automatically includes it.

---

#### Configuring the Unity Client with Cognito Values

Once you have the CDK outputs, open Unity Editor:

1. In the **Project** panel → `Assets/Resources/` → select `GameConfig.asset`

![config](images/unity_gameconfig_inspector.png)

2. In the **Inspector**, fill in:

   | Field in Inspector | CDK Output Key | Example Value |
   |---|---|---|
   | **Aws Cognito User Pool Id** | `CognitoUserPoolId` | `ap-southeast-1_AbCdEfGhI` |
   | **Aws Cognito Client Id** | `CognitoAppClientId` | `1a2b3c4d5e6f7g8h9i0j...` |
   | **Aws Cognito Region** | *(your deployment region)* | `ap-southeast-1` |

3. Press **Ctrl+S** to save.

---

#### Common Errors & Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `errorCode: USER_NOT_CONFIRMED` | User registered but not yet confirmed | Confirm via Console or CLI (see above) |
| `NotAuthorizedException: Incorrect username or password` | Wrong credentials | Double-check username/password. Note: Cognito usernames are case-sensitive |
| `UserNotFoundException` | Username doesn't exist in the User Pool | Make sure you registered with the correct User Pool ID |
| `401 Unauthorized` on protected routes | JWT token expired or not included | Re-login to get a fresh token. Check that `ApiClient.SetAuth(token)` was called |
| `Invalid client id` | Wrong `CognitoAppClientId` in `GameConfig.asset` | Copy-paste the exact `CognitoAppClientId` value from CDK output |

---

#### IAM — Permissions Required for Cognito

##### Why IAM Matters Here

![iam](images/iam.png)

Cognito doesn't work in isolation — Lambda functions need IAM permissions to **call Cognito APIs** (login, register, confirm user, etc.). On top of that, your AWS account needs IAM permissions so that **CDK can create** the Cognito User Pool during deployment.

There are two layers of IAM to be aware of:

1. **CDK deployer permissions** (the IAM account running `cdk deploy`) — needed to provision Cognito resources
2. **Lambda Execution Role permissions** — needed to call Cognito APIs at runtime

---

##### Layer 1: IAM Permissions for the CDK Deployer

The IAM account used for `aws configure` and `cdk deploy` must have permission to create Cognito resources:

```json
{
  "Effect": "Allow",
  "Action": [
    "cognito-idp:CreateUserPool",
    "cognito-idp:CreateUserPoolClient",
    "cognito-idp:DeleteUserPool",
    "cognito-idp:DeleteUserPoolClient",
    "cognito-idp:DescribeUserPool",
    "cognito-idp:UpdateUserPool",
    "cognito-idp:SetUserPoolMfaConfig",
    "cognito-idp:AdminConfirmSignUp",
    "cognito-idp:AdminCreateUser"
  ],
  "Resource": "*"
}
```

> **Simpler option for dev environments**: Use `cognito-idp:*` to allow all Cognito actions. In production, narrow this down following the **Least Privilege** principle — grant only the exact permissions needed.

---

##### Layer 2: Lambda Execution Role — Runtime Cognito Permissions

When a player hits "Login", the Lambda function (`LoginHandler`) needs to call Cognito's `InitiateAuth` API. This requires the **Lambda Execution Role** to have `cognito-idp:InitiateAuth` permission.

In this project, CDK automatically creates and attaches these permissions to Lambda via `LambdaStack.cs`. You don't need to do it manually. The CDK configuration looks like this:

```csharp
// In LambdaStack.cs
// CDK automatically creates the Execution Role and attaches required permissions
var loginFunction = new Function(this, "LoginFunction", new FunctionProps
{
    Runtime = Runtime.DOTNET_8,
    Handler = "GameBackend.Handlers::LoginHandler::Handler",
    Code = Code.FromAsset("path-to-deploy-package.zip"),
    Environment = new Dictionary<string, string>
    {
        ["COGNITO_USER_POOL_ID"] = userPool.UserPoolId,
        ["COGNITO_CLIENT_ID"]   = userPoolClient.UserPoolClientId,
    }
});

// Grant specific Cognito permissions to this Lambda's Execution Role
userPool.Grant(loginFunction, "cognito-idp:InitiateAuth");
userPool.Grant(loginFunction, "cognito-idp:SignUp");
userPool.Grant(loginFunction, "cognito-idp:AdminConfirmSignUp");
userPool.Grant(loginFunction, "cognito-idp:AdminGetUser");
```

The environment variables `COGNITO_USER_POOL_ID` and `COGNITO_CLIENT_ID` are read by Lambda at runtime to know which User Pool to call — no hardcoding in source code.

---

##### How to Check a Lambda's IAM Permissions on the Console

If you hit a permissions error when calling an API, follow these steps to inspect the Execution Role:

1. Open the [AWS Lambda Console](https://console.aws.amazon.com/lambda/).
2. Select the failing function (e.g., `LoginFunction`).

![iam-lambda-function](images/iam-lambda-function.png)

3. Go to the **Configuration** tab → **Permissions**.
4. Click the **Execution role** name (e.g., `LoginFunction-ServiceRole-xxxx`).
![iam-lambda-function-1](images/iam-lambda-function-1.png)
5. The IAM Console opens — review the list of policies attached to that role.
![iam-lambda-function-2](images/iam-lambda-function-2.png)
6. Check whether any policy allows `cognito-idp:InitiateAuth` on your User Pool's ARN.

If the permission is missing, click **Add permissions** → **Attach policies** to add the required policy.

---

##### IAM Permission Summary per Lambda Function

| Lambda Function | Cognito API Called | Description |
|---|---|---|
| `LoginHandler` | `cognito-idp:InitiateAuth` | Validates username/password and returns JWT tokens |
| `RegisterHandler` | `cognito-idp:SignUp` | Creates a new player account in the User Pool |
| `RegisterHandler` | `cognito-idp:AdminConfirmSignUp` | Auto-confirms the user (dev/auto-confirm flow only) |
| `LoginHandler` | `cognito-idp:AdminGetUser` | Fetches user info by username to return `userId` |

---

##### Common IAM Errors

| Error | Cause | Fix |
|---|---|---|
| `AccessDeniedException: User is not authorized to perform cognito-idp:InitiateAuth` | Lambda Execution Role is missing the permission | Add `cognito-idp:InitiateAuth` to the Lambda's Execution Role (see Console steps above) |
| `UnauthorizedException` when running `cdk deploy` | The IAM deployer account lacks `cognito-idp:CreateUserPool` | Add `cognito-idp:*` policy to the IAM user configured in `aws configure` |
| `ResourceNotFoundException` | Lambda is reading the wrong `COGNITO_USER_POOL_ID` from environment variables | Check Lambda Console → Configuration → Environment variables for the correct value |
