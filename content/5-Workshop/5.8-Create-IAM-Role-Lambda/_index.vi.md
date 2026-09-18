---
title: "Tạo IAM Role & Hàm AWS Lambda"
date: 2026-08-24
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Mục tiêu

Khởi tạo vai trò IAM Role cấp quyền thực thi tối thiểu (Least Privilege) và triển khai hàm AWS Lambda xử lý sự cố tự động. Hàm Lambda sẽ phân tích dữ liệu log WAF từ CloudWatch Logs để trích xuất IP vi phạm, sau đó tự động cập nhật địa chỉ IP đó vào WAF IP Set nhằm chủ động ngăn chặn các cuộc tấn công DDoS/HTTP Flood.

---

## 1. Tổng quan

Trong kiến trúc bảo mật phản ứng tự động, **AWS Lambda** đóng vai trò là "bộ não" thực thi quy trình ứng phó sự cố mà không cần sự can thiệp thủ công từ con người.

Khi hệ thống ghi nhận lượng truy cập vượt ngưỡng, quy trình phản ứng diễn ra theo các bước:

- **Phân quyền bảo mật (IAM Role):** Hàm Lambda cần được gán một **IAM Execution Role** chứa chính sách IAM Policy tinh chỉnh. Vai trò này cấp quyền truy cập chính xác vào **CloudWatch Logs** (để đọc log WAF) và **AWS WAF** (để thực thi lệnh ghi đè/thêm IP vi phạm vào `AutoBlockedIPSetV6`).
- **Xử lý tự động (Lambda Function):** Hàm Python sẽ trích xuất dữ liệu log truy cập từ CloudWatch Logs, thống kê địa chỉ IP gửi số lượng request bất thường, và tự động gọi API `UpdateIPSet` của AWS WAF để đưa IP đó vào danh sách chặn ngay tại lớp Edge.

---

## 2. Quy trình triển khai

Quy trình cấu hình IAM Role và Hàm AWS Lambda được chia thành hai bài thực hành riêng biệt:

- **Bài 5.8.1:** Khởi tạo IAM Policy và IAM Execution Role (`LambdaWAFAutoBlockRole`) cung cấp đầy đủ quyền hạn cho Lambda.
- **Bài 5.8.2:** Khởi tạo hàm AWS Lambda (`WAFAutoBlockFunction`), cấu hình biến môi trường (Environment Variables) và triển khai mã nguồn Python tự động hóa.

---

## 3. Nội dung thực hành

Thực hiện lần lượt các bài thực hành sau:

- **[5.8.1. Khởi tạo IAM Role cho Lambda](5.8.1-Create-IAM-Role-for-Lambda)**
- **[5.8.2. Khởi tạo và triển khai hàm AWS Lambda](5.8.2-Create-and-Deploy-AWS-Lambda-Function)**

---

## 4. Kết quả mong đợi

Sau khi hoàn thành chương này, bạn sẽ đạt được:

- **IAM Execution Role** có tên `LambdaWAFAutoBlockRole` được khởi tạo thành công với nguyên tắc quyền hạn tối thiểu.
- **AWS Lambda Function** có tên `WAFAutoBlockFunction` được triển khai mã nguồn Python hoàn chỉnh tại Region `us-east-1`.
- Hàm Lambda sẵn sàng nhận kích hoạt (Trigger) từ Amazon SNS hoặc CloudWatch Alarms để tự động phân tích log và cập nhật IP vi phạm vào WAF IP Set.
