---
title: "Workshop kỹ thuật"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop kỹ thuật

## LiveCap: Phụ đề song ngữ thời gian thực trên AWS

LiveCap là ứng dụng phụ đề cuộc họp chạy trên trình duyệt. Ứng dụng stream
âm thanh microphone từ React frontend đến backend FastAPI trên Amazon ECS
Fargate, tạo phụ đề song ngữ Việt ↔ Anh bằng Amazon Transcribe Streaming và
Amazon Translate, và hiển thị hai cột cạnh nhau gần thời gian thực.

Triển khai live dùng CloudFront, S3 private, Application Load Balancer, ECS
Fargate, ECR, AWS WAF, Wake Lambda và CloudWatch. Export transcript chỉ ghi
file TXT finalized vào S3 private; raw audio không bao giờ được lưu.

## Demo và mã nguồn

- **Ứng dụng:** [https://dpeohr327wt9l.cloudfront.net](https://dpeohr327wt9l.cloudfront.net)
- **Caption dashboard:** [https://dpeohr327wt9l.cloudfront.net/app](https://dpeohr327wt9l.cloudfront.net/app)
- **Health endpoint:** [https://dpeohr327wt9l.cloudfront.net/api/health](https://dpeohr327wt9l.cloudfront.net/api/health)
- **Source repository:** [https://github.com/9ducanh9/livecap](https://github.com/9ducanh9/livecap)

## Các phần trong workshop

| Phần | Nội dung |
|---|---|
| **5.1** | Tổng quan project, kiến trúc, luồng runtime và các dịch vụ AWS sử dụng |
| **5.2** | AWS account, IAM, bộ công cụ local và ước tính chi phí |
| **5.3** | Build và push Docker image; deploy ECS Fargate service |
| **5.4** | Deploy React frontend, bắt đầu phụ đề trực tiếp, export transcript |
| **5.5** | Kiểm thử, log, metric, alarm, bảo mật và tối ưu chi phí |
| **5.6** | Clean-up toàn bộ tài nguyên AWS và xác nhận kết quả đạt được |
