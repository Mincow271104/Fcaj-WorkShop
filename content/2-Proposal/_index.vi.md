---
title: "Bản đề xuất"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Uchimi StudyGamification - Ứng dụng học tập kết hợp AI và Gamification


### 1. Tổng quan dự án
Dự án được thiết kế dưới dạng ứng dụng học tập trên nền tảng máy tính (Desktop App) kết hợp công nghệ AI và hệ sinh thái Gamification. Hệ thống hướng đến mục tiêu biến hành động "mở máy tính lên học mỗi ngày" thành một thói quen tự nhiên, tự nguyện và bền vững. Thông qua việc ứng dụng vòng lặp hành vi: **Kỷ luật → Hành động → Phản hồi → Phần thưởng**, hệ thống giúp người học vừa duy trì sự tập trung cao độ, vừa được giải trí lành mạnh sau những giờ học căng thẳng.

### 2. Vấn đề cần giải quyết
Quá trình tự học hiện nay gặp phải những rào cản lớn về mặt tâm lý và động lực:
* **Thiếu phản hồi tức thời:** Thành quả học tập xuất hiện chậm, trong khi mạng xã hội hoặc game lại cung cấp lượng dopamine phản hồi tức thì, khiến người học dễ xao nhãng và bỏ cuộc.
* **Công cụ truyền thống khô khan:** Các phần mềm học tập hiện tại chủ yếu tập trung vào quản lý công việc, chưa giải quyết được bài toán duy trì động lực dài hạn.
* **Gamification thiếu thực chất:** Nếu chỉ tập trung vào phần thưởng giải trí thuần túy, người dùng sẽ dễ bị cuốn vào game mà quên đi kết quả học tập thực chất.
* **Gian lận trong hệ thống số:** Các ứng dụng máy tính dễ bị can thiệp (chỉnh sửa thời gian học, hack điểm minigame, nhân bản vật phẩm), làm mất đi tính công bằng của môi trường.

### 3. Mục tiêu dự án
Dự án hướng đến việc xây dựng một hệ sinh thái học tập cân bằng giữa **kỷ luật học tập** và **giải trí lành mạnh**:
* **Chuyển đổi mục tiêu lớn:** Chia nhỏ lộ trình học tập dài hạn thành các phiên tập trung (Focus mode) và nhiệm vụ hằng ngày có thể thực hiện được.
* **Trực quan hóa sự tiến bộ:** Cung cấp phản hồi kịp thời qua hệ thống điểm số (Knowledge Points), chuỗi ngày học (streak), vật phẩm thú cưng và bảng xếp hạng.
* **Ứng dụng AI để cá nhân hóa:** Hỗ trợ sinh lộ trình học, tạo bài kiểm tra (quiz) từ tài liệu và giám sát xao nhãng qua công nghệ AI Scan.
* **Đảm bảo tính công bằng và minh bạch:** Xây dựng hệ thống backend chống gian lận, bảo vệ tính toàn vẹn của dữ liệu tiền tệ, vật phẩm và xếp hạng.

### 4. Kiến trúc giải pháp
Dự án sử dụng mô hình **Client-Server phân tán**, kết hợp giữa tính tiện dụng của ứng dụng Desktop và khả năng mở rộng linh hoạt của Cloud.

*   **Client (Frontend):** Xây dựng bằng ReactJS và đóng gói bằng Electron thành ứng dụng Desktop độc lập. Quản lý trạng thái và lưu trữ bộ nhớ đệm cục bộ (Local Caching) bằng Redux và Electron-store nhằm tăng tốc độ phản hồi.
*   **AI & Tracking:** Sử dụng Browser Extension (AIGuard) giao tiếp qua WebSocket với Desktop App. Tích hợp AWS Bedrock (STS cross-account), Google Gemini (Cloud AI) hoặc Ollama (Local AI) cho các tính năng phân loại nội dung và hỗ trợ học tập.
*   **Backend (AWS Serverless):** Toàn bộ Business Logic (Gacha, Shop, Quest, Social) được xử lý thông qua AWS Lambda và API Gateway tại region **ap-southeast-1**. 
*   **Lưu trữ & Dữ liệu:** Amazon DynamoDB lưu trữ dữ liệu hệ thống (sử dụng **TransactWriteItems** để đảm bảo tính toàn vẹn). Amazon S3 lưu trữ tài nguyên tĩnh (assets, avatar) và CloudFront làm CDN.
*   **Dịch vụ phụ trợ:** Amazon Cognito (Xác thực/JWT), EventBridge (Cronjobs cho bảng xếp hạng/shop), DynamoDB Streams và Algolia (Tìm kiếm người dùng).

### 5. Lộ trình triển khai (Timeline)
*   **Giai đoạn 1 (Thiết lập hạ tầng):** Khởi tạo AWS resources (Cognito, DynamoDB, S3), cấu hình API Gateway và triển khai bộ khung Lambda cơ bản bằng Serverless Framework.
*   **Giai đoạn 2 (Phát triển Client):** Xây dựng giao diện ReactJS/Electron, tích hợp Focus Mode, hệ thống Daily Quest và cơ chế Local Caching / Preloading tài nguyên.
*   **Giai đoạn 3 (Tích hợp AI):** Phát triển AIGuard Extension, thiết lập WebSocket, kết nối Gemini API/Ollama để sinh lộ trình học và quiz.
*   **Giai đoạn 4 (Hệ sinh thái Gamification):** Hoàn thiện logic Minigame, Gacha, Shop, Leaderboard trên Cloud. Áp dụng các cơ chế Anti-cheat (chống gian lận).
*   **Giai đoạn 5 (Đóng gói & Vận hành):** Build file **.exe** bằng **electron-builder**, phân phối qua Google Drive. Kiểm thử end-to-end và giám sát qua CloudWatch.

### 6. Ngân sách và Tối ưu chi phí
Kiến trúc được thiết kế xoay quanh tư duy **tối ưu chi phí đám mây (Cost-optimization)**:
*   **Scale-to-Zero:** Việc sử dụng AWS Lambda và API Gateway giúp hệ thống gần như không tốn chi phí máy chủ cố định khi không có người dùng truy cập.
*   **Tùy chọn AI linh hoạt:** Người dùng có thể sử dụng AWS Bedrock, tự cấu hình API Key (Gemini) hoặc chạy Local LLM (Ollama) trực tiếp trên máy để bảo mật dữ liệu cá nhân và tiết kiệm chi phí Cloud API token, giúp nền tảng tối ưu chi phí vận hành.
*   **Tối ưu thao tác cơ sở dữ liệu:** Áp dụng bộ nhớ đệm nội bộ trên Client và cơ chế đồng bộ tổng hợp (**POST /sync-all**) nhằm giảm số lượng gọi API và giảm lượt đọc/ghi trên DynamoDB.

### 7. Đánh giá rủi ro và Giảm thiểu

| Loại rủi ro | Mô tả vấn đề | Biện pháp giảm thiểu |
| :--- | :--- | :--- |
| **Độ trễ hệ thống** | Hiện tượng Cold Start của AWS Lambda làm chậm request đầu tiên. | Giữ cấu hình RAM cho Lambda ở mức tối ưu (512MB) để cân bằng tốc độ và chi phí. |
| **Toàn vẹn dữ liệu** | Client cache dễ gây xung đột dữ liệu hoặc gian lận từ phía người dùng. | Backend luôn đóng vai trò Server Authority; dùng **TransactWriteItems** cho các giao dịch quan trọng. |
| **Tiêu thụ tài nguyên** | Electron và AI (Local) có thể tiêu tốn nhiều RAM của máy tính. | Tối ưu hóa Vite Build (tree-shaking) và cho phép người dùng linh hoạt chọn giữa Cloud AI / Local AI. |
| **Vendor Lock-in** | Phụ thuộc sâu vào hệ sinh thái AWS (Cognito, DynamoDB, Streams). | Đóng gói logic nghiệp vụ thành các module Lambda độc lập, chuẩn hóa giao thức kết nối. |