---
title: "Workshop kỹ thuật"
date: 2026-07-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop kỹ thuật

## LiveCap: Phụ đề song ngữ thời gian thực trên AWS

Workshop này mô tả phiên bản LiveCap đã hoàn thiện cho đồ án cuối khóa.
LiveCap thu âm từ microphone trong ứng dụng React, stream PCM 16 kHz qua
WebSocket đến backend FastAPI chạy trên Amazon ECS Fargate, tạo phụ đề bằng
Amazon Transcribe, dịch phần văn bản đã hoàn tất bằng Amazon Translate và hiển
thị hai cột tiếng Việt - tiếng Anh.

Demo công khai hiện dùng CloudFront, S3 private để host frontend, Application
Load Balancer, một ECS Fargate task, ECR, S3 private lưu transcript và
CloudWatch. Chức năng export chỉ lưu transcript TXT đã hoàn tất; hệ thống không
lưu raw audio.

Chương này phân biệt rõ môi trường đang chạy thật với kiến trúc target đã được
review trong Terraform. Private Fargate, NAT, WAF, wake-on-demand,
scale-to-zero, dashboard và budget là các control của target, chưa được mô tả
như tài nguyên đã deploy.

## Các phần trong workshop

1. Sản phẩm, kiến trúc và các luồng runtime.
2. Điều kiện phát triển và quyền AWS cần thiết.
3. Backend FastAPI dạng container trên ECS Fargate.
4. Phân phối frontend và tích hợp AWS thời gian thực.
5. Bảo mật, quan sát hệ thống, kiểm thử và kiểm soát chi phí.
6. An toàn release, kết quả đã xác minh và phần việc còn lại.
