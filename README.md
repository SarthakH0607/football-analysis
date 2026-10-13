# ⚽ Football Player Performance Analytics Dashboard

A portfolio-quality interactive analytics dashboard built with Python and Streamlit.

> ⚠️ **Synthetic Data Notice:** All player statistics in this dashboard are randomly generated for demonstration purposes. They do not represent real football players, clubs, or results.

---

## Features

- **Overview KPI cards** — Total players, goals, assists, minutes played, and average rating, all computed from the active filter selection.
- **Player comparison widget** — Pick any two players from the dataset and view a side-by-side stat breakdown.
- **Goals vs Assists scatter plot** — All filtered players plotted by position with hover tooltips.
- **Shots on Target vs Goals scatter plot** — Includes an OLS trend line powered by Statsmodels.
- **Team-level goals, assists & squad depth charts** — Grouped bar chart and squad-size bar chart aggregated per club.
- **Team radar comparison chart** — Compare up to 5 teams across normalised performance dimensions.
- **Position-level box plots, bar charts & scatter plots** — Visualise rating distributions, average stats, and pass accuracy by position (GK / DEF / MID / FWD).
- **Sidebar filters** — Filter by season, team, position, and individual player; all charts update instantly.
- **Searchable player statistics table** — Case-insensitive search above a sortable, full-width dataframe.
- **CSV download button** — Export the current filtered and searched dataset in one click.
- **Dark, football-inspired green pitch theme** — Custom CSS with dark background (`#0d1117`) and accent green (`#00ff85`).

---

## Project Structure

```
football-analysis-dashboard/
├── app.py                  # Main Streamlit application
├── football_players.csv    # Synthetic dataset (200 rows)
├── requirements.txt        # Python dependencies
└── README.md               # This file
```

---

## Dataset Schema

`football_players.csv` contains **200 rows** (100 synthetic players × 2 seasons). The file is generated once at startup using NumPy seed 42 for full reproducibility.

| Column           | Type   | Description                                              |
|------------------|--------|----------------------------------------------------------|
| `player_id`      | int    | Unique player identifier (1–100)                         |
| `player_name`    | string | Synthetic full name (First Last)                         |
| `team`           | string | Club name — one of 8 synthetic teams                     |
| `position`       | string | Playing position: `GK`, `DEF`, `MID`, or `FWD`          |
| `season`         | string | Season label: `"2022/23"` or `"2023/24"`                 |
| `age`            | int    | Player age between 17 and 36                             |
| `nationality`    | string | One of ~15 synthetic nationalities                       |
| `matches_played` | int    | Matches played in the season (1–38)                      |
| `minutes_played` | int    | Total minutes — `matches_played × 60–95` (randomised)   |
| `goals`          | int    | Position-weighted goals: FWD > MID > DEF > GK            |
| `assists`        | int    | Position-weighted assists                                |
| `shots_on_target`| int    | Always ≥ `goals`; capped at `goals + 10`                 |
| `key_passes`     | int    | Key passes in the season (0–80)                          |
| `pass_accuracy`  | float  | Pass accuracy percentage (60–95 %, 1 d.p.)               |
| `tackles`        | int    | Position-weighted tackles: DEF > MID > FWD > GK          |
| `yellow_cards`   | int    | Yellow cards received (0–10)                             |
| `red_cards`      | int    | Red cards received (0–3)                                 |
| `rating`         | float  | Overall player rating (5.0–9.5, 1 d.p.)                  |

---

## Local Setup

### Prerequisites

- Python 3.10 or later
- `pip` and `venv` (included with standard Python)

### Steps

**1. Clone or download the repository**

```bash
git clone https://github.com/your-username/football-analysis-dashboard.git
cd football-analysis-dashboard
```

**2. Create and activate a virtual environment**

```bash
# Create
python -m venv venv

# Activate — macOS / Linux
source venv/bin/activate

# Activate — Windows (PowerShell)
venv\Scripts\Activate.ps1
```

**3. Install dependencies**

```bash
pip install -r requirements.txt
```

**4. Run the app**

```bash
streamlit run app.py
```

**5. Open in your browser**

Navigate to [http://localhost:8501](http://localhost:8501). The synthetic dataset is generated automatically on the first run — no manual setup required.

---

## Deploying to Streamlit Cloud

Streamlit Community Cloud is the simplest zero-config deployment option for this app.

1. **Push the project to a GitHub repository** — ensure `app.py` and `requirements.txt` are at the root.
2. **Go to [https://share.streamlit.io](https://share.streamlit.io)** and sign in.
3. **Connect your GitHub account** if you haven't already.
4. **Create a new app** — select your repository and branch, and set the main file path to `app.py`.
5. **Click Deploy** — the app will build and go live in approximately 2 minutes.
6. **No environment variables are needed** — the app is fully self-contained.

---

## Technologies Used

| Library      | Version | Purpose                              |
|--------------|---------|--------------------------------------|
| Streamlit    | ≥ 1.32  | Web app framework & UI components    |
| Pandas       | ≥ 2.0   | Data loading, filtering, aggregation |
| NumPy        | ≥ 1.26  | Synthetic data generation            |
| Plotly       | ≥ 5.20  | Interactive charts and radar plots   |
| Statsmodels  | ≥ 0.14  | OLS trend line in scatter plots      |

---

## Screenshots

> 📸 *Screenshots coming soon — run the app locally to explore all pages.*

---

## License

MIT License — free for personal and commercial use.
