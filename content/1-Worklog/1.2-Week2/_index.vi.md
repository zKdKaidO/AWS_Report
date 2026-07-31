---
title: "Nhật ký tuần 2"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Tuần 2 - Phát triển chức năng Candidate-HR và lưu trữ tài liệu với Amazon S3

### Mục tiêu tuần 2:

- Phát triển các chức năng hồ sơ dành cho Candidate và HR/Company.
- Xây dựng các luồng đăng tuyển, tìm kiếm việc làm, nộp hồ sơ và quản lý ứng viên.
- Hoàn thiện cơ chế upload, kiểm tra và phân quyền truy cập tài liệu.
- Tìm hiểu Amazon S3, presigned URL và cách quản lý quyền truy cập tài nguyên bằng IAM.
- Làm quen với AWS CLI trong quá trình kiểm tra bucket và object storage.

### Các công việc thực hiện trong tuần:

| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
|---|---|---|---|---|
| 1 | Xây dựng Candidate profile và Company profile; bổ sung kiểm tra role, quyền sở hữu dữ liệu và các endpoint cập nhật thông tin hồ sơ. | 15/06/2026 | 15/06/2026 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html); [OWASP IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) |
| 2 | Phát triển API tạo, cập nhật và quản lý job posting; xây dựng luồng tìm kiếm việc làm, nộp application và HR review applicant. | 16/06/2026 | 17/06/2026 | [REST API Security Guidelines](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html); [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) |
| 3 | Tìm hiểu Amazon S3, cấu trúc bucket và object key; thực hành một số thao tác cơ bản bằng AWS CLI và cấu hình quyền truy cập tối thiểu bằng IAM. | 18/06/2026 | 18/06/2026 | [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html); [AWS CLI Command Reference for S3](https://docs.aws.amazon.com/cli/latest/reference/s3/); [IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) |
| 4 | Xây dựng document upload, file validation và permission checks; tích hợp S3 object key, presigned URL và kiểm thử tính nhất quán giữa database với object storage. | 19/06/2026 | 19/06/2026 | [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html); [Amazon S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html); [Uploading Objects with Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html) |

### Nội dung kỹ thuật đã triển khai:

Trong Tuần 2, tôi tập trung phát triển các luồng chức năng chính dành cho Ứng viên và HR/Doanh nghiệp, đồng thời tích hợp chức năng quản lý tài liệu với Amazon S3.

Đối với Ứng viên, hệ thống hỗ trợ tạo và cập nhật hồ sơ cá nhân, tìm kiếm việc làm, nộp hồ sơ ứng tuyển, tải tài liệu và theo dõi trạng thái ứng tuyển. Luồng dành cho Ứng viên được xây dựng để người dùng có thể quản lý thông tin cá nhân, xem các cơ hội việc làm, nộp hồ sơ, đính kèm tài liệu và theo dõi tiến trình tuyển dụng.

Đối với HR/Doanh nghiệp, hệ thống hỗ trợ quản lý hồ sơ công ty, tạo tin tuyển dụng, cập nhật thông tin công việc, xem danh sách ứng viên và quản lý trạng thái ứng tuyển. HR có thể duy trì thông tin doanh nghiệp, đăng cơ hội việc làm, xem hồ sơ ứng viên, kiểm tra tài liệu được tải lên và cập nhật tiến trình xử lý hồ sơ.

Một phần quan trọng của quá trình triển khai là phân tách quyền giữa Ứng viên và HR/Doanh nghiệp. Backend không chỉ kiểm tra người dùng đã đăng nhập hay chưa mà còn xác định vai trò của người dùng trước khi cho phép truy cập các chức năng nghiệp vụ.

Ví dụ, Ứng viên được phép nộp hồ sơ nhưng không được phép tạo hoặc chỉnh sửa tin tuyển dụng của doanh nghiệp. HR có thể quản lý công việc và ứng viên nhưng không thể thực hiện thao tác chỉ dành cho Ứng viên thay cho một tài khoản khác.

Kiểm tra quyền sở hữu cũng được áp dụng để ngăn người dùng truy cập hoặc chỉnh sửa tài nguyên không thuộc về mình. Điều này đặc biệt quan trọng đối với hồ sơ, tin tuyển dụng, đơn ứng tuyển và tài liệu được tải lên.

Việc kiểm tra vai trò và quyền sở hữu giúp giảm nguy cơ truy cập trực tiếp không an toàn vào tài nguyên. Ngay cả khi request chứa một resource ID hợp lệ, backend vẫn cần xác minh người dùng hiện tại có quyền thực hiện thao tác đó hay không.

### Luồng chức năng Ứng viên và HR:

Các chức năng dành cho Ứng viên và HR được xây dựng dựa trên toàn bộ vòng đời tuyển dụng.

Ứng viên bắt đầu bằng việc tạo hoặc cập nhật hồ sơ cá nhân. Thông tin này được sử dụng khi nộp đơn ứng tuyển và khi HR đánh giá mức độ phù hợp của ứng viên với công việc.

Sau khi xem các tin tuyển dụng, Ứng viên có thể gửi hồ sơ ứng tuyển. Backend kiểm tra người dùng hiện tại, xác nhận công việc tồn tại, kiểm tra dữ liệu ứng tuyển và liên kết đơn ứng tuyển với cả Ứng viên lẫn công việc được chọn.

Ứng viên cũng có thể tải CV hoặc các tài liệu hỗ trợ. Những tài liệu này được liên kết với hồ sơ hoặc đơn ứng tuyển thông qua metadata được lưu trong database.

Ở phía HR, hồ sơ doanh nghiệp chứa các thông tin liên quan đến công ty và được sử dụng trong các tin tuyển dụng. HR có thể tạo công việc mới, cập nhật mô tả công việc, xem danh sách ứng viên, truy cập thông tin được cấp quyền và cập nhật tiến trình của từng đơn ứng tuyển.

Luồng này tạo ra sự phân tách rõ ràng giữa hai nhóm người dùng, đồng thời vẫn cho phép họ tương tác thông qua dữ liệu công việc và ứng tuyển chung.

### Lưu trữ tài liệu với Amazon S3:

Chức năng quản lý tài liệu được thiết kế bằng cách tách metadata của file khỏi nội dung file thực tế.

Database lưu các thông tin có cấu trúc như chủ sở hữu, tên file, loại nội dung, thời gian tải lên, object key và hồ sơ hoặc đơn ứng tuyển liên quan. File nhị phân thực tế được lưu dưới dạng object trong Amazon S3.

Thiết kế này giúp tránh lưu trực tiếp các file lớn trong cơ sở dữ liệu quan hệ. PostgreSQL tiếp tục chịu trách nhiệm lưu dữ liệu nghiệp vụ có cấu trúc, trong khi Amazon S3 cung cấp khả năng lưu trữ object cho CV, tài liệu hỗ trợ và các file được tải lên.

Mỗi file được xác định bằng một S3 object key. Object key được backend tạo hoặc gán và được lưu cùng metadata của tài liệu trong database.

Cấu trúc object key giúp tổ chức file theo chủ sở hữu, loại tài liệu hoặc ngữ cảnh ứng tuyển. Cách này cũng làm giảm nguy cơ trùng tên file vì object key không cần phụ thuộc hoàn toàn vào tên file gốc.

Backend xác minh rằng metadata trong database và object trong S3 cùng tham chiếu đến một tài liệu. Mối quan hệ này quan trọng vì database kiểm soát quyền sở hữu và ngữ cảnh nghiệp vụ, còn S3 lưu nội dung file thực tế.

### Tích hợp Presigned URL:

S3 bucket được thiết kế ở trạng thái private thay vì công khai tài liệu bằng URL cố định.

Khi người dùng được cấp quyền cần truy cập tài liệu, backend tạo một presigned URL. URL này cung cấp quyền truy cập tạm thời vào một S3 object cụ thể mà không cần chuyển toàn bộ bucket hoặc object sang public.

Cách làm này cho phép ứng dụng duy trì kiểm soát truy cập ở backend. Trước khi tạo URL, hệ thống xác minh danh tính, vai trò và quyền sở hữu của người dùng hiện tại.

Ví dụ, Ứng viên có thể truy cập tài liệu cá nhân, trong khi HR chỉ có thể truy cập tài liệu của những ứng viên thuộc công việc do doanh nghiệp đó quản lý. Người dùng không có quyền sẽ không nhận được presigned URL hợp lệ.

Presigned URL cũng giảm nhu cầu để backend trực tiếp truyền nội dung file dung lượng lớn. Sau khi hoàn tất kiểm tra quyền, trình duyệt có thể giao tiếp trực tiếp với Amazon S3 thông qua URL tạm thời.

URL được giới hạn bởi thời gian hết hạn. Sau thời gian đó, liên kết không còn giá trị. Điều này tạo ra mức kiểm soát tốt hơn so với một public object URL tồn tại lâu dài.

### Kiểm tra file và bảo mật:

Quá trình upload bao gồm bước kiểm tra file trước khi tài liệu được chấp nhận.

Backend kiểm tra các thông tin như tên file gốc, content type, phần mở rộng và kích thước file. Các kiểm tra này giúp giảm nguy cơ lưu file không được hỗ trợ hoặc có dung lượng quá lớn.

Ứng dụng không chỉ phụ thuộc vào tên file do client cung cấp. Tên và phần mở rộng có thể gây hiểu nhầm, vì vậy validation cần dựa trên nhiều thuộc tính của request và các quy tắc phía server.

Hệ thống cũng tránh để lộ AWS credentials xuống frontend. Việc truy cập AWS được backend xử lý hoặc được thực hiện thông qua presigned URL tạm thời sau khi đã kiểm tra quyền.

IAM permission được xem xét theo nguyên tắc cấp quyền tối thiểu. Ứng dụng chỉ nên nhận những quyền S3 cần thiết cho thao tác tài liệu, chẳng hạn upload, đọc hoặc xóa object trong đúng bucket và đúng phạm vi đường dẫn.

Việc giữ bucket private, giới hạn IAM permission, kiểm tra file và xác minh quyền sở hữu tạo ra nhiều lớp bảo vệ cho hệ thống lưu trữ tài liệu.

### Thực hành AWS CLI:

Trong tuần này, tôi cũng thực hành các thao tác AWS CLI cơ bản liên quan đến Amazon S3.

AWS CLI cung cấp cách trực tiếp để xác minh AWS identity hiện tại, kiểm tra bucket, liệt kê object và xác nhận cấu trúc object key được ứng dụng sử dụng.

Việc sử dụng command line giúp tôi hiểu rõ hơn sự khác nhau giữa metadata tài liệu ở cấp ứng dụng và object được lưu ở cấp S3.

Nó cũng hỗ trợ kiểm tra file đã được tải lên đúng bucket hay chưa và object key có khớp với giá trị được lưu trong database hay không.

Quá trình thực hành này giúp tôi hiểu rõ hơn cách application code, AWS credentials, IAM permission và S3 resource tương tác trong một request upload.

### Vấn đề và cách giải quyết:

| Vấn đề | Nguyên nhân | Cách giải quyết | Trạng thái |
|---|---|---|---|
| Ứng viên và HR cần các quyền khác nhau. | Hai nhóm người dùng dùng chung backend nhưng thực hiện các nghiệp vụ khác nhau. | Bổ sung kiểm tra vai trò để giới hạn chức năng chỉ dành cho Ứng viên hoặc HR. | Hoàn thành |
| Authentication không ngăn hoàn toàn việc truy cập tài nguyên của người khác. | Người dùng hợp lệ vẫn có thể thử thay đổi resource ID trong request. | Bổ sung kiểm tra quyền sở hữu trước các thao tác với hồ sơ, đơn ứng tuyển, công việc và tài liệu. | Hoàn thành |
| Public document URL có thể làm lộ thông tin riêng tư của Ứng viên. | Public S3 object sẽ bỏ qua cơ chế phân quyền của ứng dụng. | Giữ S3 bucket private và cung cấp quyền truy cập bằng presigned URL. | Hoàn thành |
| Database record và S3 object có thể không nhất quán. | File storage và metadata được tạo ở hai hệ thống khác nhau. | Xác minh object key và document metadata trong luồng upload. | Hoàn thành |
| File được tải lên có thể không hợp lệ hoặc quá lớn. | Không thể tin tưởng hoàn toàn file do client cung cấp. | Bổ sung kiểm tra tên file, loại file, extension và kích thước trước khi lưu. | Hoàn thành |
| S3 permission quá rộng có thể tạo rủi ro bảo mật. | IAM policy rộng dễ cấu hình nhưng không phù hợp với nguyên tắc cấp quyền tối thiểu. | Giới hạn permission trong đúng bucket và các thao tác object cần thiết. | Hoàn thành |

### Kiến thức kỹ thuật đã học:

Tuần này giúp tôi hiểu rằng upload tài liệu không chỉ đơn giản là chuyển một file lên hệ thống lưu trữ.

Một chức năng upload hoàn chỉnh cần kết hợp authentication, kiểm tra vai trò, kiểm tra quyền sở hữu, validation file, object storage, quản lý metadata và kiểm soát truy cập.

Tôi hiểu rằng Amazon S3 được thiết kế cho object storage chứ không phải dữ liệu quan hệ. Database mô tả tài liệu và mối quan hệ của tài liệu với ứng dụng, trong khi S3 lưu file thực tế.

Tôi cũng học được rằng S3 object key là định danh nội bộ của object chứ không phải public link. Ứng dụng có thể sử dụng object key để tìm file trong khi bucket vẫn ở trạng thái private.

Presigned URL cho thấy cách cấp quyền truy cập tạm thời mà không làm lộ AWS credentials hoặc chuyển object sang public.

Một bài học quan trọng khác là authorization phải được đánh giá cho từng tài nguyên được yêu cầu. Người dùng đã đăng nhập không có nghĩa là được phép truy cập mọi hồ sơ, đơn ứng tuyển, công việc hoặc tài liệu.

### Kết quả tuần:

Đến cuối Tuần 2, ứng dụng đã hỗ trợ các luồng chính dành cho Ứng viên và HR/Doanh nghiệp trong hệ thống quản lý tuyển dụng.

Ứng viên có thể quản lý hồ sơ, xem công việc, nộp đơn ứng tuyển, tải tài liệu hỗ trợ và theo dõi tiến trình ứng tuyển.

HR/Doanh nghiệp có thể quản lý thông tin công ty, tạo và cập nhật tin tuyển dụng, xem danh sách ứng viên, truy cập thông tin được cấp quyền và quản lý trạng thái ứng tuyển.

Dự án cũng hoàn thành bước tích hợp dịch vụ lưu trữ AWS thực tế đầu tiên bằng cách kết nối chức năng tài liệu với Amazon S3.

File được lưu dưới dạng S3 object private, trong khi database duy trì metadata và thông tin quyền sở hữu tương ứng. Người dùng được cấp quyền có thể truy cập tài liệu thông qua presigned URL tạm thời.

Tuần này giúp củng cố cả chức năng nghiệp vụ lẫn nền tảng bảo mật của ứng dụng.

### Bài học rút ra:

Bài học chính trong Tuần 2 là authentication và authorization là hai trách nhiệm khác nhau.

Authentication xác nhận người dùng là ai, trong khi authorization xác định người dùng được phép thực hiện hành động nào. Một hệ thống an toàn cần kết hợp cả kiểm tra vai trò và quyền sở hữu tài nguyên.

Tôi cũng học được rằng cloud storage phải được tích hợp với các quy tắc nghiệp vụ của ứng dụng. Amazon S3 lưu file, nhưng backend và database xác định ai sở hữu file, file được sử dụng cho mục đích gì và ai được phép truy cập.

Sử dụng private bucket kết hợp presigned URL tạo ra thiết kế an toàn hơn so với việc cung cấp public link cố định.

Công việc trong tuần cũng cho thấy cần quan tâm đến tính nhất quán vì database và S3 là hai hệ thống riêng biệt. Ứng dụng cần duy trì mối quan hệ rõ ràng giữa document metadata và object key tương ứng.

### Kế hoạch tuần tiếp theo:

Tuần tiếp theo sẽ tập trung cải thiện giao tiếp giữa các service và chuẩn bị các luồng xử lý nền.

Các nội dung dự kiến bao gồm xử lý bất đồng bộ, message queue, retry handling, transactional outbox và giao tiếp đáng tin cậy giữa các service.

Các thành phần của ứng dụng cũng sẽ tiếp tục được chuẩn bị cho quá trình containerization và triển khai trong môi trường AWS có khả năng mở rộng.

<!--
TODO: Add screenshots, commits, API test results, S3 bucket configuration, IAM policy, or document upload evidence for this week.
Expected image directory:
static/images/worklog/week-2/
-->