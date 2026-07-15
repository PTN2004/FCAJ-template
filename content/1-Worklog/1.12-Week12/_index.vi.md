---
title: "Worklog Tuần 12"
date: 2026-07-10
weight: 12
chapter: false
pre: " <b> 1.12 </b> "
---

### Mục tiêu tuần 12:

* Tổng hợp toàn bộ các phân hệ dịch vụ đã học trong chuỗi Explore AWS Services thành một dự án kiến trúc web 3 lớp có tính sẵn sàng cao (High Availability).
* Thiết kế, triển khai thực tế, ghi nhận kết quả hoạt động của toàn bộ hạ tầng và hoàn tất báo cáo thực tập worklog 12 tuần.
* Triển khai thực tế ứng dụng LiveCap qua CI/CD, thực hiện kiểm thử tích hợp (độ trễ), phân tích cấu trúc chi phí và nghiệm thu dự án.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế sơ đồ kiến trúc ứng dụng web 3 lớp có tính sẵn sàng cao và khả năng tự động khôi phục lỗi | 07/06/2026 | 07/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Thực hành:** Tạo VPC, Subnets, Gateway định tuyến an toàn và cấu hình khởi tạo RDS MySQL Multi-AZ ở lớp cơ sở dữ liệu | 07/07/2026 | 07/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **Thực hành:** Triển khai lớp ứng dụng web với Auto Scaling Group gán IAM Role bảo mật và đấu nối qua ALB | 07/08/2026 | 07/08/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **Thực hành:** Cấu hình Route53 điều hướng tên miền về ALB và thiết lập hệ thống CloudWatch giám sát toàn bộ tài nguyên | 07/09/2026 | 07/09/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Thực hiện kiểm thử tích hợp (Integration Test), viết báo cáo tổng kết thực tập và nghiệm thu worklog 12 tuần | 07/10/2026 | 07/10/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Dự án Capstone:** Triển khai LiveCap lên AWS, thực hiện tích hợp kiểm thử hệ thống, phân tích thống kê chi phí và tổng hợp hồ sơ nghiệm thu | 07/10/2026 | 07/10/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 12:

* Xây dựng và triển khai thành công mô hình kiến trúc ứng dụng web 3 lớp hoàn chỉnh, đảm bảo tính dự phòng, khả năng co giãn và cô lập bảo mật chặt chẽ.
* Cấu hình thành công tính năng Multi-AZ cho cơ sở dữ liệu RDS MySQL giúp hệ thống tự động failover sang zone dự phòng khi xảy ra sự cố.
* Liên kết thành công tên miền DNS qua Route53 để phân phối tải qua ALB đến cụm máy chủ tự co giãn ASG.
* Hoàn thiện và tổng hợp đầy đủ nội dung báo cáo thực tập worklog 12 tuần theo đúng lộ trình đã đề ra.
* Đưa ứng dụng LiveCap vận hành thực tế trên AWS Cloud, vượt qua các bài kiểm thử chức năng và xuất bản báo cáo đánh giá hạ tầng kèm chi phí tối ưu.
