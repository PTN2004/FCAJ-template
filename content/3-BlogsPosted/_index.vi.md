---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Trong suốt quá trình tham gia chương trình FCAJ, em đã hoàn thành và đăng tải ba bài viết chia sẻ kiến thức kỹ thuật trên cộng đồng AWS Study Group:

### [Blog 1 - Tự động hóa quá trình huấn luyện hệ thống gợi ý với Amazon Personalize và AWS Glue](3.1-Blog1/)
Bài viết này giới thiệu giải pháp serverless giúp tự động hóa luồng chuẩn bị dữ liệu và huấn luyện mô hình gợi ý. Trong đó hướng dẫn chi tiết cách dùng AWS Glue để làm sạch, trích xuất dữ liệu từ các nguồn khác nhau về S3, sau đó đưa vào Amazon Personalize để tự động tạo API gợi ý sản phẩm.

### [Blog 2 - Phân tích tính năng Attack Flow Logs trên AWS Shield Advanced](3.2-Blog2/)
Bài viết này giới thiệu và phân tích tính năng Attack Flow Logs mới của dịch vụ Shield Advanced. Nội dung hướng dẫn cách ghi lại metadata chi tiết của các luồng traffic tấn công DDoS (IP, port, quốc gia, hành động ngăn chặn) và cấu hình "đường ống" truyền log tự động về S3/CloudWatch để điều tra sự cố.

### [Blog 3 - Tối ưu hóa suy luận LLM trên Amazon SageMaker AI với BentoML LLM Optimizer](3.3-Blog3/)
Bài viết hướng dẫn quy trình tự động hóa cấu hình và đánh giá hiệu năng để triển khai các mô hình ngôn ngữ lớn (LLM). Bằng cách kết hợp BentoML LLM Optimizer và Amazon SageMaker AI, giải pháp giúp tìm ra dòng máy chủ GPU và cấu hình lượng tử hóa (quantization) tối ưu nhất về cả hiệu năng lẫn chi phí.