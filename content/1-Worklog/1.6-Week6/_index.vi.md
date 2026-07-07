---
title: "Worklog Tuần 6"
date: 2026-05-29
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Hiểu rõ cơ chế hoạt động của hệ thống lưu trữ đối tượng không máy chủ (Serverless Object Storage) thông qua Amazon Simple Storage Service (S3).
* Làm chủ phương thức cấu hình bảo mật dữ liệu trên S3 thông qua Bucket Policies và các tùy chọn chặn truy cập công khai (Block Public Access).
* Thực hành thiết lập, cấu hình và khởi chạy trang web tĩnh hoạt động trực tiếp trên S3 với chi phí tối ưu.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Học lý thuyết về S3: Cấu trúc chứa đối tượng, phân cấp khóa (folders), các lớp lưu trữ (S3 Storage Classes) để tối ưu hóa chi phí dài hạn | 25/05/2026 | 25/05/2026 | <https://000057.awsstudygroup.com/> |
| 3 | - Nghiên cứu cơ chế bảo mật và phân quyền đối tượng: Block Public Access, IAM Bucket Policies, Access Control Lists (ACLs) và mã hóa dữ liệu mặc định | 26/05/2026 | 26/05/2026 | <https://000057.awsstudygroup.com/> |
| 4 | - **Thực hành:** Tạo S3 Bucket với tên duy nhất trên toàn cầu, tắt tính năng Block Public Access mặc định và thực hiện tải lên các tài nguyên trang web tĩnh thông qua Console và AWS CLI | 27/05/2026 | 27/05/2026 | <https://000057.awsstudygroup.com/> |
| 5 | - **Thực hành:** Kích hoạt tính năng Static Website Hosting cho S3 Bucket, thiết lập khai báo tài liệu mặc định `index.html` và tài liệu lỗi `error.html` | 28/05/2026 | 28/05/2026 | <https://000057.awsstudygroup.com/> |
| 6 | - **Thực hành:** Viết chính sách Bucket Policy bằng định dạng JSON để mở quyền truy cập đọc công khai (Public Read-Only) cho toàn bộ đối tượng trong bucket và truy cập kiểm tra qua địa chỉ Endpoint URL của trang web | 29/05/2026 | 29/05/2026 | <https://000057.awsstudygroup.com/> |

### Kết quả đạt được tuần 6:

* Tạo thành công và quản trị các S3 Buckets đáp ứng tiêu chuẩn lưu trữ đối tượng toàn cầu.
* Làm chủ cơ chế kiểm soát truy cập trên S3, bảo vệ dữ liệu nội bộ khỏi các lỗ hổng rò rỉ thông tin qua cấu hình chặn truy cập công khai Block Public Access.
* Khởi tạo thành công hệ thống hosting trang web tĩnh trực tiếp trên S3 mà không cần khởi dựng máy chủ ảo EC2, giúp hệ thống chịu tải tốt và tiết kiệm chi phí tối đa.
* Cấu hình và áp dụng thành thạo chính sách JSON Bucket Policy cấp quyền đọc cho người dùng công cộng truy cập trang web, đồng thời bảo vệ an toàn các thông tin cấu hình quản trị khác.
