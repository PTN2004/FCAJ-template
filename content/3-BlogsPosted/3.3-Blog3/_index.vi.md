---
title: "Blog 3"
date: 2024-07-07
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---


# Tối ưu hóa suy luận LLM trên Amazon SageMaker AI với BentoML LLM Optimizer

Việc đưa các Mô hình Ngôn ngữ Lớn (LLM) mã nguồn mở vào môi trường thực tế (production) là một quá trình phức tạp. Sự kết hợp giữa Amazon SageMaker AI và BentoML LLM Optimizer mang đến một giải pháp toàn diện để tự động hóa quá trình cấu hình, giúp giải quyết triệt để bài toán cân bằng giữa hiệu năng và chi phí hạ tầng.

Dưới đây là cấu trúc chi tiết về cách tiếp cận này:

## 1. Thách thức trong việc vận hành LLM thực tế

Khi triển khai LLM, các kỹ sư thường phải đối mặt với những rào cản kỹ thuật lớn:

* **Chi phí phần cứng đắt đỏ:** Các mô hình hàng tỷ tham số đòi hỏi các dòng GPU có dung lượng VRAM lớn, tiêu tốn rất nhiều ngân sách nếu không tối ưu đúng cách.
* **Sự đánh đổi giữa độ trễ và thông lượng:** Tăng thông lượng (throughput) thường dẫn đến tăng độ trễ (latency), làm ảnh hưởng trực tiếp đến trải nghiệm người dùng cuối.
* **Tốn thời gian tinh chỉnh thủ công:** Việc tìm ra sự kết hợp hoàn hảo giữa loại máy chủ (instance type), kỹ thuật phân mảnh (tensor parallelism) và lượng tử hóa (quantization) yêu cầu rất nhiều vòng lặp thử nghiệm (trial-and-error).

## 2. BentoML LLM Optimizer giải quyết bài toán như thế nào?

Công cụ này hoạt động như một hệ thống tự động dò tìm và định chuẩn (benchmark), thay thế hoàn toàn các thao tác phỏng đoán:

* **Tự động Profiling:** Mô phỏng các luồng truy cập thực tế để đo lường hiệu năng của mô hình trên nhiều dòng GPU khác nhau (như G5, P4 trên AWS).
* **Đóng gói chuẩn mực:** BentoML tự động hóa việc container hóa (containerization) mô hình và các thư viện phụ thuộc, đảm bảo tính nhất quán từ môi trường phát triển (dev) lên đám mây (cloud).
* **Tích hợp sâu với SageMaker AI:** Tận dụng hạ tầng quản lý hoàn toàn (fully-managed) của AWS, cho phép tự động mở rộng (auto-scaling) mượt mà và đảm bảo các tiêu chuẩn an ninh mạng cấp doanh nghiệp.

## 3. Những điểm chính cần nắm

* **Tối ưu VRAM hiệu quả:** Hệ thống cung cấp thông số chính xác để áp dụng quantization (ví dụ: FP16, INT8), giúp các LLM lớn chạy ổn định ngay cả trên GPU có bộ nhớ giới hạn.
* **Gợi ý cấu hình tối ưu nhất:** Chỉ ra đích danh loại AWS Instance mang lại thông lượng cao nhất với mức chi phí trên mỗi token (cost per token) thấp nhất.
* **Giảm thiểu rủi ro:** Ngăn chặn tình trạng cấp phát thừa (over-provisioning) gây lãng phí, hoặc thiếu tài nguyên khiến hệ thống bị nghẽn (bottleneck) khi lưu lượng truy cập tăng đột biến.

## 4. Giá trị mang lại cho doanh nghiệp

Việc áp dụng kiến trúc này giúp giải phóng hoàn toàn đội ngũ MLOps và Data Engineer khỏi các công việc viết kịch bản test thủ công. Nó giúp rút ngắn thời gian đưa các tính năng Generative AI ra thị trường (time-to-market) từ vài tuần xuống chỉ còn vài ngày, đồng thời minh bạch hóa và kiểm soát chặt chẽ chi phí cloud hàng tháng.

## 5. Các bước triển khai và sử dụng cơ bản
Để áp dụng giải pháp này vào dự án thực tế, quy trình chung thường bao gồm các bước sau:

Bước 1 - Chuẩn bị môi trường: Cài đặt thư viện bentoml, các công cụ hỗ trợ cho AWS (như bentoML-sagemaker) và thiết lập quyền truy cập (IAM Role) có quyền tương tác với SageMaker và Amazon ECR.

Bước 2 - Định nghĩa mô hình (Bento Build): Tạo các file cấu hình (như service.py và bentofile.yaml) để khai báo mô hình LLM bạn muốn sử dụng. Sau đó, chạy lệnh build của BentoML để đóng gói mô hình thành một định dạng chuẩn.

Bước 3 - Khởi chạy LLM Optimizer: Kích hoạt công cụ định chuẩn (benchmark) của BentoML. Hệ thống sẽ tự động khởi tạo các môi trường thử nghiệm trên AWS, chạy các bài kiểm tra tải (load test) với nhiều tham số phần cứng và phần mềm khác nhau.

Bước 4 - Đánh giá và cấu hình: Optimizer sẽ trả về một báo cáo trực quan so sánh thông lượng và độ trễ. Dựa vào bảng phân tích này, bạn có thể chọn ra cấu hình có chỉ số p99 latency và chi phí tối ưu nhất cho use-case của mình.

Bước 5 - Triển khai lên SageMaker: Sử dụng cấu hình đã chọn để đẩy container lên Amazon ECR và tạo SageMaker Endpoint trực tiếp thông qua các lệnh CLI của BentoML, hoàn tất quá trình đưa mô hình lên production.

Hình dưới đây là tổng quan về quy trình làm việc được thực hiện trong bài viết.
![The following figure is an overview of the workflow conducted throughout the post.](/images/blogpost/blog3_fg1.png)


Link bài trên group AWS Study Group: [TỐI ƯU HÓA HIỆU SUẤT VÀ CHI PHÍ TRIỂN KHAI LLM TRÊN AMAZON SAGEMAKER AI VỚI BENTOML LLM OPTIMIZER](https://www.facebook.com/groups/awsstudygroupfcj/posts/2206721756759451/?notif_id=1783392652127913&notif_t=group_post_approved&ref=notif)


