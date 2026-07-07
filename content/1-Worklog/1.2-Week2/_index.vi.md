---
title: "Worklog Tuần 2"
date: 2026-05-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Nắm vững kiến thức nền tảng và cơ chế xác thực, phân quyền của hệ thống AWS Identity and Access Management (IAM).
* Thiết lập chính sách bảo mật truy cập chi tiết (fine-grained control) sử dụng IAM Users, Groups, Policies.
* Thực hành cấp quyền hạn truy cập tài nguyên cho máy chủ ảo EC2 bằng cách ứng dụng IAM Roles và Instance Profiles.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Học lý thuyết về AWS IAM, quy trình kiểm tra quyền truy cập (Authentication và Authorization) và nguyên tắc phân quyền tối thiểu | 27/04/2026 | 27/04/2026 | <https://000002.awsstudygroup.com/> |
| 3 | - Phân tích cú pháp cấu trúc chính sách IAM Policy (JSON bao gồm Effect, Action, Resource, Principal và các điều kiện - Conditions) | 28/04/2026 | 28/04/2026 | <https://000002.awsstudygroup.com/> |
| 4 | - **Thực hành:** Tạo các tài khoản IAM User, tổ chức nhóm IAM Group và gán các chính sách AWS Managed và Custom Policies | 29/04/2026 | 29/04/2026 | <https://000002.awsstudygroup.com/> |
| 5 | - Nghiên cứu cơ chế hoạt động của IAM Roles dành cho các dịch vụ AWS và cơ chế Instance Profile của máy chủ ảo EC2 | 30/04/2026 | 30/04/2026 | <https://000048.awsstudygroup.com/> |
| 6 | - **Thực hành:** Tạo một IAM Role có quyền truy cập S3 Read-Only, đính kèm vào máy chủ EC2 và kiểm tra truy xuất file qua AWS CLI | 01/05/2026 | 01/05/2026 | <https://000048.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:

* Hiểu sâu sắc và phân biệt rõ ràng giữa IAM User (dành cho người dùng dài hạn) và IAM Role (dành cho thực thể tạm thời hoặc dịch vụ).
* Tự viết thành công các IAM Policies tùy chỉnh (Custom Policies) dưới dạng mã JSON để giới hạn quyền chính xác cho các nhóm đối tượng.
* Loại bỏ hoàn toàn việc lưu trữ các khóa bảo mật cứng (Access Key / Secret Key) trên máy chủ bằng cách gán IAM Role cho EC2 instance.
* Đã kiểm chứng thành công cơ chế hoạt động của IAM Role trên EC2 bằng cách thực hiện truy vấn danh sách file trên S3 thành công, đồng thời việc tải file lên (Write) bị chặn chính xác do thiếu quyền hạn.
