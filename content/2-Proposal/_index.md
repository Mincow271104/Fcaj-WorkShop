---
title: "Proposal"
date: 2026-07-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Uchimi StudyGamification - AI and Gamification Learning App


### 1. Project Overview
The project is designed as a desktop learning application combining AI technology and a Gamification ecosystem. The goal is to turn the act of "opening the computer to study every day" into a natural, voluntary, and sustainable habit. Through the application of the behavioral loop: **Discipline → Action → Feedback → Reward**, the system helps learners maintain deep focus while enjoying healthy entertainment after intense study sessions.

### 2. Problem Statement
The current self-study process faces major psychological and motivational barriers:
* **Lack of instant feedback:** Learning results appear slowly, whereas social media or games provide immediate dopamine hits, making learners easily distracted and prone to giving up.
* **Dry traditional tools:** Current learning software primarily focuses on task management and fails to solve the problem of maintaining long-term motivation.
* **Superficial gamification:** If focusing solely on pure entertainment rewards, users easily get sucked into the game and forget the actual learning outcomes.
* **Digital cheating:** Desktop apps are easily manipulated (editing study time, hacking minigame scores, duplicating items), compromising the fairness of the environment.

### 3. Project Objectives
The project aims to build a learning ecosystem that balances **learning discipline** and **healthy entertainment**:
* **Transforming large goals:** Breaking down long-term study paths into focused sessions (Focus mode) and achievable daily quests.
* **Visualizing progress:** Providing timely feedback through a scoring system (Knowledge Points), study streaks, pet items, and leaderboards.
* **Applying AI for personalization:** Supporting study plan generation, creating quizzes from documents, and monitoring distractions via AI Scan technology.
* **Ensuring fairness and transparency:** Building an anti-cheat backend system to protect the integrity of currency, items, and ranking data.

### 4. Solution Architecture
The project uses a **distributed Client-Server** model, combining the convenience of a Desktop app with the flexible scalability of the Cloud.

*   **Client (Frontend):** Built with ReactJS and packaged with Electron into a standalone Desktop app. State management and local caching (Local Caching) use Redux and Electron-store for fast responsiveness.
*   **AI & Tracking:** Uses a Browser Extension (AIGuard) to communicate via WebSocket with the Desktop App. Integrates AWS Bedrock (STS cross-account), Google Gemini (Cloud AI), or Ollama (Local AI) for content classification and study support features.
*   **Backend (AWS Serverless):** All Business Logic (Gacha, Shop, Quest, Social) is handled via AWS Lambda and API Gateway in the **ap-southeast-1** region. 
*   **Storage & Data:** Amazon DynamoDB stores system data (using **TransactWriteItems** to ensure integrity). Amazon S3 stores static resources (assets, avatars), and CloudFront serves as the CDN.
*   **Ancillary Services:** Amazon Cognito (Authentication/JWT), EventBridge (Cronjobs for leaderboards/shops), DynamoDB Streams, and Algolia (User search).

### 5. Deployment Timeline
*   **Phase 1 (Infrastructure Setup):** Initialize AWS resources (Cognito, DynamoDB, S3), configure API Gateway, and deploy the basic Lambda framework using the Serverless Framework.
*   **Phase 2 (Client Development):** Build the ReactJS/Electron interface, integrate Focus Mode, the Daily Quest system, and the Local Caching / Resource Preloading mechanism.
*   **Phase 3 (AI Integration):** Develop the AIGuard Extension, set up WebSockets, and connect the Gemini API/Ollama to generate study plans and quizzes.
*   **Phase 4 (Gamification Ecosystem):** Finalize Minigame, Gacha, Shop, and Leaderboard logic on the Cloud. Apply Anti-cheat mechanisms.
*   **Phase 5 (Packaging & Operations):** Build the **.exe** file using **electron-builder** and distribute it via Google Drive. Conduct end-to-end testing and monitor via CloudWatch.

### 6. Budget and Cost Optimization
The architecture is designed around **Cost-optimization** thinking:
*   **Scale-to-Zero:** The use of AWS Lambda and API Gateway helps the system incur almost no fixed server costs when there are no active users.
*   **Flexible AI options:** Users can utilize AWS Bedrock, configure their own API Key (Gemini), or run a Local LLM (Ollama) directly on their machine to protect personal data privacy and save Cloud API token costs, optimizing operational costs for the platform.
*   **Database operation optimization:** Applying internal caching on the Client and a bulk sync mechanism (**POST /sync-all**) to reduce API calls and read/write operations on DynamoDB.

### 7. Risk Assessment and Mitigation

| Risk Type | Problem Description | Mitigation Strategy |
| :--- | :--- | :--- |
| **System Latency** | AWS Lambda Cold Start phenomenon slowing down the first request. | Keep Lambda RAM configuration at an optimal level (512MB) to balance speed and cost. |
| **Data Integrity** | Client cache can easily cause data conflicts or user cheating. | The Backend always acts as the Server Authority; use **TransactWriteItems** for critical transactions. |
| **Resource Consumption** | Electron and AI (Local) can consume significant computer RAM. | Optimize Vite Build (tree-shaking) and allow users the flexibility to choose between Cloud AI and Local AI. |
| **Vendor Lock-in** | Deep reliance on the AWS ecosystem (Cognito, DynamoDB, Streams). | Package business logic into independent Lambda modules and standardize connection protocols. |