---
title: "Proposal"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


## Project Title

**LiveCap: Real-Time Vietnamese-English Meeting Captions on AWS**

## 1. Problem

Bilingual meetings, workshops, and classrooms are difficult to follow when a
speaker talks quickly, changes between Vietnamese and English, or uses a
language that some participants are less comfortable with. Manual note-taking
is slow, interrupts attention, and does not provide immediate translation.

Existing meeting platforms may provide captions, but they can require paid
plans, platform-specific integration, or storage of full recordings. The
project needs a browser-based solution that works independently, displays
captions in real time, and minimizes retained conversation data.

## 2. Proposed Solution

LiveCap is a web application that:

1. captures microphone audio directly in the browser;
2. converts it to 16 kHz, 16-bit, mono PCM;
3. streams chunks over WebSocket to a FastAPI backend;
4. uses Amazon Transcribe Streaming for Vietnamese/English recognition;
5. sends finalized text to Amazon Translate;
6. displays original and translated captions side by side; and
7. exports finalized captions as a TXT file through a temporary S3 download URL.

Raw audio is not stored. Only finalized transcript text is eligible for export.

## 3. User Benefits

- **Immediate understanding:** participants can read the original and translated
  captions while the speaker is talking.
- **Platform independence:** the browser application does not depend on Zoom,
  Teams, or another meeting provider.
- **Accessible review:** finalized captions can be exported as a simple TXT file.
- **Privacy-conscious MVP:** microphone audio is streamed, not retained.
- **Operational transparency:** connection state, timer, errors, and export state
  are visible in the dashboard.

## 4. Project Scope

### Included

- English and Vietnamese bilingual captioning.
- Browser microphone capture and secure WebSocket streaming.
- Finalized caption rows with timestamp and speaker label.
- Heartbeat, bounded reconnect, session timeout, and abuse limits.
- TXT transcript export to private S3 with 14-day retention.
- AWS infrastructure code, CI checks, monitoring, and cost documentation.

### Not Included

- Login, meeting rooms, or transcript history.
- Direct meeting-platform integration.
- Raw audio recording or long-term conversation storage.
- AI summarization or speaker identity recognition.
- Active-active multi-task backend while session state remains in memory.

## 5. Solution Architecture

![alt](/images/workshop/target.png)

The regional deployment uses `ap-southeast-1`. CloudFront is the global public
entrypoint. The verified live environment uses one Fargate task behind a
multi-AZ ALB; the reviewed target later moves tasks to private subnets and adds
WAF, wake-on-demand, and `0 <-> 1` scaling.

## 6. AWS Services

| Service | Role | Reason |
| --- | --- | --- |
| CloudFront | HTTPS/WSS entrypoint and path routing | Global delivery and one stable public endpoint |
| Amazon S3 | Private frontend origin and transcript storage | Durable, encrypted object storage |
| Application Load Balancer | Health checks and WebSocket/API forwarding | Managed routing to healthy Fargate targets |
| Amazon ECS Fargate | Runs the FastAPI container | Managed containers without server administration |
| Amazon ECR | Stores immutable backend images | Reproducible SHA-tagged deployments |
| Amazon Transcribe | Streaming speech-to-text | Managed real-time transcription |
| Amazon Translate | Vietnamese-English translation | Managed translation for finalized text |
| CloudWatch | Logs and operational metrics | Monitoring and troubleshooting |
| IAM | Runtime and deployment permissions | Least-privilege access without embedded keys |

## 7. Delivery Plan

| Phase | Result |
| --- | --- |
| Requirements and prototype | User flow, WebSocket contract, initial React/FastAPI application |
| Realtime processing | PCM worklet, Transcribe streams, Translate, finalized captions |
| AWS deployment | S3/CloudFront frontend, ECR image, ALB and ECS Fargate backend |
| Hardening | Session guard, reconnect, heartbeat, retention, secret hygiene |
| Infrastructure | Terraform validation, remote-state bootstrap, reviewed target VPC/WAF/cost controls |
| Submission | CI gates, production smoke test, screenshots, bilingual documentation |

## 8. Risks and Mitigations

| Risk | Mitigation |
| --- | --- |
| Transcription accuracy varies with noise/accent | Use short clear demos and expose finalized results only |
| WebSocket disconnect interrupts captions | Ping/pong and three retries with 1/2/4-second backoff |
| Managed AI usage causes unexpected cost | 30-minute timeout, global/per-IP limits, no idle AI processing |
| Sensitive conversation data is retained | Do not store raw audio; delete transcript objects after 14 days |
| Single task fails | ALB health checks and ECS replacement; document session interruption |
| Infrastructure drift damages the live demo | Import/reconcile state, review plans, and use blue/green cutover |

## 9. Success Criteria

- The public CloudFront site and `/app` dashboard load successfully.
- The health endpoint reports a healthy backend.
- A microphone session produces finalized bilingual caption rows.
- Stop and reconnect paths do not leak active sessions or workers.
- Export creates a downloadable TXT transcript without storing raw audio.
- Backend, frontend, Terraform, and secret CI gates pass.
- The live architecture and the reviewed target are documented separately.