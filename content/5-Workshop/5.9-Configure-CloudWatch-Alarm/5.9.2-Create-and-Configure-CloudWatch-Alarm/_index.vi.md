---
title: "Tạo và cấu hình CloudWatch Alarm"
date: 2026-08-24
weight: 2
chapter: false
pre: " <b> 5.9.2. </b> "
---

Bài viết này hướng dẫn chi tiết các bước tạo mới một Amazon CloudWatch Alarm trên AWS Management Console. CloudWatch Alarm sẽ liên tục theo dõi Custom Metric `WAFRequestCount` từ Log Group và tự động kích hoạt thông báo gửi tới SNS Topic `WAFAlertTopic` khi lưu lượng request vượt ngưỡng quy định.

---

## 1. Khởi tạo CloudWatch Alarm

### Bước 1: Truy cập giao diện CloudWatch Alarms

1. Đăng nhập vào **AWS Management Console**.
2. Tìm kiếm và chọn dịch vụ **CloudWatch**.
3. Tại menu điều hướng bên trái, chọn **Alarms** -> Chọn **All alarms**.
4. Đảm bảo Region đang làm việc là **US East (N. Virginia) us-east-1**.
5. Bấm nút **Create alarm**.

![Truy cập CloudWatch Alarms](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/access-alarms-menu.png)

---

### Bước 2: Chọn Metric cho Alarm

1. Nhấn nút **Select metric**.
2. Tìm kiếm và chọn **WAFCustomMetrics** (Namespace đã tạo ở bài 5.9.1).
3. Chọn mục **Metrics with no dimensions** -> Tích chọn metric **WAFRequestCount**.
4. Nhấn **Select metric**.

![Chọn Custom Metric WAFRequestCount](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/select-metric.png)

---

### Bước 3: Cấu hình Metric và Điều kiện Cảnh báo (Conditions)

1. Trong mục **Metric**:
   - **Statistic:** Chọn **Sum** (Tổng số request đếm được).
   - **Period:** Chọn **5 minutes** (Khoảng thời gian đánh giá 5 phút).
2. Trong mục **Conditions**:
   - **Threshold type:** Chọn **Static**.
   - **Whenever WAFRequestCount is...:** Chọn **Greater/Equal** (`>=`).
   - **than...:** Nhập `100` (Ngưỡng cảnh báo: 100 requests trong 5 phút).
3. Nhấn **Next**.

![Cấu hình Điều kiện Cảnh báo](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/configure-conditions.png)

---

### Bước 4: Cấu hình Hành động (Actions) & Liên kết SNS Topic

1. **Alarm state trigger:** Chọn **In alarm** (Kích hoạt khi chuyển sang trạng thái cảnh báo).
2. **Send a notification to the following SNS topic:**
   - Chọn **Select an existing SNS topic**.
   - **Send a notification to...:** Chọn SNS Topic `skylink-waf-alerts` đã khởi tạo ở bài 5.7.1.
3. Nhấn **Next**.

![Liên kết SNS Topic](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/configure-actions-sns.png)

---

### Bước 5: Tên và Đánh giá Alarm

1. **Alarm name:** Nhập `WAFHighRequestRateAlarm`.
2. **Alarm description:** Nhập `Canh bao khi luong truy cap WAF vuot qua 100 requests trong 5 phut`.
3. Nhấn **Next**.
4. Kiểm tra lại toàn bộ thông số và nhấn **Create alarm**.

![Xem lại và tạo Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm.png)
![Xem lại và tạo Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm1.png)
![Xem lại và tạo Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/finish-create-alarm2.png)

---

## 2. Kiểm tra trạng thái CloudWatch Alarm

1. Quay lại danh sách **All alarms**.
2. Kiểm tra Alarm `WAFHighRequestRateAlarm`:
   - **State:** Ban đầu sẽ ở trạng thái `OK` hoặc `Insufficient data` (sau vài phút thu thập dữ liệu sẽ về trạng thái `OK`).
   - **Conditions:** `>= 100 for 1 datapoints within 5 minutes`.

![Kiểm tra trạng thái Alarm](/images/5-Workshop/5.9-Configure-CloudWatch-Alarm/verify-alarm-status.png)

---

## 3. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **CloudWatch Alarm** có tên `WAFHighRequestRateAlarm` được khởi tạo thành công tại Region `us-east-1`.
- Alarm được liên kết trực tiếp với Custom Metric `WAFRequestCount` và thiết lập hành động thông báo tới SNS Topic `skylink-waf-alerts`.
- Toàn bộ chuỗi phát hiện sự cố (Metric Filter -> CloudWatch Alarm -> SNS Topic) đã sẵn sàng tích hợp với AWS Lambda để thực thi chặn IP tự động ở bước kiểm thử.
