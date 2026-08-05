# BẢNG GHI NHẬN KẾT QUẢ THỰC TẬP HÀNG TUẦN

**TRƯỜNG ĐẠI HỌC SÀI GÒN**  
**KHOA CÔNG NGHỆ THÔNG TIN**  

**CỘNG HÒA XÃ HỘI CHỦ NGHĨA VIỆT NAM**  
**Độc lập – Tự do – Hạnh phúc**  

*Tp. Hồ Chí Minh, ngày ... tháng .... năm 20...*

---

## MỘT SỐ THÔNG TIN LIÊN HỆ

* **Họ và tên:** Nguyễn Hữu Tính
* **Ngày sinh:** 15/02/2004
* **Mã số sinh viên:** 3122411213
* **Lớp:** DCT122C4
* **Ngành học:** Công nghệ thông tin
* **Email:** huutinhnguyen1502@gmail.com
* **Điện thoại:** 0706538943
* **Chuyên gia doanh nghiệp:** Nguyễn Gia Hưng
  * Email: hunggia@amazon.com
  * Điện thoại: 0817870645
* **Giảng viên hướng dẫn:** Cao Thái Phương Thanh
  * Email: thanh.cao@sgu.edu.vn
  * Điện thoại: 0909022966

---

## BẢNG TIẾN ĐỘ VÀ NỘI DUNG THỰC TẬP

| Tuần | Nội dung thực tập <br>*(do chuyên gia của doanh nghiệp giao)* | Kết quả thực tập <br>*(do chuyên gia của doanh nghiệp đánh giá)* |
| :--- | :--- | :--- |
| **1** <br>Từ 22/06/2026 <br>đến 27/06/2026 | **Welcome day:**<br>• Giới thiệu về công ty thực tập (chương trình AWS FCAJ Workforce Bootcamp), giới thiệu về người hướng dẫn.<br>• Công ty giới thiệu về nội quy, quy định khi làm việc và quy trình thực tập.<br>• Tìm hiểu và học các kiến thức mới về nền tảng điện toán đám mây AWS và chọn đề tài làm đồ án thực tập. | |
| **2** <br>Từ 29/06/2026 <br>đến 03/07/2026 | **Khám phá các dịch vụ cơ bản về AWS:**<br>• Tạo tài khoản AWS và quản lý chi phí với AWS Budgets.<br>• Bảo mật và phân quyền: quản lý truy cập với IAM, IAM Roles cho EC2.<br>• Mạng (Networking): Cơ bản về Amazon VPC, AWS Route 53 (Hybrid DNS), CloudFront.<br>• Máy chủ và Container cơ bản: Amazon EC2, EC2 Auto Scaling, AWS Cloud9 và nhiều thứ liên quan khác.<br>• Lưu trữ và Cơ sở dữ liệu: S3 Static Website Hosting, Amazon RDS, Amazon DynamoDB.<br>• Giám sát và Công cụ: Amazon CloudWatch, AWS CLI. | |
| **3** <br>Từ 06/07/2026 <br>đến 11/07/2026 | **Học cách dùng cơ sở dữ liệu và quen thuộc với các công cụ quản lý:**<br>**Cơ sở dữ liệu:**<br>• Static Website Hosting với Amazon S3: Tìm hiểu cách cấu hình lưu trữ đối tượng, phân quyền public và triển khai lưu trữ website tĩnh trên Amazon S3.<br>• Cơ sở dữ liệu quan hệ với Amazon RDS: Khởi tạo, cấu hình và kết nối ứng dụng với các hệ quản trị cơ sở dữ liệu quan hệ (MySQL/PostgreSQL) bằng Amazon RDS.<br>• Cơ sở dữ liệu NoSQL với Amazon DynamoDB: Khám phá kiến trúc NoSQL, cách tạo bảng, quản lý key (Partition Key/Sort Key) và thực hiện các thao tác CRUD cơ bản.<br>**Công cụ quản lý:**<br>• Giám sát hệ thống với Amazon CloudWatch: Cấu hình thu thập số liệu (Metrics), thiết lập cảnh báo (Alarms) và theo dõi log hệ thống của các tài nguyên AWS.<br>• Vận hành bằng dòng lệnh với AWS CLI: Cài đặt, cấu hình bảo mật ứng dụng cấu hình (AWS Configure) và thực thi các câu lệnh quản trị tài nguyên AWS trực tiếp từ Terminal/Command Prompt. | |
| **4** <br>Từ 13/07/2026 <br>đến 18/07/2026 | • **Thiết lập hạ tầng mã nguồn (IaC) & Mô hình Shared Library:** Sử dụng AWS CDK trên nền tảng C# (.NET 8) để tự động hóa định nghĩa và cấu hình toàn bộ hạ tầng đám mây (Amazon API Gateway, AWS Lambda, Amazon DynamoDB, Amazon Cognito). Tạo thư viện dùng chung `GameShared.dll` (chuẩn .NET Standard 2.1) chứa Data Models và DTOs; thiết lập luồng MSBuild PostBuild Event tự động đồng bộ file `.dll` sang Unity Client nhằm chống sai lệch kiểu dữ liệu.<br>• **Phát triển Module Xác thực & Quản lý phiên (Authentication System):** Tích hợp dịch vụ Amazon Cognito xử lý đăng ký/đăng nhập, cấp phát bộ ba Token (Access, ID, Refresh) và xác thực tài khoản qua Email OTP nhằm tối ưu chi phí vận hành. Lập trình API Silent Login cho phép tự động khôi phục phiên hoạt động bằng Refresh Token, giúp người chơi không cần nhập lại mật khẩu khi mở lại ứng dụng. | |
| **5** <br>Từ 20/07/2026 <br>đến 25/07/2026 | • **Xây dựng Cấu trúc Dữ liệu Vật phẩm & Thuật toán Rớt đồ (Item & Loot System):** Thiết kế Data Model chi tiết cho các nhóm vật phẩm (Weapon, Armor, Accessory, Consumable) chia theo 4 cấp độ hiếm (Common, Rare, Epic, Legendary). Lập trình thuật toán xác suất rớt đồ có trọng số (Weighted Random Algorithm) và tính toán kinh nghiệm (Gold/XP) tự động điều chỉnh tỷ lệ theo cấp độ Boss.<br>• **Phát triển API Backend Quản lý Kho đồ (Inventory Logic):** Xây dựng các hàm AWS Lambda bằng C# xử lý logic kiểm tra giới hạn túi đồ (Capacity Check tối đa 100 ô/nhân vật), logic Trang bị/Gỡ trang bị (Equip/Unequip) tuân thủ quy tắc 1 slot 1 item, tự động cộng/trừ các chỉ số nhân vật (Stats) và cập nhật thời gian thực vào cơ sở dữ liệu Amazon DynamoDB. | |
| **6** <br>Từ 27/07/2026 <br>đến 01/08/2026 | • **Tích hợp Generative AI vào Cốt truyện Game (Amazon Bedrock):** Kết nối Backend C# với dịch vụ Amazon Bedrock (Claude AI) để tự động hóa việc khởi tạo, dẫn dắt và biến đổi cốt truyện game RPG linh hoạt theo từng quyết định tương tác của người chơi.<br>• **Tối ưu hóa Hiệu năng & Giải quyết sự cố Cold Start (AWS Lambda):** Phân tích và khắc phục hiện tượng giật lag (freeze UI) trên Unity do độ trễ khởi động lạnh của .NET Lambda. Áp dụng kỹ thuật Native AOT cho các API nhẹ (Get Profile, Inventory Check) để đưa độ trễ về 20-50ms; đồng thời cấu hình AWS Lambda SnapStart cho các API xử lý phức tạp (Battle Logic, gọi Bedrock AI) nhằm tối ưu tốc độ phản hồi mà vẫn tiết kiệm tối đa chi phí hạ tầng.<br>• Tham gia event Agent Force - Deepdive. | |
| **7** <br>Từ 03/08/2026 <br>đến 08/08/2026 | | |
| **8** <br>Từ 10/08/2026 <br>đến 15/08/2026 | | |

---

**Chuyên gia doanh nghiệp hướng dẫn thực tập**  
*(Ký tên và ghi họ tên)*  

---

### **Ghi chú:**
- Chuyên gia doanh nghiệp ghi nhận kết quả thực tập của sinh viên theo tuần và gửi qua email cho giảng viên hướng dẫn khi kết thúc các tuần 3, 6 của đợt thực tập. **BẢNG GHI NHẬN KẾT QUẢ THỰC TẬP TỐT NGHIỆP HÀNG TUẦN** này là một trong những hồ sơ kèm theo quyển báo cáo thực tập tốt nghiệp[cite: 1].
- Cột **Kết quả thực tập**, chuyên gia doanh nghiệp có thể ghi *Hoàn thành tốt*, *Hoàn thành*, *Không đạt* hoặc có thể ghi nhận chi tiết hơn[cite: 1].