
# 🧠 Riverline Next-Best-Action (NBA) System

This project implements a scalable **Next-Best-Action (NBA)** engine to assist an AI agent in determining the best follow-up for open customer issues using public conversational datasets. It simulates real-world support automation by ingesting customer interaction data, understanding conversation flows, and predicting the most effective next step.

---

## 📂 Project Structure

```
.
├── main.ipynb              # Main logic: ingestion, user behavior, NBA engine
├── main_demo.ipynb         # Cleaned version for demo and evaluation
├── nba_results.csv         # Final NBA recommendations for 1000 customers
├── results.csv             # CSV in exact Riverline format
├── sample.csv              # Additional sample predictions or test data
├── README.md               # You're reading it!
```

---

## 📊 Datasets Used

1. **Customer Support on Twitter (CST)**  
   Multi-turn customer support conversations with brands on Twitter. Used to derive:
   - Conversation flows
   - User intent
   - Resolution tagging  
   [Link to dataset](https://www.kaggle.com/datasets/thoughtvector/customer-support-on-twitter)

2. **(Optional) Reddit MBTI**  
   Enriches customer personas and behavior modeling via personality traits.  
   [Link to dataset](https://www.kaggle.com/datasets/minhaozhang1/reddit-mbti-dataset)

3. **Other sources referenced** (optional enrichment only):
   - Sentiment140 (for sentiment calibration)
   - EmpatheticDialogues (for conversational patterns)
   - PAN Author Profiling (for behavior cohorting)

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

Example:
```json
{
  "customer_id": "123456",
  "channel": "email_reply",
  "send_time": "2025-07-03T16:00:00Z",
  "message": "Hi! Just checking if you're still facing the login issue. Let us know!",
  "reasoning": "Customer did not respond to previous prompt but has a history of replying via email during afternoon hours."
}
```

### 4. 📈 Evaluation

- Run end-to-end pipeline on 1000 users.
- Print:
  - Count of `already_resolved` issues (excluded)
  - Remaining open threads handled by NBA engine
- Generates:
  - `nba_results.csv`: All 1000 customer instructions
  - `results.csv`: Final result in [this exact format](https://docs.google.com/spreadsheets/d/1sTNMSkDJGibIA38MnaRFLDQ79NfPAXfOApymxZ-rP4E)

Each row includes:
- `customer_id`
- `channel`
- `send_time`
- `message`
- `reasoning`
- `chat_log`:
  ```
  Customer: I can't log in  
  Support: Try 'forgot password'  
  Customer: It didn't work  
  ```
- `issue_status`: One of `resolved`, `pending_customer_reply`, `escalated`

---

## 🧪 How to Run

### 1. Install dependencies
```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```

### 2. Launch Jupyter Notebook
```bash
jupyter notebook main.ipynb
```

### 3. Generate Outputs
- Run all cells in `main.ipynb` to:
  - Build the data pipeline
  - Observe behavior
  - Predict NBA actions
- Output files:
  - `nba_results.csv`
  - `results.csv`

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

## 📚 External References

- [Kaggle CST Dataset](https://www.kaggle.com/datasets/thoughtvector/customer-support-on-twitter)
- [Riverline Assignment Brief](https://docs.google.com/spreadsheets/d/1sTNMSkDJGibIA38MnaRFLDQ79NfPAXfOApymxZ-rP4E)
- PAN, Sentiment140, EmpatheticDialogues for enrichment

---

## ✍️ Author

**Aditya**, July 2025  
_Contact for clarification or code access: [Your Email]_
