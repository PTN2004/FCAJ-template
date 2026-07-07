---
title: "Backend dạng container trên ECS Fargate"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Backend dạng container trên ECS Fargate

Backend EC2 đơn lẻ ban đầu đã được thay bằng dịch vụ FastAPI đóng gói Docker và
chạy trên Amazon ECS Fargate. Amazon ECR lưu image immutable, ECS duy trì task
mong muốn, còn ALB health check và forward HTTP/WebSocket đến port 8000 của container.

Demo đã xác minh hiện chạy một task trong VPC hiện hữu với public IP. ALB trải
trên hai public subnet. Hệ thống có thể tự thay task lỗi nhưng không phải
active-active HA vì service được giới hạn một task khi session state còn nằm
trong memory của process.

Kiến trúc target giữ luồng ALB -> ECS, nhưng chuyển task vào hai private subnet,
tắt public IP và dùng một NAT Gateway cho outbound đến AWS API và ECR.
