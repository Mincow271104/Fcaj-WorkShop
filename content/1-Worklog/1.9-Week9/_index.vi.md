---
title: "Worklog Tuần 9"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---



### Mục tiêu tuần 9:

* Hoàn thành Chức năng AI Study Planner
* Gộp toàn bộ Setting Ai vào chung 1 mục để dễ quản lý

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Phân tích yêu cầu và thiết kế kiến trúc tổng thể cho chức năng AI Study Planner (gồm các tab Chat, Plan, Quiz và Settings) <br> - Thiết kế cấu trúc cơ sở dữ liệu để lưu trữ lộ trình học  | 29/06/2026   | 29/06/2026      | |
| 3   | - Phát triển giao diện tư vấn học tập ChatTab hỗ trợ trò chuyện thu thập thông tin <br> - Viết service Backend aiStudyService.js kết nối API Gemini và Ollama để xử lý hội thoại | 30/06/2026   | 30/06/2026      ||
| 4   | - Xây dựng giao diện hiển thị lộ trình PlanTab cho phép quản lý tiến độ học tập <br> - Phát triển module đọc tài liệu học tập (PDF, DOCX, TXT) hỗ trợ nạp ngữ cảnh vào bài học      | 01/07/2026   | 01/07/2026     |
| 5   | - Phát triển tính năng tự động tạo câu hỏi ôn tập ôn tập QuizTab.jsx tính điểm thưởng Knowledge Points (KP) <br> - Tái cấu trúc và gộp chung toàn bộ cài đặt AI vào một mục quản trị duy nhất | 02/07/2026   | 02/07/2026      |        |
| 6   | - Tối ưu hóa giao diện SCSS toàn bộ tính năng Study Planner <br> - Kiểm thử liên thông các luồng: Nhận tư vấn -> Tạo lộ trình -> Tải tài liệu -> Làm Quiz kiểm tra -> Tùy chỉnh cấu hình AI | 03/07/2026   | 03/07/2026      ||


### Kết quả đạt được tuần 9:

* Xây dựng hoàn chỉnh tính năng **AI Study Planner** tích hợp toàn diện trên ứng dụng Electron desktop bao gồm 3 phân hệ hoạt động liền mạch:
  * **Hỗ trợ trò chuyện (AI Chat):** Trợ lý ảo tự động phân tích và thu thập nhu cầu, thời gian biểu của người học.
  * **Lộ trình học tập (Study Plan):** Tự động thiết lập kế hoạch chi tiết chia theo nhiều giai đoạn, chủ đề và đề xuất tài liệu tham khảo tương ứng.
  * **Đánh giá kiến thức (AI Quiz):** Tự động sinh bộ câu hỏi trắc nghiệm củng cố kiến thức dựa theo từng giai đoạn và trả điểm thưởng Knowledge Points (KP).
* Phát triển thành công tính năng đọc file đính kèm (PDF, DOCX, TXT), tự động tóm tắt và trích xuất thông tin liên quan hỗ trợ cho cả 3 phân hệ học tập.
* Tái cấu trúc thành công giao diện cấu hình, gộp toàn bộ các thiết lập AI trước đây (như Groq API Key của AI Guard, Gemini Key của Study Planner, mô hình Ollama, danh sách chặn) về chung một trang cài đặt duy nhất giúp đơn giản hóa việc quản lý.
* Đảm bảo tính ổn định và tính tương thích cao khi hỗ trợ song song cả Gemini API (Cloud) và Ollama (Local) cho tất cả các tác vụ từ tài liệu.
* Hệ thống hoạt động mượt mà, lưu trữ dữ liệu thông qua electron-store và đồng bộ ổn định.
