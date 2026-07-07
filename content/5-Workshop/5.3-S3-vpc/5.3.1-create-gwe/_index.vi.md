---
title: "Build và publish backend container"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

# Build và publish backend container

## Bước 1: Kiểm tra backend

FastAPI cung cấp `/api/health`, REST route export và `/ws/transcribe`. Trước khi
build image, chạy:

```powershell
cd backend
python -m pip install -r requirements-dev.txt
python -m compileall app
python -m pytest
```

Baseline đồ án đã xác minh có 204 backend test pass.

## Bước 2: Build cho Fargate

Dockerfile cài dependency Python đã pin version, copy application, expose port
8000 và có container health check. Image được build theo kiến trúc runtime của
ECS và smoke test `/api/health` trước khi publish.

```powershell
$imageTag = "$(git rev-parse --short HEAD)-amd64"
docker buildx build --platform linux/amd64 --provenance=false `
  -t "livecap-backend:$imageTag" --load .
docker run --rm -d --name livecap-smoke -p 8000:8000 `
  --env-file .env "livecap-backend:$imageTag"
Invoke-RestMethod http://127.0.0.1:8000/api/health
docker stop livecap-smoke
```

Không đưa `.env` hoặc AWS credential vào image. Local dùng AWS profile; ECS
inject setting và dùng IAM role khi chạy.

## Bước 3: Push image immutable

ECR lưu backend image. Deployment tag được tạo từ Git commit thay vì `latest`,
nhờ đó task definition luôn trỏ đến artifact có thể truy vết. Demo công khai đã
xác minh dùng tag `1ef4250-amd64`.

```text
source commit -> Docker image -> ECR tag immutable -> ECS task definition
```

Đăng nhập Docker vào ECR và push đúng immutable tag. Chỉ thay repository name
khi môi trường target dùng ECR repository khác.

```powershell
$region = "ap-southeast-1"
$accountId = aws sts get-caller-identity --query Account --output text
$registry = "$accountId.dkr.ecr.$region.amazonaws.com"
$repository = "$registry/livecap-backend"
aws ecr get-login-password --region $region |
  docker login --username AWS --password-stdin $registry
docker tag "livecap-backend:$imageTag" "$repository:$imageTag"
docker push "$repository:$imageTag"
```

## Bước 4: Tạo task definition revision

Cập nhật image URI thành SHA tag vừa push, giữ container port 8000, inject
environment và gắn execution/task role. Đăng ký revision mới thay vì sửa artifact
cũ. Sau khi review revision, cập nhật ECS service và chờ service stable.

```powershell
aws ecs update-service --region ap-southeast-1 `
  --cluster <cluster-name> --service <service-name> `
  --task-definition <task-definition-family:revision>
aws ecs wait services-stable --region ap-southeast-1 `
  --cluster <cluster-name> --services <service-name>
```

Không chạy placeholder trực tiếp trên production. Phải xác nhận account,
cluster, service, task-definition diff và rollback revision trước.

## Tách IAM role

- **Task execution role:** pull image từ ECR và ghi container log.
- **Task role:** gọi Transcribe, Translate, transcript S3, CloudWatch và action
  ECS scale-down tối thiểu nếu bật.
- **Wake Lambda role:** chỉ describe/update ECS service được chọn trong target.

Cách tách này ngăn application kế thừa quyền deployment quá rộng.
