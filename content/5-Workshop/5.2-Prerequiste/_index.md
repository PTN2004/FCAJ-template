---
title: "Prerequisites"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Prerequisites

## Local Toolchain

| Tool | Reference version | Purpose |
| --- | --- | --- |
| Python | 3.11+ | FastAPI backend and tests |
| Node.js | 20 | React/Vite build and Vitest |
| Docker | Current stable | Build and smoke-test the backend image |
| AWS CLI | v2 | Read and operate the selected AWS account/region |
| Terraform | 1.10.5 | Format and validate infrastructure code |
| Git and Gitleaks | Current stable | Source control and secret scanning |

Use `ap-southeast-1` for the regional LiveCap resources. CloudFront is global;
a CloudFront-scope WAF is managed through `us-east-1` only because AWS requires
that provider region for the global Web ACL.

## AWS Access Model

Local development uses an AWS profile. The Fargate task uses an ECS task role;
credentials are not copied into `.env`, Docker images, or Git. Runtime access
is limited to the required Transcribe, Translate, transcript S3, CloudWatch,
and optional ECS idle-scaling actions.

## Backend Configuration

Copy `backend/.env.example` to an ignored local `.env` and configure:

| Variable | Reference value | Responsibility |
| --- | --- | --- |
| `AWS_REGION` | `ap-southeast-1` | Region for Transcribe, Translate, and S3 |
| `S3_BUCKET` | environment-specific | Private transcript export bucket |
| `SESSION_TIMEOUT` | `1800` | Maximum session duration in seconds |
| `MAX_CONCURRENT_SESSIONS` | `4` | Process-wide active-session limit |
| `MAX_SESSIONS_PER_IP` | `1` | Active-session limit per client IP |
| `BILINGUAL_DUAL_STREAM` | `true` | Parallel Vietnamese and English streams |
| `ALLOWED_ORIGIN` | frontend origin | CORS allowlist |
| `CLOUDWATCH_LOG_GROUP` | `livecap` | Structured logging target |

Idle scaling remains safe-by-default with
`ENABLE_IDLE_SCALE_DOWN=false`. `ECS_CLUSTER_NAME` and `ECS_SERVICE_NAME` may
be empty in local development. The retired `MAX_SPEAKERS` variable is not part
of the active configuration.

## Frontend Configuration

| Variable | Local reference | Responsibility |
| --- | --- | --- |
| `VITE_API_BASE_URL` | `http://127.0.0.1:8000` | REST API base URL |
| `VITE_WS_URL` | `ws://127.0.0.1:8000/ws/transcribe` | WebSocket endpoint |
| `VITE_WAKE_BACKEND_URL` | empty | Optional target wake path |
| `VITE_BACKEND_HEALTH_URL` | local `/api/health` | Backend readiness polling |
| `VITE_BACKEND_WAKE_TIMEOUT_SECONDS` | `120` | Maximum startup wait |
| `VITE_MAX_SESSION_SECONDS` | `1800` | UI session countdown |

The microphone starts only after backend readiness. Audio generated while the
socket is unavailable is dropped rather than buffered indefinitely.

## Safety Prerequisites

- Do not commit real `.env`, `terraform.tfvars`, backend config, state, or plans.
- Run Gitleaks before publishing changes.
- Use `terraform init -backend=false` in CI; CI never applies infrastructure.
- Reconcile existing AWS resources into state and review a plan before any apply.
- Process only audio for which the operator has permission.
