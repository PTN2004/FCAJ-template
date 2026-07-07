---
title: "Security, Observability, Testing, and Cost"
date: 2026-07-05
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Security, Observability, Testing, and Cost

## Security Controls Already Implemented

- IAM roles replace static AWS credentials in containers and source code.
- Frontend and transcript S3 buckets are private; frontend access uses OAC.
- Transcript downloads use expiring presigned URLs.
- Raw audio is not stored; transcript objects expire after 14 days.
- CORS restricts the accepted frontend origin.
- Global/per-IP session limits and a 30-minute timeout reduce abuse exposure.
- Gitleaks runs in CI, while tfstate, tfvars, plans, and real `.env` files remain untracked.
- ECR images use immutable SHA-derived tags and scanning results are reviewed.

## WAF and Network Status

The Terraform target defines separate CloudFront (`CLOUDFRONT`) and ALB
(`REGIONAL`) Web ACLs with managed rules and rate rules in COUNT mode. COUNT
observes matching traffic without blocking it. These WAF associations are not
currently deployed, so the live demo must not be described as WAF-protected.

The current task has a public IP in the existing VPC. The reviewed target adds
two public and two private subnets across two AZs, places the ALB in public
subnets, moves Fargate to private subnets, and uses one NAT Gateway. One NAT is
a cost-sensitive single-AZ outbound tradeoff, not full network HA.

## Observability

- FastAPI emits structured session and integration logs to CloudWatch when the
  handler is available, with stdout fallback.
- Backend log retention is 14 days.
- ALB, ECS, and Lambda expose standard CloudWatch metrics.
- A dashboard for ECS CPU/memory, ALB traffic/health, wake Lambda, and WAF is
  defined in target Terraform without enabling paid Container Insights.

The dashboard and WAF metrics become useful only after the corresponding
target resources are applied and associated.

## CI and Verification

GitHub Actions validates every pull request and main push:

1. Gitleaks secret scan with full Git history.
2. Python 3.11 backend compile and pytest.
3. Node 20 frontend install, tests, and production build.
4. Terraform 1.10.5 format and validation with `-backend=false`.

CI deliberately performs no production deployment, Terraform apply, destroy,
or state migration.

## Cost Controls

- Transcribe and Translate are usage-based and run mainly during active sessions.
- Session duration/concurrency limits bound accidental AI usage.
- Transcript and log retention are both 14 days.
- A configurable `$50/month` AWS Budget exists in target Terraform; alerts are
  delayed billing signals, not real-time enforcement.
- Reviewed ECS wake/idle logic allows `0 <-> 1` tasks, but the live service
  currently remains at one task for submission stability.

ALB, NAT Gateway, and WAF have fixed or baseline charges while provisioned.
Scaling ECS to zero does not remove those costs.

## Residual Risks

- CloudFront currently connects to the ALB over HTTP.
- One in-memory session registry prevents safe multi-task scaling.
- One active task means replacement interrupts live WebSocket sessions.
- ECR base-image package findings remain tracked until compatible fixes exist.
