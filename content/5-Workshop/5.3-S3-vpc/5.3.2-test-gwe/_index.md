---
title: "Deploy and Verify ECS Fargate"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

# Deploy and Verify ECS Fargate

## Runtime Components

1. An ECS task definition references the immutable ECR image, port 8000,
   environment settings, execution role, and task role.
2. An ECS service maintains the desired task and registers it in the ALB target
   group.
3. The ALB checks `/api/health` and forwards only to healthy targets.
4. CloudFront routes `/api/*` and `/ws/*` to the ALB origin.

## Session Safety

Before starting Transcribe, the WebSocket handler resolves the client IP and
checks an in-memory active-session registry. The reference limits are four
concurrent sessions globally and one per IP. Rejected clients receive
`TOO_MANY_SESSIONS` without opening costly Transcribe/Translate work.

Every exit path unregisters the session and cleans up audio queues and worker
tasks. The backend also supports ping/pong heartbeat and a 30-minute timeout.

## Availability Behavior

ECS replaces an unhealthy task and the ALB routes only after health checks pass.
Because desired/max capacity is one, replacement causes a short interruption
and terminates the active WebSocket. Increasing beyond one task requires moving
the registry to DynamoDB or Redis first.

## Verified Current State

| Item | Verified value |
| --- | --- |
| Region | `ap-southeast-1` |
| ALB | Public, spanning `1a` and `1b` |
| ECS desired/running | `1/1` |
| Task networking | Existing VPC public subnet, public IP enabled |
| Task definition | `livecap-backend-dev:5` |
| Container image | `1ef4250-amd64` |

Private task networking and `0 <-> 1` wake/idle behavior are target changes,
not claims about the current public environment.

![Network and service placement planned for the reviewed target](/images/3-Project/livecap-target-architecture.png)
