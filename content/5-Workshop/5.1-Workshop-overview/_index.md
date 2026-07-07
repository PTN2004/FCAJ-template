---
title: "Workshop Overview"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Workshop Overview

## Problem and Goals

Language barriers and fast-paced speech make bilingual meetings difficult to
follow. LiveCap provides near-real-time captions without requiring a meeting
platform integration or storing microphone recordings.

The implemented MVP can:

- capture browser microphone audio as 16 kHz, 16-bit, mono PCM;
- stream audio through a secure WebSocket;
- transcribe Vietnamese and English with parallel Transcribe streams;
- translate only finalized segments and append only finalized caption rows;
- preserve finalized rows across bounded reconnects;
- enforce a 30-minute session limit and process-local abuse limits; and
- export finalized bilingual transcripts as TXT through a presigned S3 URL.

## Verified As-Deployed Architecture

![alt](/images/workshop/architech.png)

The live backend runs in `ap-southeast-1`. The ALB spans public subnets in
`ap-southeast-1a` and `ap-southeast-1b`. The ECS service maintains one Fargate
task with a public IP in the existing VPC. ECS can replace a failed task, but
there is no active-active backend; an in-flight WebSocket session is lost when
the task is replaced.

## Services and Responsibilities

| Service | Responsibility in LiveCap |
| --- | --- |
| CloudFront | Public HTTPS/WSS entrypoint and path routing |
| Amazon S3 | Private frontend origin and private TXT transcript storage |
| ALB | Health checks and forwarding API/WebSocket traffic to port 8000 |
| ECS Fargate | Runs the containerized FastAPI backend |
| Amazon ECR | Stores backend images under immutable SHA-derived tags |
| Amazon Transcribe | Converts streaming PCM audio to partial/final text |
| Amazon Translate | Translates finalized text between English and Vietnamese |
| CloudWatch | Receives application logs and AWS service metrics |
| GitHub Actions | Runs validation-only CI without deploying |

## Main Runtime Flow

1. CloudFront serves the React/Vite frontend from private S3 through OAC.
2. The user starts capture and grants microphone permission.
3. The frontend opens `/ws/transcribe` through CloudFront and the ALB.
4. FastAPI checks global and per-IP session limits before starting AWS streams.
5. PCM chunks are sent only while the socket is open.
6. Transcribe returns partial and finalized text; only finalized segments are
   translated and appended to the transcript.
7. Bilingual captions return over Fargate -> ALB -> CloudFront -> browser.
8. Export writes a TXT object to private S3 and returns a temporary URL.

## Current Versus Target

The repository also contains a reviewed Terraform target with a dedicated
two-AZ VPC, private tasks, one NAT Gateway, WAF in COUNT mode, a wake Lambda,
ECS `0 <-> 1` scaling, a CloudWatch dashboard, and an AWS Budget. Those changes
still require state reconciliation, plan review, and blue/green cutover.

![Reviewed target architecture used for the implementation plan](/images/workshop/target.png)
