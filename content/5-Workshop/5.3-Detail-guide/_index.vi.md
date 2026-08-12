---
title : "Hướng dẫn triển khai chi tiết"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
alwaysopen : true
---

### Hướng dẫn Triển khai Hạ tầng & Backend Chi tiết

#### Tổng quan

Mục **5.3 Hướng dẫn triển khai chi tiết** cung cấp hướng dẫn toàn diện từng bước để biên dịch các thư viện dùng chung, đóng gói các hàm Lambda C# và triển khai toàn bộ hạ tầng Serverless cho tựa game **AI Dungeon RPG Adventure Game** trên đám mây AWS bằng AWS CDK.

Hướng dẫn triển khai được tổ chức thành **5 mục con chuyên biệt**:

| Mục Con | Tên Bài Viết & Nội Dung Trọng Tâm | Dịch Vụ AWS & Tài Nguyên Chính |
|---|---|---|
| **[5.3.1](5.3.1-cognito-setup/)** | **[Thiết lập AWS Cognito](5.3.1-cognito-setup/)** | User Pool, App Client, JWT Tokens, Luồng xác thực Auth |
| **[5.3.2](5.3.2-lambda/)** | **[Triển khai Backend AWS Lambda (.NET 8)](5.3.2-lambda/)** | 13 Hàm C# Lambda, `dotnet lambda package`, `GameLambdaStack` |
| **[5.3.3](5.3.3-dynamodb/)** | **[Khởi tạo & Quản lý Cơ sở dữ liệu Amazon DynamoDB](5.3.3-dynamodb/)** | 10 Bảng NoSQL, C# Entity Models, Lưu dữ liệu `SaveAsync`, `GameDatabaseStack` |
| **[5.3.4](5.3.4-Bedrock-guide/)** | **[Cấu hình AWS Bedrock & Backend](5.3.4-Bedrock-guide/)** | `amazon.nova-pro-v1:0`, Quyền Model Access, Prompt Builder, DI Registration |
| **[5.3.5](5.3.5-Deployment/)** | **[Triển khai Game WebGL lên S3 & CloudFront](5.3.5-Deployment/)** | Build WebGL từ Unity, Hosting S3 Bucket, Phân phối CloudFront CDN |

---

### Hướng dẫn từng bước triển khai Hạ tầng & Backend

Thực hiện theo các bước chi tiết dưới đây để biên dịch thư viện dùng chung monorepo, đóng gói Lambda function .NET 8 và triển khai hạ tầng Serverless hoàn chỉnh bằng AWS CDK (C#).

---

#### Bước 1: Clone Repository & Build Thư viện Shared

1. Truy cập vào thư mục làm việc của dự án:
   ```bash
   cd AI-Dungeon-RPG-Adventure-Game
   ```

2. Biên dịch thư viện C# dùng chung (`GameShared`):
   ```bash
   dotnet build shared/GameShared.csproj -c Release
   ```
   *Lưu ý: Quá trình build sẽ tự động đồng bộ file `GameShared.dll` và `GameShared.pdb` vào `Assets/Plugins/` thông qua PostBuild event được cấu hình sẵn trong file project.*

---

#### Bước 2: Build & Đóng gói .NET 8 Lambda Backend

Backend được tách thành hai project:
- `GameBackend.Core` — Logic nghiệp vụ, services, repositories
- `GameBackend.Handlers` — Lambda entry points của AWS (phụ thuộc vào Core)

1. Di chuyển tới thư mục project Handlers:
   ```bash
   cd backend/src/GameBackend.Handlers
   ```

2. Khôi phục dependency và đóng gói Lambda deployment package:
   ```bash
   dotnet lambda package --configuration Release --output-package bin/Release/net8.0/deploy-package.zip
   ```
   *Lệnh này tự động build `GameBackend.Core` như một dependency trước khi đóng gói.*

---

#### Bước 3: Bootstrap AWS CDK & Synthesize Stack

1. Chuyển sang thư mục infrastructure:
   ```bash
   cd infrastructure
   ```

2. Bootstrap môi trường AWS (chỉ cần thực hiện 1 lần cho mỗi Region/Account):
   ```bash
   cdk bootstrap aws://<TÀI_KHOẢN_AWS_CỦA_BẠN>/ap-southeast-1
   ```

3. Tổng hợp file CloudFormation để kiểm tra định nghĩa stack:
   ```bash
   cdk synth
   ```

---

#### Bước 4: Triển khai Hạ tầng (Deploy Stack)

1. Chạy lệnh triển khai toàn bộ tài nguyên:
   ```bash
   cdk deploy --all
   ```

2. Sau khi hoàn tất triển khai, AWS CDK sẽ in ra các thông tin quan trọng trên Terminal:
   - `CognitoUserPoolId` (ví dụ: `ap-southeast-1_xxxxx`)
   - `CognitoAppClientId` (ví dụ: `1h2j3k4l5m6n7o8p...`)
   - `ApiGatewayUrl` (ví dụ: `https://xxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod/`)

3. Lưu lại các giá trị này — bạn sẽ cần để cấu hình Unity Client và công cụ kiểm thử API.

---

#### Bước 5: Cấu hình Unity Client

Sau khi triển khai xong, kết nối Unity Client tới backend AWS thực:

![GameConfigSO Inspector](5.3.1-cognito-setup/images/unity_gameconfig_inspector.png)

1. Mở **Unity Editor** và load project (thư mục `Assets/` trong repository này).

2. Trong panel **Project**, điều hướng tới `Assets/Resources/` và chọn **`GameConfig.asset`**.

3. Trong panel **Inspector**, điền các giá trị từ output lệnh `cdk deploy`:

   | Field | Giá trị |
   |---|---|
   | **Api Base Url** | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/` |
   | **Api Timeout Seconds** | `30` |
   | **Aws Cognito User Pool Id** | `ap-southeast-1_xxxxx` |
   | **Aws Cognito Client Id** | `<your-cognito-app-client-id>` |
   | **Aws Cognito Region** | `ap-southeast-1` |
   | **Use Mock Mode** | ☐ *Bỏ chọn (OFF)* |
   | **Enable Api Logging** | ☑ *Chọn (ON) — khuyến nghị khi kiểm thử* |

4. Nhấn **Ctrl+S** để lưu asset. Unity Client đã được kết nối với backend AWS thực của bạn.

5. Nhấn **Play** trong Unity Editor để khởi chạy game và kiểm thử toàn bộ luồng.

> **Mẹo**: Script `ApiClient.cs` tự động load `GameConfig.asset` khi khởi động và đính kèm JWT token vào tất cả API request. Nếu asset bị thiếu hoặc `apiBaseUrl` trống, nó sẽ fallback sang URL mặc định được hardcode trong script.