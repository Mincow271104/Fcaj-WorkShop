---
title: "Worklog Tuần 10"
date: 2026-7-6
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---


### Mục tiêu tuần 10:

* Hoàn thiện và tối ưu hóa tính năng Focus Mode (kiểm soát tập trung) kết hợp với Browser Extension và hệ thống AI.
* Tích hợp thành công các mô hình AI trên nền tảng AWS Bedrock (Nova Micro, Nova Lite) để phục vụ YouTube Blocker và StudyPlanner.
* Nâng cao trải nghiệm người dùng (UX/UI) và xử lý triệt để các lỗi về giới hạn gọi API (Rate Limiting).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tích hợp hệ thống Amazon Bedrock AI (Nova Micro, Nova Lite) cho ứng dụng <br> - Xây dựng cơ chế tự động Retry (xử lý lỗi quá tải API) <br> - Cải thiện giao diện Focus Mode (tách chế độ Casual/Rank, thêm giới hạn thời gian Custom) <br> - Hiển thị chi tiết trạng thái Extension và AI <br> - Dọn dẹp Settings UI | 06/07/2026   | 06/07/2026      | |
| 3   | - Cập nhật và tối ưu các module tiếp theo... | 07/07/2026   | 07/07/2026      | |
| 4   | - Cập nhật và tối ưu các module tiếp theo... | 08/07/2026   | 08/07/2026      | |
| 5   | - Cập nhật và tối ưu các module tiếp theo... | 09/07/2026   | 09/07/2026      | |
| 6   | - Tổng kết và kiểm thử ứng dụng...           | 10/07/2026   | 10/07/2026      | |


### Kết quả đạt được tuần 10:

* Tích hợp thành công Amazon Bedrock AI với các model Nova Micro, Nova Lite thay thế cho Gemini.
* Xây dựng được luồng gọi API ổn định với cơ chế tự động Retry thông minh khi server quá tải (503) hoặc vượt quota (429).
* Giao diện Focus Mode được làm lại thân thiện và khoa học hơn (phân chia Casual và Rank mode rõ ràng, ngăn chặn chọn thời gian quá ngắn).
* Cung cấp hiển thị trạng thái hệ thống minh bạch hơn cho người dùng (Extension đã nhận những trình duyệt nào, AI đang dùng model của hãng nào).
* Tối ưu UI Cài đặt, loại bỏ các thành phần cấu hình không cần thiết (FaceTracking).


