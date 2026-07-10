---
title: "Export Transcript via S3 Presigned URL"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Export Transcript via S3 Presigned URL

## How the Export Flow Works

After a caption session ends, you can download all finalized bilingual rows as
a plain-text file. The export flow keeps raw audio completely out of storage:

```
Browser (finalized rows)
  → POST /api/sessions/{session_id}/export
  → FastAPI on Fargate
  → Serialize rows to TXT format
  → PutObject to private S3 transcript bucket
  → Generate time-limited presigned URL (24 hours by default)
  → Return presigned URL to browser
  → Browser downloads the TXT file directly from S3
```

No raw audio is ever written to S3. Only the finalized transcript TXT is stored.
The transcript object is automatically deleted after **14 days** by an S3
lifecycle rule.

## Step 1 – Stop the Session

Click **Stop** in the dashboard. The backend:

1. Closes the Transcribe stream and Translate connection
2. Unregisters the session from the in-memory registry
3. Returns a final summary with the session ID

Click **Export TXT** in the dashboard. The frontend sends:

```http
POST https://dpeohr327wt9l.cloudfront.net/api/sessions/{session_id}/export
Content-Type: application/json

{
  "segments": [
    {
      "segment_id": "seg-001",
      "speaker_label": null,
      "text_vi": "Chào buổi sáng",
      "text_en": "Good morning",
      "spoken_language": "vi",
      "timestamp_start": 0.0,
      "timestamp_end": 2.1
    }
  ]
}
```

The backend responds with a presigned URL and expiry timestamp:

```json
{
  "download_url": "https://livecap-transcripts-dev-720459752315.s3.ap-southeast-1.amazonaws.com/transcripts/session-abc123.txt?X-Amz-...",
  "expires_at": "2026-07-09T12:00:00Z"
}
```

## Step 3 – Verify the S3 Object Was Created

Using the CLI:

```powershell
aws s3 ls "s3://livecap-transcripts-dev-720459752315/transcripts/" `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

You should see a `.txt` file for each exported session. The URL expires after
24 hours (configurable via `DOWNLOAD_LINK_EXPIRATION`).

## Step 4 – Verify the Lifecycle Rule

Transcript objects are automatically removed after 14 days. Check the rule:

```powershell
aws s3api get-bucket-lifecycle-configuration `
  --bucket livecap-transcripts-dev-720459752315 `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

Expected output:

```json
{
  "Rules": [{
    "ID": "delete-old-transcripts",
    "Status": "Enabled",
    "Filter": {"Prefix": "transcripts/"},
    "Expiration": {"Days": 14}
  }]
}
```

## Privacy Guarantees

| What is stored | What is NOT stored |
|---|---|
| Finalized bilingual TXT transcript | Raw microphone audio |
| Session metadata (duration, IP hash) | Partial/interim Transcribe results |
| CloudWatch application logs (14-day retention) | User identity or PII |
