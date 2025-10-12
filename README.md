🏀 NBA MVP Data Collection & Cleaning
📘 Overview

This repository contains all scripts required to scrape, extract, and clean NBA player statistics, MVP voting results, and team standings data from Basketball Reference
.

These cleaned datasets serve as the foundation for the Machine Learning phase, which is handled in a separate part of the project.

⚙️ PHASE 1 — WEB SCRAPING
🧩 1. MVP Voting Data Scraping

Script: 1_mvp_scrape.py

Goal: Scrape the NBA MVP voting results from Basketball Reference for all seasons between 1991 and 2025.

Libraries Used:

import requests
import time


Process:

Iterates over every NBA season.

Fetches each year’s MVP voting page.

Stores HTML content locally in /mvp/ directory for offline use.

Adds a short delay between requests to avoid server overload.

Sample Code:

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/awards/awards_{year}.html"
    response = requests.get(url)
    with open(f"mvp/{year}.html", "w", encoding="utf-8") as f:
        f.write(response.text)
    print(f"✅ Saved MVP data for {year}")
    time.sleep(3)


Output Example:

✅ Saved MVP data for 1991
✅ Saved MVP data for 1992
...
✅ Saved MVP data for 2025

🧩 2. Extracting MVP Data from HTML

Script: 2_mvp_extract.py

Goal: Parse the MVP HTML files and extract tabular data into a clean CSV file.

Libraries Used:

from bs4 import BeautifulSoup
import pandas as pd
import os, io


Process:

Reads all saved .html files from /mvp/

Uses BeautifulSoup to parse the HTML tables

Cleans the data and adds a Year column

Concatenates all years into one dataset

Output: mvps.csv

Key Columns:
["Player", "Age", "Tm", "First Place Votes", "Pts Won", "Pts Max", "Share", "Year"]

🧩 3. Player Per-Game Stats Scraping

Script: 3_player_scrape_selenium.py

Goal: Scrape player per-game stats for every season from 1991 to 2025.

Libraries Used:

from selenium import webdriver
import time, random


Process:

Automates browser navigation using Selenium.

Opens each season’s stats page from Basketball Reference.

Switches to light mode (for uniform scraping).

Saves each year’s player statistics page under /players/.

Code Snippet:

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_per_game.html"
    driver.get(url)
    time.sleep(random.uniform(3, 6))
    html_source = driver.page_source
    with open(f"players/{year}.html", "w", encoding="utf-8") as f:
        f.write(html_source)
    print(f"✅ Saved player stats for {year}")


Output Example:

✅ Saved player stats for 1991
✅ Saved player stats for 1992
...
✅ Saved player stats for 2025

🧩 4. Extracting Player Stats

Script: 4_player_extract.py

Goal: Parse saved player stats HTML pages into a structured DataFrame.

Libraries Used:

from bs4 import BeautifulSoup
import pandas as pd
import os, io


Process:

Loads every .html file from /players/

Removes repeated table headers and blank rows

Extracts player statistics

Adds Year column and concatenates all years

Output: players.csv

Key Columns:
["Player", "Pos", "Age", "Tm", "G", "GS", "MP", "FG%", "3P%", "FT%", "TRB", "AST", "STL", "BLK", "PTS", "Year"]

🧩 5. Team Standings Scraping

Script: 5_team_scrape.py

Goal: Download team standings for all NBA seasons between 1991–2025.

Libraries Used:

import requests
import os
import time


Process:

Iterates over years and fetches division standings.

Saves HTML files for each season to /team/.

Includes polite delays between requests.

Code Snippet:

for year in range(1991, 2026):
    url = f"https://www.basketball-reference.com/leagues/NBA_{year}_standings.html"
    r = requests.get(url)
    with open(f"team/{year}.html", "w", encoding="utf-8") as f:
        f.write(r.text)
    print(f"✅ Saved standings for {year}")
    time.sleep(3)


Output Example:

✅ Saved standings for 1991
✅ Saved standings for 1992
...
✅ Saved standings for 2025

🧩 6. Extracting Team Standings

Script: 6_team_extract.py

Goal: Parse and clean team standings data for both conferences.

Libraries Used:

import pandas as pd
from bs4 import BeautifulSoup
import os, io


Process:

Reads each /team/{year}.html file

Extracts both Eastern and Western Conference tables

Cleans asterisks and notes from team names

Adds Year column and merges conference tables

Output: teams.csv

Key Columns:
["Team", "W", "L", "W/L%", "GB", "PS/G", "PA/G", "SRS", "Year"]

🧹 PHASE 2 — DATA CLEANING

Script: data_cleaning.py

🔧 1. Player Data Cleaning

Steps:

Removed unnecessary columns (Unnamed: 0, Rk, etc.)

Removed * from player names (used by Basketball Reference for All-Stars)

Merged duplicate player entries (e.g., traded players with multiple teams)

Filled missing or inconsistent values with 0 or NaN appropriately

Final Output: players_cleaned.csv

🔧 2. MVP Data Cleaning

Steps:

Retained only essential MVP fields:

mvps = mvps[["Player", "Year", "Pts Won", "Pts Max", "Share"]]


Replaced missing shares with 0 for non-nominated players.

Ensured data consistency with player names and seasons.

Final Output: mvps_cleaned.csv

🔧 3. Team Data Cleaning

Steps:

Removed duplicated rows (due to multiple divisions)

Cleaned asterisks in team names

Removed invalid division headers like "Division", "Team" etc.

Normalized names for matching with player data (e.g., L.A. Lakers → Los Angeles Lakers)

Final Output: teams_cleaned.csv

🔧 4. Combining Datasets

Goal: Merge player, MVP, and team data for analysis and machine learning.

Merge Logic:

combined = players.merge(mvps, how="left", on=["Player", "Year"])
combined = combined.merge(teams, how="left", on=["Team", "Year"])


Final Output: combined_data.csv

Columns Include:
[Player, Team, Pos, Age, G, PTS, AST, STL, BLK, WS, Share, W, L, SRS, Year]

📊 FINAL OUTPUT FILES SUMMARY
File Name	Description
/mvp/*.html	Raw MVP voting HTML pages
/players/*.html	Raw player per-game stats
/team/*.html	Raw team standings pages
mvps.csv	Extracted MVP data
players.csv	Extracted player stats
teams.csv	Extracted team standings
players_cleaned.csv	Cleaned player data
mvps_cleaned.csv	Cleaned MVP voting data
teams_cleaned.csv	Cleaned team standings
combined_data.csv	Fully merged dataset (ready for ML)
