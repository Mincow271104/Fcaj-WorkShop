---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---



### Mục tiêu tuần 5:

* Làm những chức năng còn lại
* lên kế hoạch chuyển đổi công nghệ sang dịch vụ AWS

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| Mon,Tue,Wed,Thu,Fri | - Tích hợp Trí tuệ nhân tạo và các tính năng nâng cao: <br> **1. Các tính năng thực hiện:** <br> + Kiểm duyệt video tinh vi bằng AI: Phân tích sâu tiêu đề, mô tả và nội dung nhằm chặn các video giải trí tinh vi lách luật. <br> + Theo dõi hiện diện người dùng: Nhận diện khuôn mặt qua WebCam, phát cảnh báo và tự động hủy bộ đếm nếu người dùng rời khỏi màn hình quá lâu. <br> + Tự động tắt ứng dụng giải trí: Quét hệ thống và tắt ngang các phần mềm mạng xã hội hoặc giải trí độc lập cài trên máy tính. <br> + Chế độ kỷ luật thép: Khóa hoàn toàn nút tạm dừng hoặc dừng, buộc tập trung cho đến khi hết giờ. <br> **2. Công nghệ áp dụng:** <br> + AI & LLMs (Phân loại video): Local AI với Ollama và Cloud AI với Groq API. <br> + Computer Vision (Face Tracking): Google MediaPipe qua WebAssembly.| 01/06/2026   | 05/06/2026      |
| 7, CN | - Lên kế hoạch chuyển đổi (Migration) sang dịch vụ AWS: <br> + Phân tích kiến trúc hiện tại để map với các dịch vụ AWS phù hợp (EC2, S3, API Gateway, DynamoDB...). <br> + Lên danh sách các tài nguyên cần tạo. | 06/06/2026 | 07/06/2026 |


### Kết quả đạt được tuần 5:
* Tích hợp thành công mô hình ngôn ngữ lớn (Local AI với Ollama và Cloud AI với Groq API) vào ứng dụng để phân loại và kiểm duyệt tự động nội dung video YouTube có độ chính xác cao.
* Ứng dụng thành công công nghệ Computer Vision (Google MediaPipe chạy qua WebAssembly) để theo dõi sự hiện diện của người dùng (AFK Tracking) theo thời gian thực mà vẫn đảm bảo hiệu suất máy tính.
* Triển khai hoàn chỉnh cơ chế Native App Killer, cho phép ứng dụng quét và tự động đóng các tiến trình giải trí (như Facebook, TikTok) chạy độc lập trên hệ điều hành Windows.
* Hoàn thiện tính năng kỷ luật thép (Hard Mode), buộc người dùng phải tuân thủ nghiêm ngặt thời gian tập trung đã đặt ra.
* Đã hoàn thành bản phân tích kiến trúc và lên kế hoạch sơ bộ để tiến hành chuyển đổi (Migration) các thành phần lưu trữ, backend của ứng dụng lên hạ tầng đám mây AWS cho giai đoạn tiếp theo.


