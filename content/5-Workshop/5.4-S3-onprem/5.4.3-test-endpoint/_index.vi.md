---
title: "Dịch và export transcript finalized"
date: 2026-07-05
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

# Dịch và export transcript finalized

## Luồng dịch

Amazon Translate chỉ được gọi cho finalized text đã chọn từ các Transcribe
stream. Backend trả caption message chuẩn hóa gồm session context, speaker
label, timestamp, original text, translated text và final marker. Dashboard chỉ
append message finalized vào hai cột caption.

## Luồng export TXT

1. Người dùng chọn **Export TXT** sau khi caption đã finalized.
2. Frontend gửi các finalized row của session đến export API.
3. FastAPI validate và serialize transcript song ngữ.
4. Backend ghi object TXT vào transcript S3 bucket private.
5. Backend trả presigned download URL có thời hạn.

Bucket được mã hóa, chặn public access và tự xóa transcript sau 14 ngày.
Presigned link hết hạn độc lập, mặc định sau 24 giờ.

## Ranh giới dữ liệu

LiveCap không upload hoặc lưu raw microphone audio trong MVP. Audio chỉ tồn tại
trên luồng browser/WebSocket/Transcribe. S3 chỉ chứa text export đã finalized,
giúp giảm chi phí storage và giới hạn dữ liệu nhạy cảm được giữ lại.

## Xử lý lỗi

Lỗi Translate hoặc export được trả về UI dưới dạng có cấu trúc và ghi log mà
không lộ credential. Session cleanup vẫn chạy khi Transcribe stream hoặc worker
lỗi để active-session count không bị leak.
