---

title: "Lưu trữ website trên Amazon S3"
date: 2026-08-24
weight: 4
chapter: false
pre: "<b>5.4. </b>"
-------------------

## 1. Tổng quan

Trong phần này, Frontend của hệ thống **SkyLink Airline** được build thành các file tĩnh bằng React/Vite và lưu trữ trên **Amazon S3**.

Amazon S3 được sử dụng để lưu trữ các file:

```text
HTML
CSS
JavaScript
Images
Static Assets
```

Quy trình triển khai:

```text
SkyLink Frontend
      ↓
 npm run build
      ↓
    dist/
      ↓
 Amazon S3
      ↓
Static Website
```

---

## 2. Quy trình triển khai

Quy trình triển khai Frontend SkyLink lên Amazon S3 gồm:

1. Build Frontend React/Vite.
2. Tạo S3 Bucket.
3. Cấu hình Bucket cho website.
4. Cấu hình quyền truy cập phù hợp.
5. Upload thư mục `dist/`.
6. Kiểm tra website trên trình duyệt.

---

## 3. Nội dung thực hành

### 5.4.1. Các bước khởi tạo S3 Bucket

#### Bước 1: Build Frontend

Mở Terminal tại thư mục:

```bash
cd airline-frontend
```

Cài đặt dependencies nếu chưa thực hiện:

```bash
npm install
```

Build project:

```bash
npm run build
```

Sau khi build thành công, thư mục `dist/` được tạo:

```text
airline-frontend/
└── dist/
    ├── assets/
    ├── index.html
    └── ...
```

**Screenshot:**

```text
static/images/workshop/5-4/frontend-build.png
```

![Frontend Build](/images/5-Workshop/5.4-frontend-s3-cloudfront/frontend-build.png?v=2)

---

#### Bước 2: Mở Amazon S3

Truy cập **AWS Management Console** → tìm kiếm **S3** → chọn **Amazon S3**.

Chọn:

**Create bucket**

![Amazon S3 Console](/images/5-Workshop/5.4-frontend-s3-cloudfront/s3-console.png?v=2)

---

#### Bước 3: Đặt tên Bucket

Nhập tên Bucket duy nhất trên toàn bộ AWS.

Ví dụ:

```text
skylink-airline-frontend-2026
```

Chọn Region đã sử dụng trong Workshop, ví dụ:

```text
ap-southeast-1
```

> Tên Bucket phải là duy nhất và không chứa khoảng trắng.

![Create S3 Bucket](/images/5-Workshop/5.4-frontend-s3-cloudfront/create-bucket.png?v=2)

---

#### Bước 4: Cấu hình quyền truy cập

Trong phần **Block Public Access settings**, thực hiện cấu hình theo mục tiêu triển khai của Workshop.

Để phục vụ phần thực hành **S3 Static Website Hosting**, có thể tắt:

**Block all public access**

AWS sẽ hiển thị cảnh báo xác nhận. Đọc kỹ cảnh báo trước khi tiếp tục.

> Trong kiến trúc hoàn thiện của SkyLink, website nên được truy cập thông qua **Amazon CloudFront** và hạn chế public access trực tiếp tới S3. Phần này sẽ được cải thiện ở bước CloudFront.

![S3 Public Access](/images/5-Workshop/5.4-frontend-s3-cloudfront/public-access.png?v=2)

---

#### Bước 5: Tạo Bucket

Giữ các thiết lập mặc định khác và chọn:

**Create bucket**

Sau khi tạo thành công, Bucket xuất hiện trong danh sách S3.

**Checkpoint:**

```text
Bucket: skylink-airline-frontend-2026
Region: ap-southeast-1
Status: Created
```

---

#### Bước 6: Upload Frontend

Mở Bucket vừa tạo → chọn **Upload**.

Chọn toàn bộ nội dung bên trong thư mục:

```text
airline-frontend/dist/
```

Sau đó chọn:

**Upload**

Cấu trúc trong Bucket:

```text
S3 Bucket
├── assets/
├── index.html
└── ...
```

![Upload Frontend](/images/5-Workshop/5.4-frontend-s3-cloudfront/upload-files.png?v=2)

---

#### Bước 7: Cấu hình Static Website Hosting

Vào:

**Bucket → Properties → Static website hosting**

Chọn:

```text
Enable
```

Thiết lập:

```text
Index document:
index.html
```

Nếu ứng dụng cần xử lý lỗi SPA:

```text
Error document:
index.html
```

Sau đó chọn **Save changes**.

![Static Website Hosting](/images/5-Workshop/5.4-frontend-s3-cloudfront/static-website-hosting.png?v=2)

---

#### Bước 8: Kiểm tra Website

Sau khi bật Static Website Hosting, S3 cung cấp **Bucket website endpoint**.

Mở endpoint trên trình duyệt.

Kết quả mong đợi:

```text
SkyLink Airline
       ↓
React Frontend
       ↓
Website displayed successfully
```

![SkyLink S3 Website](/images/5-Workshop/5.4-frontend-s3-cloudfront/skylink-s3-website.png?v=2)

---

#### Bước 9: Kiểm tra các file đã triển khai

Trong S3 Bucket cần có các file được tạo từ quá trình build:

```text
index.html
assets/
    ├── *.js
    ├── *.css
    └── images/
```

Không upload thư mục source `src/` lên S3.

Chỉ upload **nội dung của thư mục `dist/`**.

---

## 4. Kết quả mong đợi

Sau khi hoàn thành, Frontend SkyLink đã được lưu trữ trên Amazon S3.

Kiến trúc ở bước này:

```text
User
  ↓
S3 Static Website
  ↓
React/Vite Frontend
```

Kiểm tra thành công khi:

* S3 Bucket được tạo thành công.
* Frontend được build thành công.
* Các file trong `dist/` được upload lên S3.
* Static Website Hosting được bật.
* `index.html` được cấu hình làm trang chính.
* SkyLink Frontend có thể truy cập thông qua S3 Website Endpoint.

> **Lưu ý:** S3 trong bước này được sử dụng để giúp người học hiểu cách triển khai static website. Trong kiến trúc hoàn chỉnh, **Amazon CloudFront** sẽ được sử dụng phía trước S3 để phân phối nội dung và cải thiện HTTPS, hiệu năng và bảo mật.
