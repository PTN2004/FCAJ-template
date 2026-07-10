---
title: "Kiểm thử, Bảo mật & Kiểm soát chi phí"
date: 2026-07-08
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Kiểm thử, Bảo mật & Kiểm soát chi phí

## Quality gate trên GitHub Actions

Workflow trên nhánh `main` chạy bốn job độc lập trước khi thay đổi được xem là
đạt quality gate:

- Quét secret bằng Gitleaks trên toàn bộ lịch sử Git.
- Compile backend và chạy 204 test bằng pytest.
- Chạy frontend test và tạo production build.
- Kiểm tra Terraform format, `init -backend=false` và `validate`.

![GitHub Actions LiveCap đã chạy thành công](/images/5-Workshop/github-actions-ci.png)

CI chỉ thực hiện validation. Workflow không deploy, không chạy
`terraform apply`, không destroy tài nguyên và không migrate Terraform state.

## Kiểm thử chức năng

### Backend – 204 Unit Test

```powershell
cd backend
python -m pytest -v
```

Bộ test bao gồm:

- Vòng đời WebSocket session (mở, audio, đóng, timeout)
- Session registry (giới hạn global và theo IP)
- Quản lý Transcribe stream (bắt đầu, partial result, finalize)
- Tích hợp Translate (chỉ finalized, chọn ngôn ngữ)
- S3 export (serialize TXT, tạo presigned URL)
- Xử lý lỗi (lỗi Transcribe, mất kết nối, dọn dẹp)

Tất cả 204 test pass trên Python 3.11. Thời gian chạy: khoảng 8 giây.

### Frontend – 11 Vitest Test

```powershell
cd frontend
npm test
```

Bao gồm: render component, chuyển trạng thái WebSocket hook, session timer và
trạng thái lỗi quyền microphone. Không có lỗ hổng production tại thời điểm
phát hành.

### Terraform – Chỉ kiểm tra cú pháp

CI không bao giờ apply hạ tầng. Chỉ validate format và cú pháp:

```powershell
terraform -chdir=infrastructure/terraform fmt -check
terraform -chdir=infrastructure/terraform init -backend=false
terraform -chdir=infrastructure/terraform validate
```

### Quét secret bằng Gitleaks

```powershell
gitleaks detect --source . --verbose
```

Gitleaks chạy trên toàn bộ lịch sử Git. Scan sạch là điều kiện bắt buộc trước
mỗi lần `git push`.

## Log và Metric

### Application Log trên CloudWatch

Backend FastAPI emit structured JSON log vào log group `/ecs/livecap-backend-dev`
của CloudWatch. Retention log là 14 ngày.

```powershell
# Stream log trực tiếp
aws logs tail /ecs/livecap-backend-dev --follow --region ap-southeast-1 --profile livecap-camgiacntn
```

Các sự kiện log chính bạn sẽ thấy trong một phiên:
- `session_start` – mở phiên mới, kèm session ID và client IP hash
- `websocket_connect` – kết nối WebSocket được thiết lập
- `websocket_disconnect` – client ngắt kết nối hoặc timeout
- `session_end` – phiên đóng, kèm thời lượng và lý do
- `integration_error` – lỗi từ Transcribe, Translate hoặc S3

![CloudWatch log group – log group livecap](/images/5-Workshop/5.5-Policy/cloudwatch_log_groups.png)

![Chi tiết log group livecap với các log stream](/images/5-Workshop/5.5-Policy/cloudwatch_livecap_log_groups.png)

![Chi tiết log stream CloudWatch – sự kiện phiên backend](/images/5-Workshop/5.5-Policy/cloudwatch_log_group_detail.png)

![Log event mẫu từ một phiên phiên âm trực tiếp](/images/5-Workshop/5.5-Policy/cloudwatch_log_events.png)

### Metric quan trọng cần theo dõi

| Metric | Nguồn | Ý nghĩa |
|---|---|---|
| `HTTPCode_Target_5XX_Count` | ALB | Lỗi backend trả về CloudFront |
| `HealthyHostCount` | ALB Target Group | Số Fargate task healthy |
| `CPUUtilization` | ECS | CPU của task (cảnh báo nếu > 80%) |
| `MemoryUtilization` | ECS | Bộ nhớ của task |
| `BlockedRequests` | WAF | WAF đang chủ động chặn threat |
| `Tỷ lệ lỗi 4xx/5xx` | CloudFront | Tỷ lệ lỗi end-to-end |

### Tạo CloudWatch Alarm (ví dụ)

Tạo alarm kích hoạt khi ALB trả về lỗi 5XX:

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

## Bảo mật

### Đã triển khai

| Biện pháp | Cách triển khai |
|---|---|
| Không dùng root account | Deploy bằng IAM user `camgiacntn` |
| IAM least privilege | Tách biệt task execution role và task role |
| Không hardcode credential | Chỉ dùng IAM role; không có key trong `.env`, image hay Git |
| S3 frontend private | Block public access + OAC origin |
| S3 transcript private | Block public access; presigned URL hết hạn sau 24 giờ |
| HTTPS mọi nơi | CloudFront terminate viewer TLS |
| WAF ở CloudFront và ALB | Managed rule ở chế độ BLOCK; rate-based rule |
| Giới hạn CORS | `ALLOWED_ORIGIN` giới hạn frontend origin được chấp nhận |
| Giới hạn session | 4 global + 1/IP ngăn chi phí Transcribe ngoài kiểm soát |
| Hết hạn transcript | S3 lifecycle rule 14 ngày; không lưu raw audio |
| Quét secret | Gitleaks chạy trong CI trên toàn bộ lịch sử Git |
| Image tag bất biến | Git SHA tag ngăn drift do tag `latest` không cố định |

### Xác minh WAF

Cả hai Web ACL đều ở chế độ BLOCK. Probe production đã xác nhận:

- Tấn công XSS (Cross-site scripting) → HTTP 403
- Chuỗi khai thác Log4J → HTTP 403

![Xác minh bảo mật runtime WAF – XSS và Log4J bị chặn](/images/5-Workshop/livecap-runtime-security-verification.png)

## Tối ưu chi phí

### Chi phí chính hiện tại (ap-southeast-1)

| Tài nguyên | Cơ sở tính phí | Tối ưu |
|---|---|---|
| ECS Fargate | Theo vCPU-giây + GB-giây | Scale về 0 khi không dùng (tính năng target) |
| ALB | Cố định theo giờ + LCU | Phát sinh phí kể cả khi ECS = 0; chỉ xóa khi destroy toàn bộ stack |
| NAT Gateway | Theo giờ + theo GB data | Một NAT ở một AZ (trade-off chi phí) |
| Amazon Transcribe | Theo phút audio | Giới hạn session giới hạn mức sử dụng |
| Amazon Translate | Theo triệu ký tự | Chỉ dịch finalized segment |
| CloudWatch | Log ingestion + lưu trữ | Retention 14 ngày giới hạn chi phí |
| WAF | Theo ACL + theo rule | Chi phí cố định; xứng đáng với khả năng blocking |

### Biện pháp tiết kiệm chi phí đã áp dụng

- **Retention log và transcript 14 ngày** – tránh lưu trữ vô hạn.
- **Giới hạn thời lượng session (30 phút)** – giới hạn tối đa phút Transcribe mỗi phiên.
- **Giới hạn đồng thời session** – ngăn chi phí Transcribe/Translate do lạm dụng.
- **Chỉ dịch text finalized** – kết quả partial/interim bị loại trước khi dịch, tiết kiệm ký tự.
- **ECS scale-to-zero (target)** – Wake Lambda đưa desired count từ 0 → 1 khi có request đầu tiên; scale-down tự động trở về 0 sau thời gian không hoạt động.

### AWS Budget (Terraform target)

Một AWS Budget alert được định nghĩa trong Terraform target ở mức **$50/tháng**.
Nó gửi thông báo trực tiếp đến **email subscriber** khi chi tiêu thực tế hoặc
dự báo gần đến ngưỡng (không qua SNS).

> **Lưu ý:** Budget alert là tín hiệu billing bị trễ, không phải enforcement
> thời gian thực. Nó giúp bạn phát hiện chi phí ngoài kiểm soát trong vài giờ,
> không phải vài giây.