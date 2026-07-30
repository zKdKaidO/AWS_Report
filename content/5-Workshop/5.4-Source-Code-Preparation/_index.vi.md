---
title: "Chuẩn bị mã nguồn"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Chuẩn bị mã nguồn

## Mục tiêu

Chuẩn bị kho lưu trữ mã nguồn (repository) của ứng dụng ở trạng thái sẵn sàng để phục vụ công tác rà soát kiểm định và thực hiện quy trình triển khai lặp lại suôn sẻ mà không vô ý sơ sẩy để lọt lộ các thông số bí mật hay thi ẩu ảnh hưởng lầm sang tài nguyên sản xuất production.

## Bối cảnh kiến trúc

Kho chứa repository gánh trọn vây kiến trúc cho một mô hình ứng dụng đa dịch vụ (multi-service application). Hệ thống liên lạc CI/CD sẽ tự chủ chia nhỏ và gõ lệnh tiến hành cất dựng riêng biệt từng tệp image cho backend, dịch vụ chat, và (khấu nối nếu bật) dịch vụ AI, tiến ra tung tản ban bố các bản mô tả kịch bản (Kubernetes manifests), đồng thời cho thi hành build tĩnh phân chia phần việc cho frontend ra một nhánh ranh giới độc lập chuyển tản cho kho bộ đôi S3 kết hợp CloudFront.

## Cấu trúc repository

| Đường dẫn (Path) | Mục đích |
|---|---|
| `backend/` | Nơi ngụ của dịch vụ FastAPI, cấu hình dữ liệu SQLAlchemy models, tệp di cư cơ sở dữ liệu Alembic migrations, mã lệnh cho cụm worker cùng tập test nghiệm thu |
| `frontend/` | Không gian trang ứng dụng React/Vite, các bộ kết nối API client, khối thành phần route con cùng dải code test của frontend |
| `chat-service/` | Dịch vụ đàm thoả vận hanh qua Node.js, Express, Socket.IO, các lớp tiếp giao cơ sở dữ liệu DynamoDB repositories cùng tệp trung gian Redis adapter |
| `ai_service/` | Cụm dịch vụ đệm adapter chạy trên FastAPI chuyên phụ trách gọi qua SageMaker cùng luồng mã đọc hiểu và chấm điểm AI |
| `k8s/app/` | Hệ bản vẽ manifest gốc tạo lập Kubernetes namespace, các thông số service, deployments, jobs, dãn dẻo HPA và ngân sách bồi thăng PDB |
| `k8s/eks/` | Hệ manifest đặc thù chuyên gài cất cho môi trường EKS bao gồm ConfigMap, ServiceAccount, ALB Ingress cùng minh họa mẫu cho secret |
| `k8s/observability/` | Hồ sơ khai báo quy cho luồng giám sát qua hai tệp kịch bản ServiceMonitor và PrometheusRule |
| `scripts/k8s/` | Tập kịch bản tự động hỗ trợ cho thao tác triển khai cụm tại chỗ hoặc bủa giăng ra EKS |
| `scripts/ci/` | Thư viện tệp phụ giúp dây chuyền tác nghiệp trên GitHub Actions |
| `scripts/aws/` | Dải script thao tác trợ lực tài nguyên CloudFront, ALB, hàng đợi SQS, quản trị cuộn bản phát hành (rollout) cùng chăm node group |
| `.github/workflows/cicd.yml` | Linh hồn định danh điều hành trọn dây chuyền thanh tra rà soát và phát triển tiến thẳng cho môi trường production |
| `docs/architecture/` | Thư mục tài liệu quyết định kiến trúc (ADRs), luật kiểm soát cạnh tranh song song cùng cam đoan hợp đồng thi hành người tiêu dùng sự kiện |
| `Dockerfile.ai-service` | Bản phác vẽ đóng gói container image cho dịch vụ trung gian AI adapter |
| `docker-compose.yml` | Cột mác khởi tung lập trình bủa quanh các tài nguyên phụ thuộc khi chạy thử tại chỗ |

## Nhận mã nguồn dự án được cấp quyền

Thao tác thực hiện truy nạp vào kho repository chỉ được phép diễn ra trên cơ quan tài khoản đã sắm sửa ủy quyền hợp pháp:

```bash
git clone https://github.com/Temp-orgo/AWS-Internship.git
cd AWS-Internship
git status --short --branch
```

Tôi nghiêm cấm việc rò rỉ bủa lan mọi token bảo mật, chứng chỉ đăng nhập nhân chứng hoặc những địa chỉ URL từ xa mang gài theo khóa xác thực bên trong hồ sơ cẩm nang báo cáo này.

## Kiểm tra nhánh và trạng thái workflow

```bash
git remote -v
git status --short --branch
git log --oneline -5
```

Kết quả mong đợi:

- Cọc địa chỉ mạng remote kết trói chuẩn xác về kho mã nguồn có bản quyền đích xác thuộc dự án.
- Tình trạng cây mã nguồn làm việc (working tree) được thấu đáo 100% ranh giới ngay trước chuỗi búng ra lệnh triển khai.
- Cân mẫn cẩn khôn giữ vẹn toàn, ngăn cản trót lọt việc sửa đè nhầm lên các thay đổi nội bồi địa phương không mang dính líu đến dự án.

## Xác định các thư viện phụ thuộc của dịch vụ

### Backend

```bash
cd backend
python -m pip install -r requirements.txt
python -m ruff format --check .
python -m ruff check .
python -m mypy app tests
python -m pytest -m "not postgres"
```

Dịch vụ backend tích hợp ranh giới chạy nền Python 3.12 trong dây chuyền CI, vận dụng bộ khung FastAPI, thư viện SQLAlchemy, trình di cư Alembic, bộ kiểm tra thuộc tính Pydantic settings, kho móc AWS boto3, cỗ quan trắc Prometheus client cùng thư viện truy vết OpenTelemetry.

### Dịch vụ chat

```bash
cd chat-service
npm ci
npm run check
npm run lint
npm run format:check
npm run test:ci
```

Dịch vụ chat thiết dựng xoay xung quanh Node.js, framework Express, giao thức thời gian thực Socket.IO, kho lưu DynamoDB, cầu nối Redis adapter, cùng nhịp tim số liệu giám sát OpenTelemetry và Prometheus metrics.

### Frontend

```bash
cd frontend
npm ci
npm run test
npm run build
```

Frontend được dệt nên từ quy chuẩn React 18, trình tăng tốc Vite, thư viện CSS Tailwind CSS, bộ tiếp thu liên lạc Socket.IO client, công cụ gọi mạng Axios cùng tầng tầng các trạm giao tiếp API client cho từng tuyến đường riêng.

### Dịch vụ AI

```bash
docker build -f Dockerfile.ai-service -t internship-ai:ci .
```

Bộ trung gian AI adapter tiến hành phơi mở ra bên ngoài các con đường dẫn `/health/ready`, `/parse-job`, `/parse-cv`, `/match-applications`, `/rerank`, `/cv-job/group-score`, kết hợp cùng một chuỗi các luồng chấm điểm tương đương. Tại môi trường thực tiễn production, nó sẽ gọi sang bộ máy SageMaker Runtime thực thụ thay vì nặng nề lôi vác nạp trọn cỗ máy model khổng lồ về ngự ở trong các worker.

## Mẫu tập tin cấu hình môi trường

Tận dụng những tệp thông tin ví dụ có sẵn để coi như là nguồn tra khảo tham chiếu chuẩn mực:

| Tập tin | Mục đích |
|---|---|
| `k8s/app/secret.local.example.yaml` | Tệp ví dụ minh họa thông số secret cho cụm Kubernetes tại chỗ |
| `k8s/app/secret.local.yaml.example` | Mẫu bản vẽ secret tại chỗ cho nhà phát triển |
| `k8s/eks/secret.example.yaml` | Hình hài cấu trúc cho các khóa secret bên trong EKS mà không để lộ nội dung giá trị thật |
| `backend/requirements.txt` | Thư viện mã nguồn phụ thuộc phục vụ runtime backend |
| `frontend/package.json` | Hệ script tự động hóa cùng thư viện cho frontend |
| `chat-service/package.json` | Cụm script vận hanh và thư viện đi kèm của dịch vụ chat |

Tuyệt đối cấm đưa lên (commit) các tệp biến thực thể `.env`, `.env.production`, file cấu hình kubeconfig, chuỗi url kết nối database, khóa mã bảo an JWT secret, khóa truy nhập AWS, mã thông báo token GitHub, private key riêng hay các mật mã dịch vụ phát mail SES.

## Lệnh rà soát kiểm định tại chỗ

Luồng dây chuyền tự động CI trên mạng có thói quen cho thi triển các chốt trạm tra rà quan trọng như sau:

```bash
bash -n $(git ls-files '*.sh')
python scripts/ci/infrastructure.py
python -m ruff format --check backend
python -m ruff check backend
python -m mypy backend/app backend/tests
python -m pytest -m "not postgres" backend
npm run test --prefix frontend
npm run build --prefix frontend
npm run test:ci --prefix chat-service
docker build -t internship-backend:ci ./backend
docker build -t internship-chat:ci ./chat-service
docker build -f Dockerfile.ai-service -t internship-ai:ci .
git diff --check
```

Trong những kỳ máy trạm lập trình địa phương của bạn thiếu khuyết trình quản trị Docker, hãy thật thà ghi nhận việc thử nghiệm build đóng ảnh Docker là chưa chạy rà kiểm định thay vì ngoan cố tự tung tự mác đánh dấu là đã hoàn thành công xuất sắc.

## Chuẩn bị cho triển khai

Trước thời khắc khởi búng kích hoạt dây chuyền sản xuất production:

1. Thẩm tra an tâm các biến môi trường và bí mật (secrets) production của GitHub đã thọ vị nằm ngắn gũi đầy đủ theo yêu cầu.
2. Xác minh vai trò IAM Role của OIDC là `internship-github-deploy` sẵn lòng cho khoắc nhãn hoán quyền thâu vãng bên trong đúng số tài khoản AWS dự án.
3. Kịp thời đảm bảo các kho lưu container ECR hiện hữu sẵn, giăng trọn chỗ nạp cho backend, chat và cho AI (trong tình huống công tắc triển khai AI service thêu bật).
4. Xác minh tinh thần trỗi múa trọn vẻ của một hệ sinh thái gồm RDS, Redis, DynamoDB, SQS, S3, CloudFront, ALB, Lambda, SES cùng SageMaker.
5. Nghiêm rào kiên định giam ngục tham số `PROCESSING_WORKER_ENABLED` ngả ở mức false cho tận mốc ngày cỗ máy suy luận AI bám kèm rúng còi thông quan.
6. Chọn bấu nấc `deploy-app` để thao diễn đưa mã app lót vào EKS và ấn trượt chế độ `deploy-frontend` cho công tác thiêu tung giao diện lên cao.

## Các chế độ của quy trình GitHub Actions

Cấu hình bộ thi kịch bản (workflow) được khảo soát ra lệnh hậu thuẫn cho chuỗi các cơ cấu tác nghiệp linh động như:

| Chế độ | Mục đích |
|---|---|
| `validate` | Thực thi kiểm định an nộ không để xô bồ trát dội sang cụm production |
| `deploy` | Con đường đưa bản thử ra thực tiễn theo lối mòn truyền thống nay vẫn được workflow khoan nhượng chấp thuận |
| `rollout` | Gõ chuông tái khởi động (restart) trọn bộ các workload trên EKS mà chẳng nhọc sức re-build các ảnh |
| `restore-compute` | Cám ơn gọi đánh đòn chấn hưng dãn sắm lại trọn dung nấc năng lực cụm máy managed node group |
| `deploy-app` | Dựng nạp ảnh mới đẩy sang kho ECR đồng thời khai phá tiến vào EKS thuyên chuyển các workload |
| `deploy-public` | Bung trượt bản ghi manifest cắm neo ALB Ingress ra thế giới bên ngoài |
| `deploy-frontend` | Build kho mã frontend, đồng bộ thư viện lên kho S3, kêu gọi lệnh xóa bộ đệm (invalidates cache) trên CloudFront |
| `full` | Tự động thi khát khao nối liền chu trình sản xuất khép kín một hơi ráo rã trọn bộ từ A sang Z |

## Kết quả mong đợi

Kho tàng mã nguồn tựu chung đạt tiêu chí chốt duyệt thành quả khi mà:

- chuỗi tài liệu thư viện bấu vây các dịch vụ thảy đều hoan hỷ tải nạp mượt trôi
- trọn các bài kiểm định của backend, frontend cùng dịch vụ chat rềnvang đắc thắng băng đọ qua khỏi không gian test quy đúc
- các container image Docker sẵn lòng chịu gõ trúng cấu hình bạt xếp thành quả
- những trang manifest mô tả cho Kubernetes xuất xưởng không trầy móng xô xát lọt rào châm trích từ cỗ máy kiểm toán script repository
- thông số biến môi trường cùng chuỗi khóa mật secret bên phía GitHub ngạo nghễ điểm tên thọ sinh đầy đủ
- luồng ra mắt hệ thống được rành rẽ phân thân rõ mạn: workload nghiệp vụ thẳng đường thâm ngục EKS, còn giao diện web lả buông phi thăng ngụ kho S3 bấu trói CloudFront

## Các lỗi thường gặp

| Triệu chứng | Nguyên nhân | Hướng khắc phục |
|---|---|---|
| Lên tiếng ca than `Cannot find build-and-push-image.sh` | Chẳng thể dò tìm ra phương mạn cho tập script trợ giúp | Cho lệnh trỏ múa bám chuẩn đường dẫn `scripts/build-and-push-image.sh` hoặc ra quyết gán cấu hình biến `BUILD_IMAGE_SCRIPT` |
| Hỏng gục bước tải file tiêu đề Node (Node header) trong tiến trình CI cho dịch vụ chat | Khâu dịch mã tệp phụ thuộc bản địa (native dependency) rền nài thèm muốn chuỗi header cài ngay sở tại | Cần mẫn nạp dải cấu hình biến `npm_config_nodedir` nhắm trúng vào mạn thư viện headers mà hệ thống ci toolcache lưu cẩn kiệt |
| Trang frontend build ra bốc sa mù chạy sai về mạn localhost | Mang tiếng thất đức bỏ quên tham số biến cố định `VITE_API_BASE_URL=/api` cho luồng production | Nhờng kính nạp trang trọng các thông số biến Vite ở ngay thời khắc nháy còi chu trình build tĩnh phục vụ production |
| Đóng cửa cự tuyệt lệnh gõ docker build | Tiến trình điều hành Docker daemon thiêm đố ốm chết im lì giữa máy | Kích ngục hâm ấm gọi còi giục Docker daemon đứng dậy múa kiếm, hoặc trang nghiêm chốt từ ngữ khai báo khâu test build chưa qua thực thi |
| Các bí mật (secrets) kinh hãi hiện diện vọt nhảy giữa trang báo rách git diff | Sơ sót ngập bùn bỏ vây đưa file `.env` hoặc tệp bí mật sinh ra tạm bở trượt vọt lên mạn staged | Tháo dọn và đá phăng các tệp bùi bám khóa bảo an xa khỏi mác commit ra lệnh tiễn còi tráo xoay mới các thông số khóa đã phơi bãi ra xa |

## Kết luận

Thư mục chứa đựng mã nguồn ứng dụng qua chặng kiểm nghiệm gay gắt nay chính thức cán đích sẵn sàng để ra lệnh xuất phát triển khai theo khuôn nấc quy an nộ bảo mật cao cấp nhất. Mạch phân phối giao diện web buông rẽ thành chặng bay tiễn tĩnh riêng cho S3 và CloudFront, trong khi khối workload EKS an nhiên duy trì lộ trình của mình, đồng loạt vây hãm bảo an cẩn khôn nhờ sức khỏe dây chuyền quản trị thông lượng xác thực GitHub Actions phối hợp cùng tấm khiên OIDC.
