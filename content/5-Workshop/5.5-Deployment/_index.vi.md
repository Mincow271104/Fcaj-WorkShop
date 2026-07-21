---
title : "Triển khai và Vận hành"
date : 2026-07-18 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Quá trình triển khai dự án được thiết kế theo hướng tối giản hóa quy trình cài đặt cho người dùng cuối (end-user). Thay vì yêu cầu người dùng phải thiết lập môi trường phức tạp (Node.js, npm, Python,...), toàn bộ hệ thống Client (bao gồm Giao diện React và Lõi xử lý ngầm) được đóng gói thành một file thực thi duy nhất (Standalone Executable) hoạt động độc lập trên hệ điều hành Windows.

---

## 1. Quy trình Triển khai và Đóng gói (Packaging)

Dự án sử dụng kiến trúc kết hợp: **Vite + React** cho Frontend và **Electron** cho Backend cục bộ (giao tiếp hệ điều hành, quản lý tiến trình, WebSocket Server). Việc đóng gói được tự động hóa hoàn toàn thông qua **electron-builder**.

**Các bước đóng gói kỹ thuật:**

- **Bước 1 — Biên dịch Frontend (Vite Build):** 
  Toàn bộ mã nguồn React, SCSS, và assets tĩnh được Vite phân tích, tối ưu (tree-shaking) và biên dịch (minify) thành các file HTML/CSS/JS tĩnh, lưu vào thư mục **dist**.
  
- **Bước 2 — Đóng gói Electron (Electron Builder):**
  Hệ thống gộp bộ mã Backend của Electron (bao gồm các service gọi AI, quản lý file, WebSocket server), frontend tĩnh từ **dist**, và một môi trường Chromium thu nhỏ.

- **Bước 3 — Xuất bản Executable (Lệnh Build):**
  Nhà phát triển chỉ cần chạy lệnh duy nhất trên Terminal:
  ```bash
  npm run build:desktop
  ```
  Lệnh này sẽ kích hoạt quy trình trên và kết xuất ra một file cài đặt thực thi duy nhất (định dạng **.exe**). File này chứa đầy đủ mọi thành phần để chạy ứng dụng mà không phụ thuộc vào bất kỳ thư viện bên ngoài nào trên máy tính người dùng.

---

## 2. Phân phối và Tải về (Distribution)

Do dự án ưu tiên tính tiện dụng và triển khai nhanh chóng, quá trình phân phối ứng dụng Desktop được thực hiện thông qua dịch vụ lưu trữ đám mây:

1. **Lưu trữ:** File **.exe** sau khi build thành công sẽ được upload lên **Google Drive**.
2. **Chia sẻ link:** Đặt quyền truy cập phù hợp và gắn link vào trang tài liệu hoặc gửi cho giảng viên/người hướng dẫn.

*Lưu ý:* Đối với thành phần **Browser Extension (AIGuard)**, nhà phát triển nén thư mục Extension thành file **.zip** và gửi kèm link tải hoặc tải trực tiếp lên Chrome Web Store (nếu được phát hành public).

---

## 3. Cài đặt và Sử dụng Hệ thống (Installation & Usage)

Quy trình trải nghiệm của người dùng cuối (User Flow) được thiết kế cực kỳ liền mạch:

### a. Cài đặt Ứng dụng Desktop
1. Người dùng tải file **Setup.exe** từ link Google Drive được cung cấp.
2. Nhấn đúp để chạy file. Quá trình cài đặt diễn ra tự động hoàn toàn (Silent Install) vào thư mục cục bộ của Windows.
3. Một biểu tượng shortcut ứng dụng sẽ tự động xuất hiện trên màn hình Desktop.

### b. Cài đặt Browser Extension (AIGuard)
1. Người dùng mở trình duyệt (Chrome/Edge/Brave).
2. Vào trang Quản lý Tiện ích (Extensions), bật chế độ **Developer mode**.
3. Chọn **Load unpacked** và trỏ tới thư mục Extension đã tải về và giải nén (hoặc cài đặt bằng 1 click từ Chrome Web Store).

### c. Vận hành Hệ thống
1. Người dùng mở ứng dụng Desktop từ icon, đăng nhập bằng tài khoản đã được cấp (hoặc đăng ký mới).
2. **Kết nối:** Extension trên trình duyệt sẽ tự động bắt tay (handshake) với Desktop App thông qua WebSocket ngầm (**ws://localhost:8765**).
3. **Cấu hình:** Người dùng vào tab AI Settings (Cấu hình) để kích hoạt API Key (Gemini) hoặc bật chế độ AWS Bedrock, cũng như kiểm tra trạng thái model Local (Ollama) nếu muốn.
4. **Sử dụng:** Từ lúc này, người dùng có thể tự do trải nghiệm mọi tính năng:
   - Upload tài liệu học tập để chat với trợ lý AI và tự động sinh lộ trình ôn luyện (Study Planner).
   - Chọn thời gian và chế độ học, sau đó bấm bắt đầu để vào phiên học tập trung (Focus Mode).
   - Hệ thống AIGuard và Face Tracking tự động kích hoạt ngầm để bảo vệ không gian học tập khỏi các website/video giải trí.
   - Hoàn thành phiên học, nhận thưởng Knowledge Points (KP), điểm Rank, hoàn thành Daily Quest và tương tác mua sắm đồ vật cho Pet.

---

## 4. Vận hành và Cập nhật (Maintenance & Updates)

Khi có bản vá lỗi (bug fix) hoặc cập nhật tính năng mới:
- Nhà phát triển chỉ cần kéo mã nguồn, chạy lại lệnh **npm run build:desktop** và đẩy file **.exe** phiên bản mới lên Google Drive.
- Người dùng tải lại bản mới và chạy cài đặt. Phiên bản mới sẽ cài đặt đè lên phiên bản cũ một cách mượt mà.
- **An toàn dữ liệu:** Mọi dữ liệu cốt lõi (Rank, KP, vật phẩm Pet, lịch sử học) đều được lưu trữ và tính toán an toàn trên backend **AWS Serverless (DynamoDB)**. Các cài đặt cá nhân cục bộ (API keys, tùy chọn hiển thị) lưu trên máy cũng không bị mất đi trong quá trình cập nhật app.