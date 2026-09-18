---
title: "Đề xuất"
date: 2026-09-10
weight: 2
chapter: false
pre: "<b> 2. </b>"
---

Trong quá trình học tập và thực hành tại chương trình First Cloud AI Journey (FCAJ), em nhận thấy việc kết hợp kiến thức AWS với một dự án thực tế giúp việc học trở nên trực quan và hiệu quả hơn. Vì vậy, em đề xuất xây dựng Workshop triển khai hệ thống **SkyLink Airline** trên nền tảng AWS.

SkyLink Airline là một hệ thống Web hỗ trợ các chức năng liên quan đến đặt vé máy bay. Trong Workshop, em tập trung vào việc triển khai và bảo vệ phần Frontend của hệ thống trên AWS, đồng thời xây dựng cơ chế giám sát và cảnh báo khi phát hiện các request bất thường.

## 2.1. Mục tiêu đề xuất

Workshop được đề xuất với các mục tiêu chính:

- Áp dụng kiến thức AWS đã học vào một dự án Web thực tế.
- Triển khai Frontend của SkyLink Airline trên Amazon S3.
- Sử dụng Amazon CloudFront để phân phối nội dung và hỗ trợ truy cập thông qua HTTPS.
- Sử dụng Origin Access Control (OAC) để kiểm soát quyền truy cập giữa CloudFront và Amazon S3.
- Sử dụng AWS WAF để bảo vệ ứng dụng trước các request không mong muốn.
- Sử dụng IP Set và Rate-based Rule để kiểm soát các nguồn truy cập bất thường.
- Sử dụng Amazon CloudWatch để thu thập log, tạo metric và thiết lập cảnh báo.
- Sử dụng Amazon SNS để gửi thông báo qua Email khi phát hiện sự kiện cần chú ý.
- Áp dụng IAM theo nguyên tắc cấp quyền tối thiểu cần thiết.
- Giúp người học hiểu được cách các dịch vụ AWS phối hợp với nhau trong một kiến trúc Web thực tế.

## 2.2. Bài toán cần giải quyết

Khi triển khai một ứng dụng Web trên Internet, hệ thống có thể phải xử lý nhiều loại request khác nhau. Một số request có thể đến từ các nguồn không mong muốn hoặc có tần suất truy cập cao bất thường, gây ảnh hưởng đến tài nguyên và khả năng phục vụ của hệ thống.

Nếu chỉ triển khai Frontend trên một dịch vụ lưu trữ mà không có lớp bảo vệ và giám sát, việc phát hiện và xử lý các request bất thường sẽ gặp nhiều khó khăn.

Do đó, Workshop tập trung giải quyết ba vấn đề chính:

1. **Triển khai ứng dụng:** Đưa Frontend SkyLink Airline lên môi trường AWS và cung cấp khả năng truy cập ổn định thông qua CloudFront.
2. **Bảo mật:** Sử dụng AWS WAF để kiểm soát request và hạn chế các nguồn truy cập không mong muốn.
3. **Giám sát và cảnh báo:** Thu thập log, tạo metric và sử dụng CloudWatch Alarm kết hợp với SNS để phát hiện và thông báo các sự kiện bất thường.

## 2.3. Giải pháp đề xuất

Kiến trúc được đề xuất sử dụng các dịch vụ AWS theo mô hình:

```text
                         Internet
                            │
                            ▼
                       CloudFront
                            │
                            ▼
                         AWS WAF
                       ┌────┴────┐
                       │         │
                    Allow       Block
                       │         │
                       ▼         ▼
                    Amazon S3   WAF Logs
                                   │
                                   ▼
                           CloudWatch Logs
                                   │
                                   ▼
                            Metric Filter
                                   │
                                   ▼
                           CloudWatch Alarm
                                   │
                                   ▼
                              SNS Topic
                                   │
                                   ▼
                              Email Alert
```

Trong đó:

* Amazon S3: Lưu trữ các file Frontend sau khi build.
* Amazon CloudFront: Phân phối nội dung từ S3 đến người dùng và hỗ trợ HTTPS.
* AWS WAF: Kiểm tra và kiểm soát các request trước khi chúng được chuyển đến Origin.
* CloudWatch Logs: Lưu trữ log từ AWS WAF để phục vụ việc theo dõi và phân tích.
* CloudWatch Metric Filter: Chuyển các sự kiện phù hợp trong log thành metric.
* CloudWatch Alarm: Theo dõi metric và phát hiện khi số lượng request bị chặn vượt ngưỡng cấu hình.
* Amazon SNS: Gửi thông báo cảnh báo đến người đăng ký.
* Email: Nhận thông báo khi hệ thống phát hiện sự kiện cần xử lý.

## 2.4. Phạm vi thực hiện

Trong phạm vi Workshop, em tập trung vào phần Frontend và các thành phần AWS phục vụ triển khai, bảo mật và giám sát.

Các nội dung chính bao gồm:

* Chuẩn bị và build Frontend SkyLink Airline.
* Tạo và cấu hình Amazon S3.
* Triển khai Frontend lên S3.
* Cấu hình Amazon CloudFront.
* Cấu hình Origin Access Control.
* Tạo và cấu hình AWS WAF Web ACL.
* Cấu hình IP Set và Rate-based Rule.
* Thiết lập WAF Logging.
* Tạo CloudWatch Log Metric Filter.
* Cấu hình CloudWatch Alarm.
* Tạo Amazon SNS Topic và Email Subscription.
* Kiểm thử khả năng truy cập, bảo vệ và cảnh báo của hệ thống.
* Dọn dẹp các tài nguyên AWS sau khi hoàn thành Workshop.

## 2.5. Kết quả mong đợi

Sau khi hoàn thành Workshop, người học có thể:

* Triển khai thành công một Frontend Web lên AWS.
* Hiểu cách CloudFront phân phối nội dung từ S3.
* Hiểu cách sử dụng OAC để bảo vệ Origin S3.
* Biết cách sử dụng AWS WAF để kiểm soát request.
* Biết cách theo dõi hoạt động của WAF thông qua CloudWatch Logs.
* Biết cách tạo Metric Filter và CloudWatch Alarm.
* Có thể cấu hình SNS để nhận cảnh báo qua Email.
* Hiểu cách xây dựng một quy trình bảo mật và giám sát cơ bản cho ứng dụng Web trên AWS.

Thông qua đề xuất này, Workshop SkyLink Airline hướng đến việc kết hợp kiến thức lý thuyết với thực hành trên AWS, đồng thời tạo nền tảng để tiếp tục mở rộng hệ thống theo hướng Cloud, DevOps và Cloud Security.