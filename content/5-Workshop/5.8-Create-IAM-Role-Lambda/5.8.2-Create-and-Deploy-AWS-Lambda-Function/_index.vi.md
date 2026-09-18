---
title: "Khởi tạo và triển khai hàm AWS Lambda"
date: 2026-08-24
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

Bài viết này hướng dẫn chi tiết các bước tạo mới một hàm AWS Lambda bằng ngôn ngữ Python, cấu hình Execution Role, thiết lập các biến môi trường (Environment Variables) và triển khai mã nguồn tự động trích xuất IP vi phạm từ CloudWatch Logs để cập nhật vào WAF IP Set.

---

## 1. Khởi tạo AWS Lambda Function

### Bước 1: Truy cập dịch vụ AWS Lambda

1. Đăng nhập vào **AWS Management Console**.
2. Trên thanh tìm kiếm, nhập `Lambda` và chọn dịch vụ **Lambda**.
3. Đảm bảo Region đang chọn là **US East (N. Virginia) us-east-1**.
4. Bấm nút **Create function**.

![Truy cập dịch vụ AWS Lambda](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/lambda-dashboard.png)

---

### Bước 2: Cấu hình thông số cơ bản

1. Chọn **Author from scratch**.
2. **Function name**: Nhập `WAFAutoBlockFunction`.
3. **Runtime**: Chọn **Python 3.12** (hoặc phiên bản Python 3.x mới nhất).
4. **Architecture**: Giữ mặc định **x86_64**.
5. Mở mục **Change default execution role**:
   - Chọn **Use an existing role**.
   - **Existing role**: Chọn role `LambdaWAFAutoBlockRole` đã tạo ở bài 5.8.1.
6. Nhấn nút **Create function**.

![Cấu hình thông số khởi tạo Lambda](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/create-function-config.png)

![Cấu hình thông số khởi tạo Lambda](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/create-function-config1.png)

---

## 2. Cấu hình Biến môi trường & General Configuration

### Bước 1: Thiết lập Environment Variables

1. Tại giao diện chi tiết của hàm `WAFAutoBlockFunction`, chọn tab **Configuration** -> Chọn mục **Environment variables**.
2. Nhấn **Edit** và bổ sung các cặp Key - Value sau:

| Key               | Value                     | Diễn giải                                       |
| :---------------- | :------------------------ | :---------------------------------------------- |
| `LOG_GROUP_NAME`  | `aws-waf-logs-cloudfront` | Tên CloudWatch Log Group của WAF                |
| `IPSET_NAME`      | `AutoBlockedIPSetV6`      | Tên WAF IP Set lưu danh sách chặn               |
| `IPSET_SCOPE`     | `CLOUDFRONT`              | Phạm vi hoạt động của IP Set (CLOUDFRONT)       |
| `LIMIT_THRESHOLD` | `100`                     | Ngưỡng số lượng request tối đa cho phép từ 1 IP |

3. Nhấn **Save**.

![Cấu hình Biến môi trường](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/environment-variables.png)

---

### Bước 2: Điều chỉnh Timeout cho Lambda

1. Cũng tại tab **Configuration**, chọn mục **General configuration**.
2. Nhấn **Edit**.
3. **Timeout**: Điều chỉnh thời gian xử lý từ mặc định `3 sec` lên **1 min 0 sec** (để đảm bảo đủ thời gian truy vấn CloudWatch Logs Insights và cập nhật WAF API).
4. Nhấn **Save**.

![Điều chỉnh Timeout Lambda](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/general-configuration.png)

---

## 3. Triển khai Mã nguồn Python (Source Code)

### Bước 1: Nhập mã nguồn Lambda

1. Chuyển sang tab **Code**.
2. Trong khung soạn thảo `lambda_function.py`, dán đoạn mã nguồn Python tự động hóa dưới đây:

```python
import os
import time
import boto3
from datetime import datetime, timedelta

# Khởi tạo AWS SDK Clients
logs_client = boto3.client('logs', region_name='us-east-1')
waf_client = boto3.client('wafv2', region_name='us-east-1')

def lambda_handler(event, context):
    log_group_name = os.environ.get('LOG_GROUP_NAME', 'aws-waf-logs-cloudfront')
    ipset_name = os.environ.get('IPSET_NAME', 'AutoBlockedIPSetV6')
    ipset_scope = os.environ.get('IPSET_SCOPE', 'CLOUDFRONT')
    threshold = int(os.environ.get('LIMIT_THRESHOLD', '100'))

    print(f"Bắt đầu phân tích log từ Log Group: {log_group_name} với ngưỡng: {threshold}")

    # Query tìm các IP vượt ngưỡng truy cập trong 5 phút qua
    query = f"fields httpRequest.clientIP | stats count(*) as requestCount by httpRequest.clientIP | filter requestCount > {threshold}"

    end_time = datetime.utcnow()
    start_time = end_time - timedelta(minutes=5)

    start_query_response = logs_client.start_query(
        logGroupName=log_group_name,
        startTime=int(start_time.timestamp()),
        endTime=int(end_time.timestamp()),
        queryString=query
    )

    query_id = start_query_response['queryId']
    response = None

    # Chờ CloudWatch Insights hoàn tất truy vấn
    while response is None or response['status'] == 'Running':
        time.sleep(1)
        response = logs_client.get_query_results(queryId=query_id)

    offending_ips = []
    for row in response['results']:
        for field in row:
            if field['field'] == 'httpRequest.clientIP':
                ip = field['value']
                # Định dạng IPv6 /128 hoặc IPv4 /32
                ip_cidr = f"{ip}/128" if ":" in ip else f"{ip}/32"
                offending_ips.append(ip_cidr)

    print(f"Tìm thấy danh sách IP vi phạm: {offending_ips}")

    if not offending_ips:
        return {'statusCode': 200, 'body': 'Không phát hiện IP vi phạm ngưỡng.'}

    # Lấy thông tin IP Set hiện tại từ WAF
    ip_sets = waf_client.list_ip_sets(Scope=ipset_scope)
    ipset_id = None
    for ipset in ip_sets.get('IPSets', []):
        if ipset['Name'] == ipset_name:
            ipset_id = ipset['Id']
            break

    if not ipset_id:
        raise Exception(f"Không tìm thấy IP Set có tên {ipset_name}")

    get_ipset_res = waf_client.get_ip_set(Name=ipset_name, Scope=ipset_scope, Id=ipset_id)
    lock_token = get_ipset_res['LockToken']
    current_addresses = get_ipset_res['IPSet']['Addresses']

    # Gộp danh sách IP mới vào danh sách hiện có (loại bỏ trùng lặp)
    updated_addresses = list(set(current_addresses + offending_ips))

    # Cập nhật WAF IP Set
    waf_client.update_ip_set(
        Name=ipset_name,
        Scope=ipset_scope,
        Id=ipset_id,
        Addresses=updated_addresses,
        LockToken=lock_token
    )

    print(f"Đã cập nhật thành công WAF IP Set {ipset_name}. Danh sách IP mới: {updated_addresses}")

    return {
        'statusCode': 200,
        'body': f"Thành công block các IP: {offending_ips}"
    }
```

---

### Bước 2: Deploy mã nguồn

1. Sau khi dán đoạn code trên vào trình biên soạn, nhấn nút **Deploy** để lưu và áp dụng mã nguồn mới.

![Điều chỉnh Timeout Lambda](/images/5-Workshop/5.8-Create-IAM-Role-Lambda/general-configuration2.png)

---

## 4. Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- AWS Lambda Function `WAFAutoBlockFunction` được khởi tạo thành công với runtime Python 3.12 và gán đúng Execution Role `LambdaWAFAutoBlockRole`.
- Các biến môi trường `LOG_GROUP_NAME`, `IPSET_NAME`, `IPSET_SCOPE`, và `LIMIT_THRESHOLD` được thiết lập chính xác.
- Mã nguồn xử lý tự động phân tích CloudWatch Logs và gọi WAF API `UpdateIPSet` được deploy sẵn sàng để nhận signal kích hoạt tự động.
