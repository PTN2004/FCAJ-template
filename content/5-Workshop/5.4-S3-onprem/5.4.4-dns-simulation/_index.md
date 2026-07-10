---
title: "End-to-End Verification"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

# End-to-End Walkthrough

This step guides you (and workshop attendees) through testing the entire LiveCap stack from an end-user perspective, followed by system checks to confirm the architecture is working correctly.

## 1. Access the Application

1. Open your browser and navigate to your CloudFront URL: `https://dpeohr327wt9l.cloudfront.net`
2. You will see the LiveCap Landing Page.
3. Click the **Start a live session** (or Open workspace) button to enter the main dashboard (`/app`).

## 2. Start a Live Session

1. In the main dashboard, click the **Start session** button.
2. When the browser requests microphone access, select **Allow**.
3. The connection status badge (top header) will transition from **READY** → **WAKING** (waking the backend) → **LIVE** (connected).

## 3. Real-Time Translation Experience

Test the system by speaking a few clear sentences into your microphone (you can speak in English or Vietnamese).

> Example: "Hello everyone, welcome to the workshop today. Live captions are working perfectly!"

Within 2–5 seconds, your 16kHz PCM audio stream has traversed CloudFront → ALB → Fargate → Amazon Transcribe → Amazon Translate, and the results are rendered on the screen.
You will see two columns of text (Vietnamese and English) populating side-by-side in real time:

![LiveCap Live Translation Session](/images/5-Workshop/livecap-dashboard.png)

## 4. End Session and Download Transcript

1. Click **Stop session** to end the session. The WebSocket connection will close cleanly.
2. Click the **Download transcript** (or Export) button.
3. The system will call the API to securely download the TXT file from Amazon S3 (via a presigned URL) to your computer. Open the TXT file to verify the contents.

---

## 5. Technical Verification (For Admins)

After experiencing the UI, you can inspect the backend logs to understand the data flow.

### Check CloudWatch Logs

Verify that the backend emitted structured logs during the session you just ran:

```powershell
aws logs tail livecap `
  --follow `
  --since 10m `
  --region ap-southeast-1 `
  --profile livecap-camgiacntn
```

You should see session lifecycle events: `session_opened`, `audio_chunk_received`, `transcribe_finalized`, `translate_success`, `session_closed`.

### Verified Production Results

After the cutover to the target architecture (VPC, private subnets, NAT, WAF, scale-to-zero), the full production flow passed all of the following:

| Test | Result |
|---|---|
| Health endpoint | `{"status":"healthy","version":"1.0.0"}` |
| WebSocket open | 101 Switching Protocols |
| Real 16 kHz PCM transcription (Vietnamese) | Finalized text returned |
| Real 16 kHz PCM transcription (English) | Finalized text returned |
| English → Vietnamese translation | Correct translation returned |
| Ping/pong heartbeat | 30-second interval maintained |
| Clean session end (Stop button) | Session closed, registry cleared |
| S3 transcript export | TXT object created in private bucket |
| Presigned URL download | File downloaded successfully |
| WAF blocking test | XSS and Log4J probes returned HTTP 403 |
| ECS scale-to-zero (idle 300 s) | Service scaled to 0 after 5 min idle |
| ECS self-healing (wake Lambda) | Scaled 0 → 1 and healthy within ≤60 s |



![End-to-end verification: transcription, translation, and export passing in production](/images/5-Workshop/livecap-transcribe-translate-export-verification.png)

![Runtime security verification: WAF blocking and session limits](/images/5-Workshop/livecap-runtime-security-verification.png)
