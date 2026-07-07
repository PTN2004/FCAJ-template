---
title: "Worklog Tuần 8"
date: 2026-06-12
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

* Hiểu rõ cơ chế quản trị và phân giải tên miền đám mây thông qua dịch vụ Amazon Route53.
* Định cấu hình các bản ghi DNS hướng traffic người dùng đến các tài nguyên web tĩnh (S3) hoặc máy chủ (EC2).
* Thiết kế và triển khai mô hình giải pháp phân giải tên miền hỗn hợp (Hybrid DNS), cho phép truy vấn tên miền thông suốt giữa môi trường mạng nội bộ doanh nghiệp (Local) và phân vùng ảo AWS VPC.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu lý thuyết về quản trị DNS trên Route53: Phân biệt Public Hosted Zone và Private Hosted Zone, cú pháp bản ghi (A, CNAME, ALIAS, TXT) và các chính sách định tuyến (Failover, Geolocation, Latency) | 08/06/2026 | 08/06/2026 | <https://000010.awsstudygroup.com/> |
| 3 | - Nghiên cứu rào cản kết nối mạng lai (Hybrid Cloud): Khó khăn khi đồng bộ phân giải DNS giữa hệ thống máy chủ On-Premises cục bộ và máy chủ ảo trên AWS VPC | 09/06/2026 | 09/06/2026 | <https://000010.awsstudygroup.com/> |
| 4 | - **Thực hành:** Khởi tạo một Private Hosted Zone cho tên miền nội bộ doanh nghiệp, tạo bản ghi A trỏ đến địa chỉ IP nội bộ của EC2 và kiểm tra phân giải tên từ máy ảo | 10/06/2026 | 10/06/2026 | <https://000010.awsstudygroup.com/> |
| 5 | - Học cách thiết lập các điểm cuối phân giải: Cấu trúc Route53 Inbound Endpoint (nhận truy vấn từ Local) và Outbound Endpoint (gửi truy vấn về Local), thiết lập Resolver Rules | 11/06/2026 | 11/06/2026 | <https://000010.awsstudygroup.com/> |
| 6 | - **Thực hành:** Tạo các Inbound và Outbound Endpoints liên kết các subnet chỉ định, khai báo forwarding rules định hướng dải domain và kiểm thử phân giải tên miền hai chiều từ máy ảo Local simulator sang AWS | 12/06/2026 | 12/06/2026 | <https://000010.awsstudygroup.com/> |

### Kết quả đạt được tuần 8:

* Nắm rõ cách quản trị tên miền trực quan và điều khiển luồng truy cập người dùng linh hoạt với Route53.
* Triển khai thành công Private Hosted Zone giúp bảo mật thông tin tên miền máy chủ nội bộ.
* Xây dựng thành công hệ thống kết nối trung chuyển DNS lai (Hybrid DNS Resolution) sử dụng Route53 Resolver Endpoints, xử lý triệt để lỗi phân giải chéo giữa môi trường Local và Cloud.
* Đạt được khả năng phân giải tên miền 2 chiều mượt mà mà không làm gián đoạn luồng truy xuất DNS internet công cộng.
