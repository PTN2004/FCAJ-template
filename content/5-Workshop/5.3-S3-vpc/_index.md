---
title: "Containerized Backend on ECS Fargate"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Containerized Backend on ECS Fargate

The original single-EC2 backend was replaced by a Dockerized FastAPI service
on Amazon ECS Fargate. Amazon ECR stores immutable backend images, ECS maintains
the desired task, and the ALB performs health checks and forwards HTTP/WebSocket
traffic to container port 8000.

The verified demo currently runs one task in the existing VPC with a public IP.
The ALB spans two public subnets. This is self-healing at the task level, but it
is not active-active high availability because the service is intentionally
limited to one task while session state remains in memory.

The reviewed target keeps the same ALB -> ECS runtime model while moving tasks
to two private subnets, disabling public task IPs, and using one NAT Gateway for
outbound AWS API and ECR access.
