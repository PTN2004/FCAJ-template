---
title: "Worklog Tuần 9"
date: 2026-06-19
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Tìm hiểu giải pháp tối ưu hóa chi phí điện toán đám mây và thử nghiệm nhanh ứng dụng (Prototyping) với Amazon Lightsail.
* Thực hành khởi dựng nhanh các Blueprint ứng dụng tích hợp sẵn (như WordPress), thiết lập IP tĩnh và cấu hình tường lửa.
* Nghiên cứu và thực thi quy trình di chuyển nâng cấp tài nguyên từ Lightsail lên môi trường máy chủ EC2 khi hệ thống phát triển lớn.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu dịch vụ Amazon Lightsail: So sánh điểm khác biệt với EC2, tìm hiểu các gói tài nguyên cố định (bundles) và các bản mẫu ứng dụng (Blueprints như WordPress, LAMP stack) | 15/06/2026 | 15/06/2026 | <https://000045.awsstudygroup.com/> |
| 3 | - **Thực hành:** Khởi chạy một Lightsail Instance chạy hệ quản trị nội dung WordPress, cấu hình tài khoản và kiểm tra kết nối database tích hợp | 16/06/2026 | 16/06/2026 | <https://000045.awsstudygroup.com/> |
| 4 | - **Thực hành:** Tạo và gắn kết IP tĩnh (Static IP) cho máy chủ Lightsail, thiết lập mở port bảo mật (HTTP 80, HTTPS 443) trên firewall ảo của Lightsail | 17/06/2026 | 17/06/2026 | <https://000045.awsstudygroup.com/> |
| 5 | - Nghiên cứu quy trình mở rộng hệ thống (scaling) và phương pháp chuyển giao dữ liệu từ Lightsail sang EC2 | 18/06/2026 | 18/06/2026 | <https://000045.awsstudygroup.com/> |
| 6 | - **Thực hành:** Tạo bản sao lưu (Snapshot) máy chủ Lightsail, thực hiện export snapshot sang EC2, tạo máy chủ ảo EC2 mới từ snapshot và xác thực dữ liệu đồng bộ | 19/06/2026 | 19/06/2026 | <https://000045.awsstudygroup.com/> |

### Kết quả đạt được tuần 9:

* Đã làm chủ quy trình triển khai nhanh và cấu hình trơn tru các ứng dụng web thông dụng trên nền tảng Lightsail chỉ trong vài phút.
* Thiết lập kết nối ổn định bằng cách gán IP tĩnh, hạn chế rủi ro thay đổi địa chỉ IP khi khởi động lại máy chủ và cấu hình tường lửa cổng truy cập bảo mật.
* Thực hiện thành công việc nâng cấp hệ thống (Workload migration): Di chuyển toàn bộ dữ liệu từ môi trường đóng gói Lightsail sang môi trường máy chủ EC2 có khả năng mở rộng cao mà không làm mất mát cấu hình ứng dụng.
* Phân tích và đưa ra được bảng so sánh chi phí vận hành (Cost-benefit analysis) giữa hai mô hình giá cố định (Lightsail) và giá linh hoạt theo giờ (EC2).
