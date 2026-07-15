---
title: "Blog 2"
date: 2024-07-07    
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# PHÂN TÍCH TÍNH NĂNG ATTACK FLOW LOGS TRÊN AWS SHIELD ADVANCED

Gần đây mình đọc được một bài trên AWS Security Blog giới thiệu tính năng Attack Flow Logs của Shield Advanced. Trước giờ khi bị DDoS, quản trị viên chỉ biết "hệ thống đang bị đánh" qua vài metric tổng quan trên CloudWatch, chứ không thấy được chi tiết từng luồng traffic — muốn dựng lại cuộc tấn công là phải chắp vá từ nhiều nguồn. Tính năng mới này lấp đúng điểm mù đó: nó ghi lại các thông tin thiết yếu của lưu lượng tấn công như IP nguồn/đích, port, giao thức và quốc gia, ngay trong lúc sự cố đang diễn ra.

## Cơ chế hoạt động

Ngay trong khi bị tấn công, Shield ghi lại metadata của từng luồng traffic: IP nguồn/đích, port, protocol, TCP flags, số gói tin/byte — và 3 trường đáng chú ý:

* **srccountry**: quốc gia nguồn tấn công
* **location**: AWS edge location nơi traffic đi vào
* **action**: Shield đã xử lý từng luồng ra sao — bằng chứng mitigation cụ thể thay vì suy đoán.

Log xuất theo chu kỳ 5 phút (cả trong lẫn sau tấn công), mỗi file tối đa 75 MB, hỗ trợ JSON / plain text / W3C / Parquet.

## Log sẽ đi đâu?

Sau khi sinh ra, log không tự chảy về máy — nó đi qua một "đường ống" gồm 3 mảnh ghép:

* **DeliverySource** — người gửi: khai báo "log này phát ra từ protection nào của Shield".
* **DeliveryDestination** — người nhận: khai báo "log đổ về đâu" (S3, CloudWatch Logs, hoặc Firehose).
* **Delivery** — đơn vận chuyển: nối người gửi với người nhận. Tạo xong cái này log mới bắt đầu chảy.

Việc chọn nơi lưu trữ tùy nhu cầu:

* **S3** → lưu dài hạn, query bằng Athena để điều tra sau trận đánh.
* **CloudWatch Logs** → quan sát nhanh ngay trong lúc bị tấn công (dùng Logs Insights).
* **Data Firehose** → stream thẳng về SIEM bên thứ ba (ví dụ: Splunk, Elastic…).

Điểm hay của việc tách 3 mảnh này là ghép rất linh hoạt: nhiều Source có thể cùng trỏ về một Destination, cho phép tổ chức nhiều account gom hết log DDoS về một bucket trung tâm (qua cấu hình cross-account)

## Hai giới hạn cần biết trước khi sử dụng

* **Về phạm vi:** hiện chỉ hỗ trợ tài nguyên bảo vệ qua Elastic IP. Các entry point phổ biến của hệ web như CloudFront hay ALB thì chưa có. Nói cách khác, hệ đứng sau CloudFront/ALB tạm thời chưa dùng được.
 **Về chi phí:** ngoài phí subscription Shield Advanced, bật flow logs còn tốn thêm phí vended logs của CloudWatch Logs và phí của tài nguyên đích (lưu trữ S3/log group, hoặc xử lý Firehose).

![Sơ đồ luồng hoạt động của Attack Flow Logs](/images/3-BlogsPosted/blog2-1.png)

*Sơ đồ trên là flow diagram mình vẽ để minh họa luồng hoạt động, chưa phải reference architecture để triển khai*

## Tóm lại

Attack Flow Logs không giúp Shield chặn tốt hơn — nó giải quyết vấn đề có thể nhìn thấy và chứng minh được cuộc tấn công.

**Nguồn:** https://aws.amazon.com/blogs/security/gain-visibility-into-ddos-attacks-with-flow-logs-in-aws-shield-advanced/
