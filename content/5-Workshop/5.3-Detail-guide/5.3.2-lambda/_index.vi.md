---
title: "Triển khai Backend AWS Lambda (.NET 8)"
date: 2026-08-12
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

### Triển khai Backend AWS Lambda (.NET 8)

#### Tổng quan

Trong tựa game **AI Dungeon RPG Adventure Game**, **AWS Lambda** hoạt động trên môi trường **.NET 8 (C#)** đóng vai trò là tầng tính toán không máy chủ (Serverless Compute Tier). Để đảm bảo kiến trúc **Server-Authoritative** (Server quyết định kết quả) nhằm chống gian lận (cheat/hack) từ phía client, toàn bộ logic nghiệp vụ—bao gồm tính toán chiến đấu theo lượt, cập nhật kho đồ, và gọi Generative AI sinh cốt truyện—đều được thực thi bên trong các hàm Lambda phân tán.

Hệ thống Backend bao gồm **13 AWS Lambda Functions** được tổ chức thành 5 nhóm dịch vụ chính:

| Nhóm Dịch Vụ | Tên Hàm Lambda | Mô Tả Chức Năng | Tương Tác Dịch Vụ AWS |
|---|---|---|---|
| **Auth** | `RegisterFunction` | Đăng ký tài khoản người chơi & mã hóa mật khẩu | Amazon DynamoDB (`Users`) |
| **Auth** | `LoginFunction` | Xác thực đăng nhập & cấp phát JWT token | Amazon Cognito / DynamoDB |
| **Auth** | `RefreshTokenFunction` | Làm mới JWT token hết hạn | Amazon Cognito |
| **Character** | `GetCharacterFunction` | Lấy thông tin hồ sơ, level và chỉ số nhân vật | Amazon DynamoDB (`Characters`) |
| **Inventory** | `GetInventoryFunction` | Lấy danh sách trang bị & vật phẩm trong kho đồ | Amazon DynamoDB (`Inventory`) |
| **Inventory** | `EquipItemFunction` | Trang bị vật phẩm & cập nhật chỉ số nhân vật | Amazon DynamoDB (`Inventory`, `Characters`) |
| **Inventory** | `UnequipItemFunction` | Tháo trang bị & tính toán lại chỉ số | Amazon DynamoDB (`Inventory`, `Characters`) |
| **Battle** | `StartBattleFunction` | Khởi tạo phiên trận đấu Boss theo lượt | Amazon DynamoDB (`Battles`, `Bosses`) |
| **Battle** | `ResolveBattleFunction` | Tính toán sát thương, chí mạng & rớt đồ | Amazon DynamoDB (`Battles`, `LootDrops`) |
| **Story AI** | `StartStoryFunction` | Khởi tạo màn chơi cốt truyện mới | Amazon DynamoDB (`StorySessions`) |
| **Story AI** | `StoryActionFunction` | Gọi AI Bedrock (`amazon.nova-pro-v1:0`) sinh cốt truyện | **AWS Bedrock**, Amazon S3, DynamoDB |

---

### Bước 1: Đóng gói các hàm Lambda .NET 8 Backend

1. Mở terminal và di chuyển vào thư mục chứa dự án Lambda Handlers trong C# monorepo:
   ```bash
   cd backend/src/GameBackend.Handlers
   ```

2. Khôi phục các thư viện NuGet và đóng gói file thực thi thành gói Zip triển khai:
   ```bash
   dotnet lambda package --configuration Release --output-package bin/Release/net8.0/deploy-package.zip
   ```

3. Kiểm tra file `deploy-package.zip` đã được tạo thành công trong đường dẫn `bin/Release/net8.0/`.

![Dotnet Lambda Package Output](images/5.3.2-1-dotnet-build.png)

---

### Bước 2: Viết Mã Nguồn C# Lambda Handler (`StoryActionFunction`)

Dưới đây là đoạn mã nguồn C# của `StoryActionFunction.cs`, thể hiện cách hàm Lambda giao tiếp với **Amazon Bedrock (`amazon.nova-pro-v1:0`)** để sinh kịch bản và các lựa chọn cho người chơi:

```csharp
using System.Text.Json;
using Amazon.BedrockRuntime;
using Amazon.BedrockRuntime.Model;
using Amazon.Lambda.Core;
using Amazon.Lambda.APIGatewayEvents;
using GameBackend.Core.Services;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace GameBackend.Handlers
{
    public class StoryActionFunction
    {
        private readonly IAmazonBedrockRuntime _bedrockClient;
        private readonly StoryService _storyService;

        public StoryActionFunction()
        {
            _bedrockClient = new AmazonBedrockRuntimeClient();
            _storyService = new StoryService();
        }

        public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            context.Logger.LogInformation("Processing StoryActionFunction request...");

            try
            {
                var body = JsonSerializer.Deserialize<StoryActionRequest>(request.Body);
                
                // 1. Đọc prompt mẫu từ S3 / Local Cache
                string prompt = $"Player Choice: {body.ChoiceText}. Generate next narrative and 3 choices.";

                // 2. Gọi mô hình Amazon Bedrock Nova Pro
                var payload = new
                {
                    inferenceConfig = new { max_new_tokens = 500, temperature = 0.7 },
                    messages = new[] { new { role = "user", content = new[] { new { text = prompt } } } }
                };

                var invokeRequest = new InvokeModelRequest
                {
                    ModelId = "amazon.nova-pro-v1:0",
                    ContentType = "application/json",
                    Accept = "application/json",
                    Body = new MemoryStream(System.Text.Encoding.UTF8.GetBytes(JsonSerializer.Serialize(payload)))
                };

                var response = await _bedrockClient.InvokeModelAsync(invokeRequest);
                using var reader = new StreamReader(response.Body);
                string responseJson = await reader.ReadToEndAsync();

                // 3. Lưu trạng thái cốt truyện mới vào DynamoDB
                await _storyService.SaveStoryNodeAsync(body.SessionId, responseJson);

                return new APIGatewayProxyResponse
                {
                    StatusCode = 200,
                    Headers = new Dictionary<string, string> { { "Content-Type", "application/json" } },
                    Body = responseJson
                };
            }
            catch (Exception ex)
            {
                context.Logger.LogError($"Error in StoryActionFunction: {ex.Message}");
                return new APIGatewayProxyResponse { StatusCode = 500, Body = ex.Message };
            }
        }
    }
}
```

---

### Bước 3: Triển khai Hạ tầng Lambda bằng AWS CDK

1. Di chuyển vào thư mục hạ tầng `infrastructure`:
   ```bash
   cd infrastructure
   ```

2. Thực thi lệnh deploy riêng cho Stack Lambda Backend (AWS CDK sẽ tự động khởi tạo 13 hàm Lambda, IAM Roles và định tuyến API Gateway):
   ```bash
   cdk deploy GameLambdaStack
   ```

---

### Bước 4: Kiểm Tra Danh Sách Lambda Functions trên AWS Console

1. Đăng nhập vào **AWS Management Console** và mở dịch vụ **AWS Lambda** (Đảm bảo Region là `ap-southeast-1 Singapore`).
2. Chọn **Functions** ở menu bên trái. Bạn sẽ thấy đầy đủ 13 hàm Lambda đã được triển khai.

![AWS Lambda Functions List](images/5.3.2-2-lambda-list.png)

3. Nhấp chọn hàm **`StoryActionFunction`** để xem chi tiết cấu hình và trigger API Gateway.

![Lambda StoryActionFunction Details](images/5.3.2-3-lambda-detail-story.png)

4. Cuộn xuống tab **Configuration** → **Permissions** để kiểm tra quyền hạn IAM Execution Role đã được gắn các policy gọi Bedrock (`bedrock:InvokeModel`).

![Lambda IAM Execution Role Permissions](images/5.3.2-4-lambda-iam-role.png)

5. Mở **Amazon CloudWatch Logs** để kiểm tra log thực thi chi tiết sau khi thực hiện gửi request cốt truyện từ Unity Client hoặc Postman.

![CloudWatch Logs for StoryActionFunction](images/5.3.2-5-lambda-cloudwatch-logs.png)

---

### Kết quả

Bạn đã hoàn tất việc triển khai và kiểm tra **13 hàm AWS Lambda (.NET 8)** Serverless. Hệ thống Backend đã sẵn sàng xử lý các sự kiện trong game, tính toán chiến thuật và sinh cốt truyện bằng AI theo thời gian thực!
