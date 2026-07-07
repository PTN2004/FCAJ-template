---
title: "Bảo mật, quan sát, kiểm thử và chi phí"
date: 2026-07-05
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Bảo mật, quan sát, kiểm thử và chi phí

## Control bảo mật đã triển khai

- IAM role thay credential AWS tĩnh trong container và source code.
- Frontend và transcript S3 bucket đều private; frontend truy cập qua OAC.
- Transcript download dùng presigned URL có thời hạn.
- Không lưu raw audio; transcript object hết hạn sau 14 ngày.
- CORS giới hạn frontend origin được chấp nhận.
- Giới hạn session global/per-IP và timeout 30 phút giảm abuse.
- Gitleaks chạy trong CI; tfstate, tfvars, plan và `.env` thật không được track.
- ECR image dùng tag immutable từ Git SHA và kết quả scan được review.

## Trạng thái WAF và network

Terraform target định nghĩa hai Web ACL riêng: CloudFront (`CLOUDFRONT`) và ALB
(`REGIONAL`), gồm managed rule và rate rule ở COUNT mode. COUNT chỉ quan sát
traffic match, chưa block. Các WAF association này chưa deploy nên không được
mô tả live demo là đã có WAF bảo vệ.

Task hiện tại có public IP trong VPC hiện hữu. Target đã review tạo hai public
và hai private subnet trên hai AZ, đặt ALB ở public subnet, Fargate ở private
subnet và dùng một NAT Gateway. Một NAT là tradeoff tiết kiệm chi phí, đồng thời
là outbound dependency single-AZ chứ không phải network HA đầy đủ.

## Quan sát hệ thống

- FastAPI phát structured session/integration log đến CloudWatch khi handler
  khởi tạo được, nếu không sẽ fallback stdout.
- Backend log retention là 14 ngày.
- ALB, ECS và Lambda cung cấp metric CloudWatch chuẩn.
- Terraform target định nghĩa dashboard cho ECS CPU/memory, ALB traffic/health,
  wake Lambda và WAF mà không bật Container Insights tốn thêm phí.

Dashboard và WAF metric chỉ có ý nghĩa sau khi target resource tương ứng được
apply và associate.

## CI và verification

GitHub Actions kiểm tra mọi pull request và push vào main:

1. Gitleaks scan secret với toàn bộ Git history.
2. Python 3.11 compile backend và pytest.
3. Node 20 cài frontend, chạy test và production build.
4. Terraform 1.10.5 format/validate với `-backend=false`.

CI chủ ý không deploy production, Terraform apply, destroy hoặc migrate state.

## Kiểm soát chi phí

- Transcribe và Translate tính theo usage, chủ yếu chạy khi có session active.
- Giới hạn duration/concurrency kiểm soát AI usage ngoài ý muốn.
- Transcript và log cùng retention 14 ngày.
- Terraform target có AWS Budget `$50/month` cấu hình được; alert billing có độ
  trễ, không phải enforcement realtime.
- Wake/idle target cho phép ECS `0 <-> 1`, nhưng live service hiện giữ một task
  để demo ổn định.

ALB, NAT Gateway và WAF vẫn có fixed/baseline cost khi tồn tại. Scale ECS về 0
không xóa các khoản phí này.

## Rủi ro còn lại

- CloudFront hiện kết nối ALB origin qua HTTP.
- Registry in-memory chưa cho phép scale an toàn nhiều task.
- Một active task khiến thay task làm gián đoạn WebSocket đang chạy.
- Finding package nền trong ECR vẫn được theo dõi đến khi có bản vá tương thích.
