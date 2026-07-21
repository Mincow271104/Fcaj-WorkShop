---
title : "System Architecture"
date : 2026-07-17 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

# System Architecture Analysis and Design

After defining the product problem in section 5.1, the architecture design phase focuses on one question: *how can a Desktop app deliver a smooth experience while still ensuring data integrity and anti-cheat controls on the Server?* The project follows a **distributed Client–Server** model: the Frontend runs locally on the user’s machine (ReactJS + Electron), and the Backend runs fully on **AWS Serverless** in region **ap-southeast-1**.

#### Choosing Client–Server: ReactJS/Electron and AWS Serverless

**Client side (ReactJS + Electron):** Instead of a pure web app, the project packages the React UI as a real Desktop application with Electron — fitting the “study OS” concept (floating windows, taskbar, Focus widgets). On the diagram, the CLIENT block includes the **Electron App** and **Focus Guard** (Chrome Extension); the two components sync focus state over an internal **WebSocket (WS)** channel. Amplify Auth integrates directly with Amazon Cognito so the Client receives a JWT and attaches it to every API request without custom auth logic. More importantly, the **Local Caching** mechanism (Redux + Electron-store) lets users open the app and see UI assets and data immediately from the cache, optimizing response speed before the system quietly syncs latest data with the Cloud via **POST /sync-all**.

**Backend side (AWS Serverless):** All business logic (Gacha, Shop, Quest, Minigame, Social, etc.) is implemented as independent AWS Lambda functions, triggered by HTTP API, S3 Events, DynamoDB Streams, or EventBridge Schedules. Reasons for choosing Serverless over traditional EC2/ECS:

* **Scale-to-Zero:** When there are no users, the system incurs almost no fixed idle cost — suitable for development and workshop demos.
* **Automatic scaling:** Lambda and API Gateway scale with load without manually configuring Auto Scaling Groups or Load Balancers.
* **Infrastructure as Code:** The full stack is declared in **serverless.yml**; one **serverless deploy** lets CloudFormation provision Lambda, API, IAM, and Logs together.
* **Built-in integrations:** Cognito, DynamoDB, S3, EventBridge, and Streams reduce platform setup time compared with self-managed servers.

#### Architecture limitations and trade-offs

Alongside the benefits, this model introduces technical challenges that must be handled deliberately:

* **Lambda cold start:** The first request after idle time can be slower while the runtime initializes. The project mitigates this by keeping memory at 512 MB (a practical balance of speed and cost).
* **Distributed debugging:** Logic is spread across the Lambda functions shown on the diagram as **36 fn**, plus async triggers (Streams, S3, Cron) — end-to-end tracing depends on correctly configured CloudWatch Logs.
* **DynamoDB design upfront:** NoSQL does not support JOINs or complex full-text search. Every query must be designed around PK/SK and GSIs from the start; username search relies on an external service (Algolia).
* **Cache Synchronization and race conditions:** Local Client caches can conflict during rapid operations. The Backend must use **TransactWriteItems** for currency/inventory transactions and apply Server Authority — never trust Client-reported balances.
* **Vendor lock-in:** The architecture is tightly coupled to AWS (DynamoDB Streams, Cognito JWT Authorizer, IAM Roles). Moving to another cloud would require significant refactoring.
* **Electron resource cost:** The Desktop app uses more RAM than a pure web app; distribution and version updates are more complex than a web deploy.

#### Overall architecture diagram

The diagram below shows the main communication paths among the Client, AWS services, and third-party systems. Inside **AWS Cloud — ap-southeast-1** there are four groups: AUTH & SECURITY, API & COMPUTE, DATA, and ASYNC; **CloudFront** serves images from S3. Outside the main account box are **ACCOUNT B — BEDROCK** (cross-account via STS) and the **EXTERNAL** block (Algolia, Gemini).

![AWS architecture diagram](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)

Main flows on the diagram (numbered 1–10):
1. **Login:** The Electron App signs in through **Cognito User + IdP** and receives a JWT.
2. **API (JWT):** The Client calls **HTTP API + JWT**; API Gateway verifies the token before requests reach the Backend.
3. **Invoke:** The HTTP API invokes **Lambda (36 fn)** to run business logic.
4. **R/W:** Lambda reads/writes **DynamoDB + Streams**.
5. **Assets:** Lambda reads/writes **S3 Assets** (uploads, avatars, public assets, etc.).
6. **Index:** DynamoDB changes (via Streams) are pushed to **Algolia** to keep the search index in sync (implemented through the **streamIndexer** Lambda).
7. **Cron:** **EventBridge (cron ×2)** triggers Lambda workers on a schedule (Shop / Leaderboard); CloudWatch sits in the same ASYNC group for monitoring.
8. **AI Bedrock:** The Client calls **Amazon Bedrock** in **ACCOUNT B** using **STS from the app** (cross-account), bypassing the main account’s API Gateway.
9. **Gemini:** The Client calls the **Google Gemini API** directly (EXTERNAL block) when the user configures an API Key.
10. **Image CDN:** The Client loads images through **CloudFront (image CDN)**; CloudFront fetches origin objects from **S3 Assets**.

> **IAM Exec Role** (dashed line on the diagram, not numbered): grants Lambda runtime access to DynamoDB, Streams, and S3 — a permission layer, not a data flow.

Original Draw.io file: **docs/Uchimi StudyGamification.drawio** (tab *Uchimi Current Architecture*)

#### Core AWS services

The project uses Serverless Framework (Node.js 20.x) to provision infrastructure automatically. Per the architecture diagram, the compute layer is **Lambda (36 fn)**, together with **25 HTTP routes**, **2 EventBridge schedules** (cron ×2), **3 S3 triggers**, **1 DynamoDB Stream trigger**, and **1 Cognito PostConfirmation trigger**.

| Service | Purpose | Configuration details |
| :--- | :--- | :--- |
| **AWS Lambda** | Runs all business logic | API & COMPUTE group on the diagram: **36 fn**, organized into 10 modules (User, Session, Gacha, Minigame, Quest, Shop, Social, Upload, Currency, Sync). |
| **API Gateway** | HTTP API + JWT entry point for the Client | 25 routes (GET/POST/PUT), Cognito JWT Authorizer, throttling at 50 req/s and 20 concurrent. |
| **Amazon Cognito** | Authentication and identity (AUTH & SECURITY) | User Pool + IdP + JWT; PostConfirmation trigger initializes the profile (**handleInitUser**). |
| **Amazon DynamoDB** | NoSQL database (DATA) | 8 tables: User, Study, Social, Quest, Minigame, ItemData, Inventory, GachaHistory; with Streams enabled. |
| **DynamoDB Streams** | Real-time change capture | Triggers the **streamIndexer** Lambda on the User table → syncs Algolia (step 6. Index). |
| **Amazon S3** | Static asset storage (S3 Assets) | Bucket **ASSETS_BUCKET**: **public-assets/**, **avatars/**, **uploads/**; triggers Lambda on **.zip** / **.json** uploads. |
| **Amazon CloudFront** | CDN for images/assets | Origin points to S3; Client loads avatars/items via CDN URLs (step 10 on the diagram). |
| **Amazon EventBridge** | Scheduled jobs (ASYNC — cron ×2) | Leaderboard refresh every 10 minutes; weekly Shop rotation (Sunday 17:00 UTC). |
| **Amazon CloudWatch** | Monitoring and logs (ASYNC) | Auto-created Log Groups per Lambda; supports debugging and performance tracking. |
| **Amazon Bedrock** | AI models via a secondary account | Lives in **ACCOUNT B — BEDROCK**; the Client assumes a role with **STS from the app** (step 8), not through main-account Lambdas. |
| **AWS IAM** | Access control | Execution Role attached to Lambda (dashed line on the diagram); least-privilege statements in serverless.yml. |
| **AWS CloudFormation** | Infrastructure provisioning (via Serverless) | All resources in one Stack; rollback/cleanup in a single operation. |

> **Note on Region/AZ:** The system does not use VPC or EC2, so Availability Zones are not configured manually. AWS manages multi-AZ for managed services (Lambda, DynamoDB, S3) inside region ap-southeast-1.

#### AI and third-party integrations

On the diagram, integrations outside the main API path are the **EXTERNAL** block (Algolia, Gemini) and **ACCOUNT B — BEDROCK**:

**Algolia (User Search Index — EXTERNAL):** DynamoDB does not support full-text search by username. Algolia was chosen over OpenSearch for a simple API, near-realtime results, and no self-managed cluster. On the diagram this is step **6. Index**: when the User table changes, DynamoDB Streams trigger **streamIndexer** → index documents in Algolia. The Client calls **GET /friends/search** → Lambda queries Algolia and returns results.

**Amazon Bedrock (ACCOUNT B — STS from app):** The Client calls Bedrock directly in the secondary account (step **8. AI Bedrock**), using **STS** for temporary credentials — separate from the main Serverless stack. This path does not go through the **ap-southeast-1** HTTP API / Lambda.

**Google Gemini API (EXTERNAL):** The Client calls Gemini directly (step **9. Gemini**) to generate study plans and quizzes from uploaded materials. Cloud AI is optional — users can choose **Ollama (Local)** for local execution to protect personal data privacy and eliminate Cloud API token costs (details in section 5.3). Gemini is used only when the user configures their own API Key, so the app does not carry centralized LLM cost.

#### Communication flows between components

The system runs four flow groups, matching the diagram:

**Synchronous flow (Request/Response — steps 1→5):** Client signs in with Cognito and receives a JWT → sends the JWT to the HTTP API → API Gateway verifies → invokes Lambda → Lambda reads/writes DynamoDB or S3 → returns JSON to the Client. Examples: Shop purchases, quest claims, cosmetic changes, profile sync.

**Asynchronous flow (Event-Driven — steps 6→7 and related triggers):**
* **DynamoDB Streams → streamIndexer → Algolia (6. Index):** Keeps the search index in sync when the User table changes.
* **EventBridge Cron → Shop/Leaderboard workers (7. Cron):** Weekly shop rotation and leaderboard updates every 10 minutes; CloudWatch collects logs/metrics.
* **S3 ObjectCreated → processZip / processJson:** A developer uploads a **.zip** item pack or **.json** config → Lambda unpacks it, pushes assets to S3, and writes metadata to the ItemData table.
* **Cognito PostConfirmation → handleInitUser:** Creates the default profile and inventory on first registration.

**Client AI flows (steps 8→9):** The Electron App calls **Bedrock (Account B, STS)** or **Gemini (EXTERNAL)** directly — not through the main stack’s API Gateway. Users can also switch to local Ollama (section 5.3).

**Asset distribution flow (CDN — step 10):** The Client does not call the API to view avatar or item images. It hits CloudFront URLs directly → CloudFront fetches from S3 Assets when needed. Lambda is involved only for presigned upload URLs or processing newly uploaded files.

#### Security configuration and IAM permissions

**Least Privilege** is applied strictly through **serverless.yml**. The Lambda Execution Role receives only the Actions it needs on the required Resources:

* **DynamoDB:** **GetItem**, **PutItem**, **UpdateItem**, **DeleteItem**, **Query**, **Scan**, **BatchGetItem**, **BatchWriteItem**, **TransactWriteItems** — limited to the 8 system tables and related GSIs.
* **DynamoDB Streams:** **DescribeStream**, **GetRecords**, **GetShardIterator**, **ListStreams** — only on the User Table Stream ARN.
* **Amazon S3:** **GetObject**, **PutObject** — only under prefixes **public-assets/**, **avatars/**, **uploads/**.

At the API layer, sensitive routes use the **myCognitoAuth** (JWT) authorizer. API Gateway validates the token against the Cognito User Pool before invoking Lambda — the Client cannot spoof **user_id**. CORS is enabled on the HTTP API; throttling is set to 50 requests/second and 20 concurrent requests to limit light spam/DDoS.

Two IAM layers must be distinguished in operations:
* **Execution Role** (on the diagram): Lambda runtime permissions — declared in **provider.iam.role.statements**.
* **Deploy Role** (developer): permissions to create/update CloudFormation, Lambda, and API Gateway — used for **serverless deploy**, and not shown on the architecture diagram.

Sensitive environment variables (**ALGOLIA_WRITE_KEY**, **COGNITO_USER_POOL_ID**, **COGNITO_CLIENT_ID**, etc.) are loaded via **useDotenv: true** from a local **.env** file — never committed to GitHub.

#### Infrastructure setup and deployment process

Bringing the Backend to AWS follows these steps:

1. **Prepare base resources:** Create the Cognito User Pool, 8 DynamoDB tables (enable Streams on the User table), S3 bucket, and (optionally) a CloudFront distribution before deploying Lambdas.
2. **Configure .env file:** Fill in table names, bucket, Cognito IDs, Stream ARN, and Algolia credentials in **AWSServerless/.env**.
3. **Declare IaC:** All Lambdas, API routes, IAM policies, and triggers are defined in **serverless.yml** and per-module **function.yml** files (user, shop, gacha, social, upload, etc.).
4. **Deploy with one command:**

```bash
cd AWSServerless
serverless deploy
```

Serverless Framework compiles the code (esbuild bundle + minify), generates a CloudFormation template, and provisions resources in region **ap-southeast-1**.

5. **Connect the Frontend:** Update **AWSStudy-Play/.env** so **VITE_API_URL**, **VITE_COGNITO_\***, and **VITE_S3_ASSETS_URL** point to the deployed endpoints.

#### Lambda module split and trigger mechanisms

The Backend is split into 10 independent modules; each has its own **function.yml** declaring handlers and trigger types:

| Module | Trigger | Main responsibilities |
| :--- | :--- | :--- |
| **userFunction** | Cognito PostConfirmation + HTTP API | Initialize profile; update name; equip cosmetics; upload avatar |
| **sessionFunction** | HTTP API | Manage study / Focus sessions |
| **currencyFunction** | HTTP API | Convert Knowledge Point → Knowledge Core |
| **gachaFunction** | HTTP API | Server-side gacha rolls, pity, write Inventory + GachaHistory |
| **minigameFunction** | HTTP API + EventBridge Schedule | Sudoku/Minesweeper; periodic leaderboard worker |
| **questFunction** | HTTP API | Daily quests, claim rewards, update streaks |
| **shopFunction** | HTTP API + EventBridge Schedule (cron) | Fetch shop, buy items with eCoin; weekly shop rotation |
| **socialFunction** | HTTP API + DynamoDB Stream | Friends, friend requests; Algolia search; **streamIndexer** |
| **syncFunction** | HTTP API | **sync-all** / profile-inventory sync and Client cache refresh |
| **uploadFunction** | S3 ObjectCreated (file **.zip** / **.json**) | Ingest item master data + assets into S3 and the ItemData table |

Pushing a new item into the system (for example a Study Plant background) also follows the event-driven path: the developer packages the item as a ZIP (**data.json** + assets) → uploads to **s3://ASSETS_BUCKET/uploads/items/** → S3 triggers **processZip** → Lambda unpacks the ZIP, pushes files to **public-assets/items/**, and writes metadata to ItemData. If the item has **collectFrom: "eCoinShop"**, the shop worker picks it up during the weekly shop rotation.
