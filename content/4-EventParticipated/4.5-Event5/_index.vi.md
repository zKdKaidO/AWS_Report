---
title: "FCAJ x Agentic AI Build Week 2026"
date: 2026-07-26
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## Giới thiệu sự kiện

FCAJ x Agentic AI Build Week 2026 là buổi chia sẻ về quá trình xây dựng các sản phẩm Agentic AI trong môi trường hackathon. Các đội tham gia trình bày bài toán thực tế, kiến trúc hệ thống, dịch vụ AWS được sử dụng, khó khăn trong quá trình phát triển và những bài học rút ra sau khi hoàn thiện sản phẩm thử nghiệm.

Khác với chatbot chỉ tạo câu trả lời, các giải pháp Agentic AI tại sự kiện được thiết kế để hiểu mục tiêu, lập kế hoạch, sử dụng công cụ, truy xuất dữ liệu và thực hiện hành động. Bốn dự án được trình bày gồm S.H.E.P.H.E.R.D., KFC Bot Agent, Solution Architect Professional AI Native App và SignalScout.

## Thông tin sự kiện

| Thông tin | Nội dung |
|---|---|
| Tên sự kiện | FCAJ x Agentic AI Build Week 2026 |
| Ngày tham gia | 26/07/2026 |
| Hình thức | Trực tiếp |
| Chủ đề chính | Agentic AI, AWS Cloud, hackathon và phát triển sản phẩm |
| Nội dung | Chia sẻ hành trình xây dựng, kiến trúc, demo và bài học từ các đội tham gia |

## 1. S.H.E.P.H.E.R.D. – Giám sát và dự đoán mật độ đám đông

Đội 3KA chia sẻ hành trình 24 giờ phát triển S.H.E.P.H.E.R.D., một hệ thống hỗ trợ theo dõi mật độ người, tình trạng hàng chờ và nguy cơ ùn tắc thông qua camera.

Hệ thống sử dụng computer vision để phát hiện và theo dõi người trong video, sau đó chuyển dữ liệu thành các chỉ số vận hành dễ hiểu cho nhân viên quản lý địa điểm.

Các thành phần chính gồm:

- YOLO và ByteTrack để phát hiện, theo dõi người.
- Amazon SageMaker để hỗ trợ quá trình xử lý mô hình.
- Amazon Bedrock AgentCore và Strands Agent cho lớp Agentic AI.
- React dashboard để hiển thị mật độ, cảnh báo và trạng thái khu vực.
- Autonomous Monitor để theo dõi liên tục và phát hiện nguy cơ ùn tắc.
- Operator Copilot để nhân viên đặt câu hỏi bằng ngôn ngữ tự nhiên.

Những khó khăn chính của đội gồm xử lý video trực tiếp, giảm độ trễ inference, duy trì tracking giữa các frame, lựa chọn vị trí camera và giới hạn phạm vi sản phẩm trong thời gian hackathon.

Bài học quan trọng từ đội 3KA là một sản phẩm nhỏ nhưng hoàn chỉnh có giá trị hơn một ý tưởng lớn nhưng không thể demo. Việc xác định mục tiêu, phân chia vai trò và chuẩn bị kịch bản trình bày từ sớm giúp đội làm việc hiệu quả hơn.

## 2. KFC Bot Agent – Đặt món trực tiếp trong cuộc trò chuyện

OneTeam giới thiệu KFC Bot Agent, một AI Agent hỗ trợ khách hàng đặt món trực tiếp trên các nền tảng trò chuyện như Zalo, Messenger và các kênh có thể được tích hợp trong tương lai.

Giải pháp được xây dựng để giảm việc chuyển đổi giữa ứng dụng nhắn tin và ứng dụng đặt món. Người dùng có thể mô tả món ăn, số lượng, kích thước nước uống hoặc chương trình khuyến mãi bằng ngôn ngữ tự nhiên.

AI Agent xử lý yêu cầu theo quy trình:

1. Hiểu ý định đặt món.
2. Lập kế hoạch các bước cần thực hiện.
3. Truy vấn dữ liệu sản phẩm và quy tắc kinh doanh.
4. Cập nhật giỏ hàng hoặc áp dụng khuyến mãi.
5. Kiểm tra lại giỏ hàng trước khi xác nhận.

Điểm đáng chú ý của giải pháp là mô hình ngôn ngữ không tự quyết định toàn bộ kết quả. Các công cụ và dữ liệu nghiệp vụ mới là nguồn xác nhận thông tin thực tế về sản phẩm, giá, voucher và trạng thái giỏ hàng.

Kiến trúc được thiết kế theo hướng có thể mở rộng: thêm kênh giao tiếp bằng adapter, thêm doanh nghiệp bằng connector và thêm chức năng bằng tool mới. Theo số liệu thử nghiệm trong bài trình bày, hệ thống có độ trễ khoảng 3–5 giây và chi phí ước tính khoảng 0,006 USD cho mỗi đơn hàng trong kịch bản được đội xây dựng.

## 3. Solution Architect Professional AI Native App

Plan V trình bày một AI Native App hỗ trợ Solution Architect trong quá trình phân tích yêu cầu và xây dựng kiến trúc AWS.

Bài toán xuất phát từ việc Solution Architect thường phải đọc BRD hoặc PRD, trích xuất yêu cầu, xây dựng kiến trúc, vẽ sơ đồ và ước tính chi phí trong thời gian ngắn. Phần lớn công việc này phụ thuộc vào kinh nghiệm cá nhân và phải được thực hiện lại cho mỗi dự án.

Ứng dụng được thiết kế để:

- Phân tích yêu cầu từ văn bản tự nhiên và tài liệu dự án.
- Tạo Requirements Catalogue ban đầu.
- Đề xuất các lựa chọn kiến trúc ở mức tổng quan.
- Hỗ trợ kiến trúc hybrid cloud và tiêu chuẩn riêng của doanh nghiệp.
- Tạo sơ đồ Draw.io và AWS Architecture Diagram có thể chỉnh sửa.
- Sinh ước tính chi phí định hướng cho Region `ap-southeast-1`.
- Chỉ ra giả định, đề xuất và những yêu cầu còn thiếu.
- Cho phép người dùng tiếp tục chỉnh sửa qua giao diện chat.
- Hỗ trợ tạo Infrastructure as Code tự động.

Giải pháp không nhằm thay thế hoàn toàn Solution Architect. Mục tiêu là tạo ra bản nháp có căn cứ để người dùng đánh giá và điều chỉnh, thay vì phải bắt đầu từ một trang trắng.

## 4. SignalScout – Phát hiện sớm thay đổi chiến lược doanh nghiệp

SignalScout là nền tảng sử dụng AI để phát hiện sớm những tín hiệu cho thấy doanh nghiệp đang thay đổi chiến lược, tái cấu trúc hoặc điều chỉnh hoạt động.

Hệ thống thu thập và kết nối dữ liệu từ nhiều nguồn, phân tích các chỉ số tài chính và vận hành, sau đó trình bày kết quả thông qua dashboard, timeline và cảnh báo rủi ro.

Các giá trị chính của SignalScout gồm:

- Phát hiện sớm những thay đổi chiến lược.
- Kết nối các tín hiệu rời rạc thành một câu chuyện rõ ràng.
- Cung cấp bằng chứng cho từng kết luận.
- Hỗ trợ các quyết định Maintain, Adapt hoặc Accelerate.
- Giữ con người trong vai trò kiểm soát quyết định cuối cùng.
- Hỗ trợ nhóm chiến lược, quản trị rủi ro và competitive intelligence.

Kiến trúc sử dụng các dịch vụ như Amazon Bedrock, AgentCore, AWS Lambda, Amazon DynamoDB, Amazon S3, Amazon API Gateway, Amazon Cognito, Amazon CloudWatch và AWS WAF. Đội cũng phân tích nhiều mức lưu lượng và đưa ra phương án kiến trúc tối ưu chi phí thay vì chỉ tập trung xây dựng chức năng.

## Những bài học chính

Qua bốn phần trình bày, mình nhận thấy một sản phẩm Agentic AI cần nhiều thành phần hơn một mô hình ngôn ngữ. Để hoạt động đáng tin cậy, hệ thống cần kết hợp model với dữ liệu, công cụ, business rules, khả năng giám sát và sự kiểm soát của con người.

Những bài học chính gồm:

- Bắt đầu từ pain point cụ thể thay vì bắt đầu từ một mô hình AI.
- Giới hạn phạm vi MVP để có thể hoàn thiện và trình bày sản phẩm.
- AI Agent cần sử dụng nguồn dữ liệu đáng tin cậy trước khi thực hiện hành động.
- Kiến trúc nên cho phép thêm kênh, công cụ hoặc chức năng mà không phải xây dựng lại toàn bộ hệ thống.
- Chi phí, độ trễ, bảo mật và khả năng vận hành cần được xem xét ngay từ giai đoạn thiết kế.
- Việc phân chia vai trò và chuẩn bị demo có ảnh hưởng lớn đến kết quả hackathon.
- AI nên hỗ trợ quá trình ra quyết định, không nhất thiết thay thế hoàn toàn con người.

## Liên hệ với project thực tập

Nội dung sự kiện giúp mình hiểu rõ hơn cách thiết kế AI matching trong project thực tập. Thay vì để mô hình tự đưa ra toàn bộ quyết định, hệ thống cần sử dụng parser, dữ liệu có cấu trúc, tiêu chí tuyển dụng và reranker để tạo kết quả có thể giải thích.

Bài học về giới hạn phạm vi cũng phù hợp với quyết định tập trung tính năng AI matching vào phía HR. Giải pháp này giúp HR sắp xếp và xem xét ứng viên, trong khi quyết định tuyển dụng cuối cùng vẫn thuộc về người sử dụng.

## Kết luận

FCAJ x Agentic AI Build Week 2026 mang lại góc nhìn thực tế về quá trình chuyển một ý tưởng AI thành sản phẩm có thể demo. Các dự án tuy giải quyết những bài toán khác nhau nhưng đều cho thấy Agentic AI chỉ thực sự có giá trị khi được kết hợp với kiến trúc phù hợp, dữ liệu đáng tin cậy, công cụ thực hiện hành động và nhu cầu rõ ràng của người dùng.

Sự kiện cũng cho thấy hackathon không chỉ là cuộc thi về công nghệ. Đây còn là môi trường để học cách giới hạn phạm vi, phối hợp nhóm, xử lý áp lực và biến một ý tưởng thành sản phẩm hoạt động được trong thời gian ngắn.