---
title : "Hướng dẫn triển khai chi tiết"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Hướng dẫn từng bước triển khai Hạ tầng & Backend

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
   *Lưu ý: Quá trình build sẽ tự động đồng bộ file `GameShared.dll` vào `Assets/Plugins/` để Unity Client sử dụng.*

---

#### Bước 2: Build & Đóng gói .NET 8 Lambda Backend

1. Di chuyển tới thư mục dự án Backend:
   ```bash
   cd backend/src/GameBackend.Api
   ```

2. Khôi phục dependency và đóng gói Lambda deployment package:
   ```bash
   dotnet lambda package --configuration Release --output-package bin/Release/net8.0/deploy-package.zip
   ```

---

#### Bước 3: Bootstrap AWS CDK & Synthesize Stack

1. Khởi tạo hạ tầng CDK (chuyển sang thư mục infrastructure):
   ```bash
   cd infrastructure
   ```

2. Bootstrap môi trường AWS (chỉ cần thực hiện 1 lần cho mỗi Region/Account):
   ```bash
   cdk bootstrap aws://<TÀI_KHOẢN_AWS_CỦA_BẠN>/<REGION_CỦA_BẠN>
   ```

3. Tổng hợp file CloudFormation:
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
   - `CognitoUserPoolId` (ví dụ: `us-east-1_xxxxx`)
   - `CognitoAppClientId` (ví dụ: `1h2j3k4l5m6n7o8p...`)
   - `ApiGatewayUrl` (ví dụ: `https://xxxxxx.execute-api.us-east-1.amazonaws.com/prod/`)

3. Lưu lại các giá trị này để cấu hình cho Unity Client (`Assets/Resources/GameConfig.json`) hoặc công cụ kiểm thử API (Postman/Insomnia).