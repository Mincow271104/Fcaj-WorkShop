---
title : "Educational Minigame System"
date : 2026-07-20
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

## 1. Overview

The Minigame system is a strategic module integrated directly into the educational platform, serving as a bridge between cognitive training and relaxation. By applying Gamification mechanics through classic logic games like Sudoku and Minesweeper, the system helps learners reduce stress while providing motivation to maintain daily learning habits.

Technologically, the system is built entirely on a Serverless Cloud-native architecture on AWS:

- **Client Side:** The Desktop application combines Electron, React, and Redux for state management, creating a smooth user experience and leveraging local caching (Local Caching) to optimize response speeds.
- **Server Side:** Communication routes through Amazon API Gateway, business logic is processed via AWS Lambda functions, background tasks are automated using Amazon EventBridge, and all session states and scores are stored on the Amazon DynamoDB NoSQL database.

---

## 2. Resource Preparation and Deployment

### Step 1: Prepare the AWS DynamoDB database

Create the database table on AWS DynamoDB via Console or AWS CLI:
- Create a DynamoDB table with the name defined in the MINIGAME_TABLE environment variable.
- Configure the Primary Key including Partition Key (PK) and Sort Key (SK) following a Single-Table Design NoSQL model to optimize queries for Sudoku, Minesweeper, and the Leaderboard.

![Minigame table on DynamoDB](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651615/minigame_table_hrjnzo.png)

### Step 2: Prepare level data for each minigame

- Create lists of game levels as JSON files, then upload them to Amazon S3 following the designated directory structure to populate the MINIGAME_TABLE.

![Level data uploaded to S3](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651630/S3_item_ygzgfq.png)

### Step 3: Configure the Backend Lambda Service

- Build API functions to handle game logic, anti-cheat mechanisms, and reward calculations for each minigame (minesweeperService.mjs, sudokuService.mjs, minigameService.mjs).

![Game logic processing files and an example of a Lambda function](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651630/S3_item_ygzgfq.png)

- Configure all environment variables on Lambda, including MINIGAME_TABLE, USER_TABLE, and GAME_SECRET_KEY.

![Environment variables](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651652/env_zrewsm.png)

### Step 4: Set up a recurring Worker to update the Leaderboard with EventBridge

- Create an EventBridge Rule with a recurring schedule running every 10 minutes: rate(10 minutes).
- Point the Rule target directly to the Lambda function handling the leaderboard logic (handleLeaderboardWorker).

![Lambda function to load the leaderboard](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651673/leaderboard_erwqsb.png)

### Step 5: Launch the Client Frontend (Electron & Redux)

- Launch the Electron application integrated with the Redux Store (minigameReducer.js, minigameLogsReducer.js).
- Configure API services (minigameService.js, minesweeperService.js, sudokuService.js) to connect to API Gateway.

![Minigame hub interface running on the client](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651685/minigameClient_z2ytes.png)

---

## 3. Minigame System Workflow in the Educational App

### 3.1. Overall Architecture

The workflow of the Minigame system is designed as a closed Client-Server loop, combining local storage caches (Redux and Electron Store) on the desktop app and a cloud database (AWS DynamoDB) via Serverless services.

### 3.2. Detailed Flows in a Session Lifecycle

#### Flow 1: Initialization and data synchronization at Minigame Hub

- The user accesses the Minigame Hub section and selects their desired game (Sudoku or Minesweeper).
- The system synchronizes the list of levels via corresponding services (handleSyncSudokuLevels or handleSyncMinesweeperLevels). The application checks local cache for instant display, while sending a request through API Gateway to AWS Lambda to fetch the latest level list from DynamoDB if needed.

![Level list interface running on the client](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651698/level_list_nj0un8.png)

#### Flow 2: Starting a match and managing stamina resources

- When a user selects a level, the client performs a preliminary check on unlock conditions and current stamina (Sanity) compared to the required cost (sanityCost).
- A POST request (handleStartSudokuSession / handleStartMinesweeperSession) is sent to the server to initiate the session.
- The Lambda function directly deducts stamina from the user's profile, initializes the game board matrix (solutionGrid and puzzleGrid), and writes a new session record with a PENDING status into DynamoDB.

![Interface after starting a Sudoku session](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651725/open_level_vutzqf.png)

![Session data stored on DynamoDB](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651738/session_DB_qf0gep.png)

#### Flow 3: Real-time interaction and security mechanics

During the puzzle-solving process, moves (filling numbers in Sudoku or revealing cells/placing flags in Minesweeper) are recorded into action log arrays (actionLogs) temporarily stored in Redux.

The system applies security and verification mechanisms:
- **For Sudoku:** When the user requests a move validation (handleCheckSudokuStep), the server runs a verification algorithm (checkSudokuCheat) to prevent time manipulation, rapid automated bot clicks, or board tampering.
- **For Minesweeper:** Each cell reveal operation (handleRevealApi) sends a state-encoded token. The server decrypts it in memory to check if a mine is triggered to end the game, or executes the empty space spread algorithm (runFloodFill) and returns a new token for the next move.

#### Flow 4: Match conclusion and reward processing

The match concludes based on the player's actual performance:
- **Winning Scenario (WIN):** When the player accurately completes the board and submits (handleSubmitSudoku / handleSubmitMinesweeper), the server performs final validation, calculates score based on time and difficulty, awards eCoin currency, updates personal records, and sets the session state to COMPLETED on DynamoDB.
- **Losing or Quitting Scenario (LOST / QUIT):** If the player solves incorrectly, steps on a mine, or actively quits mid-game (endState = 'quit'), the system automatically refunds 50% of the initial stamina cost back to the user's wallet, and sets the session state to CANCELLED or UNCOMPLETED. This policy helps alleviate psychological pressure on the learner.

![Interface after completing a level](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651748/sudoku_win_ji9ram.png)

![Session status updated to COMPLETED on DynamoDB](https://res.cloudinary.com/lpwszzwp/image/upload/v1784651753/session_complete_iopm8d.png)

#### Flow 5: Automated background leaderboard updates

- Every 10 minutes, EventBridge triggers a Lambda Worker function (handleLeaderboardWorker).
- The Worker scans the statistics table (stats) on DynamoDB, aggregates total scores across all players, and updates the Top 10 ranking list (leaderboard) complete with an expiration timestamp (expiresAt).

---

## 4. Functional Value Analysis for an Educational App

Integrating Gamification mechanics with two logic games, Sudoku and Minesweeper, into the learning platform yields key strategic values:

### 4.1. Solving Cognitive Fatigue

- **Problem:** Learners easily become discouraged and lose focus when forced to continuously absorb large amounts of academic knowledge over extended periods.
- **Solution:** The Minigame acts as a healthy micro-break buffer. Instead of leaving the app to scroll social media, users can take a quick 3–5 minute break playing Sudoku or Minesweeper to relax while maintaining logical thinking engagement within the same app ecosystem.

### 4.2. Building a user retention loop through app economy

- **Stamina Mechanic (Sanity):** Limits consecutive game sessions to prevent excessive play, while motivating users to return to main learning lessons to earn additional stamina and eCoin resources.
- **50% Stamina Refund:** Serves as a thoughtful UX touchpoint. Refunding a portion of stamina upon failure mitigates frustration, encouraging learners to attempt harder levels without fearing total resource loss.

### 4.3. Cultivating soft skills and logical thinking

- **Sudoku:** Enhances patience, deep concentration, memory retention, and deductive reasoning by elimination.
- **Minesweeper:** Develops probability and statistical thinking, risk management, and decision-making under incomplete information.