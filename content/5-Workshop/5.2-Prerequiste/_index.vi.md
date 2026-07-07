---
title: "Điều kiện tiên quyết"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Điều kiện tiên quyết

## Bộ công cụ local

| Công cụ | Phiên bản tham chiếu | Mục đích |
| --- | --- | --- |
| Python | 3.11+ | Backend FastAPI và test |
| Node.js | 20 | Build React/Vite và chạy Vitest |
| Docker | Bản stable hiện tại | Build và smoke test backend image |
| AWS CLI | v2 | Đọc/vận hành đúng AWS account và region |
| Terraform | 1.10.5 | Format và validate hạ tầng |
| Git và Gitleaks | Bản stable hiện tại | Quản lý source và quét secret |

Các tài nguyên regional của LiveCap dùng `ap-southeast-1`. CloudFront là dịch
vụ global; WAF scope CloudFront chỉ được Terraform quản lý qua `us-east-1` vì
đây là yêu cầu của AWS đối với global Web ACL.

## Mô hình truy cập AWS

Môi trường local dùng AWS profile. Fargate task dùng ECS task role; credential
không được chép vào `.env`, Docker image hoặc Git. Quyền runtime chỉ cấp cho
Transcribe, Translate, transcript S3, CloudWatch và action ECS idle-scaling tùy chọn.

## Cấu hình backend

Copy `backend/.env.example` thành file `.env` local đã được ignore:

| Biến | Giá trị tham chiếu | Trách nhiệm |
| --- | --- | --- |
| `AWS_REGION` | `ap-southeast-1` | Region cho Transcribe, Translate và S3 |
| `S3_BUCKET` | theo môi trường | Bucket private để export transcript |
| `SESSION_TIMEOUT` | `1800` | Thời lượng session tối đa theo giây |
| `MAX_CONCURRENT_SESSIONS` | `4` | Giới hạn session active toàn process |
| `MAX_SESSIONS_PER_IP` | `1` | Giới hạn session active trên mỗi IP |
| `BILINGUAL_DUAL_STREAM` | `true` | Chạy stream tiếng Việt và Anh song song |
| `ALLOWED_ORIGIN` | frontend origin | CORS allowlist |
| `CLOUDWATCH_LOG_GROUP` | `livecap` | Đích structured logging |

Idle scaling mặc định an toàn với `ENABLE_IDLE_SCALE_DOWN=false`.
`ECS_CLUSTER_NAME` và `ECS_SERVICE_NAME` có thể để trống khi chạy local. Biến
`MAX_SPEAKERS` cũ đã bị loại khỏi cấu hình active.

## Cấu hình frontend

| Biến | Giá trị local | Trách nhiệm |
| --- | --- | --- |
| `VITE_API_BASE_URL` | `http://127.0.0.1:8000` | REST API base URL |
| `VITE_WS_URL` | `ws://127.0.0.1:8000/ws/transcribe` | WebSocket endpoint |
| `VITE_WAKE_BACKEND_URL` | để trống | Wake path tùy chọn của target |
| `VITE_BACKEND_HEALTH_URL` | local `/api/health` | Poll trạng thái backend |
| `VITE_BACKEND_WAKE_TIMEOUT_SECONDS` | `120` | Thời gian chờ startup tối đa |
| `VITE_MAX_SESSION_SECONDS` | `1800` | Countdown session trên UI |

Microphone chỉ bắt đầu sau khi backend ready. Audio sinh ra lúc socket chưa mở
sẽ bị drop thay vì buffer không giới hạn.

## Điều kiện an toàn

- Không commit `.env`, `terraform.tfvars`, backend config, state hoặc plan thật.
- Chạy Gitleaks trước khi publish thay đổi.
- CI dùng `terraform init -backend=false` và không tự apply hạ tầng.
- Import/reconcile tài nguyên AWS hiện hữu và review plan trước mọi apply.
- Chỉ xử lý audio khi người vận hành có quyền sử dụng nội dung đó.
