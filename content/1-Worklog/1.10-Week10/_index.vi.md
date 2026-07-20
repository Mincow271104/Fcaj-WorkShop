---
title: "Worklog Tuần 10"
date: 2026-07-06
weight: 10
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
| Mon | - Cấu hình IAM Role và Policy trên 2 tài khoản AWS để thiết lập quyền Cross-Account truy cập Amazon Bedrock <br> - Tích hợp hệ thống Amazon Bedrock AI với các mô hình Nova Micro và Nova Lite | 06/07/2026   | 06/07/2026      | |
| Tue | - Xây dựng cơ chế tự động Retry thông minh để xử lý lỗi quá tải API (429, 503) đảm bảo tính ổn định | 07/07/2026   | 07/07/2026      | |
| Wed | - Cải thiện giao diện Focus Mode, phân chia rõ ràng chế độ Casual và Rank, đồng thời bổ sung tính năng giới hạn thời gian Custom | 08/07/2026   | 08/07/2026      | |
| Thu | - Phát triển tính năng hiển thị chi tiết và minh bạch trạng thái của Browser Extension và model AI đang sử dụng | 09/07/2026   | 09/07/2026      | |
| Fri | - Dọn dẹp Settings UI, loại bỏ các thành phần cấu hình không cần thiết (như FaceTracking) | 10/07/2026   | 10/07/2026      | |
| Sat | - Kiểm thử liên thông toàn bộ hệ thống, tối ưu hóa hiệu năng và vá các lỗi phát sinh | 11/07/2026 | 11/07/2026 | |

### Kết quả đạt được tuần 10:

* Thiết lập thành công kết nối Cross-Account giữa 2 tài khoản AWS, cho phép gọi API Bedrock an toàn.
* Tích hợp thành công Amazon Bedrock AI với các model Nova Micro, Nova Lite thay thế cho Gemini.
* Xây dựng được luồng gọi API ổn định với cơ chế tự động Retry thông minh khi server quá tải (503) hoặc vượt quota (429).
* Giao diện Focus Mode được làm lại thân thiện và khoa học hơn (phân chia Casual và Rank mode rõ ràng, ngăn chặn chọn thời gian quá ngắn).
* Cung cấp hiển thị trạng thái hệ thống minh bạch hơn cho người dùng (Extension đã nhận những trình duyệt nào, AI đang dùng model của hãng nào).
* Tối ưu UI Cài đặt, loại bỏ các thành phần cấu hình không cần thiết (FaceTracking).
