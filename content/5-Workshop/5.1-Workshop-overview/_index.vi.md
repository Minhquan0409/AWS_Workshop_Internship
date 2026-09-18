---

title: "Tổng quan Workshop"
date: 2026-08-24
weight: 1
chapter: false
pre: "<b>5.1. </b>"
-------------------

## Mục tiêu

Workshop này nhằm triển khai hệ thống **SkyLink Airline** lên nền tảng **Amazon Web Services (AWS)**, đồng thời xây dựng một kiến trúc Cloud có khả năng mở rộng, bảo mật và giám sát.

Thông qua Workshop, hệ thống từ môi trường phát triển cục bộ sẽ được chuyển sang kiến trúc sử dụng các dịch vụ AWS như **Amazon S3, Amazon CloudFront, Amazon EC2, Amazon RDS, Application Load Balancer, Amazon SQS, AWS Lambda, AWS WAF, Amazon CloudWatch, Amazon SNS và AWS IAM**.

Sau khi hoàn thành, người thực hiện có thể triển khai hệ thống SkyLink từ đầu, kiểm tra hoạt động của ứng dụng, theo dõi log và metric, kiểm thử cơ chế bảo mật và thực hiện cleanup các tài nguyên AWS.

---

## 1. Giới thiệu bài toán và giải pháp

### 1.1. Bài toán

**SkyLink Airline** là hệ thống quản lý và đặt vé máy bay được xây dựng nhằm hỗ trợ hành khách tìm kiếm chuyến bay và thực hiện các thao tác liên quan đến đặt vé.

Hệ thống gồm hai thành phần chính:

* **Frontend:** xây dựng bằng React và Vite, cung cấp giao diện cho hành khách và các chức năng quản lý.
* **Backend:** xây dựng bằng Laravel, cung cấp REST API và xử lý nghiệp vụ của hệ thống.
* **Database:** sử dụng MySQL để lưu trữ thông tin người dùng, chuyến bay, đặt vé và các dữ liệu nghiệp vụ.
* **Queue:** Backend có cơ chế xử lý tác vụ nền thông qua Laravel Queue.

Khi chạy trong môi trường local, các thành phần của hệ thống phụ thuộc vào máy chủ phát triển và chưa tận dụng được các khả năng của Cloud như phân phối nội dung, khả năng mở rộng, monitoring và security.

### 1.2. Giải pháp

Workshop triển khai SkyLink theo mô hình Cloud:

```text
Frontend
React + Vite
      |
      v
Amazon S3
      |
      v
Amazon CloudFront
      |
     WAF

Backend
Laravel
      |
      v
Application Load Balancer
      |
      v
Amazon EC2
      |
      v
Amazon RDS for MySQL
```

Ngoài kiến trúc ứng dụng chính, Workshop bổ sung các thành phần phục vụ xử lý bất đồng bộ, bảo mật và monitoring:

```text
Amazon SQS
     |
     v
AWS Lambda
     |
     +------> AWS WAF
     |
     +------> Amazon SNS

Amazon CloudWatch
     |
     +------> Logs
     +------> Metrics
     +------> Alarms
```

Giải pháp này giúp SkyLink chuyển từ mô hình chạy cục bộ sang một hệ thống Cloud có khả năng:

* Phân phối frontend thông qua CDN.
* Tách frontend và backend.
* Sử dụng database managed service.
* Xử lý các tác vụ nền theo mô hình event-driven.
* Theo dõi hoạt động của hệ thống.
* Phát hiện và cảnh báo các sự kiện bất thường.
* Áp dụng nguyên tắc bảo mật và Least Privilege.

---

## 2. Kiến trúc hệ thống

Kiến trúc tổng thể của SkyLink trên AWS được thiết kế như sau:

![SkyLink Airline AWS Architecture](/images/5-Workshop/5.1-Workshop-overview/system_architecture.png?v=2)

### 2.1. Sơ đồ kiến trúc

```text
                              INTERNET
                                  |
                 +----------------+----------------+
                 |                                 |
                 v                                 v
        +----------------+                 +----------------+
        |  CloudFront   |                 |      ALB       |
        +-------+--------+                 +-------+--------+
                |                                  |
                v                                  v
        +----------------+                 +----------------+
        |      WAF       |                 |      EC2       |
        +-------+--------+                 | Laravel API    |
                |                          +-------+--------+
                v                                  |
        +----------------+                          |
        |       S3       |                          v
        | React / Vite   |                  +---------------+
        +----------------+                  | RDS MySQL     |
                                            +---------------+
                                                    |
                                                    |
                                             +------+------+
                                             |             |
                                             v             v
                                           SQS           Database
                                             |
                                             v
                                          Lambda
                                             |
                              +--------------+--------------+
                              |                             |
                              v                             v
                           WAF IP Set                     SNS
                                                            |
                                                            v
                                                         Email


                     +--------------------------------+
                     |          CloudWatch             |
                     | Logs / Metrics / Alarms         |
                     +--------------------------------+
```

### 2.2. Luồng Frontend

Người dùng truy cập hệ thống thông qua Internet.

Request được chuyển tới **Amazon CloudFront**. CloudFront kiểm tra request thông qua **AWS WAF** trước khi lấy static assets từ **Amazon S3**.

```text
User
  |
  v
CloudFront
  |
  v
AWS WAF
  |
  v
Amazon S3
  |
  v
React / Vite
```

Cách triển khai này giúp frontend được phân phối thông qua CDN thay vì để người dùng truy cập trực tiếp vào S3.

### 2.3. Luồng Backend

Các request API từ frontend được gửi đến Backend.

```text
React Frontend
      |
      v
Application Load Balancer
      |
      v
Amazon EC2
      |
      v
Laravel REST API
      |
      v
Amazon RDS
```

Application Load Balancer đóng vai trò phân phối request tới EC2.

Laravel Backend chịu trách nhiệm xử lý các nghiệp vụ của SkyLink như:

* Authentication.
* Flight search.
* Booking.
* Ticket management.
* User management.
* Check-in.
* Các nghiệp vụ liên quan khác.

### 2.4. Luồng xử lý bất đồng bộ

Các tác vụ không cần xử lý ngay trong HTTP request có thể được đưa vào **Amazon SQS**.

```text
Laravel
   |
   v
Amazon SQS
   |
   v
AWS Lambda
```

Cách tiếp cận này giúp giảm thời gian xử lý request và tạo nền tảng để hệ thống mở rộng theo mô hình event-driven.

### 2.5. Luồng Monitoring và Security

CloudWatch thu thập log và metric từ các thành phần của hệ thống.

```text
Application / WAF / Lambda
             |
             v
        CloudWatch
             |
             v
           Alarm
             |
             v
            SNS
             |
             v
           Email
```

AWS WAF được sử dụng để kiểm soát các request bất thường.

Trong trường hợp phát hiện security event cần xử lý tự động, Lambda có thể cập nhật WAF IP Set và gửi thông báo thông qua SNS.

---

## 3. Quy trình hoạt động của hệ thống

Quy trình hoạt động của SkyLink trên AWS gồm các bước chính.

### Bước 1 – Người dùng truy cập SkyLink

Người dùng mở website SkyLink thông qua domain hoặc CloudFront URL.

```text
User
 ↓
CloudFront
```

### Bước 2 – CloudFront và WAF xử lý request

CloudFront tiếp nhận request.

AWS WAF kiểm tra request dựa trên các rule được cấu hình.

Nếu request hợp lệ, request tiếp tục tới S3.

```text
CloudFront
     |
     v
AWS WAF
     |
     +---- Block → Request rejected
     |
     +---- Allow → S3
```

### Bước 3 – Frontend được tải từ S3

S3 cung cấp các static files của React/Vite:

```text
HTML
CSS
JavaScript
Images
Other static assets
```

Frontend được tải về trình duyệt của người dùng.

### Bước 4 – Frontend gọi Backend API

Khi người dùng tìm kiếm chuyến bay hoặc thực hiện booking, frontend gửi request tới Backend API.

```text
React
  |
  v
ALB
  |
  v
EC2
  |
  v
Laravel API
```

### Bước 5 – Backend xử lý nghiệp vụ

Laravel nhận request và xử lý nghiệp vụ.

Ví dụ với chức năng tìm kiếm chuyến bay:

```text
User
 ↓
Search Flight
 ↓
React
 ↓
Laravel API
 ↓
RDS MySQL
 ↓
Flight Data
 ↓
Laravel API
 ↓
React
 ↓
User
```

### Bước 6 – Lưu dữ liệu vào RDS

Các dữ liệu nghiệp vụ được lưu trong Amazon RDS for MySQL.

Ví dụ:

* User.
* Flight.
* Booking.
* Ticket.
* Payment information.
* Check-in information.

### Bước 7 – Xử lý tác vụ nền

Nếu có tác vụ cần xử lý bất đồng bộ:

```text
Laravel
   |
   v
SQS
   |
   v
Lambda
```

Việc tách tác vụ nền khỏi request chính giúp giảm thời gian phản hồi và tăng khả năng mở rộng.

### Bước 8 – Monitoring

CloudWatch thu thập thông tin hoạt động của hệ thống:

```text
Logs
Metrics
Errors
CPU Utilization
Lambda Invocations
WAF Events
```

### Bước 9 – Alert

Khi metric vượt ngưỡng được cấu hình:

```text
CloudWatch Alarm
       |
       v
      SNS
       |
       v
     Email
```

Người quản trị có thể nhận được cảnh báo để kiểm tra hệ thống.

---

## 4. Các dịch vụ được sử dụng

| Dịch vụ                       | Vai trò trong SkyLink                       |
| ----------------------------- | ------------------------------------------- |
| **Amazon S3**                 | Lưu trữ frontend React/Vite                 |
| **Amazon CloudFront**         | CDN và phân phối frontend                   |
| **AWS WAF**                   | Bảo vệ frontend/API khỏi request bất thường |
| **Amazon EC2**                | Chạy Laravel Backend                        |
| **Application Load Balancer** | Phân phối request tới Backend               |
| **Amazon RDS**                | Managed MySQL Database                      |
| **Amazon SQS**                | Message Queue cho xử lý bất đồng bộ         |
| **AWS Lambda**                | Xử lý event và automation                   |
| **Amazon CloudWatch**         | Logs, Metrics và Monitoring                 |
| **Amazon SNS**                | Gửi security/monitoring alerts              |
| **AWS IAM**                   | Quản lý authentication và authorization     |
| **Amazon VPC**                | Mạng riêng cho các tài nguyên AWS           |

### Lý do lựa chọn

**Amazon S3:** phù hợp với frontend React/Vite vì sau khi build, frontend trở thành các static files.

**CloudFront:** giúp phân phối frontend thông qua CDN và cung cấp HTTPS.

**EC2:** phù hợp để triển khai Laravel Backend và cho phép kiểm soát môi trường chạy ứng dụng.

**RDS:** cung cấp MySQL managed service, giảm công việc quản trị database.

**ALB:** tạo endpoint ổn định cho Backend và là nền tảng để mở rộng nhiều EC2 instance.

**SQS:** tách các tác vụ nền khỏi request chính và giảm sự phụ thuộc trực tiếp giữa các thành phần.

**Lambda:** xử lý event mà không cần duy trì server riêng cho từng tác vụ.

**WAF:** bổ sung lớp bảo vệ cho các request HTTP/HTTPS.

**CloudWatch:** cung cấp khả năng monitoring và troubleshooting.

**SNS:** gửi notification khi hệ thống phát hiện sự kiện cần chú ý.

**IAM:** áp dụng nguyên tắc Least Privilege và tránh hard-code AWS credentials.

**VPC:** kiểm soát network access giữa các AWS resources.

---

## 5. Kết quả đạt được

Sau khi hoàn thành Workshop, SkyLink được triển khai theo mô hình Cloud thay vì chỉ chạy trên môi trường local.

Các kết quả chính bao gồm:

### 5.1. Application Deployment

Frontend React/Vite được build và triển khai lên Amazon S3.

Backend Laravel được triển khai trên Amazon EC2.

Database được chuyển sang Amazon RDS for MySQL.

### 5.2. Cloud Architecture

Hệ thống có kiến trúc rõ ràng với các thành phần:

```text
CloudFront
    ↓
WAF
    ↓
S3

ALB
    ↓
EC2
    ↓
RDS
```

### 5.3. Security

Workshop áp dụng:

* IAM Role.
* Least Privilege.
* AWS WAF.
* WAF IP Set.
* HTTPS.
* Không hard-code AWS Access Key.
* Hạn chế public access đối với S3.

### 5.4. Monitoring

CloudWatch được sử dụng để theo dõi:

* Application logs.
* WAF logs.
* EC2 metrics.
* Lambda metrics.
* Error events.
* Alarm status.

### 5.5. Alerting

Khi xảy ra sự kiện vượt ngưỡng:

```text
CloudWatch
     ↓
Alarm
     ↓
SNS
     ↓
Email
```

người quản trị có thể nhận cảnh báo.

### 5.6. Khả năng mở rộng

Kiến trúc được thiết kế để có thể mở rộng trong tương lai:

```text
                 ALB
                  |
          +-------+-------+
          |       |       |
         EC2     EC2     EC2
```

Backend có thể mở rộng thành nhiều EC2 instance khi traffic tăng.

SQS cũng giúp giảm coupling giữa các thành phần và hỗ trợ xử lý bất đồng bộ.

### 5.7. Vận hành

Workshop cung cấp quy trình hoàn chỉnh:

```text
Deploy
  ↓
Test
  ↓
Monitor
  ↓
Alert
  ↓
Troubleshoot
  ↓
Cleanup
```

Qua đó, SkyLink không chỉ được triển khai trên AWS mà còn thể hiện được các nguyên tắc cơ bản của Cloud Computing: **deployment, security, monitoring, scalability và cost management**.
