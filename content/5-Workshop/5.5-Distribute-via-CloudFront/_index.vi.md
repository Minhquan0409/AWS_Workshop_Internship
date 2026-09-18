---
title: "Phân phối website qua Amazon CloudFront"
date: 2026-08-24
weight: 5
chapter: false
pre: "<b>5.5. </b>"
---

## Mục tiêu

Trong phần này, triển khai **Amazon CloudFront** để phân phối website **SkyLink Airline** được xây dựng bằng React và lưu trữ trên Amazon S3.

Sau khi hoàn thành, hệ thống sẽ chuyển từ mô hình truy cập trực tiếp vào S3 sang mô hình:

```text
User
  │
  │ HTTPS
  ▼
Amazon CloudFront
  │
  │ Origin Access Control (OAC)
  ▼
Amazon S3
  │
  └── React Frontend
  ```

## Mục tiêu chính:

* Sử dụng Amazon CloudFront làm CDN cho Frontend SkyLink Airline.
* Kết nối CloudFront với S3 Bucket đang lưu trữ website.
* Sử dụng Origin Access Control (OAC) để CloudFront truy cập S3.
* Giữ S3 Bucket ở chế độ private.
* Không cho phép người dùng truy cập trực tiếp vào S3.
* Cấu hình HTTPS cho website.
* Kiểm tra website thông qua CloudFront Domain.
* Kiểm tra khả năng truy cập và hoạt động của hệ thống.
* Thực hiện các bước bảo mật và clean-up tài nguyên.

## 1. Tổng quan
### 1.1. Vai trò của Amazon CloudFront

Amazon CloudFront là dịch vụ Content Delivery Network (CDN) của AWS, được sử dụng để phân phối nội dung website đến người dùng thông qua hệ thống Edge Location.

Trong project SkyLink Airline, Frontend React được build thành các file tĩnh như:
```text
HTML
CSS
JavaScript
Images
Fonts
SVG
```

Các file này được lưu trữ trên Amazon S3.

CloudFront đứng giữa người dùng và S3 để tiếp nhận request, cache nội dung và phân phối nội dung đến người dùng.

Mô hình triển khai:
```text
                    Internet
                        │
                        ▼
                ┌───────────────┐
                │   CloudFront  │
                │      CDN      │
                └───────┬───────┘
                        │
                        │ OAC
                        ▼
                ┌───────────────┐
                │      S3       │
                │    Private    │
                └───────┬───────┘
                        │
                        ▼
                React Frontend
```

### 1.2. Vì sao sử dụng CloudFront?

Nếu người dùng truy cập trực tiếp S3 Website Endpoint:
```text
User
  │
  ▼
S3 Website Endpoint
  │
  ▼
React Frontend
```
thì S3 phải cung cấp nội dung website trực tiếp cho Internet.

Trong kiến trúc hoàn thiện của SkyLink, S3 được giữ private và CloudFront được sử dụng làm lớp phân phối.
```text
User
  │
  ▼
CloudFront
  │
  ▼
Private S3
```

### 1.3. Kiến trúc trước khi triển khai CloudFront

Ở phần 5.4, Frontend SkyLink được lưu trữ trên Amazon S3.
```text
User
  │
  ▼
Amazon S3
  │
  ├── index.html
  ├── assets/
  └── images/
```

Trong quá trình kiểm tra Static Website Hosting, S3 Website Endpoint có thể được sử dụng để xác nhận website hoạt động.

Sau khi kiểm tra xong, Bucket được chuyển về chế độ private.

Khi truy cập trực tiếp S3 Website Endpoint:
```text
HTTP 403 Forbidden
AccessDenied
```
là kết quả mong đợi khi Bucket không còn public.

## 2. Nội dung thực hành
### 2.1. Khởi tạo Amazon CloudFront Distribution

### Bước 1: Truy cập dịch vụ Amazon CloudFront

1. Đăng nhập vào **AWS Management Console**.
2. Trên thanh tìm kiếm, nhập `CloudFront` và chọn dịch vụ **CloudFront**.
3. Tại giao diện CloudFront Dashboard, nhấn **Create distribution**.

![Truy cập CloudFront Console](/images/5-Workshop/5.5-Distribute-via-CloudFront/cloudfront-console.png)

---

### Bước 2: Cấu hình Origin Settings & Origin Access Control (OAC)

1. **Origin domain:** Bấm chọn S3 Bucket đã tạo ở bước 5.4 (ví dụ: `skylink-airline-frontend-2026.s3.ap-southeast-1.amazonaws.com`).
2. **Name:** Giữ nguyên tên gợi ý mặc định.
3. **Origin access:** Tích chọn **Origin access control settings (recommended)**.
4. Bấm chọn **Create new OAC** (nếu chưa có sẵn OAC) -> Giữ nguyên tên mặc định -> Nhấn **Create**.

![Cấu hình Origin và OAC](/images/5-Workshop/5.5-Distribute-via-CloudFront/origin-oac-config.png)

---

### Bước 3: Cấu hình Default Cache Behavior

1. **Viewer protocol policy:** Chọn **Redirect HTTP to HTTPS** để bắt buộc mã hóa toàn bộ lưu lượng truy cập.
2. **Allowed HTTP methods:** Giữ mặc định **GET, HEAD**.
3. **Cache key and origin requests:** Giữ mặc định **CachingOptimized**.

![Cấu hình Default Cache Behavior](/images/5-Workshop/5.5-Distribute-via-CloudFront/cache-behavior.png)

---

### Bước 4: Cấu hình Web Application Firewall (WAF) & Settings

1. **Web Application Firewall (WAF):** Tạm thời chọn **Do not enable security protections** (Chúng ta sẽ tích hợp AWS WAF chi tiết ở bước 5.6).
2. **Default root object:** Nhập `index.html`.
3. Kéo xuống cuối trang và nhấn **Create distribution**.

![Cấu hình Root Object và nhấn Create](/images/5-Workshop/5.5-Distribute-via-CloudFront/create-distribution.png)
![Cấu hình Root Object và nhấn Create](/images/5-Workshop/5.5-Distribute-via-CloudFront/create-distribution2.png)

### 2.2 Cập nhật S3 Bucket Policy với OAC

Sau khi khởi tạo CloudFront Distribution thành công, một thông báo màu vàng sẽ xuất hiện yêu cầu cập nhật S3 Bucket Policy để phân quyền cho OAC.

### Bước 1: Sao chép Bucket Policy từ CloudFront

1. Tại trang chi tiết của CloudFront Distribution vừa tạo, nhấn nút **Copy policy** trong hộp thông báo màu xanh dương.

![Copy S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/copy-bucket-policy.png)

---

### Bước 2: Dán Policy vào Amazon S3 Bucket

1. Quay lại dịch vụ **Amazon S3** và chọn S3 Bucket của bạn.
2. Chuyển sang tab **Permissions**.
3. Tại mục **Bucket policy**, nhấn **Edit**.
4. Dán đoạn Policy vừa copy từ CloudFront vào khung chỉnh sửa JSON.
5. Nhấn **Save changes**.

![Cập nhật S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/update-s3-policy.png)

![Cập nhật S3 Bucket Policy](/images/5-Workshop/5.5-Distribute-via-CloudFront/update-s3-policy2.png)

---

## 2.3 Kiểm tra kết quả phân phối (Distribution Domain Name)

### Bước 1: Lấy Domain Name của CloudFront

1. Quay lại giao diện **CloudFront Distributions**.
2. Tìm cột **Domain name** hoặc sao chép chuỗi **Distribution domain name** trong trang chi tiết (ví dụ: `d2n9euazwledbh.cloudfront.net`).

![Lấy CloudFront Domain Name](/images/5-Workshop/5.5-Distribute-via-CloudFront/get-domain-name.png)

---

### Bước 2: Truy cập website kiểm thử

1. Đợi trạng thái **Last modified** của Distribution chuyển từ `Deploying` sang mốc thời gian hoàn tất.
2. Mở thẻ trình duyệt mới và truy cập đường dẫn: `https://d2n9euazwledbh.cloudfront.net/`

![Kiểm tra truy cập CloudFront Domain](/images/5-Workshop/5.5-Distribute-via-CloudFront/test-website-access.png)

---

## 2.4 Kết quả mong đợi

Sau khi hoàn thành bài thực hành này:

- **CloudFront Distribution** được khởi tạo thành công và liên kết chính xác với S3 Origin.
- **S3 Bucket Policy** được cập nhật phân quyền OAC thành công, chặn hoàn toàn truy cập trực tiếp URL S3 và chỉ chấp nhận request từ CloudFront.
- Website giao diện `Skylink` truy cập mượt mà, an toàn qua giao thức HTTPS bằng tên miền CloudFront Domain Name.



