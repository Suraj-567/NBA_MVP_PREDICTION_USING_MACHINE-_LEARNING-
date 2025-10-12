# 🏀 NBA MVP Data Scraping & Cleaning

## 📘 Project Overview
This repository focuses on the **data scraping** and **data cleaning** phases of the NBA MVP Prediction project.  
We extract, organize, and clean data from **Basketball Reference** covering player statistics, MVP voting results, and team standings for all seasons between **1991 and 2025**.

The cleaned datasets serve as the foundation for the Machine Learning phase (handled separately).

---

## ⚙️ Phase 1 — Web Scraping

### 🧩 1. MVP Voting Data Scraping

**File:** `1_mvp_scrape.py`

**Purpose:**  
Scrape MVP voting results from Basketball Reference’s official awards pages for each year.

**Libraries Used:**
```python
import requests
import time
from pathlib import Path
Process:

Loops through each NBA season (1991–2025)

Fetches the MVP voting page for that season

Saves the HTML file locally in the mvp/ folder

Includes polite delays to avoid overloading the server

Example Output:

yaml
Copy code
Fetching data for the year 1991...
Successfully saved 1991.html
...
Successfully saved 2025.html
Output Folder:
/mvp/ → contains all yearly HTML files (e.g., 1991.html, 1992.html, ...)

🧩 2. MVP Data Extraction
File: 2_mvp_extract.py

Purpose:
Convert all saved MVP HTML files into a structured CSV dataset.

Libraries Used:

python
Copy code
from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
import os
Process:

Reads each .html file from /mvp/

Parses tables with BeautifulSoup

Adds a Year column

Combines all years into a single DataFrame

Output File:
mvps.csv

Sample Columns:

mathematica
Copy code
Player | Year | Pts Won | Pts Max | Share | G | MP | TRB | AST | STL | BLK | FG% | 3P% | FT% | WS | WS/48
🧩 3. Player Statistics Scraping
File: 3_player_scrape_selenium.py

Purpose:
Scrape per-game statistics for all players from Basketball Reference for every season between 1991–2025.

Libraries Used:

python
Copy code
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import random
import os
Process:

Automates Chrome using Selenium

Visits each season’s player stats page

Switches to light mode for consistent formatting

Saves each page under /players/

Waits randomly between requests to mimic human browsing

Output Folder:
/players/ → contains all yearly HTML files (e.g., 1991.html, 1992.html, ...)

Example Output:

yaml
Copy code
Fetching data for 1991... ✅ Saved 1991
...
Fetching data for 2025... ✅ Saved 2025
🧩 4. Player Data Extraction
File: 4_player_extract.py

Purpose:
Parse the HTML files containing per-game player statistics and export them to a CSV file.

Libraries Used:

python
Copy code
from bs4 import BeautifulSoup
import pandas as pd
import io
import os
Process:

Reads each file from /players/

Removes duplicate header rows (caused by scrolling tables)

Extracts player stats tables

Adds a Year column

Combines all years into one DataFrame

Output File:
players.csv

Sample Columns:

sql
Copy code
Player | Age | Team | Pos | G | GS | MP | FG | FGA | FG% | 3P | 3PA | 3P% | AST | STL | BLK | PTS | Awards | Year
🧩 5. Team Standings Scraping
File: 5_team_scrape.py

Purpose:
Scrape NBA team standings for each season between 1991 and 2025.

Libraries Used:

python
Copy code
import requests
import time
import os
Process:

Fetches each year’s team standings page

Saves HTML files under /team/

Uses 3-second delays for safe scraping

Output Folder:
/team/ → contains all yearly HTML files (e.g., 1991.html, 1992.html, ...)

Example Output:

yaml
Copy code
Fetching standings for 1991... ✅ Saved 1991.html
Fetching standings for 1992... ✅ Saved 1992.html
🧩 6. Team Standings Extraction
File: 6_team_extract.py

Purpose:
Parse the team standings HTML files and consolidate both Eastern and Western conference data into one dataset.

Libraries Used:

python
Copy code
from bs4 import BeautifulSoup
import pandas as pd
import io
import os
Process:

Reads both divs_standings_E and divs_standings_W tables

Standardizes team names and columns

Removes header rows and unnecessary footnotes

Adds Year column

Concatenates all years into one file

Output File:
teams.csv

Sample Columns:

sql
Copy code
Team | W | L | W/L% | GB | PS/G | PA/G | SRS | Year
🧹 Phase 2 — Data Cleaning
🔧 Cleaning Player Data
File: data_cleaning.py

Steps:

python
Copy code
import pandas as pd

players = pd.read_csv("players.csv")
players.drop(columns=["Unnamed: 0", "Rk"], errors="ignore", inplace=True)
players["Player"] = players["Player"].str.replace("*", "", regex=False)
players = players.groupby(["Player", "Year"], as_index=False).mean(numeric_only=True)
players.to_csv("players_cleaned.csv", index=False)
Result:

Removed unnecessary columns

Cleaned player names (removed *)

Merged duplicate entries (e.g., traded players)

Saved cleaned output as players_cleaned.csv

🔧 Cleaning MVP Data
Steps:

python
Copy code
mvps = pd.read_csv("mvps.csv")
mvps = mvps[["Player", "Year", "Pts Won", "Pts Max", "Share"]]
mvps.fillna(0, inplace=True)
mvps.to_csv("mvps_cleaned.csv", index=False)
Result:

Retained only key voting-related columns

Replaced missing values with 0

Saved cleaned output as mvps_cleaned.csv

🔧 Cleaning Team Data
Steps:

python
Copy code
teams = pd.read_csv("teams.csv")
teams = teams[~teams["Team"].str.contains("Division", na=False)]
teams["Team"] = teams["Team"].str.replace("*", "", regex=False)
teams.to_csv("teams_cleaned.csv", index=False)
Result:

Removed division headers and footnotes

Cleaned team names (removed *)

Saved cleaned output as teams_cleaned.csv

🧩 Final Combined Dataset
Merging Datasets:

python
Copy code
combined = players.merge(mvps, how="outer", on=["Player", "Year"])
combined = combined.merge(teams, how="left", on=["Team", "Year"])
combined.to_csv("combined.csv", index=False)
Result:
A unified dataset containing:

sql
Copy code
Player | Year | Team | Pos | Age | G | PTS | AST | STL | BLK | WS | Share | W | L | W/L% | SRS
📂 Output Summary
File Name	Description
/mvp/*.html	Raw MVP voting HTML pages
/players/*.html	Raw player per-game statistics HTML pages
/team/*.html	Raw team standings HTML pages
mvps_cleaned.csv	Cleaned MVP voting data
players_cleaned.csv	Cleaned player performance data
teams_cleaned.csv	Cleaned team standings data
combined.csv	Final merged dataset (ready for Machine Learning phase)
