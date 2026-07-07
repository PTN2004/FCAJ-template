---
title: "Stream audio và tạo live caption"
date: 2026-07-05
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Stream audio và tạo live caption

## Pipeline audio và WebSocket

```text
Microphone
  -> Web Audio worklet
  -> PCM mono 16 kHz, 16-bit
  -> CloudFront WSS /ws/transcribe
  -> ALB
  -> FastAPI trên Fargate
  -> Amazon Transcribe Streaming
```

Microphone chỉ bắt đầu khi backend ready và WebSocket đã mở. Chunk sinh ra lúc
socket unavailable sẽ bị drop, tránh buffer phía client tăng không giới hạn.

## Xử lý song ngữ

Khi `BILINGUAL_DUAL_STREAM=true`, backend fan-out cùng PCM input vào Transcribe
stream tiếng Việt và tiếng Anh, chọn kết quả phù hợp rồi dịch finalized segment
sang ngôn ngữ còn lại. Partial result có thể là state tạm thời, nhưng chỉ
finalized segment trở thành row cố định.

## Khả năng phục hồi kết nối

- Frontend gửi `ping` mỗi 30 giây; backend trả `pong`.
- Disconnect bất ngờ khi recording được retry tối đa ba lần với backoff 1, 2, 4 giây.
- Reconnect tạo backend session mới nhưng giữ các finalized row cũ.
- Nếu retry thất bại, audio capture dừng và người dùng phải restart session.
- Stop, disconnect, timeout, lỗi Transcribe và exception đều cleanup queue,
  worker, stream và registry.

## Guardrail cho session

UI và backend cùng giới hạn session 30 phút. Backend còn reject khi vượt giới
hạn global/per-IP trước khi mở managed AI work. Các giới hạn này kiểm soát việc
dùng Transcribe và Translate ngoài ý muốn trong MVP.

![Kết quả bước stream và trả finalized caption về dashboard](/images/3-Project/livecap-dashboard.png)
