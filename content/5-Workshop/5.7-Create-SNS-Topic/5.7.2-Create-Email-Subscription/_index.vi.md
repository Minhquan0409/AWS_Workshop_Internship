---
title: "Tạo và xác thực Email Subscription"
date: 2026-08-24
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

Bài viết này hướng dẫn chi tiết các bước đăng ký địa chỉ email cá nhân nhận thông báo từ Amazon SNS Topic và thực hiện quy trình xác thực (Confirm Subscription) để hoàn tất liên kết truyền thông tin cảnh báo.

---

## 1. Tạo Email Subscription trên Amazon SNS

### Bước 1: Truy cập SNS Topic

1. Đăng nhập vào **AWS Management Console**.
2. Truy cập dịch vụ **Simple Notification Service (SNS)**.
3. Tại menu điều hướng bên trái, chọn **Topics** và nhấn chọn SNS Topic `skylink-waf-alerts` đã khởi tạo ở bài 5.7.1.

![Chọn SNS Topic WAFAlertTopic](/images/5-Workshop/5.7-Create-SNS-Topic/select-sns-topic.png)

---

### Bước 2: Cấu hình thông số Subscription

1. Trong giao diện chi tiết của `skylink-waf-alerts`, chuyển sang tab **Subscriptions**.
2. Nhấn nút **Create subscription**.

![Nhấn Create Subscription](/images/5-Workshop/5.7-Create-SNS-Topic/click-create-subscription.png)

3. Cấu hình các thông số:
   - **Topic ARN:** Giữ nguyên ARN của `skylink-waf-alerts` (tự động điền).
   - **Protocol:** Chọn **Email**.
   - **Endpoint:** Nhập địa chỉ email cá nhân hoặc email quản trị viên của bạn (ví dụ: `admin@example.com`).
4. Nhấn nút **Create subscription** ở cuối trang.

![Cấu hình Email Subscription](/images/5-Workshop/5.7-Create-SNS-Topic/configure-subscription.png)
![Cấu hình Email Subscription](/images/5-Workshop/5.7-Create-SNS-Topic/configure-subscription2.png)

---

## 2. Xác thực Email Subscription (Confirm Subscription)

Sau khi tạo thành công, trạng thái của Subscription sẽ ở mức **Pending confirmation**. AWS sẽ tự động gửi một email xác thực đến hòm thư đã đăng ký.

### Bước 1: Kiểm tra hộp thư Email

1. Mở hộp thư điện tử của địa chỉ email đã đăng ký ở bước trước.
2. Tìm thư điện tử có tiêu đề: `AWS Notification - Subscription Confirmation` gửi từ **AWS Notifications**.

![Kiểm tra Email xác thực](/images/5-Workshop/5.7-Create-SNS-Topic/check-email-inbox.png)

---

### Bước 2: Bấm liên kết xác thực

1. Mở thư email và nhấn vào đường link **Confirm subscription**.

2. Trình duyệt sẽ mở ra trang xác nhận của AWS với thông điệp **Subscription confirmed!**.

![Màn hình xác nhận thành công](/images/5-Workshop/5.7-Create-SNS-Topic/subscription-confirmed-page.png)

---

## 3. Kiểm tra trạng thái trên Console

### Bước 1: Kiểm tra lại trên AWS SNS Dashboard

1. Quay lại trang quản lý **AWS Management Console** -> Dịch vụ **SNS** -> **Topics** -> `skylink-waf-alerts`.
2. Trong tab **Subscriptions**, kiểm tra danh sách:
   - **Status:** Chuyển từ `Pending confirmation` sang `Confirmed` (màu xanh).

![Kiểm tra trạng thái Confirmed trên Console](/images/5-Workshop/5.7-Create-SNS-Topic/verify-confirmed-status.png)

---

## 4. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- Địa chỉ email quản trị được liên kết thành công với Amazon SNS Topic `skylink-waf-alerts`.
- Trạng thái Subscription đạt mức **Confirmed**, sẵn sàng nhận các thông báo tự động từ CloudWatch Alarms gửi về hòm thư điện tử khi hệ thống phát hiện mối đe dọa.
