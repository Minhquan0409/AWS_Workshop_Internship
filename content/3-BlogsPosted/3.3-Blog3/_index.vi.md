---
title: "Blog 3"
date: 2026-08-31
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Ngăn chặn rò rĩ dữ liệu trên AWS bằng Egress Controls cho Cloud Workloads

### 1. Giới thiệu:

Khi xây dựng hệ thống trên AWS, hầu hết các doanh nghiệp đều tập trung vào việc bảo vệ lưu lượng truy cập đi vào hệ thống (Ingress Traffic) thông qua các cơ chế như Security Group, Web Application Firewall (WAF) hoặc Indentity and Access Management (IAM).

Tuy nhiên, theo AWS Security Group, một trong những rủi ro thường bị bỏ qua là lưu lượng truy cập đi ra khỏi hệ thống (Egress Traffic). Nếu không được kiểm soát, các máy chủ hoặc ứng dụng bị xâm nhập có thể gửi dữ liệu nhạy cảm ra bên ngoài mà tổ chức không hề hay biết. Đây được gọi là Data Exfiltration (Rò rỉ dữ liệu).

Trong bối cảnh các ứng dụng AI và Agentic AI ngày càng phổ biến, nguy cơ này càng trở nên nghiêm trọng hơn khi các AI Agent có khả năng truy cập dữ liệu, gọi API và tương tác với nhiều hệ thống bên ngoài. AWS đã giới thiệu một kiến trúc bảo mật nhiều lớp nhằm giúp doanh nghiệp phát hiện và ngăn chặn các hành vi rò rỉ dữ liệu trên môi trường đám mây.

### 2. Data Exfiltration là gì?

Data Exfiltration là hành vi đưa dữ liệu ra khỏi hệ thống mà không được phép.

Ví dụ:
* Một máy chủ EC2 bị khai thác lỗ hỏng bảo mật.
* Hacker cài đặt mã độc và gửi dữ liệu khách hàng ra máy chủ bên ngoài.
* Một AI Agent bị tấn công Prompt Injection và bị điều khiển để gửi dữ liệu nhạy cảm tới một dịch vụ bên ngoài.

Nếu không có cơ chế kiểm soát lưu lượng đi ra, các hành vi này rất khó bị phát hiện trong giai đoạn đầu.

### 3. Tại sao Egress Control lại quan trọng?

Nhiều hệ thống chỉ tập trung vào việc ngăn chặn truy cập trái phép từ bên ngoài. Tuy nhiên, sau khi kẻ tấn công đã xâm nhập được vào hệ thống, mục tiêu tiếp theo thường là:

* Đánh cắp dữ liệu.
* Thiết lập kết nối điều khiển từ xa (Command and Control).
* Tải thêm mã độc.
* Truyền dữ liệu ra ngoài tổ chức.

AWS nhấn mạnh rằng việc kiểm soát lưu lượng đi ra là một lớp phòng thủ quan trọng giúp giảm thiểu thiệt hại ngay cả khi hệ thống đã bị xâm nhập.

### 4. Các dịch vụ AWS hỗ trợ kiểm soát Egress Traffic

### 4.1. AWS Network Firewall

AWS Network Firewall là dịch vụ tường lửa được quản lý bởi AWS, cho phép kiểm tra và lọc lưu lượng mạng ở nhiều tầng khác nhau. Một số khả năng nổi bật:

* Chặn các tên miền không được phép.
* Kiểm soát địa chỉ IP và cổng kết nối.
* Phát hiện hành vi bất thường thông qua IDS/ IPS.
* Chặn truy cập tới các khu vực địa lý không mong muốn.
* Kiểm tra lưu lượng HTTPS thông qua TLS Inspection.
* Tích hợp Threat Intelligence để nhận diện các địa chỉ độc hại.

Thông qua Network Firewall, doanh nghiệp có thể xây dựng mô hình “chỉ cho phép những kết nối cần thiết” thay vì mở toàn bộ truy cập Internet ho máy chủ.

### 4.2. Amazon Route 53 Resolver DNS Firewall

Một kỹ thuật phổ biến của hacker là sử dụng DNS Tunneling để truyền dữ liệu ra ngoài thông qua các truy vấn DNS. Route 53 Resolver DNS Firewall giúp:

* Chặn truy cập tới các tên miền độc hại.
* Áp dụng danh sách cho phép (Allow List).
* Phát hiện Domain Generation Algobrithm (DGA).
* Phát hiện DNS Tunneling bằng AI và Machine Learning.

Nhờ đó tổ chức có thể ngăn chặn nhiều hình thức truyền dữ liệu bí mật thông qua DNS.

### 4.3. Data Perimeter

AWS giới thiệu khái niệm Data Perimeter nhằm bảo vệ dữ liệu ở cấp độ API. Các thành phần chính gồm:

* Service Control Policies (SCPs).
* Resource Control Policies (RCPs).
* IAM Policies.
* Resource Policies.
* VPC Endpoint Policies.

Mục tiêu là đảm bảo chỉ những người dùng, tài nguyên và mạng đáng tin cậy mới có thể truy cập dữ liệu của tổ chức.

### 5. Các dịch vụ phát hiện rò rỉ

### 5.1. Amazon GuardDuty

Amazon GuardDuty sử dụng Machine Learning và Threat Intelligence để phát hiện:

* DNS Data Exfiltration.
* Kết nối tới các tên miền độc hại.
* Hành vi Command và Control
* Truy cập bất thường tới Amazon S3.
* Hoạt động đáng ngờ từ các địa chỉ IP độc hại.

### 5.2. AWS Security Hub

AWS Security Hub tổng hợp các cảnh báo từ nhiều dịch vụ khác nhau như:

* GuardDuty.
* IAM Access Analyzer.
* AWS Config.
* AWS Firewall Manager.

Giúp đội ngũ bảo mật có cái nhìn tập trung về tình trạng an ninh của toàn bộ hệ thống.

### 5.3. IAM Access Analyzer

IAM Access Analyzer hỗ trợ phát hiện:

* Tài nguyên công khai ngoài ý muốn.
* Quyền truy cập quá mức.
* Chính sách chia sẻ dữ liệu không an toàn.

Điều này giúp giảm nguy cơ dữ liệu bị truy cập từ bên ngoài tổ chức.

### 6. Liên hệ với các hệ thống AI hiện nay

Một điểm thú vị trong bài viết là AWS không chỉ đề cập đến các ứng dụng truyền thống mà còn mở rộng sang Agentic AI. Các AI Agent hiện nay có thể:

* Truy cập cơ sở dữ liệu.
* Gọi API bên ngoài.
* Thực thi mã nguồn.
* Tự động thực hiện các tác vụ phức tạp.

Nếu bị khai thác qua Prompt Injection hoặc Goal Hijacking, AI Agent hoàn toàn có thể trở thành công cụ rò rỉ dữ liệu cho kẻ tấn công. Vì vậy, AWS khuyến nghị áp dụng các cơ chế Egress Control tương tự cho cả ứng dụng AI và ứng dụng truyền thống.

### 7. Bài học rút ra

Sau khi tìm hiểu bài viết, mình nhận thấy rằng bảo mật không chỉ ngăn người lạ truy cập vào hệ thống mà còn phải kiểm soát dữ liệu đi ra khỏi hệ thống. Một chiến lược bảo mật hiệu quả nên bao gồm:

* Kiểm soát lưu lượng mạng đi ra.
* Giới hạn các tên miền và địa chỉ IP được phép truy cập.
* Áp dụng nguyên tắc Least Privilege.
* Giám sát liên tục bằng GuardDuty và Security Hub.
* Thiết lập Data Perimeter để bảo vệ dữ liệu ở cấp độ API.

Đặc biệt trong thời đại AI việc kiểm soát hành vi của AI Agent cũng quan trọng không kém việc bảo vệ máy chủ hoặc ứng dụng truyền thống.

### 8. Kết luận

Bài viết của AWS cho thấy Data Exfiltration là một trong những rủi ro bảo mật quan trọng nhưng thường bị xem nhẹ trong môi trường đám mây. Bằng cách kết hợp AWS Network Firewall, Route 53 Resolver DNS Firewall, GuardDuty, Security Hub và các Data Perimeter Controls doanh nghiệp có thể xây dựng một kiến trúc phòng thủ nhiều lớp nhằm ngăn chặn và phát hiện các hành vi rủi ro dữ liệu.

Đây là một chủ đề rất đáng quan tâm đối với những ai đang học AWS Security, DevSecOps hoặc xây dựng các hệ thống AI trên nền tảng AWS hiện nay.

![Egress Controls](/images/3-BlogsPosted/egress-under-control1.png)

---

### Tài liệu tham khảo

*   [AWS egress controls for cloud workloads](https://aws.amazon.com/vi/blogs/security/prevent-data-exfiltration-aws-egress-controls-for-cloud-workloads/)
