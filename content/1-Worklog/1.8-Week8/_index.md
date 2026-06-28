---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:

* Add Daily Quests feature to the project
* Plan the AI Study Planner feature

### Tasks to be implemented this week:
| Day | Task | Start Date | Completion Date | Resources |
| --- | --- | --- | --- | --- |
| Mon, Tue, Wed, Thu | **Add Daily Quests feature to the project:** <br>- UI/UX Design & Development: QuestPanel interface (quest list, progress bar, claim reward animation). <br>- State Management: Integrate Redux for quest list, current progress, and reward status. <br>- System Logic Setup: Progress Tracker algorithm automatically updating progress based on Focus/Login actions. <br>- Reset Mechanism: 24h automatic refresh logic and real-time countdown timer. <br>- Storage & Sync: Electron IPC communication for local storage and cloud synchronization. | 22/06/2026 | 25/06/2026 | |
| Fri, Sat | **Plan the AI Study Planner feature:** <br>- Requirements Analysis: Define core user flows including Consultation Chat, Generate Plan, and Generate Quiz. <br>- Technology Evaluation: Survey LLM APIs (Groq/OpenAI), prepare sample Prompt Engineering templates. <br>- Architecture Design: Database/storage design for chat history, JSON Study Plan schema, and Quiz DB. <br>- Module Drafting: Define components to write (Chat UI, Plan Viewer, Quiz Modal) and Main/Renderer IPC specifications. <br>- Implementation Planning: Task Breakdown, time estimation, and milestones. | 26/06/2026 | 27/06/2026 | |

### Week 8 Achievements:

* **Daily Quests Feature:**
  * Completed design and successfully built the engaging QuestPanel UI with smooth progress bars and reward claiming effects.
  * Successfully integrated Redux for centralized state management, accompanied by the Progress Tracker system automated by Focus/Login events.
  * Finalized the 24h auto-reset mechanism and secure data storage/sync using Electron IPC to Local/Cloud.

* **AI Study Planner Planning:**
  * Defined the primary business flows of AI Study Planner: study advisory, automated plan generation, and review quiz generation.
  * Evaluated and selected the appropriate LLM solution (Groq/OpenAI APIs) to meet response speed requirements, and completed sample prompt templates.
  * Designed data schemas (chat history, study plan JSON schema, quiz question bank) and defined IPC channel specifications between Main and Renderer processes.
  * Decomposed work items (Task Breakdown) and estimated implementation time to prepare for coding next week.
