---
title: "Đề xuất dự án"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Đề xuất dự án

## Tên dự án

**LiveCap: Phụ đề cuộc họp Việt-Anh thời gian thực trên AWS**

## 1. Vấn đề

Cuộc họp, workshop và lớp học song ngữ khó theo dõi khi người nói nói nhanh,
chuyển giữa tiếng Việt và tiếng Anh hoặc dùng ngôn ngữ mà một số người tham gia
chưa quen. Ghi chép thủ công chậm, làm mất tập trung và không cung cấp bản dịch
ngay lập tức.

Một số nền tảng họp có caption nhưng có thể yêu cầu gói trả phí, phụ thuộc nền
tảng hoặc lưu toàn bộ bản ghi. Dự án cần một giải pháp chạy trên trình duyệt,
hoạt động độc lập, hiển thị caption trực tiếp và giảm dữ liệu hội thoại được lưu.

## 2. Giải pháp đề xuất

LiveCap là ứng dụng web thực hiện các bước:

1. thu âm microphone trực tiếp trong browser;
2. chuyển audio thành PCM mono 16 kHz, 16-bit;
3. stream chunk qua WebSocket đến backend FastAPI;
4. dùng Amazon Transcribe Streaming nhận dạng Việt/Anh;
5. gửi finalized text đến Amazon Translate;
6. hiển thị original và translated caption theo hai cột; và
7. export caption finalized thành TXT qua S3 download URL có thời hạn.

Hệ thống không lưu raw audio. Chỉ transcript text đã finalized mới được export.

## 3. Lợi ích

- **Hiểu nội dung ngay:** người tham gia đọc original và translated caption khi
  người nói vẫn đang trình bày.
- **Không phụ thuộc nền tảng họp:** ứng dụng browser không phụ thuộc Zoom, Teams
  hoặc nhà cung cấp khác.
- **Dễ xem lại:** caption finalized có thể export thành TXT đơn giản.
- **Giảm dữ liệu nhạy cảm:** microphone audio chỉ được stream, không lưu lại.
- **Vận hành minh bạch:** dashboard hiển thị connection, timer, error và trạng thái export.

## 4. Phạm vi dự án

### Bao gồm

- Caption song ngữ tiếng Anh và tiếng Việt.
- Thu microphone trên browser và stream WebSocket bảo mật.
- Finalized row có timestamp và speaker label.
- Heartbeat, reconnect có giới hạn, timeout và abuse limit.
- Export TXT vào S3 private với retention 14 ngày.
- Infrastructure code, CI, monitoring và tài liệu chi phí.

### Không bao gồm

- Login, phòng họp hoặc lịch sử transcript.
- Tích hợp trực tiếp vào meeting platform.
- Ghi raw audio hoặc lưu hội thoại dài hạn.
- AI summarization hoặc nhận dạng danh tính speaker.
- Backend active-active nhiều task khi session state còn trong memory.

## 5. Kiến trúc giải pháp

![alt](/images/5-Workshop/livecap-target-architecture.png)

Các tài nguyên regional chạy tại `ap-southeast-1`. CloudFront là public entry
point global. Môi trường live đã xác minh dùng một Fargate task sau ALB multi-AZ;
target đã review sẽ chuyển task vào private subnet và thêm WAF, wake-on-demand,
scale `0 <-> 1`.

## 6. Dịch vụ AWS

| Dịch vụ | Vai trò | Lý do chọn |
| --- | --- | --- |
| CloudFront | Entry point HTTPS/WSS và route theo path | Phân phối global và cung cấp endpoint ổn định |
| Amazon S3 | Origin frontend private và lưu transcript | Object storage bền vững, mã hóa |
| Application Load Balancer | Health check và forward WebSocket/API | Route managed đến Fargate target healthy |
| Amazon ECS Fargate | Chạy FastAPI container | Managed container, không quản trị server |
| Amazon ECR | Lưu backend image immutable | Deployment có thể truy vết theo SHA |
| Amazon Transcribe | Streaming speech-to-text | Transcription realtime managed |
| Amazon Translate | Dịch Việt-Anh | Dịch finalized text bằng service managed |
| CloudWatch | Log và metric vận hành | Monitoring và troubleshooting |
| IAM | Quyền runtime/deployment | Least privilege, không nhúng access key |

## 7. Kế hoạch thực hiện

| Giai đoạn | Kết quả |
| --- | --- |
| Yêu cầu và prototype | User flow, WebSocket contract, ứng dụng React/FastAPI ban đầu |
| Xử lý realtime | PCM worklet, Transcribe stream, Translate, finalized caption |
| Deploy AWS | Frontend S3/CloudFront, ECR image, ALB và ECS Fargate backend |
| Hardening | Session guard, reconnect, heartbeat, retention, secret hygiene |
| Infrastructure | Terraform validation, remote-state bootstrap, target VPC/WAF/cost |
| Submission | CI gate, production smoke test, screenshot, tài liệu song ngữ |

## 8. Rủi ro và giảm thiểu

| Rủi ro | Cách giảm thiểu |
| --- | --- |
| Accuracy thay đổi theo noise/accent | Demo ngắn, nói rõ và chỉ lưu finalized result |
| WebSocket disconnect làm gián đoạn | Ping/pong và ba retry backoff 1/2/4 giây |
| AI managed service phát sinh phí | Timeout 30 phút, limit global/per-IP, không xử lý khi idle |
| Lưu dữ liệu hội thoại nhạy cảm | Không lưu raw audio; transcript hết hạn sau 14 ngày |
| Một task bị lỗi | ALB health check, ECS thay task và ghi rõ session bị gián đoạn |
| Infrastructure drift ảnh hưởng demo | Reconcile state, review plan và blue/green cutover |

## 9. Tiêu chí thành công

- Public CloudFront site và dashboard `/app` tải thành công.
- Health endpoint báo backend healthy.
- Microphone session tạo finalized bilingual caption row.
- Stop/reconnect không leak active session hoặc worker.
- Export tạo TXT download mà không lưu raw audio.
- CI backend, frontend, Terraform và secret scan pass.
- Kiến trúc live và target đã review được mô tả tách biệt.