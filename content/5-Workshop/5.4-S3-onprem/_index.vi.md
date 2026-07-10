---
title: "Phân phối Frontend & Tích hợp Thời gian thực"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Phân phối Frontend & Tích hợp Thời gian thực

Phần này hướng dẫn deploy React frontend lên S3 và CloudFront, sau đó kết nối
với backend để có phụ đề song ngữ thời gian thực.

## Những gì bạn sẽ thiết lập

| Thành phần | Công nghệ | Vị trí |
|---|---|---|
| Static site | React 18 + Vite + TypeScript | S3 bucket private |
| CDN delivery | Amazon CloudFront với OAC | Global edge |
| WebSocket | Trình duyệt ↔ FastAPI | Qua CloudFront → ALB → Fargate |
| Xử lý âm thanh | Web Audio API worklet | Trong trình duyệt |
| AI services | Transcribe Streaming + Translate | Backend Fargate gọi |
| Export transcript | S3 presigned URL | Bucket transcript private |

S3 bucket đang sử dụng thực tế:

![Danh sách S3 bucket – frontend và transcript bucket](/images/5-Workshop/5.4-S3-onprem/s3_buckets_list.png)
