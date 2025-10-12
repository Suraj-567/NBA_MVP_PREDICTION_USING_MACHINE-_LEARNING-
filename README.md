🏀 NBA MVP Dataset — Web Scraping & Data Cleaning
📘 Project Overview

This repository focuses on data collection and cleaning for the NBA MVP Prediction project.
The data is scraped directly from Basketball Reference
 for the seasons 1991–2025, covering:

MVP voting results

Player per-game statistics

Team standings

After scraping, all datasets are cleaned, standardized, and merged to prepare for later machine learning analysis.

🕸️ Phase 1 — Web Scraping
🎯 Objective

To collect raw data for:

MVP Voting Results

Player Per-Game Stats

Team Standings

All data is saved locally in structured folders (/mvp, /players, /team).

🧩 1. Scraping MVP Voting Data
📝 Context

This script fetches the MVP voting tables for every season between 1991 and 2025 from Basketball Reference’s awards pages.
Each page is saved as an HTML file inside the mvp/ folder for later parsing.

💻 Code
# ---------- MVP Scraping Script ----------
import requests
import os
import time

os.makedirs("mvp", exist_ok=True)

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/awards/awards_{year}.html"
    response = requests.get(url)
    if response.status_code == 200:
        with open(f"mvp/{year}.html", "w", encoding="utf-8") as f:
            f.write(response.text)
        print(f"✅ Saved {year}.html")
    else:
        print(f"❌ Failed for {year}")
    time.sleep(3)

🧩 2. Extracting MVP Data from HTML
📝 Context

This script parses all the scraped HTML files, extracts MVP tables, and combines them into a single CSV (mvps.csv).
It uses BeautifulSoup and Pandas for parsing and tabular data handling.

💻 Code
# ---------- MVP Extraction Script ----------
import pandas as pd
from bs4 import BeautifulSoup
from io import StringIO
from pathlib import Path

mvp_folder = Path("mvp")
mvp_data = []

for file in mvp_folder.glob("*.html"):
    year = int(file.stem)
    with open(file, "r", encoding="utf-8") as f:
        html = f.read()

    soup = BeautifulSoup(html, "html.parser")
    table = soup.find("table")
    if table:
        df = pd.read_html(StringIO(str(table)))[0]
        df["Year"] = year
        mvp_data.append(df)

mvps = pd.concat(mvp_data, ignore_index=True)
mvps.to_csv("mvps.csv", index=False)
print("✅ MVP Data Extracted and Saved to mvps.csv")

🧩 3. Scraping Player Per-Game Stats
📝 Context

This Selenium script automatically opens each year’s NBA per-game statistics page and saves it as an HTML file.
Light mode is enforced for consistent formatting across seasons.

💻 Code
# ---------- Player Scraping Script ----------
from selenium import webdriver
import os, time, random

os.makedirs("players", exist_ok=True)

driver = webdriver.Chrome()

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_per_game.html"
    driver.get(url)
    driver.execute_script("document.querySelector('body').classList.remove('dark-mode');")
    time.sleep(random.uniform(2.5, 4.5))

    with open(f"players/{year}.html", "w", encoding="utf-8") as f:
        f.write(driver.page_source)
    print(f"✅ Saved player stats for {year}")

driver.quit()

🧩 4. Extracting Player Stats from HTML
📝 Context

This script reads all the saved player per-game HTML files, cleans up redundant headers, and merges them into a single CSV file (players.csv).

💻 Code
# ---------- Player Extraction Script ----------
import pandas as pd
from bs4 import BeautifulSoup
from io import StringIO
from pathlib import Path

player_folder = Path("players")
all_players = []

for file in player_folder.glob("*.html"):
    year = int(file.stem)
    with open(file, "r", encoding="utf-8") as f:
        html = f.read()

    soup = BeautifulSoup(html, "html.parser")
    table = soup.find("table", {"id": "per_game_stats"})
    if table:
        df = pd.read_html(StringIO(str(table)))[0]
        df = df[df["Rk"] != "Rk"]  # remove header repeats
        df["Year"] = year
        all_players.append(df)

players = pd.concat(all_players, ignore_index=True)
players.to_csv("players.csv", index=False)
print("✅ Player Data Extracted and Saved to players.csv")

🧩 5. Scraping Team Standings
📝 Context

This script collects division standings for each NBA season (1991–2025), saving them under /team/.

💻 Code
# ---------- Team Scraping Script ----------
import requests
import os
import time

os.makedirs("team", exist_ok=True)

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_standings.html"
    response = requests.get(url)
    if response.status_code == 200:
        with open(f"team/{year}.html", "w", encoding="utf-8") as f:
            f.write(response.text)
        print(f"✅ Saved team standings for {year}")
    else:
        print(f"❌ Failed for {year}")
    time.sleep(3)

🧩 6. Extracting Team Standings Data
📝 Context

This script parses both Eastern and Western Conference standings and saves the cleaned data into teams.csv.

💻 Code
# ---------- Team Extraction Script ----------
import pandas as pd
from bs4 import BeautifulSoup
from io import StringIO
from pathlib import Path

team_folder = Path("team")
all_teams = []

for file in team_folder.glob("*.html"):
    year = int(file.stem)
    with open(file, "r", encoding="utf-8") as f:
        html = f.read()
    soup = BeautifulSoup(html, "html.parser")

    for table_id in ["divs_standings_E", "divs_standings_W"]:
        table = soup.find("table", id=table_id)
        if table:
            df = pd.read_html(StringIO(str(table)))[0]
            df["Year"] = year
            all_teams.append(df)

teams = pd.concat(all_teams, ignore_index=True)
teams.to_csv("teams.csv", index=False)
print("✅ Team Standings Extracted and Saved to teams.csv")

🧹 Phase 2 — Data Cleaning
🎯 Objective

To clean, normalize, and merge the scraped MVP, player, and team data into a unified dataset ready for machine learning.

🧩 1. Player Data Cleaning
📝 Context

Players often appear multiple times per year (if traded).
We clean such duplicates, remove unnecessary columns, and standardize player names.

💻 Code
# ---------- Player Data Cleaning ----------
import pandas as pd

players = pd.read_csv("players.csv")

# Drop unwanted columns
players = players.drop(columns=["Rk"], errors="ignore")

# Remove '*' from player names
players["Player"] = players["Player"].str.replace("*", "", regex=False)

# Merge duplicate player-year entries
players = players.groupby(["Player", "Year"], as_index=False).mean(numeric_only=True)

players.to_csv("players_cleaned.csv", index=False)
print("✅ Cleaned player data saved as players_cleaned.csv")

🧩 2. MVP Data Cleaning
📝 Context

We focus on only the relevant MVP columns and ensure all missing values are filled for consistency.

💻 Code
# ---------- MVP Data Cleaning ----------
mvps = pd.read_csv("mvps.csv")

# Keep only important columns
mvps = mvps[["Player", "Year", "Pts Won", "Pts Max", "Share"]]

# Fill missing values
mvps.fillna(0, inplace=True)

mvps.to_csv("mvps_cleaned.csv", index=False)
print("✅ Cleaned MVP data saved as mvps_cleaned.csv")

🧩 3. Team Data Cleaning
📝 Context

Cleans and standardizes team standings data, removes unwanted characters, and ensures consistent column naming.

💻 Code
# ---------- Team Data Cleaning ----------
teams = pd.read_csv("teams.csv")

# Remove unwanted rows
teams = teams[~teams["Team"].str.contains("Division", na=False)]

# Remove '*' characters from team names
teams["Team"] = teams["Team"].str.replace("*", "", regex=False)

teams.to_csv("teams_cleaned.csv", index=False)
print("✅ Cleaned team data saved as teams_cleaned.csv")

🧩 4. Merging All Cleaned Data
📝 Context

Finally, we merge all three cleaned datasets — players, MVPs, and teams — into one combined dataset for later machine learning.

💻 Code
# ---------- Combine All Cleaned Data ----------
players = pd.read_csv("players_cleaned.csv")
mvps = pd.read_csv("mvps_cleaned.csv")
teams = pd.read_csv("teams_cleaned.csv")

# Merge player + MVP data
combined = players.merge(mvps, how="left", on=["Player", "Year"])

# Merge with team standings based on team name and year
combined = combined.merge(teams, how="left", on=["Team", "Year"])

combined.to_csv("combined_data.csv", index=False)
print("✅ Final combined dataset saved as combined_data.csv")

📂 Output Summary
File Name	Description
mvp/*.html	Raw MVP voting HTML files
players/*.html	Raw player per-game HTML files
team/*.html	Raw team standings HTML files
mvps.csv	Extracted MVP dataset
players.csv	Extracted player dataset
teams.csv	Extracted team dataset
*_cleaned.csv	Cleaned and standardized data
combined_data.csv	Final merged dataset for ML
