---
title : "Học tập cùng AI"
date : 2026-07-17 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

Dự án tích hợp AI vào hai chức năng cốt lõi: **AI Study Planner** (lập kế hoạch và ôn tập thông minh) và **AIGuard** (hệ thống giám sát kỷ luật học tập thời gian thực). Cả hai tạo thành một vòng lặp khép kín: AI giúp người dùng học *hiệu quả hơn*, AIGuard đảm bảo người dùng *thực sự học*, còn hệ thống nhiệm vụ (Quest) và bảng xếp hạng (Rank) biến kỷ luật đó thành **phần thưởng có giá trị thực**.

---

## 1. Tổng quan (Overview)

![Giao diện chính của Study Play](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_214455_qtkwxn.png)
*Hình 1: Giao diện chính của Study Play (Dashboard)*

### AI Study Planner
AI Study Planner là trợ lý học tập cá nhân toàn diện được tích hợp ngay trong ứng dụng. Người dùng có thể **upload tài liệu học tập** (PDF, DOCX, TXT), sau đó hội thoại tự nhiên với AI. AI đóng vai trò "cố vấn học thuật", chủ động thu thập thông tin về mục tiêu, khối lượng kiến thức và deadline, rồi tự động thiết kế một **lộ trình học tập chi tiết** (Study Plan) với từng Topic và thời lượng cụ thể. Từ lộ trình đó, AI sinh **bộ câu hỏi trắc nghiệm động (Quiz)** theo từng chủ đề để luyện Active Recall. Khi nộp bài, mỗi câu đúng cộng **Knowledge Points (KP)** và cập nhật tiến độ nhiệm vụ ngay lập tức trên server.

### AIGuard — Hệ thống Giám sát Kỷ luật
AIGuard là hệ thống phân tán hoạt động đồng thời trên Desktop App (Electron Backend) và Browser Extension (Chrome, Edge,...). Hệ thống **không chặn cứng video YouTube hay website**, mà dùng kiến trúc phân loại **3 tầng** để đánh giá ngữ cảnh từng nội dung. Ngoài ra, hệ thống còn tích hợp **AI App Blocker** để phát hiện và xử lý các ứng dụng giải trí đang chạy trên Windows (game, streaming, v.v.) thông qua cửa sổ cảnh báo đếm ngược 10 giây. AIGuard cũng tích hợp **Face Tracking với Liveness Detection** để chống AFK và chống gian lận bằng ảnh. Mọi vi phạm (Strike) được ghi nhận và xử lý hoàn toàn **server-side** qua AWS Lambda → DynamoDB, đảm bảo không thể giả mạo kết quả. Sau khi phiên học kết thúc, server tính điểm Rank và cập nhật tiến độ Quest — dữ liệu đẩy về client để đồng bộ Redux Store.

---

## 2. Cấu hình AI (AI Settings)

![Cấu hình AI Provider](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_214522_b8tndd.png)
*Hình 2: Người dùng có thể linh hoạt chọn các mô hình AI khác nhau cho từng chức năng*

Người dùng cấu hình AI provider cho từng chức năng trong tab **AI Settings**. Mỗi chức năng (**AI Blocker** cho AIGuard và **AI Study Planner**) được cấu hình riêng biệt với cùng 3 lựa chọn provider:

| Provider | Cơ chế | Cấu hình |
|---|---|---|
| **Ollama (Local)** | Chạy model AI trực tiếp trên máy tính người dùng | Khi mở tab AI Settings, ứng dụng tự động query **http://localhost:11434/api/tags** để quét danh sách model đang chạy, người dùng chọn từ dropdown |
| **Google Gemini (Cloud)** | Gọi API Gemini qua HTTPS | Người dùng nhập Gemini API Key, lưu trong local store, không gửi lên server |
| **AWS Bedrock (Cloud)** | Ký AWS Signature V4 + STS AssumeRole cross-account | Sử dụng Cognito credentials đã đăng nhập — không cần cấu hình thêm, chỉ cần toggle ON |

Khi bấm "Lưu", settings được lưu đồng thời vào **Redux Store** (state toàn cục) và local storage qua IPC **store:saveAiSettings**, đồng thời đồng bộ riêng cho StudyPlanner qua IPC **study:saveSettings**. Hệ thống kiểm tra model Ollama đang chọn có nằm trong danh sách **WEAK_MODELS** không — gồm các model quá nhỏ (qwen2.5:1.5b, tinyllama, phi3:mini, gemma:2b, orca-mini) sẽ bị cảnh báo vì chúng không đủ khả năng sinh JSON có cấu trúc đáng tin cậy.

---

## 3. Chi tiết luồng kỹ thuật: AI Study Planner

### a. Luồng Upload & Phân tích Tài liệu (fileParser.js + documentContext.js)

![Giao diện AI Chat thu thập thông tin và phân tích tài liệu](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_215349_cmycre.png)
*Hình 3: Trợ lý AI thu thập thông tin mục tiêu học tập và đọc hiểu tài liệu người dùng đính kèm*

**Bước 1 — Giao tiếp IPC:**
Người dùng bấm đính kèm (📎) trong **ChatTab**. Hộp thoại chọn file mở qua IPC **dialog:openFile**. Đường dẫn file được gửi qua IPC **study:uploadFile** xuống Electron Backend.

**Bước 2 — Đọc file (fileParser.js):**
- Kiểm tra kích thước: từ chối file > **25MB** để tránh tràn bộ nhớ.
- Phát hiện loại file bằng extension (.pdf, .docx, .doc, .txt).
- **Lazy loading thư viện:** **pdf-parse** và **mammoth** (DOCX parser) được import động (**await import(...)**) thay vì load khi khởi động — chỉ tải về khi thực sự cần dùng, tiết kiệm bộ nhớ startup.
- **PDF:** Dùng **pdf-parse** đọc buffer, lấy **data.text** và **data.numpages**.
- **DOCX/DOC:** Dùng **mammoth.extractRawText()** lấy text thuần.
- **TXT:** Đọc trực tiếp bằng **fs.readFileSync(path, 'utf-8')**.
- Kiểm tra sau khi đọc: nếu text < 50 ký tự → báo lỗi file trống/hỏng.

**Bước 3 — Làm sạch text (cleanText):**
Chuỗi văn bản thô được xử lý qua hàm **cleanText**:
- Chuẩn hóa line endings (\r\n → \n).
- Loại bỏ ký tự điều khiển (control characters) ngoại trừ \n và \t.
- Thu gọn nhiều dòng trống liên tiếp thành tối đa 2 dòng.
- Thu gọn nhiều khoảng trắng thành 1.

**Bước 4 — Phân đoạn thứ bậc (Hierarchical Chunking — buildChunks):**
Đây là tối ưu quan trọng nhất của luồng xử lý tài liệu. Thay vì cắt văn bản theo kích thước cố định, hệ thống phát hiện **các tiêu đề (headings)** để tạo ra các chunk có ngữ nghĩa:

*Nhận diện heading bằng các mẫu (HEADING_PATTERNS):*
- Dòng bắt đầu bằng "Chương/Chapter/Phần/Part/Bài/Lesson/Mục/Section + số".
- Dòng bắt đầu bằng số + dấu chấm + chữ hoa (1. Introduction).
- Số La Mã (IV. ...).
- Markdown heading (#, ##, ###, ####).
- Dòng VIẾT HOA TOÀN BỘ, ngắn hơn 80 ký tự.

*Cơ chế tạo chunk:*
- Mỗi khi gặp heading mới → flush chunk hiện tại, bắt đầu chunk mới với section name mới.
- Target: **250 từ/chunk**. Maximum: **400 từ/chunk**.
- Khi vượt target, ưu tiên cắt tại **dòng trống** để không cắt giữa đoạn văn.
- Chunk quá ngắn (< 30 ký tự) bị loại bỏ.
- Mỗi chunk ghi nhận: **id**, **section** (tên tiêu đề), **content**, **wordCount**.

**Bước 5 — Tóm tắt AI (summarizeDocument):**
Sau khi có chunks, Backend gọi **summarizeDocument**:
- **Gemini/Bedrock:** Gộp tất cả chunks thành **docText**, gửi toàn bộ cho AI tóm tắt 200-300 từ và liệt kê chủ đề chính.
- **Ollama (Local):** Do giới hạn context window, text được **cắt cứng tối đa 2500 ký tự** trước khi gửi. Tên tất cả sections được liệt kê riêng để AI vẫn hiểu được cấu trúc tổng thể. Temperature **0.1** (rất thấp) để kết quả ổn định, output buộc là JSON.
- Kết quả lưu: **{ summary, topics[] }**.

**Bước 6 — Lưu vào Session (documentContext.js — Singleton):**
Toàn bộ document được lưu vào **singleton in-memory** **_document**:
```
{ fileName, fileType, chunks[], summary, topics[], charCount, wordCount, pageCount, uploadedAt }
```
- Singleton sống trong suốt phiên Desktop App — không mất khi chuyển tab.
- Xóa khi người dùng bấm X hoặc upload file mới.
- Frontend hiển thị **Document Banner** (tên file + số trang) xác nhận đã sẵn sàng.

### b. Tối ưu Context Injection — 3 chiến lược khác nhau (documentContext.js)

Khi tài liệu đang active, AI không nhận toàn bộ text thô mà nhận **context được chọn lọc phù hợp với từng loại tác vụ**:

**Tầng 1 — Context cho Chat (getContextForQuery):**
- Luôn inject **summaryBlock**: tóm tắt tổng quan + danh sách chủ đề.
- Nếu câu hỏi người dùng > 3 ký tự: gọi **searchChunks(chunks, query, 3)** để tìm **tối đa 3 chunks liên quan nhất** và inject thêm **chunksBlock**.
- Thiết kế này giảm đáng kể token gửi đi so với inject toàn bộ tài liệu, đặc biệt hữu ích với Bedrock (tính phí theo token).
- Với Ollama: chỉ inject **summaryBlock** ngắn gọn (cắt còn 600 ký tự) do giới hạn context window.

**Tầng 2 — Context cho Plan Generation (getContextForPlan):**
- Thay vì tìm kiếm, lấy **8 chunks phân bổ đều** từ đầu đến cuối tài liệu (step = floor(chunks.length / 8)).
- Trả về **structureBlock** (danh sách section titles) để AI tạo plan sát với cấu trúc thực của tài liệu.
- Prompt yêu cầu AI tạo phases "closely follow this Document Structure".

**Tầng 3 — Context cho Quiz (getContextForQuiz):**
- Gọi **searchChunks(chunks, phaseName + topics, 5)** để lấy **5 chunks liên quan nhất** đến phase cần tạo quiz.
- Prompt yêu cầu AI sinh câu hỏi "strictly based 100% on this document" — tránh AI bịa ra kiến thức ngoài tài liệu.

### c. Tìm kiếm Chunk thông minh (BM25-style — searchChunks trong fileParser.js)

**searchChunks** là thuật toán tìm kiếm ngữ nghĩa nhẹ, lấy cảm hứng từ BM25:
1. Tách query thành keywords (normalize unicode, loại bỏ dấu tiếng Việt, loại từ < 3 ký tự).
2. Với mỗi chunk, tính **score**:
   - Đếm số lần keyword xuất hiện trong **content + section** (tần suất).
   - **Bonus +3 điểm** nếu keyword xuất hiện trong **section** (tên heading) — ưu tiên chunk đúng chủ đề.
3. Sắp xếp theo score giảm dần, trả về **topK** chunk cao điểm nhất.
4. Chunk score = 0 bị loại bỏ hoàn toàn.

### d. Luồng Chat Thu thập Thông tin (aiStudyService.js)

1. Người dùng gửi tin nhắn → Frontend đóng gói lịch sử (**messages[]**) và gửi qua IPC **study:chat**.
2. Backend gọi **chatWithAI(messages)**, load AI settings, lấy context từ **getContextForQuery(lastUserMsg)**.
3. **System Prompt** hướng dẫn AI đóng vai "Cố vấn học thuật", cần thu thập đủ 6 trường: **subject**, **topic**, **level**, **goal**, **totalDuration**, **dailyHours**. Khi đủ ít nhất trường 1, 2, 3 → set **readyToGenerate = true**.
4. AI hỗ trợ **tất cả ngôn ngữ** (tiếng Việt, Anh, Trung, Nhật, Hàn,...), tự động reply bằng ngôn ngữ người dùng đang dùng.
5. LLM buộc output **JSON thuần** (không có markdown): **{ reply, collectedInfo, readyToGenerate }**.
   - Ollama: bật **format: 'json'** để đảm bảo output đúng format.
   - Timeout: 120 giây cho cloud, **300 giây cho Ollama** (local model cần thêm thời gian load).
6. **parseResponse()** dùng regex **/{[\s\S]*}/ ** để trích JSON từ response — an toàn với cả output có markdown code block thừa.
7. Frontend tích lũy **collectedInfo** qua các lượt. Khi **readyToGenerate = true** → nút "Tạo Lộ trình" hiện ra.
8. Phiên chat tự động lưu local (tiêu đề = 40 ký tự đầu tin nhắn đầu tiên).

### e. Luồng Sinh Lộ trình (generateStudyPlan)

![Lộ trình học tập chi tiết](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_215432_my4exv.png)
*Hình 4: Lộ trình học tập chi tiết do AI phân tích và thiết kế sát với nội dung tài liệu*

1. Frontend gọi IPC **study:generatePlan** kèm **collectedInfo**.
2. Backend gọi **getContextForPlan()** lấy 8 chunks phân bổ đều.
3. **PLAN_SYSTEM_PROMPT** buộc LLM phải:
   - Output **JSON thuần**, không giải thích.
   - Viết **cùng ngôn ngữ với input** của người dùng (nếu input tiếng Việt → toàn bộ plan tiếng Việt).
   - Schema: **{ title, description, phases[{ id, name, duration, description, topics[], resources[{ name, url, type }], completed }] }**.
   - 4-5 phases, 2-3 topics/phase, 1-2 resources/phase (website links).
4. Temperature **0.6** — thấp hơn chat để kết quả nhất quán hơn. Ollama timeout **600 giây**.
5. Frontend nhận JSON, lưu vào Redux qua **saveStudyPlan**, tự chuyển sang PlanTab.

### f. Luồng Sinh Quiz & Tính điểm Server (generateQuiz + questService.mjs)

![Giao diện bài kiểm tra Active Recall](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646924/Screenshot_2026-07-21_215715_z2xzhx.png)
*Hình 5: Bài kiểm tra trắc nghiệm giúp người dùng ôn tập kiến thức chủ động (Active Recall)*

**Client side:**
1. Người dùng bấm "Tạo câu hỏi" ở 1 Phase bất kì trong Topic.
2. Backend gọi **getContextForQuiz(phaseName, topics)** — lấy 5 chunks liên quan.
3. **QUIZ_SYSTEM_PROMPT**: buộc AI sinh đúng **10 câu trắc nghiệm**, 4 lựa chọn A/B/C/D, cùng ngôn ngữ với phase name.
4. Schema: **{ questions[{ id, question, options[], correctAnswer }] }**. Temperature **0.7**.

**Server side (POST /study-planner/quiz-submit → questService.mjs):**
- Nhận **{ correctAnswersCount, totalQuestions }**.
- Tính **reward = correctAnswersCount × 10** KP.
- Dùng **ADD budget.knowledgePoint :kp** (DynamoDB atomic operation) — không thể giả mạo điểm.
- Gọi **updateQuestProgress(userId, "COMPLETE_QUIZ", 1)** — nhiệm vụ hoàn thành 1 bài quiz.
- Gọi **updateQuestProgress(userId, "CORRECT_QUIZ_ANSWER", correctAnswersCount)** — nhiệm vụ trả lời đúng.
- Trả về **{ earnedKP, questUpdate, profile, daily }** → Frontend đồng bộ Redux.

---

## 4. Chi tiết luồng kỹ thuật: AIGuard & Focus System

### a. Khởi động & Điều kiện bắt đầu phiên

![Bảng điều khiển thiết lập phiên tập trung](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_215747_zhrocq.png)
*Hình 6: Bảng điều khiển để bắt đầu phiên học tập trung*

1. FocusGuard component mount → gửi IPC **focus:setConfig** kèm JWT token + API URL xuống Backend. Backend lưu token cho các lần gọi API server.
2. **3 điều kiện bắt buộc** trước khi "Bắt đầu":
   - Nếu đang bật Browser thì Browser phải có Extension.
   - Extension đang kết nối WebSocket.
   - Ít nhất một AI provider đang **ready** (Bedrock / Gemini / Ollama).
3. Khi bấm "Bắt đầu" → IPC **focus:start** → Electron gọi **POST /start-study-session** lên AWS Lambda với **{ mode, durationMinutes }**. Lambda tạo session trong DynamoDB STUDY_TABLE (status: PENDING, strikeCount: 0, TTL 6 tháng), trả về **sessionId**. Nếu API fail → dùng **local_${Date.now()}** làm fallback ID.

### b. Kiến trúc WebSocket Desktop — Browser (focusEngine.js)

- **WebSocket Server:** Electron tạo **WebSocketServer** trên **ws://localhost:8765** khi khởi động ứng dụng.
- **PING/PONG:** Server gửi **PING** tới tất cả clients mỗi **10 giây** để giữ kết nối sống.
- **Extension Identification:** Khi Extension kết nối, gửi **EXTENSION_CONNECTED** kèm **browser** (Chrome, Edge, Opera, Brave, Firefox, CốcCốc). Backend lưu vào **connectedBrowsers Map<ws, browserName>**.
- **Disconnect Grace Period:** Khi Extension mất kết nối, Backend **chờ 15 giây** trước khi coi là mất kết nối thật sự (tránh false alarm khi trình duyệt refresh/reload). Trong thời gian chờ, browser vẫn được tính là "có extension".
- **Settings Sync:** Khi Extension kết nối, server gửi ngay **SETTINGS_UPDATED** kèm **allowedCategories** hiện tại.

### c. Extension Gate Check (checkGateStatus — mỗi 3 giây)

![Cảnh báo Extension Gate](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221237_dd7yme.png)
*Hình 7: Cảnh báo tự động hiện lên khi người dùng mở trình duyệt nhưng chưa bật Extension*

Đây là cơ chế bảo vệ tránh người dùng mở trình duyệt không có extension và lướt web tự do:

1. Chạy lệnh **tasklist /NH /FO CSV** để lấy danh sách process.
2. Nếu thấy browser (chrome.exe, msedge.exe, opera.exe, brave.exe...) → chạy thêm **PowerShell** với query:
   ```
   Get-Process | Where-Object { $_.MainWindowHandle -ne 0 -and $_.ProcessName -match '^(chrome|msedge|opera)$' }
   ```
   **Tối ưu:** Lọc `MainWindowHandle != 0` để loại bỏ **"ghost processes"** (Edge Startup Boost, background processes không có cửa sổ) — tránh false positive.
3. So sánh danh sách browser đang chạy với **connectedBrowsers**. Browser nào chạy mà không có extension bị đưa vào **currentlyMissing**.
4. **Grace period 10 giây:** Browser mới mở cần thời gian để extension boot và kết nối. Chỉ sau 10 giây mà vẫn không có extension mới tính là **definitelyMissing**.
5. Nếu phát hiện browser không có extension **trong khi timer đang chạy** → tự động **stopTimerForcefully()**.

### d. AI-powered App Blocker — Phân loại & Chặn ứng dụng Windows (appBlocker.js + focusEngine.js)

![Cửa sổ cảnh báo và đếm ngược 10 giây của App Blocker](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646925/Screenshot_2026-07-21_215830_mtsjj5.png)
*Hình 8: Overlay cảnh báo đếm ngược 10 giây khi AI phát hiện ứng dụng giải trí đang chạy*

Đây là chức năng giám sát ứng dụng chạy trực tiếp trên Windows. Khác với việc chặn cứng theo danh sách, hệ thống dùng AI để phân loại mọi ứng dụng đang chạy có cửa sổ.

**Bước 1 — Quét process foreground (mỗi 3 giây — focusEngine.js):**
Thay vì dùng **tasklist** (trả về mọi process kể cả background service, driver), Backend chạy lệnh PowerShell:
```powershell
Get-Process | Where-Object { $_.MainWindowHandle -ne 0 } | Select-Object ProcessName
```
Lọc `MainWindowHandle != 0` đảm bảo chỉ lấy các ứng dụng đang có cửa sổ hiển thị thực sự — loại bỏ hoàn toàn background services, drivers, và các "ghost processes". Ngoài ra, danh sách **SYSTEM_PROCESSES** (chrome, msedge, electron, explorer,...) được lọc tiếp để tránh xử lý trùng lặp với Extension Gate.

**Bước 2 — In-flight deduplication:**
Trước khi gửi lên AI, hệ thống kiểm tra **inFlightChecks** (một Set): nếu process đã đang được AI phân loại thì bỏ qua, không tạo request trùng lặp.

**Bước 3 — Lấy Windows metadata (PowerShell VersionInfo):**
Với mỗi process mới, **getWindowsAppMetadata()** chạy PowerShell để đọc **VersionInfo** từ file **.exe**:
```powershell
$vi = (Get-Item $path).VersionInfo
$vi | Select-Object FileDescription, ProductName, CompanyName, OriginalFilename | ConvertTo-Json
```
Kết quả gồm: **fileDescription**, **productName**, **companyName**, **fileName** — cung cấp ngữ cảnh phong phú hơn nhiều so với chỉ dùng tên process.

**Bước 4 — Phân loại AI (classifyApp — Bedrock → Gemini → Ollama):**
Metadata được gửi lên AI theo thứ tự ưu tiên provider: **Bedrock → Gemini → Ollama**. Nếu provider được chọn thất bại, hệ thống tự động thử provider tiếp theo.

*APP_SYSTEM_PROMPT* có hai luật cốt lõi:
- **ALLOW:** Công cụ năng suất, code editor, phần mềm phát triển, office, communication tools (chat/email), tiện ích hệ thống, và mọi phần mềm liên quan đến công việc.
- **BLOCK:** Video game, game launcher, ứng dụng streaming âm nhạc/phim (Spotify, VLC, Netflix,...), nền tảng giải trí, và bất kỳ ứng dụng nào có mục đích chính là giải trí.

Temperature **0.1** — cực thấp để đảm bảo kết quả nhất quán, không sáng tạo.

**Bước 5 — Cache 24h:**
Kết quả phân loại được lưu vào **appCache** (Map với key là processName lowercase). Lần tiếp theo gặp cùng process → trả về ngay từ cache mà không gọi AI lại, tiết kiệm đáng kể thời gian và chi phí token.

**Bước 6 — Overlay 10 giây (Floating Electron Window):**
Khi AI phán quyết **BLOCK**, Backend tạo một cửa sổ Electron nhỏ (**BrowserWindow**) hiển thị ngay phía trên thanh taskbar với thông tin:
- Tên ứng dụng bị chặn và processName.
- Lý do chặn từ AI (AI Verdict).
- Thanh đếm ngược 10 giây.
- Nút **"Đóng ngay ứng dụng này"** — người dùng có thể chủ động đóng để thoát vi phạm.

Cửa sổ overlay được tạo với `alwaysOnTop: true`, `focusable: false` (không tranh chấp tiêu điểm - focus - của ứng dụng đang bị chặn), `transparent: true` và hiển thị thông minh: nếu mini widget Focus Mode đang hiển thị, overlay tự động dịch chuyển lên phía trên để tránh đè nhau.

**Bước 7 — Xử lý kết quả 10 giây:**
Trong suốt 10 giây, Backend kiểm tra mỗi 1 giây xem app có còn chạy không:
- **Người dùng tự đóng app (trong 10s):** Overlay biến mất, **không tính Strike**.
- **Người dùng bấm nút "Đóng ngay":** App bị **taskkill /F /IM**, **không tính Strike**.
- **Hết 10 giây, app vẫn chạy:** Backend tự động **taskkill /F /IM** và gọi **recordAppStrike()**.

**Bước 8 — Ghi nhận Strike:**
**recordAppStrike()** gọi cùng API **POST /strike** như Web/Video blocker — gửi **sessionId** lên AWS Lambda, server tăng **strikeCount** trong DynamoDB. Nếu đủ 3 Strikes → **endSessionFail()**.

### e. AI Health Check liên tục (mỗi 5 giây — focusEngine.js)

Trong suốt thời gian timer active, một background interval mỗi **5 giây** gọi **getAiStatus()**:
- Kiểm tra song song trạng thái của **tất cả providers** (Bedrock, Gemini, Ollama, Groq) bằng **Promise.all()**.
- Xác định **activeProvider** theo thứ tự ưu tiên: Bedrock → Gemini → Ollama → Groq.
- Nếu không còn provider nào **ready** → **tự động dừng phiên** (**stopTimerForcefully**) và gửi sự kiện **ai-status-lost** về Frontend.
- Đảm bảo phiên học luôn có AI sẵn sàng để phân loại nội dung.

### f. Hệ thống Phân loại YouTube 3 Tầng (aiGuard.js)

![Video YouTube Bị Chặn](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646925/Screenshot_2026-07-21_221420_hauneg.png)
*Hình 9: Tính năng chặn video giải trí trên YouTube bằng AI Guard trên trình duyệt*

**Bước 0 — Trích xuất siêu dữ liệu (extractor.js — MAIN world):**
- **extractor.js** chạy trong MAIN world (quyền truy cập trực tiếp vào JS của trang YouTube).
- Lắng nghe event **__fg_extract__** từ **content.js**. Khi kích hoạt, truy cập **player.getPlayerResponse()** (ưu tiên) hoặc **window.ytInitialPlayerResponse** (fallback) để lấy: **videoId**, **title**, **author**, **channelId**, **category**, **keywords**, **description** (500 ký tự đầu).
- Chỉ gửi nếu **videoId** trùng với URL hiện tại (tránh stale data từ SPA navigation).
- Trên SPA navigation (**yt-navigate-finish**), retry ở 500ms / 1.5s / 3s / 5s.

**Tầng 1 — Phân loại tĩnh (Whitelist / Blacklist / Category):**
- Kiểm tra **channelId**/**author** theo **Channel Whitelist** (**channels.json**): Khan Academy, 3Blue1Brown, CrashCourse, Veritasium, TED, TED-Ed, MIT OCW, freeCodeCamp, Fireship, Kurzgesagt, Numberphile, SmarterEveryDay, Vsauce, MinutePhysics, Computerphile, The Organic Chemistry Tutor, Professor Leonard, Traversy Media, The Coding Train, Academind, Bro Code, Web Dev Simplified. → **ALLOW** ngay.
- Kiểm tra **Channel Blacklist** (tùy chỉnh người dùng). → **BLOCK** ngay.
- Kiểm tra **category** theo **allowedCategories** (mặc định: Education, Science & Technology): → **ALLOW**.
- Kiểm tra **BLOCKED_CATEGORIES** (cứng): Gaming, Music, Entertainment, Comedy, Film & Animation, Movies, Shows, Trailers, Anime & Animation. → **BLOCK**.
- Category không thuộc danh sách nào (Sports, People & Blogs, Howto,...) → **UNCERTAIN** → lên Tầng 2.

**Tầng 2 — Phân loại AI thời gian thực (aiGuard.js classifyContent):**

*Luồng gửi request:*
- **content.js** hiện overlay "🤖 AI đang phân tích..." → gửi **CLASSIFY_VIDEO** + metadata → **background.js** → WebSocket → focusEngine.js → gọi **classifyContent(metadata)**.
- **buildUserPrompt(metadata)**: ghép Title, Channel, Category, Keywords (tối đa 10), Description (tối đa 300 ký tự) thành prompt gọn.

*Thứ tự ưu tiên provider:*
1. **AWS Bedrock** (nếu provider = bedrock).
2. **Google Gemini** (nếu provider = gemini + có apiKey). Safety check: nếu selectedModel không bắt đầu bằng "gemini" (người dùng nhầm để model Ollama) → tự động fallback về **gemini-2.0-flash**.
3. **Ollama** (nếu provider = ollama + đang chạy + có model bắt đầu bằng "qwen3").
4. Nếu không có provider nào → mặc định **BLOCK**.

*Temperature 0.1* cho tất cả classification (kết quả nhất quán, ít sáng tạo).

*parseAiResponse():* Dùng regex trích JSON an toàn. Nếu regex fail → tìm từ khóa ALLOW/BLOCK trong text (fallback). Nếu không tìm thấy gì → mặc định **BLOCK** (an toàn nhất).

*Classification Cache (24h TTL):*
- Cache theo **videoId** (video), **domain** (webpage) bằng **Map<key, { result, timestamp }>**.
- Video/domain đã phân loại → trả về ngay, không gọi AI lần 2.
- Có thể xóa cache thủ công qua **clearCache()**.

*Kết quả gửi ngược:* focusEngine → WebSocket → **background.js** → **content.js** gỡ overlay, áp dụng ALLOW/BLOCK kèm **reason** và **provider** (biết AI nào đã phán xét).

**Tầng 3 — Đếm ngược 10 giây & Strike:**
- **BLOCK**: overlay chặn xuất hiện, video bị **pause + mute** mỗi 500ms (ngăn tua lại/bật lại).
- Đồng thời hiện **đếm ngược 10 giây** với progress bar.
- Rời trang trong 10 giây: không vi phạm.
- Ở lại > 10 giây: **STRIKE_REPORT** → WebSocket → **recordStrike()** → **POST /strike** → DynamoDB.

### g. Tối ưu Gemini API — Fallback Chain tự động (geminiApi.js)

Gemini API thường xuyên bị 503 (overloaded) hoặc 429 (rate limit). **geminiRequestWithFallback** giải quyết bằng cơ chế:

**Chain 5 models (theo thứ tự thử):**
```
[preferredModel] → gemini-flash-latest → gemini-2.0-flash → gemini-2.0-flash-lite → gemini-2.5-flash → gemini-2.5-pro
```
(Preferred model của người dùng được thử trước, không bao giờ skip)

**Chiến lược retry theo loại lỗi:**
| Lỗi | Hành động |
|---|---|
| **503 Overloaded** | Retry tối đa 2 lần, mỗi lần chờ **10 giây**. Nếu vẫn lỗi → chuyển model tiếp theo |
| **429 Rate Limit** | Parse thời gian chờ từ message (retry in Xs), chờ đúng thời gian đó + 1 giây. Tối đa 2 lần |
| **404 Not Found / Limit 0** | Retry 2 lần, chờ **5 giây**. Nếu vẫn lỗi → chuyển model tiếp theo |
| **Timeout (AbortController)** | Chuyển ngay sang model tiếp theo |
| **Tất cả models đều fail** | Chờ **15 giây** rồi restart toàn bộ chain. Tối đa 2 lần global retry |

### h. AWS Bedrock — Không cần SDK, Cross-account STS (bedrockApi.js)

Thay vì dùng **@aws-sdk/client-bedrock-runtime** nặng nề, hệ thống tự implement:
- **AWS Signature V4** từ scratch bằng Node.js **crypto** module — ký HMAC-SHA256 cho mọi request.
- **STS AssumeRole Cross-Account:** Dùng Cognito credentials (Account A) để gọi **sts.amazonaws.com/AssumeRole** lấy temporary credentials cho role **CrossAccountBedrockRole** trong Account B (chứa Bedrock). Credentials được cache và tự động refresh khi hết hạn.
- **Token logging:** Mỗi lần gọi Bedrock thành công, log **In=X, Out=Y tokens** để theo dõi chi phí.

### i. Hard-block Web và Phân loại Trang web bất kỳ

![Trang web giải trí bị chặn](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221356_rlu6ah.png)
*Hình 10: Màn hình ngăn chặn khi truy cập vào trang web giải trí trong giờ học*

**Hard-blocked Web:**
- Danh sách cứng: **facebook.com**, **twitter.com/x.com**, **tiktok.com**, **threads.net**, **instagram.com**.
- Phát hiện → overlay ngay lập tức (không qua AI). Đếm ngược 10s + Strike.
- Kiểm tra lại mỗi 3 giây để ngăn bypass overlay.

**Trang web bất kỳ (classifyWebPage):**
1. Kiểm tra **Safe Whitelist** (hardcode): Google, GitHub, Stack Overflow, Notion, ChatGPT, Claude, MDN, LeetCode, Coursera, Udemy, Figma, Slack, Zoom,... → Bỏ qua.
2. Kiểm tra **Domain Cache** (24h TTL, theo domain).
3. Domain mới: hiện badge nhỏ "🛡️ AI đang phân tích...", sau 2 giây trích metadata (**url**, **domain**, **title**, **description**, **og:title**, **og:description**, **keywords**, **h1**) → gửi **CLASSIFY_PAGE** qua WebSocket.
4. **WEB_SYSTEM_PROMPT** khắt khe hơn YOUTUBE_SYSTEM_PROMPT: Chỉ ALLOW nếu rõ ràng là giáo dục/công việc. **Đặc biệt block fiction/novel/manga dù tên miền gọi là "đọc sách" hay "văn học"**. Mặc định BLOCK nếu không chắc chắn.
5. Thứ tự provider giống YouTube. Cache theo domain.

### j. Nhận diện khuôn mặt & Liveness Detection (faceTracker.js)

![Hệ thống Face Tracking giám sát người dùng học tập](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215846_z1iind.png)
*Hình 11: Camera liên tục theo dõi vị trí khuôn mặt để tính toán chống treo máy (AFK) và giả mạo (Liveness)*

**Khởi tạo:**
- Load **MediaPipe BlazeFace Short Range (TFLite, float16)** qua CDN jsDelivr.
- Ưu tiên **GPU** (delegate: GPU). Nếu fail → auto fallback **CPU**.
- Webcam mở ở **320×240** để tiết kiệm tài nguyên.
- Mô hình chạy **100% local** — không gửi hình ảnh lên server.

**Vòng lặp phát hiện (mỗi 2 giây):**
- **faceDetector.detect(videoEl)** phân tích frame webcam.
- **Thấy mặt:** Ghi **(cx, cy)** (tâm bounding box) vào rolling buffer 15 điểm → **tracking**.
- **Không thấy mặt:** Đếm thời gian vắng mặt → **warning**. Sau **5 phút** → **onAfkTimeout** → phiên dừng.

**Liveness Detection (chống ảnh/video):**
- Buffer đủ 15 điểm → tính **variance** tọa độ khuôn mặt.
- Người thật có vi chuyển động → variance > **0.5 px²**.
- variance < 0.5 px² **3 lần liên tiếp** (~90 giây) → **SPOOF DETECTED** → phiên dừng.

**Tạm dừng thông minh:** Khi Extension gửi yêu cầu AI classify video, Desktop phát **ai-classifying: true** → Face Tracking tạm dừng (**pauseTracking**) để giảm tải CPU. Khi AI trả về → **resumeTracking()** và reset **lastFaceDetectedAt** tránh AFK giả.

### k. Xử lý Vi phạm (Strike) & Kết thúc phiên ở Server (studySessionService.mjs)

![Phiên tập trung hoàn tất](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221113_yzsprj.png)
*Hình 12: Thông báo tổng kết kết quả học tập khi kết thúc phiên thành công*

![Session thất bại](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_221104_ydyx8z.png)
*Hình 13: Thông báo phiên thất bại nếu người dùng vi phạm quá 3 lần (3 Strikes)*

**Record Strike (POST /strike):**
- Lambda đọc session từ DynamoDB, tăng **strikeCount** thêm 1.
- Nếu **strikeCount >= 3**: **tự động** set **status = FAILED** + ghi **endTime**. Client chỉ gửi **sessionId**, server tự quyết định.

**Kết thúc phiên (POST /end-study-session) — Thuật toán 100% server-side:**
```
strikeCount >= 3         → FAILED  (vi phạm quá nhiều)
elapsed >= expected - 30s → COMPLETED (học đủ thời gian, +30s tolerance cho network delay)
elapsed < expected - 30s:
  + casual mode → COMPLETED (được phép dừng sớm)
  + rank mode   → FAILED    (bắt buộc hoàn thành)
```
- **COMPLETED** + **rank mode**: **earnedPoints = floor(elapsed / 60)** → **ADD studyStats.rankScore :pts** vào USER_TABLE.
- Gọi **updateQuestProgress(userId, "FOCUS", studiedMinutes)** — nhiệm vụ tập trung học.
- Trả về **{ status, earnedPoints, questUpdate, profile, daily }**.
- Desktop phát **focus:sessionEndData** → Frontend gọi **ingestServerData()** → đồng bộ toàn bộ Redux Store.

---

## 5. Chi tiết luồng kỹ thuật: Chế độ Học và Thi Đua (Casual vs Rank Mode)

Hệ thống Focus cung cấp hai chế độ: Casual (Linh hoạt) và Rank (Kỷ luật). Sự khác biệt nằm ở cách hệ thống xử lý vòng đời của phiên học, đặc biệt là cơ chế **đếm vi phạm (Strike)** và **đánh giá kết quả** giữa Client (Electron) và Server (AWS).

### a. Khởi tạo phiên (Client-side Config)

![Giao diện Rank Mode cực hạn](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215911_j5oxrp.png)
*Hình 14: Giao diện Rank Mode khóa chặt nút dừng, bắt buộc người dùng rèn luyện kỷ luật*

- **Casual Mode:** Cho phép người dùng cấu hình thời lượng từ các mốc nhỏ (15, 25, 45, 60 phút) hoặc custom. Giao diện hiển thị nút "Stop" cho phép ngắt ngang.
- **Rank Mode:** Bắt buộc thời lượng tối thiểu 30 phút. Giao diện **vô hiệu hóa và ẩn hoàn toàn** nút "Stop" để ép buộc tính kỷ luật (Deep Work).
- **Cơ chế chung:** Cả hai chế độ đều gửi IPC **focus:start** gọi API **POST /start-study-session** tạo bản ghi DynamoDB (lưu **sessionId**, **mode**, **status: PENDING**, **strikeCount: 0**).

### b. Giám sát thời gian thực & Xử lý Vi phạm (Strike)
Quá trình đếm Strike được kích hoạt bởi 3 nguồn: từ Browser Extension (**STRIKE_REPORT** qua WebSocket), từ App Blocker (AI phân loại ứng dụng Windows và hết 10 giây đếm ngược), hoặc từ Extension Gate (phát hiện browser không có extension). Cả ba luồng Strike đều gọi cùng một API **POST /strike** lên server:

- **Luồng xử lý khi có sessionId hợp lệ (có kết nối server):**
  - **Client-side:** Khi có vi phạm, **focusEngine.js** lập tức gọi API **POST /strike** truyền lên **sessionId**.
  - **Server-side (AWS Lambda studySessionService.mjs):**
    - Đọc session từ DynamoDB, tăng **strikeCount** hiện tại + 1.
    - Nếu **newCount >= 3**: Lambda tự động **ép trạng thái FAILED** (SET strikeCount = 3, status = "FAILED", endTime = :now), sau đó trả về **{ strikeCount: 3, sessionEnded: true }**.
    - Nếu **< 3**: Chỉ update **strikeCount** và trả về **{ sessionEnded: false }**.
  - **Phản hồi Client:** Electron nhận **sessionEnded: true**, lập tức phát **session-failed** lên React, kết thúc ngay phiên học mà không cần chờ đếm ngược hết giờ.

- **Luồng xử lý khi không có sessionId hợp lệ (local fallback):**
  - Khi API **startSession** thất bại lúc khởi động, Backend dùng **local_${Date.now()}** làm sessionId tạm.
  - Khi có Strike, **focusEngine.js** nhận ra sessionId bắt đầu bằng **local_** → **không gọi API server**, thay vào đó tự tăng biến cục bộ **strikeCount++** và gửi IPC **strike-recorded** lên React để cập nhật UI.
  - Nếu **strikeCount >= 3** ở client → Electron tự gọi **endSessionFail()**.

### c. Kết thúc và Đánh giá phiên (Server-side Logic — studySessionService.mjs)
Khi hết giờ (hoặc bị ép kết thúc), Client gọi **POST /end-session**. Thuật toán đánh giá được thực hiện **100% trên Server** dựa vào **elapsed** (thời gian thực học) và **expected** (thời gian cam kết):
- **Trường hợp kết thúc sớm (elapsed < expected - 30s):**
  - **Casual Mode:** Server đánh giá trạng thái là **COMPLETED** (vì Casual cho phép dừng sớm).
  - **Rank Mode:** Server đánh giá trạng thái là **FAILED** (do Rank cấm dừng sớm, vi phạm kỷ luật).
- **Trường hợp vi phạm từ trước (Strike >= 3):**
  - Đã bị đánh dấu **FAILED** từ trước thông qua API **/strike**.
- **Trường hợp hoàn thành đủ thời gian:** Cả hai đều đạt **COMPLETED**. Tuy nhiên, **chỉ Rank Mode** mới kích hoạt luồng tính điểm Rank: **earnedPoints = floor(elapsed / 60)**. Lambda chạy lệnh **UpdateCommand** với biểu thức **ADD studyStats.rankScore :pts** vào **USER_TABLE**.
- **Nhiệm vụ (Quest):** Dù **FAILED** hay **COMPLETED**, cả hai chế độ đều kích hoạt hàm **updateQuestProgress** để cộng tiến độ số phút học (nhiệm vụ: Tập trung học tập) dựa trên **elapsed** thực tế.

---

## 6. Chi tiết luồng kỹ thuật: Hệ thống Nhiệm vụ ngày (Quest System — questService.mjs)

Hệ thống nhiệm vụ được thiết kế chặt chẽ giữa React (Redux) và AWS Serverless (DynamoDB) để đảm bảo không có lỗ hổng lợi dụng (exploit) phần thưởng.

### a. Luồng Khởi tạo & Làm mới nhiệm vụ (Daily Reset)

![Hệ thống nhiệm vụ hàng ngày (Daily Quests) và nhận thưởng](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215927_nwm2bu.png)
*Hình 15: Hệ thống nhiệm vụ hàng ngày khuyến khích người dùng duy trì thói quen*

**Bước 1 — Trigger và Kiểm tra hạn (syncService.mjs):**
Mỗi khi khởi động app hoặc gọi API **GET /daily**, Backend kích hoạt **getOrRefreshDaily()**. Hàm này query bản ghi **daily** của user trong DynamoDB. Nếu thuộc tính **expiresAt** < Unix time hiện tại (đã sang ngày mới), luồng làm mới được kích hoạt.

**Bước 2 — Sinh nhiệm vụ ngẫu nhiên:**
- Truy xuất toàn bộ kho nhiệm vụ (Master Quests) thông qua cache **getCachedQuests()** để giảm số lượng Read Capacity Unit (RCU) trên DynamoDB.
- Chọn cứng 2 nhiệm vụ cốt lõi không bao giờ thay đổi: **focus_daily** (Thời gian tập trung) và **all_daily** (Hoàn thành tất cả nhiệm vụ khác).
- Bỏ 2 quest trên khỏi pool, chạy thuật toán **Fisher-Yates Shuffle** để xáo trộn và bốc ngẫu nhiên 3 nhiệm vụ phụ.

**Bước 3 — Tính toán Streak (Chuỗi học tập liên tiếp):**
- Server so sánh **lastFocusDate**. Nếu người dùng bỏ lỡ ngày hôm qua (lastFocusDay !== todayDay - 1), Backend reset thuộc tính **streak = 0** và **timeToStreak = 30** (số phút cần để khôi phục streak).

**Bước 4 — Ghi nhận dữ liệu mới:**
- Lưu đè danh sách nhiệm vụ mới vào **QUEST_TABLE** thông qua **PutCommand** (cài đặt **expiresAt** bằng Unix time của 23:59:59 ngày hiện tại).
- Cập nhật chuỗi streak mới vào **USER_TABLE** bằng **UpdateCommand**.

### b. Luồng Cập nhật Tiến độ Nội bộ (updateQuestProgress)
Mọi hành động (trả lời đúng câu hỏi Quiz, kết thúc phiên Focus) đều không gọi trực tiếp API cập nhật Quest từ Client mà thông qua hàm nội bộ ở Backend:
1. Đọc lại **daily** từ **QUEST_TABLE**. Kiểm tra **expiresAt** để đảm bảo chưa qua ngày mới (ngăn cập nhật lùi).
2. Duyệt qua mảng quests, tìm quest có **type** trùng khớp sự kiện và biến **isCompleted === false**.
3. Tăng **progress += amount**. Nếu **progress >= target**, đánh dấu **isCompleted = true**.
4. Nếu có một quest con chuyển sang **isCompleted**, hệ thống tự động tăng **progress** cho quest tổng **all_daily**. Khi **all_daily.progress == 4** → **all_daily** hoàn thành.
5. Ghi ngược toàn bộ mảng quest biến đổi vào DynamoDB và trả bộ dữ liệu mới về Client để Redux Store đồng bộ lại UI.

### c. Luồng Nhận thưởng (POST /daily/claim)
Đây là khâu quan trọng nhất được bảo vệ kỹ lưỡng bằng giao dịch nguyên tử (Atomic).

**Bước 1 — Xác thực:** Server kiểm tra điều kiện khắt khe **isCompleted === true** và **isClaimed === false**.

**Bước 2 — Atomic Transaction (DynamoDB TransactWrite):** Tránh hoàn toàn lỗi Race Condition (Client gửi 2 request nhận thưởng cùng 1 mili-giây để nhân đôi phần thưởng). Cả 2 lệnh sau phải thành công cùng lúc hoặc thất bại cùng lúc:
- **Lệnh 1:** **Update** cờ **isClaimed = true** trong **QUEST_TABLE**.
- **Lệnh 2:** Dùng biểu thức **ADD knowledgePoint :reward** để cộng điểm KP vào **USER_TABLE**.

**Bước 3 — Đồng bộ:** Backend trả về bộ object **profile** (KP mới) và **daily** (trạng thái quest mới). Client nhận được liền dispatch action vào Redux, cập nhật hiển thị đồng xu nảy lên mà không cần fetch lại toàn bộ dữ liệu.