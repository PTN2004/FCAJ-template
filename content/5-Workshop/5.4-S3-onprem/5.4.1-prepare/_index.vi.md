---
title: "Build & Deploy React Frontend lên S3 + CloudFront"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# Build & Deploy React Frontend lên S3 + CloudFront

## Bước 1 – Test và build frontend

```powershell
cd frontend
npm ci          # clean install từ package-lock.json
npm test        # chạy 11 Vitest unit test
npm run build   # kiểm tra TypeScript + bundle Vite production
```

Thư mục `dist/` lúc này chứa toàn bộ static asset. Baseline đã xác minh có:
- **11 frontend test pass**
- Không có lỗ hổng bảo mật `npm audit` nào ở môi trường production

Preview ở local trước khi upload:

```powershell
npm run preview -- --host 127.0.0.1
# Mở http://127.0.0.1:4173 và kiểm tra cả / (landing) và /app (dashboard)
```

## Bước 2 – Cấu hình biến môi trường frontend

Frontend lấy URL backend từ file `.env` tại thời điểm build. Với local
development, copy file mẫu:

```powershell
Copy-Item .env.example .env
```

Với production deployment, set các biến môi trường này trước khi build:

| Biến | Giá trị production | Mục đích |
|---|---|---|
| `VITE_API_BASE_URL` | `https://dpeohr327wt9l.cloudfront.net` | REST API base |
| `VITE_WS_URL` | `wss://dpeohr327wt9l.cloudfront.net/ws/transcribe` | WebSocket endpoint |
| `VITE_WAKE_BACKEND_URL` | `/api/wake` | Lambda wake path |
| `VITE_BACKEND_HEALTH_URL` | `/api/health` | Poll trạng thái backend |
| `VITE_BACKEND_WAKE_TIMEOUT_SECONDS` | `120` | Thời gian chờ khởi động tối đa |
| `VITE_MAX_SESSION_SECONDS` | `1800` | Giới hạn session 30 phút |

## Bước 3 – Upload lên S3 bucket frontend private

S3 frontend bucket được cấu hình:
- **Block all public access** được bật
- Server-side encryption (AES-256)
- Versioning được bật
- Truy cập chỉ qua CloudFront Origin Access Control

Đồng bộ output build lên S3:

```powershell
aws s3 sync dist "s3://livecap-frontend-dev-720459752315" `
  --delete `
  --region ap-southeast-1 `
  --profile livecap-camgiacntn
```

> **Cảnh báo:** `--delete` xóa bất kỳ object S3 nào không có trong `dist/`.
> Kiểm tra kỹ tên bucket đích trước khi chạy.

## Bước 4 – Xóa cache CloudFront

Sau khi upload asset mới, tạo CloudFront invalidation để edge location phục vụ
file mới ngay lập tức:

```powershell
aws cloudfront create-invalidation `
  --distribution-id E39ADG0ES17RP1 `
  --paths "/*" `
  --profile livecap-camgiacntn
```

Chờ invalidation hoàn tất (thường 30–60 giây):

```powershell
# Kiểm tra trạng thái distribution
aws cloudfront list-distributions --profile livecap-camgiacntn `
  --query "DistributionList.Items[*].{Id:Id,Domain:DomainName,Status:Status}"
```

CloudFront distribution đang phục vụ LiveCap:

![Danh sách CloudFront distribution – distribution livecap trạng thái Deployed](/images/5-Workshop/5.4-S3-onprem/cloudfront_distributions.png)

## Bước 5 – Cấu hình routing CloudFront

CloudFront dùng behavior để định tuyến các path khác nhau đến các origin khác nhau:

| Pattern path | Origin | Ghi chú |
|---|---|---|
| `/api/*` | ALB DNS name | REST endpoint (health, export) |
| `/ws/*` | ALB DNS name | Forward WebSocket upgrade |
| `/api/wake` | Lambda function URL | Wake-on-demand (target stack) |
| `/*` (mặc định) | S3 bucket private qua OAC | Static React asset |

Một CloudFront Function sẽ rewrite các React route không có extension (ví dụ
`/app`) thành `/index.html` để refresh trang không bị trả về 403 từ S3.

## Bước 6 – Xác minh landing page

Mở CloudFront URL trên trình duyệt:

```
https://dpeohr327wt9l.cloudfront.net
```

Landing page sẽ load với hero section ("Every voice, in the room."), thẻ xem trước
phụ đề song ngữ động, các tính năng nổi bật và phần bảo mật:

![LiveCap landing page – hero section, thẻ phụ đề song ngữ, tính năng và phần bảo mật](/images/5-Workshop/livecap-landing.png)

Nút **"Open workspace"** ở góc trên bên phải và nút **"Start a live session"**
đều điều hướng trực tiếp đến `/app`.
