---
title: "Worklog Tuần 4"
date: 2026-05-15
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Hiểu rõ kiến trúc thiết lập mạng và mô hình định tuyến trong AWS.
* Thiết kế phân chia dải địa chỉ IP và tự triển khai hệ thống mạng ảo tùy chỉnh Amazon Virtual Private Cloud (VPC).
* Xây dựng phân vùng mạng bảo mật gồm Public Subnet (vùng công khai) và Private Subnet (vùng riêng tư), kết hợp cấu hình Internet Gateway (IGW) cùng NAT Gateway.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Nghiên cứu lý thuyết thiết kế mạng VPC: Quy tắc chia dải IP CIDR block, cách tính toán subnetting và thiết kế kiến trúc phân vùng an toàn | 11/05/2026 | 11/05/2026 | <https://000003.awsstudygroup.com/> |
| 3 | - Tìm hiểu chức năng của các thực thể mạng: Bảng định tuyến (Route Tables), cổng Internet Gateway (IGW), NAT Gateway, Security Groups và Network ACLs | 12/05/2026 | 12/05/2026 | <https://000003.awsstudygroup.com/> |
| 4 | - **Thực hành:** Tạo VPC tùy chỉnh với dải mạng 10.0.0.0/16 và chia nhỏ thành 2 Public Subnet cùng 2 Private Subnet trải rộng trên 2 Vùng khả dụng (AZs) | 13/05/2026 | 13/05/2026 | <https://000003.awsstudygroup.com/> |
| 5 | - **Thực hành:** Thiết lập cấu hình các bảng định tuyến Route Tables, gán Internet Gateway cho vùng Public và triển khai NAT Gateway nằm tại Public Subnet để phục vụ vùng Private | 14/05/2026 | 14/05/2026 | <https://000003.awsstudygroup.com/> |
| 6 | - **Thực hành:** Tạo máy ảo thử nghiệm tại cả hai vùng mạng, thực hiện SSH bridge qua Bastion Host và kiểm tra khả năng kết nối mạng một chiều từ Private Subnet ra Internet | 15/05/2026 | 15/05/2026 | <https://000003.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

* Đã tự tay thiết kế và cấu hình thành công hạ tầng mạng ảo AWS VPC hoàn thiện theo đúng mô hình kiến trúc phân lớp bảo mật chuẩn.
* Thiết lập và phân tách chính xác các luồng định tuyến (Route Tables) để đảm bảo cô lập hoàn toàn môi trường cơ sở dữ liệu/ứng dụng phía sau.
* Cấu hình hoạt động trơn tru cho NAT Gateway giúp các máy chủ trong Private Subnet thực hiện kết nối outbound ra ngoài Internet để tải tài nguyên cập nhật phần mềm, đồng thời chặn hoàn toàn các nỗ lực truy cập trái phép chiều ngược lại.
* Nắm vững các bước khắc phục sự cố mất kết nối mạng ở tầng VPC (Route Table configurations, Subnet associations, Gateway attachments).
