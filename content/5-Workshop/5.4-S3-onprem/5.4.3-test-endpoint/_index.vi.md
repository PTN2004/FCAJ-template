---
title: "Export Transcript qua S3 Presigned URL"
date: 2026-07-08
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Export Transcript qua S3 Presigned URL

## Luồng export hoạt động như thế nào

Sau khi phiên phụ đề kết thúc, bạn có thể tải xuống toàn bộ caption row song
ngữ đã finalized dưới dạng file text thuần. Luồng export này đảm bảo raw audio
hoàn toàn không được lưu trữ:

```
Trình duyệt (finalized row)
  → POST /api/sessions/{session_id}/export
  → FastAPI trên Fargate
  → Serialize row sang định dạng TXT
  → PutObject vào S3 transcript bucket private
  → Tạo presigned URL có thời hạn (mặc định 24 giờ)
  → Trả presigned URL về trình duyệt
  → Trình duyệt tải file TXT trực tiếp từ S3
```

Raw audio không bao giờ được ghi vào S3. Chỉ transcript TXT finalized mới được
lưu. Object transcript được tự động xóa sau **14 ngày** nhờ S3 lifecycle rule.

## Bước 1 – Dừng phiên

Bấm **Stop** trên dashboard. Backend sẽ:

1. Đóng Transcribe stream và kết nối Translate
2. Hủy đăng ký session khỏi registry trong bộ nhớ
3. Trả về tóm tắt cuối phiên cùng session ID

## Bước 2 – Export transcript

Bấm **Export TXT** trên dashboard. Frontend gửi:

```http
POST https://dpeohr327wt9l.cloudfront.net/api/sessions/{session_id}/export
Content-Type: application/json

{
  "segments": [
    {
      "segment_id": "seg-001",
      "speaker_label": null,
      "text_vi": "Chào buổi sáng",
      "text_en": "Good morning",
      "spoken_language": "vi",
      "timestamp_start": 0.0,
      "timestamp_end": 2.1
    }
  ]
}
```

Backend phản hồi với presigned URL và thời điểm hết hạn:

```json
{
  "download_url": "https://livecap-transcripts-dev-720459752315.s3.ap-southeast-1.amazonaws.com/transcripts/session-abc123.txt?X-Amz-...",
  "expires_at": "2026-07-09T12:00:00Z"
}
```

## Bước 3 – Xác minh S3 object đã được tạo

Dùng CLI:

```powershell
aws s3 ls "s3://livecap-transcripts-dev-720459752315/transcripts/" `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

Bạn sẽ thấy một file `.txt` cho mỗi phiên đã export. URL hết hạn sau 24 giờ
(cấu hình qua `DOWNLOAD_LINK_EXPIRATION`).

## Bước 4 – Xác minh Lifecycle Rule

Object transcript được tự động xóa sau 14 ngày. Kiểm tra rule:

```powershell
aws s3api get-bucket-lifecycle-configuration `
  --bucket livecap-transcripts-dev-720459752315 `
  --region ap-southeast-1 --profile livecap-camgiacntn
```

Kết quả mong đợi:

```json
{
  "Rules": [{
    "ID": "delete-old-transcripts",
    "Status": "Enabled",
    "Filter": {"Prefix": "transcripts/"},
    "Expiration": {"Days": 14}
  }]
}
```

## Đảm bảo quyền riêng tư

| Những gì được lưu | Những gì KHÔNG được lưu |
|---|---|
| Transcript TXT song ngữ đã finalized | Raw audio từ microphone |
| Metadata phiên (thời lượng, IP hash) | Kết quả Transcribe partial/interim |
| Application log CloudWatch (lưu 14 ngày) | Danh tính người dùng hoặc PII |
