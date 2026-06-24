---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

* Modify the project's outputs according to the team's new database schema
* Optimize how the project utilizes AI
* Upgrade the AI Block feature for websites

### Tasks to be implemented this week:
| Day | Task | Start Date | Completion Date | Resources |
| --- | --- | --- | --- | --- |
| Mon, Tue | - Restructure client input and output data to be fully compatible with the new Amazon DynamoDB database schema. <br>- Test data correctness during synchronization via API Gateway. | 15/06/2026 | 16/06/2026 | |
| Wed, Thu, Fri, Sat | - Optimize the performance of calling AI classification models, building a caching mechanism to minimize requests to Cloud API (Groq) and Local AI (Ollama). <br>- Research and integrate smart AI filtering features to automatically analyze and block unidentified web URLs outside the static list. | 17/06/2026 | 20/06/2026 | |

### Week 7 Achievements:
* Completed restructuring of client Input/Output data, enabling smooth user profile, quest, and study session history synchronization with the new Amazon DynamoDB database.
* Successfully optimized the operation of the AI content classification model, reducing unnecessary API requests and significantly improving system response times.
* Successfully developed an advanced website filtering feature (AI Block) to automatically recognize and decide to block or allow access to unfamiliar pages (unidentified web URLs) based on content context analysis.
