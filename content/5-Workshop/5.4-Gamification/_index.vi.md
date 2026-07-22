---
title : "Hệ thống Minigame Học tập"
date : 2026-07-20
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

## 1. Tổng quan

Hệ thống Minigame là phân hệ chiến lược được tích hợp trực tiếp vào nền tảng học tập, đóng vai trò là gạch nối giữa việc rèn luyện tư duy và giải trí thư giãn. Bằng cách ứng dụng cơ chế Gamification với các tựa game kinh điển như Sudoku và Minesweeper, hệ thống giúp người học giảm bớt căng thẳng, đồng thời tạo động lực duy trì thói quen học tập hàng ngày.

Về mặt kỹ thuật, hệ thống vận hành theo kiến trúc Serverless Cloud-native trên nền tảng AWS:

- **Phía Client:** Ứng dụng Desktop kết hợp Electron, React và Redux quản lý trạng thái, tạo trải nghiệm mượt mà và nạp dữ liệu từ bộ nhớ đệm cục bộ (Local Caching) để tối ưu tốc độ phản hồi.
- **Phía Server:** Giao tiếp qua Amazon API Gateway, xử lý logic nghiệp vụ bằng các hàm AWS Lambda, tự động hóa tác vụ ngầm bằng Amazon EventBridge, và lưu trữ trạng thái, điểm số trên cơ sở dữ liệu NoSQL Amazon DynamoDB.

---

## 2. Chuẩn bị tài nguyên và triển khai

### Bước 1: Chuẩn bị cơ sở dữ liệu AWS DynamoDB

Thực hiện tạo bảng trên AWS DynamoDB qua Console hoặc AWS CLI:
- Tạo bảng DynamoDB với tên bảng định nghĩa trong biến môi trường MINIGAME_TABLE.
- Thiết lập khóa chính bao gồm Partition Key (PK) và Sort Key (SK) theo mô hình NoSQL đơn bảng (Single-Table Design) để tối ưu hóa truy vấn cho Sudoku, Minesweeper và Bảng xếp hạng.

![Bảng Minigame trên DynamoDB](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651615/minigame_table_hrjnzo.png)

### Bước 2: Chuẩn bị dữ liệu màn chơi cho từng minigame

- Tạo các danh sách màn chơi dưới dạng file JSON và tải lên Amazon S3 theo đúng cấu trúc thư mục quy định để nạp dữ liệu vào MINIGAME_TABLE.

![Dữ liệu màn chơi được tải lên S3](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651630/S3_item_ygzgfq.png)

### Bước 3: Cấu hình Backend Lambda Service

- Xây dựng các hàm API xử lý logic trò chơi, cơ chế chống gian lận và tính toán điểm thưởng cho từng minigame (minesweeperService.mjs, sudokuService.mjs, minigameService.mjs).

![Các file xử lý logic game và ví dụ 1 hàm Lambda](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651630/S3_item_ygzgfq.png)

- Khai báo đầy đủ các biến môi trường trên Lambda gồm MINIGAME_TABLE, USER_TABLE và GAME_SECRET_KEY.

![Biến môi trường](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651652/env_zrewsm.png)

### Bước 4: Thiết lập Worker tự động cập nhật bảng xếp hạng với EventBridge

- Tạo một EventBridge Rule thiết lập lịch trình chạy tự động 10 phút một lần: rate(10 minutes).
- Cấu hình Target của Rule trỏ trực tiếp đến hàm Lambda xử lý bảng xếp hạng (handleLeaderboardWorker).

![Hàm Lambda tải bảng xếp hạng](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651673/leaderboard_erwqsb.png)

### Bước 5: Khởi chạy Client Frontend (Electron & Redux)

- Khởi chạy ứng dụng Electron kết hợp Redux Store (minigameReducer.js, minigameLogsReducer.js).
- Cấu hình các API service (minigameService.js, minesweeperService.js, sudokuService.js) để kết nối với API Gateway.

![Giao diện Minigame Hub chạy trên client](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651685/minigameClient_z2ytes.png)

---

## 3. Luồng vận hành hệ thống Minigame trong ứng dụng học tập

### 3.1. Kiến trúc tổng thể

Luồng hoạt động của hệ thống Minigame được thiết kế theo mô hình khép kín Client-Server, kết hợp bộ nhớ đệm cục bộ (Redux và Electron Store) trên ứng dụng Desktop và cơ sở dữ liệu đám mây (AWS DynamoDB) thông qua các dịch vụ Serverless.

### 3.2. Chi tiết các luồng trong vòng đời một phiên chơi

#### Luồng 1: Khởi tạo và đồng bộ dữ liệu tại Minigame Hub

- Người dùng truy cập phân vùng Minigame Hub và chọn trò chơi mong muốn (Sudoku hoặc Minesweeper).
- Hệ thống tiến hành đồng bộ danh sách màn chơi qua các service tương ứng (handleSyncSudokuLevels hoặc handleSyncMinesweeperLevels). Ứng dụng kiểm tra bộ nhớ đệm cục bộ để hiển thị ngay lập tức, đồng thời gửi request qua API Gateway đến AWS Lambda để nạp danh sách màn chơi mới nhất từ DynamoDB nếu cần.

![Giao diện tải danh sách màn chơi chạy trên client](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651698/level_list_nj0un8.png)

#### Luồng 2: Bắt đầu ván đấu và quản lý tài nguyên thể lực

- Khi chọn một màn chơi, Client thực hiện kiểm tra sơ bộ điều kiện mở khóa và lượng thể lực hiện có (Sanity) so với chi phí yêu cầu (sanityCost).
- Yêu cầu POST (handleStartSudokuSession / handleStartMinesweeperSession) được gửi lên server để bắt đầu phiên chơi.
- Hàm Lambda trừ trực tiếp điểm thể lực trong hồ sơ người dùng, khởi tạo ma trận màn chơi (solutionGrid và puzzleGrid), đồng thời ghi bản ghi phiên chơi mới với trạng thái PENDING vào DynamoDB.

![Giao diện sau khi bắt đầu session chơi Sudoku](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651725/open_level_vutzqf.png)

![Dữ liệu session được lưu trên DynamoDB](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651738/session_DB_qf0gep.png)

#### Luồng 3: Tương tác thời gian thực và cơ chế bảo mật

Trong quá trình giải đố, các nước đi (điền số ở Sudoku hoặc mở ô/cắm cờ ở Minesweeper) được ghi nhận thành chuỗi nhật ký hành động (actionLogs) lưu tạm ở Redux.

Hệ thống áp dụng các cơ chế xác thực an toàn:
- **Đối với Sudoku:** Khi người dùng kiểm tra nước đi (handleCheckSudokuStep), server chạy thuật toán kiểm tra (checkSudokuCheat) để ngăn chặn hành vi tua thời gian, click tự động quá nhanh hoặc sửa đổi cấu trúc đề bài.
- **Đối với Minesweeper:** Mỗi thao tác mở ô (handleRevealApi) gửi kèm token mã hóa trạng thái. Server giải mã trên RAM để xác định nếu trúng mìn sẽ kết thúc ván đấu, hoặc thực hiện loang vùng trống (runFloodFill) và trả về token mới cho nước đi tiếp theo.

#### Luồng 4: Kết thúc ván đấu và xử lý phần thưởng

Ván đấu kết thúc tùy theo kết quả thực tế của người chơi:
- **Thắng cuộc (WIN):** Khi người chơi hoàn thành đúng bàn cờ và gửi yêu cầu nộp bài (handleSubmitSudoku / handleSubmitMinesweeper), server xác minh lần cuối, tính điểm theo thời gian và độ khó, cộng tiền thưởng eCoin, cập nhật kỷ lục cá nhân và chuyển trạng thái phiên chơi thành COMPLETED trên DynamoDB.
- **Thua cuộc hoặc thoát sớm (LOST / QUIT):** Nếu giải sai, trúng mìn hoặc chủ động thoát giữa chừng (endState = 'quit'), hệ thống tự động hoàn trả 50% chi phí thể lực ban đầu vào tài khoản người dùng, đồng thời chuyển trạng thái session thành CANCELLED hoặc UNCOMPLETED. Chính sách này giúp giảm gánh nặng tâm lý thất bại cho người học.

![Giao diện sau khi hoàn thành màn chơi](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651748/sudoku_win_ji9ram.png)

![Trạng thái session được cập nhật trên DynamoDB thành COMPLETED](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651753/session_complete_iopm8d.png)

#### Luồng 5: Cập nhật bảng xếp hạng tự động ngầm

- Định kỳ 10 phút một lần, EventBridge kích hoạt hàm Lambda Worker (handleLeaderboardWorker).
- Worker tiến hành quét bảng thống kê (stats) trên DynamoDB, tổng hợp điểm số toàn bộ người chơi để cập nhật danh sách Top 10 bảng xếp hạng (leaderboard) kèm thời gian hết hạn (expiresAt).

---

## 4. Phân tích giá trị chức năng đối với một ứng dụng học tập

Việc tích hợp cơ chế Gamification cùng hai tựa game tư duy Sudoku và Minesweeper vào nền tảng học tập mang lại các giá trị chiến lược quan trọng:

### 4.1. Giải quyết bài toán mệt mỏi nhận thức (Cognitive Fatigue)

- **Bài toán:** Người học dễ bị nản chí và giảm khả năng tập trung khi phải tiếp thu liên tục khối lượng kiến thức lớn trong thời gian dài.
- **Giải pháp:** Minigame tạo ra một vùng đệm giải lao lành mạnh (micro-break). Thay vì rời ứng dụng để truy cập mạng xã hội, người dùng có thể giải trí ngắn 3–5 phút với Sudoku hoặc Minesweeper để vừa thư giãn vừa duy trì trạng thái kích thích tư duy logic trong cùng một ứng dụng.

### 4.2. Xây dựng vòng lặp gắn kết người dùng qua nền kinh tế ứng dụng

- **Cơ chế thể lực (Sanity):** Giới hạn số lượt chơi để tránh việc giải trí quá độ, đồng thời tạo động lực cho người dùng quay trở lại hoàn thành các bài học chính để tích lũy thêm điểm thể lực và tài nguyên eCoin.
- **Cơ chế hoàn trả 50% thể lực:** Là điểm chạm tinh tế trong thiết kế trải nghiệm người dùng. Việc hoàn trả một phần tài nguyên khi thua cuộc giúp xoa dịu cảm giác thất bại, khuyến khích người học sẵn sàng thử sức lại ở những mức độ khó hơn.

### 4.3. Rèn luyện kỹ năng mềm và tư duy logic

- **Sudoku:** Giúp rèn luyện tính kiên nhẫn, sự tập trung cao độ, khả năng ghi nhớ và suy luận loại trừ.
- **Minesweeper:** Hỗ trợ phát triển tư duy xác suất thống kê, quản trị rủi ro và ra quyết định trong điều kiện thông tin chưa đầy đủ.