---
title : "Hệ thống Minigame Học tập"
date : 2026-07-20
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

## 1. TỔNG QUAN (OVERVIEW)

Hệ thống Minigame là một phân hệ (module) chiến lược được tích hợp trực tiếp vào nền tảng học tập, đóng vai trò là "gạch nối" hoàn hảo giữa việc rèn luyện tư duy và giải trí thư giãn. Bằng cách ứng dụng cơ chế Gamification với các tựa game kinh điển (Sudoku và Minesweeper), hệ thống không chỉ giúp người học giảm bớt căng thẳng mà còn tạo ra động lực giữ chân họ ở lại lâu hơn với ứng dụng.

Về mặt kỹ thuật, hệ thống được xây dựng hoàn toàn theo kiến trúc Serverless Cloud-native trên nền tảng AWS.

*   **Phía Client:** Ứng dụng Desktop sử dụng Electron, React và Redux để quản lý trạng thái, tạo trải nghiệm mượt mà và lưu trữ bộ nhớ đệm cục bộ (Local Caching) giúp tăng tốc độ phản hồi.
*   **Phía Server:** Giao tiếp qua Amazon API Gateway, xử lý logic nghiệp vụ bằng các hàm AWS Lambda, tự động hóa tác vụ ngầm bằng Amazon EventBridge, và lưu trữ toàn bộ trạng thái, điểm số trên cơ sở dữ liệu NoSQL Amazon DynamoDB.

## 2. LUỒNG VẬN HÀNH HỆ THỐNG MINIGAME TRONG ỨNG DỤNG HỌC TẬP

### 2.1. Kiến trúc tổng thể
Luồng hoạt động của hệ thống Minigame được thiết kế theo mô hình khép kín Client-Server, kết hợp kho lưu trữ cục bộ (Redux & Electron Store) trên thiết bị người dùng và cơ sở dữ liệu đám mây (AWS DynamoDB) thông qua các dịch vụ Serverless.

### 2.2. Chi tiết các luồng trong vòng đời một phiên chơi

**Luồng 1: Khởi tạo và Đồng bộ dữ liệu tại Hub (MinigameHub)**
*   Người dùng truy cập vào phân vùng Minigame Hub và lựa chọn trò chơi mong muốn (Sudoku hoặc Minesweeper).
*   Hệ thống tiến hành đồng bộ danh sách các màn chơi thông qua các service tương ứng (**handleSyncSudokuLevels** hoặc **handleSyncMinesweeperLevels**). Tại đây, ứng dụng kiểm tra bộ nhớ đệm cục bộ (Redux Store và Electron Store) để tối ưu hiển thị, hoặc gửi request qua API Gateway đến AWS Lambda để nạp danh sách màn chơi mới nhất từ server.

**Luồng 2: Bắt đầu ván đấu & Quản lý tài nguyên (sanity)**
*   Khi người dùng chọn một màn chơi cụ thể, client thực hiện kiểm tra sơ bộ điều kiện mở khóa và lượng thể lực hiện có (**sanity**) so với chi phí yêu cầu (**sanityCost**).
*   Sau khi vượt qua kiểm tra, yêu cầu POST (**handleStartSudokuSession** / **handleStartMinesweeperSession**) được gửi lên server.
*   Phía backend (Lambda) tiến hành trừ trực tiếp chi phí **sanity** trong thông tin Profile của người dùng, khởi tạo cấu trúc bản đồ tương ứng (tạo ma trận giải pháp **solutionGrid**, đục lỗ tạo đề bài **puzzleGrid**), đồng thời ghi bản ghi phiên chơi với trạng thái **PENDING** vào DynamoDB.

**Luồng 3: Tương tác thời gian thực trong ván đấu & Cơ chế bảo mật**
*   Trong quá trình người dùng giải đố, các thao tác điền số (Sudoku) hoặc mở ô/cắm cờ (Minesweeper) được ghi nhận thành các mảng log hành động (**actionLogs**) lưu trên Redux.
*   **Cơ chế bảo mật & Chống gian lận:**
    *   **Đối với Sudoku:** Mỗi lần người dùng yêu cầu kiểm tra bước đi (**handleCheckSudokuStep**), server sẽ chạy thuật toán phân tích hành vi (**checkSudokuCheat**) để ngăn chặn việc tua ngược thời gian, click tự động bằng bot quá nhanh, hoặc sửa đổi đề bài.
    *   **Đối với Minesweeper:** Mỗi cú click mở ô (**handleRevealApi**) sẽ gửi kèm token mã hóa trạng thái; server giải mã trực tiếp trên RAM, kiểm tra nếu dẫm mìn sẽ kết thúc ván đấu, hoặc tiến hành loang vùng trống (**runFloodFill**) và cấp phát token mới cho nước đi tiếp theo.

**Luồng 4: Kết thúc ván đấu**
Ván đấu kết thúc dựa trên kết quả thực tế của người chơi:
*   **Trường hợp Thắng cuộc (WIN):** Người chơi hoàn thành chính xác bàn cờ và gửi yêu cầu nộp bài (**handleSubmitSudoku** / **handleSubmitMinesweeper**). Server xác thực lần cuối, tính toán điểm số dựa trên thời gian và độ khó, cộng thưởng tiền tệ **eCoin**, cập nhật kỷ lục cá nhân (Personal Best), và chuyển trạng thái phiên chơi trên DynamoDB thành **COMPLETED**.
*   **Trường hợp Thua cuộc hoặc Thoát sớm (LOST / QUIT):** Nếu người chơi giải sai, dẫm phải mìn, hoặc chủ động thoát giữa chừng (**endState = 'quit'**), hệ thống kích hoạt chính sách hoàn trả 50% chi phí **sanity** ban đầu vào ví người dùng, cập nhật trạng thái session thành **CANCELLED** hoặc **UNCOMPLETED**. Cơ chế này giúp giảm áp lực tâm lý cho người học.

**Luồng 5: Cập nhật Xếp hạng tự động ngầm (Background Worker)**
*   Cứ mỗi 10 phút, bộ định tuyến AWS EventBridge tự động kích hoạt một hàm Lambda Worker (**handleLeaderboardWorker**).
*   Worker này sẽ quét (Scan) bảng dữ liệu thống kê (**stats**) trên DynamoDB của tất cả người chơi, sắp xếp theo tổng điểm để tạo ra bảng xếp hạng Top 10 (**leaderboard**) kèm theo thời gian hết hạn (**expiresAt**).

## 3. PHÂN TÍCH GIÁ TRỊ CHỨC NĂNG ĐỐI VỚI MỘT ỨNG DỤNG HỌC TẬP

Việc tích hợp cơ chế Gamification với hai tựa game tư duy (Sudoku & Minesweeper) vào nền tảng học tập mang lại các giá trị chiến lược cốt lõi sau:

### 3.1. Giải quyết bài toán "Mệt mỏi nhận thức" (Cognitive Fatigue)
*   **Vấn đề:** Người học thường dễ bị nản chí, giảm tập trung khi phải tiếp thu lượng lớn kiến thức hàn lâm liên tục trong thời gian dài trên ứng dụng.
*   **Giải pháp:** Minigame đóng vai trò là một "vùng đệm giải lao lành mạnh" (Micro-break). Thay vì thoát app để lướt mạng xã hội (dễ gây mất tập trung hoàn toàn), người dùng chuyển sang giải một ván Sudoku hoặc Minesweeper nhanh trong 3–5 phút để kích thích tư duy logic mà vẫn giữ chân họ trong hệ sinh thái của app.

### 3.2. Xây dựng Vòng lặp Gắn kết (Core Retention Loop) thông qua Kinh tế Ứng dụng
*   **Cơ chế Sanity:** Ngăn chặn tình trạng cày cuốc vô độ, đồng thời tạo động lực cho người dùng quay trở lại với các bài học chính trên app để "cày" lại thể lực (**sanity**) và tài nguyên (**eCoin**).
*   **Cơ chế Hoàn trả 50% Sanity:** Đây là một điểm chạm UX được tinh chỉnh để tối ưu hóa tâm lý người chơi. Việc hoàn trả tài nguyên giúp giảm thiểu tối đa sự ức chế khi thất bại, ngăn chặn tình trạng người dùng nản lòng và rời bỏ ứng dụng. Nhờ đó, người chơi được khuyến khích liên tục thử thách bản thân ở những màn khó hơn mà không lo sợ rủi ro hao hụt tài nguyên quá nặng.

### 3.3. Rèn luyện Kỹ năng Mềm và Tư duy Logic cho Người học
*   **Sudoku:** Rèn luyện sự kiên nhẫn, khả năng tập trung cao độ, ghi nhớ và suy luận loại trừ – những kỹ năng bổ trợ trực tiếp cho việc giải quyết các bài toán phức tạp.
*   **Minesweeper:** Rèn luyện tư duy xác suất thống kê, quản trị rủi ro và ra quyết định dưới áp lực thông tin bất đối xứng.
