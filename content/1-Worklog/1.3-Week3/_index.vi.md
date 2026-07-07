---
title: "Worklog Tuần 3"
date: 2026-05-08
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Thiết lập và làm quen với môi trường phát triển mã nguồn cloud (Cloud IDE) thông qua dịch vụ AWS Cloud9.
* Cài đặt, cấu hình và sử dụng thành thạo giao diện dòng lệnh AWS CLI để tương tác trực tiếp với các tài nguyên trên đám mây.
* Xây dựng kỹ năng tự động hóa bằng cách viết script CLI kết hợp truy vấn lọc dữ liệu nâng cao.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu lý thuyết về AWS Cloud9, ưu thế của lập trình trực tuyến và cơ chế tự động tạm dừng (Auto-hibernate) để tiết kiệm chi phí | 04/05/2026 | 04/05/2026 | <https://000049.awsstudygroup.com/> |
| 3 | - **Thực hành:** Khởi chạy một môi trường Cloud9 dựa trên EC2 instance, cấu hình tự động ngủ đông sau 30 phút rảnh rỗi và viết mã thử nghiệm | 05/05/2026 | 05/05/2026 | <https://000049.awsstudygroup.com/> |
| 4 | - Tìm hiểu cách thức cài đặt AWS CLI, cấu hình các profile bảo mật và định dạng hiển thị kết quả (JSON, Table, Text) | 06/05/2026 | 06/05/2026 | <https://000011.awsstudygroup.com/> |
| 5 | - **Thực hành:** Cài đặt AWS CLI v2 trên máy ảo Cloud9, cấu hình định danh qua lệnh `aws configure` và thực hiện các câu lệnh kiểm thử cơ bản | 07/05/2026 | 07/05/2026 | <https://000011.awsstudygroup.com/> |
| 6 | - **Thực hành:** Sử dụng AWS CLI thực hiện thao tác quản lý S3 (tạo bucket, upload file, đồng bộ thư mục) và viết kịch bản bash shell kết hợp lọc dữ liệu bằng flag `--query` | 08/05/2026 | 08/05/2026 | <https://000011.awsstudygroup.com/> |

### Kết quả đạt được tuần 3:

* Triển khai thành công môi trường lập trình trực tuyến Cloud9 tích hợp sẵn các công cụ phát triển cần thiết.
* Thiết lập chính xác cơ chế tự động ngủ đông (Auto-sleep) giúp tối ưu hóa chi phí điện toán, tránh lãng phí khi không sử dụng.
* Cài đặt thành công AWS CLI v2, làm chủ việc quản lý các access profile để phân tách quyền hạn giữa các môi trường (Dev, Staging).
* Sử dụng thành thạo các câu lệnh CLI quản trị tài nguyên lưu trữ và tính toán.
* Viết thành công một file script tự động hóa bằng Bash shell, trích xuất danh sách các server đang chạy và tự động xuất báo cáo định dạng JSON gửi lên S3 bucket.
