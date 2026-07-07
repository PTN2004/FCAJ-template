---
title: "Build và phân phối React frontend"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# Build và phân phối React frontend

## Bước 1: Test và build

```powershell
cd frontend
npm ci
npm test
npm run build
```

Build chạy TypeScript checking và tạo output Vite `dist`. Baseline đã xác minh
có 11 frontend test pass và không có production npm vulnerability đã biết tại
thời điểm release.

Kiểm tra site build trước khi upload:

```powershell
npm run preview -- --host 127.0.0.1
# Mở http://127.0.0.1:4173 và kiểm tra / cùng /app.
```

## Bước 2: Lưu static asset ở S3 private

Frontend bucket chặn public access, bật encryption và versioning. Browser truy
cập qua CloudFront Origin Access Control thay vì S3 website endpoint hoặc
bucket policy public.

Sau khi xác nhận AWS account và bucket, đồng bộ output build:

```powershell
aws s3 sync dist "s3://<frontend-bucket>" --delete `
  --region ap-southeast-1
```

`--delete` xóa object không còn trong `dist`; phải review destination bucket
trước khi chạy.

## Bước 3: Cấu hình route CloudFront

| Path | Origin |
| --- | --- |
| `/` và static asset | S3 frontend private |
| `/api/*` | Application Load Balancer |
| `/ws/*` | ALB với WebSocket upgrade |
| `/api/wake` | Lambda origin có IAM trong target đã review |

CloudFront terminate HTTPS/WSS phía viewer. Kết nối CloudFront-to-ALB hiện dùng
HTTP; đây là security gap còn lại, cần ACM/custom origin domain trong bước sau.

## Bước 4: Tách runtime configuration

Local dùng `frontend/.env`. Giá trị production được inject lúc build thay vì
hard-code trong component. Nếu không cấu hình wake URL, frontend bỏ qua wake
flow tùy chọn và kết nối theo luồng bình thường.

Sau upload, invalidate CloudFront để file mới hiển thị mà không phải chờ cache
lifetime thông thường:

```powershell
aws cloudfront create-invalidation `
  --distribution-id <distribution-id> --paths "/*"
```

Landing page và dashboard `/app` đã được xác minh trên desktop và viewport
mobile rộng 390 px.

![Kết quả bước triển khai React landing page qua CloudFront](/images/3-Project/livecap-landing.png)
