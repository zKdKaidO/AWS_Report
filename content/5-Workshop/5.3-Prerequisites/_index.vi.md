---
title: "Điều kiện tiên quyết"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---
# Điều kiện tiên quyết

## Mục tiêu

Chuẩn bị sẵn sàng các tài khoản, quyền hạn truy cập, công cụ kỹ thuật, đường dẫn mã nguồn, thông số hạn ngạch (quotas) cùng các biến môi trường thiết yếu để tiến hành triển khai và nghiệm thu hệ thống Internship Application Tracker.

## Bối cảnh kiến trúc

Dự án này ứng dụng hệ điều hành dây chuyền tích hợp GitHub Actions phối hợp cùng tiêu chuẩn xác thực không dùng khóa AWS OIDC trong quy trình triển khai môi trường production. Tuy vậy, tập công cụ lập trình tại chỗ trên máy tiếp tục giữ tầm quan trọng đặc biệt nhằm phục vụ công tác thanh tra, can thiệp nghiệm thu nóng trong hoàn cảnh nguy trượt và cứu rỗi phục hồi dữ liệu thủ công. Nghiêm cấm mọi ý định dùng các khóa AWS Access Key mang chu kỳ tồn sinh vô hạn thi thoảng đính vào dòng chảy CI/CD.

## Yêu cầu về quyền truy cập

| Khu vực truy cập | Yêu cầu |
|---|---|
| Tài khoản AWS | Được trao quyền tiếp thu và thao tác hợp pháp vào bên trong tài khoản AWS thuộc dự án; mã số ID tài khoản đã được che khuất |
| Khu vực AWS (Region) | `ap-southeast-1` |
| Kho lưu trữ GitHub | Truy nhập suôn sẻ địa chỉ `https://github.com/Temp-orgo/AWS-Internship` |
| Môi trường GitHub Actions | Các biến môi trường cùng thông tin bảo mật cho cấu trúc production environment secrets và variables |
| EKS | Có đủ uy quyền gọi lệnh cập nhật kubeconfig và tự do tiến vào xem xe rà soát trong không gian namespace `internship` |
| CloudFront và S3 | Hưởng quyền đọc soát cấu hình distribution, bucket policy, kiểm tra file đối tượng (objects), đồng thời gọi thực thi invalidations |
| RDS, Redis, DynamoDB, SQS, Lambda, SES, SageMaker | Hưởng trọn quyền đọc soát cho khâu thẩm tra nghiệm thu; role triển khai (deployment role) bắt buộc sắm đủ dải quyền ghi (write) trúng vào mục tiêu cho phép |

## Các công cụ cần thiết

| Công cụ | Phiên bản hoặc cấu hình bắt buộc | Bằng chứng |
|---|---|---|
| Git | Số phiên bản được kiểm định theo thời gian thực trên máy chạy thực thi. | Máy tính cá nhân (Local shell) |
| AWS CLI | Số phiên bản cần được rà soát trực tiếp trên hệ thống sở tại. | Các tệp script triển khai chạy lệnh `aws` |
| kubectl | Phiên bản được đối nghiệm trúng máy hiện trường. | GitHub Actions sử dụng gói `azure/setup-kubectl@v5` |
| Helm | Số phiên bản tự thẩm rà theo cỗ máy thi công. | Sách chỉ dẫn vận hành tại chỗ sử dụng Helm |
| Docker | Số phiên bản do hệ điều hành tại chỗ phán bảo. | Hệ thống CI tích cực bọc đóng image Docker |
| Node.js | Phiên bản `22` thiết định tại `.github/workflows/cicd.yml`; trên máy cá nhân tự chủ kiểm tra. | Biến môi trường trong dây chuyền `NODE_VERSION` |
| npm | Góp mặt đồng loạt chung thuyền cùng trình chạy Node.js trên CI hoặc máy địa phương. | `npm ci`, `npm run build`, `npm run test` |
| Python | Phiên bản `3.12` bấu chốt trong `.github/workflows/cicd.yml`; trên máy cá nhân cần kiểm nghiệm thực tế. | Biến môi trường workflow `PYTHON_VERSION` |
| pip | Công cụ thiết yếu lo toan cài cất trút thư viện phụ thuộc cho backend. | `backend/requirements.txt` |
| jq | Bộ phân rã chuỗi bị triệu thuyên bởi tập lệnh kịch bản chăm khéo CloudFront. | `scripts/aws/ensure-cloudfront.sh` |
| Hugo Extended | Phiên bản `0.134.3` cấm chốt tại luồng GitHub Actions của báo cáo; số tại chỗ phải thẩm tra. | Workflow cho báo cáo (Report workflow) |
| Terraform | Không mang dấu vết hiện diện ràng buộc thực chất trong bộ bối cảnh kho repository đã thám rà. | Hạ tầng được kiến tạo bằng chuỗi tệp YAML kết hợp script |

## Các tài nguyên AWS buộc phải có sẵn hoặc được khởi tạo

| Tài nguyên | Giá trị hoặc trạng thái bắt buộc |
|---|---|
| Cụm EKS | `internship-prod` |
| Namespace Kubernetes | `internship` |
| S3 bucket cho frontend | `internship-prod-frontend-<AWS_ACCOUNT_ID>` |
| S3 bucket cho tài liệu tải lên / archive | `internship-prod-uploads-<AWS_ACCOUNT_ID>` |
| CloudFront distribution | `EQIGYNECXDYL8` |
| ALB DNS | `k8s-internshippublic-48101b50ad-85486086.ap-southeast-1.elb.amazonaws.com` |
| RDS PostgreSQL | `internship-prod-postgres` |
| ElastiCache Redis | `internship-prod-redis` |
| Các bảng DynamoDB cho chat | `ChatUsers`, `ChatGroups`, `ChatMessages` |
| Bảng DynamoDB khử lặp cho Lambda | `InternshipLambdaEventDedupe` |
| Hàng đợi SQS | `internship-prod-outbox` |
| Hàng đợi lỗi SQS DLQ | `internship-prod-outbox-dlq` |
| SageMaker endpoint | `internship-qwen3-4b` |
| Hàm Lambda (Lambda function) | `internship-outbox-handler` |

## Hạn ngạch (Quota) AWS cần kiểm tra

| Dịch vụ | Hạn ngạch cần kiểm tra | Lý do quan trọng |
|---|---|---|
| EKS | Hạn ngạch về số lượng cluster và managed node groups | Rất tối thượng để phân rã lịch cho các workload cắm neo |
| EC2 | Hạn ngạch về số lõi vCPU, địa chỉ Elastic IP, bộ chuyển tiếp NAT Gateway và số lượng ENI | Máy trạm EKS, các mục tiêu từ ALB cùng rãnh tháo NAT hoàn toàn phụ thuộc vào thông số này |
| ALB | Hạn ngạch cân bằng tải Load balancer cùng target group | Ingress controller sẽ thi gõ sinh tạo tài nguyên cân bằng tải này |
| RDS | Số con máy DB instances và trữ lượng ổ lưu chữ | Thi đắc lực cho cụm vận hành PostgreSQL |
| ElastiCache | Hạn ngạch về số lượng Node | Gánh cụm Redis replication group |
| DynamoDB | Số lượng bàn cấu hình Table cùng định dạng trần thông lượng của tài khoản | Nuôi sống bộ đôi bàn chứa thư tín Chat và bảng cọc khử lặp cho Lambda |
| SQS | Quy mô số hàng đợi Queue cùng giới hạn tốc độ lưu lót tin nhắn | Nới đường cho quy trình tung phát ra thông báo outbox |
| Lambda | Khả năng thực thi song song (Concurrency) | Dành cho cỗ máy người tiêu dùng (consumer) đón nhật sự kiện SQS |
| SES | Tình trạng bị giam hãm ngục sandbox hoặc ranh giới trần thư từ cho phép ở production | Rất cốt lõi để test luồng phát thử nghiệp thư điện tử tin báo khói |
| SageMaker | Giới hạn về số máy endpoint instance, đặc sắc ưu việt là hạn ngạch GPU nếu dùng | Phán xử an ninh sức khỏe duy trì online cùng quy mô phục hồi cho AI |
| CloudFront | Giới hạn về lượng distribution cùng định ngạch số lần gọi lệnh invalidation | Cơ quan tối cao cho luồng tải nộp tệp và nâng cấp giao tiếp frontend |

## Biến và bí mật (Secrets) của GitHub Actions

Quy trình tự động (workflow) hấp thụ thông số cấu hình thông qua các biến môi trường và bí mật (secrets) được gìn giữ trong hệ thống GitHub Environment. Danh sách bên dưới chỉ minh họa tên trường dữ liệu và tuyệt đối không bao giờ được phép phơi lộ giá trị nhạy cảm thật sự trên văn bản công cộng.

### Các biến bắt buộc hoặc thường dùng

| Biến | Mục đích sử dụng |
|---|---|
| `AWS_REGION` | Khu vực đích, mặc định `ap-southeast-1` |
| `AWS_ACCOUNT_ID` | Số định danh tài khoản AWS đích, trích mang sang từ tài khoản thuộc về dự án |
| `EKS_CLUSTER_NAME` | Tên cụm EKS, mặc định `internship-prod` |
| `K8S_NAMESPACE` | Không gian tên Kubernetes namespace, mặc định `internship` |
| `EKS_DEPLOY_ENABLED` | Bắt buộc định giá trị `true` khi có nhu cầu thiển thau triển khai ứng dụng vào cụm |
| `ENABLE_ALB_INGRESS` | Cọc giữ điều phối việc có ban bố bạt cài cấu hình cho ALB Ingress public hay chăng |
| `FRONTEND_DEPLOY_ENABLED` | Bắt buộc gắn bằng `true` mỗi lúc nạp bồi tệp lên cho frontend |
| `CREATE_CLOUDFRONT` | Trao quyền nặn đúc CloudFront distribution mới nếu số định danh cũ khuyết trống |
| `FRONTEND_BUCKET` | Danh xưng hòm lưu S3 của frontend |
| `CLOUDFRONT_DISTRIBUTION_ID` | Mã định danh duy nhất (ID) của CloudFront distribution |
| `ALB_DNS` | Chuỗi danh xưng miền định hướng của ALB origin dùng lúc kết dính ngược vào trong |
| `ECR_REPOSITORY_BACKEND` | Tên kho lưu trữ ảnh cho backend ECR, mặc định `internship-backend` |
| `ECR_REPOSITORY_CHAT` | Tên kho lưu trữ ảnh cho dịch vụ chat ECR, mặc định `internship-chat` |
| `ECR_REPOSITORY_AI` | Tên kho lưu trữ ảnh cho dịch vụ AI trên ECR, mặc định `internship-ai` |
| `AI_DEPLOY_ENABLED` | Công tắc cho phép triển khai pod trung gian adapter cho AI |
| `PROCESSING_WORKER_ENABLED` | Quyết đoán lệnh dãn bẻ tăng số lượng replica cho cụm worker |
| `SAGEMAKER_ENDPOINT_NAME` | Tên máy chủ suy đoán SageMaker endpoint, mặc định `internship-qwen3-4b` |
| `IRSA_ROLE_ARN` | Định vị chú thích IAM Role dành cho pod bên trong runtime |

### Các bí mật (secrets) bắt buộc

| Bí mật | Mục đích sử dụng |
|---|---|
| `AWS_ROLE_TO_ASSUME` | Vai trò IAM Role an ninh để GitHub OIDC nhận diện hoán vị quyền |
| `SECRET_KEY` | Khóa bảo mật cốt lõi dành riêng cho quá trình sinh khóa JWT ở lớp backend |
| `DATABASE_URL` | Chuỗi kết nối trực diện với máy chủ cơ sở dữ liệu PostgreSQL |
| `REDIS_URL` | Chuỗi kết nối trực diện tới cụm nhớ bộ đệm Redis |
| `OUTBOX_QUEUE_URL` | Chuỗi định hướng dẫn nhập trúng vào hàng đợi chủ lực của SQS |
| `AI_SERVICE_API_KEY` | Khóa xác nhận chéo (nếu bật) cho lời gõ cửa từ worker đi sang dịch vụ AI |

## Lệnh kiểm tra và xác minh

Thực hiện nhấp chạy loạt câu lệnh dưới đây (buộc phải bấu kèm uy quyền định danh tài khoản hợp pháp):

```bash
aws sts get-caller-identity
aws configure get region
aws eks update-kubeconfig --name internship-prod --region ap-southeast-1
kubectl get namespace internship
kubectl get nodes -o wide
kubectl get deployments -n internship
```

Kiểm tra định danh và các con số phiên bản công cụ trên máy:

```bash
git --version
aws --version
kubectl version --client
helm version
docker version
node --version
npm --version
python --version
jq --version
hugo version
```

## Kết quả mong đợi

- Số ID của tài khoản AWS thuộc về tài khoản chuyên quản của dự án và nhất thiết phải được bôi nhọ che giấu kỹ từ mọi bức ảnh hay tệp log ném công khai.
- Khu vực thao tác được khai báo nằm chuẩn xác tại vùng `ap-southeast-1`.
- Trình lệnh `kubectl` thành công thám thính và xem xe được trạng thái không gian namespace `internship`.
- Luồng thi công từ GitHub Actions múa kiếm an toàn hoán vị thọ nhập vai trò deploy thông qua bộ giáp OIDC.
- Mọi mảng thông tin bí mật cốt tử đều nằm cẩn nhẫn im lặng sâu thẳm ở hòm giấu secret của GitHub environment, cấm tuyệt lộ diện ra tập code ban đầu.

## Các lỗi thường gặp

| Triệu chứng | Nguyên nhân | Hướng khắc phục |
|---|---|---|
| Lỗi từ chối quyền `AccessDenied` trong tiến trình triển khai | OIDC role thiếu bấu mất thông số phân chia quyền lực thi thoảng ở các API rạch ròi | Bồi nạp thêm chi tiết cụ thể hành vi tác động vào IAM Policy của Role, gìn vững ranh giới đặc quyền tối thiểu |
| `kubectl` móc trật sang cụm máy lạ hoắc không như ý | Cọc kube context được chọn hiện thời chẳng nằm đúng mạn cụm production | Kêu gọi thi lệnh `kubectl config current-context` rồi tái lập kubeconfig |
| Công tác triển khai frontend bị bỏ khinh không trôi qua | Khóa `FRONTEND_DEPLOY_ENABLED` bị cài sai bằng false, hoặc bị vô tâm gắn theo kiểu biến cục bộ không cho phép bộ lệnh rào `if` gặm nhấm | Chỉnh sửa gài chuẩn thông số vào trong hệ cấu hình biến repository hoặc môi trường GitHub đúng tầm bắt của workflow |
| Khâu dãn tải worker lầm lỗi trượt dốc | Bật thông số `PROCESSING_WORKER_ENABLED=true` giữa lúc AI service vẫn mờ xịt nằm lê chơ chưa trỗi sắm xong sức khỏe | Chỉ ra quyết lệnh kích hoạt worker sau mốc cọc dịch vụ AI cùng máy chủ SageMaker endpoint thi thố rực rỡ trọn trạng thái sắm sửa xong |
| Thông báo `jq is required` trỗi khóc | Công cụ `jq` bốc sa biến vắng trong cỗ máy phát triển hiện trăng | Cài đặt sung túc trình biên giải `jq` trước phút giục gọi chạy kịch bản CloudFront ngay ở máy địa phương |

## Kết luận

Toàn cảnh bộ môi trường hạ tầng nay chính thức thỏa mãn 100% điều kiện chín múa để tiến thẳng sang chặng sửa sắm chuẩn bị mã nguồn cùng thi thao triển khai, ngay khi mọi đầu cọc quyền bính tài khoản, kho tệp công cụ, cấu trúc biến an ninh OIDC, tập mật tín secrets, giới rào hạn ngạch quotas và dải vạch kết nối Kubernetes đều kiên kỵ băng đọ thông suôn thành toàn bước thẩm định.
