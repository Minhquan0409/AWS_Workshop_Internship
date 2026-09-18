---
title: "Khởi tạo IAM Role cho Lambda"
date: 2026-08-24
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

Bài viết này hướng dẫn chi tiết các bước tạo mới một IAM Policy chính xác và khởi tạo IAM Execution Role cho hàm AWS Lambda trên AWS Management Console. IAM Role này cấp các quyền truy cập tối thiểu (Least Privilege) giúp Lambda đọc dữ liệu log từ CloudWatch Logs và cập nhật IP vi phạm vào WAF IP Set.

---

## 1. Khởi tạo IAM Policy cho Lambda

### Bước 1: Truy cập dịch vụ IAM

1. Đăng nhập vào **AWS Management Console**.
2. Trên thanh tìm kiếm, nhập `IAM` và chọn dịch vụ **IAM**.
3. Tại menu điều hướng bên trái, chọn **Policies**.
4. Nhấn nút **Create policy**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu.png)

---

### Bước 2: Định nghĩa chính sách IAM (JSON)

1. Trong giao diện Create policy, chọn tab **JSON**.
2. Dán đoạn mã JSON phân quyền chi tiết dưới đây vào khung soạn thảo:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CloudWatchLogsPermissions",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:FilterLogEvents",
        "logs:GetLogEvents"
      ],
      "Resource": "*"
    },
    {
      "Sid": "WAFGetAndUpdateIPSetPermissions",
      "Effect": "Allow",
      "Action": ["wafv2:GetIPSet", "wafv2:UpdateIPSet"],
      "Resource": "*"
    }
  ]
}
```

3. Nhấn nút **Next**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu1.png)

---

### Bước 3: Đặt tên và lưu IAM Policy

1. Tại mục **Policy name**, nhập `AWSLambdaWAFAutoBlockPolicy`.
2. **Description**: Nhập `Chinh sach cap quyen cho Lambda doc CloudWatch Logs va cap nhat WAF IP Set.`
3. Nhấn **Create policy**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu2.png)
![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu3.png)

---

## 2. Khởi tạo IAM Execution Role

### Bước 1: Tạo IAM Role mới

1. Tại menu điều hướng bên trái giao diện IAM, chọn **Roles**.
2. Nhấn nút **Create role**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu4.png)

---

### Bước 2: Chọn Trusted Entity

1. **Trusted entity type**: Chọn **AWS service**.
2. **Use case**: Chọn **Lambda**.
3. Nhấn **Next**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu5.png)

---

### Bước 3: Gắn IAM Policy vào Role

1. Trên thanh tìm kiếm chính sách, nhập `AWSLambdaWAFAutoBlockPolicy`.
2. Tích chọn ô vuông trước policy **AWSLambdaWAFAutoBlockPolicy** vừa tạo ở Phần 1.
3. Nhấn **Next**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu6.png)

---

### Bước 4: Hoàn tất tạo IAM Role

1. **Role name**: Nhập `LambdaWAFAutoBlockRole`.
2. **Description**: Nhập `Execution Role cho ham Lambda tu dong phan tich log va block IP tren AWS WAF.`
3. Kiểm tra lại thông tin policy đã đính kèm.
4. Nhấn **Create role**.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu7.png)
![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu8.png)

---

## 3. Kiểm tra kết quả

Sau khi hoàn thành tạo Role, kiểm tra giao diện chi tiết của `LambdaWAFAutoBlockRole`:

- **Role Name**: `LambdaWAFAutoBlockRole`
- **ARN**: `arn:aws:iam::881081146390:role/LambdaWAFAutoBlockRole`
- **Permissions**: Đã đính kèm thành công chính sách `AWSLambdaWAFAutoBlockPolicy`.

![Truy cập IAM Policies](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/iam-policies-menu9.png)

---

## 4. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- IAM Policy `AWSLambdaWAFAutoBlockPolicy` được định nghĩa chuẩn xác các quyền `wafv2:GetIPSet`, `wafv2:UpdateIPSet` và các quyền thao tác trên CloudWatch Logs.
- IAM Role `LambdaWAFAutoBlockRole` được khởi tạo sẵn sàng để gán trực tiếp làm Execution Role cho hàm AWS Lambda ở bài 5.8.2.
