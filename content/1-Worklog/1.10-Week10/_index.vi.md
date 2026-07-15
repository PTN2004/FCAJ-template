---
title: "Worklog Tuần 10"
date: 2026-06-26
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Nắm vững kiến thức và kỹ năng thiết kế hệ thống có tính sẵn sàng cao (High Availability), tự động cân bằng tải và co giãn tài nguyên.
* Triển khai bộ cân bằng tải ứng dụng Application Load Balancer (ALB) và thiết lập các nhóm đích (Target Groups) có cơ chế Health Check tự động.
* Xây dựng cụm máy chủ tự động co giãn Auto Scaling Group (ASG) kết hợp cấu hình tự động hóa qua Launch Templates.
* Tối ưu hóa tính ổn định của phiên LiveCap thông qua cơ chế heartbeat, logic tự động kết nối lại và thiết lập vòng đời lưu trữ tự động dọn dẹp file.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu lý thuyết các mô hình mở rộng: Phân biệt mở rộng theo chiều dọc (Vertical scaling) và chiều ngang (Horizontal scaling), cách thức cấu hình mở rộng tự động dựa theo metrics hệ thống | 22/06/2026 | 22/06/2026 | <https://000006.awsstudygroup.com/> |
| 3 | - Tìm hiểu cơ chế hoạt động phối hợp giữa: Launch Templates (mẫu khởi chạy), Target Groups (nhóm đích), cơ chế Health Check và bộ cân bằng tải ALB | 23/06/2026 | 23/06/2026 | <https://000006.awsstudygroup.com/> |
| 4 | - **Thực hành:** Khởi tạo Application Load Balancer (ALB), cấu hình Target Group và cài đặt các tham số Health Check (như path kiểm tra `/index.html`, số lần thử lại, thời gian timeout) | 24/06/2026 | 24/06/2026 | <https://000006.awsstudygroup.com/> |
| 5 | - **Thực hành:** Tạo Launch Template chứa cấu hình loại máy chủ, ổ đĩa, Security Group và đoạn mã script User Data giúp tự động cài đặt Apache Web Server khi boot máy | 25/06/2026 | 25/06/2026 | <https://000006.awsstudygroup.com/> |
| 6 | - **Thực hành:** Khởi tạo Auto Scaling Group liên kết với ALB và Target Group, thiết lập chính sách co giãn (Scaling Policy) dựa trên mức CPU sử dụng trung bình là 60%, cài đặt công cụ stress test trên EC2 để kiểm thử quá trình tự động tăng/giảm quy mô máy chủ | 26/06/2026 | 26/06/2026 | <https://000006.awsstudygroup.com/> |
| 6 | - **Dự án Capstone:** Lập trình cơ chế heartbeat trên WebSocket, viết hàm xử lý kết nối lại (reconnect) và cấu hình S3 Lifecycle Policy tự dọn dẹp transcript | 26/06/2026 | 26/06/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 10:

* Triển khai thành công bộ cân bằng tải ALB, giúp phân phối lưu lượng truy cập đồng đều đến các máy chủ trong Private Subnets, đồng thời tự động loại bỏ các máy chủ gặp lỗi ra khỏi cụm đích.
* Làm chủ công cụ Launch Template giúp chuẩn hóa cấu hình cài đặt ban đầu cho hàng loạt máy chủ mà không cần thao tác tay.
* Xây dựng thành công hệ thống Auto Scaling Group tự động co giãn số lượng instance dựa theo tải thực tế của hệ thống.
* Đã xác thực cơ chế tự phục hồi lỗi (Self-healing): Khi chủ động dừng 1 máy chủ, ASG đã nhận diện trạng thái lỗi và tự động kích hoạt máy chủ mới để đảm bảo dung lượng tối thiểu yêu cầu cho ứng dụng.
* Thiết lập thành công các giải pháp bảo vệ kết nối và cấu hình tự động xóa tệp tin transcript trên Amazon S3 sau 14 ngày để tối ưu chi phí và bảo mật.
