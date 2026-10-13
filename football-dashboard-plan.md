# Football Player Performance Analytics Dashboard — Implementation Plan

## Top-Level Overview

Build a single-file, portfolio-quality interactive football (soccer) analytics dashboard using Python and Streamlit.
The application loads a synthetic CSV dataset of 100+ players, applies sidebar filters, and renders four pages of
analysis with Plotly charts, KPI cards, a searchable stats table, and a CSV download button.
The UI uses a dark, football-inspired green pitch theme.

All four deliverables — `app.py`, `football_players.csv`, `requirements.txt`, `README.md` — live at the project
root. No external APIs or databases are required.

---

## Project Structure

```
football-analysis-dashboard/
├── app.py                  # Single Streamlit application (all logic, pages, charts)
├── football_players.csv    # Synthetic dataset (generated once and committed)
├── requirements.txt        # Python dependencies
└── README.md               # Setup, run, and deploy instructions
```

---

## Dataset Schema

File: `football_players.csv`

| Column           | Type    | Description                                         |
|------------------|---------|-----------------------------------------------------|
| player_id        | int     | Unique identifier (1–N)                             |
| player_name      | string  | Synthetic full name (First Last)                    |
| team             | string  | Club name (8 synthetic teams)                       |
| position         | string  | GK / DEF / MID / FWD                               |
| season           | string  | "2022/23" or "2023/24"                              |
| age              | int     | 17–36                                               |
| nationality      | string  | One of ~15 countries                                |
| matches_played   | int     | 1–38                                                |
| minutes_played   | int     | matches_played × 60–95 (randomised per match)       |
| goals            | int     | Position-weighted random (FWD > MID > DEF > GK)    |
| assists          | int     | Position-weighted random                            |
| shots_on_target  | int     | goals ≤ shots_on_target ≤ goals + 10               |
| key_passes       | int     | 0–80                                                |
| pass_accuracy    | float   | 60–95 %                                             |
| tackles          | int     | Position-weighted (DEF > MID > FWD > GK)           |
| yellow_cards     | int     | 0–10                                                |
| red_cards        | int     | 0–3                                                 |
| rating           | float   | Standalone random 5.0–9.5 (rounded to 1 dp)        |

Minimum rows: 200 (100 players × 2 seasons). Seed is fixed (numpy seed 42) for reproducibility.
File is labelled synthetic in the header comment and in the dashboard UI.

---

## KPI Calculations

All KPIs are computed from the **filtered** DataFrame (after sidebar filters are applied):

| KPI Card                | Formula                                              |
|-------------------------|------------------------------------------------------|
| Total Players           | `df['player_id'].nunique()`                          |
| Total Goals             | `df['goals'].sum()`                                  |
| Total Assists           | `df['assists'].sum()`                                |
| Total Minutes Played    | `df['minutes_played'].sum()`                         |
| Average Player Rating   | `df['rating'].mean()` rounded to 2 dp               |

---

## Sub-Tasks

---

### Sub-Task 1 — Generate the Synthetic Dataset (`football_players.csv`)

**Intent**
Create a committed, reproducible CSV file that all dashboard pages read from.
Labelling it synthetic prevents any confusion with real-world statistics.

**Expected Outcomes**
- `football_players.csv` exists at the project root.
- Contains ≥ 200 rows (100 players × 2 seasons).
- All column types are correct; no nulls.
- A comment block at the top of the generation script (inline in `app.py`) documents the seed and schema.

**Todo List**
1. Write a `generate_dataset()` function in `app.py` that uses `numpy` seed 42.
2. Define 8 team names, 4 position codes, 2 season strings, ~15 nationalities, and a first/last name pool.
3. For each of 100 players assign a fixed team, position, age, and nationality.
4. Duplicate the player list for the second season and randomise per-season stats independently.
5. Apply position-weighted distributions for goals, assists, tackles (FWD scores most, GK scores fewest).
6. Ensure `shots_on_target >= goals` for every row.
7. Round `pass_accuracy` to 1 dp and `rating` to 1 dp.
8. Save to `football_players.csv` and add a `# Synthetic data — do not treat as real results` header note in README.

**Relevant Context**
- No external data source; pure `numpy`/`pandas` generation.
- Called once at startup with a guard: `if not os.path.exists('football_players.csv'): generate_dataset()`.

**Status**: [ ] pending

---

### Sub-Task 2 — Application Skeleton and Sidebar Filters (`app.py` scaffold)

**Intent**
Set up the Streamlit app entry point, global page config, custom CSS theme, data loader, and sidebar
filter widgets so that every subsequent page sub-task can rely on a filtered DataFrame.

**Expected Outcomes**
- `st.set_page_config` configured with a football-themed title, icon, and wide layout.
- Custom CSS injected via `st.markdown(..., unsafe_allow_html=True)` that sets a dark background
  (`#0d1117`), accent green (`#00ff85`), and card styles.
- `load_data()` function reads the CSV, infers types, and returns a DataFrame.
- Sidebar renders: Season multiselect, Team multiselect, Position multiselect, Player multiselect (optional).
- A `filter_data(df, seasons, teams, positions, players)` function returns the filtered DataFrame.
- Input validation: if filters produce an empty DataFrame, `st.warning` is shown and execution stops.
- Page navigation is implemented with `st.sidebar.radio` (Overview / Player Analysis / Team Analysis / Position Analysis).

**Todo List**
1. Add `st.set_page_config(...)` as the very first Streamlit call.
2. Define and inject CSS string for dark theme, green accents, KPI card styles, table styles.
3. Write `load_data()` — reads CSV, caches with `@st.cache_data`.
4. Build sidebar: logo/title text, nav radio, then filter widgets in order (Season → Team → Position → Player).
5. Write `filter_data()` applying each active filter sequentially.
6. Add empty-result guard after filtering.
7. Dispatch to the correct page render function based on nav selection.

**Relevant Context**
- Streamlit version pinned in `requirements.txt` (≥ 1.32).
- `@st.cache_data` is the modern cache decorator (replaces deprecated `@st.cache`).
- CSS must not break Streamlit's own component styling; scope selectors carefully.

**Status**: [ ] pending

---

### Sub-Task 3 — Overview Page

**Intent**
Give the user an at-a-glance summary of the filtered dataset through KPI metric cards and a top-performers bar chart.

**Expected Outcomes**
- Five KPI cards render in a single `st.columns(5)` row: Total Players, Total Goals, Total Assists,
  Total Minutes Played, Average Rating.
- A horizontal bar chart shows the Top 10 players by goals (filtered data).
- A second horizontal bar chart shows the Top 10 players by rating.
- All values derived from the filtered DataFrame (no hardcoding).
- Charts use the green accent palette and dark background to match the theme.
- A prominent "⚠️ Synthetic Data" disclaimer banner appears below the page heading.

**Todo List**
1. Create `render_overview(df)` function.
2. Use `st.columns(5)` and `st.metric()` for KPI cards; wrap each in a styled `div` for card effect.
3. Build Top-10 goals bar chart with `px.bar` (sorted descending, colour scale green).
4. Build Top-10 rating bar chart with `px.bar`.
5. Add disclaimer banner using a styled `st.markdown` block.

**Relevant Context**
- `st.metric` supports `delta` — leave delta empty for this dashboard (no period-on-period comparison needed).
- Plotly `px.bar` with `orientation='h'` gives horizontal bars.

**Status**: [ ] pending

---

### Sub-Task 4 — Player Analysis Page

**Intent**
Allow the user to deep-dive into individual player performance through a comparison selector,
scatter plots, and a searchable stats table.

**Expected Outcomes**
- A player comparison widget lets the user pick two players and renders a side-by-side stat table.
- A Goals vs Assists scatter plot shows all filtered players (colour = position).
- A Shots on Target vs Goals scatter plot with a trend line.
- A searchable, sortable player stats table (`st.dataframe` with `st.text_input` search).
- A CSV download button for the filtered and searched data.

**Todo List**
1. Create `render_player_analysis(df)` function.
2. Player comparison: two `st.selectbox` widgets, build a transposed comparison DataFrame, display with `st.table`.
3. Goals vs Assists scatter: `px.scatter` with `color='position'`, hover showing player name, team, rating.
4. Shots vs Goals scatter: `px.scatter` with `trendline='ols'` (requires `statsmodels` — add to requirements).
5. Search input filters player name column (case-insensitive `str.contains`).
6. Display filtered table with `st.dataframe(use_container_width=True)`.
7. `st.download_button` serialises the visible DataFrame to CSV via `df.to_csv(index=False)`.

**Relevant Context**
- `trendline='ols'` in Plotly Express requires `statsmodels` to be installed.
- Download button MIME type: `text/csv`.
- Search box should be placed directly above the table.

**Status**: [ ] pending

---

### Sub-Task 5 — Team Analysis Page

**Intent**
Aggregate player stats to team level and surface team-level goals, assists, and squad size comparisons.

**Expected Outcomes**
- A grouped bar chart comparing total goals and total assists per team.
- A bar chart showing number of players per team (squad depth).
- A radar/spider chart comparing average stats across teams for up to five selected teams.
- Team summary table with totals and averages.

**Todo List**
1. Create `render_team_analysis(df)` function.
2. Aggregate `df.groupby('team')` to get sum of goals, assists, minutes, mean rating, and player count.
3. Grouped bar chart: `px.bar` with `barmode='group'`, goals and assists as two series.
4. Squad depth bar chart: `px.bar` on player counts per team.
5. Radar chart: use `go.Scatterpolar` — allow user to select up to 5 teams via `st.multiselect`.
   Normalise each stat (0–1) before plotting to keep axes comparable.
6. Display team summary `st.dataframe`.

**Relevant Context**
- `go.Scatterpolar` is in `plotly.graph_objects`; import as `go`.
- Normalisation: `(val - min) / (max - min)` per column across all teams.

**Status**: [ ] pending

---

### Sub-Task 6 — Position Analysis Page

**Intent**
Compare performance profiles across the four positions (GK, DEF, MID, FWD) to highlight role-specific patterns.

**Expected Outcomes**
- A box plot of rating distribution per position.
- A bar chart of average goals, assists, and tackles per position.
- A scatter plot of pass accuracy vs key passes, coloured by position.
- A table of position averages for all numeric columns.

**Todo List**
1. Create `render_position_analysis(df)` function.
2. Rating box plot: `px.box(df, x='position', y='rating', color='position')`.
3. Average stats bar chart: aggregate by position, melt for multi-series `px.bar`.
4. Pass accuracy vs key passes scatter: `px.scatter(color='position')`.
5. Position averages table: `df.groupby('position').mean(numeric_only=True).round(2)` displayed with `st.dataframe`.

**Relevant Context**
- `pd.melt` reshapes wide aggregation into a long format suitable for grouped Plotly bar charts.

**Status**: [ ] pending

---

### Sub-Task 7 — `requirements.txt` and `README.md`

**Intent**
Provide exact dependencies and clear instructions so any developer can run or deploy the dashboard.

**Expected Outcomes**
- `requirements.txt` lists all packages with minimum version pins.
- `README.md` includes: project description, dataset disclaimer, local setup steps, Streamlit Cloud deploy steps, screenshot placeholder, and feature list.

**Todo List**
1. Write `requirements.txt`:
   ```
   streamlit>=1.32.0
   pandas>=2.0.0
   numpy>=1.26.0
   plotly>=5.20.0
   statsmodels>=0.14.0
   ```
2. Write `README.md` with sections: Overview, Dataset, Features, Setup, Running Locally, Deploying to Streamlit Cloud, Contributing.
3. Add synthetic data disclaimer prominently in README.

**Relevant Context**
- Streamlit Cloud deployment requires `requirements.txt` at the repo root and `app.py` as the entry point.
- No `Procfile` or Docker configuration needed for Streamlit Cloud.

**Status**: [ ] pending

---

## Architecture Diagram

```
app.py
│
├── generate_dataset()         — creates football_players.csv if missing
├── load_data()                — @st.cache_data CSV reader
├── filter_data()              — applies sidebar filter selections
│
├── Sidebar
│   ├── Navigation radio
│   └── Filters: Season / Team / Position / Player
│
└── Pages (dispatched by nav)
    ├── render_overview()
    ├── render_player_analysis()
    ├── render_team_analysis()
    └── render_position_analysis()
```

---

## Open Questions / Decisions Already Made

| Decision | Choice |
|---|---|
| Code structure | Single `app.py` file |
| Player rating | Standalone random float 5.0–9.5 |
| Dataset size | 200 rows (100 players × 2 seasons) |
| Data source | Synthetic only — no real API |
| Trend line library | `statsmodels` (OLS via Plotly Express) |
