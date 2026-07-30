---
title: "Tổng quan"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục tiêu

Workshop này trình bày tường tận cách chuẩn bị mã nguồn, triển khai, kiểm thử xác minh, giám sát, xử lý sự cố và thực hiện quy trình dọn dẹp tài nguyên an toàn cho ứng dụng Internship Application Tracker trên hạ tầng AWS.

Môi trường production đích được xác định như sau:

| Thông số | Giá trị |
|---|---|
| Dự án | Internship Application Tracker |
| Tài khoản AWS | Tài khoản AWS thuộc dự án, ID đã được ẩn |
| Vùng (Region) | `ap-southeast-1` |
| Môi trường | `production` |
| Cụm EKS (EKS cluster) | `internship-prod` |
| Namespace Kubernetes | `internship` |
| Địa chỉ frontend công cộng | `https://dhm2rz5nmsibj.cloudfront.net` |
| Kho lưu trữ mã nguồn | `https://github.com/Temp-orgo/AWS-Internship` |

## Phạm vi

Workshop này chỉ tập trung phản ánh kiến trúc production chính thức hiện tại. Frontend được lưu trữ trên Amazon S3 và phân phối qua CloudFront. Các thiết lập Kubernetes cũ của frontend như Deployment, Service, HPA và PDB đã được tháo dỡ hoàn toàn.

Cụm EKS chịu trách nhiệm vận hành các tiến trình chạy liên tục sau:

- `backend`
- `chat-service`
- `backend-outbox-dispatcher`
- `backend-processing-worker`
- `ai-service`

Các dịch vụ AWS được quản lý tích hợp trực tiếp vào ứng dụng bao gồm: RDS PostgreSQL, DynamoDB, ElastiCache Redis, S3, SQS, Lambda, SES, SageMaker, CloudFront, ALB, ECR, IAM và CloudWatch.

## Bối cảnh kiến trúc

Trình duyệt của người dùng sẽ tiếp cận CloudFront đầu tiên. CloudFront phân luồng các request tải tài liệu tĩnh cho một S3 bucket riêng tư (private bucket), trong khi các request tới đường dẫn động sẽ được định tuyến sang Application Load Balancer (ALB):

- Đường dẫn mặc định `*` -> S3 bucket của frontend
- `/api/*` -> ALB -> FastAPI backend
- `/chat/*` -> ALB -> dịch vụ chat
- `/socket.io/*` -> ALB -> dịch vụ chat

PostgreSQL lưu trữ các bản ghi nghiệp vụ mang tính chất giao dịch (transaction) và hàng đợi công việc của worker. DynamoDB lưu giữ trích đoạn tin nhắn chat và bản ghi khử lặp cho Lambda (dedupe records). Redis mang đến kênh thông tin pub/sub cho Socket.IO kết nối linh hoạt qua lại giữa các pod thuộc dịch vụ chat. SQS đóng vai trò dãn buông sự phụ thuộc ràng buộc gay gắt (decouples) từ những sự kiện nghiệp vụ ra khỏi tải thực thi của Lambda. Lambda đảm nhận việc sao lưu các văn bản sự kiện ra kho lưu trữ S3 và phát lệnh ra thư điện tử qua SES. Tiến trình processing worker có nhiệm vụ réo gọi sang `ai-service`, từ đây bộ chuyển giao này mới trực tiếp gọi tới máy chủ suy luận SageMaker endpoint mang tên `internship-qwen3-4b`.

## Đối tượng sử dụng

Workshop này được chấp bút hướng tới:

- các thực tập sinh cloud engineering có nhiệm vụ thuyết trình và giải trình về một kiến trúc triển khai thực tiễn trên hạ tầng AWS
- các sinh viên ngành lập trình backend và DevOps có nhu cầu đưa lên hệ thống một mô hình ứng dụng đa dịch vụ
- các giám sát thẩm tra chuyên nghiệp muốn được kiểm chứng về thiết kế hạ tầng, quy trình vận hành cùng phương châm hủy th d d d th 
- những thành viên tham gia kế thừa và bảo trì ứng dụng Internship Application Tracker trong tương lai

## Phạm vi chức năng

Ứng dụng đảm bảo phục vụ các năng lực:

- xác thực tài khoản đăng nhập cho Ứng viên và bộ phận HR
- đăng thông tin tuyển dụng và tra cứu việc làm
- hỗ trợ Ứng viên nộp đơn ứng tuyển
- theo dõi luồng trạng thái cho từng hồ sơ
- tải lên máy chủ các bản CV cùng tập hồ sơ tài liệu bám kèm
- hệ thống đọc chiết tách CV, tháo bóc JD và khớp chấm điểm tương đương qua tiến trình bất đồng bộ
- tính năng chat thời gian thực qua lại giữa đôi bên
- trát ban bố thông báo sự kiện bám quy chuẩn transactional outbox
- gửi email tự động tiếp tin cảnh báo với AWS Lambda và Amazon SES

## Phạm vi kỹ thuật

Hạng mục thi công kỹ thuật bao hàm:

- Frontend phát triển bởi bộ đôi React và Vite
- Backend lập trình trên FastAPI
- Dịch vụ chat chạy trên nền Node.js, Express, và Socket.IO
- PostgreSQL phối ghép nhuần nhuyễn bộ theo dõi cơ sở dữ liệu Alembic migrations
- Cụm các bảng DynamoDB chứa dữ liệu chat
- Bộ giao tiếp Redis Socket.IO adapter
- Hệ sinh thái cụm EKS gồm Deployments, Services, HPA, PDB, ServiceAccount, ConfigMap, Secret, Jobs và Ingress
- Các S3 buckets chuyên trách phục vụ tĩnh hóa cho frontend, tệp tin tải ra tải vào và rãnh chứa archive nhật ký sự kiện
- CDN CloudFront distribution hòa quyện hai nguồn origin từ S3 và ALB
- Hàng đợi cơ sở SQS Standard queue và hàng đợi giam tin nhắn lỗi DLQ
- Máy chủ tiêu thụ sự kiện Lambda đi kèm cọc bảo vệ tính bất biến bằng DynamoDB
- Cỗ máy suy luận SageMaker endpoint ẩn an an giấu sát phía sau vi dịch vụ trung gian ai-adapter
- Chuỗi phát hành CI/CD trên GitHub Actions vận dụng an toàn cơ chế xác thực AWS OIDC

## Điều kiện tiên quyết

Người thực thi cần nắm vững trọn quy tắc ủy quyền được chấp thuận cho phép truy nhập tài khoản AWS, kho lưu trữ mã nguồn trên GitHub, cụm EKS và cụm các thông số cấu hình production của dự án. Tất cả thông số số hiệu phiên bản công cụ phải được thẩm ra rành rẽ trước từ môi trường thực tế, trừ phi mã nguồn có đóng ghi chú định danh rõ trong các tập lệnh quy củ.

## Quy trình triển khai

1. Kiểm tra kỹ càng kho chứa mã nguồn và các tập tin kịch bản tự động hóa triển khai.
2. Xác minh lại tài khoản AWS, mã khu vực Region, IAM policy cùng chuỗi liên thông định dạng OIDC từ GitHub.
3. Cấp mới hoặc thẩm định chu đáo các bộ phận cấu thành hạ tầng AWS.
4. Thiết lập lần lượt dịch vụ cho RDS, Redis, DynamoDB, S3, SQS, Lambda, SageMaker cùng distribution cho CloudFront.
5. Biên dịch tiến trình đóng gói image cho backend, chat, AI rồi ném nạp lên kho lưu trữ Amazon ECR.
6. Triển khai tiếp nối backend, chat, dispatcher, worker và AI adapter vào bên trong vòng tay cụm EKS.
7. Build khoá nguồn giao diện tĩnh Vite cho frontend và đăng bạt tập tin thẳng lên hòm S3.
8. Gọi lệnh vô hiệu hóa bộ nhớ đệm (invalidating cache) trên CDN CloudFront.
9. Chạy các lệnh rà soát sức khỏe dịch vụ kết hợp tiến trình nghiệm thu trọn gói kiểm thử đầu cuối.
10. Ghi nhận và theo dõi các luồng nhật ký log, chỉ số thông lượng metrics và tín hiệu báo tháo động cảnh báo alarms.
11. Vận dụng dữ liệu bấu víu minh chứng để phân giải từng rắc rối lỗi lọt bế tắc có thể xảy đến ra quyết liệt.
12. Xuất chiêu quy dọn tháo rỡ từng chu trình an toàn theo trật tự phụ thuộc ngay khi lớp học không còn nhu cầu níu bấu duy trì tiếp.

## Kết quả mong đợi

Sau chuỗi thực thi triển khai mĩ mãn:

- CloudFront đón nhận và phân phát trôi chảy frontend.
- `/api/health/ready` đáp trả tín hiệu mượt mà từ sau hàng vây FastAPI backend.
- `/chat/health/ready` báo tin xanh mướt ra từ cụm dịch vụ chat.
- Mạch giao thông Socket.IO thông chao qua nấc `/socket.io`.
- Khung sọ PostgreSQL, DynamoDB, Redis, SQS, Lambda, SES, S3, cùng cỗ máy SageMaker đồng bộ móc xâu kết cấu bám bó bền chặt.
- Chuỗi thợ thủ công trên GitHub Actions đắc lực băng băng kiểm chứng ráo rã và bung phát bản phát hành êm trôi qua ranh giới bảo an OIDC.
- Hệ thống trỗi dậy sinh nhai thọ nhịp, chiết rót tinh gọn sự kiện nghiệp vụ và xuất nạp trôi trót lọt tệp tin quy gọn sang mạn Lambda cùng bãi neo lưu giữ.

## Kiểm chứng

Sự dụng cụm lệnh căn bản này để tiến hành khám xét nhanh tình trạng sau giai đoạn thâu thao triển khai:

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --name internship-prod --region ap-southeast-1
kubectl get deployments,pods,svc,hpa,pdb -n internship
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/api/health/ready
curl -fsS https://dhm2rz5nmsibj.cloudfront.net/chat/health/ready
```

## Các lỗi thường gặp

| Triệu chứng | Nguyên nhân có thể | Lệnh chẩn đoán đầu tiên |
|---|---|---|
| CloudFront trả về đúng frontend tuy nhiên các request API lại báo lỗi rách | Lỗi phát sinh ở ALB origin hoặc thiết lập behavior phi logic | `aws cloudfront get-distribution --id EQIGYNECXDYL8` |
| Pod backend chưa đạt trạng thái ready | Sai sót cấu hình liên kết cơ sở dữ liệu PostgreSQL hoặc mất thông tin mật mã secret | `kubectl logs deployment/backend -n internship --tail=100` |
| Route kiểm định sức khỏe của dịch vụ chat trả về `503` | Không thể bắt tuyến đến dịch vụ Redis hoặc bảng bên trong DynamoDB | `kubectl logs deployment/chat-service -n internship --tail=100` |
| Tiến trình worker bị bất động (disabled) dai dẳng | Pod ai-service hoặc cụm máy chủ cống suy luận SageMaker endpoint chưa trọn trạng thái sẵn sàng | `aws sagemaker describe-endpoint --endpoint-name internship-qwen3-4b --region ap-southeast-1` |
| Lambda tiếp đón các chuỗi sự kiện bị trùng lặp | Đặc trưng hoàn toàn bản năng của quy luật giao tin ít-nhất-một-lần bên phía SQS | Kiểm tra tình hình bảng tháo rách khử lặp `InternshipLambdaEventDedupe` |

## Kết luận

Chương Tổng quan này đóng mốc lập móng vững vàng làm điểm tham lam bồi tự vững cho các bài thực thi tiếp giáp phía sau trong workshop: từ chiến tuyến frontend cõng trên vai bộ đôi S3 kết hợp CloudFront, các chuỗi vi dịch vụ bền bỉ nội tại EKS, dữ liệu giao dịch quy vãng PostgreSQL, dữ liệu tin nhắn an cư bên trong DynamoDB, mạch tin Redis chuyên bồi đắp pub/sub cho realtime, nhịp tim SQS và Lambda vọt phóng xử lý sự kiện cho đến bãi rèn suy luận SageMaker cống hiến cho sức mạnh của AI.
