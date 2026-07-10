---
title: "Kiểm thử End-to-End"
date: 2026-07-08
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

# Hướng dẫn trải nghiệm End-to-End

Bước này hướng dẫn bạn (và những người tham gia workshop) cách trải nghiệm toàn bộ luồng hệ thống LiveCap từ góc độ người dùng cuối, kết hợp với việc kiểm tra hệ thống để xác nhận kiến trúc hoạt động.

## 1. Truy cập ứng dụng

1. Mở trình duyệt và truy cập vào URL CloudFront của bạn: `https://dpeohr327wt9l.cloudfront.net`
2. Bạn sẽ thấy trang Landing Page của LiveCap.
3. Bấm vào nút **Start a live session** (hoặc Open workspace) để vào giao diện chính (`/app`).

## 2. Bắt đầu phiên dịch thuật

1. Tại giao diện chính, bấm nút **Start session**.
2. Khi trình duyệt yêu cầu quyền truy cập Micro, hãy chọn **Cho phép (Allow)**.
3. Trạng thái kết nối (góc trên) sẽ chuyển từ **READY** → **WAKING** (đánh thức backend) → **LIVE** (đã kết nối).

## 3. Trải nghiệm dịch thuật theo thời gian thực

Hãy thử nghiệm hệ thống bằng cách nói một vài câu rõ ràng vào microphone (bạn có thể nói tiếng Anh hoặc tiếng Việt).

> Ví dụ: "Chào mọi người, chào mừng đến với buổi workshop hôm nay. Live captions are working perfectly!"

Trong vòng 2–5 giây, luồng âm thanh PCM 16kHz của bạn đã đi qua CloudFront → ALB → Fargate → Amazon Transcribe → Amazon Translate và trả về kết quả lên màn hình.
Bạn sẽ thấy hai cột văn bản (Tiếng Việt và Tiếng Anh) xuất hiện song song, đồng bộ theo thời gian thực:

![Giao diện LiveCap đang dịch thuật song ngữ theo thời gian thực](/images/5-Workshop/livecap-dashboard.png)

## 4. Kết thúc và tải Transcript

1. Bấm **Stop session** để kết thúc phiên. Kết nối WebSocket sẽ được đóng sạch sẽ.
2. Bấm nút **Download transcript** (hoặc Export).
3. Hệ thống sẽ gọi API tải file TXT từ Amazon S3 (thông qua presigned URL) về máy tính của bạn. Mở file TXT để kiểm tra nội dung.

---

## 5. Xác minh kỹ thuật (Dành cho Admin)

Sau khi trải nghiệm giao diện, bạn có thể kiểm tra các log backend để hiểu rõ luồng đi của dữ liệu.

### Kiểm tra CloudWatch Logs

Kiểm tra backend đã phát ra các log cấu trúc (structured log) trong phiên vừa rồi:

```powershell
aws logs tail livecap `
  --follow `
  --since 10m `
  --region ap-southeast-1 `
  --profile livecap-camgiacntn
```

Bạn sẽ thấy các sự kiện vòng đời phiên: `session_opened`, `audio_chunk_received`, `transcribe_finalized`, `translate_success`, `session_closed`.

### Kết quả kiểm thử chuẩn production

Sau khi hoàn thành cutover sang kiến trúc target (VPC, private subnet, NAT, WAF, scale-to-zero), toàn bộ luồng production đã pass tất cả bài test sau:

| Bài test | Kết quả |
|---|---|
| Health endpoint | `{"status":"healthy","version":"1.0.0"}` |
| Mở WebSocket | 101 Switching Protocols |
| Phiên âm tiếng Việt PCM 16 kHz thực | Trả về finalized text |
| Phiên âm tiếng Anh PCM 16 kHz thực | Trả về finalized text |
| Dịch tiếng Anh → tiếng Việt | Trả về bản dịch chính xác |
| Heartbeat ping/pong | Duy trì interval 30 giây |
| Kết thúc phiên sạch (nút Stop) | Session đóng, registry cleared |
| Export transcript S3 | TXT object được tạo trong bucket private |
| Tải xuống qua presigned URL | File tải thành công |
| Kiểm thử WAF blocking | XSS và Log4J probe trả về HTTP 403 |
| ECS scale-to-zero (idle 300s) | Service scale về 0 sau 5 phút không dùng |
| ECS self-healing (wake Lambda) | Scale từ 0 → 1 và healthy trong 60s |

![Xác minh end-to-end: phiên âm, dịch và export pass trên production](/images/5-Workshop/livecap-transcribe-translate-export-verification.png)

![Xác minh bảo mật runtime: WAF blocking và giới hạn session](/images/5-Workshop/livecap-runtime-security-verification.png)
