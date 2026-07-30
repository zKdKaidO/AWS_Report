---
title: "Hạ tầng AWS"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---
# Hạ tầng AWS

## Mục tiêu

Tài liệu hóa chi tiết toàn bộ hạ tầng AWS phục vụ cho công tác triển khai production thực tế và cung cấp đầy đủ các chuỗi lệnh kiểm chứng an toàn ứng với từng cụm tài nguyên cốt lõi trong hệ thống.

## Phạm vi

Chương tài liệu này tập trung ghi nhận trọn vẹn những tài nguyên đã thực thụ triển khai có kèm thư viện bằng chứng bảo an, điểm tên thẳng thắn những thông số trường mục còn ngụ đợi lệnh chiết xuất rà tra cẩu bằng AWS CLI, và ra riêng danh mục đề xuất gia cố an ninh tự nguyện cho nhà phát triển bám đuổi về sau.

## Bối cảnh kiến trúc

Hạ tầng nền tảng tự do bấu víu cho một kiến trúc kết hợp vi diệu giữa hai chiến tuyến Cloud CDN ngoại vi và cỗ máy container tập trung:

- Cầu nối CloudFront và kho S3 chuyên trách phân phối trang giao diện frontend.
- Cánh cổng ALB luân chuyển gạn chia tải truy nhập động lao thẳng vào bên trong EKS.
- Cụm máy chủ EKS vững chí gìn bám, cưu mang toàn cõi cho chuỗi workload phần mềm ngụ trực trăn dài hạn.
- Cơ quan cơ sở dữ liệu được quản lý (managed databases), kênh truyền tin, kho object storage, dịch vụ Lambda, máy gửi mail SES cùng cỗ cống suy luận SageMaker chễm trệ dựng cao tường mang lại dịch vụ nền tảng tráng kiệt cho hệ thống.
- Dây chuyền tự động trên GitHub Actions ung dung múa trượt triển khai qua ranh giới OIDC mà vội buông rơi hoàn toàn việc cầu cạnh các khóa AWS access keys vô hạn ngày lì lợm.

## Các tài nguyên đã được triển khai

| Khu vực | Tài nguyên | Trạng thái |
|---|---|---|
| Tài khoản và khu vực | Tài khoản AWS của dự án, vùng `ap-southeast-1` | Đã che giấu (redacted) số ID tài khoản |
| EKS | `internship-prod` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 và kịch bản workflow |
| Namespace | `internship` | Đã xác minh qua tài liệu kịch bản manifests |
| ALB | `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| S3 bucket cho frontend | `internship-prod-frontend-<AWS_ACCOUNT_ID>` | Đã ẩn phần đuôi tài khoản |
| S3 bucket lưu tệp / archive | `internship-prod-uploads-<AWS_ACCOUNT_ID>` | Đã ẩn phần đuôi tài khoản |
| CloudFront | Distribution `EQIGYNECXDYL8` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| RDS | `internship-prod-postgres` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| Redis | `internship-prod-redis`, trạng thái `available` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| DynamoDB | `ChatUsers`, `ChatGroups`, `ChatMessages`, `InternshipLambdaEventDedupe` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| SQS | `internship-prod-outbox`, `internship-prod-outbox-dlq` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| SageMaker | Endpoint `internship-qwen3-4b` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| Lambda | `internship-outbox-handler` | Đã xác minh qua hồ sơ bằng chứng Tuần 8 |
| Các IAM roles | `internship-github-deploy`, `internship-eks-runtime`, `internship-sagemaker-execution`, `internship-eks-node-role`, `internship-lambda-outbox-role`, `AmazonEKSLoadBalancerControllerRole` | Đã xác minh từ hệ thông cấu hình triển khai và tệp chứng cứ Tuần 8 |

## Nền tảng mạng

Cụm máy EKS trong bối cảnh sản xuất production nhất định khẩn khỏm đòi hỏi cấu trúc mạng VPC có chia đôi phân cách giữa các subnet public và private:

- Các subnet public thọ nuôi cụm máy cân bằng tải hứng lưu lượng từ thế giới public bên ngoài và kho vây các máy NAT Gateway.
- Các subnet private bao vây an ninh êm bế nuôi dưỡng cụm máy cọc worker nodes của EKS cùng đường rẽ vào bên trong của mạng dịch vụ AWS được quản lý.
- Một cống Internet Gateway có chức năng khai quan cho lưu lượng public trượt thiền vào và mở đàng cho lưu lượng từ NAT ung dung phi xuất ngoại.
- Một trạm chuyển giao NAT Gateway ra mặt bao sắm cho dàn máy nằm an tịnh trong lòng subet private đặc quyền tiếp cận ra vạch ngoài Internet để bồi nạp lấy container image, trao đổi chuỗi gọi tới AWS APIs, tải về tệp gói phụ trợ, đồng thời giao đãi mạch nảy theo cụm điều phối control-plane.

Cần bằng chứng:

- Thông số định danh VPC ID, tuy rằng biến cố lỗi xử lý trong sự cố khắc phục đã kiên thấu xác tín số `vpc-0cfee519122ae18b4` từng bị sử dụng để cầu khấn rước AWS Load Balancer Controller múa việc.
- Các mã số nhận diện Public subnet IDs.
- Các mã số nhận diện Private subnet IDs.
- Danh bạ số liệu bảng định tuyến Route table IDs.
- Nhóm bảo mật Security group IDs và điều luật tường lửa vây kèm.
- Số hiệu của NAT Gateway ID kết hợp mã cấp tài khoản địa chỉ Elastic IP allocation ID.

Kiểm chứng:

```bash
aws ec2 describe-vpcs --region ap-southeast-1
aws ec2 describe-subnets --region ap-southeast-1
aws ec2 describe-route-tables --region ap-southeast-1
aws ec2 describe-nat-gateways --region ap-southeast-1
aws ec2 describe-security-groups --region ap-southeast-1
```

## IAM và OIDC

Hệ luồng trên GitHub Actions ứng dụng an ninh cơ quan xác minh AWS OIDC. Kịch bản workflow phán bảo ràng buộc các uy quyền:

```yaml
permissions:
  id-token: write
  contents: read
```

Các tác vụ phục vụ triển khai liên tục gõ gọi gói công cụ `aws-actions/configure-aws-credentials@v6` kèm chuỗi thông số `role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}` và chốt trần bằng cọc `allowed-account-ids: ${{ env.AWS_ACCOUNT_ID }}`.

Các workload chạy bên trong môi trường runtime thụ hưởng vượng bổng thông qua ServiceAccount của Kubernetes mang tên `internship-app`. Whenever tham số biến `IRSA_ROLE_ARN` được truyền sang đầy đủ, kịch bản `scripts/k8s/deploy-eks.sh` ngay lập tức thi triển khâu render nội sung tệp `k8s/eks/serviceaccount.yaml` đồng thời áp luôn chuỗi chú thích (annotation) cho role vào manifest.

Kiểm chứng:

```bash
aws iam get-role --role-name internship-github-deploy
aws iam get-role --role-name internship-eks-runtime
aws iam get-role --role-name internship-lambda-outbox-role
kubectl get serviceaccount internship-app -n internship -o yaml
```

## ECR

Luồng quy trình tự động tiến hành build ra các image cắm mã nhãn bám trúng chuỗi SHA và khắt khe thanh rà kiểm định hai image của backend và chat trước thời điểm ra phất cờ cho mác triển khai:

| Image | Kho ECR mặc định |
|---|---|
| Backend | `internship-backend` |
| Dịch vụ chat | `internship-chat` |
| Bộ đệm AI adapter | `internship-ai` |

Kiểm chứng:

```bash
aws ecr describe-repositories --region ap-southeast-1
aws ecr describe-images --region ap-southeast-1 --repository-name internship-backend --image-ids imageTag=<GITHUB_SHA>
aws ecr describe-images --region ap-southeast-1 --repository-name internship-chat --image-ids imageTag=<GITHUB_SHA>
```

## Cụm EKS và các workload

Cụm EKS chuyên gánh tải và vận hành:

| Workload | Loại | Ghi chú |
|---|---|---|
| `backend` | Deployment và Service | Dịch vụ FastAPI mở thông tại port `8000`, 2 replica |
| `chat-service` | Deployment và Service | Dịch vụ Node.js/Socket.IO chạy bám tại port `3000`, 2 replica |
| `backend-outbox-dispatcher` | Deployment và Service | Dành sẵn port cho metrics là `9101`, 1 replica |
| `backend-processing-worker` | Deployment và Service | Mở thông port giám sát metrics tại `9102`, được trao tự chủ đóng xuống mức 0 replica chừng nào AI chưa khang an |
| `ai-service` | Deployment và Service | Pod trung gian FastAPI adapter mở nghênh tại port `8010`, 1 replica |
| `backend-migrate` | Job | Nhiệm vụ có trọng trách khởi thực thi câu lệnh `alembic upgrade head` |
| `chat-init` | Job | Tác vụ ban khải tạo bảng cho chuỗi CSDL chat bên trong DynamoDB |

Kiểm chứng:

```bash
aws eks describe-cluster --name internship-prod --region ap-southeast-1
aws eks list-nodegroups --cluster-name internship-prod --region ap-southeast-1
kubectl get deployments,pods,svc,hpa,pdb,jobs -n internship -o wide
```

## AWS Load Balancer Controller và ALB

Tài nguyên Ingress chủ lực trong môi trường production mang tên định danh `internship-public` và kết nối bấu giữ các chuỗi chú thích thuộc về AWS Load Balancer Controller:

- mô hình (scheme): `internet-facing`
- kiểu mục tiêu (target type): `ip`
- listener: HTTP `80`
- đường dẫn kiểm thử sức khỏe (health check path): `/health/ready`
- thời gian chờ rỗi (idle timeout): `3600`
- cơ chế bám dính của target group (target group stickiness): cho phép bật trong `86400` giây
- các dải phân tuyến (paths): `/api`, `/chat`, `/socket.io`

Kiểm chứng:

```bash
kubectl get ingress internship-public -n internship -o yaml
kubectl get deployment,pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller -o wide
aws elbv2 describe-load-balancers --region ap-southeast-1
aws elbv2 describe-target-groups --region ap-southeast-1
```

## S3 và CloudFront

Luồng phân phối cho frontend tận dụng triệt để:

- kho lưu trữ đóng kín riêng tư `internship-prod-frontend-<AWS_ACCOUNT_ID>`
- CDN CloudFront distribution `EQIGYNECXDYL8`
- cơ quan truy nhập gốc an toàn CloudFront Origin Access Control
- cấu hình behavior mặc định thả trôi về phía kho S3
- các behavior dành riêng cho dải `/api/*`, `/chat/*`, `/socket.io/*` được bẻ cua lao thẳng về cổng ALB
- cơ chế bồi đường dự phòng SPA fallback dành cho hai mã lỗi 403 cùng 404 cho rẽ bẻ trúng tệp `/index.html`

Kiểm chứng:

```bash
aws s3api get-public-access-block --bucket internship-prod-frontend-<AWS_ACCOUNT_ID>
aws cloudfront get-distribution --id EQIGYNECXDYL8
aws cloudfront list-invalidations --distribution-id EQIGYNECXDYL8
```

## Các tài nguyên liên lạc và hướng sự kiện

Những cấu trúc đã trỗi sinh hoàn tất trong môi trường chạy hiện tại:

| Tài nguyên | Cấu hình |
|---|---|
| Hàng đợi chủ lực SQS | `internship-prod-outbox`, định mốc thời gian ẩn (visibility timeout) là 120 giây, giới hạn lưu tệp 4 ngày, bật mã hóa SSE |
| Hàng đợi lưu lỗi SQS DLQ | `internship-prod-outbox-dlq`, cẩn nhẫn giữ tệp suốt 14 ngày, số kỳ thu thử lại `maxReceiveCount` là 5 |
| Hàm Lambda | `internship-outbox-handler`, cấu hình kích nổ từ trigger hàng đợi SQS |
| Bảng khử lặp sự kiện (Dedupe table) | `InternshipLambdaEventDedupe` |
| Bãi chôn giữ vãn (Archive location) | `s3://internship-prod-uploads-<AWS_ACCOUNT_ID>/outbox-archive/...` |
| Dịch vụ gửi thư điện tử | Amazon SES |

Kiểm chứng:

```bash
aws sqs get-queue-url --queue-name internship-prod-outbox --region ap-southeast-1
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws lambda get-function --function-name internship-outbox-handler --region ap-southeast-1
aws lambda list-event-source-mappings --function-name internship-outbox-handler --region ap-southeast-1
aws dynamodb describe-table --table-name InternshipLambdaEventDedupe --region ap-southeast-1
```

Tôi đã chủ động ghi nhận ra một trường hợp vênh đâm đụng độ chéo trong đặt tên: bộ đôi tệp script kịch bản kho lưu repository `scripts/aws/provision-outbox-sqs.sh` cùng file `.ps1` đang an ủi thi theo giá trị mặc định `internship-outbox-events` với chu kỳ visibility timeout ngã ngắn ở mốc `60` giây, giữa lúc con hàng đợi vận hành rực rỡ tại production thật sự ngoài thực tế lại oai dũng mang xưng danh `internship-prod-outbox` và giữ trọn mốc thời gian visibility timeout dài tận `120` giây. Tôi tự trọng duy trì tín niệm bấu vào thư viện minh chứng hạ tầng production hiện trường như mạn nguồn chân lý cho cõi thực thi, và dĩ nhiên xin cẩn kính dãn công việc canh chỉnh lại chuỗi script bị sa nhầm trong repo mã gốc coi như một nghiệp vụ độc lập không nằm trong phạm vi chỉnh lý của báo cáo.

## Các tài nguyên SageMaker

Cấu trúc nghiệp vụ quy định cụm worker sẽ thi phác gõ qua cổng dịch vụ bộ đệm trung gian `ai-service`, tới luợng mình pod `ai-service` mới giục còi gõ thẳng vào cõi SagMaker endpoint `internship-qwen3-4b`. Trình kịch bản triển khai đòi hỏi cọc cấm SagMaker endpoint phải hoành múa uy phong ngã đúng trại thái `InService` trước phút nới cọc biến tự động bật công tắc cho phép `PROCESSING_WORKER_ENABLED=true` thực thi.

Kiểm chứng:

```bash
aws sagemaker describe-endpoint --endpoint-name internship-qwen3-4b --region ap-southeast-1
kubectl get deployment,svc ai-service -n internship
```

## Đề xuất củng cố bảo mật (Hardening)

| Khu vực | Đề xuất gia cố |
|---|---|
| ALB | Khóa gạt chặn thi rào cấm không cho truy nhập trần lộ trực diện vào ALB nhằm bắt luồng lưu thông công cộng 100% lọt qua mạn CDN CloudFront nếu có thể |
| CloudFront | Nạp thêm tên miền riêng custom domain cùng chứng chỉ bảo an ACM certificate cho bạt sắc production uy tín |
| S3 | Soái kỹ lại cấu trúc bucket policy ra quy cấm ngặt nghẽo bảo đảm chỉ dành chốn vào cho duy nhất cống CloudFront và cụm workload thuyên chuyển bắp |
| WAF | Chiêu nạp tường lửa ứng dụng web AWS WAF chắn hộ trước mặt CloudFront bện cọc cản bước các luồng rỗng tai ương hạ độc |
| Nhật ký (Logs) | Đóng cố định thông số giới hạn bảo lưu tệp nhật ký CloudWatch retention trọn các log của ứng dụng và Lambda |
| Sao lưu (Backups) | Rà thẩm cọc retention lưu hồ sơ sao lưu tự động cho RDS cùng nhuần kiệt tiêu chuẩn điểm khôi phục PITR |
| Cảnh báo (Alarms) | Gắn lắp các mạch báo nguy khẩn thiết cho hàng đợi DLQ, tỉ lệ báo hỏng từ Lambda, lỗi hệ CSDL RDS, chao lạch ALB và mức ngốn nợ SageMaker |
| Chi phí | Cài kịch bản theo chuỗi lịch biểu để ngưng vãn hoặc tháo bỏ vĩnh viễn SageMaker endpoint ngoài lúc phục vụ demo/kiểm thử |

## Kết quả mong đợi

Bộ sườn kiến trúc hạ tầng vỗ tay hoàn trọn công đoạn sắm sinh khi mạng nối mạch lạc, các trạm máy con EKS giễu cao trạng thái Ready, cổng cống ALB điều rẽ luồng sức khỏe thọ khương mượt mà, hai trụ S3 kết hợp CloudFront ngạo mạn phát rạng trang frontend, kho dữ liệu ngự vững, nhịp tim SQS và Lambda ráo chiết tệp tin sự kiện và cụm thần kinh SageMaker an ngự trước trần kích hoạt đàn worker bám việc.

## Các lỗi thường gặp

| Triệu chứng | Nguyên nhân gốc rễ | Hướng khắc phục |
|---|---|---|
| Cụm máy node của EKS rền ngả khước từ tham dự vào cluster | Bảng định tuyến ranh giới private chao trượt lọt mạn trúng một NAT Gateway từng bị hủy tháo xô phai | Hồi sinh tại cho lập mới cỗ máy NAT rồi kiên mẫn bùng tạc xây tạo khôi phục lại node group |
| Trình điều khiển ALB controller bất lực mù tịt chẳng thấu tên VPC | Bước tra cứu siêu dữ liệu (metadata) lọt thẳm quá mốc thời hạn | Chủ động ấn nhét tham số biến cờ `--aws-region=ap-southeast-1` cùng `--aws-vpc-id=vpc-0cfee519122ae18b4` bồi thẳng vào lệnh cấu hình |
| Luồng điều hướng API trên cống CloudFront dội báo lỗi trật lất | Thông tin miền DNS của ALB bị khai khuyết hoặc sai quy luật của cache behavior | Dò soát lại thận trọng trường tham số origin và chỉnh ngay các điều khoản trong cache behavior của CloudFront |
| Vòng lặp rà cứu SQS thông trả lỗi chăng thấu bóng hình hàng đợi ở đâu | Tát gọi nhầm tên hàng đợi SQS | Ấn chỉnh gõ chuẩn chuỗi cộc danh mác `internship-prod-outbox` |
| Khâu ra mắt cụm worker lâm nạn ngã lọt giếng lầm gào khóc | Máy chủ suy đoán SageMaker endpoint lả lướt nằm chìm ở tình trạng phi `InService` | Quyết tâm giữ nghẽo nhịp công tắc worker ở thể khóa tịt (disabled) cho tới khi cổng sấm AI hưng hoan phất cờ thọ bệnh sẵn sàng |

## Kết luận

Chương trình về Hạ tầng AWS khơi mở, ban tặng tấm sơ đồ định trát phân chia tài nguyên trọn khắp thế trận production thực tế, đồng thời trang bị sẵn chuỗi vũ khí dòng lệnh tra xét thẩm rà trứ bệ làm nòng cốt dạo bước ung dung trước các môn công tác khởi dựng CSDL, triển mác backend, thiêu web frontend, gắn thấu quan trắc, thí trù nghiệm thu cùng tháo gỡ quy thu chi cước tài nguyên bám sát kế tiếp ngay sau đây.
