---
title: "Workshop"
date: 2026-07-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Technical Workshop

## LiveCap: Real-Time Bilingual Captions on AWS

This workshop documents the LiveCap implementation completed for the final
project. LiveCap captures microphone audio in a React web application, streams
16 kHz PCM over WebSocket to a FastAPI backend on Amazon ECS Fargate, creates
captions with Amazon Transcribe, translates finalized text with Amazon
Translate, and displays Vietnamese and English captions side by side.

The public demo uses CloudFront, private S3 frontend hosting, an Application
Load Balancer, one ECS Fargate task, ECR, a private transcript bucket, and
CloudWatch. Export stores finalized TXT transcripts only; raw audio is not
stored.

This chapter distinguishes the verified live environment from the reviewed
Terraform target. Private Fargate networking, NAT, WAF, wake-on-demand,
scale-to-zero, the dashboard, and the budget are target controls and are not
presented as already deployed.

## Workshop Sections

1. Product, architecture, and runtime flows.
2. Development and AWS prerequisites.
3. Containerized FastAPI backend on ECS Fargate.
4. Frontend delivery and real-time AWS integrations.
5. Security, observability, testing, and cost controls.
6. Release safety, verified results, and remaining work.
