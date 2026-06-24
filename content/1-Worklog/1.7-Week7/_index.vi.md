---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---



### Mục tiêu tuần 7:

* Thay đổi đầu ra của dự án theo DB mới của team
* Tối ưu cách dự án sử dụng AI
* Nâng cấp AI Block cho website


### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2,3   | - Tái cấu trúc dữ liệu đầu vào (Input) và đầu ra (Output) phía Client để tương thích hoàn toàn với cấu trúc schema mới của cơ sở dữ liệu Amazon DynamoDB. <br>- Kiểm thử tính đúng đắn của dữ liệu khi đồng bộ thông qua API Gateway. | 15/06/2026   | 16/06/2026      | |
| 4,5,6,7   | - Tối ưu hóa hiệu suất gọi mô hình AI phân loại, xây dựng cơ chế cache kết quả để giảm thiểu số lượng request lên Cloud API (Groq) và Local AI (Ollama). <br>- Nghiên cứu và tích hợp thêm tính năng lọc thông minh sử dụng AI để tự động phân tích và chặn các đường dẫn (Web URLs) không xác định nằm ngoài danh sách tĩnh. | 17/06/2026   | 20/06/2026      | |
 
 
 
### Kết quả đạt được tuần 7:
* Hoàn thành việc điều chỉnh cấu trúc dữ liệu Input/Output phía Client, giúp hệ thống đồng bộ dữ liệu người dùng, nhiệm vụ và lịch sử học tập trơn tru với cơ sở dữ liệu Amazon DynamoDB mới.
* Tối ưu hóa thành công cơ chế hoạt động của AI phân loại nội dung video, giảm thiểu số lượng API requests không cần thiết và cải thiện đáng kể tốc độ phản hồi của hệ thống.
* Phát triển thành công tính năng lọc nâng cao (AI Block) cho website, giúp tự động nhận dạng và đưa ra quyết định chặn/cho phép đối với các trang web lạ (Web URLs không xác định) dựa trên phân tích ngữ cảnh nội dung.
