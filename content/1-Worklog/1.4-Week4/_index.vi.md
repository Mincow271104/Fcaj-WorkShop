---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---



### Mục tiêu tuần 4:

* Làm chức năng đầu tiên của đề tài
* Lên văn phòng, tham dự Event.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| Mon,Tue | - Lên kế hoạch để làm chức năng cho đề tài: <br> + Sử dụng công nghệ bình thường để làm trước, sau đó sẽ chuyển qua thành các dịch vụ AWS sau.                                                                                  | 25/05/2026   | 26/05/2026      |
| 4,5,6,7,CN   | - Bắt đầu xây dựng chức năng cốt lõi và cơ chế chặn cơ bản: <br> **1. Các tính năng thực hiện:** <br> + Quản lý phiên tập trung: Tạo bộ đếm thời gian và chế độ chặn thủ công. <br> + Chặn website mạng xã hội: Hiển thị màn hình khóa che toàn bộ nội dung khi truy cập các trang mạng xã hội trong giờ tập trung. <br> + Lọc video YouTube cơ bản: Trích xuất danh mục nội bộ của video để phân loại nhanh, cho phép tùy chỉnh danh sách miễn trừ và đồng bộ cài đặt theo thời gian thực. <br> + Bảo vệ chống lách luật: Quét các trình duyệt đang mở để ngăn chặn việc người dùng cố tình tắt hoặc không cài tiện ích mở rộng. <br> **2. Công nghệ áp dụng:** <br> + Desktop App: Electron, Node.js, WebSocket, Electron Store (cho Backend) và Vanilla JS, HTML, CSS (cho Frontend). <br> + Browser Extension: Manifest V3, Vanilla JS/CSS.| 27/05/2026   | 31/05/2026      |


### Kết quả đạt được tuần 4:

* Khởi tạo thành công nền tảng cho ứng dụng Desktop bằng framework Electron, Node.js và cấu trúc cơ bản cho Browser Extension (Manifest V3).
* Xây dựng hoàn chỉnh tính năng quản lý phiên tập trung trên giao diện Desktop (bao gồm đồng hồ đếm ngược Pomodoro và chế độ bật/tắt thủ công).
* Thiết lập thành công kênh giao tiếp thời gian thực giữa Desktop App và Browser Extension thông qua WebSocket.
* Xây dựng tính năng màn hình khóa (Overlay) trên Extension, chặn đứng việc truy cập vào các mạng xã hội giải trí (Facebook, TikTok, Instagram...) trong giờ tập trung.
* Triển khai cơ chế trích xuất dữ liệu (DOM Extraction) trên YouTube để lấy danh mục (Category) nội bộ của video, qua đó thiết lập bộ lọc tự động: chặn danh mục giải trí và thả hành lang cho danh mục học tập.
* Hoàn thành cơ chế chống lách luật (Extension Gate): Hệ thống Desktop có khả năng quét các tiến trình trình duyệt đang chạy và tự động vô hiệu hóa nếu phát hiện người dùng cố tình tắt tiện ích mở rộng.


