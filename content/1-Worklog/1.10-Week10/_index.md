---
title: "Week 10 Worklog"
date: 2026-07-06
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:

* Complete and optimize the Focus Mode feature combined with Browser Extension and AI system.
* Successfully integrate AI models on the AWS Bedrock platform (Nova Micro, Nova Lite) to power YouTube Blocker and StudyPlanner.
* Enhance user experience (UX/UI) and thoroughly handle API rate-limiting edge cases.

### Tasks to be carried out this week:
| Day | Task                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| Mon | - Configure IAM Roles and Policies across 2 AWS accounts to establish Cross-Account access for Amazon Bedrock <br> - Integrate Amazon Bedrock AI (Nova Micro, Nova Lite) as an alternative or alongside Gemini | 07/06/2026   | 07/06/2026      | |
| Tue | - Implement a smart auto-retry mechanism to handle API overload errors (429, 503) for stability | 07/07/2026   | 07/07/2026      | |
| Wed | - Improve Focus Mode UI by separating Casual and Rank modes clearly, and add Custom time limit settings | 07/08/2026   | 07/08/2026      | |
| Thu | - Develop a transparent status display for the Browser Extension and the currently active AI model | 07/09/2026   | 07/09/2026      | |
| Fri | - Clean up Settings UI by removing unnecessary configurations (like FaceTracking) | 07/10/2026   | 07/10/2026      | |
| Sat | - Perform end-to-end system testing, optimize performance and fix bugs | 07/11/2026 | 07/11/2026 | |

### Week 10 Achievements:

* Successfully established Cross-Account connection between 2 AWS accounts, enabling secure Bedrock API calls.
* Successfully integrated Amazon Bedrock AI using Nova Micro and Nova Lite models as robust alternatives to Gemini.
* Built a stable API calling flow with an intelligent auto-retry mechanism when the server is overloaded (503) or quota exceeded (429).
* Re-designed Focus Mode interface to be more user-friendly (clear division of Casual and Rank modes, preventing overly short durations).
* Provided transparent system status for users (showing which browsers the Extension has detected, and which provider's AI model is active).
* Optimized Settings UI by removing unnecessary configurations (like FaceTracking LLM).
