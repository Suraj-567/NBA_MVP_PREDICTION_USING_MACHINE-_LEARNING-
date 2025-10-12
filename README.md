#🏀 NBA MVP Prediction — Data Scraping & Cleaning

This repository contains the data scraping and cleaning pipeline for the NBA MVP Prediction project.
The goal of this phase is to collect, parse, and clean player, team, and MVP voting data from Basketball Reference
 for the seasons 1991–2025.

##⚙️ Phase 1 — Web Scraping
###🧩 1. Scraping MVP Voting Data

**Context:**
This script scrapes MVP voting tables for every year from 1991 to 2025 and saves each page as an HTML file in the mvp/ folder.

**Code:**

import requests
import time

base_url = "https://www.basketball-reference.com/awards/awards_{year}.html"

for year in range(1991, 2026):
    print(f"Fetching data for the year {year}...")
    url = base_url.format(year=year)
    html = requests.get(url).text
    with open(f"mvp/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Successfully saved {year}.html")
    time.sleep(3)

###🧩 2. Extracting MVP Voting Data

**Context:**
Once HTML files are saved, this script parses all the mvp/ pages and extracts the MVP tables into a single CSV file named mvps.csv.

**Code:**

from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("mvp").glob("*.html"))
dfs = []

for file in files:
    with open(file, encoding="utf-8") as f:
        page = f.read()
    soup = BeautifulSoup(page, "html.parser")
    table = soup.find(name="table")
    df = pd.read_html(StringIO(str(table)))[0]
    df["Year"] = file.stem
    dfs.append(df)

mvps = pd.concat(dfs)
mvps.to_csv("mvps.csv", index=False)
print("✅ Saved all MVP data to mvps.csv")

###🧩 3. Scraping Player Statistics (Per Game)

**Context:**
This script uses Selenium to scrape player per-game stats for all seasons (1991–2025).
Each year’s page is stored as an HTML file in the player/ folder.

**Code:**

from selenium import webdriver
import time, random
from pathlib import Path

Path("player").mkdir(exist_ok=True)

driver = webdriver.Chrome()

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_per_game.html"
    print(f"Fetching data for {year}...")
    driver.get(url)
    time.sleep(random.uniform(3, 5))  # polite scraping delay
    html = driver.page_source
    with open(f"player/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Saved player stats for {year}")
    
driver.quit()

####🧩 4. Extracting Player Statistics

**Context:**
This script parses all the player/ HTML files and extracts clean per-game statistics for each season.
The final dataset is saved as players.csv.

**Code:**

from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("player").glob("*.html"))
all_players = []

for file in files:
    with open(file, encoding="utf-8") as f:
        page = f.read()
    soup = BeautifulSoup(page, "html.parser")
    table = soup.find("table", {"id": "per_game_stats"})
    df = pd.read_html(StringIO(str(table)))[0]
    df = df[df["Rk"] != "Rk"]  # remove header duplicates
    df["Year"] = file.stem
    all_players.append(df)

players = pd.concat(all_players)
players.to_csv("players.csv", index=False)
print("✅ Saved all player stats to players.csv")

###🧩 5. Scraping Team Standings

**Context:**
This script scrapes NBA team standings (Eastern and Western Conferences) for each year from 1991 to 2025 and saves them to the team/ folder.

**Code:**

import requests
import os
import time

os.makedirs("team", exist_ok=True)
base_url = "https://www.basketball-reference.com/leagues/NBA_{year}_standings.html"

for year in range(1991, 2026):
    print(f"Fetching standings for {year}...")
    url = base_url.format(year=year)
    html = requests.get(url).text
    with open(f"team/{year}.html", "w+", encoding="utf-8") as f:
        f.write(html)
    print(f"✅ Saved team standings for {year}")
    time.sleep(3)

####🧩 6. Extracting Team Standings

**Context:**
This script extracts both Eastern and Western Conference standings from all saved team pages and combines them into one dataset (teams.csv).

**Code:**

from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
from pathlib import Path

files = sorted(Path("team").glob("*.html"))
dfs = []

for file in files:
    with open(file, encoding="utf-8") as f:
        page = f.read()
    soup = BeautifulSoup(page, "html.parser")

teams = pd.concat(dfs)
teams.to_csv("teams.csv", index=False)
print("✅ Saved all team standings to teams.csv")

##🧹 Phase 2 — Data Cleaning
###🔧 Cleaning Player Data

**Context:**
This script cleans and standardizes the player dataset:

Removes redundant columns and rows

Strips symbols like * from player names

Handles players traded mid-season (TOT team case)

Saves the cleaned dataset as players_cleaned.csv

**Code:**

import pandas as pd

players = pd.read_csv("players.csv")
players = players[players["Player"].notna()]

# Clean names and drop unnecessary columns
players["Player"] = players["Player"].str.replace("*", "", regex=False)
players.drop(columns=["Rk"], inplace=True, errors="ignore")

# Handle duplicates (due to multiple teams)
players = players.groupby(["Player", "Year"], as_index=False).mean(numeric_only=True)

players.to_csv("players_cleaned.csv", index=False)
print("✅ Cleaned player data saved as players_cleaned.csv")

###🔧 Cleaning MVP Data

**Context:**
This step ensures consistency in MVP dataset:

Keeps only essential columns

Fills missing MVP scores for players not in the vote list

Saves as mvps_cleaned.csv

**Code:**

mvps = pd.read_csv("mvps.csv")

mvps = mvps[["Player", "Year", "Pts Won", "Pts Max", "Share"]]
mvps["Share"] = mvps["Share"].fillna(0)

mvps["Player"] = mvps["Player"].str.replace("*", "", regex=False)

mvps.to_csv("mvps_cleaned.csv", index=False)
print("✅ Cleaned MVP data saved as mvps_cleaned.csv")

###🔧 Cleaning Team Data

**Context:**
This cleans the team dataset:

Removes division headers

Standardizes names

Converts numeric columns

Saves as teams_cleaned.csv

**Code:**

teams = pd.read_csv("teams.csv")

teams = teams[teams["Team"] != "Division"]
teams["Team"] = teams["Team"].str.replace("*", "", regex=False)

teams.to_csv("teams_cleaned.csv", index=False)
print("✅ Cleaned team data saved as teams_cleaned.csv")

####🧩 Final Combined Dataset

**Context:**
Finally, all three datasets (Players, MVPs, and Teams) are merged on common keys (Player, Year, Team) to form a unified dataset for analysis.

**Code:**

players = pd.read_csv("players_cleaned.csv")
mvps = pd.read_csv("mvps_cleaned.csv")
teams = pd.read_csv("teams_cleaned.csv")

combined = players.merge(mvps, how="left", on=["Player", "Year"])
combined = combined.merge(teams, how="left", on=["Team", "Year"])

combined.fillna(0, inplace=True)
combined.to_csv("combined_data.csv", index=False)
print("✅ Final merged dataset saved as combined_data.csv")
