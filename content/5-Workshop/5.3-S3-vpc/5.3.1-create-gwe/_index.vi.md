---
title: "Build & Push Container Image lên ECR"
date: 2026-07-08
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

# Build & Push Container Image lên ECR

## Bước 1 – Chạy test backend ở local

Trước khi build bất cứ thứ gì, hãy xác minh backend biên dịch được và tất cả
test đều pass:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements-dev.txt

# Kiểm tra Python syntax
python -m compileall app

# Chạy toàn bộ 204 test
python -m pytest
```

Kết quả mong đợi: **204 test pass**, không có lỗi biên dịch.

## Bước 2 – Cấu hình .env local

Copy file mẫu và chỉnh sửa cho môi trường của bạn:

```powershell
Copy-Item .env.example .env
```

Các biến cần cập nhật:

| Biến | Giá trị | Ghi chú |
|---|---|---|
| `AWS_REGION` | `ap-southeast-1` | Region cho tất cả AWS call |
| `S3_BUCKET` | `livecap-transcripts-dev-720459752315` | Tên bucket transcript của bạn |
| `ALLOWED_ORIGIN` | `http://localhost:5173` | URL frontend local |
| `BILINGUAL_DUAL_STREAM` | `true` | Bật cả tiếng Việt và tiếng Anh |

**Không bao giờ** đặt AWS access key vào file này. Backend dùng IAM profile
được set trong môi trường của bạn.

## Bước 3 – Xác minh backend chạy được ở local

```powershell
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Mở terminal khác:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/api/health
```

Kết quả mong đợi:

```json
{"status": "healthy", "version": "1.0.0"}
```

## Bước 4 – Build Docker image cho Fargate

ECS Fargate chạy trên Linux `amd64`. Dùng Docker Buildx để build đúng platform
bất kể máy của bạn dùng kiến trúc nào:

```powershell
# Tag image theo Git commit SHA hiện tại (immutable)
$imageTag = "$(git rev-parse --short HEAD)-amd64"

docker buildx build --platform linux/amd64 --provenance=false `
  -t "livecap-backend:$imageTag" --load .
```

Smoke-test image trên máy local trước khi push:

```powershell
docker run --rm -d --name livecap-smoke -p 8000:8000 `
  --env-file .env "livecap-backend:$imageTag"

Start-Sleep 3
Invoke-RestMethod http://127.0.0.1:8000/api/health

docker stop livecap-smoke
```

## Bước 5 – Xác thực Docker với ECR và Push

ECR dùng token ngắn hạn để xác thực Docker. Lấy token mới và push image:

```powershell
$region    = "ap-southeast-1"
$accountId = aws sts get-caller-identity --query Account --output text `
               --profile livecap-camgiacntn
$registry  = "$accountId.dkr.ecr.$region.amazonaws.com"
$repository = "$registry/livecap-backend"

# Đăng nhập
aws ecr get-login-password --region $region --profile livecap-camgiacntn |
  docker login --username AWS --password-stdin $registry

# Tag và push
docker tag "livecap-backend:$imageTag" "$repository:$imageTag"
docker push "$repository:$imageTag"
```

Sau khi push, image xuất hiện trong ECR với tag bất biến được tạo từ Git
commit SHA:

![Danh sách ECR repository – hiển thị repository livecap-backend](/images/5-Workshop/5.3-S3-vpc/ecr_repositories.png)

![Danh sách image ECR – tag bất biến theo định dạng Git SHA](/images/5-Workshop/5.3-S3-vpc/ecr_images_list.png)

## Bước 6 – Phân tách IAM Role

LiveCap dùng hai IAM role riêng biệt cho ECS task:

**Task Execution Role** (`livecap-execution-role`)
- Pull image từ ECR
- Ghi container log vào CloudWatch

**Task Role** (`livecap-task-role`)
- Gọi Amazon Transcribe Streaming
- Gọi Amazon Translate
- Ghi/đọc từ S3 bucket transcript private
- Ghi metric vào CloudWatch
- Tùy chọn: gọi `ecs:UpdateService` để idle scale-down

Việc phân tách này đảm bảo ứng dụng đang chạy không bao giờ có quyền deploy
mà nó không cần.
