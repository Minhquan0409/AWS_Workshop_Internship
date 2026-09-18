---
title: "Cấu hình AWS WAF & Logging"
date: 2026-08-24
weight: 6
chapter: false
pre: "<b>5.6. </b>"
---

## Mục tiêu

Trong phần này, triển khai **AWS WAF (Web Application Firewall)** để bảo vệ website **SkyLink Airline** đang được phân phối thông qua Amazon CloudFront.

Bên cạnh đó, cấu hình **AWS WAF Logging** để ghi nhận các request đi qua Web ACL, hỗ trợ theo dõi, kiểm tra và phát hiện các request bị chặn.

Sau khi hoàn thành, kiến trúc hệ thống sẽ được mở rộng:

```text
User
  │
  │ HTTPS
  ▼
Amazon CloudFront
  │
  │ AWS WAF
  ▼
Web ACL
  │
  │ Allowed requests
  ▼
Amazon S3
  │
  └── React Frontend
  ```

### Mục tiêu chính
* Tạo Web ACL trên AWS WAF.
* Liên kết Web ACL với CloudFront Distribution của SkyLink Airline.
* Sử dụng AWS Managed Rules để tăng khả năng bảo vệ website.
* Kiểm tra request được AWS WAF cho phép hoặc chặn.
* Cấu hình logging để ghi nhận hoạt động của AWS WAF.
* Kiểm tra log và các request được ghi nhận.
* Hiểu cách CloudFront kết hợp với AWS WAF để bảo vệ website.
* Thực hiện clean-up các tài nguyên không cần thiết để hạn chế phát sinh chi phí.

---

## 1. Tổng quan
### 1.1. Vai trò của AWS WAF

AWS WAF (Web Application Firewall) là dịch vụ Firewall ở tầng ứng dụng của AWS, giúp kiểm soát các HTTP/HTTPS request gửi đến các tài nguyên được hỗ trợ như Amazon CloudFront.

Trong project SkyLink Airline, AWS WAF được đặt phía trước CloudFront để kiểm tra request trước khi request được chuyển đến S3.

Mô hình:
```
                    Internet
                        │
                        ▼
                ┌───────────────┐
                │     User      │
                └───────┬───────┘
                        │ HTTPS
                        ▼
                ┌───────────────┐
                │  CloudFront   │
                │      CDN      │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │    AWS WAF    │
                │    Web ACL    │
                └───────┬───────┘
                        │
                 Allowed Request
                        │
                        ▼
                ┌───────────────┐
                │      S3       │
                │    Private    │
                └───────┬───────┘
                        │
                        ▼
                React Frontend
```
AWS WAF có thể kiểm tra request dựa trên các điều kiện như:
```
IP Address
HTTP Method
URI
Headers
Query String
Request Rate
Managed Rules
```
Trong Workshop này, AWS WAF được sử dụng ở mức cơ bản để minh họa cách bảo vệ website CloudFront.

---

### 1.2. Vai trò của Web ACL

Web ACL (Web Access Control List) là tập hợp các rule được sử dụng để quyết định cách AWS WAF xử lý request.

Một Web ACL có thể chứa nhiều rule.

Ví dụ:
```
Incoming Request
       │
       ▼
    Web ACL
       │
       ├── AWS Managed Rule
       │
       ├── IP Rule
       │
       └── Rate-based Rule
       │
       ▼
   Allow / Block
```
Trong project SkyLink Airline, Web ACL được liên kết với CloudFront Distribution.

---

### 1.3. AWS Managed Rules

AWS cung cấp các AWS Managed Rules nhằm hỗ trợ bảo vệ ứng dụng khỏi một số nhóm request phổ biến.

Trong Workshop này có thể sử dụng: AWSManagedRulesCommonRuleSet

Rule set này cung cấp các rule phổ biến để kiểm tra request HTTP/HTTPS.

Việc sử dụng Managed Rules giúp giảm số lượng rule phải tự xây dựng.

---

## 2. Nội dung thực hành
### 2.1 Tạo WAF IP Set
### 2.1.1. Khởi tạo WAF IP Set

### Bước 1: Truy cập dịch vụ AWS WAF

1. Đăng nhập vào **AWS Management Console**.
2. Trên thanh tìm kiếm, nhập `WAF` và chọn dịch vụ **AWS WAF & Shield**.
3. Tại menu điều hướng bên trái, chọn **IP sets**.

![Truy cập WAF IP Sets](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/waf-ip-sets-menu.png)

---

### Bước 2: Thiết lập thông số IP Set

1. Tại mục **Region**, chọn **Global (CloudFront)**.
2. Nhấn nút **Create IP set**.
3. Nhập các thông tin chi tiết:
   - **IP set name:** `AutoBlockedIPSetV6`
   - **Description:** `IP Set chứa danh sách IP bị tự động chặn bởi Lambda`
   - **Region:** Giữ mặc định `Global (CloudFront)`
   - **IP version:** Chọn **IPv6**.
   - **IP addresses:** Giữ trống (không nhập IP nào) vì danh sách này sẽ được cập nhật tự động bởi AWS Lambda khi phát hiện vi phạm.

![Cấu hình IP Set Name và Region](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/configure-ip-set.png)

---

### Bước 3: Hoàn tất tạo IP Set

1. Kéo xuống cuối trang và nhấn nút **Create IP set**.

![Nhấn Create IP Set](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/finish-create-ip-set.png)

---

### 2.1.2. Kiểm tra danh sách WAF IP Set

Sau khi khởi tạo thành công, danh sách IP sets sẽ hiển thị `AutoBlockedIPSetV6` với các thông số:

- **Region:** Global (CloudFront)
- **IP version:** IPv6
- **Capacity:** 1

---

### 2.1.3. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **WAF IP Set** có tên `AutoBlockedIPSetV6` được khởi tạo thành công ở Scope `Global (CloudFront)`.
- Danh sách IP ban đầu được giữ trống thành công, sẵn sàng để gán vào Web ACL ở phần 2.2 và để hàm Lambda ghi đè/thêm các IP vi phạm ở bài 5.8.

---

### 2.2. Tạo và cấu hình Web ACL

Phần này hướng dẫn chi tiết các bước tạo mới một Web Access Control List (Web ACL) trên AWS WAF, cấu hình các quy tắc bảo vệ (Rate-based rule và IP Set rule) và liên kết trực tiếp với Amazon CloudFront Distribution để bảo vệ hệ thống.

---

### 2.2.1. Khởi tạo Web ACL

### Bước 1: Truy cập tạo Web ACL

1. Đăng nhập vào **AWS Management Console**.
2. Truy cập dịch vụ **AWS WAF & Shield**.
3. Tại menu điều hướng bên trái, chọn **Web ACLs**.
4. Tại mục **Region**, chọn **Global (CloudFront)**.
5. Nhấn nút **Create web ACL**.

![Truy cập Web ACLs](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/web-acl-menu.png)

---

### Bước 2: Cấu hình thông tin chung (Describe web ACL)

1. **Name:** `skylink-airline-waf`
2. **Description:** `AWS WAF protection for the SkyLink Airline CloudFront distribution.`
3. **Resource type:** Giữ mặc định **CloudFront distributions**.
4. **Associated AWS resources:**
   - Nhấn nút **Add AWS resources**.
   - Chọn **Amazon CloudFront distributions**.
   - Tích chọn tên CloudFront Distribution đã khởi tạo ở bài 5.5.
   - Nhấn **Add**.
5. Nhấn **Next**.

![Cấu hình thông tin Web ACL](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/describe-web-acl.png)
![Cấu hình thông tin Web ACL](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/describe-web-acl2.png)

---

### 2.2.2. Cấu hình Rules và Actions

### Bước 1: Thêm IP Set Rule (Chặn các IP vi phạm)

1. Tại bước **Add rules and rule groups**, nhấn **Add rules** -> chọn **Add my own rules and rule groups**.
2. Cấu hình quy tắc chặn IP:
   - **Rule type:** Chọn **IP set**.
   - **Name:** `BlockAutoIPSetRule`
   - **IP set:** Chọn IP Set `AutoBlockedIPSetV6` đã tạo ở bài 2.1.
   - **Source IP location:** Chọn **Source IP address**.
   - **Action:** Chọn **Block**.
3. Nhấn **Add rule**.

![Cấu hình IP Set Rule](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/add-ip-set-rule.png)

---

### Bước 2: Thêm Rate-based Rule (Giới hạn lưu lượng truy cập)

1. Tiếp tục nhấn **Add rules** -> chọn **Add my own rules and rule groups**.
2. Cấu hình quy tắc Rate limit:
   - **Rule type:** Chọn **Rate-based rule**.
   - **Name:** `HTTPRateLimitRule`
   - **Rate limit:** Nhập ngưỡng giới hạn, ví dụ: `100` (hoặc `100` - `2000` tùy theo yêu cầu bài lab).
   - **Evaluation window:** Chọn **5 minutes** (hoặc thời gian mặc định).
   - **Criteria to aggregate requests:** Chọn **IP address** -> **Source IP address**.
   - **Action:** Chọn **Block**.
3. Nhấn **Add rule**.

![Cấu hình Rate-based Rule](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/add-rate-rule.png)

---

### Bước 3: Cấu hình Default Web ACL Action

1. Tại mục **Default action**, chọn **Allow** (Cho phép tất cả các request thông thường không vi phạm quy tắc).
2. Nhấn **Next**.

---

### 2.2.3. Hoàn tất cài đặt Web ACL

### Bước 1: Thiết lập thứ tự ưu tiên (Set rule priority)

1. Giữ nguyên thứ tự quy tắc (đảm bảo `BlockAutoIPSetRule` đứng trên hoặc được đánh giá hợp lý so với `HTTPRateLimitRule`).
2. Nhấn **Next**.

---

### Bước 2: Review và tạo Web ACL

1. Xem lại toàn bộ thông số đã thiết lập.
2. Nhấn **Create web ACL** ở cuối trang.

![Hoàn tất tạo Web ACL](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/finish-web-acl.png)

---

### 2.2.4. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **Web ACL** có tên `skylink-airline-waf` được khởi tạo thành công tại Scope `Global (CloudFront)`.
- Tích hợp thành công **IP Set Rule** (`BlockAutoIPSetRule`) kết nối với `AutoBlockedIPSetV6`.
- Tích hợp thành công **Rate-based Rule** (`HTTPRateLimitRule`) để theo dõi và giới hạn lượng request từ một IP.
- Web ACL đã gắn thành công vào **CloudFront Distribution**, sẵn sàng lọc và xử lý lưu lượng mạng ở lớp Edge.

---

### 2.3 Cấu hình WAF Access Logging sang CloudWatch Logs

Phần này hướng dẫn chi tiết các bước khởi tạo một Amazon CloudWatch Log Group chuẩn tên quy định và kích hoạt tính năng ghi nhật ký (Access Logging) trên AWS WAF để tự động chuyển toàn bộ log truy cập về CloudWatch.

---

### 2.3.1. Khởi tạo CloudWatch Log Group

### Bước 1: Truy cập dịch vụ CloudWatch Logs

1. Đăng nhập vào **AWS Management Console**.
2. Trên thanh tìm kiếm, nhập `CloudWatch` và chọn dịch vụ **CloudWatch**.
3. Tại menu điều hướng bên trái, mở mục **Logs** và chọn **Log groups**.
4. Đảm bảo Region đang làm việc là **US East (N. Virginia) us-east-1** (vì WAF CloudFront bắt buộc đẩy log về Region này).

![Truy cập CloudWatch Log Groups](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/cloudwatch-log-groups-menu.png)

---

### Bước 2: Tạo Log Group chuẩn tên AWS WAF

1. Nhấn nút **Create log group**.
2. **Log group name:** Nhập chính xác tiền tố bắt buộc của AWS WAF: `aws-waf-logs-cloudfront` (hoặc `aws-waf-logs-website-protection`).
   > **Lưu ý quan trọng:** Tên Log Group của AWS WAF bắt buộc phải bắt đầu bằng `aws-waf-logs-` thì WAF Console mới nhận diện và cho phép liên kết.
3. **Retention setting:** Chọn thời gian lưu trữ log (ví dụ: **1 day** hoặc **7 days** để tiết kiệm chi phí cho bài lab).
4. Nhấn **Create**.

![Khởi tạo CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/create-log-group.png)
![Khởi tạo CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/create-log-group2.png)

---

### 2.3.2. Kích hoạt WAF Logging trên Web ACL

### Bước 1: Mở giao diện Cấu hình Logging trên AWS WAF

1. Quay lại dịch vụ **AWS WAF & Shield**.
2. Chọn **Web ACLs** ở menu bên trái -> Chọn **Global (CloudFront)**.
3. Bấm chọn Web ACL `skylink-airline-waf` đã khởi tạo ở bài 2.2.
4. Chuyển sang tab **Logging and metrics**.
5. Tại mục **Logging**, nhấn nút **Enable**.

![Bật Logging trên Web ACL](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/enable-waf-logging.png)

---

### Bước 2: Liên kết với CloudWatch Log Group

1. **Logging destination:** Chọn **CloudWatch Logs log group**.
2. **CloudWatch Logs log group:** Chọn đúng Log Group `aws-waf-logs-cloudfront` vừa tạo ở Phần 1.
3. **Redacted fields (Tùy chọn):** Giữ mặc định (không ẩn trường thông tin nào) hoặc chọn ẩn các trường nhạy cảm nếu cần.
4. **Filter logs (Tùy chọn):** Giữ mặc định để ghi nhận toàn bộ log (All traffic).
5. Nhấn **Save**.

![Liên kết WAF với CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/link-log-destination.png)
![Liên kết WAF với CloudWatch Log Group](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/link-log-destination2.png)

---

### 2.3.3. Kiểm tra dữ liệu Log ghi nhận

### Bước 1: Phát sinh lưu lượng truy cập

1. Truy cập vào tên miền CloudFront Distribution (đã lấy ở bài 5.5) trên trình duyệt hoặc gửi một số request tới trang web để tạo truy cập thực tế.

---

### Bước 2: Kiểm tra Stream Log trong CloudWatch

1. Quay lại dịch vụ **CloudWatch** -> **Log groups** -> chọn `aws-waf-logs-cloudfront`.
2. Trong tab **Log streams**, kiểm tra danh sách các Log Stream mới xuất hiện chứa dữ liệu nhật ký dạng JSON của AWS WAF.

![Kiểm tra WAF Log Streams](/images/5-Workshop/5.6-Configure-AWS-WAF-Logging/verify-log-streams.png)

---

### 2.3.4. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **CloudWatch Log Group** tên `aws-waf-logs-cloudfront` được khởi tạo thành công tại Region `us-east-1`.
- **WAF Access Logging** được kích hoạt thành công trên Web ACL `skylink-airline-waf`.
- Toàn bộ lưu lượng HTTP/HTTPS gửi tới CloudFront được ghi nhận chi tiết dưới dạng JSON Log Streams trong CloudWatch, sẵn sàng làm nguồn dữ liệu kích hoạt cho CloudWatch Alarms và Lambda tự động hóa ở bài 5.7 & 5.8.