---
title: "Event3"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “AWS FCAJ Agent Forge - Deepdive”
### Thông tin sự kiện

- **Tên sự kiện:** AWS FCAJ Agent Forge - Deepdive
- **Ngày:** 01/08/2026
- **Vai trò:** Người tham dự
- **Quy mô:** người tham dự, diễn giả và ban tổ chức đến từ AWS, AWS Partner Network (APN) và cộng đồng.
---

### Mục tiêu sự kiện

- Giới thiệu tổng quan về hệ thống Agentic AI và các cấp độ tự chủ của AI Agent.
- Hướng dẫn thiết kế và triển khai AI Agent chuẩn Level 300 cho môi trường Production của doanh nghiệp.
- Đi sâu vào kiến trúc Amazon Bedrock Agent Core với 3 thành phần cốt lõi: Runtime, Identity và Gateway.
- Hướng dẫn thực hành hands-on trên nền tảng AWS để chuyển đổi từ lý thuyết sang ứng dụng thực tế.

### Diễn giả

- **Nghĩa** – Main Presenter (Chia sẻ kiến thức chuyên sâu & Lý thuyết hệ thống Amazon Bedrock Agent Core)
- **Hải Anh** – Co-presenter (Phụ trách và hướng dẫn các buổi thực hành Hands-on lab)

### Nội dung chính

#### Tổng quan về Agentic AI & Các chuẩn giao tiếp mới

- **Định nghĩa Agentic AI**: Hệ thống phần mềm tự chủ có khả năng hiểu yêu cầu (reasoning), lập kế hoạch (planning) và thực thi các tác vụ (executing) theo từng bước.
- **Mức độ tự chủ**: Đi từ Assistant đơn giản đến Workflow cố định (Deterministic Workflow) có kiểm soát bởi con người (Human-in-the-loop), cho đến Multi-agent system hoàn toàn tự chủ.
- **Giao thức giao tiếp mới**: Bên cạnh HTTP/REST API truyền thống, hệ thống AI Agent sử dụng giao thức **MCP** (Model Context Protocol cho Agent kết nối Tool) và **A2A** (Agent-to-Agent cho các Agent trao đổi với nhau).
- **Strand SDK**: Bộ công cụ mã nguồn mở SDK do AWS phát triển riêng để tối ưu việc khởi tạo và quản lý Agent trên AWS.

#### Amazon Bedrock Agent Core – Runtime Environment

- **Môi trường Serverless**: Vận hành theo mô hình pay-as-you-go, hỗ trợ tự động mở rộng từ real-time request đến các công việc chạy ngầm kéo dài (long-running jobs).
- **Phương thức triển khai**: Đóng gói linh hoạt qua Docker Image (ECR), tập tin nén (S3) hoặc xây dựng trực tiếp qua giao diện template.
- **Công nghệ cô lập Firecracker MicroVM**: Mỗi phiên làm việc của người dùng (user session) được tách biệt hoàn toàn ở cấp độ phần cứng bằng MicroVM, đảm bảo tính bảo mật và không rò rỉ dữ liệu.
- **Phiên bản & Endpoint**: Hỗ trợ đánh quản lý phiên bản (versioning), Alias (Dev/Prod) và chiến lược triển khai cuộn (rollout 5%-10%) giúp dễ dàng rollback khi gặp sự cố.
- **Bidirectional Streaming**: Hỗ trợ truyền tải dữ liệu hai chiều thời gian thực cho cả văn bản (Text) và giọng nói/hình ảnh (Audio/Vision).

#### Amazon Bedrock Agent Core – Identity Layer

- **Quản trị xác thực & phân quyền**: Cung cấp các lớp bảo vệ Inbound Authentication (khi người dùng gọi Agent) và Outbound Authentication (khi Agent gọi công cụ ngoài).
- **Cơ chế hoán đổi Token bảo mật**: Sử dụng JWT token từ người dùng, chuyển đổi sang **WHAT** (Workload Access Token) kết hợp giữa Agent và User identity để tránh lộ credential cá nhân.
- **Token Vault**: Kho lưu trữ an toàn các khóa bảo mật và OAuth credentials cho các dịch vụ bên thứ ba.
- **Tích hợp AWS Cognito**: Tích hợp sẵn với hệ thống người dùng và IAM Role policy trên AWS.

#### Amazon Bedrock Agent Core – Gateway Layer

- **Lớp Middleware trung gian**: Giúp kết nối hàng trăm Agent tới hàng nghìn công cụ (Tools/MCP Servers) mà không bị phụ thuộc kết nối điểm-nối-điểm (point-to-point) phức tạp.
- **Human-in-the-loop**: Cho phép quản trị viên xem xét và phê duyệt (approve/deny) các yêu cầu quan trọng nằm ngoài quy trình tự động.
- **Semantic Search & Indexing**: Tìm kiếm công cụ phù hợp dựa trên ngữ nghĩa (MCP Schema), giúp Agent lựa chọn chính xác công cụ cần dùng.
- **Interceptors & Guardrails**: Bộ lọc dữ liệu đầu vào/đầu ra giúp loại bỏ thông tin nhạy cảm (PII) trước khi trả về câu trả lời cho người dùng.
- **Kết nối mạng Enterprise**: Hỗ trợ các mô hình mạng Public, Half-Private và Fully Private (thông qua AWS PrivateLink/VPC) phù hợp cho các hạ tầng On-premise hoặc Ngân hàng.

### Bài học rút ra

#### Tư duy thiết kế hệ thống (Design Mindset)

- **Cân bằng giữa tự chủ và kiểm soát**: Trong môi trường doanh nghiệp, không nên giao 100% quyền quyết định cho AI; cần áp dụng mô hình Deterministic Workflow kết hợp Human-in-the-loop.
- **Lựa chọn mô hình AI phù hợp**: Tùy vào bài toán (xuất phản hồi nhanh hay xử lý tác vụ phức tạp/code) mà lựa chọn giữa các dòng Claude (Haiku, Sonnet, Opus) hay Amazon Nova.

#### Kiến trúc kỹ thuật & Bảo mật (Technical Architecture)

- **Bảo mật đa lớp**: Đảm bảo an toàn thông qua JWT token exchange, cách ly bằng MicroVM và triển khai qua đường truyền riêng PrivateLink.
- **Tối ưu hóa thời gian phản hồi**: Áp dụng Async & Long-running jobs kết hợp Bidirectional Streaming để giữ trải nghiệm người dùng mượt mà.
- **Quản lý công cụ tập trung**: Sử dụng Agent Gateway làm chuẩn giao tiếp trung gian qua mô hình MCP Schema.

#### Chiến lược triển khai (Modernization Strategy)

- **Đưa lên Production an toàn**: Áp dụng cơ chế Canary Rollout (5-10% traffic) và lưu giữ Log/Metrics đầy đủ trên CloudWatch để giám sát an toàn.
- **Tối ưu chi phí**: Tận dụng hạ tầng Serverless để chỉ trả tiền theo lượng sử dụng thực tế.

### Áp dụng vào công việc

- **Xây dựng AI Agent chuẩn enterprise**: Ứng dụng dịch vụ Amazon Bedrock Agent Core để đóng gói và vận hành AI Agent thay vì tự phát triển hạ tầng từ đầu.
- **Tích hợp giao thức MCP & A2A**: Chuẩn hóa cách Agent tương tác với cơ sở dữ liệu nội bộ và các API bên thứ ba.
- **Cấu hình Gateway & Identity**: Thiết lập chính sách bảo mật, Token Vault và bộ lọc Guardrails để bảo vệ dữ liệu nhạy cảm.
- **Áp dụng Strand SDK**: Sử dụng Strand SDK để viết code khởi tạo Agent nhanh chóng trên môi trường AWS.

### Trải nghiệm sự kiện

Tham gia buổi workshop **“AWS FCAJ Agent Forge - Deepdive”** mang lại nhiều giá trị thực tế, cung cấp góc nhìn toàn diện về cách đưa hệ thống Generative AI từ Proof of Concept (PoC) lên môi trường Production thực tế. Những trải nghiệm nổi bật gồm:

#### Học hỏi từ các diễn giả kinh nghiệm
- Diễn giả chia sẻ góc nhìn chuẩn Level 300 từ các chuyên gia AWS, đi thẳng vào các bài toán thực tế của doanh nghiệp.
- Giải thích chi tiết các khái niệm phức tạp như Token Exchange, MicroVM isolation và MCP protocol một cách dễ hiểu.

#### Trải nghiệm Hands-on thực tế
- Được kết hợp giữa lý thuyết chuyên sâu và các bài thực hành trực tiếp trên AWS console.
- Hiểu rõ quy trình tạo Runtime, cấu hình Gateway và kết nối Identity ngay tại buổi học.

#### Đúc kết bài học giá trị
- Xây dựng Agent cho doanh nghiệp đòi hỏi tiêu chuẩn cao về Security, Scalability và Governance.
- Việc ứng dụng Bedrock Agent Core giúp tiết kiệm thời gian triển khai và tuân thủ các Best Practices của AWS.

> Tổng thể, sự kiện không chỉ đem lại lượng kiến thức kỹ thuật đồ sộ mà còn giúp định hình rõ lộ trình xây dựng và vận hành hệ thống AI Agent an toàn, hiệu quả cho các dự án thực tế.