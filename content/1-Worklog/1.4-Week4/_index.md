---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Implement the first feature of the project.
* Go to the office, attend the Event.

### Tasks to be implemented this week:
| Day | Task | Start Date | Completion Date | Resources |
| --- | --- | --- | --- | --- |
| Mon, Tue | - Plan to build features for the project: <br> + Use normal technologies first, then switch to AWS services later. | 25/05/2026 | 26/05/2026 | |
| Wed, Thu, Fri, Sat, Sun | - Start building core features and basic blocking mechanisms: <br> **1. Implemented features:** <br> + Centralized session management: Create a timer and manual block mode. <br> + Social media website blocking: Show a full-screen lock overlay when accessing social media sites during focus hours. <br> + Basic YouTube video filtering: Extract internal video categories for quick classification, allowing custom exemption lists and real-time settings synchronization. <br> + Anti-bypass protection: Scan open browsers to prevent users from intentionally disabling or not installing the extension. <br> **2. Applied technologies:** <br> + Desktop App: Electron, Node.js, WebSocket, Electron Store (for Backend) and Vanilla JS, HTML, CSS (for Frontend). <br> + Browser Extension: Manifest V3, Vanilla JS/CSS. | 27/05/2026 | 31/05/2026 | |

### Week 4 Achievements:

* Successfully initialized the foundation for the Desktop application using Electron framework, Node.js, and the basic structure for Browser Extension (Manifest V3).
* Completely built the centralized session management feature on the Desktop interface (including Pomodoro countdown timer and manual toggle mode).
* Successfully established a real-time communication channel between the Desktop App and Browser Extension via WebSocket.
* Built the lock screen (Overlay) feature on the Extension, blocking access to entertainment social networks (Facebook, TikTok, Instagram...) during focus hours.
* Implemented the data extraction mechanism (DOM Extraction) on YouTube to get the internal category of the video, thereby setting up an automatic filter: blocking entertainment categories and allowing educational ones.
* Completed the anti-bypass mechanism (Extension Gate): The Desktop system can scan running browser processes and automatically disable them if it detects the user intentionally turned off the extension.
