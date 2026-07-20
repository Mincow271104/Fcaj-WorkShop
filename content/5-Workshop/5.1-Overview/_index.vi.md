---
title: "Tổng quan"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Bối cảnh dự án và bài toán thiết kế hệ thống

## Bối cảnh dự án

Một trong những khó khăn lớn của quá trình tự học không nằm ở khả năng tiếp cận kiến thức mà ở việc duy trì sự tập trung và động lực trong khoảng thời gian đủ dài để đạt được kết quả có ý nghĩa. Thành quả học tập thường xuất hiện chậm và khó quan sát ngay, trong khi mạng xã hội hoặc các ứng dụng giải trí liên tục cung cấp phản hồi tức thời. Vì vậy, người học có thể nhận thức rõ lợi ích của việc học nhưng vẫn gặp khó khăn khi bắt đầu, duy trì lịch học hoặc nhận ra sự tiến bộ của bản thân.

Hiện tượng này có liên quan một phần đến hệ thống học tập dựa trên phần thưởng của não bộ. Dopamine không nên được hiểu đơn giản là “chất tạo cảm giác vui”; nó còn tham gia vào quá trình hình thành kỳ vọng, học hỏi từ kết quả và củng cố những hành vi được xem là có giá trị. Khi một hành động nhận được phản hồi rõ ràng, chẳng hạn như sự tiến bộ có thể quan sát, một thành tích hoặc phần thưởng có ý nghĩa, não bộ có xu hướng liên kết hành động đó với kết quả tích cực. Phản hồi phù hợp và được lặp lại có thể hỗ trợ hình thành vòng lặp hành vi:

> **Hành động → Phản hồi → Phần thưởng → Động lực lặp lại**

Uchimi StudyGamification vận dụng nguyên lý này thông qua phiên tập trung, nhiệm vụ hằng ngày, chỉ số tiến độ, thành tích, phần thưởng, minigame và tương tác xã hội. Các yếu tố trên không nhằm thay thế giá trị của kiến thức hoặc biến học tập thành hoạt động thu thập phần thưởng. Thay vào đó, chúng giúp giảm rào cản tâm lý khi bắt đầu, làm cho tiến bộ trở nên rõ ràng và hỗ trợ người dùng từng bước chuyển từ động lực bên ngoài sang động lực nội tại cùng một thói quen học tập bền vững.

## Phát biểu bài toán và mục tiêu dự án

Các công cụ học tập truyền thống thường tập trung vào cung cấp nội dung và quản lý công việc nhưng chưa hỗ trợ đầy đủ việc duy trì động lực dài hạn. Ngược lại, một hệ thống phần thưởng thuần giải trí có thể thu hút người dùng nhưng không tạo ra kết quả học tập thực chất. Vì vậy, Uchimi cần làm cho việc học đủ hấp dẫn để người dùng tự nguyện tham gia, đồng thời bảo đảm hệ thống gamification trên cloud luôn công bằng, nhất quán, phản hồi nhanh, an toàn, có khả năng mở rộng và tối ưu chi phí.

Dự án xây dựng một ứng dụng học tập trên máy tính, kết hợp **ReactJS và Electron ở phía client** với **kiến trúc AWS Serverless ở phía backend**. Mục tiêu trung tâm là giúp việc học trở thành một hoạt động tự nguyện, có thể lặp lại thường xuyên bằng cách mang đến cho mỗi phiên học một mục tiêu rõ ràng, phản hồi kịp thời và cảm giác đạt được thành quả. Các mục tiêu cụ thể gồm:

- chia mục tiêu dài hạn thành những hành động nhỏ có thể thực hiện hằng ngày;
- trực quan hóa tiến bộ qua thời gian tập trung, chuỗi ngày học, nhiệm vụ, điểm số và thành tích;
- sử dụng phần thưởng và minigame như sự củng cố tích cực sau hoạt động học tập có ý nghĩa;
- hỗ trợ lập kế hoạch cá nhân hóa và ôn tập kiến thức bằng các tính năng có AI hỗ trợ;
- xây dựng môi trường tương tác xã hội công bằng thông qua bạn bè và bảng xếp hạng; và
- cung cấp trải nghiệm phản hồi nhanh với hạ tầng có thể mở rộng nhưng vẫn kiểm soát chi phí.

Dự án không đặt mục tiêu ép buộc người dùng học nhiều hơn. Hệ thống hướng đến việc tạo điều kiện để họ sẵn sàng bắt đầu, nhìn thấy sự tiến bộ và chủ động quay lại học tập thường xuyên.

## Đối tượng hướng đến

### Học sinh và sinh viên

Học sinh, sinh viên thường phải tiếp nhận lượng kiến thức lớn, hoàn thành bài tập đúng thời hạn và chuẩn bị cho các kỳ thi. Những khó khăn phổ biến là mất tập trung, trì hoãn, áp lực và khó đo lường tiến bộ mỗi ngày. Uchimi giúp chuyển các mục tiêu tổng quát thành phiên tập trung và nhiệm vụ hằng ngày, hỗ trợ ôn tập bằng bài kiểm tra và cung cấp phản hồi ngay sau từng hoạt động. Chế độ học xếp hạng cùng thống kê tiến độ cũng tạo ra cấu trúc rõ ràng hơn cho người dùng cần tăng tính kỷ luật.

### Người đi làm

Người đi làm thường học để nâng cao kỹ năng chuyên môn, học ngoại ngữ, chuẩn bị chứng chỉ hoặc theo đuổi sở thích cá nhân. Hạn chế chính của nhóm này là quỹ thời gian ít và bị chia nhỏ, thay vì có thời khóa biểu cố định. Uchimi hỗ trợ họ bằng kế hoạch linh hoạt, phiên học ngắn, khả năng đồng bộ tiến độ và các tính năng học tập có AI hỗ trợ, qua đó giúp duy trì việc học mà không tạo thêm một gánh nặng sau giờ làm việc.

### Giá trị chung dành cho hai nhóm

Mặc dù có lịch trình và mục tiêu khác nhau, cả hai nhóm đều cần được hỗ trợ để bắt đầu công việc, duy trì tính đều đặn và nhận biết sự tiến bộ. Uchimi chuyển mục tiêu dài hạn thành các hoạt động vừa sức, ghi nhận nỗ lực theo thời gian và kết nối việc học với phản hồi cũng như hoạt động giải trí phù hợp. Người dùng vẫn làm chủ mục tiêu và tự lựa chọn nhịp độ phù hợp với hoàn cảnh cá nhân. Cách tiếp cận tự nguyện, lấy người dùng làm trung tâm là điều kiện quan trọng để hình thành thói quen bền vững thay vì chỉ tạo sự hứng thú ngắn hạn.

## Các bài toán kỹ thuật cốt lõi

### 1. Chống gian lận trong hoạt động học tập

Thời lượng học, trạng thái hoàn thành, điểm xếp hạng, tiến độ nhiệm vụ và phần thưởng không thể được quyết định hoàn toàn từ dữ liệu client gửi lên, bởi ứng dụng máy tính có thể bị chỉnh sửa và request có thể bị phát lại. Server phải giữ vai trò quyết định đối với các quy tắc quan trọng. Mỗi phiên học vì vậy cần gắn với người dùng đã xác thực, thời điểm do server ghi nhận, quá trình chuyển trạng thái hợp lệ, cơ chế strike được kiểm soát và kết quả được tính ở phía server. Request trùng lặp, hết hạn, sai thứ tự hoặc không phù hợp với trạng thái đã lưu phải bị từ chối hoặc được xử lý an toàn. Cơ chế này bảo vệ không chỉ phần thưởng mà còn độ tin cậy của streak, kết quả học xếp hạng và bảng xếp hạng.

### 2. Chống gian lận trong minigame

Minigame có mô hình rủi ro khác với phiên học: người dùng có thể sửa điểm, gửi đáp án đã biết, tự động hóa thao tác, thay đổi thời gian hoặc nhận một kết quả nhiều lần. Cấu hình trò chơi và đáp án cần được server sinh hoặc kiểm chứng. Việc xác minh có thể kết hợp độ chính xác của đáp án, thời gian do server ghi nhận, khoảng hoàn thành hợp lý, mẫu thao tác, giới hạn phiên và cơ chế chỉ quyết toán phần thưởng một lần. Kết quả có dấu hiệu bất thường không được ảnh hưởng đến tiền tệ, tiến độ nhiệm vụ hoặc bảng xếp hạng. Các kiểm tra phải bảo đảm công bằng nhưng không theo dõi client quá mức làm tăng độ trễ và chi phí lưu trữ.

### 3. Tính toàn vẹn và nhất quán của dữ liệu trên cloud

Tiến độ học tập, Knowledge Point, tiền tệ trong hệ thống, kho vật phẩm, phần thưởng, lịch sử Gacha và quan hệ bạn bè là các dữ liệu có liên hệ với nhau. Một request hết thời gian chờ hoặc được gửi lại không được làm nhân đôi vật phẩm, trao thưởng hai lần hoặc khiến số dư bị âm. Hệ thống vì vậy cần sử dụng quy tắc do server quyết định, kiểm tra đầu vào, xử lý idempotent, cập nhật có điều kiện và bộ đếm nguyên tử của DynamoDB, cùng transaction cho những thao tác phải cùng thành công hoặc cùng thất bại. Các thao tác kinh tế quan trọng cũng cần lịch sử có khả năng truy vết. Những cơ chế này đặc biệt cần thiết đối với mua vật phẩm, chuyển đổi tiền tệ, nhận thưởng nhiệm vụ, Gacha và cập nhật quan hệ bạn bè hai chiều.

### 4. Đồng bộ hiệu quả giữa client và server

Ứng dụng máy tính cần hiển thị thông tin hữu ích ngay khi khởi động, kể cả khi mạng chậm hoặc tạm thời gián đoạn. Tuy nhiên, việc client liên tục gọi từng API để kiểm tra dữ liệu mới sẽ làm tăng độ trễ, lưu lượng mạng, số lần thực thi Lambda và lượt đọc cơ sở dữ liệu. Uchimi cần một chiến lược thân thiện với chế độ ngoại tuyến, kết hợp nạp cache cục bộ, phát hiện thay đổi theo phiên bản hoặc thời điểm cập nhật, endpoint đồng bộ tổng hợp, cập nhật gia tăng và chính sách hết hạn cho dữ liệu có thể chấp nhận độ trễ. Xử lý theo sự kiện trong backend giúp tự động lan truyền các thay đổi nội bộ. Cơ chế giải quyết xung đột phải phân biệt dữ liệu có thể hợp nhất tại client với dữ liệu quyết định như số dư, phần thưởng và kết quả xếp hạng—những dữ liệu luôn phải lấy server làm nguồn tin cậy cuối cùng.

### 5. Tối ưu chi phí vận hành cloud server

Mức độ hoạt động của người dùng thay đổi theo thời gian, vì vậy hạ tầng được cấp phát cố định có thể gây lãng phí khi ít truy cập. Các dịch vụ AWS Serverless như Lambda, API Gateway, DynamoDB, S3 và EventBridge có thể mở rộng theo nhu cầu và tính phí theo mức sử dụng. Tuy nhiên, serverless không đồng nghĩa chi phí luôn thấp. Hệ thống cần giảm API call trùng lặp, gộp thao tác tương thích, cache dữ liệu master ít thay đổi, tránh quét DynamoDB không cần thiết, thiết kế index phù hợp, tự động xóa bản ghi tạm bằng TTL và chuyển công việc không khẩn cấp sang xử lý theo lịch hoặc bất đồng bộ. Giới hạn tần suất API, chỉ số vận hành và cảnh báo chi phí giúp bảo vệ nền tảng trước tải xử lý kém hiệu quả hoặc request tăng đột biến.

## Các yêu cầu xuyên suốt

- **Bảo mật:** xác thực bằng Amazon Cognito, bảo vệ API bằng JWT, kiểm tra dữ liệu bên ngoài và áp dụng quyền IAM theo nguyên tắc đặc quyền tối thiểu.
- **Khả năng quan sát:** sử dụng log có cấu trúc, chỉ số vận hành, theo dõi lỗi và cảnh báo chi phí.
- **Khả năng mở rộng và bảo trì:** phân tách backend theo miền nghiệp vụ, quản lý hạ tầng dưới dạng mã nguồn và kiểm soát phiên bản giao tiếp client–server.
- **Quyền riêng tư và thiết kế có trách nhiệm:** chỉ thu thập dữ liệu cần thiết và bảo đảm gamification luôn phục vụ, không lấn át mục tiêu học tập.

## Giá trị kỳ vọng của dự án

Uchimi minh họa khả năng kết hợp thiết kế hành vi, học tập có AI hỗ trợ, công nghệ ứng dụng máy tính và kiến trúc cloud-native trong một hệ thống giáo dục thực tiễn. Đối với người dùng, nền tảng hỗ trợ tinh thần tự nguyện, trực quan hóa tiến bộ và hình thành thói quen dài hạn. Về mặt kỹ thuật, dự án là một trường hợp nghiên cứu về nền tảng gamification trên AWS với server làm nguồn dữ liệu tin cậy, bảo đảm tính nhất quán, thân thiện với chế độ ngoại tuyến và kiểm soát chi phí vận hành.
