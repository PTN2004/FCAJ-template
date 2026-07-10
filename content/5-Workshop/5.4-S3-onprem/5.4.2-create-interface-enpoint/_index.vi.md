---
title: "Phụ đề trực tiếp – Transcribe & Translate thực chiến"
date: 2026-07-08
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

# Phụ đề trực tiếp – Transcribe & Translate thực chiến

## Pipeline thời gian thực hoạt động như thế nào

Khi người dùng bấm **Start**, chuỗi sự kiện sau diễn ra tự động:

```
Microphone trình duyệt
  → Web Audio API worklet (resample về 16 kHz)
  → Các chunk PCM nhị phân gửi qua WebSocket (WSS)
  → CloudFront /ws/transcribe
  → Application Load Balancer
  → FastAPI trên ECS Fargate (port 8000)
  → Amazon Transcribe Streaming (hai stream song song: vi-VN và en-US)
  → Amazon Translate (chỉ finalized segment)
  → Caption row trả về qua đường WebSocket ngược lại
  → Dashboard phụ đề trình duyệt
```

Microphone chỉ bắt đầu thu âm **sau khi** health check backend pass và kết nối
WebSocket đã được thiết lập. Audio được tạo ra lúc socket chưa mở sẽ bị drop
thay vì buffer.

## Chế độ dual-stream song ngữ

Với `BILINGUAL_DUAL_STREAM=true`, backend chia mỗi chunk PCM vào hai Amazon
Transcribe stream song song:

1. **`vi-VN`** – nhận dạng giọng tiếng Việt
2. **`en-US`** – nhận dạng giọng tiếng Anh

Một bộ arbitrator chọn ngôn ngữ có finalized segment trước, rồi gửi text đó
đến Amazon Translate để tạo ngôn ngữ còn lại. Kết quả là một caption row song
ngữ:

```
[Tiếng Việt gốc]  |  [Bản dịch tiếng Anh]
[Tiếng Anh gốc]   |  [Bản dịch tiếng Việt]
```

Chỉ các segment **finalized** mới trở thành caption row vĩnh viễn. Kết quả
partial (interim) có thể hiển thị tạm thời trên UI nhưng không bao giờ được lưu.

## Khả năng phục hồi kết nối

| Sự kiện | Hành vi |
|---|---|
| Hoạt động bình thường | Frontend gửi `ping` mỗi 30 giây; backend trả lời `pong` |
| Mất kết nối bất ngờ | Frontend thử lại tối đa 3 lần: 1 s, 2 s, 4 s backoff |
| Kết nối lại thành công | Session backend mới bắt đầu; finalized row được giữ nguyên trên UI |
| Hết lần thử lại | Âm thanh dừng thu; người dùng phải bấm Start lại |
| Timeout 30 phút | Backend đóng session; frontend hiển thị "Phiên đã kết thúc" |

## Giới hạn session

Backend từ chối kết nối mới khi vượt giới hạn:

- **4 session đồng thời** toàn hệ thống (một ECS task, bộ nhớ process)
- **1 session trên mỗi client IP**

Các giới hạn này ngăn chặn chi phí Transcribe/Translate phát sinh ngoài kiểm
soát cho MVP. Trước khi scale lên nhiều task hơn, registry session phải chuyển
sang DynamoDB hoặc Redis (state chung giữa các task).

## Bắt đầu phiên phụ đề trực tiếp

1. Mở `https://dpeohr327wt9l.cloudfront.net`
2. Bấm **Start captioning** để đến `/app`
3. Bấm **Start session** – frontend wake backend nếu cần, rồi poll health
4. Cho phép microphone khi trình duyệt hỏi
5. Nói tiếng Anh hoặc tiếng Việt
6. Xem các dòng caption song ngữ đã finalized xuất hiện trên dashboard

Dashboard ở trạng thái **READY** (sẵn sàng) trước khi bắt đầu phiên:

![Dashboard phụ đề LiveCap trạng thái READY hiển thị nút Start session](/images/5-Workshop/livecap-production-dashboard-ready.png)

Sau khi bấm **Start session**, trạng thái chuyển sang **WAKING** trong lúc frontend
đánh thức ECS backend (scale từ 0 → 1) và đồng bộ stream phiên âm:

![Dashboard phụ đề LiveCap trạng thái WAKING hiển thị nút Connecting](/images/5-Workshop/livecap-dashboard-waking.png)

Khi đã kết nối, layout song ngữ hiện ra với hai cột **VIETNAMESE** và **ENGLISH** song song.
Các segment đã hoàn tất sẽ lần lượt xuất hiện dưới dạng các caption row:

![Dashboard phụ đề LiveCap hiển thị 2 cột caption song ngữ sau khi bắt đầu](/images/5-Workshop/livecap-dashboard.png)

Dashboard cung cấp nút điều khiển **Start session**, **Stop session**, **Download transcript**,
và **PURGE SESSION CACHE**; bộ đếm thời gian phiên 30 phút cùng thời gian đã trôi qua; chọn nguồn âm thanh;
huy hiệu trạng thái kết nối thời gian thực (READY / WAKING / LIVE / LOST); hai cột văn bản
gốc/bản dịch; cùng các trạng thái kết nối lại và báo lỗi trên desktop và mobile.
Chỉ các segment đã hoàn tất (finalized) mới được giữ lại và xuất ra file tải về.

## Trạng thái kết nối lỗi

Nếu kết nối WebSocket bị đứt (ví dụ: backend timeout do không có âm thanh hoặc mạng yếu),
dashboard sẽ hiển thị huy hiệu **LOST** cùng các toast báo lỗi:

- **SYSTEM ERROR** – toast màu đỏ với thông báo lỗi cụ thể (ví dụ: "Your request timed
  out because no new audio was received for 15 seconds.")
- **STREAM DISRUPTED** – toast cảnh báo màu vàng yêu cầu kiểm tra internet và khởi động lại

![Dashboard LiveCap hiển thị trạng thái LOST cùng lỗi System Error và Stream Disrupted](/images/5-Workshop/livecap-dashboard-error-state.png)

Hãy bấm **DISMISS** trên hộp thoại lỗi hệ thống, rồi bấm **Start session** lần nữa để kết nối lại.

## Nếu microphone bị chặn?

Nếu trình duyệt chặn truy cập microphone, LiveCap dừng trước khi mở bất kỳ
stream nào và hiển thị thông báo lỗi **HARDWARE ACCESS ERROR** ở khu vực chính
thay vì để một session lỗi đang mở. Hãy cho phép microphone trong cài đặt site
của trình duyệt và tải lại `/app`.
