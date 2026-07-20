---
title : "Kiến trúc hệ thống"
date : 2026-07-17 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# Phân tích và Thiết kế hệ thống kiến trúc

Sau khi xác định bài toán sản phẩm ở mục 5.1, giai đoạn thiết kế kiến trúc tập trung trả lời câu hỏi: *làm sao để một ứng dụng Desktop có trải nghiệm mượt mà, vẫn đảm bảo tính toàn vẹn dữ liệu và chống gian lận ở phía Server?* Dự án Uchimi được thiết kế theo mô hình **Client–Server phân tán**, trong đó Frontend chạy cục bộ trên máy người dùng (ReactJS + Electron) và Backend vận hành hoàn toàn trên nền tảng **AWS Serverless** tại region `ap-southeast-1`.

#### Lựa chọn mô hình Client–Server: ReactJS/Electron và AWS Serverless

**Phía Client (ReactJS + Electron):** Thay vì xây dựng ứng dụng web thuần, dự án chọn Electron để đóng gói giao diện React thành ứng dụng Desktop thực sự — phù hợp với concept "OS học tập" (cửa sổ nổi, taskbar, widget Focus, Chrome Extension đồng bộ). Amplify Auth tích hợp trực tiếp với Amazon Cognito, giúp Client nhận JWT và gắn vào mọi request API mà không cần tự xử lý logic xác thực phức tạp. Quan trọng hơn, kiến trúc **Offline-First** (Redux + Electron-store) cho phép người dùng mở app và thấy dữ liệu ngay lập tức từ cache cục bộ, sau đó hệ thống mới âm thầm đồng bộ với Cloud qua API `POST /sync-all`.

**Phía Backend (AWS Serverless):** Toàn bộ business logic (Gacha, Shop, Quest, Minigame, Social...) được triển khai dưới dạng các hàm AWS Lambda độc lập, kích hoạt qua HTTP API, S3 Event, DynamoDB Streams hoặc EventBridge Schedule. Lý do chọn Serverless thay vì EC2/ECS truyền thống:

* **Scale-to-Zero:** Khi không có người dùng, hệ thống gần như không phát sinh chi phí duy trì cố định.
* **Tự động mở rộng:** Lambda và API Gateway tự scale theo tải mà không cần cấu hình Auto Scaling Group hay Load Balancer thủ công.
* **Infrastructure as Code:** Toàn bộ hạ tầng khai báo trong `serverless.yml`, deploy một lệnh `serverless deploy` → CloudFormation tạo đồng bộ Lambda, API, IAM, Logs.
* **Tích hợp sẵn:** Cognito, DynamoDB, S3, EventBridge, Streams — giảm thời gian dựng nền tảng so với tự cài đặt trên máy chủ riêng.

#### Hạn chế và rào cản của kiến trúc

Bên cạnh các ưu điểm, mô hình này cũng đặt ra nhiều thách thức kỹ thuật cần được xử lý có chủ đích:

* **Cold Start Lambda:** Request đầu tiên sau một khoảng thời gian rảnh có thể chậm hơn do Lambda phải khởi tạo runtime. Dự án giảm thiểu bằng cách giữ memory ở mức 512 MB (sweet spot giữa tốc độ và chi phí).
* **Debug phân tán:** Logic nằm rải rác trên 20 hàm Lambda cùng các trigger bất đồng bộ (Streams, S3, Cron) — theo dõi end-to-end đòi hỏi CloudWatch Logs được cấu hình đúng.
* **Thiết kế DynamoDB từ đầu:** NoSQL không hỗ trợ JOIN hay full-text search phức tạp. Mọi truy vấn phải được thiết kế quanh PK/SK và GSI ngay từ đầu; tìm kiếm người dùng theo tên phải nhờ dịch vụ ngoài (Algolia).
* **Offline-First và Race Condition:** Client cache dữ liệu local dễ gây xung đột khi đồng bộ. Backend buộc phải dùng `TransactWriteItems` cho giao dịch tiền tệ/inventory và áp dụng nguyên tắc Server Authority — không tin số liệu Client gửi lên.
* **Vendor Lock-in:** Kiến trúc gắn chặt với AWS (DynamoDB Streams, Cognito JWT Authorizer, IAM Role). Di chuyển sang cloud khác sẽ tốn chi phí refactor đáng kể.
* **Electron nặng tài nguyên:** Ứng dụng Desktop tiêu thụ RAM nhiều hơn web thuần; quá trình phân phối và cập nhật phiên bản phức tạp hơn deploy web.

#### Sơ đồ kiến trúc tổng thể

Sơ đồ dưới đây mô tả luồng giao tiếp chính giữa Client, các dịch vụ AWS và hệ thống bên thứ ba. Toàn bộ tài nguyên AWS nằm trong region `ap-southeast-1`; Algolia và Gemini API nằm ngoài khung AWS Cloud.

![Sơ đồ kiến trúc AWS](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)

Luồng chính trên sơ đồ:
1. Client đăng nhập qua **Cognito**, nhận JWT.
2. Mọi request API đi qua **API Gateway** (JWT Authorizer verify token) → **Lambda** xử lý → đọc/ghi **DynamoDB** hoặc **S3**.
3. Thay đổi bảng User → **DynamoDB Streams** → Lambda `streamIndexer` → đồng bộ **Algolia**.
4. **EventBridge** cron kích hoạt worker làm mới Shop / Leaderboard định kỳ.
5. Client xem ảnh qua **CloudFront** (CDN) lấy origin từ **S3**.
6. **IAM Execution Role** (nét đứt trên sơ đồ) cấp quyền cho Lambda truy cập DynamoDB, Streams và S3 — đây là lớp phân quyền runtime, không phải luồng dữ liệu.

#### Các dịch vụ AWS cốt lõi

Dự án sử dụng Serverless Framework (Node.js 20.x) để triển khai tự động cấu hình hạ tầng. Cấu trúc tổng thể bao gồm **20 Lambda functions**, **25 HTTP routes**, **2 EventBridge schedules**, **3 S3 triggers**, **1 DynamoDB Stream trigger** và **1 Cognito PostConfirmation trigger**.

| Dịch vụ | Mục đích sử dụng | Chi tiết cấu hình |
| :--- | :--- | :--- |
| **AWS Lambda** | Xử lý toàn bộ Business Logic | Phân chia thành 10 module: User, Session, Gacha, Minigame, Quest, Shop, Social, Upload, Currency, Sync. |
| **API Gateway** | Cổng giao tiếp HTTP API cho Client | 25 routes (GET/POST/PUT), JWT Authorizer từ Cognito, Throttling 50 req/s và 20 concurrent. |
| **Amazon Cognito** | Xác thực và định danh người dùng | User Pool + JWT; Trigger PostConfirmation tự khởi tạo profile (`handleInitUser`). |
| **Amazon DynamoDB** | Cơ sở dữ liệu NoSQL | 8 bảng: User, Study, Social, Quest, Minigame, ItemData, Inventory, GachaHistory. |
| **DynamoDB Streams** | Bắt sự kiện thay đổi dữ liệu realtime | Kích hoạt Lambda `streamIndexer` trên User table → đồng bộ Algolia. |
| **Amazon S3** | Lưu trữ tài nguyên tĩnh | Bucket `ASSETS_BUCKET`: `public-assets/`, `avatars/`, `uploads/`; trigger Lambda khi upload `.zip`/`.json`. |
| **Amazon CloudFront** | CDN phân phối ảnh/assets | Origin trỏ về S3; Client xem avatar/item qua URL CDN thay vì gọi S3 trực tiếp. |
| **Amazon EventBridge** | Lập lịch tác vụ tự động (Cron) | Cập nhật Leaderboard mỗi 10 phút; làm mới Shop hàng tuần (Chủ Nhật 17:00 UTC). |
| **Amazon CloudWatch** | Giám sát và Logs | Tự động tạo Log Group cho mỗi Lambda; hỗ trợ debug và theo dõi hiệu năng. |
| **AWS IAM** | Quản lý quyền truy cập | Execution Role gắn vào Lambda; cấp quyền least-privilege qua `serverless.yml`. |
| **AWS CloudFormation** | Provision hạ tầng (qua Serverless) | Mọi tài nguyên gom thành một Stack; rollback/xóa sạch bằng một thao tác. |

> **Ghi chú về Region/AZ:** Hệ thống không sử dụng VPC hay EC2, nên không cần cấu hình Availability Zone thủ công. AWS tự quản lý multi-AZ cho các dịch vụ managed (Lambda, DynamoDB, S3) bên trong region `ap-southeast-1`.

#### Tích hợp dịch vụ bên thứ ba

Bên cạnh hệ sinh thái AWS, dự án tích hợp hai dịch vụ ngoài để bổ sung khả năng mà AWS không cung cấp sẵn ở mức đơn giản:

**Algolia (User Search Index):** DynamoDB không hỗ trợ full-text search theo tên người dùng. Algolia được chọn thay OpenSearch vì API đơn giản, kết quả gần realtime và không đòi hỏi vận hành cluster riêng — phù hợp quy mô dự án thực tập. Luồng đồng bộ: mỗi khi bảng User thay đổi, DynamoDB Streams kích hoạt Lambda `streamIndexer`, hàm này index document lên Algolia. Client gọi `GET /friends/search` → Lambda truy vấn Algolia và trả kết quả.

**Google Gemini API (AI Study Planner):** Hỗ trợ sinh lộ trình học và quiz từ tài liệu người dùng upload. Tuy nhiên, hệ thống không ép buộc dùng Cloud AI — người dùng có thể chọn **Ollama (Local)** để xử lý hoàn toàn offline trên máy cá nhân (chi tiết ở mục 5.3). Gemini chỉ được dùng khi người dùng tự cấu hình API Key, giúp ứng dụng không gánh chi phí LLM tập trung.

#### Luồng giao tiếp giữa các thành phần

Hệ thống vận hành theo ba loại luồng chính:

**Luồng đồng bộ (Request/Response):** Đây là luồng phổ biến nhất, chiếm phần lớn 25 HTTP routes. Client gửi request kèm JWT → API Gateway verify token qua Cognito Authorizer → invoke Lambda tương ứng → Lambda đọc/ghi DynamoDB hoặc S3 → trả JSON về Client. Ví dụ: mua item Shop, claim quest, đổi cosmetic, sync profile.

**Luồng bất đồng bộ (Event-Driven):** Các sự kiện phát sinh từ thay đổi dữ liệu hoặc lịch trình, không cần Client chủ động gọi:
* **DynamoDB Streams → streamIndexer → Algolia:** Đồng bộ search index khi User table thay đổi.
* **S3 ObjectCreated → processZip / processJson:** Dev upload file `.zip` item hoặc `.json` config → Lambda tự bung, đẩy assets lên S3 và ghi metadata vào ItemData table.
* **EventBridge Cron → Shop/Leaderboard worker:** Tự động rotate shop hàng tuần, cập nhật bảng xếp hạng mỗi 10 phút.
* **Cognito PostConfirmation → handleInitUser:** Tự khởi tạo profile, inventory mặc định khi user đăng ký lần đầu.

**Luồng phân phối tài nguyên (CDN):** Client không gọi API để xem ảnh avatar hay item asset. Thay vào đó, Client truy cập trực tiếp URL CloudFront → CloudFront lấy file gốc từ S3 (origin) nếu chưa có trong cache. Lambda chỉ tham gia khi cần tạo presigned URL upload hoặc xử lý file mới.

#### Cấu hình bảo mật và Quyền truy cập (IAM Permissions)

Nguyên tắc **Least Privilege** được áp dụng nghiêm ngặt thông qua file `serverless.yml`. Lambda Execution Role chỉ được cấp đúng các Action trên đúng Resource cần thiết:

* **DynamoDB:** `GetItem`, `PutItem`, `UpdateItem`, `DeleteItem`, `Query`, `Scan`, `BatchGetItem`, `BatchWriteItem`, `TransactWriteItems` — giới hạn trên 8 bảng hệ thống và GSI tương ứng.
* **DynamoDB Streams:** `DescribeStream`, `GetRecords`, `GetShardIterator`, `ListStreams` — chỉ trên ARN của User Table Stream.
* **Amazon S3:** `GetObject`, `PutObject` — chỉ trong các prefix `public-assets/*`, `avatars/*`, `uploads/*`.

Ở tầng API, mọi route nhạy cảm gắn authorizer `myCognitoAuth` (JWT). API Gateway kiểm tra token với Cognito User Pool trước khi cho phép invoke Lambda — Client không thể giả mạo user_id. CORS được bật trên HTTP API; Throttling giới hạn 50 requests/giây và 20 concurrent requests để chống spam/DDoS nhẹ.

Cần phân biệt hai lớp IAM khi vận hành:
* **Execution Role** (trong sơ đồ): quyền runtime của Lambda khi đang chạy — khai báo trong `provider.iam.role.statements`.
* **Deploy Role** (của developer): quyền tạo/cập nhật CloudFormation, Lambda, API Gateway — dùng khi chạy `serverless deploy`, không xuất hiện trên sơ đồ kiến trúc.

Biến môi trường nhạy cảm (`ALGOLIA_WRITE_KEY`, `COGNITO_USER_POOL_ID`, `COGNITO_CLIENT_ID`...) được load qua `useDotenv: true` từ file `.env` cục bộ — không commit lên GitHub.

#### Quy trình thiết lập và triển khai hạ tầng

Việc đưa Backend lên AWS được thực hiện theo các bước:

1. **Chuẩn bị tài nguyên nền:** Tạo Cognito User Pool, 8 bảng DynamoDB (bật Stream trên User table), S3 bucket và (tuỳ chọn) CloudFront distribution trước khi deploy Lambda.
2. **Cấu hình `.env`:** Điền tên bảng, bucket, Cognito ID, Stream ARN, Algolia credentials vào `AWSServerless/.env`.
3. **Khai báo IaC:** Toàn bộ Lambda, API routes, IAM, triggers được định nghĩa trong `serverless.yml` và các file `function.yml` con (user, shop, gacha, social, upload...).
4. **Deploy một lệnh:**

```bash
cd AWSServerless
serverless deploy
```

Serverless Framework tự biên dịch code (esbuild bundle + minify), tạo CloudFormation template và provision tài nguyên đồng bộ trên region `ap-southeast-1`.

5. **Kết nối Frontend:** Cập nhật `AWSStudy-Play/.env` trỏ `VITE_API_URL`, `VITE_COGNITO_*`, `VITE_S3_ASSETS_URL` về endpoint thực tế sau deploy.

#### Phân tách module Lambda và cơ chế kích hoạt

Backend được chia thành 10 module độc lập, mỗi module có file `function.yml` riêng khai báo handler và loại trigger:

| Module | Trigger | Chức năng chính |
| :--- | :--- | :--- |
| **userFunction** | Cognito PostConfirmation + HTTP API | Khởi tạo profile; cập nhật tên; trang bị cosmetic; upload avatar |
| **sessionFunction** | HTTP API | Quản lý phiên học / Focus session |
| **currencyFunction** | HTTP API | Quy đổi Knowledge Point → Knowledge Core |
| **gachaFunction** | HTTP API | Roll gacha server-side, pity, ghi Inventory + GachaHistory |
| **minigameFunction** | HTTP API + EventBridge Schedule | Sudoku/Minesweeper; worker leaderboard định kỳ |
| **questFunction** | HTTP API | Daily quest, claim reward, cập nhật streak |
| **shopFunction** | HTTP API + EventBridge Schedule (cron) | Lấy shop, mua item eCoin; rotate shop hàng tuần |
| **socialFunction** | HTTP API + DynamoDB Stream | Bạn bè, kết bạn; search Algolia; `streamIndexer` |
| **syncFunction** | HTTP API | `sync-all` / sync profile-inventory cho offline-first |
| **uploadFunction** | S3 ObjectCreated (`.zip` / `.json`) | Ingest item master data + assets vào S3 và ItemData table |

Quy trình đẩy item mới vào hệ thống (ví dụ background Study Plant) cũng tuân theo luồng event-driven: Developer đóng gói item thành file ZIP (gồm `data.json` + assets) → upload lên `s3://ASSETS_BUCKET/uploads/items/` → S3 trigger Lambda `processZip` → Lambda bung ZIP, đẩy file lên `public-assets/items/` và ghi metadata vào bảng ItemData. Nếu item có `collectFrom: "eCoinShop"`, shop worker sẽ tự nhặt item đó khi rotate shop hàng tuần.
