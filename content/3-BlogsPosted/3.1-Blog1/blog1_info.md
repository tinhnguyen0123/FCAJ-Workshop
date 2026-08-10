**Góc dev: 5 bài học rút ra khi kết hợp AWS Serverless & GenAI vào dự án game 2D**

Dạo này team mình đang làm một dự án game nhập vai 2D kết hợp Generative AI (AI Dungeon RPG) để dẫn dắt cốt truyện tự động. Stack hiện tại là .NET 8, AWS Serverless và Unity. Dự án vẫn đang trong quá trình hoàn thiện (WIP), nhưng đoạn dựng kiến trúc Serverless Monorepo đã cho team mình ăn không ít "hành". 

Dưới đây là 5 vấn đề thực tế team đã và đang xử lý, anh em nào đang đụng tới AWS, Bedrock hay Unity có thể tham khảo qua: 

**1\. AWS Lambda bị Cold Start làm game khựng lại**

Lúc test API xử lý lượt chơi, thỉnh thoảng Lambda mất tới 3 \- 5 giây mới khởi động xong. Người chơi bấm chọn hành động một cái là màn hình đứng hình, tưởng bị văng game. Nguyên nhân là lần đầu gọi, AWS phải mất thời gian khởi tạo container và tải các assembly .NET 8\. 

Team mình gỡ bằng cách chuyển biên dịch Backend sang Native AOT để giảm kích thước binary, đồng thời bật thêm AWS Lambda SnapStart nên độ trễ khởi động hiện đã kéo xuống dưới 1 giây. 

**2\. Amazon Bedrock trả lời hay nhưng toàn... phá luật game**

Khi chạy thử prompt từ dịch vụ Bedrock, lắm lúc con AI "phiêu" quá tự cho nhân vật nhặt được đồ xịn hoặc tự hồi full máu dù trong túi không có bình máu nào. Nguyên nhân do LLM có tính sáng tạo cao, nếu không rào kỹ sẽ rất dễ bị chệch logic game. 

Giải pháp là team tách prompt ra 3 layer rõ ràng (file system\_prompt giữ luật cứng, story\_prompt lưu bối cảnh, summary\_prompt tóm tắt lịch sử). Đồng thời ở phía Backend, team dựng riêng một lớp kiểm duyệt và xử lý dữ liệu. Nhiệm vụ của nó là "lọc" lại câu trả lời từ AI: soi xem có bị tự chế luật hay cho đồ gian lận không, sau đó ép toàn bộ phản hồi về dạng JSON chuẩn chỉnh rồi mới lưu vào DynamoDB hay gửi về Unity để test. 

**3\. Lệch Data DTO giữa Backend và Unity**

Vì game đang phát triển nên cấu trúc nhân vật, item hay Boss đổi liên tục. Mỗi lần phía Backend sửa DTO là bên Unity lại bị lỗi parse JSON hoặc lệch kiểu dữ liệu. 

Để xử lý triệt để, team tạo hẳn một Shared Library (GameShared.dll) chứa toàn bộ Data Models & DTOs dùng chung. Kết hợp với MSBuild PostBuild Event, hễ build Backend xong là file .dll này tự động sync sang thư mục Assets/Plugins/ của Unity. Hai bên đồng bộ dữ liệu hoàn toàn mà không cần sửa tay.

**4\. Trải nghiệm đăng nhập với Amazon Cognito**

Dự án dùng Amazon Cognito để quản lý tài khoản và xác thực qua Email OTP. Tuy nhiên, nếu lần nào mở lại game cũng bắt người dùng nhập lại từ đầu thì rất phiền. 

Team đã triển khai Silent Login API sử dụng Refresh Token để tự động khôi phục phiên làm việc ngầm bên dưới client. 

**5\. Tối ưu bảng DynamoDB**

DynamoDB là NoSQL, nếu chọn Partition Key / Sort Key không chuẩn từ đầu thì sau này muốn lấy lịch sử câu chuyện (StorySession) hay kho đồ (Inventory) sẽ phải dùng lệnh Scan rất chậm và tốn chi phí. 

Hiện team đang chuẩn hóa lại sơ đồ Data Modeling: dùng Partition Key là UserId, Sort Key là SessionId\#Timestamp, kết hợp Global Secondary Index (GSI) để query tối ưu nhất. 

Dự án vẫn đang trong quá trình hoàn thiện nên chắc chắn sẽ còn nhiều bài học kỹ thuật thú vị khác trong thời gian tới. Hy vọng những chia sẻ nhỏ này sẽ mang lại chút kinh nghiệm bổ ích cho anh em nào đang tìm hiểu về mô hình Serverless hoặc ứng dụng Generative AI vào làm game.