---
title: "Bảo mật và Chi phí"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

## Mục tiêu

Tài liệu hóa chi tiết toàn bộ các biện pháp kiểm soát bảo mật và phương pháp tiếp cận ước toán chi phí dành cho hạ tầng AWS đã được bủa giăng ra môi trường production thực tế.

## Bối cảnh kiến trúc

Nền tảng ứng dụng gánh tải xử lý và thuyên rã muôn thông tin cốt lõi bao gồm tài khoản người dùng, tài liệu CV, đơn ứng tuyển, thư tín đàm thoại chat, luồng phân tích suy luận AI, báo còi thông cảm sự kiện, cùng chuỗi thư tín email. Hệ biện pháp rào chắn an nộ bảo mật bị ép buộc gánh sứ mệnh ôm bế và che chắn trọn dải cho danh xưng xác thực, các thông số mật mã bí mật (secrets), dữ liệu riêng tư, quyền vai trò bấu trong chu trình triển khai, kho cất trữ, những vạch cầu định tuyến mạng và khâu múa rẽ xử trí tin sự kiện. Trong một thế trận giáp kiềm, bài toán kìm giữ chi phí bắt buồng dốc tâm lực trông canh gắt gao nhĩ đàn hạ tầng thọ sinh thói quen ngụ trọ 24/7 không hở trần như: cụm máy EKS, trạm giao cẩu NAT Gateway, kho RDS PostgreSQL, cụm ElastiCache Redis, bộ rẽ ALB, đi cùng cõ máy suy luận AI SageMaker.

## Các biện pháp kiểm soát bảo mật

| Khu vực | Biện pháp yêu cầu hoặc đã triển khai | Trạng thái |
|---|---|---|
| GitHub OIDC | Tiến trình GitHub Actions thông quan hoán quyền bấu vào AWS role thông qua giao thức OIDC với chuỗi tham cờ `id-token: write` | Đã triển khai trong workflow |
| Khóa truy nhập AWS kiên bám dài hạn (Long-lived AWS keys) | Tuyệt đối chối bẻ việc găm giam các chìa khóa truy cứu quyền hạn AWS vô hạn kỳ bên trong cối GitHub hay trong hồ sơ mã nguồn | Biện pháp bắt buộc (Required control) |
| Quyền hạn IAM tối thiểu (IAM least privilege) | Dải phân vai cho triển khai, thời gian chạy runtime, các con node, SageMaker, Lambda hay trình quản trị ALB cần thi nặn ở mức tối giản theo thực tế thao tác | Đã áp dụng ở cấp độ kiến trúc; cọc minh chứng tài liệu chính sách policy vẫn ráo đợi |
| IRSA | ServiceAccount của Kubernetes `internship-app` có khả năng được gán nhãn annotation với vai trò runtime IAM | Đã triển khai trong tập lệnh script |
| Kubernetes secrets | Khóa cọc `SECRET_KEY`, `DATABASE_URL`, `REDIS_URL`, `OUTBOX_QUEUE_URL`, và (tự do tùy nhĩ) khóa AI API key thảy đều được bơm tiêm vãng trú vào trong `internship-secrets` | Đã triển khai trong kịch bản script |
| Kho lưu S3 riêng tư cho frontend (S3 private frontend bucket) | Khóa chốt phong tỏa mọi ý định trần lộ cho thế giới public mở vào bucket, thi chuyển nạp theo cộc quyền CloudFront OAC | Đã triển khai trong `ensure-cloudfront.sh`; cọc chứng cứ thao thấu runtime ráo đón nạp bồi |
| Giao thức HTTPS trên CloudFront | Cống mạng CloudFront chủ động ra trát ép điều khiển lượt tra cập của cộng đồng phải qua ranhHTTPS an nộ | Đã triển khai trong cấu hình phụ trợ |
| Kiểm soát tuyến đường ALB | ALB buông rào cản ngăn kiên định, thọ cho lọt cối đi vào cụm backend/chat đối với đúng nhĩ 3 tuyến đường `/api`, `/chat`, `/socket.io` | Đã triển khai trong manifest Ingress |
| Truy cập kín cơ sở dữ liệu RDS | PostgreSQL được đặt sau security group riêng và không mở inbound trực tiếp từ Internet | Đã xác minh: `sg-07bb82c3c3c31b61e` chỉ cho phép TCP `5432` từ `sg-06f3c7732550ce8fd` |
| Truy cập riêng cho Redis / Valkey | ElastiCache / Valkey không mở truy cập public | Đã xác minh: `sg-0405e9a0dbadd61af` chỉ cho phép TCP `6379` từ EKS SG |
| Mã hóa dữ liệu SSE cho SQS | Hàng đợi chủ lực lẫn con bãi lỗi DLQ nghiêm gạt công tắc mã hóa bạt theo tiêu chuẩn máy chủ SSE | Đã xác minh qua hồ sơ chứng cứ Tuần 8 |
| Mã hóa dữ liệu DynamoDB | `ChatGroups`, `ChatMessages`, `ChatUsers`, `InternshipLambdaEventDedupe` dùng encryption với AWS Owned Key | Đã xác minh; PITR hiện tắt trên tất cả bảng |
| Mã hóa RDS (RDS encryption) | Cả `internship-prod-postgres` và `internship-tracker-db` bật KMS encryption | Đã xác minh |
| Mã hóa đường truyền (Encryption in transit) | Luồn HTTPS trót lọt qua CloudFront; tuy nhĩ thông số bấu rào cho giao thức an ninh TLS chốn các vi dịch vụ và trạm cơ sở dữ liệu nội bộ thi ra lời nài bằng chứng | Đã chứng minh một phần |
| Quản lý và cách ly bí mật (Secret handling) | Nghiêm cấm gõ lệnh commit hão đưa bất kỳ file `.env`, mã xác thực token, chìa khóa, mật khẩu hay đường dẫn dữ liệu DB lên repository | Biện pháp bắt buộc |
| Hàng đợi thư hỏng DLQ | Hạ tầng SQS DLQ khôn ngoan tựu chốt gom nhốt riêng mác các tin nhắn bị thợ consumer xô thất bại liên miên | Đã xác minh qua hồ sơ chứng cứ Tuần 8 |
| Tính bất biến (Idempotency) | Khấu chốt bất biến tại PostgreSQL, bảng outbox, nhĩ cờ `clientMessageId` từ mạn chat cùng hào cống khử lặp DynamoDB từ phía Lambda | Đã triển khai; riêng luồng rà test khói Lambda hoan thọ xác minh |
| Sao lưu và điểm khôi phục PITR (Backup and PITR) | RDS có backup retention; DynamoDB PITR chưa bật | Đã xác minh: `internship-prod-postgres` giữ backup 7 ngày, `internship-tracker-db` giữ 1 ngày; PITR DynamoDB đang tắt |
| Gia cố rào cản cấm chui thẳng bãi ALB | Kịch bản tự nguyện kêu gọi việc trói nghẹt bãi ALB origin sao cho thọ nhĩ thấu lời gọi duy nhất từ đằng CDN CloudFront | Đề xuất gia cố (Optional hardening) |
| Tường lửa AWS WAF | Trù cất chiêu huy động thêm AWS WAF án cữ ngay trước cánh CloudFront mang lại lớp rào bảo an mĩ mãn cho cõi production | Đề xuất gia cố |

## Xác thực và phân quyền (Authentication and authorization)

Backend khai thác nền tảng xác thực tựa theo chứng chỉ thẻ JWT (JWT-based authentication) đi liền rèm khắt khe phân mác định tuyến dựa vào vai trò, dệt thành hào lũy rào cản phân minh rõ chuỗi quy trình thao tác giữa ứng viên (candidate) và bộ phận nhân sự (HR). Bên mạn giao diện frontend, phiên tiếp vãng của khách thọ an ngụ cẩn khôn trong bộ nhớ tại chỗ với phần đầu nhãn là chuỗi ký tự `internship_tracker_` và khôn khéo ra lệnh trút dẹp tiệt mọi thông tin trong kho lưu session/local storage ngay thì khoảnh mác khi chốt phiên cờ rùng còi mãn hạn hết hiệu lực. Cụm các trạm bảo an điều hướng trên mạn frontend chủ trạch tựu bẻ rẽ con đường lọt về cho người dùng đúng theo quy cọ quyền hạn họ thụ hưởng.

Kỳ vọng bắt buộc về an nộ bảo mật:

- Mật khẩu truy cập thi thoảng tuyệt đối phải được chuyển hóa thành bảng mã hàm băm (hashed).
- Chìa khóa bí mật JWT secret khẩn nài ngụ an tịnh trong lòng hòm cơ quan quản trị bí mật, cấm tiệt sa ngã rơi rỗng vào bãi mã nguồn repository.
- Các tuyến giao tiếp cho ứng viên lẫn HR ở chiến tuyến backend buộc thi trát khắt khe đối rà chủ sở hữu dữ liệu đi đôi cọ thẩm sát vai trò quyền hạn.
- Những điểm số do cơ quan trí tuệ AI suy bạt thi ra ranh mục chỉ sắm vóc dáng lời khuyến cẩu, cự tuyệt việc bấu bão coi đó làm cơ quan định trát phán xét quyết định duy nhất trong tiến trình thọ nạp tuyển dụng personnel.

## Bảo vệ dữ liệu

| Loại dữ liệu (Data type) | Yêu cầu bảo vệ |
|---|---|
| CV và hồ sơ văn kiện ứng tuyển | Cất nạp bên trong lòng S3 riêng tư, thẩm định danh tính ra vào ráo riết, khước nhục mọi cơ quan in log bung trọn vẹn văn bản raw thô |
| Hồ sơ ứng tuyển (Applications) | Ghi nhận chễm trệ trong cối PostgreSQL có áp chuỗi rà soát chủ quyền tài nguyên khắt khe |
| Tin nhắn đàm thoại chat | Gạt nạp trọn trong DynamoDB, trao ranh rào thẩm tra visa thông vãn cho chính vi dịch vụ chat service can thiệp |
| Thông tin sự kiện outbox (Outbox events) | Cẩn mật né hẫng việc nhả nhật ký log vãi lộ trường payload chứa tin nhạy cảm ở thì hiện đại và tương lai; trỏ lưu bãi archive đúng trần văn thư tối thiểu cần cho nghiệp vụ |
| Các câu lệnh cẩu rào gửi cho AI (AI prompts) | Nghiêm răn không gõ log phô bày trọn nội sung CV của con người; bộ đệm trung gian AI chỉ bọc rẽ in ra chữ ký hàm băm (hash) và con số đếm lượng ký tự trong chuỗi prompt |
| Bí mật (Secrets) | Giam yên tịnh trong hòm bí ẩn GitHub secrets phối hợp cọ Kubernetes Secret; vĩnh viễn bẻ bãi không vãi phơi xuất ra văn kiện bao giờ |

## Lệnh kiểm tra bảo mật

```bash
aws iam get-role --role-name internship-github-deploy
aws iam get-role --role-name internship-eks-runtime
kubectl get serviceaccount internship-app -n internship -o yaml
kubectl describe secret internship-secrets -n internship
aws s3api get-public-access-block --bucket <FRONTEND_BUCKET_NAME>
aws sqs get-queue-attributes --queue-url <OUTBOX_QUEUE_URL> --attribute-names All --region ap-southeast-1
aws rds describe-db-instances --db-instance-identifier internship-prod-postgres --region ap-southeast-1
```

Tôi xin thề kiên trinh chối từ, không nhã nhặn in thọ hay gõ sao chép trút đổ các chuỗi giá trị mật mã secret tự trong Kubernetes, mạn GitHub hay hạ tầng AWS trườn ngã lộ ra ngoài các mặt giấy trong hồ sơ tài liệu của báo cáo này.

## Nguồn bằng chứng về chi phí

Phần chi phí dùng bằng chứng AWS Budgets do người dùng cung cấp. Đây là trạng thái ngân sách tại thời điểm trích xuất bằng chứng, không phải bảng giá tự suy đoán và không thay thế hóa đơn AWS cuối kỳ.

| Budget | Giới hạn | Thực tế | Dự báo | Trạng thái |
|---|---:|---:|---:|---|
| `Monthly` | `$100` | `$0.00` | `$17.26` | HEALTHY |
| `My Monthly Cost Budget` | `$100` | `$156.02` | `$173.28` | EXCEEDED |

Budget `My Monthly Cost Budget` đã vượt giới hạn `$100`, với chi phí thực tế `$156.02` và dự báo `$173.28`. Vì vậy báo cáo không nên nói chi phí đang ở mức an toàn nếu dùng budget này làm nguồn kiểm chứng.

## Các giả định khi tính toán chi phí

Không đưa giá theo giờ hoặc tổng chi phí tháng nếu chưa có AWS Pricing Calculator export hoặc AWS Bills/Cost Explorer đủ kỳ. Các cấu hình có thể dùng làm đầu vào cho bảng dự toán gồm:

| Cost driver | Cấu hình đã xác minh | Ghi chú |
|---|---|---|
| VPC / NAT Gateway | `nat-16294630aebc49598`, 3 Elastic IP đang gắn | NAT Gateway là chi phí cố định đáng chú ý; có thể cân nhắc giảm số AZ nếu không cần HA |
| Elastic IP chưa gắn | `13.251.12.233` (`eipalloc-0a9b85e5ce92d2c02`) | Nên release nếu không dùng |
| RDS PostgreSQL | Hai instance `db.t4g.micro`, mỗi instance 20 GB | Cần xác nhận có cần giữ cả `internship-prod-postgres` và `internship-tracker-db` không |
| ElastiCache / Valkey | `cache.t4g.micro`, encryption at rest và transit enabled | Chi phí chạy liên tục theo node |
| EKS | Cluster `internship-prod` và workload backend/chat/AI/worker/dispatcher trong EKS | Không tính frontend vào EKS vì frontend chạy trên S3 và CloudFront |
| SageMaker | Có log group `/aws/sagemaker/Endpoints/internship-qwen3-4b` | Cần kiểm tra endpoint hiện có đang idle không |
| CloudWatch Logs | Log groups đã xác minh cho EKS, Lambda, SageMaker và VPC Flow Logs | Cần cấu hình retention nếu muốn kiểm soát phí lưu log |

## Bảng ước tính chi phí

Chưa có bảng AWS Pricing Calculator export hoặc hóa đơn AWS cuối kỳ đủ để chốt dự toán chi phí dài hạn. Phần hiện tại chỉ ghi nhận cost drivers, budget status và các hạng mục tối ưu được đề xuất dựa trên bằng chứng đã cung cấp.

## Vệ sinh tài liệu bằng chứng (Evidence hygiene)

Trước thời điểm ra quyết cọc đem bất cứ văn ảnh chụp mạn hình nào cõi AWS, hay chép đệm một luân nhật ký log nào bồi thẳng vào lòng báo cáo này, tôi luôn giữ một nguyên tắc sắt đá tuyệt trần: bóp gi giếm ẩn trót lọt trọn vẹn số định danh tài khoản AWS (account ID), chuỗi chìa khóa truy nhập access keys, secret keys, mật khẩu vãng cõi cơ sở dữ liệu, con đường kết nối connection strings, thông ký thọ nằm chốn Kubernetes Secret, mã thẻ thông quan GitHub token, đường link chia cẩu chuế ký tạm (presigned URLs), và toàn bãi bất cứ cõi tệp log nào bồi mang cõng cọc thông ký bảo mật.

Cuộc tổng thám sát soát cẩu kho bãi bằng chứng của tôi tuy tự hào không bão tuột lộ ranh giới cọc chìa khóa access keys hiển trần hay token từ GitHub nào ra văn bản sa mác thô hỏng, song chính trong ranh đường dẫn tệp `02-eks/pod-logs-tail.txt`, tôi bồi thót tim tóm trúng cọc một chuỗi kết nối Redis cõng trọn theo thông tin xác thực nằm sa gục bên giữa dòng nhật ký lúc khởi chạy (startup logs) của `chat-service`. Để cự tuyệt án phạm rào cản an ninh, tôi kiêu dũng chối cự việc sao phó nạp tuệ nguyên bản tệp thô thảm rách đó lên. Tôi chỉ đệm vãi đưa ranh giới câu rẽ tường trình tổng quan, hay nhã nhặn hoán gõ lật nhĩ thế chỗi toàn văn cọc nhạy cảm đó thành từ khóa an sinh trung lập `<REDACTED_REDIS_CONNECTION_STRING>`.

## Các hạng mục hao phí tài nguyên lớn nhất

Cơ quan cất trọ hàm chứa nguy cơ gieo bão ngốn gặm chi phí cao nấc nhất trong chặng đường tiếp chặng là:

- Số giờ mở máy thọ múa không nghỉ trọ cõi SagMaker real-time endpoint, vô cùng kịch khốc nếu cọc EC2 gán nuôi trạm endpoint sa vào loại máy chuyên dùng năng lực GPU.
- Cụm điều phối EKS (control plane) và lực lượng 2 trạm EC2 worker nodes nạp mình chay liên tục ngày đêm cõi 24/7.
- Năng lực phục vụ cống hiến lưu trữ thường trực được cung phụng từ hai tay RDS và ElastiCache Redis.
- Cước thu chi phí theo giờ cho ALB kết cọc cùng khối hao tổn cọc lượng đếm đơn vị LCU.
- Hao phí duy trì CDN CloudFront đi với chi cước băng thông đường truyền (data transfer) một khi lưu lượng cộng đồng kéo theo bọt lớn dốc ngược lên.
- Phí tổn nuốt log vào kho CloudWatch và phí chôn bảo lưu retention nếu như hệ luôn ráo lệnh ép bật cơ chế ghi nhật ký siêu tỷ mẩn (verbose logs).

Khối chứng cứ lộc tài từ Tuần 8 phán bão gắt gao rằng chi cướ thực tế không còn trườn bò xấp xỉ mức con số không ngon ơ nữa. Cụm SageMaker, cơ sở dữ liệu RDS, nhịp cất luồn lưu thông qua mạng VPC/NAT, lực lượng EC2 cho worker node và ranh cụm EKS đương nhiên buộc thọ coi như dàn nguyên nhân hao cước chủ lực múa gánh tải cõi trần. Nhất tâm ghi mác nhĩ kịch bản bám lệnh cất còi cho tắt vãn, hoặc cẩu lệnh chôn xóa sạch SageMaker endpoint ngay sau nhịp dãn demo là điều vô cùng tối trọng chừng nào tác nghiệp suy bạt AI realtime không còn bị nài cầu cho thao thọ thực tế.

## Các giải pháp tối ưu chi phí

| Giải pháp | Hiệu quả |
|---|---|
| Chôn xóa hẳn hoặc bồi cài còi đặt lịch auto thi rác cho SagMaker endpoint chặng khi idle thảnh thơi nhàn rỗi | Phá vỡ bẻ xô thành công thủ lĩnh số 1, ngốn hao chi phí kinh bão bạo nhất thuộc mạn cõi dịch vụ AI |
| Gặt nhĩ công tắc khóa kiềm nhôt im processing worker (disabled) bất cứ khúc khi luồng test AI chả kích cho chạy | Triệt rào ngốn phí vô ngã do khâu tự thọ cẩu múa tự gõ thử lại retry nhầm cõi và cắt xô giờ thi cho thợ worker |
| Tinh chỉnh lựa đúng thông cọc cõi (Right-size) cho node group trên EKS | Bồi dãn nhã thâu chi phí trút vào máy cỗ EC2 cùng mạn dung sai đĩa cứng EBS |
| Nháo ép buông dãn bớt (Scale down) lực lượng các workload phi-production sau thì khoảnh mác gõ thi thọ demo | Thu gạt, xóa cắt chi trả không đáng có cho đám tài nguyên máy tính trọ ngụ 24/7 vô công rách thợ |
| Khảo rà nghiêm mẫn tính cấp khẩn của cỗi cống NAT Gateway | Chặn tiệt khoản thu khổng lồ trả theo chuỗi giờ chay NAT và chi phí tính mác trên từng GB dữ liệu thọ bạt qua nó chừng nào mạn riêng tư cấm thọ cầu rào chui xuất Internet |
| Ứng chiêu nạp bổ sung các VPC endpoint ở chốn cung mạn trúng đích thiết thực | Hại thi trút nhã khoản phí cho khâu dữ liệu luân cẩu chuyển rễ qua trạm NAT đối với mạch giao thông chui sang các hạ tầng cõi AWS |
| Áp luật vòng đời lưu chiểu cho S3 (S3 lifecycle rules) | Ra chiêu sa trượt cọc các tệp tải cẩu hay văn thư archive lâu năm bế lui xuồng kho rẻ thọ chi hơn, hay lệnh dập tắt bãi rách xác test |
| Ban chính sách vòng đời cho kho container ECR (ECR lifecycle policy) | Ra lệnh quấy xao tự động tẩy bẻ cạn kiệt dải image mang chữ ký SHA cũ rách sau khi mãn kỳ bảo an |
| Giới rào quy bạt thời gian bảo lưu (retention) cõi CloudWatch | Xóa tan bãi thảm rách duy trì lưu trữ lâu nấc cạn thế kiếp cho một khối log bão táp |
| Áp kịch bản tính cước theothực dùng (on-demand) cho DynamoDB chặng giai đoạn lưu lượng cộng đồng bất bạo gập ghềnh | Ngăn chừa kịch bản xô hỏng hao tài chi bạo vì tự mãn ném ra dải cấp phát dung tài quá mức (over-provisioning) ở kỳ khải mở đầu tiên |
| Thiết lập ngân sách rào AWS Budget đi đôi nhĩ cảnh báo hao cước (billing alert) | Chộp gọn ngay cọc dấu hiệu thu phí bổng thọt dị thường trước khi ngọn lửa cháy rách bổng leo bão chót vót |

## Kết quả mong đợi

Tôi tin tưởng khôn khéo trúng rào cất khải thông nghiệm thu trang sách về Bảo mật và Chi phí khi văn bản phác thấu tường tinh chuỗi rào an nộ bảo mật đã cho dệt thiêu, bẻ gõ kiêm vãng nài rõ các rãnh hở minh chứng còn thiếu khuyết, giữ trọn thanh liêm chối cự cống nhã trần lộ bí mật (secret exposure), đồng thời ra vãng phương pháp lập bão giá tính chi cướ tường tuệ mà thề chối khước muôn suy đoán tự nặn bịa đặt ra những thông số bảng giá theo giờ huyễn hoặc trên AWS.

## Các lỗi thường gặp

| Triệu chứng | Nguyên nhân | Hướng khắc phục |
|---|---|---|
| Tóm vãng cọc khóa AWS kiên trinh bám hạn ngụ giữa lòng cõi CI | Lầm lọt sa thói chép gõ cấu hình xác thực tĩnh thiếp đè vào thay cho luôn hoán quyền OIDC | Bốc trục tháo xóa tiệt cọ con chìa khóa đó đi, thực thi xoay vần khóa mới, và trút bồi sử dụng qua vai trò IAM OIDC role |
| Luông chui vãng kho S3 khơi cõi hớ rộng công khai (direct public access) | Cấu trúc rào quyền hạn của hòm bucket policy hay chốt gác thọ bạt bốc lọt mác sai lệch | Trân cọc thi nặn dựng lại thế giam cản cõi public access block phối cọc cam kết trút quyền cho CloudFront OAC policy |
| Tiến trình Lambda sập r rãnh dội sinh cọc 2 lần tác vụ phụ (duplicate side effects) | Bỏng khuyết cất rỗng cơ quan khử lặp sự kiện (event dedupe) | Yêu cầu buông dùng chiêu thức ghi dữ liệu theo điều kiện trên DynamoDB bấu kiên định theo chuế `eventId` |
| Hóa đơn thu phí cướ phình bạo rợt kinh chấn (unexpected high bill) | Để hớ ngọ bãi chôn quên bẻ cho cụm SageMaker, ranh con NAT hay máy cõi EKS nhở ngọ múa chay vô công cõi | Thanh rà tức thời trong cỗ Cost Explorer và nháy pháo trút gõ xóa hẳn hay cho im tịt ngay các đàn tài nguyên ngốn phí lớn lao đang nhàn rỗi |
| Bảng dự toán chi phí chứa số chưa có nguồn | Chưa có export từ AWS Pricing Calculator hoặc hóa đơn AWS cuối kỳ | Không chốt số dự toán dài hạn; chỉ ghi budget status và đánh dấu dự toán steady-state là công việc đề xuất |

## Kết luận

Toàn cõi quy trình đưa lên AWS tuân thủ một thế trận an an mật dựa trên tín niệm bấu rào cõi chứng cứ (evidence-based security model) kết bấu cùng mô hình tài lộc ngân khoản có tính thực chiến bám đất ráo rã nhất (practical cost model). Mọi thông ký trường biến còn bốc hụt chưa bão chốt trong chặng Bảo mật và Chi phí này đều ráo thọ thi ra lời gõ cầu minh chứng qua các tệp cẩu từ bộ lệnh AWS CLI, bảng theo cẩu cõi Cost Explorer, hoặc văn bảng từ cỗ tính toán AWS Pricing Calculator trước thì thời khắc cho thả bút đóng mác phai trút ra các phó con số khẳng định ráo rã sau cùng.
