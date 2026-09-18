---
title: "Blog 2"
date: 2026-08-15
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Xây dựng Pipeline Machine Learning End-to-End với Amazon SageMaker

Khi bắt đầu tìm hiểu về Machine Learning, hầu hết chúng ta đều dành nhiều thời gian để lựa chọn thuật toán, xử lý dữ liệu và tối ưu các chỉ số như Accuracy, Precision hay F1-Score. Sau nhiều lần thử nghiệm, khi mô hình đạt được kết quả mong muốn, cảm giác như bài toán đã được giải quyết.

Nhưng thực tế lại không đơn giản như vậy.

Một mô hình Machine Learning chỉ thực sự mang lại giá trị khi nó có thể phục vụ người dùng trong môi trường thực tế. Điều đó đồng nghĩa với việc mô hình cần được triển khai thành một dịch vụ, có khả năng xử lý hàng nghìn yêu cầu dự đoán, dễ dàng cập nhật khi có dữ liệu mới và luôn được giám sát để đảm bảo hoạt động ổn định.

Đây cũng là điểm khác biệt giữa một mô hình chạy trên máy tính cá nhân và một hệ thống Machine Learning trong môi trường Production.

Để giải quyết bài toán này, các doanh nghiệp thường xây dựng Machine Learning Pipeline End-to-End – một quy trình tự động hóa toàn bộ vòng đời của mô hình, từ dữ liệu đầu vào cho đến khi mô hình được triển khai và vận hành. Thay vì thực hiện thủ công từng công đoạn, các bước sẽ được kết nối thành một pipeline thống nhất, giúp giảm sai sót, tăng khả năng tái sử dụng và rút ngắn đáng kể thời gian triển khai.

Trong hệ sinh thái AWS, Amazon SageMaker là dịch vụ được xây dựng nhằm đơn giản hóa chính quy trình đó.

### Machine Learning Pipeline End-to-End là gì?

Có thể hiểu đơn giản, Machine Learning Pipeline End-to-End là một chuỗi các bước liên kết với nhau để chuyển đổi dữ liệu thô thành một dịch vụ dự đoán có thể sử dụng trong thực tế. Thay vì chỉ tập trung vào việc huấn luyện mô hình, pipeline còn bao gồm nhiều giai đoạn khác như lưu trữ dữ liệu, tiền xử lý, triển khai mô hình và giám sát hệ thống sau khi đưa vào vận hành.

Mỗi thành phần đều đảm nhận một vai trò riêng nhưng được kết nối chặt chẽ với nhau. Khi một bước hoàn thành, kết quả sẽ được chuyển tiếp cho bước tiếp theo, tạo nên một quy trình xuyên suốt và có thể tự động hóa gần như hoàn toàn.

Đây cũng là nền tảng của các hệ thống MLOps hiện đại.

### Amazon S3 – Nơi lưu trữ dữ liệu của toàn bộ hệ thống:

Trong một pipeline Machine Learning, dữ liệu luôn là thành phần quan trọng nhất.

Amazon S3 thường được sử dụng như kho lưu trữ trung tâm cho toàn bộ dự án. Không chỉ dữ liệu ban đầu được lưu tại đây mà các tập dữ liệu sau khi xử lý, mã nguồn huấn luyện, Model Artifacts hay các kết quả đánh giá cũng được quản lý tập trung trên cùng một dịch vụ.

Việc lưu trữ tập trung mang lại nhiều lợi ích. Các thành phần khác trong pipeline có thể truy cập cùng một nguồn dữ liệu, tránh tình trạng dữ liệu phân tán ở nhiều nơi. Đồng thời, Amazon S3 có khả năng mở rộng gần như không giới hạn cùng độ bền dữ liệu rất cao, phù hợp với các bài toán Machine Learning có khối lượng dữ liệu lớn.

### Tiền xử lý dữ liệu với SageMaker Processing Jobs:

Sau khi dữ liệu được lưu trữ, bước tiếp theo là chuẩn bị dữ liệu trước khi huấn luyện mô hình.

Trong thực tế, dữ liệu hiếm khi ở trạng thái hoàn hảo. Có thể tồn tại các giá trị bị thiếu, dữ liệu trùng lặp hoặc định dạng không đồng nhất. Nếu đưa trực tiếp những dữ liệu này vào mô hình thì chất lượng dự đoán sẽ bị ảnh hưởng đáng kể.

Amazon SageMaker Processing Jobs cho phép tự động hóa toàn bộ công đoạn này.

Người phát triển chỉ cần chuẩn bị chương trình xử lý dữ liệu bằng Python hoặc các framework quen thuộc, SageMaker sẽ tự động khởi tạo môi trường thực thi, chạy chương trình và lưu kết quả trở lại Amazon S3.

Những công việc như làm sạch dữ liệu, chuẩn hóa đặc trưng, tạo Feature Engineering hay chia dữ liệu thành tập Train và Validation đều có thể được thực hiện trong bước này mà không cần quản lý bất kỳ máy chủ nào.

### Huấn luyện mô hình với SageMaker Training Jobs:

Khi dữ liệu đã sẵn sàng, pipeline sẽ chuyển sang bước huấn luyện mô hình.

Thông thường, để huấn luyện một mô hình Machine Learning, người phát triển phải chuẩn bị máy chủ, cài đặt thư viện, cấu hình môi trường và đảm bảo tài nguyên tính toán đủ mạnh. Điều này vừa mất thời gian vừa làm tăng chi phí vận hành.

Amazon SageMaker Training Jobs giúp đơn giản hóa toàn bộ quá trình đó.
Người dùng chỉ cần cung cấp mã nguồn huấn luyện, chỉ định vị trí dữ liệu trên Amazon S3 và cấu hình các Hyperparameters cần thiết. SageMaker sẽ tự động khởi tạo tài nguyên tính toán, tải dữ liệu, thực hiện huấn luyện và lưu mô hình sau khi hoàn thành.

Các mô hình sau khi huấn luyện sẽ được lưu dưới dạng Model Artifacts trên Amazon S3 để phục vụ cho bước triển khai tiếp theo.

Nhờ vậy, nhóm phát triển có thể tập trung vào việc cải thiện thuật toán thay vì dành nhiều thời gian quản lý hạ tầng.

### Giám sát hệ thống với Amazon CloudWatch và Amazon SNS:

Việc triển khai thành công một mô hình mới chỉ là bước khởi đầu.

Trong môi trường Production, điều quan trọng không kém là phải biết hệ thống đang hoạt động như thế nào.

Amazon CloudWatch liên tục thu thập các Metrics của Endpoint như số lượng yêu cầu, thời gian phản hồi, mức sử dụng CPU, bộ nhớ cũng như lưu lại toàn bộ Logs trong quá trình suy luận.

Thông qua các dữ liệu này, người quản trị có thể nhanh chóng phát hiện những dấu hiệu bất thường, đánh giá hiệu năng của hệ thống và xác định nguyên nhân khi xảy ra lỗi.

Bên cạnh đó, CloudWatch Alarms có thể được cấu hình để tự động theo dõi các ngưỡng đã thiết lập. Khi phát hiện điều kiện bất thường, Alarm sẽ kích hoạt Amazon SNS để gửi Email hoặc các thông báo khác đến quản trị viên.

Nhờ cơ chế này, hệ thống luôn được giám sát liên tục mà không cần theo dõi thủ công.

### Tại sao nên xây dựng Machine Learning Pipeline?

Nhiều người cho rằng Machine Learning chỉ xoay quanh việc lựa chọn thuật toán phù hợp hoặc tối ưu các chỉ số đánh giá. Tuy nhiên, trong thực tế, phần lớn công việc lại nằm ở cách xây dựng và vận hành toàn bộ hệ thống xung quanh mô hình.

Một Machine Learning Pipeline hoàn chỉnh mang lại rất nhiều lợi ích.

Quy trình được tự động hóa giúp giảm thiểu sai sót khi triển khai và đảm bảo mọi lần huấn luyện đều tuân theo cùng một quy trình. Việc lưu trữ dữ liệu tập trung giúp các thành phần trong hệ thống dễ dàng chia sẻ và tái sử dụng dữ liệu. Các dịch vụ được quản lý hoàn toàn của AWS cũng giúp giảm đáng kể công sức quản lý hạ tầng, đồng thời cho phép mở rộng hệ thống khi nhu cầu sử dụng tăng lên.

Quan trọng hơn, việc giám sát liên tục giúp phát hiện sớm các vấn đề trong quá trình vận hành, từ đó nâng cao tính ổn định và độ tin cậy của toàn bộ hệ thống.

### Kết luận

Machine Learning ngày nay không còn chỉ là việc xây dựng một mô hình có độ chính xác cao. Để mô hình thực sự tạo ra giá trị, cần có một quy trình hoàn chỉnh bao gồm lưu trữ dữ liệu, tiền xử lý, huấn luyện, triển khai và giám sát sau khi đưa vào vận hành.

Amazon SageMaker cùng với các dịch vụ như Amazon S3, Amazon CloudWatch và Amazon SNS giúp kết nối tất cả những thành phần này thành một Machine Learning Pipeline End-to-End. Nhờ đó, người phát triển có thể giảm đáng kể thời gian xây dựng hạ tầng, tập trung nhiều hơn vào việc cải thiện chất lượng mô hình và sẵn sàng triển khai các ứng dụng Machine Learning trong môi trường Production.

Nếu bạn đang bắt đầu tìm hiểu về MLOps hoặc muốn triển khai Machine Learning trên AWS, việc nắm được cách các dịch vụ này phối hợp với nhau sẽ là một nền tảng rất hữu ích trước khi đi sâu vào những pipeline phức tạp hơn.

![Processing Container](/images/3-BlogsPosted/Processing-1.png)

---

### Tài liệu tham khảo

*   [Amazon SageMaker Documentation](https://docs.aws.amazon.com/sagemaker/)
*   [Amazon SageMaker Pipelines Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines.html)
*   [Amazon SageMaker Processing Jobs](https://docs.aws.amazon.com/sagemaker/latest/dg/processing-job.html)
*   [Amazon SageMaker Training Jobs](https://docs.aws.amazon.com/sagemaker/latest/dg/train-model.html)
*   [Amazon SageMaker Endpoints (Real-time Inference)](https://docs.aws.amazon.com/sagemaker/latest/dg/realtime-endpoints.html)
*   [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
*   [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
*   [Amazon SNS Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)