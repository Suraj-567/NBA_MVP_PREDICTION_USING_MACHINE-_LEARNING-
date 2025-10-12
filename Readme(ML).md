🏀 NBA MVP Prediction using Machine Learning
📘 Overview

This project aims to analyze and predict the NBA Most Valuable Player (MVP) using player and team statistics.
We built a full machine learning pipeline including data scraping, cleaning, preprocessing, model training, and evaluation.
Our goal was to identify the most likely MVP player for the 2025 NBA season using historical data from 1991–2025.




⚙️ Phase 1 — Data Scraping

We scraped historical NBA player stats and MVP voting results using BeautifulSoup and requests libraries.
Data was collected from public sources (like Basketball Reference) including:

Player season stats (points, assists, rebounds, shooting %s)

Team performance (W/L %, SRS, offensive & defensive ratings)

MVP voting shares (Pts Won, Pts Max, Share)

After combining and aligning player statistics with their MVP voting results, we created:

player_mvp_stats.csv


Each record represents a player’s season performance, containing 40+ features such as:

Feature	Description
Player	Player Name
Team	Team played for
Pos	Playing position
G, PTS, AST, TRB	Key performance metrics
W/L%, PS/G, PA/G, SRS	Team performance metrics
Pts Won, Pts Max, Share	MVP voting results
is_mvp	Binary target (1 = MVP winner)
🧹 Phase 2 — Data Cleaning

File: 2_data_cleaning.py

Key steps performed:

Removed index-like columns (e.g. Unnamed: 0)

Trimmed whitespace and coerced types

Converted all numeric-like columns to numeric (float64)

Handled missing values

Numeric → filled with median

Categorical → filled with mode

Recomputed FG% from FG/FGA if missing

Removed duplicates by (Player, Year, Team)

Checked for sanity ranges (Year, Teams, Positions)

Saved cleaned dataset → player_mvp_stats_cleaned.csv

✅ Output summary:

16,249 rows × 41 columns

Missing values handled completely

Year range: 1991–2025

is_mvp distribution: 35 MVPs vs 16,214 non-MVPs

🔀 Phase 3 — Data Splitting

We split the dataset by Year:

Train: 1991–2024 seasons

Test: 2025 season

This ensures our test set represents the upcoming season we want to predict.

train_df = df[df["Year"] < 2025]
test_df = df[df["Year"] == 2025]


Feature-target separation:

Target: is_mvp

Features: All player/team stats except identifiers like Player, Team, and leaky columns (Pts Won, Share).

Train/test shapes:

X_train: (15684, 38)
X_test : (565, 38)

🧠 Phase 4 — Feature Encoding & Scaling

We encoded and standardized features using Scikit-learn pipelines:

Type	Columns	Encoder
Categorical	Pos	OneHotEncoder
Categorical	Team	OrdinalEncoder
Numeric	All numeric stats	StandardScaler

All transformations combined via ColumnTransformer.
Processed features were saved as .npy arrays for training.

✅ Output:

X_train_processed: (15684, 40)
X_test_processed : (565, 40)

🤖 Phase 5 — Model Training

We trained and compared four ML models:

Model	Type	Class Weight
Logistic Regression	Linear	Balanced
Ridge Classifier	Linear (L2)	Balanced
Gaussian Naive Bayes	Probabilistic	-
Random Forest	Ensemble	Balanced
🏆 Logistic Regression

Accuracy: 0.9965

Recall (MVP): 1.0

Top Predicted MVP: Shai Gilgeous-Alexander
✅ Correctly identified the MVP with realistic probability ranking.

🧩 Ridge Classifier

Accuracy: 0.9611

Recall (MVP): 1.0

Top Predicted MVP: Nikola Jokić
Works well, but weaker confidence calibration.

🧠 Naive Bayes

Accuracy: 0.9681

Recall (MVP): 1.0

Top Predicted MVP: Jayson Tatum
Tends to overpredict due to Gaussian assumptions.

🌲 Random Forest

Accuracy: 0.9982

Recall (MVP): 0.0

Top Predicted MVP: Shai Gilgeous-Alexander
Fails on recall due to class imbalance — needs resampling.

📊 Phase 6 — Model Comparison Summary
Model	Accuracy	MVP Recall	MVP Precision	Top Predicted MVP
Logistic Regression	0.9965	1.0	0.3333	Shai Gilgeous-Alexander
Ridge Classifier	0.9611	1.0	0.0435	Nikola Jokić
Naive Bayes	0.9681	1.0	0.0526	Jayson Tatum
Random Forest	0.9982	0.0	0.0	Shai Gilgeous-Alexander

✅ Winner: Logistic Regression

Best overall precision–recall balance

Correct MVP detection

High interpretability

🏁 Phase 7 — Final Prediction (2025)

Top 10 predicted MVP candidates:

Rank	Player	Team	Probability
1	Shai Gilgeous-Alexander	OKC Thunder	1.000
2	Nikola Jokić	Denver Nuggets	0.640
3	Jalen Williams	OKC Thunder	0.537
4	Jayson Tatum	Boston Celtics	0.061
5	James Harden	LA Clippers	0.037
6	Giannis Antetokounmpo	Milwaukee Bucks	0.007
7	LeBron James	LA Lakers	0.006
8	Donovan Mitchell	Cleveland Cavaliers	0.001
9	Darius Garland	Cleveland Cavaliers	0.001
10	Luka Dončić	LA Lakers	0.0005
🔧 Phase 8 — Model Tuning

We optimized hyperparameters using GridSearchCV (5-fold CV):

Model	Best Params	Best CV F1
Logistic Regression	{'C': 1, 'penalty': 'l2'}	0.8571
Ridge Classifier	{'alpha': 1, 'solver': 'saga'}	0.8345
Random Forest + SMOTE	{'rf__max_depth': 10, 'rf__n_estimators': 300}	0.8123

Tuning confirmed Logistic Regression as the best performing and most stable model.

📈 Evaluation Metrics Summary
Metric	Best Model (LogReg)	Score
Accuracy	0.9965	
Precision (MVP)	0.3333	
Recall (MVP)	1.0	
F1-Score	0.5	
ROC-AUC	1.000	
💡 Insights

Logistic Regression gives strong and interpretable performance for rare-event prediction (MVP).

Ridge and Naive Bayes are useful baselines.

Random Forest overfits and underperforms on extreme imbalance.

Shai Gilgeous-Alexander was correctly predicted as 2025 MVP front-runner.

🧭 Future Improvements

Use SMOTE + Ensemble methods for better rare-class recall.

Add advanced stats like Win Shares, VORP, BPM.

Experiment with XGBoost or LightGBM.

Use ranking-based loss functions for MVP prediction.


####How to Run

Open Notebook

jupyter notebook Machine\ _learning.ipynb


Step-by-Step Execution

Run each cell in order.

Sections are organized as follows:

Data Cleaning

Data Splitting

Preprocessing

Model Training & Evaluation

Model Tuning

Final Evaluation & Top MVP Prediction

Data Output

Cleaned dataset: player_mvp_stats_cleaned.csv

Preprocessed features and labels: X_train_processed.npy, X_test_processed.npy, y_train.npy, y_test.npy

Visualization

Correlation heatmaps and evaluation plots are generated using matplotlib and seaborn.
