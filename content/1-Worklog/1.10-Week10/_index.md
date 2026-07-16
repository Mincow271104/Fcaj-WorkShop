---
title: "Week 10 Worklog"
date: 2026-7-6
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---


### Week 10 Objectives:

* Complete and optimize the Focus Mode feature combined with Browser Extension and AI system.
* Successfully integrate AI models on the AWS Bedrock platform (Nova Micro, Nova Lite) to power YouTube Blocker and StudyPlanner.
* Enhance user experience (UX/UI) and thoroughly handle API rate-limiting edge cases.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | - Integrated Amazon Bedrock AI (Nova Micro, Nova Lite) for the application <br> - Implemented auto-retry mechanism (handling 429, 503 limits) <br> - Improved Focus Mode UI (separated Casual/Rank modes, added Custom time limits) <br> - Displayed detailed Extension and AI status <br> - Cleaned up Settings UI | 07/06/2026 | 07/06/2026      | |
| 3   | - Update and optimize subsequent modules...  | 07/07/2026 | 07/07/2026      | |
| 4   | - Update and optimize subsequent modules...  | 07/08/2026 | 07/08/2026      | |
| 5   | - Update and optimize subsequent modules...  | 07/09/2026 | 07/09/2026      | |
| 6   | - Finalize and test the application...       | 07/10/2026 | 07/10/2026      | |


### Week 10 Achievements:

* Successfully integrated Amazon Bedrock AI using Nova Micro and Nova Lite models as robust alternatives to Gemini.
* Built a stable API calling flow with an intelligent auto-retry mechanism when the server is overloaded (503) or quota exceeded (429).
* Re-designed Focus Mode interface to be more user-friendly (clear division of Casual and Rank modes, preventing overly short durations).
* Provided transparent system status for users (showing which browsers the Extension has detected, and which provider's AI model is active).
* Optimized Settings UI by removing unnecessary configurations (like FaceTracking LLM).
