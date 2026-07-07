---
title: "Build and Publish the Backend Container"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

# Build and Publish the Backend Container

## Step 1: Verify the Backend

The FastAPI application exposes `/api/health`, REST export routes, and
`/ws/transcribe`. Before building an image, run:

```powershell
cd backend
python -m pip install -r requirements-dev.txt
python -m compileall app
python -m pytest
```

The verified submission baseline contains 204 passing backend tests.

## Step 2: Build for Fargate

The repository Dockerfile installs pinned Python dependencies, copies the
application, exposes port 8000, and includes a container health check. Build
for the ECS runtime architecture and smoke-test `/api/health` locally.

```powershell
$imageTag = "$(git rev-parse --short HEAD)-amd64"
docker buildx build --platform linux/amd64 --provenance=false `
  -t "livecap-backend:$imageTag" --load .
docker run --rm -d --name livecap-smoke -p 8000:8000 `
  --env-file .env "livecap-backend:$imageTag"
Invoke-RestMethod http://127.0.0.1:8000/api/health
docker stop livecap-smoke
```

Do not bake `.env` or AWS credentials into the image. Local execution uses an
AWS profile; ECS injects settings and uses IAM roles at runtime.

## Step 3: Push an Immutable Image

ECR stores the backend image. The deployment tag is derived from the Git commit
instead of `latest`, so a task definition points to a reproducible artifact.
The verified public demo uses image tag `1ef4250-amd64`.

```text
source commit -> Docker image -> immutable ECR tag -> ECS task definition
```

Authenticate Docker and push the same immutable tag. Replace the repository
name only when the target environment uses another ECR repository.

```powershell
$region = "ap-southeast-1"
$accountId = aws sts get-caller-identity --query Account --output text
$registry = "$accountId.dkr.ecr.$region.amazonaws.com"
$repository = "$registry/livecap-backend"
aws ecr get-login-password --region $region |
  docker login --username AWS --password-stdin $registry
docker tag "livecap-backend:$imageTag" "$repository:$imageTag"
docker push "$repository:$imageTag"
```

## Step 4: Create a Task Definition Revision

Update the task definition image URI to the pushed SHA tag, keep container port
8000, inject environment values, and attach the execution/task roles. Register
a new revision rather than mutating an old artifact. Then update the ECS service
to that reviewed revision and wait for stability.

```powershell
aws ecs update-service --region ap-southeast-1 `
  --cluster <cluster-name> --service <service-name> `
  --task-definition <task-definition-family:revision>
aws ecs wait services-stable --region ap-southeast-1 `
  --cluster <cluster-name> --services <service-name>
```

Do not use these placeholders blindly in production. Confirm the account,
cluster, service, task-definition diff, and rollback revision first.

## IAM Separation

- **Task execution role:** pull from ECR and write container logs.
- **Task role:** call Transcribe, Translate, transcript S3, CloudWatch, and the
  optional least-privilege ECS scale-down action.
- **Wake Lambda role:** only describe/update the selected ECS service in the
  reviewed target.

This separation prevents the application from inheriting broad deployment
permissions.
