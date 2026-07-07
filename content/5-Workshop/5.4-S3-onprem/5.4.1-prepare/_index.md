---
title: "Build and Deliver the React Frontend"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

# Build and Deliver the React Frontend

## Step 1: Test and Build

```powershell
cd frontend
npm ci
npm test
npm run build
```

The build runs TypeScript checking and creates the Vite `dist` output. The
verified baseline has 11 passing frontend tests and zero known production npm
vulnerabilities at release time.

Verify the generated site before upload:

```powershell
npm run preview -- --host 127.0.0.1
# Open http://127.0.0.1:4173 and check / plus /app.
```

## Step 2: Store Static Assets Privately

The frontend S3 bucket blocks public access and uses encryption and versioning.
Browser access goes through CloudFront Origin Access Control rather than an S3
website endpoint or public bucket policy.

After confirming the AWS account and bucket name, synchronize the built output:

```powershell
aws s3 sync dist "s3://<frontend-bucket>" --delete `
  --region ap-southeast-1
```

`--delete` removes objects not present in `dist`; review the destination bucket
before running it.

## Step 3: Configure CloudFront Routes

| Path | Origin |
| --- | --- |
| `/` and static assets | Private frontend S3 bucket |
| `/api/*` | Application Load Balancer |
| `/ws/*` | Application Load Balancer with WebSocket upgrade |
| `/api/wake` | IAM-protected Lambda origin in the reviewed target only |

CloudFront terminates viewer HTTPS/WSS. The current CloudFront-to-ALB origin is
HTTP, which is recorded as a residual security gap for a later ACM/custom
origin-domain improvement.

## Step 4: Keep Runtime Configuration External

Local development uses `frontend/.env`. Production values are injected during
the build and are not hard-coded into reusable components. If no wake URL is
configured, the frontend skips the optional wake flow and connects normally.

After upload, invalidate CloudFront so the new files become visible without
waiting for the normal cache lifetime:

```powershell
aws cloudfront create-invalidation `
  --distribution-id <distribution-id> --paths "/*"
```

The deployed landing page and `/app` dashboard were verified on desktop and a
390 px mobile viewport.

![Step result: React landing page served through CloudFront](/images/3-Project/livecap-landing.png)
