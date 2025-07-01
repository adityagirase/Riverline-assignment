
# 🧠 Riverline Next-Best-Action (NBA) System

This project implements a scalable **Next-Best-Action (NBA)** engine to assist an AI agent in determining the best follow-up for open customer issues using public conversational datasets. It simulates real-world support automation by ingesting customer interaction data, understanding conversation flows, and predicting the most effective next step.

---

## 📂 Project Structure

```
.
├── main.ipynb              # Main logic: ingestion, user behavior, NBA engine
├── main_demo.ipynb         # Cleaned version for demo and evaluation
├── nba_results.csv         # Final NBA recommendations for 1000 customers
├── results.csv             # Result file of kaggle sample dataset
├── sample.csv              # Additional sample predictions or test data
├── README.md               # You're reading it!
```

---

## ⚙️ System Overview

### 1. 🛠 Data Pipeline

- Pulls and processes tweets from CST.
- Normalizes multi-source text into a single interaction table.
- Ensures **idempotency** via hash-based deduplication.
- Outputs a clean conversation log for each customer ID.

### 2. 👁️‍🗨️ User Behavior Modeling

- Extracts **conversation flows** from multi-turn support threads.
- Tags each thread with:
  - `nature_of_support_request` (e.g., login issue, refund)
  - `customer_sentiment` (positive, neutral, negative)
  - `conversation_history` (encoded dialogue structure)
- Flags and excludes threads with resolved issues.
- Visualizes behavior cohorts using clustering and PCA.

### 3. 🧠 NBA Engine

- Predicts best follow-up action based on prior behavior patterns.
- Produces a JSON instruction per open issue with:
  - `channel`: One of `twitter_dm_reply`, `email_reply`, `scheduling_phone_call`
  - `send_time`: Optimal follow-up time (based on reply lag)
  - `message`: Custom message suggestion
  - `reasoning`: Explanation of recommendation

> **Approach Used**: A hybrid rule-based + lightweight classifier approach. Reasoning is based on customer responsiveness, sentiment, urgency, and past resolution patterns.

### 4. 📈 Evaluation

- Run end-to-end pipeline on 1000 users.
- Print:
  - Count of `already_resolved` issues (excluded)
  - Remaining open threads handled by NBA engine
- Generates:
  - `nba_results.csv`: All 1000 customer instructions
  - `results.csv`: Final result of kaggle sample dataset


---


## 🧾 Design Choices & Assumptions

- **Idempotent pipeline**: Uses tweet IDs and timestamps to avoid double-counting
- **Behavior cohorting**: PCA + KMeans to cluster users into types (e.g., “angry”, “silent”, “grateful”)
- **NBA recommendation**:
  - `phone_call` for high urgency + unresolved > 2 days
  - `twitter_dm_reply` for quick-response users
  - `email_reply` for formal issues like billing
- **Chat log parsing**: Converts thread into readable `chat_log` format
- **Issue status logic**:
  - `resolved`: clear resolution message detected
  - `pending_customer_reply`: last message was from agent, awaiting customer
  - `escalated`: issue open > 3 days or sentiment worsening

---

## 📈 Metrics Used

| Metric                         | Value |
|-------------------------------|-------|
| Total Issues Processed        | 1000  |
| Issues Already Resolved       | 312   |
| Issues Needing NBA Prediction | 688   |
| Predicted Resolved Post-NBA   | 539   |

> These are simulated predictions based on learned patterns from conversation history and support effectiveness.

---

