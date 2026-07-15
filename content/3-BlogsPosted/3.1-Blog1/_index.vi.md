---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tự động hóa quá trình huấn luyện hệ thống gợi ý với Amazon Personalize và AWS Glue

Cá nhân hóa trải nghiệm người dùng luôn mang lại sự tăng trưởng doanh thu rõ rệt cho các doanh nghiệp. Tuy nhiên, việc đưa Machine Learning (ML) vào xây dựng một hệ thống gợi ý (Recommendation Engine) thực tế lại là một rào cản lớn. Sự kết hợp giữa AWS Glue và Amazon Personalize mang đến một giải pháp kiến trúc hoàn toàn serverless, giúp tự động hóa quá trình chuẩn bị dữ liệu và huấn luyện mô hình mà không cần xây dựng một Data Lake phức tạp.

Dưới đây là cấu trúc chi tiết về cách tiếp cận này:

## 1. Thách thức trong việc xây dựng hệ thống gợi ý thực tế

Khi bắt tay vào triển khai cá nhân hóa, các tổ chức thường gặp phải những rào cản kỹ thuật sau:

* **Sự phân mảnh dữ liệu:** Trong các hệ thống microservices hiện đại, dữ liệu thường nằm rải rác ở nhiều nguồn khác nhau (Relational DB, NoSQL, Data Warehouse). Việc thu thập và đồng bộ hóa chúng tốn rất nhiều công sức.
* **Hạn chế của hệ thống rule-based:** Nhiều công ty vẫn đang sử dụng các quy tắc thủ công (rule-based) để gợi ý sản phẩm. Các hệ thống này rất kém thông minh, khó bảo trì khi quy mô sản phẩm tăng lên.
* **Thiếu hụt chuyên môn ML:** Việc xây dựng, huấn luyện và tối ưu hóa các thuật toán gợi ý đòi hỏi một đội ngũ Data Scientist chuyên sâu, điều mà không phải doanh nghiệp nào cũng có sẵn.

## 2. AWS Glue và Amazon Personalize giải quyết bài toán như thế nào?

Kiến trúc này phân tách rõ ràng luồng xử lý dữ liệu và luồng huấn luyện AI, tận dụng tối đa các dịch vụ quản lý hoàn toàn (fully-managed):

* **Tự động hóa tích hợp dữ liệu (AWS Glue):** Đóng vai trò là một dịch vụ ETL serverless. AWS Glue Crawlers tự động quét các nguồn dữ liệu đang phân mảnh để nhận diện cấu trúc (schema). Sau đó, các Glue ETL Jobs (chạy trên Apache Spark) sẽ làm sạch, chuẩn hóa và xuất dữ liệu ra định dạng CSV lưu trữ tại Amazon S3.
![alt text](/images/3-BlogsPosted/blog1-2.png)
*Sử dụng AWS Glue để xuất các tập dữ liệu từ những nguồn dữ liệu không đồng nhất sang Amazon S3.*


* **Huấn luyện ML tự động (Amazon Personalize):** Sau khi dữ liệu hội tụ về S3, Personalize tiếp quản toàn bộ phần việc phức tạp của Machine Learning. Nó tự động chọn thuật toán phù hợp nhất dựa trên ba tập dữ liệu: Tương tác (Interactions), Người dùng (Users), và Sản phẩm (Items).
![alt text](/images/3-BlogsPosted/blog1-1.png)
*Amazon Personalize: từ các tập dữ liệu đến API gợi ý*


* **Kiến trúc Serverless End-to-End:** Cả hai dịch vụ đều tự động mở rộng theo nhu cầu thực tế, loại bỏ hoàn toàn gánh nặng quản trị máy chủ cho đội ngũ kỹ thuật.
![alt text](/images/3-BlogsPosted/blog1-3.png)
*Kiến trúc toàn diện kết hợp quy trình xuất dữ liệu sử dụng AWS Glue, quy trình huấn luyện MLOps và Amazon Personalize.*

## 3. Những điểm chính cần nắm

* **Giải quyết mượt mà bài toán "Cold Start":** Bằng cách kết hợp dữ liệu Tương tác bắt buộc với các Siêu dữ liệu (Metadata) của Người dùng và Sản phẩm, hệ thống có thể đưa ra gợi ý chính xác ngay cả với những khách hàng mới hoặc sản phẩm vừa ra mắt.
* **Chống gãy luồng dữ liệu:** Tính năng cập nhật schema tự động của AWS Glue Data Catalog giúp Data Pipeline không bị gián đoạn khi các team microservices thay đổi cấu trúc database độc lập.
* **Cung cấp Inference API trực tiếp:** Amazon Personalize không chỉ huấn luyện mô hình mà còn tự động đóng gói và cung cấp một API suy luận (Inference API) để ứng dụng có thể gọi và lấy kết quả gợi ý theo thời gian thực.

## 4. Giá trị mang lại cho doanh nghiệp

Kiến trúc này giúp các tổ chức có thể nhanh chóng triển khai một Proof of Concept (POC) bằng chính dữ liệu lịch sử hiện có của mình. Nó rút ngắn time-to-market, tối ưu hóa chi phí vận hành và giúp doanh nghiệp sở hữu một Recommendation Engine thông minh mà không cần phải đầu tư xây dựng Data Lake khổng lồ hay đội ngũ chuyên gia ML hùng hậu ngay từ đầu.

## 5. Các bước triển khai và sử dụng cơ bản

Để áp dụng giải pháp này vào dự án thực tế, quy trình chung bao gồm các bước sau:

* **Bước 1 - Trích xuất cấu trúc dữ liệu:** Sử dụng AWS Glue Crawlers để quét các nguồn dữ liệu hiện tại (từ S3, RDS, DynamoDB...) và tự động tạo các bảng siêu dữ liệu trong AWS Glue Data Catalog.
* **Bước 2 - Chuyển đổi và xuất dữ liệu:** Cấu hình AWS Glue ETL Jobs (Python hoặc Scala) để map các cột dữ liệu theo đúng chuẩn định dạng mà Personalize yêu cầu. Sau đó, xuất các file CSV này vào một Amazon S3 bucket.
* **Bước 3 - Khởi tạo Dataset Group:** Trong Amazon Personalize, tạo một Dataset Group mới và định nghĩa các Schema tương ứng cho tập dữ liệu Interactions, Users, và Items.
* **Bước 4 - Huấn luyện mô hình (Training):** Import dữ liệu từ S3 vào Personalize và tiến hành tạo Solution. Dịch vụ sẽ tự động huấn luyện và tinh chỉnh (fine-tune) mô hình Machine Learning.
* **Bước 5 - Triển khai Endpoint:** Tạo một Campaign từ Solution vừa huấn luyện. Personalize sẽ cung cấp một API Endpoint để ứng dụng của bạn có thể tích hợp và truy vấn các gợi ý sản phẩm ngay lập tức.

Hình dưới đây là tổng quan về quy trình tự động hóa luồng dữ liệu và huấn luyện được thực hiện trong bài viết.

Link bài trên group AWS Study Group: [TỰ ĐỘNG HÓA HỆ THỐNG GỢI Ý VỚI AMAZON PERSONALIZE VÀ AWS GLUE](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201959723902321/)