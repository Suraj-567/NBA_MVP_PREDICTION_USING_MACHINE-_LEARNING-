# 🏀 NBA MVP Prediction using Machine Learning

> **Project Duration:** 1991–2025  
> **Goal:** Predict the NBA **Most Valuable Player (MVP)** using machine learning models based on historical player and team statistics.

---

## 📘 Overview

This project analyzes and predicts the **NBA MVP** by building a full **machine learning pipeline** — from **data scraping** to **model evaluation**.  
We used player and team statistics from **Basketball Reference** and trained multiple ML models to identify the most probable MVP for the **2025 NBA season**.

---

## ⚙️ Phase 1 — Data Scraping

**Libraries Used:** `BeautifulSoup`, `requests`, `pandas`

We scraped and combined data from public basketball databases to create a unified dataset of **player performance** and **MVP voting results**.

### 📊 Data Collected:
- **Player Season Stats:** Points, Assists, Rebounds, Shooting %, etc.  
- **Team Performance:** Win/Loss %, SRS, Offensive & Defensive Ratings  
- **MVP Voting:** Points Won, Points Max, Share  

After merging and aligning data, the final dataset was stored as:


### 🧾 Example Features:
| Feature | Description |
|----------|--------------|
| Player | Player Name |
| Team | Team Played For |
| Pos | Playing Position |
| G, PTS, AST, TRB | Core Player Stats |
| W/L%, PS/G, PA/G, SRS | Team Performance Metrics |
| Pts Won, Pts Max, Share | MVP Voting Stats |
| is_mvp | Binary Target (1 = MVP Winner) |

---

## 🧹 Phase 2 — Data Cleaning

**File:** `2_data_cleaning.py`  
We cleaned and standardized all scraped data to ensure consistency and accuracy.

### 🧼 Key Steps:
- Removed redundant columns (`Unnamed: 0`, etc.)
- Trimmed whitespace and coerced data types
- Converted numeric-like columns to float64
- Handled missing values:
  - Numeric → filled with **median**
  - Categorical → filled with **mode**
- Recomputed missing FG% as `FG / FGA`
- Removed duplicates based on `(Player, Year, Team)`
- Ensured valid ranges for Year, Position, and Team

✅ **Cleaned Output:**  
`player_mvp_stats_cleaned.csv` → `16,249 rows × 41 columns`

**Summary:**
- All missing values handled  
- Data covers **1991–2025**  
- Class balance: 35 MVPs vs 16,214 Non-MVPs  

---

## 🔀 Phase 3 — Data Splitting

We split data **chronologically** to avoid leakage from future seasons.

| Dataset | Years | Size |
|----------|--------|------|
| Train | 1991–2024 | 15,684 rows |
| Test | 2025 | 565 rows |

**Target:** `is_mvp`  
**Features:** All numeric & categorical stats excluding identifiers (`Player`, `Team`, `Pts Won`, `Share`).

```python
train_df = df[df["Year"] < 2025]
test_df = df[df["Year"] == 2025]
```
 ## 🧠Phase 4 — Feature Encoding & Scaling
We preprocessed data using Scikit-learn Pipelines and ColumnTransformer.

| Type | Columns | Encoder |
| :--- | :--- | :--- |
| Categorical | Pos | OneHotEncoder |
| Categorical | Team | OrdinalEncoder |
| Numeric | All numeric stats | StandardScaler |

✅ **Output Files:**
* `X_train_processed.npy`
* `X_test_processed.npy`
* `y_train.npy`
* `y_test.npy`

| Dataset | Shape |
| :--- | :--- |
| **X\_train\_processed** | (15684, 40) |
| **X\_test\_processed** | (565, 40) |

---

 ## 🤖Phase 5 — Model Training
We trained and evaluated four ML models to predict the MVP:

| Model | Type | Class Weight |
| :--- | :--- | :--- |
| **Logistic Regression** | Linear | Balanced |
| **Ridge Classifier** | Linear (L2) | Balanced |
| **Gaussian Naive Bayes** | Probabilistic | - |
| **Random Forest** | Ensemble | Balanced |

### 🏆 Logistic Regression
* **Accuracy:** 0.9965
* **Recall (MVP):** 1.0
* ✅ **Predicted MVP:** Shai Gilgeous-Alexander
* ✔️ Correctly identified actual 2025 MVP

### 🧩 Ridge Classifier
* **Accuracy:** 0.9611
* **Recall (MVP):** 1.0
* **Top MVP:** Nikola Jokić
* Lower confidence calibration

### 🧠 Naive Bayes
* **Accuracy:** 0.9681
* **Recall (MVP):** 1.0
* **Top MVP:** Jayson Tatum
* Overpredicts due to Gaussian assumptions

### 🌲 Random Forest
* **Accuracy:** 0.9982
* **Recall (MVP):** 0.0
* **Top MVP:** Shai Gilgeous-Alexander
* Fails on recall due to class imbalance

---

## 📊Phase 6 — Model Comparison Summary
| Model | Accuracy | MVP Recall | MVP Precision | Top Predicted MVP |
| :--- | :--- | :--- | :--- | :--- |
| **Logistic Regression** | 0.9965 | 1.0 | 0.3333 | Shai Gilgeous-Alexander |
| **Ridge Classifier** | 0.9611 | 1.0 | 0.0435 | Nikola Jokić |
| **Naive Bayes** | 0.9681 | 1.0 | 0.0526 | Jayson Tatum |
| **Random Forest** | 0.9982 | 0.0 | 0.0 | Shai Gilgeous-Alexander |

🏁 **Winner: Logistic Regression**
* ✅ Best balance between precision & recall
* ✅ Correct MVP identification
* ✅ High interpretability

---

## 🏀 Phase 7 — Final Prediction (2025 Season)
| Rank | Player | Team | Probability |
| :--- | :--- | :--- | :--- |
| 1 | Shai Gilgeous-Alexander | OKC Thunder | 1.000 |
| 2 | Nikola Jokić | Denver Nuggets | 0.640 |
| 3 | Jalen Williams | OKC Thunder | 0.537 |
| 4 | Jayson Tatum | Boston Celtics | 0.061 |
| 5 | James Harden | LA Clippers | 0.037 |
| 6 | Giannis Antetokounmpo | Milwaukee Bucks | 0.007 |
| 7 | LeBron James | LA Lakers | 0.006 |
| 8 | Donovan Mitchell | Cleveland Cavaliers | 0.001 |
| 9 | Darius Garland | Cleveland Cavaliers | 0.001 |
| 10 | Luka Dončić | Dallas Mavericks | 0.0005 |

🎯 **Final Prediction: Shai Gilgeous-Alexander — correctly identified as 2025 MVP 🏆**

---

## 🔧 Phase 8 — Model Tuning
We optimized hyperparameters using GridSearchCV (5-fold cross-validation).

| Model | Best Params | Best CV F1 |
| :--- | :--- | :--- |
| **Logistic Regression** | `{'C': 1, 'penalty': 'l2'}` | 0.8571 |
| **Ridge Classifier** | `{'alpha': 1, 'solver': 'saga'}` | 0.8345 |
| **Random Forest + SMOTE** | `{'max_depth': 10, 'n_estimators': 300}` | 0.8123 |

✅ **Confirmed:** Logistic Regression = Most stable and accurate model.

### 📈 Evaluation Metrics Summary
| Metric | Logistic Regression (Best) | Score |
| :--- | :--- | :--- |
| Accuracy | ✅ | 0.9965 |
| Precision (MVP) | ✅ | 0.3333 |
| Recall (MVP) | ✅ | 1.0 |
| F1-Score | ✅ | 0.5 |
| ROC-AUC | ✅ | 1.000 |

### 💡 Key Insights
* 🔹 **Logistic Regression** is highly interpretable and effective for rare-event prediction (MVP).
* 🔹 **Ridge & Naive Bayes** serve as strong linear baselines.
* 🔹 **Random Forest** overfits and struggles with extreme class imbalance.
* 🔹 Correct prediction of **Shai Gilgeous-Alexander** validates model performance.

---

## 🧭 Future Improvements
🚀 To enhance future performance:
* Use SMOTE + Ensemble methods for better minority-class recall.
* Include advanced stats like Win Shares, VORP, BPM.
* Experiment with XGBoost or LightGBM.
* Use ranking-based loss functions tailored for MVP ranking tasks.

---

 ## ⚡How to Run
### Open the Notebook
```bash
jupyter notebook "Machine _learning.ipynb"
```
### Run Step-by-Step Sections

📊 Data Cleaning

✂️ Data Splitting

⚙️ Preprocessing

🤖 Model Training & Evaluation

🔧 Model Tuning

🏁 Final Evaluation & Top MVP Prediction

### Outputs Generated

* Cleaned dataset → player_mvp_stats_cleaned.csv

* Processed feature arrays → X_train_processed.npy, X_test_processed.npy

* Target arrays → y_train.npy, y_test.npy


## 📉 Visualization

### 📊 Visuals generated using Matplotlib and Seaborn:

* Correlation Heatmaps

* Feature Importance Charts

* MVP Probability Bar Plots

* Confusion Matrices

## 🙌 Acknowledgements

* Data Source: Basketball Reference

* Libraries Used: Pandas, NumPy, Scikit-learn, BeautifulSoup, Selenium, Matplotlib, Seaborn
* Year Range: 1991–2025
