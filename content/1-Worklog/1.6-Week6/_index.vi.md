---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---



### Mục tiêu tuần 6:

*Hoàn thành việc chuyển đổi be sang dịch vụ aws

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| Mon,Tue,Wed,Thu,Fri,Sat | - Chuyển đổi Backend sang dịch vụ AWS: <br>&emsp; + Amazon API Gateway: Cổng tiếp nhận và điều hướng (routing) các API requests từ Client. <br>&emsp; + AWS Lambda: Đảm nhận chạy logic nghiệp vụ (Serverless Backend) như đồng bộ dữ liệu (sync), quản lý nhiệm vụ (quest), và quản lý phiên học (session). <br>&emsp; + Amazon DynamoDB: Cơ sở dữ liệu NoSQL lưu trữ thông tin Profile (user, streak), Nhiệm vụ (Quest), và Lịch sử học tập (Study Sessions). <br>&emsp; + Amazon Cognito: Quản lý tài khoản, xác thực người dùng và bảo mật API bằng JWT Token. <br> Lên văn phòng, trao đổi với các thành viên và lên kế hoạch cho chức năng mới và nâng cấp chức năng cũ                                                                                              | 08/06/2026  | 14/06/2026      |
 


### Kết quả đạt được tuần 6:
* Chuyển đổi thành công Backend sang dịch vụ AWS:
  * Tích hợp Amazon API Gateway để tiếp nhận và điều hướng toàn bộ các API requests từ Client.
  * Triển khai thành công AWS Lambda xử lý các logic nghiệp vụ backend (đồng bộ dữ liệu, quản lý nhiệm vụ, quản lý phiên học).
  * Thiết lập cơ sở dữ liệu Amazon DynamoDB để lưu trữ thông tin Profile (user, streak), Nhiệm vụ (Quest) và Lịch sử học tập (Study Sessions).
  * Áp dụng Amazon Cognito để quản lý tài khoản, xác thực người dùng và bảo mật hệ thống API bằng JWT Token.
* Lên văn phòng làm việc, thảo luận trực tiếp với các thành viên để thống nhất kế hoạch phát triển các chức năng mới và cải tiến các chức năng hiện có.



