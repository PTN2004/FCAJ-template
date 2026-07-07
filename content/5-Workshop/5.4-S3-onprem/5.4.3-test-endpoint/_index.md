---
title: "Translate and Export Finalized Transcripts"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Translate and Export Finalized Transcripts

## Translation Flow

Amazon Translate is called only for finalized text selected from the
Transcribe streams. The backend returns a normalized caption message containing
the session context, speaker label, timestamp, original text, translated text,
and final-state marker. The dashboard appends finalized messages to the two
caption columns.

## TXT Export Flow

1. The user selects **Export TXT** after captions have been finalized.
2. The frontend sends the finalized session rows to the export API.
3. FastAPI validates and serializes the bilingual transcript.
4. The backend writes a TXT object to the private transcript S3 bucket.
5. S3 returns a time-limited presigned download URL through the backend.

The bucket is encrypted, blocks public access, and removes transcript objects
after 14 days. Presigned links expire independently (24 hours by default).

## Data Boundary

LiveCap does not upload or store raw microphone audio in the MVP. Audio exists
only in the browser/WebSocket/Transcribe streaming path. S3 contains finalized
text exports, which reduces storage cost and limits retained sensitive data.

## Failure Handling

Translate or export failures return structured errors to the UI and are logged
without exposing credentials. A session cleanup still runs when a Transcribe
stream or internal worker fails, preventing leaked active-session counts.
