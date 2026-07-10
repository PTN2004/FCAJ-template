---
title: "Điều kiện tiên quyết"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Điều kiện tiên quyết

Trước khi bắt đầu thực hành workshop này, hãy đảm bảo bạn đã chuẩn bị đầy đủ
các mục dưới đây. Bỏ qua bất kỳ mục nào đều có thể khiến quá trình triển khai
hoặc kiểm thử bị lỗi giữa chừng.

## AWS Account và quyền IAM

Bạn cần một AWS account và một IAM identity có các quyền phù hợp. **Không dùng
root account** cho bất kỳ bước triển khai nào.

Tạo một IAM user riêng (ví dụ `camgiacntn` đang được dùng trong project này) và
đính kèm policy tùy chỉnh cấp đúng các action cần thiết:

| Nhóm quyền | Action cần thiết |
|---|---|
| ECR | `ecr:GetAuthorizationToken`, `ecr:BatchCheckLayerAvailability`, `ecr:PutImage`, `ecr:InitiateLayerUpload`, `ecr:UploadLayerPart`, `ecr:CompleteLayerUpload`, `ecr:DescribeRepositories`, `ecr:ListImages` |
| ECS | `ecs:RegisterTaskDefinition`, `ecs:UpdateService`, `ecs:DescribeServices`, `ecs:DescribeClusters`, `ecs:ListTasks`, `ecs:DescribeTasks` |
| S3 | `s3:ListBucket`, `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:GetBucketLocation` |
| CloudFront | `cloudfront:CreateInvalidation`, `cloudfront:ListDistributions`, `cloudfront:GetDistribution` |
| VPC & Mạng | `ec2:Describe*`, `ec2:CreateVpc`, `ec2:CreateSubnet`, `ec2:CreateInternetGateway`, `ec2:CreateNatGateway`, `ec2:CreateRouteTable`, `ec2:CreateSecurityGroup` và các action paired `Delete*`/`Attach*`/`Detach*` tương ứng |
| ELB | `elasticloadbalancing:*` (cần cho ALB, listener, target group) |
| Lambda | `lambda:CreateFunction`, `lambda:UpdateFunctionCode`, `lambda:UpdateFunctionConfiguration`, `lambda:AddPermission`, `lambda:GetFunction`, `lambda:DeleteFunction` |
| WAF | `wafv2:CreateWebACL`, `wafv2:AssociateWebACL`, `wafv2:GetWebACL`, `wafv2:UpdateWebACL`, `wafv2:DeleteWebACL` |
| CloudWatch Logs | `logs:CreateLogGroup`, `logs:PutRetentionPolicy`, `logs:DescribeLogGroups`, `logs:DeleteLogGroup` |
| Budgets | `budgets:ModifyBudget`, `budgets:ViewBudget` |
| STS | `sts:GetCallerIdentity` |
| IAM | `iam:PassRole` (bắt buộc cho Terraform để gán role cho ECS/Lambda), các action read-only để xác minh role |

Triển khai thực tế dùng IAM user `camgiacntn` (account `720459752315`). ECS Fargate
task dùng **task role** để có thể gọi Transcribe, Translate và S3 lúc runtime mà
không cần nhúng credential vào container image.

![Danh sách IAM users](/images/5-Workshop/5.2-Prerequisite/iam_users.png)

![IAM roles lọc theo livecap – hiển thị task role và execution role](/images/5-Workshop/5.2-Prerequisite/iam_livecap_roles.png)

## Bộ công cụ cần cài đặt

Cài đặt các công cụ sau trước khi bắt đầu. Phiên bản chính xác rất quan trọng
với Terraform và Python.

| Công cụ | Phiên bản cần dùng | Lý do |
|---|---|---|
| **Python** | 3.11+ | Backend FastAPI và 204 unit test |
| **Node.js** | 20 LTS | Build React/Vite và 11 frontend test |
| **Docker** (có `buildx`) | Stable hiện tại | Build image `linux/amd64` cho Fargate |
| **AWS CLI** | v2 | Xác thực ECR, đồng bộ S3, tạo CloudFront invalidation |
| **Terraform** | 1.10.5 | Format và validate hạ tầng; apply cần review thủ công |
| **Git** | Bất kỳ bản gần đây | Clone repo và tạo immutable image tag từ commit SHA |
| **Gitleaks** | Stable hiện tại | Quét secret trước mỗi lần `git push` |

Cấu hình AWS profile cho region `ap-southeast-1`:

```powershell
aws configure --profile livecap-camgiacntn
# AWS Access Key ID:     <key của bạn>
# AWS Secret Access Key: <secret của bạn>
# Default region:        ap-southeast-1
# Default output format: json
```

Xác minh kết nối:

```powershell
aws sts get-caller-identity --profile livecap-camgiacntn
```

Kết quả mong đợi:

```json
{
  "UserId": "AIDA...",
  "Account": "720459752315",
  "Arn": "arn:aws:iam::720459752315:user/camgiacntn"
}
```

## Kiến thức nền cần có

Workshop sẽ diễn ra nhanh hơn nếu bạn đã biết sơ những mục sau:

- **Docker** cơ bản: build, tag, push image.
- **AWS ECS/Fargate**: cluster, service, task definition, task role là gì.
- **WebSocket** cơ bản: trình duyệt upgrade HTTP lên kết nối hai chiều như thế nào.
- **S3** cơ bản: bucket, object, policy, presigned URL.
- **CloudFront** cơ bản: distribution, origin, behavior, OAC.
- **Terraform** cơ bản: `init`, `plan`, `apply`, provider — đủ để đọc file `.tf`
  mà không bị lạc.
- **Python** cơ bản: đủ để đọc handler FastAPI và pytest test.

Bạn **không cần** phải là chuyên gia ở bất kỳ mảng nào. Workshop sẽ giải thích
chi tiết từng bước.

## Ước tính chi phí

Chạy LiveCap trên AWS trong một phiên workshop ngắn tốn rất ít tiền, nhưng một
số tài nguyên có chi phí cố định trong khi đang được cấp phát.

| Tài nguyên | Chi phí ước tính |
|---|---|
| ECS Fargate (0.25 vCPU, 0.5 GB, `ap-southeast-1`) | ~$0.011/giờ |
| Application Load Balancer | ~$0.022/giờ + LCU |
| NAT Gateway | ~$0.059/giờ + data |
| CloudFront | 1 TB đầu miễn phí; $0.12/GB sau đó |
| S3 | Không đáng kể khi dùng workshop |
| Amazon Transcribe Streaming | $0.024/phút âm thanh |
| Amazon Translate | $15/triệu ký tự (2M đầu miễn phí/tháng) |
| AWS WAF | ~$5/Web ACL/tháng + $1/rule/tháng |

> **Mẹo:** Thực hiện phần Clean-up cuối workshop để xóa tất cả tài nguyên và
> tránh phát sinh chi phí liên tục. NAT Gateway và ALB là hai khoản chi phí
> cố định lớn nhất.

## Checklist an toàn trước khi bắt đầu

- [ ] Không dùng root account.
- [ ] IAM credential nằm trong `~/.aws/credentials`, không nằm trong bất kỳ
      file `.env` nào.
- [ ] File `.env`, `terraform.tfvars`, `backend.hcl` và `*.tfstate` thật đã được
      gitignore và sẽ không bị commit.
- [ ] Đã cài Gitleaks và sẽ chạy trước mỗi lần `git push`.
- [ ] Bạn hiểu rằng `terraform destroy` là **bước riêng, cần review** và không
      được chạy tự động.
