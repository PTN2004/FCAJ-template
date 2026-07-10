---
title: "Testing, Security & Cost Controls"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Testing, Security & Cost Controls

## GitHub Actions Quality Gates

The `main` branch workflow runs four independent jobs before a change is
considered to have passed the quality gate:

- Scan the full Git history for secrets with Gitleaks.
- Compile the backend and run 204 pytest tests.
- Run frontend tests and create the production build.
- Check Terraform formatting, `init -backend=false`, and `validate`.

![Successful LiveCap GitHub Actions run](/images/5-Workshop/github-actions-ci.png)

CI is validation-only. The workflow does not deploy, run `terraform apply`,
destroy resources, or migrate Terraform state.

## Functional Testing

### Backend – 204 Unit Tests

```powershell
cd backend
python -m pytest -v
```

The test suite covers:

- WebSocket session lifecycle (open, audio, close, timeout)
- Session registry (global and per-IP limits)
- Transcribe stream management (start, partial results, finalize)
- Translate integration (finalized-only, language selection)
- S3 export (TXT serialization, presigned URL generation)
- Error handling (Transcribe error, connection drop, cleanup)

All 204 tests pass on Python 3.11. Running time: approximately 8 seconds.

### Frontend – 11 Vitest Tests

```powershell
cd frontend
npm test
```

Covers: component rendering, WebSocket hook state transitions, session timer,
and microphone permission error states. Zero production vulnerabilities at
release time.

### Terraform – Syntax Validation Only

CI never applies infrastructure. It validates format and syntax only:

```powershell
terraform -chdir=infrastructure/terraform fmt -check
terraform -chdir=infrastructure/terraform init -backend=false
terraform -chdir=infrastructure/terraform validate
```

### Secret Scanning with Gitleaks

```powershell
gitleaks detect --source . --verbose
```

Gitleaks runs against the full Git history. A clean scan is required before any
`git push`.

## Logs and Metrics

### CloudWatch Application Logs

The FastAPI backend emits structured JSON logs to the `/ecs/livecap-backend-dev`
CloudWatch log group. Log retention is 14 days.

```powershell
# Stream live logs
aws logs tail /ecs/livecap-backend-dev --follow --region ap-southeast-1 --profile livecap-camgiacntn
```

Key log events you will see during a session:
- `session_start` – new session opened, with session ID and client IP hash
- `websocket_connect` – WebSocket connection established
- `websocket_disconnect` – client disconnected or timed out
- `session_end` – session closed, with duration and reason
- `integration_error` – error from Transcribe, Translate, or S3

![CloudWatch log groups showing the livecap log group](/images/5-Workshop/5.5-Policy/cloudwatch_log_groups.png)

![Livecap log group detail with log streams](/images/5-Workshop/5.5-Policy/cloudwatch_livecap_log_groups.png)

![CloudWatch log stream detail showing backend session events](/images/5-Workshop/5.5-Policy/cloudwatch_log_group_detail.png)

![Sample log events from a live transcription session](/images/5-Workshop/5.5-Policy/cloudwatch_log_events.png)

### Key Metrics to Monitor

| Metric | Source | What it tells you |
|---|---|---|
| `HTTPCode_Target_5XX_Count` | ALB | Backend errors returned to CloudFront |
| `HealthyHostCount` | ALB Target Group | Number of healthy Fargate tasks |
| `CPUUtilization` | ECS | Task CPU usage (alert if > 80%) |
| `MemoryUtilization` | ECS | Task memory usage |
| `BlockedRequests` | WAF | WAF is actively blocking threats |
| `4xx/5xx error rate` | CloudFront | End-to-end error rate |

### Setting a CloudWatch Alarm (Example)

Create an alarm that fires when the ALB returns 5XX errors:

```powershell
aws cloudwatch put-metric-alarm `
  --alarm-name "livecap-alb-5xx" `
  --metric-name "HTTPCode_Target_5XX_Count" `
  --namespace "AWS/ApplicationELB" `
  --statistic Sum `
  --period 300 `
  --threshold 5 `
  --comparison-operator GreaterThanOrEqualToThreshold `
  --evaluation-periods 1 `
  --alarm-actions "arn:aws:sns:ap-southeast-1:720459752315:livecap-alerts" `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

## Security Controls

### Already Deployed

| Control | Implementation |
|---|---|
| No root account usage | Deployment uses IAM user `camgiacntn` |
| IAM least privilege | Separate task execution role and task role |
| No hardcoded credentials | IAM roles only; no keys in `.env`, images, or Git |
| Private S3 frontend | Block public access + OAC origin |
| Private S3 transcripts | Block public access; presigned URLs expire in 24 hours |
| HTTPS everywhere | CloudFront terminates viewer TLS |
| WAF at CloudFront and ALB | Managed rules in BLOCK mode; rate-based rules |
| CORS restriction | `ALLOWED_ORIGIN` limits accepted frontend origin |
| Session limits | 4 global + 1 per IP prevent runaway Transcribe cost |
| Transcript expiry | 14-day S3 lifecycle rule; no raw audio stored |
| Secret scanning | Gitleaks runs in CI on full Git history |
| Immutable image tags | Git SHA tags prevent accidental `latest` drift |

### WAF Verification

Both Web ACLs are in BLOCK mode. Production probes confirmed:

- Cross-site scripting (XSS) attempts ? HTTP 403
- Log4J exploit strings ? HTTP 403

![WAF runtime security verification – blocked XSS and Log4J probes](/images/5-Workshop/livecap-runtime-security-verification.png)

## Cost Optimization

### Current Cost Drivers (ap-southeast-1)

| Resource | Cost basis | Optimization |
|---|---|---|
| ECS Fargate | Per vCPU-second + GB-second | Scale to 0 when idle (target feature) |
| ALB | Fixed hourly + LCU | Incurs cost even when ECS is at 0; only removed by destroying the full stack |
| NAT Gateway | Hourly + per-GB data | Single NAT in one AZ (cost trade-off) |
| Amazon Transcribe | Per minute of audio | Session limits cap usage |
| Amazon Translate | Per million characters | Only finalized segments are translated |
| CloudWatch | Log ingestion + storage | 14-day retention limits storage cost |
| WAF | Per ACL + per rule | Fixed baseline; worth it for blocking |

### Cost-Saving Practices Already Applied

- **14-day log and transcript retention** – avoids indefinite storage growth.
- **Session duration limit (30 min)** – bounds maximum Transcribe minutes per session.
- **Session concurrency limits** – prevent abuse-driven Transcribe/Translate costs.
- **Translate only finalized text** – partial/interim results are discarded before
  translation, saving characters.
- **ECS scale-to-zero (target)** – Wake Lambda brings desired count from 0 ? 1
  on first request; idle scale-down returns it to 0 after inactivity.

### AWS Budget (Target Terraform)

An AWS Budget alert is defined in the Terraform target at **$50/month**. It sends
a notification directly to an **email subscriber** when actual or forecast spend
approaches the threshold (not via SNS).

> **Note:** Budget alerts are a delayed billing signal, not real-time enforcement.
> They help you notice runaway costs within hours, not seconds.
