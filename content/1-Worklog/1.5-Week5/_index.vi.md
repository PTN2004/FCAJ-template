---
title: "Worklog Tuần 5"
date: 2026-05-22
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Làm chủ các khái niệm máy chủ ảo trên AWS thông qua dịch vụ Amazon Elastic Compute Cloud (EC2).
* Triển khai, cấu hình bảo mật tường lửa và vận hành hệ điều hành máy chủ Linux bên trong phân vùng mạng ảo tùy chỉnh.
* Học lý thuyết về ổ lưu trữ khối mạng, thực hành khởi tạo, định dạng và gắn kết ổ đĩa bổ sung Amazon EBS.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu dịch vụ EC2: Cách chọn Instance Family phù hợp hiệu năng (General, Compute, Memory), lựa chọn AMI và các mô hình mua sắm | 18/05/2026 | 18/05/2026 | <https://000004.awsstudygroup.com/> |
| 3 | - **Thực hành:** Khởi tạo và chạy một máy chủ ảo EC2 sử dụng hệ điều hành Linux (t3.micro) nằm tại Public Subnet của VPC đã triển khai ở Tuần 4 | 19/05/2026 | 19/05/2026 | <https://000004.awsstudygroup.com/> |
| 4 | - **Thực hành:** Thiết lập Security Group mở cổng SSH (port 22) và HTTP (port 80), tải Key Pair cá nhân và thực hiện kết nối SSH vào máy chủ từ local terminal | 20/05/2026 | 20/05/2026 | <https://000004.awsstudygroup.com/> |
| 5 | - Học lý thuyết về ổ lưu trữ khối Amazon EBS (phân biệt các dòng gp2, gp3, io2) và thực hiện tạo mới ổ đĩa gp3 dung lượng 10GB rồi gắn (attach) vào EC2 instance | 21/05/2026 | 21/05/2026 | <https://000004.awsstudygroup.com/> |
| 6 | - **Thực hành:** Đăng nhập máy chủ, thực hiện phân vùng (partition), định dạng hệ thống file ext4, cấu hình mount ổ đĩa EBS mới vào thư mục `/var/www/html` và cài đặt Apache Web Server | 22/05/2026 | 22/05/2026 | <https://000004.awsstudygroup.com/> |

### Kết quả đạt được tuần 5:

* Khởi tạo, bật/tắt và quản lý trạng thái máy chủ ảo EC2 thành thạo trên Console.
* Thiết lập tường lửa bảo mật tầng host (Security Group) ngăn chặn các IP lạ dò quét cổng, chỉ mở cổng cần thiết cho công việc.
* Thực hiện thành công việc đính kèm và quản lý dữ liệu lưu trữ bền vững (persistent block storage) với EBS.
* Làm chủ kỹ năng quản trị đĩa trên hệ điều hành Linux: Sử dụng các câu lệnh kiểm tra phân vùng `lsblk`, định dạng đĩa `mkfs.ext4`, gắn đĩa `mount` và cập nhật file `/etc/fstab` để tự động khôi phục liên kết đĩa khi khởi động lại.
* Deployed và vận hành thành công một máy chủ Web Apache phục vụ người dùng, lưu trữ nội dung trang web trực tiếp trên phân vùng ổ cứng EBS mới được gắn.
