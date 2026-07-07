---
title: "Stream Audio and Produce Live Captions"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Stream Audio and Produce Live Captions

## Audio and WebSocket Pipeline

```text
Microphone
  -> Web Audio worklet
  -> 16 kHz, 16-bit, mono PCM chunks
  -> CloudFront WSS /ws/transcribe
  -> ALB
  -> FastAPI on Fargate
  -> Amazon Transcribe Streaming
```

The microphone starts only after the backend is ready and the WebSocket is
open. Chunks produced while the socket is unavailable are dropped, preventing
unbounded client buffering.

## Bilingual Processing

With `BILINGUAL_DUAL_STREAM=true`, the backend fans the same PCM input into
Vietnamese and English Transcribe streams, arbitrates results, and translates
the selected finalized segment into the other language. Partial results may be
shown as transient state, but only finalized segments become permanent rows.

## Connection Resilience

- The frontend sends `ping` every 30 seconds; the backend returns `pong`.
- An unexpected recording-time disconnect retries at most three times with
  1, 2, and 4 second backoff.
- A reconnect creates a new backend session while preserving finalized rows.
- If retries fail, audio capture stops and the user must restart the session.
- Stop, disconnect, timeout, Transcribe error, and internal exceptions all run
  queue, worker, stream, and registry cleanup.

## Session Guardrails

The UI and backend share a 30-minute maximum duration. The backend also rejects
excess global/per-IP sessions before starting managed AI service work. These
limits bound accidental Transcribe and Translate usage for the MVP.

![Step result: finalized captions returned to the live dashboard](/images/3-Project/livecap-dashboard.png)
