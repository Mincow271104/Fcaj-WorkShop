---
title : "Kiến trúc hệ thống"
date : 2026-07-17 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# Phân tích và Thiết kế hệ thống kiến trúc

Sau khi xác định bài toán sản phẩm ở mục 5.1, giai đoạn thiết kế kiến trúc tập trung trả lời câu hỏi: *làm sao để một ứng dụng Desktop có trải nghiệm mượt mà, vẫn đảm bảo tính toàn vẹn dữ liệu và chống gian lận ở phía Server?* Dự án được thiết kế theo mô hình **Client–Server phân tán**, trong đó Frontend chạy cục bộ trên máy người dùng (ReactJS + Electron) và Backend vận hành hoàn toàn trên nền tảng **AWS Serverless** tại region **ap-southeast-1**.

#### Lựa chọn mô hình Client–Server: ReactJS/Electron và AWS Serverless

**Phía Client (ReactJS + Electron):** Thay vì xây dựng ứng dụng web thuần, dự án chọn Electron để đóng gói giao diện React thành ứng dụng Desktop thực sự — phù hợp với concept "OS học tập" (cửa sổ nổi, taskbar, widget Focus). Trên sơ đồ, khối CLIENT gồm **Electron App** và **Focus Guard** (Chrome Extension); hai thành phần giao tiếp nội bộ qua **WebSocket (WS)** để đồng bộ trạng thái tập trung. Amplify Auth tích hợp trực tiếp với Amazon Cognito, giúp Client nhận JWT và gắn vào mọi request API mà không cần tự xử lý logic xác thực phức tạp. Quan trọng hơn, cơ chế **Caching cục bộ** (Redux + Electron-store) cho phép người dùng mở app và thấy dữ liệu giao diện/vật phẩm ngay lập tức từ bộ nhớ đệm, tối ưu tốc độ phản hồi trước khi hệ thống âm thầm đồng bộ dữ liệu mới nhất với Cloud qua API **POST /sync-all**.

**Phía Backend (AWS Serverless):** Toàn bộ business logic (Gacha, Shop, Quest, Minigame, Social...) được triển khai dưới dạng các hàm AWS Lambda độc lập, kích hoạt qua HTTP API, S3 Event, DynamoDB Streams hoặc EventBridge Schedule. Lý do chọn Serverless thay vì EC2/ECS truyền thống:

* **Scale-to-Zero:** Khi không có người dùng, hệ thống gần như không phát sinh chi phí duy trì cố định — phù hợp giai đoạn phát triển và demo workshop.
* **Tự động mở rộng:** Lambda và API Gateway tự scale theo tải mà không cần cấu hình Auto Scaling Group hay Load Balancer thủ công.
* **Infrastructure as Code:** Toàn bộ hạ tầng khai báo trong **serverless.yml**, deploy một lệnh **serverless deploy** → CloudFormation tạo đồng bộ Lambda, API, IAM, Logs.
* **Tích hợp sẵn:** Cognito, DynamoDB, S3, EventBridge, Streams — giảm thời gian dựng nền tảng so với tự cài đặt trên máy chủ riêng.

#### Hạn chế và rào cản của kiến trúc

Bên cạnh các ưu điểm, mô hình này cũng đặt ra nhiều thách thức kỹ thuật cần được xử lý có chủ đích:

* **Cold Start Lambda:** Request đầu tiên sau một khoảng thời gian rảnh có thể chậm hơn do Lambda phải khởi tạo runtime. Dự án giảm thiểu bằng cách giữ memory ở mức 512 MB (sweet spot giữa tốc độ và chi phí).
* **Debug phân tán:** Logic nằm rải rác trên các hàm Lambda (sơ đồ ghi nhận **36 fn**) cùng các trigger bất đồng bộ (Streams, S3, Cron) — theo dõi end-to-end đòi hỏi CloudWatch Logs được cấu hình đúng.
* **Thiết kế DynamoDB từ đầu:** NoSQL không hỗ trợ JOIN hay full-text search phức tạp. Mọi truy vấn phải được thiết kế quanh PK/SK và GSI ngay từ đầu; tìm kiếm người dùng theo tên phải nhờ dịch vụ ngoài (Algolia).
* **Đồng bộ Cache và Race Condition:** Client lưu dữ liệu đệm local có thể dẫn tới xung đột nếu thao tác dồn dập. Backend buộc phải dùng **TransactWriteItems** cho giao dịch tiền tệ/inventory và áp dụng nguyên tắc Server Authority — không tin số liệu Client gửi lên.
* **Vendor Lock-in:** Kiến trúc gắn chặt với AWS (DynamoDB Streams, Cognito JWT Authorizer, IAM Role). Di chuyển sang cloud khác sẽ tốn chi phí refactor đáng kể.
* **Electron nặng tài nguyên:** Ứng dụng Desktop tiêu thụ RAM nhiều hơn web thuần; quá trình phân phối và cập nhật phiên bản phức tạp hơn deploy web.

#### Sơ đồ kiến trúc tổng thể

Sơ đồ dưới đây mô tả luồng giao tiếp chính giữa Client, các dịch vụ AWS và hệ thống bên thứ ba. Trong khung **AWS Cloud — ap-southeast-1** có bốn nhóm: AUTH & SECURITY, API & COMPUTE, DATA, ASYNC; **CloudFront** phân phối ảnh từ S3. Ngoài khung chính còn có **ACCOUNT B — BEDROCK** (cross-account qua STS) và khối **EXTERNAL** (Algolia, Gemini).

![Sơ đồ kiến trúc AWS](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)

Luồng chính trên sơ đồ (đánh số 1–10):
1. **Login:** Electron App đăng nhập qua **Cognito User + IdP**, nhận JWT.
2. **API (JWT):** Client gọi **HTTP API + JWT**; API Gateway verify token trước khi cho vào Backend.
3. **Invoke:** HTTP API invoke **Lambda (36 fn)** để xử lý business logic.
4. **R/W:** Lambda đọc/ghi **DynamoDB + Streams**.
5. **Assets:** Lambda đọc/ghi **S3 Assets** (upload, avatar, public assets…).
6. **Index:** Thay đổi trên DynamoDB (Streams) được đẩy sang **Algolia** để đồng bộ search index (chi tiết triển khai qua Lambda **streamIndexer**).
7. **Cron:** **EventBridge (cron ×2)** kích hoạt worker Lambda định kỳ (Shop / Leaderboard); CloudWatch nằm cùng nhóm ASYNC để giám sát.
8. **AI Bedrock:** Client gọi **Amazon Bedrock** ở **ACCOUNT B** bằng cơ chế **STS từ app** (cross-account), không đi qua API Gateway của Account chính.
9. **Gemini:** Client gọi trực tiếp **Google Gemini API** (khối EXTERNAL) khi người dùng cấu hình API Key.
10. **CDN ảnh:** Client tải ảnh qua **CloudFront (image CDN)**; CloudFront lấy origin từ **S3 Assets**.

> **IAM Exec Role** (nét đứt trên sơ đồ, không đánh số): cấp quyền runtime cho Lambda truy cập DynamoDB, Streams và S3 — đây là lớp phân quyền, không phải luồng dữ liệu.

File Draw.io gốc: **docs/Uchimi StudyGamification.drawio** (tab *Uchimi Current Architecture*)

#### Các dịch vụ AWS cốt lõi

Dự án sử dụng Serverless Framework (Node.js 20.x) để triển khai tự động cấu hình hạ tầng. Theo sơ đồ kiến trúc, lớp compute gồm **Lambda (36 fn)**; kèm theo **25 HTTP routes**, **2 EventBridge schedules** (cron ×2), **3 S3 triggers**, **1 DynamoDB Stream trigger** và **1 Cognito PostConfirmation trigger**.

| Dịch vụ | Mục đích sử dụng | Chi tiết cấu hình |
| :--- | :--- | :--- |
| **AWS Lambda** | Xử lý toàn bộ Business Logic | Nhóm API & COMPUTE trên sơ đồ: **36 fn**, tổ chức theo 10 module (User, Session, Gacha, Minigame, Quest, Shop, Social, Upload, Currency, Sync). |
| **API Gateway** | Cổng giao tiếp HTTP API + JWT cho Client | 25 routes (GET/POST/PUT), JWT Authorizer từ Cognito, Throttling 50 req/s và 20 concurrent. |
| **Amazon Cognito** | Xác thực và định danh người dùng (AUTH & SECURITY) | User Pool + IdP + JWT; Trigger PostConfirmation tự khởi tạo profile (**handleInitUser**). |
| **Amazon DynamoDB** | Cơ sở dữ liệu NoSQL (DATA) | 8 bảng: User, Study, Social, Quest, Minigame, ItemData, Inventory, GachaHistory; kèm Streams. |
| **DynamoDB Streams** | Bắt sự kiện thay đổi dữ liệu realtime | Kích hoạt Lambda **streamIndexer** trên User table → đồng bộ Algolia (bước 6. Index). |
| **Amazon S3** | Lưu trữ tài nguyên tĩnh (S3 Assets) | Bucket **ASSETS_BUCKET**: **public-assets/**, **avatars/**, **uploads/**; trigger Lambda khi upload file **.zip** / **.json**. |
| **Amazon CloudFront** | CDN phân phối ảnh/assets | Origin trỏ về S3; Client xem avatar/item qua URL CDN (bước 10 trên sơ đồ). |
| **Amazon EventBridge** | Lập lịch tác vụ tự động (ASYNC — cron ×2) | Cập nhật Leaderboard mỗi 10 phút; làm mới Shop hàng tuần (Chủ Nhật 17:00 UTC). |
| **Amazon CloudWatch** | Giám sát và Logs (ASYNC) | Tự động tạo Log Group cho mỗi Lambda; hỗ trợ debug và theo dõi hiệu năng. |
| **Amazon Bedrock** | AI model qua Account phụ | Nằm trong **ACCOUNT B — BEDROCK**; Client assume role bằng **STS từ app** (bước 8), không đi qua Lambda Account chính. |
| **AWS IAM** | Quản lý quyền truy cập | Execution Role gắn vào Lambda (nét đứt trên sơ đồ); cấp quyền least-privilege qua **serverless.yml**. |
| **AWS CloudFormation** | Provision hạ tầng (qua Serverless) | Mọi tài nguyên gom thành một Stack; rollback/xóa sạch bằng một thao tác. |

> **Ghi chú về Region/AZ:** Hệ thống không sử dụng VPC hay EC2, nên không cần cấu hình Availability Zone thủ công. AWS tự quản lý multi-AZ cho các dịch vụ managed (Lambda, DynamoDB, S3) bên trong region **ap-southeast-1**.

#### Tích hợp AI và dịch vụ bên thứ ba

Trên sơ đồ, các tích hợp ngoài luồng API chính gồm khối **EXTERNAL** (Algolia, Gemini) và **ACCOUNT B — BEDROCK**:

**Algolia (User Search Index — EXTERNAL):** DynamoDB không hỗ trợ full-text search theo tên người dùng. Algolia được chọn thay OpenSearch vì API đơn giản, kết quả gần realtime và không đòi hỏi vận hành cluster riêng. Luồng trên sơ đồ là bước **6. Index**: mỗi khi bảng User thay đổi, DynamoDB Streams kích hoạt Lambda **streamIndexer** → index document lên Algolia. Client gọi **GET /friends/search** → Lambda truy vấn Algolia và trả kết quả.

**Amazon Bedrock (ACCOUNT B — STS từ app):** Client gọi Bedrock trực tiếp ở Account phụ (bước **8. AI Bedrock**), dùng **STS** để nhận credential tạm thời — tách biệt với stack Serverless Account chính. Luồng này không đi qua HTTP API / Lambda của **ap-southeast-1**.

**Google Gemini API (EXTERNAL):** Client gọi trực tiếp Gemini (bước **9. Gemini**) để hỗ trợ sinh lộ trình học và quiz từ tài liệu upload. Hệ thống không ép buộc dùng Cloud AI — người dùng có thể chọn **Ollama (Local)** để xử lý trực tiếp trên máy cá nhân nhằm bảo vệ quyền riêng tư dữ liệu học tập cá nhân và không tốn phí API Key Cloud (chi tiết ở mục 5.3). Gemini chỉ được dùng khi người dùng tự cấu hình API Key, giúp ứng dụng không gánh chi phí LLM tập trung.

#### Luồng giao tiếp giữa các thành phần

Hệ thống vận hành theo bốn nhóm luồng, tương ứng mạch trên sơ đồ:

**Luồng đồng bộ (Request/Response — bước 1→5):** Client đăng nhập Cognito nhận JWT → gửi request kèm JWT tới HTTP API → API Gateway verify → invoke Lambda → Lambda đọc/ghi DynamoDB hoặc S3 → trả JSON về Client. Ví dụ: mua item Shop, claim quest, đổi cosmetic, sync profile.

**Luồng bất đồng bộ (Event-Driven — bước 6→7 và trigger phụ):**
* **DynamoDB Streams → streamIndexer → Algolia (6. Index):** Đồng bộ search index khi User table thay đổi.
* **EventBridge Cron → Shop/Leaderboard worker (7. Cron):** Tự động rotate shop hàng tuần, cập nhật bảng xếp hạng mỗi 10 phút; CloudWatch thu log/metric.
* **S3 ObjectCreated → processZip / processJson:** Dev upload file **.zip** item hoặc **.json** config → Lambda tự bung, đẩy assets lên S3 và ghi metadata vào ItemData table.
* **Cognito PostConfirmation → handleInitUser:** Tự khởi tạo profile, inventory mặc định khi user đăng ký lần đầu.

**Luồng AI từ Client (bước 8→9):** Electron App gọi thẳng **Bedrock (Account B, STS)** hoặc **Gemini (EXTERNAL)** — không đi qua API Gateway của stack chính. Người dùng cũng có thể chuyển sang Ollama local (mục 5.3).

**Luồng phân phối tài nguyên (CDN — bước 10):** Client không gọi API để xem ảnh avatar hay item asset. Thay vào đó, Client truy cập trực tiếp URL CloudFront → CloudFront lấy file gốc từ S3 Assets nếu chưa có trong cache. Lambda chỉ tham gia khi cần tạo presigned URL upload hoặc xử lý file mới.

#### Cấu hình bảo mật và Quyền truy cập (IAM Permissions)

Nguyên tắc **Least Privilege** được áp dụng nghiêm ngặt thông qua file **serverless.yml**. Lambda Execution Role chỉ được cấp đúng các Action trên đúng Resource cần thiết:

* **DynamoDB:** **GetItem**, **PutItem**, **UpdateItem**, **DeleteItem**, **Query**, **Scan**, **BatchGetItem**, **BatchWriteItem**, **TransactWriteItems** — giới hạn trên 8 bảng hệ thống và GSI tương ứng.
* **DynamoDB Streams:** **DescribeStream**, **GetRecords**, **GetShardIterator**, **ListStreams** — chỉ trên ARN của User Table Stream.
* **Amazon S3:** **GetObject**, **PutObject** — chỉ trong các prefix **public-assets/**, **avatars/**, **uploads/**.

Ở tầng API, mọi route nhạy cảm gắn authorizer **myCognitoAuth** (JWT). API Gateway kiểm tra token với Cognito User Pool trước khi cho phép invoke Lambda — Client không thể giả mạo **user_id**. CORS được bật trên HTTP API; Throttling giới hạn 50 requests/giây và 20 concurrent requests để chống spam/DDoS nhẹ.

Cần phân biệt hai lớp IAM khi vận hành:
* **Execution Role** (trong sơ đồ): quyền runtime của Lambda khi đang chạy — khai báo trong **provider.iam.role.statements**.
* **Deploy Role** (của developer): quyền tạo/cập nhật CloudFormation, Lambda, API Gateway — dùng khi chạy **serverless deploy**, không xuất hiện trên sơ đồ kiến trúc.

Biến môi trường nhạy cảm (**ALGOLIA_WRITE_KEY**, **COGNITO_USER_POOL_ID**, **COGNITO_CLIENT_ID**...) được load qua **useDotenv: true** từ file **.env** cục bộ — không commit lên GitHub.

#### Quy trình thiết lập và triển khai hạ tầng

Việc đưa Backend lên AWS được thực hiện theo các bước:

1. **Chuẩn bị tài nguyên nền:** Tạo Cognito User Pool, 8 bảng DynamoDB (bật Stream trên User table), S3 bucket và (tuỳ chọn) CloudFront distribution trước khi deploy Lambda.
2. **Cấu hình file .env:** Điền tên bảng, bucket, Cognito ID, Stream ARN, Algolia credentials vào file **AWSServerless/.env**.
3. **Khai báo IaC:** Toàn bộ Lambda, API routes, IAM, triggers được định nghĩa trong **serverless.yml** và các file **function.yml** con (user, shop, gacha, social, upload...).
4. **Deploy một lệnh:**

```bash
cd AWSServerless
serverless deploy
```

Serverless Framework tự biên dịch code (esbuild bundle + minify), tạo CloudFormation template và provision tài nguyên đồng bộ trên region **ap-southeast-1**.

5. **Kết nối Frontend:** Cập nhật **AWSStudy-Play/.env** trỏ **VITE_API_URL**, **VITE_COGNITO_\***, **VITE_S3_ASSETS_URL** về endpoint thực tế sau deploy.

#### Phân tách module Lambda và cơ chế kích hoạt

Backend được chia thành 10 module độc lập, mỗi module có file **function.yml** riêng khai báo handler và loại trigger:

| Module | Trigger | Chức năng chính |
| :--- | :--- | :--- |
| **userFunction** | Cognito PostConfirmation + HTTP API | Khởi tạo profile; cập nhật tên; trang bị cosmetic; upload avatar |
| **sessionFunction** | HTTP API | Quản lý phiên học / Focus session |
| **currencyFunction** | HTTP API | Quy đổi Knowledge Point → Knowledge Core |
| **gachaFunction** | HTTP API | Roll gacha server-side, pity, ghi Inventory + GachaHistory |
| **minigameFunction** | HTTP API + EventBridge Schedule | Sudoku/Minesweeper; worker leaderboard định kỳ |
| **questFunction** | HTTP API | Daily quest, claim reward, cập nhật streak |
| **shopFunction** | HTTP API + EventBridge Schedule (cron) | Lấy shop, mua item eCoin; rotate shop hàng tuần |
| **socialFunction** | HTTP API + DynamoDB Stream | Bạn bè, kết bạn; search Algolia; **streamIndexer** |
| **syncFunction** | HTTP API | **sync-all** / đồng bộ profile-inventory và làm mới cache cục bộ tại Client |
| **uploadFunction** | S3 ObjectCreated (file **.zip** / **.json**) | Ingest item master data + assets vào S3 và ItemData table |

Quy trình đẩy item mới vào hệ thống (ví dụ background Study Plant) cũng tuân theo luồng event-driven: Developer đóng gói item thành file ZIP (gồm **data.json** + assets) → upload lên **s3://ASSETS_BUCKET/uploads/items/** → S3 trigger Lambda **processZip** → Lambda bung ZIP, đẩy file lên **public-assets/items/** và ghi metadata vào bảng ItemData. Nếu item có **collectFrom: "eCoinShop"**, shop worker sẽ tự nhặt item đó khi rotate shop hàng tuần.
