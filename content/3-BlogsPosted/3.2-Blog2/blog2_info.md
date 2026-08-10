**Tiêu đề**: Học AWS qua một dự án game 2D tích hợp AI

Khi bắt tay vào làm một game nhập vai 2D tương tác với AI bằng kiến trúc Serverless (dùng Unity với backend .NET 8, AWS Lambda, Bedrock), cứ thấy AI trả lời mượt trong game là sướng rồi. Nhưng khi ứng dụng bắt đầu phình to và các lượt gọi AI diễn ra liên tục, nhóm mới nhận ra một bài học đắt giá: Nếu không tính toán chi phí và bảo mật ngay từ đầu, hóa đơn AWS cuối tháng thực sự sẽ là một cú sốc lớn. 

Anh em nào đang định làm dự án có sử dụng AI trên AWS có thể tham khảo bài chia sẻ này để tránh mất tiền oan nhé:

**1\. Tiền Token LLM tăng theo cấp số nhân nếu cứ nhét toàn bộ lịch sử chat** 

Mỗi lần người chơi tương tác, app phải gọi qua Bedrock (dùng Claude hoặc Llama). Mấy model này tính tiền theo cả token đầu vào lẫn đầu ra. Nếu cứ mỗi turn lại gửi toàn bộ lịch sử trò chuyện từ đầu đến cuối thì prompt càng ngày càng dài

Để giải quyết, nhóm mình dùng file summary\_prompt nén bớt các lượt chơi cũ thành một đoạn tóm tắt ngắn gọn trước khi gửi cho AI. Đồng thời, những dữ liệu cố định như thông tin địa điểm hay thuộc tính nhân vật sẽ được cache lại vào DynamoDB chứ không hỏi AI lại từ đầu.

**2\. Siết chặt bảo mật IAM theo nguyên tắc Least Privilege**

Lúc mới dựng app, vì muốn code chạy nhanh nên mình hay tiện tay gán quyền AdministratorAccess hoặc FullAccess cho Lambda. Cách này giúp dev nhanh lúc đầu, nhưng lại là lỗ hổng bảo mật cực kỳ nguy hiểm nếu đưa lên môi trường thật. Nhờ dùng AWS CDK, nhóm rèn được thói quen phân quyền tối thiểu (Least Privilege). Function Lambda nào chỉ cần đọc dữ liệu thì gán grantReadData(), function nào gọi Bedrock thì chỉ cấp đúng quyền bedrock:InvokeModel. Việc này nhằm hạn chế một vài lỗ hổng bảo mật mà hệ thống có thể gặp phải.

**3\. Cài đặt AWS Budgets và CloudWatch Alarms trước khi gõ dòng code đầu tiên** 

Nhiều bạn dev mới làm quen với Cloud rất dễ dính quả đắng mất tiền oan vì vô tình để lặp vòng lặp vô tận gọi API hoặc quên xóa tài nguyên thử nghiệm. Kinh nghiệm xương máu là hãy thiết lập AWS Budgets ngay từ đầu để nhận mail cảnh báo nếu mà chi phí chạm ngưỡng 5 đô hay 10 đô là gửi mail báo ngay. Ngoài ra còn cài CloudWatch Alarm theo dõi số request và tỷ lệ error của Lambda để phát hiện sớm nếu app dính loop hay bị spam request rác.

Mấy bài học này nghe thì lý thuyết nhưng đúng là phải tự tay làm, tự trải nghiệm cảm giác lo nơm nớp vì hóa đơn thì mới nhớ lâu được. Mình cũng chỉ mới ở giai đoạn bắt đầu làm quen với AWS thôi, nên chắc chắn còn nhiều chỗ cần tối ưu thêm. Hy vọng chút trải nghiệm thực tế này hữu ích với các bạn.