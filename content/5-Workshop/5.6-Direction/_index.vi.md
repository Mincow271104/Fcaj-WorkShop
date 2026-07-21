---
title : "Hướng phát triển"
date : 2026-07-17 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

Với kiến trúc Serverless linh hoạt trên nền tảng AWS và ứng dụng Client sử dụng ReactJS + Electron, dự án đã xây dựng được một nền tảng cốt lõi vững chắc. Tuy nhiên, để hệ thống thực sự trở thành một hệ sinh thái học tập toàn diện, nhóm phát triển đã nhìn nhận rõ các điểm cần cải thiện cũng như vạch ra lộ trình mở rộng chiến lược trong tương lai.

### 5.6.1. Các điểm cần cải thiện của dự án

Dựa trên quá trình kiểm thử và vận hành thực tế, dự án hiện đang tồn đọng một số hạn chế cần được ưu tiên khắc phục trong các phiên bản tiếp theo:

*   **Tối ưu hóa hiệu suất (Performance) của AI Scan:** Do đặc thù của ứng dụng Electron kết hợp việc chạy liên tục module AI để quét hành vi/màn hình người dùng nhằm phát hiện xao nhãng, ứng dụng hiện tại có thể tiêu tốn khá nhiều tài nguyên phần cứng (CPU/RAM). Cần nghiên cứu tối ưu hóa các mô hình AI nhẹ hơn (Lightweight models) hoặc áp dụng cơ chế xử lý luồng (multithreading) hiệu quả hơn ở Client để máy tính không bị giật lag khi người dùng vừa học vừa bật ứng dụng nặng.
*   **Tối ưu hóa cơ chế Caching và Preloading tài nguyên:** Tăng cường lưu trữ bộ nhớ đệm cục bộ (Local Storage/Electron-store) để tiền tải (preload) hình ảnh, vật phẩm UI và cấu hình tĩnh. Giúp ứng dụng khởi động tức thì, phản hồi mượt mà và giảm tối đa số lượng request dư thừa lên AWS Serverless mà không phụ thuộc vào xử lý lưu trữ offline.
*   **Tăng cường bảo mật và chống gian lận (Anti-cheat):** Bổ sung các rào chắn kỹ thuật chặt chẽ hơn để ngăn chặn tình trạng người dùng sử dụng tool (auto-click, macro) để cày tiền tệ trong Minigame hoặc tìm cách qua mặt hệ thống AI Scan khi đang trong chế độ Focus Session.

---

### 5.6.2. Các chức năng phát triển thêm trong tương lai

Bên cạnh việc khắc phục hạn chế, dự án định hướng mở rộng thành một hệ sinh thái EdTech (Giáo dục & Công nghệ) thông qua các nâng cấp sau:

#### 1. Nâng cấp Trí tuệ Nhân tạo (AI)
*   **Xử lý Đa phương tiện (Multimodal AI):** Nâng cấp khả năng của Trợ lý AI để không chỉ đọc hiểu văn bản thuần túy mà còn nhận diện hình ảnh, giải nghĩa công thức toán học phức tạp, hoặc trích xuất nội dung từ video bài giảng để tóm tắt cho người dùng.
*   **Cá nhân hóa lộ trình học:** Ứng dụng Machine Learning để phân tích biểu đồ thời gian tập trung của người dùng, từ đó AI có thể đưa ra lời khuyên về khung giờ học hiệu quả nhất (Golden hours) cho từng cá nhân.
*   **Voice Chat AI:** Tích hợp công nghệ nhận diện và tổng hợp giọng nói (Speech-to-Text / Text-to-Speech), biến AI thành một người bạn đồng hành hỗ trợ luyện tập giao tiếp ngoại ngữ thực tế ngay trong ứng dụng.

#### 2. Đào sâu Hệ sinh thái Gamification
*   **Mở rộng Kho Minigame:** Bổ sung thêm các trò chơi tư duy mang tính giáo dục cao như giải đố từ vựng (Crossword), Sudoku, hoặc Flashcard tương tác. Các màn chơi sẽ được thiết kế logic để tái sử dụng module tạo màn chơi tự động hiện có.
*   **Hệ thống Thú cưng ảo (Virtual Pet):** Cải tiến vòng quay Gacha để mở khóa các "Trợ lý Thú cưng". Thú cưng sẽ xuất hiện trên màn hình Focus Session dưới dạng overlay, có trạng thái vui/buồn dựa trên tiến độ duy trì thói quen (Streak) của người dùng, tạo thêm sự gắn kết sâu sắc về mặt cảm xúc.

#### 3. Tương tác Xã hội & Quản lý Giáo dục (Social & Ed-Management)
*   **Phòng học ảo đa người dùng (Co-Study Room):** Cho phép người dùng tạo các phòng học chung theo thời gian thực. Bạn bè có thể cùng bật camera/mic, chia sẻ bộ đếm giờ Pomodoro để tăng tính giám sát ngang hàng (Peer pressure), giúp nhau cùng tiến bộ. Tính năng tìm kiếm phòng học sẽ được tối ưu bởi Algolia.
*   **Hệ thống Quản lý Học viên (Classroom Management):** Mở rộng tệp khách hàng sang mô hình B2B/B2C bằng cách cho phép Giáo viên hoặc Gia sư tạo "Phòng học/Lớp học" riêng:
    *   Giáo viên có quyền quản lý danh sách học viên, theo dõi chi tiết biểu đồ thời gian tập trung (strikeCount, rankScore), và mức độ xao nhãng của từng cá nhân.
    *   Giáo viên có thể thiết lập các Lộ trình học (Study Plan) chung cho cả lớp, giao Nhiệm vụ nhóm (Group Quests) và sử dụng hệ thống Tiền tệ/Bảng xếp hạng nội bộ của lớp để vinh danh những cá nhân xuất sắc nhất.