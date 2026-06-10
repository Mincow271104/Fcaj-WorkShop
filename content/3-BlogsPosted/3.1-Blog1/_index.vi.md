---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

# SESSION POLICIES TRONG AMAZON EKS POD IDENTITY

Amazon EKS Pod Identity vừa bổ sung tính năng session policies, cho phép bạn thu hẹp quyền IAM một cách linh hoạt và chính xác cho từng pod mà không cần tạo thêm nhiều IAM roles riêng biệt. Đây là bước tiến quan trọng giúp áp dụng nguyên tắc least privilege hiệu quả hơn trong môi trường Kubernetes quy mô lớn.

Các điểm chính cần nắm:

* Session policy là một IAM policy inline được chỉ định khi tạo hoặc cập nhật Pod Identity association.
* Quyền hiệu quả = intersection (giao) giữa permissions của IAM role và session policy → session policy chỉ có thể thu hẹp, không thể mở rộng quyền.
* Giúp tránh tình trạng over-permissioning khi reuse chung một IAM role cho nhiều workloads có nhu cầu khác nhau.
* Hỗ trợ cả same-account và cross-account (qua IAM role chaining).
* Giảm đáng kể số lượng IAM roles cần quản lý, tránh chạm giới hạn quota IAM trong cluster lớn.
* Cấu hình dễ dàng qua AWS Management Console, AWS CLI hoặc AWS SDK khi tạo association giữa Kubernetes ServiceAccount và IAM role.

Tính năng này đặc biệt hữu ích khi bạn có nhiều ứng dụng chạy trên cùng một IAM role nhưng cần giới hạn quyền khác nhau (ví dụ: một pod chỉ đọc S3 bucket cụ thể, pod khác chỉ gọi một số API nhất định).

...Hình ảnh...

...Link...

...Hướng dẫn...