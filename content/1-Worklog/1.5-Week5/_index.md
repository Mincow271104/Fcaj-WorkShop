---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Implement the remaining features
* Plan the technology migration to AWS services

### Tasks to be implemented this week:
| Day | Task | Start Date | Completion Date | Resources |
| --- | --- | --- | --- | --- |
| Mon, Tue, Wed, Thu, Fri | - Integrate Artificial Intelligence and advanced features: <br> **1. Implemented features:** <br> + Sophisticated video moderation with AI: Deeply analyze title, description, and content to block subtle entertainment videos bypassing rules. <br> + User presence tracking: Facial recognition via WebCam, issue warnings and automatically cancel the timer if the user leaves the screen for too long. <br> + Automatic closing of entertainment apps: Scan the system and forcibly close independent social media or entertainment software installed on the computer. <br> + Hard discipline mode: Completely lock the pause or stop button, forcing focus until the time is up. <br> **2. Applied technologies:** <br> + AI & LLMs (Video classification): Local AI with Ollama and Cloud AI with Groq API. <br> + Computer Vision (Face Tracking): Google MediaPipe via WebAssembly. | 01/06/2026 | 05/06/2026 | |
| Sat, Sun | - Plan the migration to AWS services: <br> + Analyze the current architecture to map with appropriate AWS services (EC2, S3, API Gateway, DynamoDB...). <br> + Create a list of resources to be provisioned. | 06/06/2026 | 07/06/2026 | |


### Week 5 Achievements:
* Successfully integrated large language models (Local AI with Ollama and Cloud AI with Groq API) into the application to automatically classify and moderate YouTube video content with high accuracy.
* Successfully applied Computer Vision technology (Google MediaPipe running via WebAssembly) to track user presence (AFK Tracking) in real-time while ensuring computer performance.
* Fully deployed the Native App Killer mechanism, allowing the application to scan and automatically close entertainment processes (like Facebook, TikTok) running independently on the Windows operating system.
* Perfected the hard discipline feature (Hard Mode), forcing users to strictly adhere to the set focus time.
* Completed the architecture analysis and preliminary plan to migrate the application's storage and backend components to AWS cloud infrastructure for the next phase.
