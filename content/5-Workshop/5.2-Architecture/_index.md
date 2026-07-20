---
title: "System Architecture"
date: 2026-07-17 
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Analysis and System Architecture Design

After defining the product problem in section 5.1, the architecture design phase focuses on answering the question: *how to make a Desktop application with a smooth experience, while ensuring data integrity and preventing cheating on the Server side?* The Uchimi project is designed using a **distributed Client–Server** model, where the Frontend runs locally on the user's machine (ReactJS + Electron) and the Backend operates entirely on the **AWS Serverless** platform in the `ap-southeast-1` region.

#### Choosing the Client–Server Model: ReactJS/Electron and AWS Serverless

**Client side (ReactJS + Electron):** Instead of building a pure web app, the project chose Electron to package the React UI into an actual Desktop app — fitting the "Learning OS" concept (floating window, taskbar, Focus widget, syncing Chrome Extension). Amplify Auth integrates directly with Amazon Cognito, helping the Client receive JWTs and attach them to every API request without having to manually handle complex authentication logic. More importantly, the **Offline-First** architecture (Redux + Electron-store) allows users to open the app and instantly see data from the local cache, then the system silently syncs with the Cloud via the `POST /sync-all` API.

**Backend side (AWS Serverless):** The entire business logic (Gacha, Shop, Quest, Minigame, Social...) is deployed as independent AWS Lambda functions, triggered via HTTP API, S3 Events, DynamoDB Streams, or EventBridge Schedules. Reasons for choosing Serverless over traditional EC2/ECS:

* **Scale-to-Zero:** When there are no users, the system incurs almost no fixed maintenance costs.
* **Automatic scaling:** Lambda and API Gateway automatically scale based on load without manual Auto Scaling Group or Load Balancer configurations.
* **Infrastructure as Code:** The entire infrastructure is declared in `serverless.yml`, deploying with a single `serverless deploy` command → CloudFormation synchronously creates Lambda, API, IAM, and Logs.
* **Built-in integration:** Cognito, DynamoDB, S3, EventBridge, Streams — reducing platform setup time compared to installing them on a private server.

#### Limitations and Barriers of the Architecture

Besides the advantages, this model also poses several technical challenges that need intentional handling:

* **Lambda Cold Start:** The first request after an idle period might be slower because Lambda has to initialize the runtime. The project minimizes this by keeping the memory at 512 MB (the sweet spot between speed and cost).
* **Distributed Debugging:** Logic is scattered across 20 Lambda functions along with asynchronous triggers (Streams, S3, Cron) — end-to-end tracking requires properly configured CloudWatch Logs.
* **DynamoDB Design from scratch:** NoSQL does not support JOINs or complex full-text search. All queries must be designed around PK/SK and GSIs from the start; searching for users by name requires an external service (Algolia).
* **Offline-First and Race Conditions:** Client local data caching can easily cause conflicts during synchronization. The Backend must use `TransactWriteItems` for currency/inventory transactions and apply the Server Authority principle — not trusting the data sent by the Client.
* **Vendor Lock-in:** The architecture is tightly coupled with AWS (DynamoDB Streams, Cognito JWT Authorizer, IAM Role). Migrating to another cloud would incur significant refactoring costs.
* **Resource-heavy Electron:** Desktop apps consume more RAM than pure web apps; the distribution and version update process is more complex than web deployment.

#### Overall Architecture Diagram

The diagram below describes the main communication flow between the Client, AWS services, and third-party systems. All AWS resources are located in the `ap-southeast-1` region; Algolia and Gemini API are outside the AWS Cloud frame.

![AWS Architecture Diagram](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)

Main flows on the diagram:
1. Client logs in via **Cognito**, receives a JWT.
2. All API requests go through **API Gateway** (JWT Authorizer verifies token) → **Lambda** processes → reads/writes to **DynamoDB** or **S3**.
3. Changes in the User table → **DynamoDB Streams** → `streamIndexer` Lambda → syncs to **Algolia**.
4. **EventBridge** cron triggers the worker to refresh the Shop / Leaderboard periodically.
5. Client views images via **CloudFront** (CDN) pulling origin from **S3**.
6. **IAM Execution Role** (dashed line on the diagram) grants Lambda permissions to access DynamoDB, Streams, and S3 — this is the runtime authorization layer, not a data flow.

#### Core AWS Services

The project uses the Serverless Framework (Node.js 20.x) to automate infrastructure configuration deployment. The overall structure includes **20 Lambda functions**, **25 HTTP routes**, **2 EventBridge schedules**, **3 S3 triggers**, **1 DynamoDB Stream trigger**, and **1 Cognito PostConfirmation trigger**.

| Service | Purpose | Configuration Details |
| :--- | :--- | :--- |
| **AWS Lambda** | Processes all Business Logic | Divided into 10 modules: User, Session, Gacha, Minigame, Quest, Shop, Social, Upload, Currency, Sync. |
| **API Gateway** | HTTP API communication port for Client | 25 routes (GET/POST/PUT), JWT Authorizer from Cognito, Throttling 50 req/s and 20 concurrent. |
| **Amazon Cognito** | User authentication and identity | User Pool + JWT; PostConfirmation trigger automatically initializes profile (`handleInitUser`). |
| **Amazon DynamoDB** | NoSQL Database | 8 tables: User, Study, Social, Quest, Minigame, ItemData, Inventory, GachaHistory. |
| **DynamoDB Streams** | Captures real-time data change events | Triggers the `streamIndexer` Lambda on the User table → syncs with Algolia. |
| **Amazon S3** | Stores static resources | `ASSETS_BUCKET` bucket: `public-assets/`, `avatars/`, `uploads/`; triggers Lambda when uploading `.zip`/`.json`. |
| **Amazon CloudFront** | CDN distributing images/assets | Origin points to S3; Client views avatars/items via CDN URL instead of calling S3 directly. |
| **Amazon EventBridge** | Automated task scheduling (Cron) | Updates Leaderboard every 10 minutes; refreshes Shop weekly (Sunday 17:00 UTC). |
| **Amazon CloudWatch** | Monitoring and Logs | Automatically creates a Log Group for each Lambda; supports debugging and performance tracking. |
| **AWS IAM** | Access rights management | Execution Role attached to Lambda; grants least-privilege permissions via `serverless.yml`. |
| **AWS CloudFormation** | Provisions infrastructure (via Serverless) | All resources are grouped into a single Stack; rollback/clean up with one action. |

> **Note on Region/AZ:** The system does not use VPC or EC2, so there is no need to configure Availability Zones manually. AWS manages multi-AZ automatically for managed services (Lambda, DynamoDB, S3) within the `ap-southeast-1` region.

#### Third-party Service Integration

Besides the AWS ecosystem, the project integrates two external services to add capabilities that AWS does not readily provide at a simple level:

**Algolia (User Search Index):** DynamoDB does not support full-text search by username. Algolia was chosen over OpenSearch because of its simple API, near real-time results, and no need to operate a separate cluster — fitting the scale of an internship project. Sync flow: whenever the User table changes, DynamoDB Streams triggers the `streamIndexer` Lambda, which indexes the document to Algolia. The Client calls `GET /friends/search` → Lambda queries Algolia and returns the results.

**Google Gemini API (AI Study Planner):** Supports generating study paths and quizzes from user-uploaded documents. However, the system does not force the use of Cloud AI — users can choose **Ollama (Local)** to process entirely offline on their personal machine (details in section 5.3). Gemini is only used when users manually configure their API Key, helping the app avoid centralized LLM costs.

#### Communication Flows Between Components

The system operates based on three main types of flows:

**Synchronous flow (Request/Response):** This is the most common flow, making up the majority of the 25 HTTP routes. The Client sends a request with a JWT → API Gateway verifies the token via Cognito Authorizer → invokes the corresponding Lambda → Lambda reads/writes to DynamoDB or S3 → returns JSON to the Client. Examples: buying Shop items, claiming quests, changing cosmetics, syncing profiles.

**Asynchronous flow (Event-Driven):** Events generated from data changes or schedules, without the Client actively calling:
* **DynamoDB Streams → streamIndexer → Algolia:** Syncs the search index when the User table changes.
* **S3 ObjectCreated → processZip / processJson:** Dev uploads an item `.zip` file or config `.json` → Lambda automatically extracts it, pushes assets to S3, and writes metadata to the ItemData table.
* **EventBridge Cron → Shop/Leaderboard worker:** Automatically rotates the shop weekly, updates the leaderboard every 10 minutes.
* **Cognito PostConfirmation → handleInitUser:** Automatically initializes a profile and default inventory when a user first registers.

**Resource Delivery Flow (CDN):** The Client does not call APIs to view avatar images or item assets. Instead, the Client accesses the CloudFront URL directly → CloudFront fetches the original file from S3 (origin) if it is not already in the cache. Lambda is only involved when a presigned URL is needed for uploading or processing new files.

#### Security Configuration and Access Rights (IAM Permissions)

The **Least Privilege** principle is strictly applied through the `serverless.yml` file. The Lambda Execution Role is only granted the exact Actions on the exact Resources needed:

* **DynamoDB:** `GetItem`, `PutItem`, `UpdateItem`, `DeleteItem`, `Query`, `Scan`, `BatchGetItem`, `BatchWriteItem`, `TransactWriteItems` — limited to the 8 system tables and their corresponding GSIs.
* **DynamoDB Streams:** `DescribeStream`, `GetRecords`, `GetShardIterator`, `ListStreams` — only on the ARN of the User Table Stream.
* **Amazon S3:** `GetObject`, `PutObject` — only within the `public-assets/*`, `avatars/*`, `uploads/*` prefixes.

At the API layer, every sensitive route is attached with the `myCognitoAuth` (JWT) authorizer. API Gateway checks the token with the Cognito User Pool before allowing Lambda invocation — the Client cannot fake the user_id. CORS is enabled on HTTP APIs; Throttling limits 50 requests/second and 20 concurrent requests to prevent light spam/DDoS.

Two IAM layers need to be distinguished during operation:
* **Execution Role** (in the diagram): the runtime permissions of Lambda while it is running — declared in `provider.iam.role.statements`.
* **Deploy Role** (of the developer): permissions to create/update CloudFormation, Lambda, API Gateway — used when running `serverless deploy`, not shown on the architecture diagram.

Sensitive environment variables (`ALGOLIA_WRITE_KEY`, `COGNITO_USER_POOL_ID`, `COGNITO_CLIENT_ID`...) are loaded via `useDotenv: true` from the local `.env` file — they are not committed to GitHub.

#### Infrastructure Setup and Deployment Process

Bringing the Backend to AWS is done in these steps:

1. **Prepare foundational resources:** Create Cognito User Pool, 8 DynamoDB tables (enable Stream on the User table), S3 bucket, and (optionally) CloudFront distribution before deploying Lambda.
2. **Configure `.env`:** Fill in table names, buckets, Cognito IDs, Stream ARNs, and Algolia credentials in `AWSServerless/.env`.
3. **Declare IaC:** All Lambdas, API routes, IAM, and triggers are defined in `serverless.yml` and child `function.yml` files (user, shop, gacha, social, upload...).
4. **Deploy with one command:**

```bash
cd AWSServerless
serverless deploy
```

Serverless Framework automatically compiles the code (esbuild bundle + minify), generates the CloudFormation template, and provisions resources synchronously in the `ap-southeast-1` region.

5. **Connect Frontend:** Update `AWSStudy-Play/.env` to point `VITE_API_URL`, `VITE_COGNITO_*`, and `VITE_S3_ASSETS_URL` to the actual endpoints after deployment.

#### Lambda Module Separation and Trigger Mechanism

The Backend is divided into 10 independent modules, each having its own `function.yml` file declaring the handler and trigger type:

| Module | Trigger | Main Functions |
| :--- | :--- | :--- |
| **userFunction** | Cognito PostConfirmation + HTTP API | Initialize profile; update name; equip cosmetics; upload avatar |
| **sessionFunction** | HTTP API | Manage study sessions / Focus sessions |
| **currencyFunction** | HTTP API | Convert Knowledge Point → Knowledge Core |
| **gachaFunction** | HTTP API | Roll gacha server-side, handle pity, record Inventory + GachaHistory |
| **minigameFunction** | HTTP API + EventBridge Schedule | Sudoku/Minesweeper; periodic leaderboard worker |
| **questFunction** | HTTP API | Daily quests, claim rewards, update streaks |
| **shopFunction** | HTTP API + EventBridge Schedule (cron) | Retrieve shop, purchase items with eCoins; weekly shop rotation |
| **socialFunction** | HTTP API + DynamoDB Stream | Friends management, add friend; Algolia search; `streamIndexer` |
| **syncFunction** | HTTP API | `sync-all` / sync profile-inventory for offline-first |
| **uploadFunction** | S3 ObjectCreated (`.zip` / `.json`) | Ingest item master data + assets into S3 and ItemData table |

The process of pushing new items into the system (for example, the Study Plant background) also follows an event-driven flow: The developer packages the item into a ZIP file (including `data.json` + assets) → uploads it to `s3://ASSETS_BUCKET/uploads/items/` → S3 triggers the `processZip` Lambda → Lambda extracts the ZIP, pushes files to `public-assets/items/`, and writes metadata to the ItemData table. If the item has `collectFrom: "eCoinShop"`, the shop worker will automatically pick up that item during the weekly shop rotation.