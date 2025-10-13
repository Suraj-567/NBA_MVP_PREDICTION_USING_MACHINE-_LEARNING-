# 🏀 NBA MVP Prediction — Data Scraping & Cleaning

![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)
![BeautifulSoup](https://img.shields.io/badge/BeautifulSoup-Used-success.svg)
![Selenium](https://img.shields.io/badge/Selenium-Automation-brightgreen.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-orange.svg)
![Status](https://img.shields.io/badge/Phase-Data%20Collection%20%26%20Cleaning-lightgrey.svg)

---

## 📘 Project Overview

This repository is part of the **NBA MVP Prediction Project**, focusing on the **data scraping** and **data cleaning** phases.  
The goal is to collect, parse, and preprocess **player**, **team**, and **MVP voting** data from [Basketball Reference](https://www.basketball-reference.com/) for the seasons **1991–2025**.

> 🧠 The cleaned dataset (`combined_data.csv`) serves as the foundation for the **Machine Learning phase**, where MVP prediction models are trained and evaluated.

---

## ⚙️ Phase 1 — Web Scraping

This phase collects raw data from Basketball Reference using `requests`, `BeautifulSoup`, and `Selenium`.

### 🧩 1. Scraping MVP Voting Data

**Goal:**  
Fetch yearly MVP voting pages (1991–2025) and save them as HTML files in the `mvp/` folder.

```python
import requests, time

base_url = "https://www.basketball-reference.com/awards/awards_{year}.html"

for year in range(1991, 2026):
    print(f"Fetching data for {year}...")
    url = base_url.format(year=year)
    html = requests.get(url).text
    with open(f"mvp/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Saved {year}.html")
    time.sleep(3)
```
### 🧩 2. Extracting MVP Voting Tables

**Goal:** 
Parse saved HTML files, extract MVP voting tables, and merge them into a single dataset (mvps.csv).

```from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("mvp").glob("*.html"))
dfs = []

for file in files:
    with open(file, encoding="utf-8") as f:
        soup = BeautifulSoup(f.read(), "html.parser")
    table = soup.find("table")
    df = pd.read_html(StringIO(str(table)))[0]
    df["Year"] = file.stem
    dfs.append(df)

mvps = pd.concat(dfs)
mvps.to_csv("mvps.csv", index=False)
print("✅ Saved all MVP data to mvps.csv")
````
### 🧩 3. Scraping Player Statistics (Per Game)

**Goal:** 
Use Selenium to fetch player per-game stats for all seasons (1991–2025).

```from selenium import webdriver
import time, random
from pathlib import Path

Path("player").mkdir(exist_ok=True)
driver = webdriver.Chrome()

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_per_game.html"
    print(f"Fetching player stats for {year}...")
    driver.get(url)
    time.sleep(random.uniform(3, 5))
    html = driver.page_source
    with open(f"player/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Saved player stats for {year}")

driver.quit()
```
### 🧩 4. Extracting Player Statistics

**Goal:** 
Parse and clean all player stat pages into a consolidated dataset (players.csv).

```from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("player").glob("*.html"))
all_players = []

for file in files:
    with open(file, encoding="utf-8") as f:
        soup = BeautifulSoup(f.read(), "html.parser")
    table = soup.find("table", {"id": "per_game_stats"})
    df = pd.read_html(StringIO(str(table)))[0]
    df = df[df["Rk"] != "Rk"]  # Remove duplicate headers
    df["Year"] = file.stem
    all_players.append(df)

players = pd.concat(all_players)
players.to_csv("players.csv", index=False)
print("✅ Saved all player stats to players.csv")
```
### 🧩 5. Scraping Team Standings

**Goal:** 
Collect team standings (Eastern and Western Conferences) for every season (1991–2025).

```import requests, os, time

os.makedirs("team", exist_ok=True)
base_url = "https://www.basketball-reference.com/leagues/NBA_{year}_standings.html"

for year in range(1991, 2026):
    print(f"Fetching standings for {year}...")
    html = requests.get(base_url.format(year=year)).text
    with open(f"team/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Saved team standings for {year}")
    time.sleep(3)
```
### 🧩 6. Extracting Team Standings

**Goal:** 
Extract standings data into a unified dataset (teams.csv).

```from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("team").glob("*.html"))
dfs = []

for file in files:
    with open(file, encoding="utf-8") as f:
        soup = BeautifulSoup(f.read(), "html.parser")
    # (Extraction logic to combine East/West tables)
    # Append to dfs after parsing

teams = pd.concat(dfs)
teams.to_csv("teams.csv", index=False)
print("✅ Saved all team standings to teams.csv")
```
## 🧹 Phase 2 — Data Cleaning

This phase focuses on refining raw scraped data into analysis-ready CSV files.

### 🔧 Cleaning Player Data (players_cleaned.csv)

***Actions:***

Removed redundant columns and null rows

Stripped special symbols (e.g., * from player names)

Averaged stats for players traded mid-season (TOT entries)

```import pandas as pd

players = pd.read_csv("players.csv")
players = players[players["Player"].notna()]
players["Player"] = players["Player"].str.replace("*", "", regex=False)
players.drop(columns=["Rk"], inplace=True, errors="ignore")
players = players.groupby(["Player", "Year"], as_index=False).mean(numeric_only=True)
players.to_csv("players_cleaned.csv", index=False)
print("✅ Cleaned player data saved")
```
### 🔧 Cleaning MVP Data (mvps_cleaned.csv)

***Actions:***

Kept only relevant columns (Player, Year, Pts Won, Pts Max, Share)

Filled missing Share values with 0

Removed formatting symbols from player names
```
mvps = pd.read_csv("mvps.csv")
mvps = mvps[["Player", "Year", "Pts Won", "Pts Max", "Share"]]
mvps["Share"] = mvps["Share"].fillna(0)
mvps["Player"] = mvps["Player"].str.replace("*", "", regex=False)
mvps.to_csv("mvps_cleaned.csv", index=False)
print("✅ Cleaned MVP data saved")
```
### 🔧 Cleaning Team Data (teams_cleaned.csv)

***Actions:***

Removed division headers

Standardized team names

Converted numeric columns

```teams = pd.read_csv("teams.csv")
teams = teams[teams["Team"] != "Division"]
teams["Team"] = teams["Team"].str.replace("*", "", regex=False)
teams.to_csv("teams_cleaned.csv", index=False)
print("✅ Cleaned team data saved")
```
### 🔗 Final Combined Dataset (combined_data.csv)

**Goal:** 
Merge all three datasets — players, MVP votes, and team standings — into a unified dataset for machine learning.

```
players = pd.read_csv("players_cleaned.csv")
mvps = pd.read_csv("mvps_cleaned.csv")
teams = pd.read_csv("teams_cleaned.csv")

combined = players.merge(mvps, how="left", on=["Player", "Year"])
combined = combined.merge(teams, how="left", on=["Team", "Year"])
combined.fillna(0, inplace=True)
combined.to_csv("combined_data.csv", index=False)

print("✅ Final merged dataset saved as combined_data.csv")
```
