---
title: "Overview"
date: 2026-07-19
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Project Background and System Design Problems

## Project Background

One of the biggest challenges in self-study is not accessing knowledge, but maintaining focus and motivation long enough to achieve meaningful results. Learning outcomes often appear slowly and are hard to see immediately, while social media or entertainment apps constantly provide instant feedback. Therefore, learners may clearly understand the benefits of studying but still struggle to start, keep up with a schedule, or notice their own progress.

This phenomenon is partly related to the brain's reward-based learning system. Dopamine should not just be understood as a "feel-good chemical"; it also plays a role in forming expectations, learning from results, and reinforcing behaviors deemed valuable. When an action gets clear feedback, such as observable progress, an achievement, or a meaningful reward, the brain tends to link that action with a positive outcome. Appropriate and repeated feedback can help form a behavior loop:

> **Action → Feedback → Reward → Motivation to repeat**

Uchimi StudyGamification applies this principle through focus sessions, daily tasks, progress indicators, achievements, rewards, minigames, and social interaction. These elements are not meant to replace the value of knowledge or turn learning into a reward-collecting activity. Instead, they help reduce the psychological barrier to getting started, make progress clear, and support users in gradually shifting from external motivation to internal motivation and a sustainable study habit.

## Problem Statement and Project Goals

Traditional learning tools often focus on providing content and managing tasks but do not fully support maintaining long-term motivation. In contrast, a purely entertainment-based reward system might attract users but does not create real learning outcomes. Therefore, Uchimi needs to make learning engaging enough for users to join voluntarily, while ensuring the cloud-based gamification system is always fair, consistent, responsive, secure, scalable, and cost-optimized.

The project builds a desktop learning app, combining **ReactJS and Electron on the client side** with **AWS Serverless architecture on the backend**. The core goal is to make learning a voluntary, easily repeatable activity by giving each study session a clear goal, timely feedback, and a sense of achievement. Specific goals include:

- breaking down long-term goals into small, actionable daily steps;
- visualizing progress through focus time, study streaks, tasks, scores, and achievements;
- using rewards and minigames as positive reinforcement after meaningful learning activities;
- supporting personalized planning and knowledge review with AI-assisted features;
- building a fair social interaction environment through friends and leaderboards; and
- providing a highly responsive experience with a scalable yet cost-controlled infrastructure.

The project does not aim to force users to study more. The system aims to create conditions so they are ready to start, can see their progress, and proactively return to study regularly.

## Target Audience

### Students

High school and college students often have to absorb a large amount of knowledge, complete assignments on time, and prepare for exams. Common struggles include loss of focus, procrastination, stress, and difficulty measuring daily progress. Uchimi helps turn general goals into focus sessions and daily tasks, supports reviewing with quizzes, and provides feedback right after each activity. Ranked study modes and progress statistics also create a clearer structure for users who need more discipline.

### Working Professionals

Working professionals often study to improve professional skills, learn foreign languages, prepare for certificates, or pursue personal hobbies. The main limitation for this group is having little and fragmented time, instead of a fixed schedule. Uchimi supports them with flexible planning, short study sessions, progress syncing, and AI-assisted learning features, thereby helping them maintain their studies without creating an extra burden after work.

### Shared Values for Both Groups

Despite having different schedules and goals, both groups need support to get started, stay consistent, and recognize their progress. Uchimi turns long-term goals into manageable activities, acknowledges effort over time, and connects learning with feedback as well as appropriate entertainment. Users are still in control of their goals and can choose a pace that fits their personal situation. A voluntary, user-centric approach is key to forming a lasting habit rather than just creating short-term interest.

## Core Technical Problems

### 1. Preventing Cheating in Learning Activities

Study time, completion status, ranking points, task progress, and rewards cannot be entirely decided by data sent from the client, because desktop apps can be modified and requests can be replayed. The server must be the decision-maker for important rules. Therefore, each study session needs to be linked to an authenticated user, recorded server time, valid state transitions, controlled strike mechanisms, and server-calculated results. Duplicate, expired, out-of-order, or invalid requests must be rejected or handled safely. This mechanism protects not only rewards but also the reliability of streaks, ranked results, and leaderboards.

### 2. Preventing Cheating in Minigames

Minigames have a different risk model than study sessions: users might alter scores, send known answers, automate actions, change the time, or claim a result multiple times. Game configurations and answers must be generated or verified by the server. Verification can combine answer accuracy, server-recorded time, reasonable completion windows, action patterns, session limits, and a one-time reward settlement mechanism. Suspicious results must not affect currency, task progress, or leaderboards. Checks must ensure fairness but without excessively tracking the client, which would increase latency and storage costs.

### 3. Data Integrity and Consistency in the Cloud

Learning progress, Knowledge Points, in-system currency, item inventory, rewards, Gacha history, and friend relationships are all interconnected. A timed-out or re-sent request must not duplicate items, give double rewards, or cause negative balances. Thus, the system needs to use server-determined rules, input validation, idempotent processing, conditional updates, and atomic counters in DynamoDB, along with transactions for operations that must succeed or fail together. Important economic actions also require a traceable history. These mechanisms are especially necessary for purchasing items, currency exchange, claiming task rewards, Gacha, and updating two-way friend relationships.

### 4. Efficient Synchronization Between Client and Server

The desktop app needs to display useful information right at startup, even if the network is slow or temporarily disconnected. However, having the client constantly call every API to check for new data will increase latency, network traffic, Lambda executions, and database reads. Uchimi needs an offline-friendly strategy, combining local caching, version or timestamp-based change detection, aggregated sync endpoints, incremental updates, and expiration policies for data that can tolerate delays. Event-driven processing in the backend helps automatically spread internal changes. The conflict resolution mechanism must distinguish between data that can be merged on the client and decisive data like balances, rewards, and ranking results—which must always rely on the server as the final source of truth.

### 5. Optimizing Cloud Server Operational Costs

User activity levels change over time, so permanently allocated infrastructure can waste money during low-traffic periods. AWS Serverless services like Lambda, API Gateway, DynamoDB, S3, and EventBridge can scale on demand and charge based on usage. However, serverless does not always mean low cost. The system needs to reduce duplicate API calls, bundle compatible actions, cache rarely changing master data, avoid unnecessary DynamoDB scans, design appropriate indexes, automatically delete temporary records using TTL, and shift non-urgent tasks to scheduled or asynchronous processing. API rate limits, operational metrics, and cost alerts help protect the platform from inefficient processing loads or sudden traffic spikes.

## Cross-cutting Requirements

- **Security:** authenticate with Amazon Cognito, protect APIs with JWT, validate external data, and apply IAM permissions based on the principle of least privilege.
- **Observability:** use structured logs, operational metrics, error tracking, and cost alerts.
- **Scalability and Maintainability:** separate the backend by business domains, manage infrastructure as code, and control client-server communication versions.
- **Privacy and Responsible Design:** only collect necessary data and ensure gamification always serves, rather than overshadows, the learning goals.

## Expected Project Value

Uchimi illustrates the ability to combine behavioral design, AI-assisted learning, desktop app technology, and cloud-native architecture in a practical educational system. For users, the platform supports a voluntary mindset, visualizes progress, and builds long-term habits. Technically, the project is a case study on a gamification platform on AWS with the server as the reliable data source, ensuring consistency, offline-friendliness, and controlled operational costs.