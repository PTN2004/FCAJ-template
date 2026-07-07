---
title: "Deploy và xác minh ECS Fargate"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

# Deploy và xác minh ECS Fargate

## Thành phần runtime

1. ECS task definition tham chiếu ECR image immutable, port 8000, environment,
   execution role và task role.
2. ECS service duy trì task mong muốn và đăng ký task vào ALB target group.
3. ALB kiểm tra `/api/health` và chỉ forward đến target healthy.
4. CloudFront route `/api/*` và `/ws/*` đến ALB origin.

## An toàn session

Trước khi mở Transcribe, WebSocket handler xác định client IP và kiểm tra active
session registry trong memory. Giới hạn tham chiếu là bốn session toàn hệ thống
và một session trên mỗi IP. Client vượt giới hạn nhận `TOO_MANY_SESSIONS` mà
không mở công việc Transcribe/Translate tốn phí.

Mọi đường kết thúc đều unregister session, cleanup audio queue và worker task.
Backend còn hỗ trợ heartbeat ping/pong và timeout 30 phút.

## Cơ chế availability

ECS thay task unhealthy, còn ALB chỉ route sau khi health check pass. Vì
desired/max capacity là một, quá trình thay task gây gián đoạn ngắn và kết thúc
WebSocket active. Muốn tăng trên một task phải chuyển registry sang DynamoDB
hoặc Redis trước.

## Trạng thái hiện tại đã xác minh

| Hạng mục | Giá trị đã xác minh |
| --- | --- |
| Region | `ap-southeast-1` |
| ALB | Public, trải trên `1a` và `1b` |
| ECS desired/running | `1/1` |
| Network của task | Public subnet trong VPC hiện hữu, có public IP |
| Task definition | `livecap-backend-dev:5` |
| Container image | `1ef4250-amd64` |

Task private và wake/idle `0 <-> 1` là thay đổi target, không phải mô tả môi
trường public hiện tại.

![Network và service placement trong target đã review](/images/3-Project/livecap-target-architecture.png)
