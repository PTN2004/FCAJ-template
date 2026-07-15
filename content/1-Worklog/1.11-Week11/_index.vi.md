---
title: "Worklog Tuần 11"
date: 2026-07-03
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

* Làm chủ các tính năng giám sát tài nguyên (Observability), tập hợp log và thiết lập cảnh báo tự động thông qua Amazon CloudWatch.
* Cấu hình cài đặt trình thu thập dữ liệu (Agent) trên máy chủ để đồng bộ hóa logs hệ điều hành và ứng dụng về quản trị tập trung.
* Thiết lập hệ thống tự động gửi cảnh báo khẩn cấp qua Email tích hợp Amazon CloudWatch Alarms và Amazon Simple Notification Service (SNS).
* Khai báo toàn bộ hạ tầng dự án LiveCap dưới dạng mã nguồn Terraform, bao gồm CloudFront, S3, ECS và ALB.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu hệ thống quan sát Amazon CloudWatch: Tìm hiểu các thành phần chỉ số (Metrics), bảng theo dõi (Dashboards), cảnh báo (Alarms) và nhật ký ghi chép (Log Groups) | 29/06/2026 | 29/06/2026 | <https://000008.awsstudygroup.com/> |
| 3 | - **Thực hành:** Cài đặt CloudWatch Unified Agent trên máy chủ ảo EC2, tạo file cấu hình `config.json` để thu thập dữ liệu bộ nhớ RAM và dung lượng trống ổ đĩa | 30/06/2026 | 30/06/2026 | <https://000008.awsstudygroup.com/> |
| 4 | - **Thực hành:** Thiết kế giao diện CloudWatch Dashboard tùy chỉnh, tạo các widget biểu đồ đường và dạng số hiển thị tình trạng CPU, Memory, Disk và Network của cụm máy chủ | 01/07/2026 | 01/07/2026 | <https://000008.awsstudygroup.com/> |
| 5 | - **Thực hành:** Tạo một Amazon SNS Topic, đăng ký nhận tin nhắn qua email cá nhân, thiết lập một CloudWatch Alarm kích hoạt khi CPU trung bình vượt quá 80% trong 5 phút và liên kết gửi tin qua SNS | 02/07/2026 | 02/07/2026 | <https://000008.awsstudygroup.com/> |
| 6 | - **Thực hành:** Cấu hình CloudWatch Agent đẩy toàn bộ file log truy cập (Access log) và log lỗi (Error log) của Apache Web Server lên CloudWatch Log Groups, thực hiện các câu lệnh truy vấn tìm kiếm log lỗi thông qua Logs Insights | 03/07/2026 | 03/07/2026 | <https://000008.awsstudygroup.com/> |
| 6 | - **Dự án Capstone:** Xây dựng mã nguồn hạ tầng Terraform cho các thành phần ALB, ECS Fargate, CloudFront OAC, S3 buckets và phân quyền IAM | 03/07/2026 | 03/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 11:

* Triển khai thành công CloudWatch Unified Agent trên hệ điều hành Linux để giám sát chi tiết các chỉ số RAM/Disk vốn không được thu thập mặc định bởi AWS.
* Thiết kế bảng điều khiển CloudWatch Dashboard trực quan hóa toàn bộ số liệu đo lường hiệu năng của máy chủ theo thời gian thực, nâng cao năng lực giám sát.
* Cấu hình hoạt động ổn định hệ thống cảnh báo tự động, nhận được email thông báo thời gian thực khi chạy stress test đẩy CPU của EC2 vượt ngưỡng 80%.
* Quản lý tập trung toàn bộ nhật ký hoạt động của ứng dụng web, có khả năng viết các câu lệnh truy vấn lọc log lỗi nhanh chóng và hiệu quả bằng Logs Insights để debug hệ thống mà không cần SSH trực tiếp vào máy chủ.
* Tự động hóa hoàn toàn quy trình triển khai ứng dụng LiveCap qua Terraform, đảm bảo quy chuẩn bảo mật phân quyền tối giản và cô lập mạng.
