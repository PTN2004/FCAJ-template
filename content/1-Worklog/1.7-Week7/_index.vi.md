---
title: "Worklog Tuần 7"
date: 2026-06-05
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Tìm hiểu về dịch vụ cơ sở dữ liệu quan hệ được quản lý hoàn toàn (Managed Relational Database Service) thông qua Amazon RDS.
* Đảm bảo tính bảo mật và độc lập cho dữ liệu bằng cách đặt database bên trong các Private Subnets thuộc VPC.
* Triển khai DB Subnet Groups và thiết lập cấu hình tường lửa Security Groups cho phép liên kết an toàn từ lớp Web Server (EC2) sang lớp Database (RDS).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu lý thuyết về Amazon RDS: Các DB engines hỗ trợ, cơ chế tạo bản sao dự phòng (Read Replicas), sao lưu tự động (Backups) và cấu hình nhóm tham số (Parameter Groups) | 01/06/2026 | 01/06/2026 | <https://000005.awsstudygroup.com/> |
| 3 | - Phân tích kiến trúc bảo vệ dữ liệu: Tạo nhóm mạng con cơ sở dữ liệu (DB Subnet Group), phân chia định tuyến mạng riêng tư và cơ chế dự phòng Multi-AZ | 02/06/2026 | 02/06/2026 | <https://000005.awsstudygroup.com/> |
| 4 | - **Thực hành:** Tạo một DB Subnet Group liên kết với các Private Subnets thuộc VPC đã thiết kế ở Tuần 4 | 03/06/2026 | 03/06/2026 | <https://000005.awsstudygroup.com/> |
| 5 | - **Thực hành:** Khởi tạo một DB Instance chạy MySQL hoặc PostgreSQL trên Amazon RDS, thiết lập tài khoản quản trị và chu kỳ bảo trì hệ thống | 04/06/2026 | 04/06/2026 | <https://000005.awsstudygroup.com/> |
| 6 | - **Thực hành:** Cấu hình Security Group cho RDS chỉ chấp nhận kết nối port 3306 đến từ Security Group của máy chủ Web EC2; cài đặt MySQL client trên EC2 và thực hiện truy vấn kiểm tra kết nối cơ sở dữ liệu | 05/06/2026 | 05/06/2026 | <https://000005.awsstudygroup.com/> |

### Kết quả đạt được tuần 7:

* Triển khai thành công hệ quản trị cơ sở dữ liệu quan hệ được quản trị tự động trên Amazon RDS.
* Thực hiện cô lập an toàn lớp dữ liệu trong vùng Private Subnet, ngăn chặn mọi đường truyền trực tiếp từ Internet vào máy chủ chứa dữ liệu.
* Thiết lập chính xác luật tường lửa liên kết (Security Group referencing), chỉ cho phép luồng traffic xuất phát từ các máy chủ ứng dụng được chỉ định gửi truy vấn SQL đến cơ sở dữ liệu.
* Đăng nhập thành công và thực thi các câu lệnh SQL khởi tạo bảng, truy vấn dữ liệu từ xa từ máy chủ EC2 đến hệ thống RDS MySQL.
