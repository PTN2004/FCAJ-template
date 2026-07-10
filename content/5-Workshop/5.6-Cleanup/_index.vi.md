---
title: "Clean-up & Kết quả cần đạt"
date: 2026-07-08
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

# Clean-up & Kết quả cần đạt

## Clean-up

Sau khi hoàn thành workshop, hãy xóa tất cả tài nguyên để tránh phát sinh chi
phí liên tục.

> [!CAUTION]
> **Toàn bộ stack LiveCap được quản lý bởi Terraform.** Xóa thủ công bằng CLI
> có thể gây **Terraform state drift** – khiến lần `apply` tiếp theo bị lỗi hoặc
> tạo ra tài nguyên trùng lặp. Luôn ưu tiên dùng `terraform destroy` sau khi
> đã review plan cẩn thận.

### Bước 1 – Review Terraform plan trước khi destroy

```powershell
cd infrastructure/terraform   # hoặc thư mục chứa Terraform state của bạn

# Xem toàn bộ tài nguyên sẽ bị xóa
terraform plan -destroy -out=destroy.tfplan

# Đọc kỹ output, đặc biệt chú ý:
# - WAF association phải được detach trước ALB/CloudFront
# - Listener và Target Group phải xóa trước ALB
# - S3 object (kể cả versioned objects) phải làm trống trước khi xóa bucket
# - NAT Gateway cần vài phút để delete; EIP phải release sau
```

Chỉ tiến hành khi bạn đã xác nhận danh sách tài nguyên là đúng.

### Bước 2 – Làm trống S3 buckets (bắt buộc trước khi destroy)

Terraform không tự xóa S3 object và versioned object. Làm trống thủ công trước:

```powershell
# Làm trống frontend bucket (không có versioning)
aws s3 rm s3://livecap-frontend-dev-720459752315 --recursive `
  --region ap-southeast-1 --profile livecap-camgiacntn

# Làm trống transcript bucket (bao gồm delete markers và versions)
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

### Bước 3 – Terraform destroy

```powershell
# Apply plan đã review ở Bước 1
terraform apply destroy.tfplan
```

Terraform sẽ xử lý đúng thứ tự dependency:
WAF disassociation → Listeners/Target Groups → ALB → ECS Service → ECS Cluster
→ Lambda → ECR → NAT Gateway → Subnets → VPC → IAM Roles → CloudWatch Log Groups
→ S3 Buckets → CloudFront Distribution → Budget Alert.

### Bước 4 – Xác minh còn tài nguyên nào chưa xóa

Sau khi `terraform destroy` hoàn tất, kiểm tra thủ công:

```powershell
# Kiểm tra ECS cluster
aws ecs list-clusters --region ap-southeast-1 --profile livecap-camgiacntn

# Kiểm tra ECR repository
aws ecr describe-repositories --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "repositories[?contains(repositoryName, 'livecap')].repositoryName"

# Kiểm tra S3 buckets
aws s3 ls --profile livecap-camgiacntn | Select-String "livecap"

# Kiểm tra ALB
aws elbv2 describe-load-balancers --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "LoadBalancers[?contains(LoadBalancerName, 'livecap')].LoadBalancerName"

# Kiểm tra Lambda
aws lambda list-functions --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "Functions[?contains(FunctionName, 'livecap')].FunctionName"

# Kiểm tra log group
aws logs describe-log-groups --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "logGroups[?contains(logGroupName, 'livecap')].logGroupName"

# Kiểm tra Elastic IP còn lại (từ NAT Gateway)
aws ec2 describe-addresses --region ap-southeast-1 --profile livecap-camgiacntn `
  --query "Addresses[?AssociationId==null].PublicIp"
```

Nếu còn Elastic IP không được liên kết, release thủ công qua EC2 Console →
Elastic IPs → Actions → Release Elastic IP address.

### Bước 5 – Xóa ECR image trước khi xóa repository (nếu Terraform không xóa được)

```powershell
# Force xóa repository và tất cả image
aws ecr delete-repository `
  --repository-name livecap-backend `
  --force `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

### Bước 6 – Hủy AWS Budget (nếu không do Terraform quản lý)

```powershell
aws budgets delete-budget `
  --account-id 720459752315 `
  --budget-name livecap-monthly-budget `
  --profile livecap-camgiacntn
```


---


---

## Kết quả cần đạt

Sau khi hoàn thành workshop này, bạn cần đạt được:

### Kiến trúc

- Giải thích được LiveCap giải quyết vấn đề gì và ai hưởng lợi
- Mô tả được kiến trúc AWS: CloudFront, S3, ALB, ECS Fargate, ECR,
  Transcribe, Translate, CloudWatch, WAF, Lambda
- Giải thích vai trò của từng dịch vụ và cách chúng kết nối với nhau
- Theo dõi được toàn bộ đường đi request từ microphone trình duyệt đến hiển thị phụ đề

### Triển khai

- Build Docker image `linux/amd64` từ backend Python FastAPI
- Push image bất biến lên Amazon ECR bằng Git SHA tag
- Tạo ECS task definition và cập nhật Fargate service
- Build React/Vite frontend và sync lên S3 private
- Tạo CloudFront invalidation để refresh edge cache
- Cấu hình CloudFront path behavior cho static asset, API và WebSocket

### Kiểm thử

- Chạy bộ test backend (pytest) và frontend (Vitest)
- Xác minh health endpoint qua CloudFront
- Test một phiên phiên âm WebSocket trực tiếp end-to-end
- Export transcript và xác minh S3 object và presigned URL

### Quan sát hệ thống

- Đọc được structured application log trong CloudWatch
- Hiểu metric ALB và ECS và ý nghĩa của chúng
- Tạo được CloudWatch alarm cơ bản cho điều kiện lỗi

### Bảo mật

- Hiểu mô hình IAM least-privilege (task execution role vs. task role)
- Xác minh S3 bucket private chỉ truy cập được qua CloudFront OAC
- Hiểu cách WAF chặn managed threat mà không cần chạm vào code ứng dụng
- Giải thích tại sao raw audio không bao giờ được lưu và cách transcript expiry hoạt động

### Chi phí

- Xác định được tài nguyên chi phí cố định (ALB, NAT Gateway, WAF) so với
  dịch vụ theo mức sử dụng (Transcribe, Translate)
- Giải thích cách giới hạn session và lifecycle rule kiểm soát chi phí liên tục
- Hiểu vai trò của AWS Budget alert và giới hạn của nó
- Xóa sạch tất cả tài nguyên để tránh phát sinh chi phí sau workshop
