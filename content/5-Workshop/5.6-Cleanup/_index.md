---
title: "Clean-up & Learning Outcomes"
date: 2026-07-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Clean-up & Learning Outcomes

## Clean-up

After completing the workshop, delete all resources to avoid ongoing charges.

> [!CAUTION]
> **The full LiveCap stack is managed by Terraform.** Deleting resources
> manually with the CLI can cause **Terraform state drift** – leading to errors
> on the next `apply` or duplicate resource creation. Always prefer
> `terraform destroy` after carefully reviewing the plan.

### Step 1 – Review the Terraform destroy plan

```powershell
cd infrastructure/terraform   # or the directory that holds your Terraform state

# Preview everything that will be deleted
terraform plan -destroy -out=destroy.tfplan

# Read the output carefully, paying attention to:
# - WAF association must be detached before ALB/CloudFront
# - Listeners and Target Groups must be removed before the ALB
# - S3 objects (including versioned objects) must be emptied before bucket deletion
# - NAT Gateway takes a few minutes to delete; EIP must be released after
```

Only proceed when you have confirmed the resource list is correct.

### Step 2 – Empty S3 buckets (required before destroy)

Terraform does not remove S3 objects or versioned objects automatically.
Empty them manually first:

```powershell
# Empty the frontend bucket (no versioning)
aws s3 rm s3://livecap-frontend-dev-720459752315 --recursive `
  --region ap-southeast-1 --profile livecap-camgiacntn

# Empty the transcript bucket (including delete markers and versions)
aws s3api list-object-versions `
  --bucket livecap-transcripts-dev-720459752315 `
  --region ap-southeast-1 `
  --query "Versions[].{Key:Key,VersionId:VersionId}" `
  --output json | ConvertFrom-Json | ForEach-Object {
    aws s3api delete-object `
      --bucket livecap-transcripts-dev-720459752315 `
      --key $_.Key `
      --version-id $_.VersionId `
      --region ap-southeast-1 --profile livecap-camgiacntn
  }
```

### Step 3 – Terraform destroy

```powershell
# Apply the plan reviewed in Step 1
terraform apply destroy.tfplan
```

Terraform handles the correct dependency order:
WAF disassociation → Listeners/Target Groups → ALB → ECS Service → ECS Cluster
→ Lambda → ECR → NAT Gateway → Subnets → VPC → IAM Roles → CloudWatch Log Groups
→ S3 Buckets → CloudFront Distribution → Budget Alert.

### Step 4 – Verify no resources remain

After `terraform destroy` completes, spot-check manually:

```powershell
# ECS cluster
aws ecs list-clusters --region ap-southeast-1 --profile livecap-camgiacntn

# ECR repositories
aws ecr describe-repositories --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "repositories[?contains(repositoryName, 'livecap')].repositoryName"

# S3 buckets
aws s3 ls --profile livecap-camgiacntn | Select-String "livecap"

# ALB
aws elbv2 describe-load-balancers --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "LoadBalancers[?contains(LoadBalancerName, 'livecap')].LoadBalancerName"

# Lambda
aws lambda list-functions --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "Functions[?contains(FunctionName, 'livecap')].FunctionName"

# CloudWatch log groups
aws logs describe-log-groups --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "logGroups[?contains(logGroupName, 'livecap')].logGroupName"

# Unassociated Elastic IPs left over from NAT Gateway
aws ec2 describe-addresses --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "Addresses[?AssociationId==null].PublicIp"
```

If any unassociated Elastic IPs remain, release them via EC2 Console →
Elastic IPs → Actions → Release Elastic IP address.

### Step 5 – Force-delete ECR repository (if Terraform cannot remove it)

```powershell
aws ecr delete-repository `
  --repository-name livecap-backend `
  --force `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

### Step 6 – Delete the AWS Budget (if not managed by Terraform)

```powershell
aws budgets delete-budget `
  --account-id 720459752315 `
  --budget-name livecap-monthly-budget `
  --profile livecap-camgiacntn
```

---




---


## Learning Outcomes

After completing this workshop, you should be able to:

### Architecture

- Explain what problem LiveCap solves and who benefits from it
- Describe the AWS architecture: CloudFront, S3, ALB, ECS Fargate, ECR,
  Transcribe, Translate, CloudWatch, WAF, Lambda
- Explain the role of each service and how they are connected
- Trace the full request path from browser microphone to caption display

### Deployment

- Build a Docker image for `linux/amd64` from a Python FastAPI backend
- Push an immutable image to Amazon ECR using Git SHA tags
- Create an ECS task definition and update a Fargate service
- Build a React/Vite frontend and sync it to a private S3 bucket
- Create a CloudFront invalidation to refresh edge caches
- Configure CloudFront path behaviors for static assets, API, and WebSocket

### Testing

- Run the backend test suite (pytest) and frontend tests (Vitest)
- Verify the health endpoint through CloudFront
- Test a live WebSocket transcription session end-to-end
- Export a transcript and verify the S3 object and presigned URL

### Observability

- Read structured application logs in CloudWatch
- Understand ALB and ECS metrics and what they indicate
- Create a basic CloudWatch alarm for error conditions

### Security

- Understand the IAM least-privilege model (task execution vs. task role)
- Verify that S3 buckets are private and only accessible through CloudFront OAC
- Understand how WAF blocks managed threats without touching application code
- Explain why raw audio is never stored and how transcript expiry works

### Cost

- Identify the fixed-cost resources (ALB, NAT Gateway, WAF) vs. usage-based
  services (Transcribe, Translate)
- Explain how session limits and retention rules bound ongoing costs
- Understand the role of an AWS Budget alert and its limitations
- Clean up all resources to avoid ongoing charges after the workshop
