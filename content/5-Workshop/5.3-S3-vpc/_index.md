---
title: "Backend Deployment – ECS Fargate"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Backend Deployment – ECS Fargate

This section walks through deploying the LiveCap FastAPI backend as a
containerized service on Amazon ECS Fargate. By the end, you will have a
running backend that responds to health checks and is reachable from the
Application Load Balancer.

## What You Will Deploy

| Component | Technology | Where |
|---|---|---|
| Container image | Docker / FastAPI | Amazon ECR |
| Container runtime | ECS Fargate | Private subnet, no public IP |
| Load balancer | Application Load Balancer | Public subnets, two AZs |
| Service registry | ECS Service `livecap-target-service-dev` | Scale-to-zero: 0 ↔ 1, Lambda wake, idle scale-down after 300 s |
| Network | Custom VPC `10.20.0.0/16` | ap-southeast-1 |

The current live cluster is `livecap-cluster-dev`:

![ECS cluster list showing livecap-cluster-dev with 1 running task](/images/5-Workshop/5.3-S3-vpc/ecs_clusters.png)
