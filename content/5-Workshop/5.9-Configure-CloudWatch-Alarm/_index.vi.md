---
title: "Cấu hình CloudWatch Alarm"
date: 2026-08-24
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

### Mục tiêu

Khởi tạo và thiết lập Amazon CloudWatch Alarm để liên tục giám sát lượng request truy cập gửi tới hệ thống. Khi số lượng request vượt quá ngưỡng an toàn cho phép, CloudWatch Alarm sẽ tự động chuyển sang trạng thái `ALARM` và phát tín hiệu thông báo đến Amazon SNS Topic để kích hoạt chuỗi phản ứng tự động.

---

## 1. Tổng quan

Trong mô hình bảo mật tự động hóa, **Amazon CloudWatch Alarm** đóng vai trò là cơ chế phát hiện sự cố (Detection Mechanism). Dịch vụ này thu thập các chỉ số (Metrics) từ AWS WAF hoặc CloudWatch Logs để đánh giá tình trạng lưu lượng mạng theo thời gian thực.

Quy trình hoạt động của CloudWatch Alarm bao gồm:

- **Giám sát chỉ số (Metric Monitoring):** Lấy dữ liệu từ WAF Access Logs hoặc CloudWatch Log Group `aws-waf-logs-cloudfront` để đo đạc số lượng truy cập trong một khoảng thời gian nhất định (ví dụ: 1 phút hoặc 5 phút).
- **Kích hoạt cảnh báo (Alarm Trigger):** Khi số lượng request vượt ngưỡng quy định (ví dụ: > 100 requests/5 phút), Alarm sẽ lập tức đổi trạng thái sang `ALARM`.
- **Phát tín hiệu thông báo (SNS Notification):** CloudWatch Alarm tự động gửi thông điệp cảnh báo tới SNS Topic `skylink-waf-alerts`, từ đó gửi email thông báo cho quản trị viên và kích hoạt hàm AWS Lambda `WAFAutoBlockFunction` thực thi chặn IP.

---

## 2. Quy trình triển khai

Quy trình cấu hình CloudWatch Alarm được chia thành hai bài thực hành chi tiết:

- **Bài 5.9.1:** Tạo CloudWatch Log Metric Filter để trích xuất chỉ số đếm truy cập từ WAF Log Group.
- **Bài 5.9.2:** Khởi tạo CloudWatch Alarm (`WAFHighRequestRateAlarm`), thiết lập ngưỡng kích hoạt và liên kết hành động gửi thông báo tới SNS Topic `skylink-waf-alerts`.

---

## 3. Nội dung thực hành

Thực hiện lần lượt các bài thực hành sau:

- **[5.9.1. Tạo CloudWatch Log Metric Filter](5.9.1-Create-CloudWatch-Log-Metric-Filter)**
- **[5.9.2. Tạo và cấu hình CloudWatch Alarm](5.9.2-Create-and-Configure-CloudWatch-Alarm)**

---

## 4. Kết quả mong đợi

Sau khi hoàn thành chương này, bạn sẽ đạt được:

- **CloudWatch Metric Filter** được thiết lập thành công trên Log Group `aws-waf-logs-cloudfront`.
- **CloudWatch Alarm** có tên `WAFHighRequestRateAlarm` được khởi tạo thành công tại Region `us-east-1`.
- Hệ thống sẵn sàng tự động phát hiện các cuộc tấn công HTTP Flood/DDoS và gửi tín hiệu kích hoạt chuỗi ứng phó sự cố tự động.
