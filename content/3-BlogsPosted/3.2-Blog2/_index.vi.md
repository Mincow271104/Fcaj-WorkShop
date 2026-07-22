---
title: "AWS Well-Architected: Chi Phí Ẩn Trong Kiến Trúc Cloud"
date: 2026-07-21
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
## Chia sẻ về AWS Well-Architected Framework: Những “chi phí ẩn” trong kiến trúc Cloud

## Giới thiệu

Đây là bài blog thứ hai trong quá trình mình tìm hiểu về Amazon Web Services và cách xây dựng hệ thống trên nền tảng Cloud. Ở bài viết này, mình muốn tìm hiểu sâu hơn về việc một kiến trúc AWS không chỉ cần hoạt động được mà còn phải bảo đảm các yếu tố như bảo mật, độ tin cậy, hiệu suất, chi phí và khả năng vận hành lâu dài.

Trong quá trình tìm kiếm tài liệu, mình đã đọc bài viết **“The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework”** trên AWS Architecture Blog (03/03/2026, Ryan Dsouza và Bradley Acar). Nội dung bài viết khiến mình chú ý vì nó đề cập đến những chi phí không xuất hiện trực tiếp trên hóa đơn AWS nhưng vẫn có thể gây ảnh hưởng lớn đến hệ thống, chẳng hạn như sự cố bảo mật, downtime, mất dữ liệu hoặc sử dụng tài nguyên không hiệu quả.

![Hình 1 – Bài blog gốc](/images/3-BlogsPosted/3.2-Blog2/hinh_1_bai_blog_goc.png)

Thông qua Blog 2 này, mình sẽ tóm tắt nội dung chính của bài viết, nêu ra những điểm mạnh và hạn chế theo cảm nhận cá nhân, đồng thời liên hệ với kiến trúc đồ án AWS mà nhóm mình đang thực hiện.

**Bài viết gốc:**  
[The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)

---

## 1. Nội dung chính của bài viết

AWS Well-Architected Framework là bộ hướng dẫn giúp người xây dựng hệ thống hiểu được ưu điểm và nhược điểm của các quyết định kiến trúc. Framework cũng giúp đánh giá kiến trúc theo các best practices của AWS và xác định những phần cần cải thiện.

AWS Well-Architected Framework dựa trên sáu trụ cột:

- **Operational Excellence** – Vận hành hiệu quả
- **Security** – Bảo mật
- **Reliability** – Độ tin cậy
- **Performance Efficiency** – Hiệu suất
- **Cost Optimization** – Tối ưu chi phí
- **Sustainability** – Tính bền vững

![Hình 2 – Sáu trụ cột của AWS Well-Architected Framework](/images/3-BlogsPosted/3.2-Blog2/hinh_2_sau_tru_cot.png)

*Hình 2. Sáu trụ cột của AWS Well-Architected Framework. Nguồn: Amazon Web Services.*

![Hình 2b – AWS Well-Architected Framework](https://res.cloudinary.com/dakqspssm/image/upload/v1784652986/well-architected-framework_er6nc7.png)

*Hình 2b. AWS Well-Architected Framework – sáu trụ cột hỗ trợ Business Outcomes.*

Nguồn tham khảo:  
[The pillars of the AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)

Theo mình, sáu trụ cột này cho thấy một hệ thống tốt không thể chỉ tập trung duy nhất vào chi phí hoặc hiệu năng. Các yếu tố bảo mật, độ tin cậy, vận hành và tính bền vững cần được cân bằng trong quá trình thiết kế.

Bài viết tập trung vào ba khu vực mà kiến trúc chưa tối ưu có thể tạo ra chi phí ẩn: bảo mật, khả năng sẵn sàng và hiệu quả sử dụng tài nguyên.

![Hình 3 – Ba nhóm chi phí ẩn](/images/3-BlogsPosted/3.2-Blog2/hinh_3_ba_chi_phi_an.png)

### 1.1. Chi phí từ sự cố bảo mật

Khi phân quyền IAM quá rộng, không bật MFA, không mã hóa dữ liệu hoặc không giám sát hệ thống đầy đủ, tổ chức có thể đối mặt với truy cập trái phép hay sự cố bảo mật. Hậu quả không chỉ là chi phí khắc phục kỹ thuật mà còn có thể ảnh hưởng đến uy tín, mục tiêu kinh doanh và yêu cầu tuân thủ.

Bài viết đề xuất áp dụng nguyên tắc **least privilege**, xác thực đa yếu tố, kiểm tra policy định kỳ, mã hóa dữ liệu khi lưu trữ và truyền tải, bảo vệ hạ tầng, liên tục giám sát và xây dựng kế hoạch phản ứng sự cố.

### 1.2. Chi phí do hệ thống ngừng hoạt động

Một hệ thống thiếu khả năng dự phòng có thể bị gián đoạn khi một thành phần gặp lỗi. Theo bài viết, downtime có thể gây mất năng suất, mất doanh thu, không đáp ứng SLA và khiến khách hàng không hài lòng. Trong một số trường hợp, tổ chức còn phải chịu tiền phạt hoặc thuê chuyên gia bên ngoài để khắc phục.

AWS Well-Architected khuyến nghị thiết kế hệ thống có khả năng chịu lỗi, sử dụng cơ chế dự phòng và tự động failover, tự động mở rộng theo nhu cầu, sao lưu dữ liệu và thường xuyên kiểm tra kế hoạch phục hồi.

### 1.3. Chi phí do sử dụng tài nguyên không hiệu quả

Một lỗi phổ biến là cấp quá nhiều CPU, RAM hoặc dung lượng lưu trữ để tránh hệ thống bị chậm. Tuy nhiên, nhiều workload không hoạt động liên tục 24/7; có workload ngừng vào cuối tuần, chỉ chạy vài ngày mỗi tháng hoặc thay đổi theo mùa.

Bài viết khuyến nghị **right-sizing** tài nguyên, loại bỏ tài nguyên không sử dụng, theo dõi chi phí thường xuyên và tìm hiểu các mô hình giá như On-Demand, Reserved Instances, Savings Plans và Spot Instances.

---

## 2. Điểm mạnh của bài viết theo cảm nhận của mình

Điểm mình đánh giá cao nhất là bài viết không xem tối ưu chi phí đơn giản là tìm dịch vụ AWS rẻ hơn. Một quyết định tiết kiệm trước mắt nhưng làm giảm bảo mật hoặc độ tin cậy có thể tạo ra chi phí lớn hơn trong tương lai.

Bài viết liên kết khá tốt giữa quyết định kiến trúc và tác động đến doanh nghiệp. Một cấu hình sai không chỉ làm hệ thống bị lỗi mà còn có thể ảnh hưởng đến doanh thu, trải nghiệm khách hàng, SLA, uy tín và yêu cầu tuân thủ.

Các giải pháp được đề cập khá cụ thể như:

- Least privilege
- MFA
- Mã hóa dữ liệu
- Monitoring
- Backup
- Disaster recovery
- Auto Scaling
- Right-sizing tài nguyên

Mình cũng thích quan điểm cho rằng việc đánh giá kiến trúc không phải là một cuộc kiểm toán để tìm lỗi cá nhân. Tài liệu AWS mô tả quá trình review nên được thực hiện nhất quán, theo hướng không đổ lỗi, nhẹ nhàng và mang tính trao đổi.

Nguồn tham khảo:  
[The review process](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-review-process.html)

---

## 3. Điểm hạn chế theo cảm nhận của mình

Bài viết trình bày khá rộng nhưng chưa có một kiến trúc mẫu hoàn chỉnh để người đọc quan sát trước và sau khi tối ưu. Nếu có thêm sơ đồ, số liệu chi phí và kết quả sau khi khắc phục thì nội dung sẽ dễ hình dung hơn.

Bài viết đưa ra nhiều khuyến nghị nhưng chưa chỉ rõ nhóm sinh viên hoặc doanh nghiệp nhỏ nên thực hiện phần nào trước. Với người mới học AWS, việc áp dụng tất cả best practices cùng lúc có thể khiến kiến trúc trở nên phức tạp.

Theo quan điểm cá nhân, nhóm sinh viên có thể bắt đầu từ những việc cơ bản:

1. Bật MFA.
2. Kiểm tra IAM Policy.
3. Chặn public access không cần thiết trên S3.
4. Thiết lập AWS Budget.
5. Bật logging và monitoring.
6. Kiểm tra backup.
7. Xóa tài nguyên không còn sử dụng.

Ngoài ra, do đây là bài viết chính thức của AWS nên giải pháp chủ yếu dựa trên dịch vụ và công cụ của AWS. Bài viết chưa so sánh nhiều với cách tiếp cận kiến trúc của Azure hoặc Google Cloud.

---

## 4. Liên hệ với đồ án của nhóm mình

Đồ án của nhóm mình hiện sử dụng các dịch vụ như:

- Amazon Cognito
- Amazon S3
- Amazon CloudFront
- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB

Kiến trúc tham khảo trong Serverless Applications Lens của AWS cũng sử dụng các dịch vụ tương tự.

![Hình 4 – Kiến trúc tham khảo](/images/3-BlogsPosted/3.2-Blog2/hinh_4_kien_truc_do_an.png)

Nguồn kiến trúc tham khảo:  
[Serverless Applications Lens – Web application](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/web-application.html)

Sau khi đọc bài viết, mình nhận ra rằng việc hệ thống chạy được chưa có nghĩa là kiến trúc đã tốt. Với chức năng lưu trữ ảnh đại diện trên S3, nhóm không chỉ cần upload và hiển thị ảnh thành công mà còn cần xem xét:

- Bucket S3 có đang bị public không cần thiết hay không.
- IAM Role đã được cấp đúng quyền tối thiểu hay chưa.
- Người dùng có thể sửa hoặc xóa ảnh của người khác hay không.
- Dữ liệu có được mã hóa, giám sát và ghi log phù hợp hay không.
- CloudFront có được cấu hình cache phù hợp hay không.
- Có giới hạn định dạng và dung lượng file upload hay không.
- Có lifecycle policy hoặc cơ chế xóa ảnh không còn sử dụng hay không.
- Hệ thống xử lý thế nào khi upload lên S3 thành công nhưng cập nhật DynamoDB thất bại.

Đây chính là những câu hỏi mà Well-Architected Framework hướng người thiết kế phải suy nghĩ trước khi đưa ứng dụng vào sử dụng thực tế.

---

## 5. Kết luận

Theo cảm nhận của mình, giá trị lớn nhất của AWS Well-Architected Framework là giúp chúng ta chuyển từ tư duy:

> **“Xây dựng hệ thống cho chạy được”**

sang:

> **“Xây dựng hệ thống an toàn, ổn định, hiệu quả và có thể phát triển lâu dài.”**

Một kiến trúc tốt không nhất thiết phải sử dụng thật nhiều dịch vụ AWS. Quan trọng hơn là lựa chọn dịch vụ phù hợp và hiểu rõ sự đánh đổi giữa bảo mật, hiệu suất, độ tin cậy và chi phí.

Well-Architected Framework Review gồm ba giai đoạn chính:

1. **Prepare**
2. **Review**
3. **Improve**

![Hình 5 – Quy trình WAFR](/images/3-BlogsPosted/3.2-Blog2/hinh_5_quy_trinh_wafr.png)

Nguồn tham khảo:  
[AWS Well-Architected Framework Review](https://docs.aws.amazon.com/wellarchitected/latest/userguide/wa-framework-review.html)

Qua bài viết này, mình rút ra rằng những vấn đề tốn kém nhất đôi khi không xuất hiện ngay trên hóa đơn AWS. Chúng có thể nằm trong một quyền IAM được cấu hình quá rộng, một bản backup chưa từng được kiểm tra, một dịch vụ thiếu khả năng phục hồi hoặc một tài nguyên vẫn hoạt động dù không còn được sử dụng.

---

## Nguồn tham khảo

1. [AWS Architecture Blog – The Hidden Price Tag](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)
2. [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
3. [The pillars of the framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html)
4. [The review process](https://docs.aws.amazon.com/wellarchitected/latest/framework/the-review-process.html)
5. [Serverless Applications Lens – Web application](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/web-application.html)
6. [AWS Well-Architected Framework Review](https://docs.aws.amazon.com/wellarchitected/latest/userguide/wa-framework-review.html)

> Ghi chú: Các hình trong bài là hình minh họa do người viết tổng hợp dựa trên nội dung tài liệu chính thức AWS; không phải ảnh chụp nguyên bản từ tài liệu AWS.

*Nguồn: AWS Architecture Blog*
*Tài liệu gốc: [The Hidden Price Tag: Uncovering Hidden Costs in Cloud Architectures with the AWS Well-Architected Framework](https://aws.amazon.com/blogs/architecture/the-hidden-price-tag-uncovering-hidden-costs-in-cloud-architectures-with-the-aws-well-architected-framework/)*