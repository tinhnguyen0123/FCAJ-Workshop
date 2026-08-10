---
title: "Bản đề xuất"
date: 2026-08-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Bản Đề Xuất: Trò Chơi Nhập Vai Khám Phá Hầm Ngục AI

## 1. Giới thiệu tổng quan
Dự án mang đến một làn gió mới cho dòng game phiêu lưu nhập vai (RPG) 2D bằng cách ứng dụng Trí tuệ Nhân tạo tạo sinh (Generative AI) vào việc xây dựng kịch bản. Thay vì phụ thuộc vào một máy chủ truyền thống nặng nề, hệ thống được vận hành hoàn toàn trên nền tảng AWS Serverless.

Người chơi có thể tự do tạo hình nhân vật và dấn thân vào những cuộc hành trình không có kịch bản định trước. Thay vào đó, diễn biến câu chuyện, các chướng ngại vật hay cả những trận đánh Boss theo lượt đều được AWS Bedrock tạo ra trực tiếp (real-time) dựa vào từng quyết định của người chơi. Trải nghiệm đồ hoạ 2D mượt mà được xử lý bởi Unity Client, trong khi toàn bộ logic phức tạp chạy ngầm trên hệ thống Backend .NET 8 của AWS.

## 2. Vấn đề và Giải pháp

### Những hạn chế của Game RPG hiện tại
*   **Nội dung nghèo nàn, lặp lại:** Các tựa game nhập vai hiện nay đa phần đều dùng kịch bản tĩnh (hard-coded). Dù nhà phát triển có cố gắng tạo ra nhiều nhánh truyện đến đâu, người chơi cuối cùng vẫn sẽ khám phá hết, dẫn đến sự nhàm chán và làm giảm giá trị chơi lại.
*   **Gánh nặng chi phí máy chủ:** Việc duy trì các máy chủ trạng thái (stateful) truyền thống đòi hỏi nguồn ngân sách lớn cho phần cứng rảnh rỗi, đồng thời rất khó để mở rộng hệ thống kịp thời khi lượng người chơi tăng vọt.

### Hướng đi đột phá của dự án
*   **Cốt truyện thiên biến vạn hóa với AI:** Nhờ tích hợp sức mạnh của Mô hình Ngôn ngữ Lớn (LLM) qua AWS Bedrock, trò chơi có khả năng liên tục sinh ra bối cảnh và phản hồi cốt truyện linh hoạt theo từng hành động cụ thể của game thủ.
*   **Tối ưu hóa với Serverless:** Những cơ chế cốt lõi như đăng nhập, hòm đồ, hay logic tính toán chiến đấu đều được ủy thác cho AWS Lambda, kết hợp với tốc độ truy xuất siêu tốc từ Amazon DynamoDB. Điều này không chỉ giúp hệ thống tự động mở rộng theo lưu lượng người dùng mà còn tối ưu hóa chi phí vận hành với cơ chế "dùng bao nhiêu, trả bấy nhiêu".

## 3. Tổng quan kiến trúc hệ thống

Dự án áp dụng mô hình 100% Serverless trên AWS, tách bạch rõ ràng giữa phần Game Client và Backend để tối đa hóa tính bảo mật cũng như hiệu năng.

![AWS Architecture Diagram](images/aws-architecture.png)
*(Sơ đồ kiến trúc tổng thể của dự án)*

*   **Amazon API Gateway & Cognito:** Đóng vai trò là cửa ngõ bảo mật, quản lý luồng đăng nhập, đăng ký và cấp quyền truy cập (JWT Token) cho người chơi. Mọi thao tác gọi API đều phải đi qua chốt chặn này.
*   **Lớp Xử Lý (AWS Lambda - .NET 8):** Nơi chứa các hàm độc lập xử lý mọi logic nghiệp vụ quan trọng: từ tính toán sát thương trong chiến đấu, quản lý kho đồ, cho đến việc làm cầu nối giao tiếp với AI.
*   **Lớp Dữ Liệu (Amazon DynamoDB):** Đóng vai trò là cơ sở dữ liệu NoSQL lưu trữ thông tin nhân vật, tiến trình chơi và vật phẩm với độ trễ phản hồi cực thấp.
*   **AWS Bedrock:** Trái tim sáng tạo của game, chuyên tiếp nhận các câu lệnh ngữ cảnh (prompt) và đưa ra các đoạn văn bản diễn biến theo thời gian thực.

## 4. Chi tiết triển khai công nghệ

Dự án được tổ chức theo cấu trúc **Monorepo**, giúp đồng bộ hóa các model dữ liệu (DTOs) dễ dàng giữa C# trên Unity Client và C# trên Lambda Backend.
*   **Giao diện Client (Frontend):** Phát triển trên nền tảng Unity 2D sử dụng Universal Render Pipeline (URP), kết nối và tương tác với máy chủ thông qua các API chuẩn RESTful.
*   **Hạ tầng dưới dạng mã (IaC):** Toàn bộ cơ sở hạ tầng được định nghĩa bằng mã thông qua **AWS CDK (C#)**, giúp việc triển khai lên các môi trường (như Dev hay Prod) diễn ra đồng bộ, tự động và ít rủi ro.
*   **Cơ chế Bảo mật:** Tuân thủ mô hình Server-Authoritative, nghĩa là Backend nắm quyền quyết định tối hậu. Mọi chỉ số máu, sát thương hay rơi đồ đều được tính toán trên AWS Lambda, giúp loại bỏ triệt để nguy cơ gian lận (cheat/hack) từ phía Client.

## 5. Kế hoạch và Lộ trình thực hiện

*   **Giai đoạn 1 (22/06/2026 - 05/07/2026):** Chốt phương án kiến trúc tổng thể, khởi tạo hạ tầng qua AWS CDK, đồng thời đưa vào hoạt động hệ thống xác thực Amazon Cognito và cơ sở dữ liệu DynamoDB.
*   **Giai đoạn 2 (06/07/2026 - 19/07/2026):** Tích hợp AWS Bedrock. Xây dựng logic tự động đóng gói `Prompt` từ ngữ cảnh và phát triển bộ Parser để chuyển đổi văn bản phản hồi của AI thành chuỗi JSON chuẩn nạp vào game.
*   **Giai đoạn 3 (20/07/2026 - 02/08/2026):** Lập trình các logic Backend quan trọng như: Cơ chế chiến đấu theo lượt (Turn-based Combat), sinh Boss và hệ thống quản lý kho đồ (Inventory).
*   **Giai đoạn 4 (03/08/2026 - 15/08/2026):** Lắp ghép hoàn chỉnh Unity Client với Backend API. Tiến hành kiểm thử toàn diện (End-to-End Testing) và tìm cách rút ngắn thời gian phản hồi của AI Bedrock.

## 6. Dự trù ngân sách

Lợi thế tuyệt đối của kiến trúc Serverless là việc khai thác triệt để các gói miễn phí (Free Tier) của AWS trong giai đoạn chạy thử nghiệm:
*   **AWS Cognito / Lambda / DynamoDB:** $0.00 (Hoàn toàn miễn phí trong giới hạn cho phép).
*   **AWS Bedrock:** Tính phí dựa trên số Token tiêu thụ (Dự kiến tốn khoảng $1.00 - $5.00/tháng đối với lưu lượng test của nhóm).
*   **Amazon API Gateway & CloudWatch:** Dao động từ $0.50 - $1.00/tháng.
*   **Tổng ngân sách ước tính:** **~$1.50 - $6.00 / tháng**. Mức chi phí trong mơ cho một hệ thống có khả năng tự mở rộng.

## 7. Phân tích và quản lý rủi ro

| Yếu tố Rủi ro | Mức độ | Phương án Khắc phục |
| :--- | :--- | :--- |
| **Độ trễ phản hồi của AI (Latency)** | Cao | Bổ sung hiệu ứng "đang suy nghĩ" hoặc "loading" mượt mà trên giao diện Unity để người chơi không cảm thấy bị gián đoạn. |
| **Lỗi định dạng JSON từ AI** | Trung bình | Xây dựng các lớp kiểm tra (Validator) tại Backend; tự động bắt lỗi và kích hoạt cơ chế Retry/Fallback khi AI trả về sai cấu trúc kịch bản. |
| **Vượt giới hạn chi phí Token AI** | Thấp | Cài đặt chặt chẽ tham số `max_tokens` cho từng request API và thiết lập tính năng cảnh báo chi tiêu tự động qua AWS Budgets. |

## 8. Mục tiêu đạt được

*   **Nâng tầm trải nghiệm người chơi:** Mang đến một tựa game luôn mới mẻ, giá trị chơi lại vô tận nhờ kịch bản không giới hạn từ Generative AI.
*   **Xây dựng Bộ khung chuẩn (Framework):** Hoàn thiện một cấu trúc chuẩn mực kết hợp giữa Unity và hệ sinh thái AWS .NET 8 Serverless. Bộ khung này hoàn toàn có thể tái sử dụng dễ dàng cho các dự án game trực tuyến hoặc app tương tác trong tương lai.
*   **Chứng minh tính hiệu quả chi phí:** Khẳng định khả năng thiết kế, xây dựng và vận hành một hệ thống trò chơi phức tạp với mức chi phí hạ tầng ban đầu gần như bằng không.