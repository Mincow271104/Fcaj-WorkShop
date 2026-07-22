---
title: "Workshop"
date: 2026-07-17
weight: 5
chapter: true
pre: " <b> 5. </b> "
---

# Uchimi StudyGamification - AI & Gamification Learning App

Welcome to the technical documentation for the **Uchimi StudyGamification** project. The project is built with a **ReactJS + Electron** architecture on the Client side and an **AWS Serverless** platform on the Server side, aiming to optimize operational costs while delivering an excellent learning experience, making self-study no longer feel restrictive or forced.

* **Project Demo Video:** [Watch Demo Video](https://youtu.be/8Bw4-nwUaTc)
* **Architecture diagram:** ![AWS architecture diagram](https://res.cloudinary.com/dqblg6ont/image/upload/v1784555384/Uchimi_StudyGamification-Uchimi_Architecture.drawio_nn9xhr.png)
* **Frontend Source Code:** [GitHub Repository - AWSStudy-Play](https://github.com/KhangChinh/AWSStudy-Play)
* **Backend Source Code:** [GitHub Repository - AWSServerless](https://github.com/KhangChinh/AWSServerless)
* **Download Desktop App (.exe):** [Download Setup (.exe)](https://drive.google.com/file/d/1Qkdv_nLrQWICSZFCWqytDioq4r35Vyz4/view?usp=sharing)

## Overview

In an era of rapid digital development, the biggest challenge of self-study is no longer the difficulty of accessing knowledge. The real difficulty lies in the **ability to maintain long-term focus and discipline** so that learning is not interrupted. Learners are easily distracted by countless temptations when using computers, making it extremely difficult to build a sustainable study habit.

**The Uchimi StudyGamification project** was created to address this problem. The core goal of the project is to turn the act of "turning on the computer to study every day" into a natural, sustainable, and exciting habit.

The system provides a comprehensive solution, representing the perfect intersection between **iron discipline** and **healthy entertainment**:
*   **Disciplined AI Learning:** Applies AI Scan technology to track and instantly block distracting behaviors when users operate on their computers, helping to maintain deep focus.
*   **Entertainment Ecosystem (Minigames):** After intense study sessions, users step into a relaxing space with engaging minigames. This combination turns academic pressure into a well-deserved reward, helping to effectively recharge energy.

By applying the **Discipline → Action → Feedback → Reward** loop, the project not only retains users but also gradually creates an entirely new study lifestyle.

## Documentation Outline

This documentation will dive deep into analyzing and providing detailed guidelines for the technical aspects of the project, including:

1. **[Project Context](5.1-Overview/)**
   * Analysis of the project's background, core objectives, and target audience.
   * Details of the technical problem to be solved to meet practical needs.
2. **[Project Architecture](5.2-Architecture/)**
   * Analysis of the reasons for choosing AWS Serverless architecture and technical barriers.
   * How to set up permissions, security barriers, and integrate third-party services.
   * Details of the AWS services used, construction methods, and the overall Architecture Diagram [drawio].
3. **[Study with AI](5.3-Study_with_AI/)**
   * Detailed description of the AI assistant's workflow and AI Scan technology.
   * How AI processes data, the benefits of the models/extensions used, and why this method is highly effective.
   * Setup guide for each AI type, study modes, and the quest system.
4. **[Gamification Ecosystem](5.4-Gamification/)**
   * Analysis of the seamless transition flow from the Study module to Minigames.
   * General description of the minigames' operating mechanics and core functions.
   * The Leaderboard system to increase competitiveness and user retention.
5. **[Deployment and Operations](5.5-Deployment/)**
   * Detailed guide on the application deployment process (how to package the build, download, and install).
   * Description of the practical operation flow and how users utilize the system.
6. **[Direction and Development](5.6-Direction/)**
   * Reflection and evaluation of existing limitations to improve.
   * Development roadmap and expected features to be integrated in the future.

---

## Source Code

- **Frontend (ReactJS + Electron):** https://github.com/KhangChinh/AWSStudy-Play
- **Backend (AWS Serverless):** https://github.com/KhangChinh/AWSServerless