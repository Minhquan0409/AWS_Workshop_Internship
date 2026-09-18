---
title: "Tạo CloudWatch Log Metric Filter"
date: 2026-08-24
weight: 1
chapter: false
pre: " <b> 5.9.1. </b> "
---

Bài viết này hướng dẫn chi tiết các bước khởi tạo một CloudWatch Log Metric Filter trên AWS Management Console. Metric Filter đóng vai trò quét toàn bộ dữ liệu log từ AWS WAF Log Group để đếm số lượng bản ghi (requests) và chuyển đổi thành một Custom Metric giúp CloudWatch Alarm đo đạc theo thời gian thực.

---

## 1. Khởi tạo Log Metric Filter

### Bước 1: Truy cập Log Group của AWS WAF

1. Đăng nhập vào **AWS Management Console**.
2. Tìm kiếm và chọn dịch vụ **CloudWatch**.
3. Tại menu điều hướng bên trái, mở mục **Logs** -> Chọn **Log groups**.
4. Đảm bảo Region đang làm việc là **US East (N. Virginia) us-east-1**.
5. Bấm chọn Log Group `aws-waf-logs-cloudfront` đã khởi tạo ở bài 5.6.3.

![Chọn Log Group AWS WAF](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/select-log-group.png)

---

### Bước 2: Tạo Metric Filter

1. Trong giao diện chi tiết của `aws-waf-logs-cloudfront`, chuyển sang tab **Metric filters**.
2. Nhấn nút **Create metric filter**.

![Chọn Create Metric Filter](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/click-create-metric-filter.png)

---

### Bước 3: Định nghĩa Filter Pattern

1. **Filter pattern:** Nhập `{ $.httpRequest.clientIP = "*" }` (mẫu này khớp với tất cả bản ghi log WAF chứa địa chỉ client IP).
2. Mở mục **Test pattern** chọn một Log Stream thực tế để kiểm tra thử nghiệm xem pattern có hoạt động chính xác hay không.
3. Nhấn **Next**.

![Định nghĩa Filter Pattern](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/define-filter-pattern.png)

---

### Bước 4: Thiết lập Metric Details

1. **Filter name:** Nhập `WAFRequestCountFilter`.
2. **Metric namespace:** Nhập `WAFCustomMetrics` (hoặc chọn namespace tùy chỉnh cá nhân).
3. **Metric name:** Nhập `WAFRequestCount`.
4. **Metric value:** Nhập `1` (mỗi bản ghi log khớp pattern sẽ được tính là 1 đơn vị request).
5. **Default value:** Để trống hoặc nhập `0`.
6. **Unit:** Chọn **Count**.
7. Nhấn **Next**.

![Cấu hình Metric Details](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm//configure-metric-details.png)

---

### Bước 5: Xem lại và hoàn tất

1. Kiểm tra lại toàn bộ thông số cấu hình.
2. Nhấn **Create metric filter**.

![Hoàn tất tạo Metric Filter](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-metric-filter.png)
![Hoàn tất tạo Metric Filter](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-metric-filter1.png)

---

## 2. Kiểm tra Custom Metric vừa tạo

1. Quay lại tab **Metric filters** trong Log Group `aws-waf-logs-cloudfront`.
2. Kiểm tra filter `WAFRequestCountFilter` hiển thị thành công.
3. Nhấn vào tên Custom Metric `WAFRequestCount` (hoặc truy cập mục **Metrics** -> **All metrics** -> **WAFCustomMetrics**) để xem biểu đồ thống kê lưu lượng request thực tế theo thời gian.

![Kiểm tra Custom Metric hiển thị](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/verify-custom-metric.png)

---

## 3. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **CloudWatch Log Metric Filter** có tên `WAFRequestCountFilter` được gắn thành công vào Log Group `aws-waf-logs-cloudfront`.
- Custom Metric `WAFRequestCount` thuộc Namespace `WAFCustomMetrics` được khởi tạo, liên tục ghi nhận dữ liệu đếm request từ WAF Log Streams và sẵn sàng làm nguồn dữ liệu đo đạc cho CloudWatch Alarm ở bài 5.9.2.
