---
title : "Future Directions"
date : 2026-07-17 
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

# Future Development Directions

With a flexible Serverless architecture on AWS and a Client application using ReactJS + Electron, Uchimi StudyGamification has established a solid core foundation. However, for the project to truly become a comprehensive and optimal educational ecosystem, the development team has identified areas requiring improvement and mapped out a strategic expansion roadmap for the future.

### 5.6.1. Areas for Improvement

Based on practical testing and operations, the project currently has a few limitations that need to be prioritized in upcoming versions:

*   **AI Scan Performance Optimization:** Due to the nature of the Electron app continuously running an AI module to scan user behavior/screens for distractions, the current application can consume significant hardware resources (CPU/RAM). It is necessary to research and optimize using Lightweight models or implement more efficient multithreading mechanisms on the Client side to prevent system lag when users study while running heavy applications.
*   **Perfecting Offline Mode:** Because the architecture relies entirely on AWS Serverless (API Gateway, Lambda, DynamoDB), the application is highly dependent on internet connectivity. We need to develop local caching mechanisms (Local Storage/IndexedDB) on Electron. Upon losing connection, study time and quest progress will be temporarily saved locally, then automatically and securely synced to the Cloud once the connection is restored.
*   **Enhanced Security and Anti-cheat:** Implement stricter technical barriers to prevent users from utilizing tools (auto-click, macros) to farm currency in Minigames or attempting to bypass the AI Scan system during a Focus Session.

---

### 5.6.2. Future Features and Expansion

In addition to overcoming current limitations, the project aims to expand into a comprehensive EdTech (Education Technology) ecosystem through the following upgrades:

#### 1. AI Upgrades
*   **Multimodal AI:** Upgrade the AI Assistant's capabilities to not only understand plain text but also recognize images, solve complex mathematical formulas, or extract content from lecture videos for user summarization.
*   **Personalized Learning Paths:** Apply Machine Learning to analyze users' focus time charts, enabling the AI to recommend the most effective study timeframes (Golden hours) for each individual.
*   **Voice Chat AI:** Integrate Speech-to-Text and Text-to-Speech technologies, turning the AI into a companion that supports practical foreign language communication practice directly within the app.

#### 2. Deepening the Gamification Ecosystem
*   **Minigame Expansion:** Add highly educational puzzle games such as Crosswords, Sudoku, or interactive Flashcards. These levels will be logically designed to reuse the existing automated level generation module.
*   **Virtual Pet System:** Enhance the Gacha system to unlock "Pet Assistants". Pets will appear as overlays on the Focus Session screen, displaying happy/sad emotional states based on the user's habit maintenance progress (Streak), creating a deeper emotional connection.

#### 3. Social & Ed-Management
*   **Multiplayer Co-Study Rooms:** Allow users to create shared study rooms in real-time. Friends can turn on cameras/microphones and share a Pomodoro timer to increase peer pressure, helping each other progress. Room search functionality will be optimized by Algolia.
*   **Classroom Management System:** Expand the customer base to a B2B/B2C model by allowing Teachers or Tutors to create private "Study Rooms/Classrooms":
    *   Teachers will have the authority to manage the student roster, track detailed focus time charts (`strikeCount`, `rankScore`), and monitor the distraction levels of each individual.
    *   Teachers can set up common Study Plans for the entire class, assign Group Quests, and utilize the class's internal Currency/Leaderboard system to honor outstanding individuals.