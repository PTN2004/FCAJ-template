---
title: "Technical Workshop"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Technical Workshop

## LiveCap: Real-Time Bilingual Captions on AWS

LiveCap is a browser-based meeting caption application that streams microphone
audio from a React frontend to a FastAPI backend on Amazon ECS Fargate, produces
bilingual Vietnamese ↔ English captions using Amazon Transcribe Streaming and
Amazon Translate, and displays them side by side in near real time.

The live deployment uses CloudFront, private S3, an Application Load Balancer,
ECS Fargate, ECR, AWS WAF, a Wake Lambda, and CloudWatch. Transcript export
writes only finalized TXT files to a private S3 bucket; raw audio is never stored.

## Live Demo and Source

- **Application:** [https://dpeohr327wt9l.cloudfront.net](https://dpeohr327wt9l.cloudfront.net)
- **Caption dashboard:** [https://dpeohr327wt9l.cloudfront.net/app](https://dpeohr327wt9l.cloudfront.net/app)
- **Health endpoint:** [https://dpeohr327wt9l.cloudfront.net/api/health](https://dpeohr327wt9l.cloudfront.net/api/health)
- **Source repository:** [https://github.com/9ducanh9/livecap](https://github.com/9ducanh9/livecap)

## Workshop Sections

| Section | Topic |
|---|---|
| **5.1** | Project overview, architecture, runtime flow, and AWS services used |
| **5.2** | AWS account, IAM, local toolchain, and cost estimates |
| **5.3** | Build and push the Docker image; deploy the ECS Fargate service |
| **5.4** | Deploy the React frontend, start live captioning, export transcripts |
| **5.5** | Testing, logs, metrics, alarms, security controls, and cost optimization |
| **5.6** | Clean-up all AWS resources and verify learning outcomes |
