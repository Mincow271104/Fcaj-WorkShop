---
title: "Workshop"
date: 2026-07-17
weight: 5
chapter: true
pre: " <b> 5. </b> "
---

# Uchimi StudyGamification - Ứng dụng học tập kết hợp AI và Gamification


Chào mừng anh/ chị đã đến với tài liệu kỹ thuật của dự án **Uchimi StudyGamification**. Dự án được xây dựng với kiến trúc **ReactJS + Electron** ở phía Client và nền tảng **AWS Serverless** ở phía Server, hướng đến mục tiêu tối ưu hóa chi phí vận hành nhưng vẫn mang lại trải nghiệm học tập tuyệt vời, giúp quá trình tự học không còn gò bó hay mang tính ép buộc.

* **Video Demo dự án:** [Xem Video Demo](https://youtu.be/8Bw4-nwUaTc)
* **Sơ đồ kiến trúc dự án:** ![Sơ đồ kiến trúc AWS](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)
* **Mã nguồn Frontend:** [GitHub Repository - AWSStudy-Play](https://github.com/KhangChinh/AWSStudy-Play)
* **Mã nguồn Backend:** [GitHub Repository - AWSServerless](https://github.com/KhangChinh/AWSServerless)
* **Tải ứng dụng Desktop (.exe):** [Tải bản Setup (.exe)](https://drive.google.com/file/d/1Qkdv_nLrQWICSZFCWqytDioq4r35Vyz4/view?usp=sharing)

## Tổng quan

Trong thời đại công nghệ số phát triển mạnh mẽ, thách thức lớn nhất của quá trình tự học không còn nằm ở việc khó tiếp cận kiến thức. Khó khăn thực sự nằm ở **khả năng duy trì sự tập trung lâu dài và tính kỷ luật** để việc học không bị gián đoạn. Người học rất dễ bị xao nhãng bởi vô vàn cám dỗ khi sử dụng máy tính, khiến việc hình thành một thói quen học tập bền vững trở nên vô cùng khó khăn.

**Dự án Uchimi StudyGamification** ra đời nhằm giải quyết bài toán này. Mục tiêu cốt lõi của dự án là biến hành động "mở máy tính lên học mỗi ngày" thành một thói quen tự nhiên, bền vững và đầy hứng thú. 

Hệ thống mang đến một giải pháp kết hợp hiệu quả giữa **kỷ luật học tập** và **giải trí lành mạnh**:
*   **Học tập kỷ luật cùng AI:** Áp dụng công nghệ AI Scan để theo dõi (track) và ngăn chặn ngay lập tức các hành vi xao nhãng khi người dùng thao tác trên máy tính, giúp duy trì sự tập trung cao độ.
*   **Hệ sinh thái giải trí (Minigame):** Sau những giờ học tập căng thẳng, người dùng sẽ được bước vào không gian thư giãn với các minigame thú vị. Sự kết hợp này biến những áp lực học hành thành phần thưởng xứng đáng, giúp tái tạo năng lượng hiệu quả.

Bằng cách áp dụng vòng lặp **Kỷ luật → Hành động → Phản hồi → Phần thưởng**, dự án không chỉ giữ chân người dùng mà còn từng bước kiến tạo nên một phong cách học tập hoàn toàn mới.

## Mục lục kiến trúc

Tài liệu này sẽ đi sâu vào việc phân tích và hướng dẫn chi tiết các khía cạnh kỹ thuật của dự án, bao gồm:

1. **[Bối cảnh dự án](5.1-Overview/)**
   * Phân tích bối cảnh ra đời, mục tiêu cốt lõi và đối tượng người dùng hướng đến.
   * Chi tiết bài toán kỹ thuật cần giải quyết để đáp ứng nhu cầu thực tiễn.
2. **[Kiến trúc dự án](5.2-Architecture/)**
   * Phân tích lý do lựa chọn kiến trúc AWS Serverless và rào cản kỹ thuật.
   * Cách thiết lập phân quyền, rào chắn bảo mật và tích hợp các dịch vụ bên thứ 3.
   * Chi tiết các dịch vụ AWS được sử dụng, cách xây dựng và hình ảnh Sơ đồ kiến trúc tổng thể [drawio].
3. **[Học tập cùng với AI](5.3-Study_with_AI/)**
   * Mô tả chi tiết quy trình hoạt động của trợ lý AI và công nghệ AI Scan.
   * Cách AI xử lý dữ liệu, lợi ích của các model/extension được sử dụng và lý do phương pháp này mang lại hiệu quả cao.
   * Hướng dẫn quy trình setup từng loại AI, các chế độ học tập và hệ thống nhiệm vụ.
4. **[Hệ sinh thái game](5.4-Gamification/)**
   * Phân tích luồng chuyển tiếp mượt mà từ phân hệ Học tập sang Minigame.
   * Mô tả chung về cơ chế hoạt động của các minigame và các chức năng cốt lõi.
   * Hệ thống xếp hạng (Leaderboard) giúp tăng tính cạnh tranh và giữ chân người dùng.
5. **[Triển khai và Vận hành](5.5-Deployment/)**
   * Hướng dẫn chi tiết quy trình triển khai ứng dụng (cách đóng gói bản build, cách tải về và cài đặt).
   * Mô tả luồng thao tác thực tế và cách người dùng sử dụng hệ thống.
6. **[Định hướng và phát triển](5.6-Direction/)**
   * Nhìn nhận, đánh giá các điểm còn hạn chế cần cải thiện.
   * Lộ trình phát triển và các chức năng dự kiến sẽ được tích hợp trong tương lai.

---

## Mã nguồn

- **Frontend (ReactJS + Electron):** https://github.com/KhangChinh/AWSStudy-Play
- **Backend (AWS Serverless):** https://github.com/KhangChinh/AWSServerless