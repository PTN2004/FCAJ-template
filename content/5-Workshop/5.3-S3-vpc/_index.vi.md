---
title: "Triển khai Backend – ECS Fargate"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Triển khai Backend – ECS Fargate

Phần này hướng dẫn triển khai backend FastAPI của LiveCap dưới dạng container
service trên Amazon ECS Fargate. Cuối phần này bạn sẽ có một backend đang chạy,
phản hồi health check và có thể truy cập từ Application Load Balancer.

## Những gì bạn sẽ triển khai

| Thành phần | Công nghệ | Vị trí |
|---|---|---|
| Container image | Docker / FastAPI | Amazon ECR |
| Container runtime | ECS Fargate | Private subnet, không có public IP |
| Load balancer | Application Load Balancer | Public subnet, hai AZ |
| Service registry | ECS Service `livecap-target-service-dev` | Scale-to-zero: 0 ↔1, wake bằng Lambda, idle scale-down sau 300s |
| Mạng | Custom VPC `10.20.0.0/16` | ap-southeast-1 |

Cluster đang chạy thực tế là `livecap-cluster-dev`:

![Danh sách ECS cluster – livecap-cluster-dev với 1 task đang chạy](/images/5-Workshop/5.3-S3-vpc/ecs_clusters.png)
