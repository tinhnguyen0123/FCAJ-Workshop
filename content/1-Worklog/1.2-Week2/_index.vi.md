---
title: "Nhật ký công việc Tuần 2"
date: 2026-07-03
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Tích hợp hệ thống xác thực (Cognito) và thiết kế Data Transfer Objects (DTOs).

### Các công việc triển khai trong tuần này (29/06/2026 - 04/07/2026):

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành |
| --- | --- | --- | --- |
| Thứ 2 | - Khởi tạo Amazon Cognito (User Pool, App Client) cho hệ thống xác thực người dùng. | 29/06/2026 | 29/06/2026 |
| Thứ 3 | - Lập trình API Đăng nhập/Đăng ký và xử lý cấp phát JWT Tokens (Access/Refresh Token). | 30/06/2026 | 30/06/2026 |
| Thứ 4 | - Thiết kế các DTOs (Character, Battle, Inventory) trong thư viện GameShared. | 01/07/2026 | 01/07/2026 |
| Thứ 5 | - Cấu hình serialization JSON cho các DTOs để đồng bộ dữ liệu giữa Backend và Unity. | 02/07/2026 | 02/07/2026 |
| Thứ 6 | - Thiết kế giao diện UI cơ bản trên Unity và tích hợp luồng xác thực Cognito vào game. | 03/07/2026 | 03/07/2026 |

### Kết quả đạt được:

* Xây dựng thành công hệ thống Đăng nhập/Đăng ký an toàn với Amazon Cognito.
* Thống nhất được cấu trúc dữ liệu giao tiếp (DTOs) giữa Client và Server.
* Game Unity đã có thể đăng nhập và lưu trữ phiên hoạt động.
