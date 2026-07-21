---
title : "Study with AI"
date : 2026-07-17 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

The project integrates AI into two core functions: **AI Study Planner** (smart planning and review) and **AIGuard** (real-time study discipline monitoring system). Both create a closed loop: AI helps users learn *more efficiently*, AIGuard ensures users *actually study*, and the task system (Quest) and leaderboard (Rank) turn that discipline into **real valuable rewards**.

---

## 1. Overview

![Study Play Main Interface](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_214455_qtkwxn.png)
*Figure 1: Study Play Main Interface (Dashboard)*

### AI Study Planner
A comprehensive personal academic assistant integrated right into the application. Users can **upload study materials** (PDF, DOCX, TXT), then converse naturally with AI. AI acts as an "academic advisor", proactively gathering information about goals, knowledge volume, and deadlines, then automatically designing a **detailed study plan** with specific Topics and durations. From that roadmap, AI generates **dynamic multiple-choice question sets (Quiz)** by topic to practice Active Recall. Upon submission, each correct answer adds **Knowledge Points (KP)** and instantly updates quest progress on the server.

### AIGuard — Discipline Monitoring System
A distributed system running simultaneously on the Desktop App (Electron Backend) and Browser Extension (Chrome, Edge,...). The system **does not hard-block YouTube videos or websites**, but uses a **3-tier** classification architecture to evaluate the context of each content. In addition, the system integrates an **AI App Blocker** to detect and handle entertainment applications running on Windows (games, streaming, etc.) via a 10-second countdown warning window. AIGuard also integrates **Face Tracking with Liveness Detection** to prevent AFK and photo cheating. All violations (Strikes) are recorded and processed entirely **server-side** via AWS Lambda → DynamoDB, ensuring results cannot be tampered with. After the study session ends, the server calculates Rank points and updates Quest progress — data is pushed to the client to synchronize the Redux Store.

---

## 2. AI Settings

![AI Provider Configuration](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_214522_b8tndd.png)
*Figure 2: Users can select from various local and cloud AI providers*

Users configure the AI provider for each function in the **AI Settings** tab. Each function (**AI Blocker** for AIGuard and **AI Study Planner**) is configured separately with the same 3 provider options:

| Provider | Mechanism | Configuration |
|---|---|---|
| **Ollama (Local)** | Runs AI models directly on the user's computer | When opening the AI Settings tab, the app automatically queries **http://localhost:11434/api/tags** to scan the list of running models, users select from a dropdown |
| **Google Gemini (Cloud)** | Calls Gemini API via HTTPS | Users enter Gemini API Key, saved in local store, not sent to the server |
| **AWS Bedrock (Cloud)** | AWS Signature V4 + STS AssumeRole cross-account signing | Uses logged-in Cognito credentials — no additional configuration needed, just toggle ON |

When clicking "Save", settings are saved simultaneously to the **Redux Store** (global state) and local storage via IPC **store:saveAiSettings**, and synced separately for StudyPlanner via IPC **study:saveSettings**. The system checks if the selected Ollama model is in the **WEAK_MODELS** list — including overly small models (qwen2.5:1.5b, tinyllama, phi3:mini, gemma:2b, orca-mini) which will be flagged with a warning because they lack the capability to generate reliable structured JSON.

---

## 3. Technical Flow Details: AI Study Planner

### a. Document Upload & Parsing Flow (fileParser.js + documentContext.js)

![AI Chat Interface gathering info and parsing doc](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_215349_cmycre.png)
*Figure 3: AI Assistant gathering goals and analyzing uploaded documents*

**Step 1 — IPC Communication:**
The user clicks attach (📎) in **ChatTab**. The file selection dialog opens via IPC **dialog:openFile**. The file path is sent via IPC **study:uploadFile** down to the Electron Backend.

**Step 2 — Read file (fileParser.js):**
- Size check: rejects files > **25MB** to avoid memory overflow.
- Detects file type by extension (.pdf, .docx, .doc, .txt).
- **Lazy loading libraries:** **pdf-parse** and **mammoth** (DOCX parser) are imported dynamically (**await import(...)**) instead of loading at startup — only downloaded when actually needed, saving startup memory.
- **PDF:** Uses **pdf-parse** to read buffer, extracts **data.text** and **data.numpages**.
- **DOCX/DOC:** Uses **mammoth.extractRawText()** to get raw text.
- **TXT:** Reads directly using **fs.readFileSync(path, 'utf-8')**.
- Post-read check: if text < 50 characters → reports empty/corrupted file error.

**Step 3 — Clean text (cleanText):**
The raw text string is processed through the **cleanText** function:
- Normalizes line endings (\r\n → \n).
- Removes control characters except \n and \t.
- Collapses multiple consecutive empty lines to a maximum of 2 lines.
- Collapses multiple spaces into 1.

**Step 4 — Hierarchical Chunking (buildChunks):**
This is the most critical optimization of the document pipeline. Instead of cutting text by fixed size, the system detects **headings** to create semantic chunks:

*Heading recognition by patterns (HEADING_PATTERNS):*
- Lines starting with "Chapter/Part/Lesson/Section + number".
- Lines starting with a number + dot + uppercase letter (1. Introduction).
- Roman numerals (IV. ...).
- Markdown headings (#, ##, ###, ####).
- ALL CAPS lines, shorter than 80 characters.

*Chunking mechanism:*
- Whenever a new heading is encountered → flush current chunk, start a new chunk with the new section name.
- Target: **250 words/chunk**. Maximum: **400 words/chunk**.
- When exceeding the target, it prioritizes cutting at an **empty line** to avoid splitting paragraphs.
- Chunks that are too short (< 30 characters) are discarded.
- Each chunk records: **id**, **section** (heading title), **content**, **wordCount**.

**Step 5 — AI Summarization (summarizeDocument):**
After having chunks, the Backend calls **summarizeDocument**:
- **Gemini/Bedrock:** Combines all chunks into **docText**, sends everything to AI to summarize in 200-300 words and list main topics.
- **Ollama (Local):** Due to context window limits, the text is **hard-capped at a maximum of 2500 characters** before sending. All section names are listed separately so AI still understands the overall structure. Temperature is **0.1** (very low) for stable results, output is forced to be JSON.
- Saved result: **{ summary, topics[] }**.

**Step 6 — Save to Session (documentContext.js — Singleton):**
The entire document is saved into an **in-memory singleton** **_document**:
```
{ fileName, fileType, chunks[], summary, topics[], charCount, wordCount, pageCount, uploadedAt }
```
- The singleton lives throughout the Desktop App session — it is not lost when switching tabs.
- Deleted when the user clicks X or uploads a new file.
- The Frontend displays a **Document Banner** (file name + page count) confirming it is ready.

### b. Context Injection Optimization — 3 different strategies (documentContext.js)

When a document is active, AI does not receive the entire raw text but receives **context selectively tailored to each type of task**:

**Tier 1 — Context for Chat (getContextForQuery):**
- Always injects **summaryBlock**: general summary + topic list.
- If the user query > 3 characters: calls **searchChunks(chunks, query, 3)** to find **up to 3 most relevant chunks** and injects an additional **chunksBlock**.
- This design significantly reduces outgoing tokens compared to injecting the whole document, especially useful with Bedrock (billed per token).
- With Ollama: only injects a concise **summaryBlock** (truncated to 600 characters) due to context window limits.

**Tier 2 — Context for Plan Generation (getContextForPlan):**
- Instead of searching, it takes **8 chunks evenly distributed** from the beginning to the end of the document (step = floor(chunks.length / 8)).
- Returns **structureBlock** (list of section titles) so AI creates a plan closely following the actual structure of the document.
- The prompt requires AI to create phases that "closely follow this Document Structure".

**Tier 3 — Context for Quiz (getContextForQuiz):**
- Calls **searchChunks(chunks, phaseName + topics, 5)** to get the **5 most relevant chunks** to the phase needing a quiz.
- The prompt requires AI to generate questions "strictly based 100% on this document" — avoiding AI hallucinating knowledge outside the document.

### c. Smart Chunk Search (BM25-style — searchChunks in fileParser.js)

**searchChunks** is a lightweight semantic search algorithm inspired by BM25:
1. Splits the query into keywords (normalize unicode, remove Vietnamese diacritics, exclude words < 3 characters).
2. For each chunk, calculates **score**:
   - Counts keyword occurrences in **content + section** (frequency).
   - **Bonus +3 points** if the keyword appears in the **section** (heading name) — prioritizing on-topic chunks.
3. Sorts by descending score, returning the top **topK** highest-scoring chunks.
4. Chunks with a score of 0 are completely discarded.

### d. Information Gathering Chat Flow (aiStudyService.js)

1. User sends a message → Frontend packages history (**messages[]**) and sends via IPC **study:chat**.
2. Backend calls **chatWithAI(messages)**, loads AI settings, gets context from **getContextForQuery(lastUserMsg)**.
3. **System Prompt** guides AI to act as an "Academic Advisor", needing to collect 6 fields: **subject**, **topic**, **level**, **goal**, **totalDuration**, **dailyHours**. When at least fields 1, 2, and 3 are sufficient → sets **readyToGenerate = true**.
4. AI supports **all languages** (Vietnamese, English, Chinese, Japanese, Korean, etc.), automatically replying in the user's active language.
5. LLM is forced to output **pure JSON** (no markdown): **{ reply, collectedInfo, readyToGenerate }**.
   - Ollama: enables **format: 'json'** to ensure correct format.
   - Timeout: 120 seconds for cloud, **300 seconds for Ollama** (local models need more loading time).
6. **parseResponse()** uses regex to extract JSON from the response — safe even with extraneous markdown code blocks.
7. Frontend accumulates **collectedInfo** across turns. When **readyToGenerate = true** → the "Create Plan" button appears.
8. The chat session is automatically saved locally (title = first 40 characters of the first message).

### e. Plan Generation Flow (generateStudyPlan)

![Detailed Study Plan](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_215432_my4exv.png)
*Figure 4: Detailed study plan generated by AI based on the document's content*

1. Frontend calls IPC **study:generatePlan** with **collectedInfo**.
2. Backend calls **getContextForPlan()** to get 8 evenly distributed chunks.
3. **PLAN_SYSTEM_PROMPT** forces the LLM to:
   - Output **pure JSON**, no explanations.
   - Write in **the same language as the user's input** (if input is Vietnamese → entire plan is in Vietnamese).
   - Schema: **{ title, description, phases[{ id, name, duration, description, topics[], resources[{ name, url, type }], completed }] }**.
   - 4-5 phases, 2-3 topics/phase, 1-2 resources/phase (website links).
4. Temperature **0.6** — lower than chat for more consistent results. Ollama timeout **600 seconds**.
5. Frontend receives JSON, saves it to Redux via **saveStudyPlan**, and automatically switches to PlanTab.

### f. Quiz Generation & Server Scoring Flow (generateQuiz + questService.mjs)

![Active Recall Quiz Interface](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646924/Screenshot_2026-07-21_215715_z2xzhx.png)
*Figure 5: Active Recall quiz interface designed to reinforce knowledge*

**Client side:**
1. User clicks "Generate Questions" in any Phase within a Topic.
2. Backend calls **getContextForQuiz(phaseName, topics)** — retrieving 5 relevant chunks.
3. **QUIZ_SYSTEM_PROMPT**: forces AI to generate exactly **10 multiple-choice questions**, 4 options A/B/C/D, in the same language as the phase name.
4. Schema: **{ questions[{ id, question, options[], correctAnswer }] }**. Temperature **0.7**.

**Server side (POST /study-planner/quiz-submit → questService.mjs):**
- Receives **{ correctAnswersCount, totalQuestions }**.
- Calculates **reward = correctAnswersCount × 10** KP.
- Uses **ADD budget.knowledgePoint :kp** (DynamoDB atomic operation) — points cannot be forged.
- Calls **updateQuestProgress(userId, "COMPLETE_QUIZ", 1)** — task completion for 1 quiz.
- Calls **updateQuestProgress(userId, "CORRECT_QUIZ_ANSWER", correctAnswersCount)** — correct answer task.
- Returns **{ earnedKP, questUpdate, profile, daily }** → Frontend synchronizes Redux.

---

## 4. Technical Flow Details: AIGuard & Focus System

### a. Startup & Session Start Conditions

![Focus Session Setup Control Panel](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_215747_zhrocq.png)
*Figure 6: The control panel for starting a new Focus Session*

1. FocusGuard component mounts → sends IPC **focus:setConfig** with JWT token + API URL down to Backend. Backend saves the token for future server API calls.
2. **3 mandatory conditions** before "Start":
   - If Browser is open, it must have the Extension installed.
   - Extension is connected via WebSocket.
   - At least one AI provider is **ready** (Bedrock / Gemini / Ollama).
3. Upon clicking "Start" → IPC **focus:start** → Electron calls **POST /start-study-session** to AWS Lambda with **{ mode, durationMinutes }**. Lambda creates a session in DynamoDB STUDY_TABLE (status: PENDING, strikeCount: 0, TTL 6 months), returns **sessionId**. If API fails → uses **local_${Date.now()}** as a fallback ID.

### b. Desktop — Browser WebSocket Architecture (focusEngine.js)

- **WebSocket Server:** Electron creates a **WebSocketServer** on **ws://localhost:8765** when the app starts.
- **PING/PONG:** Server sends **PING** to all clients every **10 seconds** to keep connections alive.
- **Extension Identification:** When the Extension connects, it sends **EXTENSION_CONNECTED** along with **browser** (Chrome, Edge, Opera, Brave, Firefox, CốcCốc). Backend saves it to **connectedBrowsers Map<ws, browserName>**.
- **Disconnect Grace Period:** When the Extension disconnects, Backend **waits 15 seconds** before considering it a true disconnect (preventing false alarms during browser refresh/reload). During this wait, the browser is still considered to "have the extension".
- **Settings Sync:** Upon Extension connection, server immediately sends **SETTINGS_UPDATED** with current **allowedCategories**.

### c. Extension Gate Check (checkGateStatus — every 3 seconds)

![Extension Gate Warning](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221237_dd7yme.png)
*Figure 7: Warning popup triggered when opening a browser without the extension enabled*

This is a protection mechanism to prevent users from opening browsers without the extension and browsing freely:

1. Runs **tasklist /NH /FO CSV** command to get the process list.
2. If a browser is found (chrome.exe, msedge.exe, opera.exe, brave.exe...) → runs additional **PowerShell** with query:
   ```
   Get-Process | Where-Object { $_.MainWindowHandle -ne 0 -and $_.ProcessName -match '^(chrome|msedge|opera)$' }
   ```
   **Optimization:** Filters `MainWindowHandle != 0` to eliminate **"ghost processes"** (Edge Startup Boost, background processes without windows) — avoiding false positives.
3. Compares running browser list with **connectedBrowsers**. Any running browser without an extension is added to **currentlyMissing**.
4. **10-second grace period:** Newly opened browsers need time for the extension to boot and connect. Only if missing after 10 seconds is it considered **definitelyMissing**.
5. If a browser without an extension is detected **while the timer is running** → automatically **stopTimerForcefully()**.

### d. AI-powered App Blocker — Windows App Classification & Blocking (appBlocker.js + focusEngine.js)

![App Blocker Warning and 10s Countdown](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646925/Screenshot_2026-07-21_215830_mtsjj5.png)
*Figure 8: 10-second countdown warning when an entertainment app is detected*

This function monitors applications running directly on Windows. Unlike hard-blocking by list, the system uses AI to classify any running application with a window.

**Step 1 — Scan foreground processes (every 3 seconds — focusEngine.js):**
Instead of using **tasklist** (which returns all processes including background services and drivers), the Backend runs a PowerShell command:
```powershell
Get-Process | Where-Object { $_.MainWindowHandle -ne 0 } | Select-Object ProcessName
```
Filtering `MainWindowHandle != 0` ensures it only grabs applications currently displaying a window — completely eliminating background services, drivers, and "ghost processes". Furthermore, the **SYSTEM_PROCESSES** list (chrome, msedge, electron, explorer,...) is filtered out to avoid overlapping with Extension Gate processing.

**Step 2 — In-flight deduplication:**
Before sending to AI, the system checks **inFlightChecks** (a Set): if a process is already being classified by AI, it is skipped to prevent duplicate requests.

**Step 3 — Fetch Windows metadata (PowerShell VersionInfo):**
For each new process, **getWindowsAppMetadata()** runs PowerShell to read **VersionInfo** from the **.exe** file:
```powershell
$vi = (Get-Item $path).VersionInfo
$vi | Select-Object FileDescription, ProductName, CompanyName, OriginalFilename | ConvertTo-Json
```
Results include: **fileDescription**, **productName**, **companyName**, **fileName** — providing much richer context than just the process name.

**Step 4 — AI Classification (classifyApp — Bedrock → Gemini → Ollama):**
Metadata is sent to AI in provider priority order: **Bedrock → Gemini → Ollama**. If the selected provider fails, the system automatically tries the next one.

*APP_SYSTEM_PROMPT* has two core rules:
- **ALLOW:** Productivity tools, code editors, development software, office suites, communication tools (chat/email), system utilities, and any work-related software.
- **BLOCK:** Video games, game launchers, music/movie streaming apps (Spotify, VLC, Netflix,...), entertainment platforms, and any application primarily for entertainment.

Temperature is **0.1** — extremely low to ensure consistent, non-creative results.

**Step 5 — 24h Cache:**
Classification results are saved to **appCache** (Map with lowercase processName as key). The next time the same process is encountered → it returns immediately from cache without calling AI again, saving significant time and token costs.

**Step 6 — 10-second Overlay (Floating Electron Window):**
When AI verdicts **BLOCK**, Backend creates a small Electron window (**BrowserWindow**) displaying right above the taskbar with info:
- Name of the blocked app and processName.
- Block reason from AI (AI Verdict).
- 10-second countdown bar.
- **"Close this app immediately"** button — users can proactively close to avoid a strike.

The overlay window is created with `alwaysOnTop: true`, `focusable: false` (doesn't steal focus from the blocked app), `transparent: true` and displays smartly: if the mini Focus Mode widget is showing, the overlay automatically shifts up to avoid overlapping.

**Step 7 — 10-second Result Handling:**
Throughout the 10 seconds, Backend checks every 1 second if the app is still running:
- **User closes app manually (within 10s):** Overlay disappears, **no Strike recorded**.
- **User clicks "Close immediately":** App is killed via **taskkill /F /IM**, **no Strike recorded**.
- **After 10 seconds, app still running:** Backend automatically executes **taskkill /F /IM** and calls **recordAppStrike()**.

**Step 8 — Strike Recording:**
**recordAppStrike()** calls the same **POST /strike** API as the Web/Video blocker — sends **sessionId** to AWS Lambda, server increments **strikeCount** in DynamoDB. If reaching 3 Strikes → **endSessionFail()**.

### e. Continuous AI Health Check (every 5 seconds — focusEngine.js)

While the timer is active, a background interval every **5 seconds** calls **getAiStatus()**:
- Checks the status of **all providers** (Bedrock, Gemini, Ollama, Groq) in parallel using **Promise.all()**.
- Determines the **activeProvider** by priority order: Bedrock → Gemini → Ollama → Groq.
- If no provider remains **ready** → **automatically stops session** (**stopTimerForcefully**) and sends **ai-status-lost** event to Frontend.
- Ensures the study session always has an AI ready for content classification.

### f. 3-Tier YouTube Classification System (aiGuard.js)

![YouTube Video Blocked](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646925/Screenshot_2026-07-21_221420_hauneg.png)
*Figure 9: AI Guard blocking an entertainment YouTube video directly in the browser*

**Step 0 — Metadata Extraction (extractor.js — MAIN world):**
- **extractor.js** runs in the MAIN world (direct access to YouTube's page JS).
- Listens for **__fg_extract__** event from **content.js**. When triggered, accesses **player.getPlayerResponse()** (priority) or **window.ytInitialPlayerResponse** (fallback) to extract: **videoId**, **title**, **author**, **channelId**, **category**, **keywords**, **description** (first 500 characters).
- Only sends if **videoId** matches the current URL (preventing stale data from SPA navigation).
- On SPA navigation (**yt-navigate-finish**), retries at 500ms / 1.5s / 3s / 5s.

**Tier 1 — Static Classification (Whitelist / Blacklist / Category):**
- Checks **channelId**/**author** against **Channel Whitelist** (**channels.json**): Khan Academy, 3Blue1Brown, CrashCourse, Veritasium, TED, TED-Ed, MIT OCW, freeCodeCamp, Fireship, Kurzgesagt, Numberphile, SmarterEveryDay, Vsauce, MinutePhysics, Computerphile, The Organic Chemistry Tutor, Professor Leonard, Traversy Media, The Coding Train, Academind, Bro Code, Web Dev Simplified. → **ALLOW** immediately.
- Checks **Channel Blacklist** (user customized). → **BLOCK** immediately.
- Checks **category** against **allowedCategories** (default: Education, Science & Technology): → **ALLOW**.
- Checks **BLOCKED_CATEGORIES** (hardcoded): Gaming, Music, Entertainment, Comedy, Film & Animation, Movies, Shows, Trailers, Anime & Animation. → **BLOCK**.
- Categories not in any list (Sports, People & Blogs, Howto,...) → **UNCERTAIN** → proceeds to Tier 2.

**Tier 2 — Real-time AI Classification (aiGuard.js classifyContent):**

*Request flow:*
- **content.js** shows "🤖 AI is analyzing..." overlay → sends **CLASSIFY_VIDEO** + metadata → **background.js** → WebSocket → focusEngine.js → calls **classifyContent(metadata)**.
- **buildUserPrompt(metadata)**: combines Title, Channel, Category, Keywords (max 10), Description (max 300 characters) into a concise prompt.

*Provider priority order:*
1. **AWS Bedrock** (if provider = bedrock).
2. **Google Gemini** (if provider = gemini + has apiKey). Safety check: if selectedModel doesn't start with "gemini" (user mistakenly left Ollama model) → auto fallbacks to **gemini-2.0-flash**.
3. **Ollama** (if provider = ollama + running + model starts with "qwen3").
4. If no provider → defaults to **BLOCK**.

*Temperature 0.1* for all classifications (consistent results, low creativity).

*parseAiResponse():* Uses regex to safely extract JSON. If regex fails → searches for keywords ALLOW/BLOCK in text (fallback). If nothing found → defaults to **BLOCK** (safest).

*Classification Cache (24h TTL):*
- Cached by **videoId** (video), **domain** (webpage) using **Map<key, { result, timestamp }>**.
- Previously classified video/domain → returns immediately, skips 2nd AI call.
- Cache can be manually cleared via **clearCache()**.

*Return flow:* focusEngine → WebSocket → **background.js** → **content.js** removes overlay, applies ALLOW/BLOCK with **reason** and **provider** (showing which AI judged it).

**Tier 3 — 10-Second Countdown & Strike:**
- **BLOCK**: blocking overlay appears, video is **paused + muted** every 500ms (preventing playback/skipping).
- Simultaneously shows **10-second countdown** with a progress bar.
- Leaving the page within 10 seconds: no violation.
- Staying > 10 seconds: **STRIKE_REPORT** → WebSocket → **recordStrike()** → **POST /strike** → DynamoDB.

### g. Gemini API Optimization — Auto Fallback Chain (geminiApi.js)

Gemini API frequently encounters 503 (overloaded) or 429 (rate limit). **geminiRequestWithFallback** solves this with a mechanism:

**Chain of 5 models (tested in order):**
```
[preferredModel] → gemini-flash-latest → gemini-2.0-flash → gemini-2.0-flash-lite → gemini-2.5-flash → gemini-2.5-pro
```
(User's preferred model is tried first, never skipped)

**Retry strategy by error type:**
| Error | Action |
|---|---|
| **503 Overloaded** | Retry max 2 times, waiting **10 seconds** each. If still fails → switch to next model |
| **429 Rate Limit** | Parse wait time from message (retry in Xs), wait exact time + 1 sec. Max 2 times |
| **404 Not Found / Limit 0** | Retry 2 times, waiting **5 seconds**. If still fails → switch to next model |
| **Timeout (AbortController)** | Immediately switch to next model |
| **All models fail** | Wait **15 seconds** then restart entire chain. Max 2 global retries |

### h. AWS Bedrock — No SDK Needed, Cross-account STS (bedrockApi.js)

Instead of using the heavy **@aws-sdk/client-bedrock-runtime**, the system custom-implements it:
- **AWS Signature V4** from scratch using Node.js **crypto** module — signing HMAC-SHA256 for every request.
- **STS AssumeRole Cross-Account:** Uses Cognito credentials (Account A) to call **sts.amazonaws.com/AssumeRole** for temporary credentials to role **CrossAccountBedrockRole** in Account B (hosting Bedrock). Credentials are cached and auto-refreshed upon expiry.
- **Token logging:** On every successful Bedrock call, logs **In=X, Out=Y tokens** to track costs.

### i. Hard-block Web and Any Webpage Classification

![Entertainment Webpage Blocked](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221356_rlu6ah.png)
*Figure 10: Blocking interface when attempting to read fictional stories during study time*

**Hard-blocked Web:**
- Hardcoded list: **facebook.com**, **twitter.com/x.com**, **tiktok.com**, **threads.net**, **instagram.com**.
- Detection → instant overlay (bypasses AI). 10s countdown + Strike.
- Re-checks every 3 seconds to prevent overlay bypassing.

**Any Webpage (classifyWebPage):**
1. Checks **Safe Whitelist** (hardcoded): Google, GitHub, Stack Overflow, Notion, ChatGPT, Claude, MDN, LeetCode, Coursera, Udemy, Figma, Slack, Zoom,... → Bypasses.
2. Checks **Domain Cache** (24h TTL, by domain).
3. New Domain: shows small badge "🛡️ AI is analyzing...", after 2 seconds extracts metadata (**url**, **domain**, **title**, **description**, **og:title**, **og:description**, **keywords**, **h1**) → sends **CLASSIFY_PAGE** via WebSocket.
4. **WEB_SYSTEM_PROMPT** is stricter than YOUTUBE_SYSTEM_PROMPT: Only ALLOW if clearly educational/work-related. **Specifically blocks fiction/novels/manga even if the domain name suggests "reading" or "literature"**. Defaults to BLOCK if uncertain.
5. Provider priority same as YouTube. Cached by domain.

### j. Face Recognition & Liveness Detection (faceTracker.js)

![Face Tracking System monitoring study session](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215846_z1iind.png)
*Figure 11: Face Tracking system monitoring AFK and Liveness using local AI*

**Initialization:**
- Loads **MediaPipe BlazeFace Short Range (TFLite, float16)** via jsDelivr CDN.
- Prioritizes **GPU** (delegate: GPU). If fails → auto fallback to **CPU**.
- Webcam opens at **320×240** to save resources.
- Model runs **100% locally** — no images are sent to the server.

**Detection Loop (every 2 seconds):**
- **faceDetector.detect(videoEl)** analyzes webcam frame.
- **Face seen:** Records **(cx, cy)** (bounding box center) into a rolling 15-point buffer → **tracking**.
- **No face seen:** Counts absence time → **warning**. After **5 minutes** → **onAfkTimeout** → session stops.

**Liveness Detection (anti-photo/video):**
- Buffer reaches 15 points → calculates facial coordinate **variance**.
- Real humans have micro-movements → variance > **0.5 px²**.
- variance < 0.5 px² for **3 consecutive times** (~90 seconds) → **SPOOF DETECTED** → session stops.

**Smart Pause:** When Extension sends request for AI to classify video, Desktop emits **ai-classifying: true** → Face Tracking pauses (**pauseTracking**) to reduce CPU load. When AI responds → **resumeTracking()** and resets **lastFaceDetectedAt** to prevent fake AFK.

### k. Strike Handling & Server-side Session End (studySessionService.mjs)

![Focus Session Completed](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646926/Screenshot_2026-07-21_221113_yzsprj.png)
*Figure 12: Session summary popup displayed upon successful completion*

![Focus Session Failed](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646923/Screenshot_2026-07-21_221104_ydyx8z.png)
*Figure 13: Session failed notification triggered after 3 strikes*

**Record Strike (POST /strike):**
- Lambda reads session from DynamoDB, increments **strikeCount** by 1.
- If **strikeCount >= 3**: **automatically** sets **status = FAILED** + records **endTime**. Client only sends **sessionId**, server decides.

**End Session (POST /end-study-session) — 100% Server-side Algorithm:**
```
strikeCount >= 3         → FAILED  (too many violations)
elapsed >= expected - 30s → COMPLETED (studied enough time, +30s tolerance for network delay)
elapsed < expected - 30s:
  + casual mode → COMPLETED (allowed to stop early)
  + rank mode   → FAILED    (mandatory completion)
```
- **COMPLETED** + **rank mode**: **earnedPoints = floor(elapsed / 60)** → **ADD studyStats.rankScore :pts** into USER_TABLE.
- Calls **updateQuestProgress(userId, "FOCUS", studiedMinutes)** — focused study task.
- Returns **{ status, earnedPoints, questUpdate, profile, daily }**.
- Desktop emits **focus:sessionEndData** → Frontend calls **ingestServerData()** → syncs entire Redux Store.

---

## 5. Technical Flow Details: Casual vs Rank Mode

The Focus System provides two modes: Casual (Flexible) and Rank (Disciplined). The difference lies in how the system handles the study session lifecycle, specifically the **violation counting (Strike)** mechanism and **result evaluation** between Client (Electron) and Server (AWS).

### a. Session Initialization (Client-side Config)

![Rank Mode Extreme Interface](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215911_j5oxrp.png)
*Figure 14: Rank Mode interface with the "Stop" button permanently disabled to enforce discipline*

- **Casual Mode:** Allows users to configure duration from small intervals (15, 25, 45, 60 minutes) or custom. The UI displays a "Stop" button allowing premature interruption.
- **Rank Mode:** Mandates a minimum duration of 30 minutes. The UI **disables and completely hides** the "Stop" button to enforce discipline (Deep Work).
- **Common Mechanism:** Both modes send IPC **focus:start** calling API **POST /start-study-session** to create a DynamoDB record (saving **sessionId**, **mode**, **status: PENDING**, **strikeCount: 0**).

### b. Real-time Monitoring & Strike Handling
The Strike counting process is triggered by 3 sources: from Browser Extension (**STRIKE_REPORT** via WebSocket), from App Blocker (AI classifying Windows apps and 10-second countdown end), or from Extension Gate (detecting browser without extension). All three Strike flows call the same API **POST /strike** to the server:

- **Handling flow with valid sessionId (server connected):**
  - **Client-side:** Upon violation, **focusEngine.js** instantly calls API **POST /strike** passing the **sessionId**.
  - **Server-side (AWS Lambda studySessionService.mjs):**
    - Reads session from DynamoDB, increments current **strikeCount** + 1.
    - If **newCount >= 3**: Lambda automatically **forces FAILED state** (SET strikeCount = 3, status = "FAILED", endTime = :now), then returns **{ strikeCount: 3, sessionEnded: true }**.
    - If **< 3**: Only updates **strikeCount** and returns **{ sessionEnded: false }**.
  - **Client Response:** Electron receives **sessionEnded: true**, immediately emits **session-failed** to React, ending the study session instantly without waiting for the timer to run out.

- **Handling flow without valid sessionId (local fallback):**
  - When **startSession** API fails at startup, Backend uses **local_${Date.now()}** as a temporary sessionId.
  - Upon a Strike, **focusEngine.js** recognizes the sessionId starts with **local_** → **does not call server API**, instead locally increments **strikeCount++** and sends IPC **strike-recorded** to React to update UI.
  - If **strikeCount >= 3** on the client → Electron natively calls **endSessionFail()**.

### c. Session End & Evaluation (Server-side Logic — studySessionService.mjs)
When time runs out (or forced end), the Client calls **POST /end-session**. The evaluation algorithm is executed **100% on the Server** based on **elapsed** (actual study time) and **expected** (committed time):
- **Case of premature end (elapsed < expected - 30s):**
  - **Casual Mode:** Server evaluates status as **COMPLETED** (since Casual allows stopping early).
  - **Rank Mode:** Server evaluates status as **FAILED** (since Rank forbids stopping early, violating discipline).
- **Case of prior violation (Strike >= 3):**
  - Already marked **FAILED** previously via the **/strike** API.
- **Case of full completion:** Both achieve **COMPLETED**. However, **only Rank Mode** triggers the Rank point calculation flow: **earnedPoints = floor(elapsed / 60)**. Lambda runs **UpdateCommand** with expression **ADD studyStats.rankScore :pts** into **USER_TABLE**.
- **Quests:** Whether **FAILED** or **COMPLETED**, both modes trigger the **updateQuestProgress** function to add study minute progress (task: Focused studying) based on actual **elapsed** time.

---

## 6. Technical Flow Details: Daily Quest System (questService.mjs)

The quest system is tightly designed between React (Redux) and AWS Serverless (DynamoDB) to ensure no exploit loopholes exist for rewards.

### a. Initialization & Daily Reset Flow

![Daily Quests System and Reward Claiming](https://res.cloudinary.com/nbwjjr1y/image/upload/v1784646922/Screenshot_2026-07-21_215927_nwm2bu.png)
*Figure 15: Daily quests system incentivizing users to maintain learning habits*

**Step 1 — Trigger and Deadline Check (syncService.mjs):**
Whenever the app starts or calls **GET /daily** API, Backend triggers **getOrRefreshDaily()**. This function queries the user's **daily** record in DynamoDB. If **expiresAt** < current Unix time (crossed into a new day), the refresh flow activates.

**Step 2 — Generate Random Quests:**
- Retrieves entire quest pool (Master Quests) via cache **getCachedQuests()** to reduce Read Capacity Units (RCU) on DynamoDB.
- Hard-selects 2 core quests that never change: **focus_daily** (Focus time) and **all_daily** (Complete all other quests).
- Removes these 2 quests from the pool, runs **Fisher-Yates Shuffle** algorithm to randomize and draw 3 side quests.

**Step 3 — Streak Calculation (Consecutive Study Days):**
- Server compares **lastFocusDate**. If the user missed yesterday (lastFocusDay !== todayDay - 1), Backend resets attribute **streak = 0** and **timeToStreak = 30** (minutes needed to restore streak).

**Step 4 — Record New Data:**
- Overwrites the new quest list into **QUEST_TABLE** via **PutCommand** (setting **expiresAt** to Unix time of 23:59:59 of the current day).
- Updates the new streak into **USER_TABLE** using **UpdateCommand**.

### b. Internal Progress Update Flow (updateQuestProgress)
Every action (correct Quiz answer, finishing Focus session) does not call the Quest update API directly from the Client but via an internal Backend function:
1. Re-reads **daily** from **QUEST_TABLE**. Checks **expiresAt** to ensure it hasn't crossed a new day (preventing retro-updates).
2. Iterates through quests array, finds quest with matching event **type** and **isCompleted === false**.
3. Increments **progress += amount**. If **progress >= target**, marks **isCompleted = true**.
4. If a child quest turns **isCompleted**, the system automatically increments **progress** for the master **all_daily** quest. When **all_daily.progress == 4** → **all_daily** is completed.
5. Writes the modified quest array back to DynamoDB and returns the new dataset to Client for Redux Store to resync the UI.

### c. Claim Reward Flow (POST /daily/claim)
This is the most critical step, carefully protected by an Atomic transaction.

**Step 1 — Validation:** Server strictly checks conditions **isCompleted === true** and **isClaimed === false**.

**Step 2 — Atomic Transaction (DynamoDB TransactWrite):** Completely avoids Race Condition errors (Client sending 2 claim requests in the same millisecond to double rewards). Both following commands must succeed simultaneously or fail simultaneously:
- **Command 1:** **Update** flag **isClaimed = true** in **QUEST_TABLE**.
- **Command 2:** Uses expression **ADD knowledgePoint :reward** to add KP points to **USER_TABLE**.

**Step 3 — Sync:** Backend returns the new **profile** object (new KP) and **daily** (new quest state). Client receives and immediately dispatches action into Redux, updating the bouncing coin UI without needing to refetch all data.
