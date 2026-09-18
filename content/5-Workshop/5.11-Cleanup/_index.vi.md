---
title: "Dọn dẹp tài nguyên"
date: 2026-08-24
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Mục tiêu

Sau khi hoàn tất quá trình kiểm thử và nghiệm thu hệ thống, việc dọn dẹp các tài nguyên đã tạo trên AWS là bước quan trọng nhằm tránh phát sinh chi phí không mong muốn (Cost Optimization) và đảm bảo an toàn bảo mật cho tài khoản AWS.

---

## Các bước dọn dẹp tài nguyên

Để đảm bảo các tài nguyên không bị vướng phụ thuộc (Dependency Errors) khi xóa, hãy thực hiện lần lượt theo thứ tự bên dưới:

### Bước 1: Xóa CloudFront Distribution

1. Truy cập **AWS CloudFront Console** -> chọn **Distributions**.
2. Chọn Distribution đã tạo cho dự án (`https://d2n9euazwledbh.cloudfront.net/`).
3. Bấm **Disable** và chờ trạng thái chuyển hẳn sang _Disabled_.
4. Sau khi đã _Disabled_, chọn Distribution đó và bấm **Delete**.

![Disable và xóa CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution.png)
![Disable và xóa CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution2.png)
![Disable và xóa CloudFront Distribution](/images/5-Workshop/5.11-Cleanup/buoc1-delete-cloudfront-distribution3.png)

---

### Bước 2: Gỡ bỏ Web ACL khỏi WAF & Xóa WAF Rules / IP Sets

1. Truy cập **AWS WAF** -> chọn **Web ACLs** (chuyển Region sang **Global (CloudFront)**).
2. Chọn Web ACL của dự án -> chuyển sang tab **Associated AWS resources** -> Bấm **Disassociate** để hủy liên kết với CloudFront Distribution (nếu còn).
   ![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets.png)
3. Vào tab **Rules** -> Xóa các Rule chặn IP đã tạo.
   ![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets2.png)
4. Bấm **Delete** để xóa Web ACL.
   ![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets3.png)
   ![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets4.png)
5. Chuyển sang mục **IP sets** (Region **Global (CloudFront)**) -> Chọn và xóa lần lượt cả 2 IP Sets:
   - `AutoBlockedIPSetV6`
   - `AutoBlockedIPSet`

![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets5.png)
![Gỡ liên kết Web ACL và xóa WAF IP Sets](/images/5-Workshop/5.11-Cleanup/buoc2-delete-waf-and-ipsets6.png)

---

### Bước 3: Xóa AWS Lambda Function & IAM Role/Policy

1. Truy cập **AWS Lambda Console** -> chọn **Functions**.
2. Chọn hàm `WAFAutoBlockFunction` -> Bấm **Actions** -> Chọn **Delete**.

![Xóa Lambda Function WAFAutoBlockFunction](/images/5-Workshop/5.11-Cleanup/buoc3a-delete-lambda-function1.png)
![Xóa Lambda Function WAFAutoBlockFunction](/images/5-Workshop/5.11-Cleanup/buoc3a-delete-lambda-function2.png)

3. Truy cập **IAM Console**:
   - Vào mục **Roles** -> Tìm và xóa Role `LambdaWAFAutoBlockRole`.
   - Vào mục **Policies** -> Tìm và xóa Policy `LambdaWAFAutoBlockPolicy`.

![Xóa IAM Role và IAM Policy](/images/5-Workshop/5.11-Cleanup/buoc3b-delete-iam-role-policy1.png)
![Xóa IAM Role và IAM Policy](/images/5-Workshop/5.11-Cleanup/buoc3b-delete-iam-role-policy2.png)

---

### Bước 4: Xóa CloudWatch Alarms & Log Groups

1. Truy cập **CloudWatch Console** -> chọn **Alarms** -> **All alarms**.
2. Chọn `WAFHighRequestRateAlarm` -> Bấm **Actions** -> Chọn **Delete**.

![Xóa CloudWatch Alarm](/images/5-Workshop/5.11-Cleanup/buoc4a-delete-cloudwatch-alarm.png)

3. Vào mục **Logs** -> chọn **Log groups**.
4. Chọn và xóa các Log Group sau:
   - `aws-waf-logs-cloudfront`
   - `/aws/lambda/WAFAutoBlockFunction`

![Xóa CloudWatch Log Groups](/images/5-Workshop/5.11-Cleanup/buoc4b-delete-log-groups.png)
![Xóa CloudWatch Log Groups](/images/5-Workshop/5.11-Cleanup/buoc4b-delete-log-groups1.png)

---

### Bước 5: Xóa Amazon SNS Topic & Subscriptions

1. Truy cập **Amazon SNS Console** -> chọn **Topics**.
2. Chọn `skylink-waf-alerts` -> Bấm **Delete**.
3. Vào mục **Subscriptions** -> Chọn các Subscription liên quan đến Email và Lambda -> Bấm **Delete**.

![Xóa Amazon SNS Topic và Subscriptions](/images/5-Workshop/5.11-Cleanup/buoc5-delete-sns-topic.png)
![Xóa Amazon SNS Topic và Subscriptions](/images/5-Workshop/5.11-Cleanup/buoc5-delete-sns-topic1.png)

---

### Bước 6: Xóa S3 Bucket / Origin Server (Nếu có)

1. Truy cập **Amazon S3 Console**.
2. Chọn S3 Bucket lưu trữ mã nguồn website/portfolio.
3. Bấm **Empty** để xóa toàn bộ dữ liệu bên trong Bucket.
4. Bấm **Delete** để xóa hẳn S3 Bucket.

![Dọn dẹp và xóa Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket1.png)
![Dọn dẹp và xóa Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket2.png)
![Dọn dẹp và xóa Amazon S3 Bucket](/images/5-Workshop/5.11-Cleanup/buoc6-delete-s3-bucket3.png)

---

## Bảng xác nhận hoàn tất dọn dẹp

| Dịch vụ AWS    | Tài nguyên cần xóa                                               | Trạng thái           |
| :------------- | :--------------------------------------------------------------- | :------------------- |
| **CloudFront** | Distribution (`https://d2n9euazwledbh.cloudfront.net/`)                    | **Đã xóa (Deleted)** |
| **AWS WAF**    | Web ACL, `AutoBlockedIPSetV6`, `AutoBlockedIPSet`                | **Đã xóa (Deleted)** |
| **AWS Lambda** | Function `WAFAutoBlockFunction`                                  | **Đã xóa (Deleted)** |
| **IAM**        | Role `LambdaWAFAutoBlockRole`, Policy `LambdaWAFAutoBlockPolicy` | **Đã xóa (Deleted)** |
| **CloudWatch** | Alarm `WAFHighRequestRateAlarm`, Log Groups WAF & Lambda         | **Đã xóa (Deleted)** |
| **Amazon SNS** | Topic `skylink-waf-alerts` & Email Subscriptions                      | **Đã xóa (Deleted)** |

> **Lưu ý:** Việc xóa tài nguyên là vĩnh viễn và không thể khôi phục. Toàn bộ cấu hình hệ thống và mã nguồn dự án đã được sao lưu an toàn tại [GitHub Repository](https://github.com/quangg199/SkyLink-Airline-System).
