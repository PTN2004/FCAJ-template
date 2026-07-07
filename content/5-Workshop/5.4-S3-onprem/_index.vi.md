---
title: "Phân phối frontend và tích hợp thời gian thực"
date: 2026-07-05
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Phân phối frontend và tích hợp thời gian thực

Frontend LiveCap dùng React 18, TypeScript, Vite, Tailwind CSS và GSAP. Route
`/` là landing page có scroll animation; `/app` là dashboard nhẹ dùng trong
session caption trực tiếp.

CloudFront là entry point duy nhất của browser. CloudFront phục vụ static asset
từ S3 private qua OAC và route `/api/*`, `/ws/*` đến ALB. Backend tiếp tục điều
phối Transcribe, Translate, transcript export và session cleanup.

Animation nặng chỉ nằm ở landing page. Dashboard live ưu tiên transcript ổn
định, control microphone, trạng thái kết nối và session timer.
