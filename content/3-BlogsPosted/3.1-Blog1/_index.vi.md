---
title: "Góc dev: 5 bài học khi làm game 2D với AWS Serverless & GenAI"
date: 2026-08-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

> **Nguồn bài viết:** [Bài đăng trên nhóm AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2236558767109083)

Dạo này team mình đang tập trung phát triển một dự án game nhập vai 2D kết hợp Generative AI (AI Dungeon RPG) để tự động dẫn dắt cốt truyện. Stack công nghệ hiện tại tụi mình đang dùng là **.NET 8, AWS Serverless và Unity**. Dự án vẫn đang trong giai đoạn hoàn thiện (WIP), nhưng thú thật là ngay từ khâu dựng kiến trúc Serverless Monorepo đã cho team "ăn hành" không ít. 😅

Dưới đây là 5 bài học thực tế, xương máu mà team đã và đang xử lý. Anh em nào đang có ý định đụng tới hệ sinh thái AWS, Amazon Bedrock hay làm game Unity có thể tham khảo qua nhé!

### 1. Nỗi ám ảnh "Cold Start" của AWS Lambda làm game bị khựng

Lúc mới test API xử lý lượt chơi, thỉnh thoảng hàm Lambda mất tới 3 - 5 giây mới khởi động xong. Hậu quả là người chơi vừa bấm chọn hành động xong thì màn hình đứng hình luôn, cứ tưởng là bị văng game. Nguyên nhân sâu xa là ở lần gọi đầu tiên (Cold Start), AWS phải mất thời gian để khởi tạo môi trường container và tải các thư viện (assembly) của .NET 8.

**👉 Cách giải quyết của team:** 
Tụi mình quyết định chuyển sang biên dịch Backend bằng **Native AOT** để ép dung lượng binary nhỏ nhất có thể. Đồng thời bật thêm tính năng **AWS Lambda SnapStart**. Nhờ combo này mà độ trễ khởi động hiện tại đã được kéo xuống dưới 1 giây, người chơi hầu như không còn cảm nhận được độ trễ nữa.

![AWS Lambda Metrics](images/lambda_metrics.png)
*(Biểu đồ theo dõi tần suất gọi và độ trễ của AWS Lambda trên CloudWatch)*

---

### 2. Amazon Bedrock trả lời hay nhưng toàn... phá luật game

Khi chạy thử prompt từ dịch vụ AI (Amazon Bedrock), nhiều lúc con AI "phiêu" quá nên tự động cho nhân vật nhặt được đồ xịn, hoặc tự hồi full máu dù trong túi rõ ràng không có bình máu nào! Bản chất của các mô hình ngôn ngữ lớn (LLM) là tính sáng tạo rất cao, nên nếu mình không rào luật kỹ thì AI rất dễ bị "chệch" khỏi logic game ban đầu.

**👉 Cách giải quyết của team:**
Mình đã tách câu lệnh (prompt) ra làm 3 lớp (layer) rõ ràng: 
- `system_prompt` để giữ các luật cứng ngắc của game.
- `story_prompt` để lưu bối cảnh hiện tại.
- `summary_prompt` để tóm tắt lịch sử hành động.

Ngoài ra, ở phía Backend, team dựng thêm một lớp **kiểm duyệt và xử lý dữ liệu**. Nó đóng vai trò như một màng lọc: soi xem AI có tự chế luật hay bơm đồ gian lận không. Sau khi kiểm tra an toàn, hệ thống sẽ ép toàn bộ phản hồi về định dạng JSON chuẩn chỉnh rồi mới lưu xuống cơ sở dữ liệu hoặc gửi về cho Unity hiển thị.

![Unity Gameplay Screenshot](images/unity_gameplay.png)
*(Giao diện cốt truyện AI được render tự động trong game Unity)*

---

### 3. Vấn đề "lệch pha" Data DTO giữa Backend và Unity

Vì game đang trong giai đoạn phát triển liên tục nên các cấu trúc nhân vật, item hay chỉ số Boss bị thay đổi xoành xoạch. Khổ nỗi, cứ mỗi lần phía Backend (C#) sửa đổi DTO là bên Unity lại lập tức báo lỗi parse JSON hoặc báo sai kiểu dữ liệu. Sửa tay từng bên thì vô cùng mất thời gian.

**👉 Cách giải quyết của team:**
Tụi mình đã tạo hẳn một **Shared Library** (file `GameShared.dll`) chuyên chứa toàn bộ Data Models & DTOs dùng chung cho cả hai bên. Tiếp đó, cấu hình thêm *MSBuild PostBuild Event*: hễ cứ build Backend xong là file `.dll` này sẽ tự động được copy thẳng sang thư mục `Assets/Plugins/` của Unity. Nhờ vậy, hai bên luôn đồng bộ dữ liệu 100% mà không cần phải can thiệp thủ công.

---

### 4. Trải nghiệm đăng nhập với Amazon Cognito

Game của mình dùng **Amazon Cognito** để quản lý tài khoản người chơi và xác thực qua mã OTP gửi tới Email. Tuy nhiên, nếu lần nào mở app lên cũng bắt người dùng đăng nhập lại từ đầu thì trải nghiệm rất "tệ".

**👉 Cách giải quyết của team:**
Để khắc phục, team đã triển khai cơ chế **Silent Login API** ngầm. Bằng cách lưu lại và sử dụng Refresh Token, client (game) có thể tự động khôi phục phiên làm việc trước đó một cách mượt mà, người dùng vào game là chơi được ngay.

---

### 5. Bài toán tối ưu bảng dữ liệu DynamoDB

Làm quen với NoSQL như **DynamoDB** là một thử thách. Nếu ngay từ đầu bạn chọn `Partition Key` hay `Sort Key` không chuẩn, thì sau này lúc muốn truy xuất lại lịch sử cốt truyện (StorySession) hay danh sách vật phẩm (Inventory), hệ thống sẽ phải dùng đến lệnh `Scan`. Lệnh này vừa chạy cực chậm vừa "đốt" rất nhiều chi phí tài nguyên AWS.

**👉 Cách giải quyết của team:**
Hiện tại, tụi mình đang phải ngồi chuẩn hóa lại sơ đồ Data Modeling. Team quyết định dùng `UserId` làm Partition Key, `SessionId#Timestamp` làm Sort Key, và kết hợp thiết lập thêm các **Global Secondary Index (GSI)** để đảm bảo mọi câu query đều được tối ưu hóa tối đa.

---
> *Dự án vẫn đang trong quá trình hoàn thiện nên chắc chắn sẽ còn nhiều bài học kỹ thuật thú vị khác trong thời gian tới. Hy vọng những chia sẻ nhỏ này sẽ mang lại chút kinh nghiệm bổ ích cho anh em nào đang tìm hiểu về mô hình Serverless hoặc ứng dụng Generative AI vào làm game.*