# Internship Report & AWS Security Workshop

Website báo cáo thực tập của **Trương Minh Quân**, được xây dựng bằng [Hugo](https://gohugo.io/) và sử dụng theme `hugo-theme-learn`.

Nội dung được trình bày bằng tiếng Anh và tiếng Việt, bao gồm nhật ký thực tập, đề xuất, bài viết, sự kiện, workshop AWS, tự đánh giá và phản hồi.

## Nội dung chính

- **Worklog:** Nhật ký công việc theo từng tuần.
- **Proposal:** Đề xuất và định hướng thực tập.
- **Blogs Posted:** Các bài blog đã đăng.
- **Events Participated:** Các sự kiện đã tham gia.
- **Workshop:** Hướng dẫn triển khai giải pháp bảo vệ website tĩnh trên AWS.
- **Self-evaluation:** Tự đánh giá kết quả thực tập.
- **Sharing and Feedback:** Chia sẻ và phản hồi.

## Workshop AWS

Workshop chính xây dựng một kiến trúc serverless để phát hiện và tự động chặn các địa chỉ IP có hành vi truy cập bất thường vào website tĩnh.

Các dịch vụ AWS được sử dụng:

- Amazon S3 để lưu trữ website tĩnh.
- Amazon CloudFront để phân phối nội dung qua CDN.
- AWS WAF để lọc và chặn lưu lượng truy cập.
- Amazon CloudWatch để theo dõi chỉ số và kích hoạt cảnh báo.
- Amazon SNS để gửi thông báo.
- AWS Lambda để tự động cập nhật danh sách IP bị chặn.
- AWS IAM để quản lý quyền truy cập giữa các dịch vụ.

Quy trình workshop bao gồm chuẩn bị project, triển khai S3 và CloudFront, cấu hình WAF và logging, thiết lập SNS, IAM, Lambda, CloudWatch Alarm, kiểm thử và dọn dẹp tài nguyên.

## Yêu cầu

- [Hugo Extended](https://gohugo.io/installation/) phiên bản phù hợp.
- Git.
- Trình duyệt web hiện đại.

Kiểm tra Hugo đã được cài đặt:

```bash
hugo version
```

## Chạy website ở môi trường local

Clone repository và chuyển vào thư mục project:

```bash
git clone https://github.com/minhquan0409/AWS_Workshop_Internship.git
cd AWS_Workshop_Internship
```

Khởi động Hugo development server:

```bash
hugo server -D --baseURL="http://localhost:1313/AWS_Workshop_Internship/"
```

Mở địa chỉ được Hugo hiển thị trong terminal, thường là `http://localhost:1313/AWS_Workshop_Internship/`.

## Build website

Tạo bản build production trong thư mục `public/`:

```bash
hugo --minify
```

Thư mục `public/` là output HTML tĩnh được sinh từ nội dung trong `content/`, layout và cấu hình Hugo.

## Cấu trúc thư mục

```text
.
├── archetypes/      # Template tạo nội dung mới
├── content/         # Nội dung báo cáo bằng tiếng Anh và tiếng Việt
├── layouts/         # Partial và shortcode tùy chỉnh
├── static/          # Tài nguyên tĩnh do project quản lý
├── themes/          # Hugo theme
├── config.toml      # Cấu hình site, ngôn ngữ và menu
└── public/          # Website đã build, được Hugo sinh ra
```

## Ngôn ngữ

Website hỗ trợ hai ngôn ngữ:

- English: `/en/`
- Tiếng Việt: `/vi/`

Ngôn ngữ mặc định và thông tin deploy được cấu hình trong [`config.toml`](config.toml).

## Thông tin thực tập

- **Sinh viên:** Trương Minh Quân
- **Trường:** Đại học Xây dựng Hà Nội
- **Chuyên ngành:** Công nghệ thông tin
- **Lớp:** 67PM2
- **Đơn vị thực tập:** Amazon Web Services Vietnam Company Limited
- **Chương trình:** Workforce Bootcamp - First Cloud AI Journey
- **Thời gian:** 18/07/2026 - 27/09/2026

## Tác giả

**Trương Minh Quân**  
Email: [truongminhquan0904@gmail.com](mailto:truongminhquan0904@gmail.com)
