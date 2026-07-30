---
title: "Tự đánh giá"
date: 2026-30-07
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Tự đánh giá

## Tổng quan đánh giá

Phần tự đánh giá này phản ánh quá trình học tập và những đóng góp của tôi trong dự án Internship Application Tracker. Các mức điểm được đưa ra dựa trên quá trình tôi trực tiếp tham gia phân tích kiến trúc, chuẩn bị triển khai AWS, nghiên cứu tích hợp AI, xử lý sự cố, xây dựng tài liệu và kiểm chứng hệ thống. Đây không phải là kết quả đánh giá chính thức từ người hướng dẫn thực tập.

Thông qua dự án, tôi có cơ hội tiếp cận thực tế với việc kết nối mã nguồn ứng dụng và hạ tầng điện toán đám mây. Tôi nhận ra rằng triển khai một hệ thống hoàn chỉnh không chỉ yêu cầu viết mã nguồn mà còn liên quan đến mạng, quyền IAM, container, Kubernetes, các dịch vụ AWS được quản lý, quy trình CI/CD, giám sát, bảo mật, chi phí và tài liệu kỹ thuật.

## Đánh giá kỹ năng

| Nhóm kỹ năng | Điểm tự đánh giá | Bằng chứng và nhận xét |
|---|---:|---|
| Kiến trúc AWS và lựa chọn dịch vụ | 8/10 | Tôi có thể giải thích vai trò và cách kết nối giữa CloudFront, S3, ALB, EKS, RDS, Redis, DynamoDB, SQS, Lambda, SES, SageMaker và ECR trong dự án. Tôi cũng chú ý hơn đến việc phân biệt kiến trúc mục tiêu với các dịch vụ đã thực sự được triển khai và kiểm chứng. |
| Triển khai Cloud và Kubernetes | 7.5/10 | Tôi đã làm việc với cấu trúc triển khai backend, dịch vụ chat thời gian thực, worker và các workload liên quan đến AI trên Amazon EKS. Tôi hiểu vai trò của Deployment, Service, health probe, autoscaling, container image và load balancing, nhưng vẫn cần thêm kinh nghiệm để có thể tự vận hành EKS một cách độc lập. |
| Tích hợp AI và machine learning | 8/10 | Tôi đã nghiên cứu cách kết nối processing worker, AI service và SageMaker endpoint trong ứng dụng. Tôi cũng có kinh nghiệm làm việc với mô hình ngôn ngữ chạy local, thiết kế prompt có cấu trúc, kiểm tra JSON đầu ra, grounding, loại bỏ dữ liệu trùng lặp và đánh giá kết quả xử lý mô tả công việc và CV. Tôi cần học thêm về fine-tuning, giám sát mô hình trong môi trường production và tối ưu chi phí inference. |
| Backend và kiến trúc hướng sự kiện | 8/10 | Tôi hiểu cách PostgreSQL lưu dữ liệu nghiệp vụ có tính giao dịch, Redis hỗ trợ dữ liệu tạm thời cần tốc độ cao, DynamoDB lưu trữ dữ liệu có khả năng mở rộng và SQS kết hợp với Lambda để xử lý sự kiện bất đồng bộ. Tôi cũng hiểu tầm quan trọng của idempotency, transactional outbox, retry và dead-letter queue trong hệ thống phân tán. |
| CI/CD và tự động hóa | 7.5/10 | Tôi đã tìm hiểu cách GitHub Actions, AWS OIDC, Amazon ECR, quy trình build image, deployment workflow và kiểm tra rollout hỗ trợ việc phát hành hệ thống có thể lặp lại. Tôi có thể theo dõi và viết tài liệu cho quy trình này, nhưng vẫn cần thêm kinh nghiệm để tự thiết kế toàn bộ pipeline từ đầu. |
| Nhận thức về bảo mật và IAM | 7/10 | Tôi hiểu tầm quan trọng của việc không sử dụng access key dài hạn, sử dụng OIDC và IAM role, tách biệt secret, giới hạn quyền truy cập S3, mã hóa hàng đợi và áp dụng nguyên tắc least privilege. Điểm tôi cần cải thiện là khả năng thiết kế và kiểm tra IAM policy một cách có hệ thống. |
| Xử lý sự cố và kiểm chứng hệ thống | 8.5/10 | Một trong những kỹ năng tôi cải thiện rõ nhất là xác định vấn đề dựa trên log, kết quả lệnh, trạng thái triển khai và bằng chứng runtime. Tôi đã tham gia xử lý hoặc phân tích các vấn đề liên quan đến networking, EKS node, ALB, điều kiện chạy GitHub Actions, SQS, CloudFront routing và cấu hình tài nguyên AWS. |
| Tài liệu kỹ thuật và truyền đạt | 8.5/10 | Tôi đã hệ thống hóa thông tin dự án thành proposal kiến trúc, workshop triển khai, worklog theo tuần, bằng chứng kiểm thử, tài liệu xử lý sự cố, phân tích bảo mật, phân tích chi phí và báo cáo tổng kết. Tôi cải thiện khả năng trình bày một quy trình kỹ thuật theo hướng có cấu trúc và có thể tái thực hiện. |

## Điểm mạnh

- Tôi có khả năng kết nối mã nguồn ứng dụng, dịch vụ Cloud, cấu hình triển khai và bằng chứng runtime thành một mô tả hệ thống thống nhất.
- Khi gặp vấn đề kỹ thuật, tôi ưu tiên kiểm tra log và hành vi thực tế của hệ thống thay vì chỉ dựa trên giả định.
- Tôi có thể nghiên cứu phương pháp tích hợp AI và chuyển chúng thành quy trình xử lý phù hợp với backend.
- Tôi chú ý phân biệt rõ phần đã phát triển, phần đã triển khai thành công, phần mới chỉ được đề xuất và phần cần cải thiện trong tương lai.
- Tôi có khả năng viết lại các quy trình triển khai phức tạp thành tài liệu để người khác có thể thực hiện theo.
- Tôi hiểu những đánh đổi cơ bản giữa độ tin cậy, bảo mật, khả năng mở rộng, hiệu năng và chi phí AWS.

## Khó khăn chính

- Amazon EKS bao gồm nhiều thành phần liên kết với nhau nên lỗi có thể xuất phát từ Kubernetes, IAM, networking, load balancer, container image hoặc AWS controller thay vì chỉ từ mã nguồn ứng dụng.
- Triển khai mô hình AI yêu cầu cân bằng giữa độ chính xác, thời gian phản hồi, tài nguyên GPU, độ sẵn sàng của endpoint và chi phí vận hành.
- Một số dịch vụ AWS có thể tạo ra chi phí lớn khi hoạt động liên tục, đặc biệt là EKS, NAT Gateway, cơ sở dữ liệu và SageMaker endpoint.
- Xử lý lỗi IAM tương đối phức tạp vì một quy trình triển khai có thể liên quan đến nhiều dịch vụ và nhiều execution role khác nhau.
- Việc thu thập bằng chứng đáng tin cậy yêu cầu kiểm tra cả cấu hình và kết quả runtime, thay vì chỉ dựa vào sơ đồ kiến trúc hoặc mã nguồn.
- Khi làm việc trong repository chung, cần thao tác Git cẩn thận để tránh ghi đè công việc của thành viên khác hoặc commit nhầm các file chỉ dùng tại máy local.

## Bài học rút ra

- Sơ đồ kiến trúc cần phản ánh đúng trạng thái triển khai và phải ghi rõ những dịch vụ mới chỉ nằm trong kế hoạch.
- Log runtime, screenshot, kết quả CLI và kết quả kiểm thử thành công là những bằng chứng mạnh hơn so với file cấu hình đơn lẻ.
- Hệ thống phân tán phải dự đoán trước khả năng nhận sự kiện trùng lặp, retry, lỗi một phần và dịch vụ tạm thời không khả dụng.
- Phần tích hợp AI nên cung cấp một interface ổn định cho ứng dụng để có thể thay đổi mô hình hoặc phương thức triển khai mà không ảnh hưởng nhiều đến hệ thống còn lại.
- Health probe và autoscaling trên Kubernetes chỉ hoạt động hiệu quả khi resource request, dependency và hành vi của ứng dụng được cấu hình đúng.
- Bảo mật cần được xem xét ngay từ giai đoạn thiết kế kiến trúc thay vì chỉ bổ sung sau khi quá trình triển khai gặp lỗi.
- Tối ưu chi phí cần xem xét đồng thời thời gian hoạt động của tài nguyên, data transfer, storage, log và giá của các dịch vụ được quản lý.
- Tài liệu kỹ thuật là một phần của quy trình phát triển vì nó hỗ trợ triển khai, xử lý sự cố, chuyển giao kiến thức và đánh giá kết quả.

## Nội dung cần cải thiện

- Tăng kinh nghiệm thực hành độc lập với việc tạo EKS cluster, thiết kế VPC, cấu hình ingress, autoscaling và observability.
- Cải thiện khả năng thiết kế IAM policy theo nguyên tắc least privilege, permission boundary, resource-level permission và các công cụ phân tích policy.
- Nghiên cứu sâu hơn các phương pháp đánh giá hệ thống đối sánh công việc và CV, bao gồm xây dựng tập dữ liệu, retrieval metrics, chất lượng reranking và human evaluation.
- Học thêm về fine-tuning, quantization, asynchronous inference, endpoint autoscaling và các phương án phục vụ mô hình có chi phí thấp hơn.
- Xây dựng integration test và end-to-end test tự động cho các luồng đăng nhập, đăng tin tuyển dụng, ứng tuyển, xử lý CV, chat thời gian thực và gửi thông báo.
- Cải thiện hệ thống giám sát bằng CloudWatch dashboard, alarm, tracing, failure-rate metric và cảnh báo chi phí.
- Thực hành xây dựng kế hoạch khôi phục sự cố, kiểm tra backup cơ sở dữ liệu, point-in-time recovery và quy trình phản ứng sự cố.

## Kế hoạch phát triển

1. Tự xây dựng một môi trường AWS có quy mô nhỏ để thực hành VPC, EKS, ALB, RDS, IAM và CI/CD từ đầu.
2. Tiếp tục phát triển pipeline AI xử lý mô tả công việc và CV với structured output, grounding, reranking và phương pháp đánh giá có thể đo lường.
3. Bổ sung smoke test tự động và end-to-end test trên trình duyệt cho các luồng chính của Candidate và HR.
4. Cải thiện khả năng quan sát hệ thống thông qua log tập trung, metric, dashboard, alarm và request ID có thể truy vết.
5. So sánh SageMaker endpoint với các phương án inference khác dựa trên độ chính xác, độ trễ, khả năng mở rộng và chi phí hàng tháng.
6. Hoàn thiện bộ bằng chứng triển khai gồm screenshot, kết quả CLI, build log, runtime log, kết quả kiểm thử và thông tin chi phí AWS.

## Định hướng nghề nghiệp

Dự án thực tập này giúp tôi xác định rõ hơn sự quan tâm của mình đối với Cloud Engineering, phát triển backend, DevOps và các hệ thống tích hợp AI. Tôi đặc biệt quan tâm đến những công việc kết hợp giữa phát triển phần mềm, hạ tầng Cloud, tự động hóa, độ tin cậy của hệ thống và ứng dụng AI vào các bài toán thực tế.

Trong tương lai, tôi muốn có khả năng tự thiết kế và triển khai một hệ thống hoàn chỉnh, từ kiến trúc ứng dụng và xử lý AI đến hạ tầng, bảo mật, giám sát, kiểm thử và tối ưu chi phí.