---
title: "Deploy & Xác minh ECS Fargate Service"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

# Deploy & Xác minh ECS Fargate Service

## Bước 1 – Task Definition do Terraform quản lý

Task definition của LiveCap được quản lý hoàn toàn bởi Terraform –
**không deploy bằng file JSON thủ công**. Mỗi khi bạn cập nhật image tag
trong biến Terraform và chạy `terraform apply`, Terraform sẽ tự đăng ký
revision task definition mới và trigger rolling update cho ECS service.

Các trường quan trọng trong task definition thực tế:

| Trường | Giá trị |
|---|---|
| Family | `livecap-target-backend-dev` |
| Container name | `livecap-backend` |
| Image | `<account>.dkr.ecr.ap-southeast-1.amazonaws.com/livecap-backend:<sha>-amd64` |
| Port mapping | Container 8000 → Host 8000 |
| CPU | 256 (0.25 vCPU) |
| Memory | 512 MB |
| Execution role | `livecap-execution-role` (pull image, ghi log) |
| Task role | `livecap-task-role` (gọi Transcribe, Translate, S3) |
| Network mode | `awsvpc` |

Để xem revision hiện tại đang chạy:

```powershell
aws ecs describe-task-definition `
  --task-definition livecap-target-backend-dev `
  --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "taskDefinition.{Family:family, Revision:revision, Image:containerDefinitions[0].image}"
```

## Bước 2 – Cập nhật ECS Service qua Terraform

Service đúng trong cluster là `livecap-target-service-dev`.
Terraform là cách được khuyến nghị để cập nhật service – nó xử lý đúng thứ tự
dependency và tránh drift:

```powershell
# Xem trạng thái service hiện tại
aws ecs describe-services `
  --cluster livecap-cluster-dev `
  --services livecap-target-service-dev `
  --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "services[0].{Status:status, Running:runningCount, Desired:desiredCount}"
```

Nếu cần force deploy revision mới (chỉ dùng trong môi trường dev, ngoài Terraform):

```powershell
$cluster = "livecap-cluster-dev"
$service = "livecap-target-service-dev"
$taskDef = "livecap-target-backend-dev:<revision>"  # thay bằng số revision thực tế

aws ecs update-service `
  --cluster $cluster `
  --service $service `
  --task-definition $taskDef `
  --force-new-deployment `
  --region ap-southeast-1 `
  --profile livecap-camgiacntn

# Chờ service ổn định
aws ecs wait services-stable `
  --cluster $cluster `
  --services $service `
  --region ap-southeast-1 `
  --profile livecap-camgiacntn
```

## Bước 3 – Xác minh service đang chạy

Kiểm tra trạng thái service từ console hoặc CLI:

```powershell
aws ecs describe-services `
  --cluster livecap-cluster-dev `
  --services livecap-target-service-dev `
  --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "services[0].{Status:status,Running:runningCount,Desired:desiredCount}"
```

Kết quả mong đợi:

```json
{
  "Status": "ACTIVE",
  "Running": 1,
  "Desired": 1
}
```

AWS Console hiển thị service với một task đang chạy:

![Chi tiết ECS service – 1 task đang chạy, trạng thái ACTIVE](/images/5-Workshop/5.3-S3-vpc/ecs_service_detail.png)

![Tab Tasks của ECS – task đang chạy với task ID và trạng thái](/images/5-Workshop/5.3-S3-vpc/ecs_tasks_running.png)

## Bước 4 – Xác minh ALB health check

ALB kiểm tra `/api/health` trên port 8000 mỗi 30 giây. Chỉ target nào pass
health check mới nhận được traffic.

Xem ALB và target group trên console:

![Danh sách Application Load Balancer – ALB livecap](/images/5-Workshop/5.3-S3-vpc/alb_list.png)

![Chi tiết ALB – target group và listener rule](/images/5-Workshop/5.3-S3-vpc/alb_detail.png)

Test health endpoint trực tiếp qua CloudFront:

```powershell
Invoke-RestMethod https://dpeohr327wt9l.cloudfront.net/api/health
```

Kết quả mong đợi:

```json
{"status": "healthy", "version": "1.0.0"}
```

## Bước 5 – Xác minh ECS Cluster

Trang chi tiết cluster hiển thị tất cả service, task đang chạy và capacity
provider. Cluster healthy có desired = running ở tất cả service.

![Trang chi tiết ECS cluster livecap-cluster-dev với các service](/images/5-Workshop/5.3-S3-vpc/ecs_cluster_detail.png)

## Session Safety

Trước khi backend mở bất kỳ stream Transcribe hay Translate nào, nó kiểm tra
registry session trong bộ nhớ:

- Tối đa **4 session đồng thời** toàn hệ thống (process-wide)
- Tối đa **1 session active trên mỗi client IP**

Client bị từ chối nhận thông báo `TOO_MANY_SESSIONS` qua WebSocket mà không
gây ra bất kỳ chi phí AI service nào. Mọi đường thoát session (stop, timeout,
lỗi, mất kết nối) đều dọn sạch audio queue, worker task và registry entry.
