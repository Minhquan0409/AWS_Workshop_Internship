---

title: "Điều kiện chuẩn bị"
date: 2026-08-24
weight: 2
chapter: false
pre: "<b>5.2. </b>"
-------------------

## Mục tiêu

Trước khi triển khai hệ thống **SkyLink Airline** lên AWS, cần chuẩn bị đầy đủ môi trường phát triển, mã nguồn, tài khoản AWS và các công cụ cần thiết.

Mục tiêu của phần này là đảm bảo người thực hiện có thể:

* Chạy SkyLink thành công trên môi trường local.
* Kiểm tra Frontend React/Vite.
* Kiểm tra Backend Laravel.
* Kết nối Backend với MySQL.
* Cài đặt và cấu hình AWS CLI.
* Chuẩn bị các thông tin cần thiết cho quá trình triển khai.
* Đảm bảo tài khoản AWS có quyền cần thiết nhưng vẫn tuân thủ nguyên tắc Least Privilege.

---

## 1. Công cụ và tài nguyên cần chuẩn bị

### 1.1. Mã nguồn SkyLink Airline

Workshop sử dụng project **SkyLink Airline System** làm ứng dụng mẫu để triển khai lên AWS.

Cấu trúc project gồm hai thành phần chính:

```text
SkyLink-Airline-System
│
├── airline-frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── vite.config.*
│
└── airline-backend
    ├── app/
    ├── routes/
    ├── database/
    ├── public/
    ├── .env
    └── composer.json
```

> Cấu trúc thư mục thực tế có thể thay đổi tùy theo phiên bản mã nguồn được sử dụng.

**Source code:**

`https://github.com/quangg199/SkyLink-Airline-System`

---

### 1.2. Tài khoản AWS

Cần có một tài khoản AWS đang hoạt động.

Tài khoản này sẽ được sử dụng để tạo và quản lý các tài nguyên phục vụ Workshop.

Các dịch vụ AWS dự kiến sử dụng:

| Nhóm           | AWS Service               |
| -------------- | ------------------------- |
| Networking     | Amazon VPC                |
| Frontend       | Amazon S3                 |
| CDN            | Amazon CloudFront         |
| Security       | AWS WAF                   |
| Backend        | Amazon EC2                |
| Load Balancing | Application Load Balancer |
| Database       | Amazon RDS for MySQL      |
| Messaging      | Amazon SQS                |
| Serverless     | AWS Lambda                |
| Monitoring     | Amazon CloudWatch         |
| Notification   | Amazon SNS                |
| Identity       | AWS IAM                   |

---

### 1.3. AWS Region

Trong Workshop, nên sử dụng **một Region thống nhất** cho phần lớn các tài nguyên.

Ví dụ:

```text
Region: Asia Pacific (Singapore)
Region code: ap-southeast-1
```

Việc sử dụng cùng một Region giúp:

* Dễ quản lý tài nguyên.
* Giảm độ phức tạp khi cấu hình.
* Hạn chế latency giữa các dịch vụ.
* Dễ kiểm tra và cleanup tài nguyên.

> Nếu tài khoản AWS của bạn đang sử dụng Region khác, cần thay `ap-southeast-1` bằng Region thực tế trong toàn bộ Workshop.

![AWS Region](/images/5-Workshop/5.2-Prerequisite/aws-region.png?v=2)

---

### 1.4. Máy tính cá nhân

Máy tính dùng để thực hiện Workshop cần có kết nối Internet ổn định.

Môi trường Windows có thể sử dụng:

```text
Operating System:
Windows 10 / Windows 11

Terminal:
Git Bash / PowerShell / CMD

Browser:
Google Chrome / Microsoft Edge
```

---

### 1.5. Node.js và npm

Node.js được sử dụng để cài đặt dependency và build Frontend React/Vite.

Kiểm tra phiên bản:

```bash
node -v
npm -v
```

Ví dụ:

```text
Node.js: v20.x
npm: 10.x
```

Sau khi kiểm tra, chuyển đến thư mục Frontend:

```bash
cd airline-frontend
```

Cài đặt dependency:

```bash
npm install
```

Build Frontend:

```bash
npm run build
```

Nếu build thành công, thư mục production thường được tạo:

```text
dist/
```

**Code snippet:**

```bash
node -v
npm -v

cd airline-frontend
npm install
npm run dev
```
---

### 1.6. PHP, Composer và Laravel

Backend của SkyLink được xây dựng bằng Laravel.

Cần chuẩn bị:

```text
PHP
Composer
Laravel
```

Kiểm tra:

```bash
php -v
composer -V
```

Chuyển đến thư mục Backend:

```bash
cd airline-backend
```

Cài đặt dependency:

```bash
composer install
```

Kiểm tra Laravel:

```bash
php artisan --version
```

**Code snippet:**

```bash
php -v
composer -V
php artisan --version
```
---

### 1.7. MySQL

SkyLink sử dụng MySQL cho cơ sở dữ liệu.

Trong môi trường local, có thể sử dụng MySQL thông qua XAMPP hoặc một MySQL Server riêng.

Ví dụ cấu hình:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

> `DB_PORT` và các thông tin database phải được thay đổi theo môi trường local thực tế.

Kiểm tra kết nối database bằng Laravel:

```bash
php artisan migrate:status
```

Nếu project đã có database/schema hoàn chỉnh, có thể sử dụng database hiện tại thay vì chạy migration lại.

---

### 1.8. AWS CLI

AWS CLI được sử dụng để quản lý AWS resources thông qua terminal.

Kiểm tra:

```bash
aws --version
```

Ví dụ:

```text
aws-cli/2.x.x
```

Sau khi cài đặt, cấu hình AWS CLI:

```bash
aws configure
```

Nhập các thông tin:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Ví dụ:

```text
Default region name: ap-southeast-1
Default output format: json
```

> Không đưa Access Key hoặc Secret Access Key vào GitHub, file Markdown, screenshot hoặc `.env` của project.

Kiểm tra cấu hình:

```bash
aws sts get-caller-identity
```

Lệnh này trả về thông tin identity đang được AWS CLI sử dụng.

**Code snippet:**

```bash
aws --version
aws configure
aws sts get-caller-identity
```

---

### 1.9. Git

Git được sử dụng để quản lý source code và version của project.

Kiểm tra:

```bash
git --version
```

Clone project:

```bash
git clone https://github.com/quangg199/SkyLink-Airline-System.git
```

Sau đó kiểm tra:

```bash
cd SkyLink-Airline-System
git status
```

**Code snippet:**

```bash
git clone https://github.com/quangg199/SkyLink-Airline-System.git

cd SkyLink-Airline-System

git status
```

---

### 1.10. IAM

Tài khoản triển khai cần có quyền truy cập các dịch vụ AWS được sử dụng trong Workshop.

Trong quá trình triển khai, IAM được sử dụng để:

* Xác thực người dùng.
* Cấp quyền cho AWS CLI.
* Cấp IAM Role cho EC2.
* Cấp quyền cho Lambda.
* Truy cập S3.
* Ghi log vào CloudWatch.
* Tương tác với SQS và SNS.

Nguyên tắc được áp dụng:

> **Grant only the permissions required by each component.**

Không nên sử dụng Access Key của tài khoản Root cho AWS CLI.

---

## 2. Các bước chuẩn bị chi tiết

### Step 1 – Clone source code

Clone project SkyLink:

```bash
git clone https://github.com/quangg199/SkyLink-Airline-System.git
```

Di chuyển vào thư mục project:

```bash
cd SkyLink-Airline-System
```

Kiểm tra:

```bash
git status
```

**Kết quả mong đợi:**

```text
On branch ...
nothing to commit, working tree clean
```

---

### Step 2 – Kiểm tra Frontend

Di chuyển vào Frontend:

```bash
cd airline-frontend
```

Cài đặt package:

```bash
npm install
```

Build:

```bash
npm run build
```

Kết quả:

```text
airline-frontend/
└── dist/
    ├── assets/
    ├── index.html
    └── ...
```

Thư mục `dist` sẽ được sử dụng ở bước triển khai Frontend lên Amazon S3.

---

### Step 3 – Kiểm tra Backend

Di chuyển vào Backend:

```bash
cd ../airline-backend
```

Cài đặt dependency:

```bash
composer install
```

Kiểm tra Laravel:

```bash
php artisan --version
```

Nếu project sử dụng file `.env.example`, tạo `.env`:

```bash
cp .env.example .env
```

Trên Windows CMD có thể sử dụng:

```cmd
copy .env.example .env
```

Sau đó tạo application key:

```bash
php artisan key:generate
```

---

### Step 4 – Cấu hình database local

Cập nhật `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=airline_db
DB_USERNAME=root
DB_PASSWORD=
```

Kiểm tra:

```bash
php artisan migrate:status
```

Nếu database hoạt động bình thường, Backend đã có thể kết nối MySQL.

---

### Step 5 – Chạy Backend local

Khởi động Laravel:

```bash
php artisan serve
```

Backend sẽ chạy tại:

```text
http://127.0.0.1:8000
```

![Laravel Backend Running](/images/5-Workshop/5.2-Prerequisite/backend-running.png?v=2)

---

### Step 6 – Chạy Frontend local

Mở terminal mới.

Di chuyển đến Frontend:

```bash
cd airline-frontend
```

Chạy:

```bash
npm run dev
```

Vite sẽ cung cấp URL local, ví dụ:

```text
http://localhost:5173
```
![SkyLink Frontend Running](/images/5-Workshop/5.2-Prerequisite/frontend-build.png?v=2)

---

### Step 7 – Kiểm tra kết nối Frontend và Backend

Frontend phải được cấu hình để gọi đúng Backend API.

Ví dụ:

```env
VITE_API_URL=http://127.0.0.1:8000
```

> Tên biến môi trường thực tế cần sử dụng đúng theo source code của SkyLink.

Kiểm tra trên trình duyệt:

```text
Frontend
    |
    v
Backend API
    |
    v
MySQL
```

Thực hiện một chức năng đơn giản, ví dụ:

```text
Login
Search Flight
View Flight
```

Nếu request được xử lý thành công, môi trường local đã sẵn sàng.

---

### Step 8 – Kiểm tra AWS CLI

Chạy:

```bash
aws sts get-caller-identity
```

Kết quả cần trả về thông tin identity của AWS account.

Ví dụ:

```json
{
    "UserId": "********",
    "Account": "************",
    "Arn": "arn:aws:iam::************:user/..."
}
```

Không chụp hoặc công khai thông tin nhạy cảm nếu screenshot được đưa lên website công khai.

---

### Step 9 – Kiểm tra AWS Region

Thiết lập Region:

```bash
aws configure set region ap-southeast-1
```

Kiểm tra:

```bash
aws configure get region
```

Kết quả:

```text
ap-southeast-1
```

---

### Step 10 – Kiểm tra tài nguyên AWS

Trước khi bắt đầu triển khai, truy cập AWS Console và kiểm tra Region.

Các dịch vụ sẽ được sử dụng trong các bước tiếp theo:

```text
IAM
VPC
EC2
S3
CloudFront
RDS
Elastic Load Balancing
SQS
Lambda
WAF
CloudWatch
SNS
```
---

### Step 11 – Chuẩn bị file cấu hình

Các file quan trọng cần chuẩn bị:

```text
SkyLink-Airline-System/
│
├── airline-frontend/
│   ├── package.json
│   ├── vite.config.*
│   └── .env
│
└── airline-backend/
    ├── composer.json
    ├── artisan
    ├── .env
    └── database/
```

### Lưu ý về bảo mật

Không commit các file chứa secret:

```text
.env
*.pem
AWS Access Key
AWS Secret Access Key
Database Password
JWT Secret
Application Secret
```

`.gitignore` cần đảm bảo các file nhạy cảm không được commit.

Ví dụ:

```gitignore
.env
.env.*
!.env.example

*.pem
*.key
```

---

## 3. Kết quả mong đợi

Sau khi hoàn thành phần chuẩn bị, môi trường triển khai phải đáp ứng các điều kiện sau:

### 3.1. Source code

Project SkyLink đã được clone và kiểm tra:

```text
SkyLink-Airline-System
        |
        +---- airline-frontend
        |
        +---- airline-backend
```

### 3.2. Frontend

Frontend có thể:

```text
npm install
       ↓
npm run build
       ↓
dist/
```

và chạy được trên môi trường local.

### 3.3. Backend

Laravel Backend có thể:

```text
composer install
       ↓
.env configured
       ↓
Database connected
       ↓
php artisan serve
```

### 3.4. Database

MySQL hoạt động và Backend có thể kết nối tới database `airline_db`.

### 3.5. AWS CLI

AWS CLI hoạt động:

```bash
aws sts get-caller-identity
```

và trả về đúng AWS account.

### 3.6. AWS Environment

Region triển khai đã được xác định:

```text
ap-southeast-1
```

Các dịch vụ cần thiết đã được xác định và sẵn sàng triển khai.

### 3.7. Security

Không có AWS credentials hoặc secret quan trọng được hard-code vào source code.

Môi trường đã sẵn sàng để chuyển sang bước tiếp theo:

```text
Local Environment
       ↓
Source Code Ready
       ↓
AWS CLI Ready
       ↓
AWS Account Ready
       ↓
      5.3
AWS Infrastructure Deployment
```
