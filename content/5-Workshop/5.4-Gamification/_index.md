---
title : "Educational Minigame System"
date : 2026-07-20
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

## 1. OVERVIEW

The Minigame system is a strategic module integrated directly into the educational platform, serving as the perfect "bridge" between cognitive training and relaxation. By applying Gamification mechanics through classic logic games (Sudoku and Minesweeper), the system not only helps learners reduce stress but also creates a strong incentive to retain them longer within the application.

Technologically, the system is built entirely on a Serverless Cloud-native architecture on AWS.

*   **Client Side:** The Desktop application utilizes Electron, React, and Redux for state management, creating a seamless experience, and utilizing local caching for fast responsiveness.
*   **Server Side:** Communication routes through Amazon API Gateway, business logic is processed via AWS Lambda functions, background tasks are automated using Amazon EventBridge, and all states and scores are stored on the Amazon DynamoDB NoSQL database.

## 2. MINIGAME SYSTEM WORKFLOW IN THE EDUCATIONAL APP

### 2.1. Overall Architecture
The workflow of the Minigame system is designed as a closed Client-Server loop, combining local storage caches (Redux & Electron Store) on the user's device and a cloud database (AWS DynamoDB) via Serverless services.

### 2.2. Detailed Flows in a Session Lifecycle

**Flow 1: Initialization and Data Synchronization at Hub (MinigameHub)**
*   The user accesses the Minigame Hub section and selects their desired game (Sudoku or Minesweeper).
*   The system synchronizes the list of levels via corresponding services (**handleSyncSudokuLevels** or **handleSyncMinesweeperLevels**). Here, the application checks the local cache (Redux Store and Electron Store) to optimize display speed, or sends a request through API Gateway to AWS Lambda to fetch the latest level list from the server.

**Flow 2: Starting a Match & Resource Management (sanity)**
*   When a user selects a specific level, the client performs a preliminary check on unlock conditions and current stamina (**sanity**) compared to the required cost (**sanityCost**).
*   Upon passing the check, a POST request (**handleStartSudokuSession** / **handleStartMinesweeperSession**) is sent to the server.
*   The backend (Lambda) directly deducts the **sanity** cost from the user's Profile, initializes the corresponding map structure (generating the **solutionGrid** matrix, creating holes for the **puzzleGrid**), and simultaneously writes the session record with a **PENDING** state into DynamoDB.

**Flow 3: Real-time Interaction & Security Mechanics**
*   During the puzzle-solving process, actions such as filling in numbers (Sudoku) or revealing cells/placing flags (Minesweeper) are recorded into action log arrays (**actionLogs**) stored in Redux.
*   **Security & Anti-Cheat Mechanics:**
    *   **For Sudoku:** Every time the user requests a move check (**handleCheckSudokuStep**), the server runs a behavioral analysis algorithm (**checkSudokuCheat**) to prevent time manipulation, rapid automated bot clicks, or puzzle tampering.
    *   **For Minesweeper:** Each cell reveal click (**handleRevealApi**) sends a state-encoded token; the server decrypts it directly in RAM, checks if a mine is triggered to end the game, or executes the empty space spread (**runFloodFill**) and issues a new token for the next move.

**Flow 4: Match End**
The match concludes based on the player's actual results:
*   **Winning Scenario (WIN):** The player accurately completes the board and sends a submission request (**handleSubmitSudoku** / **handleSubmitMinesweeper**). The server performs a final validation, calculates the score based on time and difficulty, rewards the user with **eCoin** currency, updates their Personal Best, and changes the session state on DynamoDB to **COMPLETED**.
*   **Losing or Quitting Scenario (LOST / QUIT):** If the player solves incorrectly, steps on a mine, or actively quits mid-game (**endState = 'quit'**), the system activates a refund policy returning 50% of the initial **sanity** cost back to the user's wallet, updating the session state to **CANCELLED** or **UNCOMPLETED**. This mechanism helps alleviate psychological pressure on the learner.

**Flow 5: Automated Background Leaderboard Update (Background Worker)**
*   Every 10 minutes, the AWS EventBridge router automatically triggers a Lambda Worker function (**handleLeaderboardWorker**).
*   This Worker scans (Scan command) the statistics table (**stats**) on DynamoDB across all players, sorts them by total score, and generates a Top 10 ranking list (**leaderboard**) complete with an expiration time (**expiresAt**).

## 3. FUNCTIONAL VALUE ANALYSIS FOR AN EDUCATIONAL APP

Integrating Gamification mechanics with two logic-based games (Sudoku & Minesweeper) into the learning platform yields the following core strategic values:

### 3.1. Solving "Cognitive Fatigue"
*   **Problem:** Learners easily become discouraged and lose focus when forced to continuously absorb large amounts of academic knowledge for extended periods on the app.
*   **Solution:** The Minigame acts as a healthy "Micro-break" buffer. Instead of exiting the app to scroll through social media (which completely breaks concentration), users switch to a quick 3–5 minute round of Sudoku or Minesweeper to stimulate logical thinking while remaining within the app's ecosystem.

### 3.2. Building a Core Retention Loop through App Economy
*   **Sanity Mechanic:** Prevents excessive "grinding" while simultaneously motivating users to return to main learning lessons on the app to "farm" stamina (**sanity**) and resources (**eCoin**).
*   **50% Sanity Refund Mechanic:** This is a fine-tuned UX touchpoint engineered to optimize player psychology. Refunding resources minimizes frustration upon failure, preventing users from feeling discouraged and abandoning the app. Consequently, players are continuously encouraged to challenge themselves on harder levels without fearing a devastating loss of resources.

### 3.3. Cultivating Soft Skills and Logical Thinking for Learners
*   **Sudoku:** Trains patience, intense focus, memory retention, and deductive reasoning by elimination—skills that directly complement complex problem-solving.
*   **Minesweeper:** Trains probability and statistical thinking, risk management, and decision-making under the pressure of asymmetrical information.