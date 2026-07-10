---
title: "Frontend Delivery & Real-Time Integrations"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Frontend Delivery & Real-Time Integrations

This section covers deploying the React frontend to S3 and CloudFront, then
connecting it to the backend for live bilingual captioning.

## What You Will Set Up

| Component | Technology | Where |
|---|---|---|
| Static site | React 18 + Vite + TypeScript | Private S3 bucket |
| CDN delivery | Amazon CloudFront with OAC | Global edge |
| WebSocket | Browser ↔ FastAPI | Through CloudFront → ALB → Fargate |
| Audio processing | Web Audio API worklet | In the browser |
| AI services | Transcribe Streaming + Translate | Called by the Fargate backend |
| Transcript export | S3 presigned URL | Private transcript bucket |

The current live S3 buckets:

![S3 bucket list showing frontend and transcript buckets](/images/5-Workshop/5.4-S3-onprem/s3_buckets_list.png)
