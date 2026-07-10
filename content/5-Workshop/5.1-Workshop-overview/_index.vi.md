---
title: "Tổng quan workshop"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Tổng quan workshop

## LiveCap giải quyết vấn đề gì?

Trong các cuộc họp đa ngôn ngữ, người tham gia thường khó theo dõi khi cuộc
trò chuyện diễn ra bằng một ngôn ngữ mà họ không thông thạo. Các giải pháp
truyền thống hoặc cần phiên dịch viên (tốn kém và có độ trễ), hoặc phụ thuộc
vào plugin của nền tảng họp vốn lưu trữ bản ghi âm trên máy chủ của bên thứ ba.

**LiveCap** là ứng dụng phụ đề song ngữ thời gian thực chạy trên trình duyệt:

- Thu âm từ microphone **ngay trong trình duyệt** – không cần cài thêm plugin hay app.
- Stream PCM 16 kHz qua **WebSocket bảo mật** trực tiếp đến backend.
- Tạo phụ đề song ngữ **Việt ↔ Anh** gần thời gian thực bằng Amazon Transcribe
  và Amazon Translate.
- Chỉ lưu **transcript TXT đã hoàn tất** vào S3 private; raw audio không bao giờ
  được ghi lại.

## Đối tượng sử dụng

| Đối tượng | Mục đích sử dụng |
|---|---|
| Người tham gia cuộc họp | Theo dõi cuộc trò chuyện song ngữ mà không cần phiên dịch |
| Team có AWS account | Triển khai một prototype SaaS AI hoàn chỉnh end-to-end |
| Cloud practitioner | Học ECS Fargate, WebSocket streaming và các AWS AI service cùng lúc |

## Nhóm use-case

LiveCap thuộc hai nhóm chồng nhau:

- **AI/ML Application** – dùng Amazon Transcribe Streaming và Amazon Translate
  làm lớp xử lý trí tuệ nhân tạo cốt lõi.
- **Web Application** – phục vụ React frontend qua CloudFront + S3 và FastAPI
  backend qua ECS Fargate đằng sau ALB.

## Các dịch vụ AWS sử dụng

| Dịch vụ | Vai trò trong LiveCap |
|---|---|
| **Amazon CloudFront** | Điểm vào HTTPS/WSS công khai; định tuyến static assets và API/WebSocket |
| **Amazon S3** | Host frontend private (OAC) và lưu transcript TXT private |
| **Application Load Balancer** | Health check và forward HTTP/WebSocket vào container Fargate |
| **Amazon ECS Fargate** | Runtime container serverless cho backend FastAPI |
| **Amazon ECR** | Registry image container bất biến (Git SHA tag) |
| **Amazon Transcribe Streaming** | Chuyển PCM 16 kHz thành partial/final text |
| **Amazon Translate** | Dịch finalized segment giữa tiếng Việt và tiếng Anh |
| **AWS WAF** | Chặn managed threats và rate abuse ở CloudFront và ALB |
| **AWS Lambda** | Wake-on-demand: scale ECS từ 0 → 1 trước phiên đầu tiên |
| **Amazon CloudWatch** | Nhận structured log và metric từ ứng dụng và dịch vụ AWS |
| **AWS IAM** | Role least-privilege cho ECS task, Lambda và ECR execution |

## Kiến trúc đang chạy đã xác minh

Sơ đồ dưới đây thể hiện chính xác các tài nguyên AWS đang live tính đến thời
điểm nộp workshop. Target Terraform (private subnet, NAT Gateway, scale-to-zero,
WAF, dashboard, budget) đã được deploy qua blue/green cutover sau đó.

![Kiến trúc LiveCap đang hoạt động đã xác minh](/images/5-Workshop/livecap-target-architecture.png)

Landing page LiveCap (phục vụ từ CloudFront qua S3 private với OAC):

![LiveCap landing page hiển thị hero section với thẻ xem trước phụ đề song ngữ](/images/5-Workshop/livecap-landing.png)

## Luồng hoạt động chính

1. **CloudFront** phục vụ frontend React/Vite từ **S3 private** qua Origin Access
   Control (OAC) – bucket không có public access.
2. Người dùng bấm **Start**; frontend gọi `/api/wake` qua CloudFront, định tuyến
   đến **Lambda function** để scale ECS từ 0 → 1 nếu cần.
3. Frontend poll `/api/health`, xin quyền microphone, rồi mở `/ws/transcribe` qua
   WSS: CloudFront → ALB → Fargate.
4. **FastAPI** kiểm tra giới hạn session toàn hệ thống và theo IP trước khi mở
   AWS stream.
5. PCM 16 kHz được gửi đến **Amazon Transcribe Streaming** (hai stream song song:
   tiếng Việt và tiếng Anh).
6. Chỉ các segment **finalized** được gửi đến **Amazon Translate** và thêm vào
   bảng phụ đề song ngữ; kết quả partial chỉ hiển thị tạm thời rồi bị loại bỏ.
7. Phụ đề đi ngược về: Fargate → ALB → CloudFront → trình duyệt.
8. **Export**: frontend POST finalized rows đến `/api/sessions/{id}/export`, backend
   ghi TXT vào S3 private và trả về presigned URL có thời hạn.
