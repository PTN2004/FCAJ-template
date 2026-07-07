---
title: "Xác minh workflow end-to-end"
date: 2026-07-05
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

# Xác minh workflow end-to-end

## Luồng trình bày cho reviewer

1. Mở landing page CloudFront và `/api/health`.
2. Vào `/app`, chọn **Start** và cấp quyền microphone.
3. Xác nhận UI chuyển sang Recording và WebSocket session bắt đầu.
4. Nói một câu ngắn bằng tiếng Anh hoặc tiếng Việt.
5. Xác nhận một finalized bilingual row xuất hiện ở hai cột.
6. Kiểm tra heartbeat giữ socket và **Stop** kết thúc sạch.
7. Chọn **Export TXT** và mở temporary download link.

## Gate tự động

| Khu vực | Gate |
| --- | --- |
| Backend | `python -m compileall app` và 204 test |
| Frontend | 11 test và production build |
| Terraform | format, `init -backend=false` và validate |
| Secret | Gitleaks scan toàn history |
| Container | health check và smoke test local |
| Dependency | production npm audit và review ECR scan |

GitHub Actions chạy Backend, Frontend, Terraform và Secret scan trên pull
request và push vào main. CI không deploy, apply Terraform hoặc migrate state.

![Các verification job đã pass trên GitHub Actions](/images/3-Project/github-actions-ci.png)

## Bằng chứng production đã xác minh

Ngày 2026-07-04, luồng production đã pass CloudFront `/`, `/app`, health,
WebSocket start, ping/pong, transcription PCM 16 kHz thật, dịch Anh-Việt, stop
sạch, S3 export và tải TXT qua presigned URL. Browser UI test với microphone WAV
kiểm soát được đã append ba finalized bilingual row và dừng sạch.

## Trường hợp lỗi mong đợi

- Từ chối microphone tạo lỗi rõ ràng trên UI.
- Vượt session limit trả `TOO_MANY_SESSIONS` trước khi mở AWS stream.
- Disconnect bất ngờ chỉ retry ba lần rồi dừng capture.
- Timeout tự kết thúc session ở 30 phút.
- ALB không gửi traffic đến ECS target unhealthy.
