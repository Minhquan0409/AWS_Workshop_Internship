---

title: "Chuẩn bị dự án"
date: 2026-08-24
weight: 3
chapter: false
pre: "<b>5.3. </b>"
-------------------

### Mục tiêu

Chuẩn bị mã nguồn **SkyLink Airline** để sẵn sàng triển khai Frontend và Backend lên AWS.

Trong phần này, thực hiện kiểm tra cấu trúc project, cài đặt dependency, cấu hình biến môi trường và build Frontend.

---

## 1. Kiểm tra cấu trúc dự án

Sau khi clone source code, kiểm tra cấu trúc:

```text
SkyLink-Airline-System/
├── airline-frontend/
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.*
│
└── airline-backend/
    ├── app/
    ├── routes/
    ├── database/
    ├── public/
    ├── artisan
    ├── composer.json
    └── .env
```

**Source code:**

`https://github.com/quangg199/SkyLink-Airline-System`

---

## 2. Chuẩn bị Frontend

### Bước 1: Cài đặt dependencies

Di chuyển vào thư mục Frontend:

```bash
cd airline-frontend
npm install
```

### Bước 2: Cấu hình API

Tạo hoặc cập nhật file `.env`:

```env
VITE_API_URL=http://127.0.0.1:8000
```

> Tên biến môi trường cần được điều chỉnh theo cấu hình thực tế của source code SkyLink.

### Bước 3: Build Frontend

Chạy:

```bash
npm run build
```

Sau khi build thành công, thư mục `dist/` được tạo:

```text
airline-frontend/
└── dist/
    ├── assets/
    └── index.html
```

Thư mục `dist/` sẽ được sử dụng để triển khai lên **Amazon S3**.

**Code snippet:**

```bash
cd airline-frontend
npm install
npm run build
```
---

## 3. Chuẩn bị Backend

### Bước 1: Cài đặt dependencies

Di chuyển vào Backend:

```bash
cd ../airline-backend
```

Cài đặt Composer dependencies:

```bash
composer install
```

### Bước 2: Cấu hình môi trường

Nếu chưa có `.env`, tạo từ `.env.example`:

```bash
cp .env.example .env
```

Sau đó tạo Laravel application key:

```bash
php artisan key:generate
```

### Bước 3: Cấu hình Database

Cập nhật các thông tin database trong `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

> Các thông số trên chỉ là cấu hình môi trường local. Khi triển khai lên AWS, `DB_HOST` sẽ được thay bằng endpoint của Amazon RDS.

Kiểm tra kết nối:

```bash
php artisan migrate:status
```

---

## 4. Kiểm thử dự án trên Local

Trước khi triển khai lên AWS, cần đảm bảo hệ thống hoạt động ổn định trên local.

### Chạy Backend

```bash
php artisan serve
```

Backend:

```text
http://127.0.0.1:8000
```

### Chạy Frontend

Mở terminal mới:

```bash
cd airline-frontend
npm run dev
```

Frontend:

```text
http://localhost:5173
```

### Kiểm tra chức năng

Thực hiện một số thao tác:

```text
Login
   ↓
Search Flight
   ↓
View Flight
   ↓
Booking
```
---

## 5. Chuẩn bị cho AWS Deployment

Sau khi kiểm thử local thành công, chuẩn bị các thành phần cho quá trình triển khai:

```text
Frontend
   ↓
npm run build
   ↓
dist/
   ↓
Amazon S3
```

```text
Backend
   ↓
Laravel Application
   ↓
Docker / EC2
   ↓
Application Load Balancer
```

```text
Database
   ↓
MySQL
   ↓
Amazon RDS
```

Các file deployment có thể được đặt trong thư mục:

```text
deployment/
├── Dockerfile
├── docker-compose.yml
├── scripts/
└── cloudformation/
```

Nếu sử dụng CloudFormation hoặc script tự động, các file tương ứng sẽ được cung cấp trong phần triển khai.

**File download:**

```text
static/downloads/workshop/
```

---

## 6. Kết quả mong đợi

Sau khi hoàn thành phần chuẩn bị dự án:

* Source code SkyLink được kiểm tra đầy đủ.
* Frontend React/Vite cài đặt dependencies thành công.
* Frontend build thành công và tạo thư mục `dist/`.
* Backend Laravel cài đặt Composer dependencies thành công.
* File `.env` được cấu hình phù hợp.
* Backend kết nối được với MySQL.
* Frontend và Backend hoạt động trên môi trường local.
* Project sẵn sàng để triển khai lên AWS.

```text
SkyLink Source Code
        ↓
Frontend Ready
        ↓
Backend Ready
        ↓
Database Ready
        ↓
Local Test Passed
        ↓
   AWS Deployment
```
