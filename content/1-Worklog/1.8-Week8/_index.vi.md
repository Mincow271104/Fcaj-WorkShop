---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---



### Mục tiêu tuần 8:

* Thêm chức năng Nhiệm vụ ngày cho dự án
* Lên kế hoạch làm chức năng AI Study PLanner

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| Mon,Tue,Wed,Thu | **Thêm chức năng Nhiệm vụ ngày (Daily Quests) cho dự án:** <br>- Thiết kế & Xây dựng UI/UX: Giao diện QuestPanel (danh sách nhiệm vụ, thanh tiến trình, hiệu ứng claim reward). <br>- Quản lý Trạng thái: Tích hợp Redux cho danh sách quest, tiến độ và trạng thái phần thưởng. <br>- Thiết lập Logic: Thuật toán tự động cập nhật tiến độ (Progress Tracker) từ sự kiện Focus, Login. <br>- Cơ chế Reset: Logic tự động làm mới nhiệm vụ sau 24h và đồng hồ đếm ngược. <br>- Lưu trữ & Đồng bộ: Giao tiếp IPC Electron để lưu trữ cục bộ và đồng bộ hóa đám mây. | 22/06/2026 | 25/06/2026 | |
| Fri,Sat | **Lên kế hoạch phát triển chức năng AI Study Planner:** <br>- Phân tích yêu cầu: Xác định luồng Chat tư vấn, Tạo lộ trình (Generate Plan) và Tạo trắc nghiệm (Generate Quiz). <br>- Đánh giá Công nghệ: Khảo sát mô hình LLM (Groq/OpenAI), chuẩn bị Prompt Engineering mẫu. <br>- Thiết kế Kiến trúc: Sơ đồ lưu trữ lịch sử chat, cấu trúc JSON Study Plan và Quiz DB. <br>- Phác thảo Module: Định nghĩa các component cần viết (Chat UI, Plan Viewer, Quiz Modal) và thiết kế IPC Main/Renderer. <br>- Lập kế hoạch: Chia nhỏ các task cụ thể, ước lượng thời gian và lập cột mốc phát triển. | 26/06/2026 | 27/06/2026 | |

### Kết quả đạt được tuần 8:

* **Chức năng Nhiệm vụ ngày (Daily Quests):**
  * Thiết kế hoàn chỉnh và xây dựng thành công giao diện QuestPanel sinh động với thanh tiến độ và hiệu ứng claim phần thưởng mượt mà.
  * Tích hợp thành công Redux để quản lý chặt chẽ trạng thái nhiệm vụ, kèm theo hệ thống Progress Tracker tự động đếm dựa trên hành động Focus/Login của người dùng.
  * Hoàn thiện cơ chế tự động Reset nhiệm vụ sau 24h và đồng bộ hóa dữ liệu lưu trữ qua IPC xuống Local/Cloud an toàn.

* **Kế hoạch AI Study Planner:**
  * Xác định rõ ràng các luồng nghiệp vụ chính của AI Study Planner: Tư vấn học tập, Tự động thiết lập lộ trình học, và Tạo bài trắc nghiệm ôn tập.
  * Đánh giá và lựa chọn giải pháp LLM phù hợp (Groq/OpenAI API) đáp ứng yêu cầu tốc độ phản hồi nhanh, đồng thời hoàn thiện các prompt mẫu.
  * Thiết kế xong cấu trúc dữ liệu lưu trữ (lịch sử chat, schema JSON lộ trình, ngân hàng câu hỏi quiz) và các đặc tả giao tiếp IPC giữa Main và Renderer process.
  * Phân rã chi tiết đầu việc (Task Breakdown), ước lượng thời gian thực hiện để sẵn sàng bắt tay vào viết code cho tuần tiếp theo.
