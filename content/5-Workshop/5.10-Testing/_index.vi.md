---
title: "Kiểm thử hệ thống"
date: 2026-08-24
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Mục tiêu

Sau khi hoàn tất cấu hình toàn bộ hạ tầng (CloudFront, AWS WAF, CloudWatch Logs, Metric Filter, CloudWatch Alarm, AWS Lambda và SNS Topic), bước kiểm thử này nhằm xác minh tính đúng đắn và khả năng tự động phản ứng của hệ thống đối với đợt tấn công HTTP Flood.

---

## Các đường link về project

- **Website Demo:** [https://d2n9euazwledbh.cloudfront.net/](https://d2n9euazwledbh.cloudfront.net/)
- **GitHub Repository:** [https://github.com/quangg199/SkyLink-Airline-System](https://github.com/quangg199/SkyLink-Airline-System)

---

### Kịch bản kiểm thử

- **Mục tiêu:** Mô phỏng tấn công gửi dồn dập vượt quá **100 requests trong 5 phút** từ máy kiểm thử đến địa chỉ CloudFront Distribution để kích hoạt chuỗi xử lý tự động.
- **Kỳ vọng đầu ra:**
  1. CloudWatch Log Group `aws-waf-logs-cloudfront` ghi nhận đầy đủ log truy cập.
  2. CloudWatch Alarm `WAFHighRequestRateAlarm` chuyển sang trạng thái **`IN ALARM`**.
  3. Hàm AWS Lambda `WAFAutoBlockFunction` tự động kích hoạt, bóc tách chính xác địa chỉ IP (IPv4/IPv6) vi phạm.
  4. Địa chỉ IP vi phạm được đẩy vào WAF IP Set (`AutoBlockedIPSetV6` hoặc `AutoBlockedIPSet`).
  5. AWS WAF áp dụng quy tắc chặn, trả về mã lỗi **`403 Forbidden`** khi IP đó tiếp tục truy cập.
  6. Hệ thống gửi email thông báo tự động qua SNS với tiêu đề `[AWS WAF AUTO-BLOCK] Đã chặn thành công IP vi phạm` chứa thông tin chi tiết của IP.

---

### Các bước thực hiện kiểm thử

#### Bước 1: Giả lập lưu lượng truy cập lớn (HTTP Flood Simulation)

Mở **Command Prompt (CMD)** hoặc **PowerShell** trên máy kiểm thử và chạy vòng lặp gửi 200 HTTP requests tới CloudFront:

```cmd
for /L %i in (1,1,200) do @curl -s -o nul -w "Request %i - Status: %{http_code}\n" https://d2n9euazwledbh.cloudfront.net/
```

> **Ghi chú:** Quan sát kết quả hiển thị trên màn hình, ban đầu sẽ là các mã HTTP `200` thành công.

![Kết quả chạy vòng lặp gửi 200 HTTP requests bằng CMD/PowerShell](/images/5-Workshop/5.10-Testing/buoc1-http-flood-simulation.png)

#### Bước 2: Kiểm tra Log và CloudWatch Alarm

**CloudWatch Logs:**

- Truy cập **CloudWatch → Log groups → aws-waf-logs-cloudfront**.
- Kiểm tra các Log Stream mới nhất để đảm bảo các request gửi từ `curl` đã được WAF ghi nhận.

![Log Group aws-waf-logs-cloudfront với các Log Stream mới nhất](/images/5-Workshop/5.10-Testing/buoc2-cloudwatch-log-group.png)

**CloudWatch Alarm:**

- Truy cập **CloudWatch → Alarms → All alarms**.
- Quan sát báo động `WAFHighRequestRateAlarm`. Sau 1–3 phút, Metric `WAFRequestCount` sẽ vượt ngưỡng 100 và Alarm chuyển sang trạng thái **In alarm**.

![CloudWatch Alarm WAFHighRequestRateAlarm chuyển trạng thái In alarm](/images/5-Workshop/5.10-Testing/buoc2-cloudwatch-alarm-in-alarm.png)

#### Bước 3: Kiểm tra nhật ký thực thi AWS Lambda (CloudWatch Logs / Execution Logs)

- Truy cập **AWS Lambda → chọn hàm `WAFAutoBlockFunction` → chọn tab Monitor → chọn View CloudWatch logs**.
- Mở Log Stream mới nhất và kiểm tra nội dung nhật ký:

```text
Bắt đầu phân tích log từ Log Group: aws-waf-logs-cloudfront với ngưỡng: 100
Tìm thấy danh sách IP vi phạm tổng cộng: ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
Danh sách IP tương thích với IP Set (IPV6): ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
Đã cập nhật thành công WAF IP Set AutoBlockedIPSetV6. Danh sách IP mới: ['2405:4802:1d5e:a560:782e:f496:9ce7:d360/128']
Đã gửi email thông báo chi tiết IP bị chặn qua SNS thành công.
```

![Nhật ký thực thi (Execution Logs) của hàm Lambda WAFAutoBlockFunction](/images/5-Workshop/5.10-Testing/buoc3-lambda-execution-logs.png)

#### Bước 4: Kiểm tra WAF IP Set và phản hồi chặn (Blocking Verification)

**Kiểm tra WAF IP Set trên AWS Console:**

- Truy cập **AWS WAF → IP sets** (chọn Region **Global (CloudFront)**).
- Mở `AutoBlockedIPSetV6` (hoặc `AutoBlockedIPSet`), xác nhận địa chỉ IP của máy kiểm thử đã được thêm tự động vào danh sách.

![WAF IP Set AutoBlockedIPSetV6 chứa địa chỉ IP vi phạm](/images/5-Workshop/5.10-Testing/buoc4-waf-ip-set.png)

**Kiểm tra phản hồi chặn thực tế:**

Mở CMD và gửi lại 1 request kiểm tra:

```dos
curl -I https://d2n9euazwledbh.cloudfront.net/
```

**Kết quả:** Trả về mã lỗi `HTTP/1.1 403 Forbidden` (Xác nhận chặn thành công).

![Phản hồi HTTP 403 Forbidden khi IP đã bị chặn tiếp tục truy cập](/images/5-Workshop/5.10-Testing/buoc4-http-403-forbidden.png)

#### Bước 5: Kiểm tra Email thông báo từ SNS

Kiểm tra hộp thư đến của Gmail đã đăng ký với SNS Topic, bạn sẽ nhận được 2 email:

1. **Email cảnh báo ngưỡng từ CloudWatch Alarm:** Thông báo hệ thống bị vượt ngưỡng request.

   ![Email cảnh báo ngưỡng request từ CloudWatch Alarm](/images/5-Workshop/5.10-Testing/buoc5-email-cloudwatch-alarm.png)

2. **Email thông báo chi tiết từ Lambda:**
   - **Subject:** `[AWS WAF AUTO-BLOCK] Đã chặn thành công IP vi phạm`
   - **Body:**

     ```text
     Hệ thống tự động phát hiện và chặn IP truy cập bất thường:

     - Danh sách IP vừa bị chặn: 2405:4802:1d5e:a560:782e:f496:9ce7:d360/128
     - Tên WAF IP Set: AutoBlockedIPSetV6
     - Phiên bản IP: IPV6
     - Thời gian thực thi: 2026-09-10 17:39:23 UTC

     Các IP trên đã được tự động thêm vào WAF IP Set và bị từ chối truy cập.
     ```

   ![Email thông báo chi tiết IP bị chặn gửi từ Lambda qua SNS](/images/5-Workshop/5.10-Testing/buoc5-email-lambda-auto-block.png)

---

### Đánh giá kết quả

| Hạng mục kiểm thử   | Trạng thái kỳ vọng                              | Trạng thái thực tế                                      | Kết luận   |
| ------------------- | ----------------------------------------------- | ------------------------------------------------------- | ---------- |
| Ghi nhận Log WAF    | Đẩy log truy cập về CloudWatch Logs             | Log xuất hiện trong Log Group `aws-waf-logs-cloudfront` | Đạt (PASS) |
| Kích hoạt Alarm     | Chuyển sang `IN ALARM` khi > 100 requests/5 min | Alarm kích hoạt chính xác khi đạt ngưỡng                | Đạt (PASS) |
| Tự động hóa Lambda  | Trích xuất chính xác IP vi phạm (`clientIp`)    | Trích xuất thành công IPv6/IPv4 vi phạm                 | Đạt (PASS) |
| Cập nhật WAF IP Set | Thêm IP vi phạm vào IP Set tương ứng            | Đã tự động gộp IP vào `AutoBlockedIPSetV6`              | Đạt (PASS) |
| Phản hồi từ chối    | Phản hồi lỗi `403 Forbidden` đối với IP bị chặn | Chặn thành công, trả về HTTP 403 khi curl               | Đạt (PASS) |
| Gửi thông báo Email | Gửi email thông báo chi tiết IP bị chặn         | Email SNS gửi về Gmail đầy đủ thông số IP               | Đạt (PASS) |

**Đánh giá chung:** Hệ thống phản ứng hoàn toàn tự động, phát hiện và ngăn chặn thành công các truy cập vượt ngưỡng theo đúng thiết kế kiến trúc đề ra.
