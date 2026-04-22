# Data Dictionary — IPL Complete Dataset

**Project:** IPL Performance Analytics — Capstone 2  
**Source:** Kaggle — IPL Complete Dataset (Raw)  
**Last Updated:** April 2026

---

## File 1: matches.csv

**Shape:** 1,095 rows × 20 columns (+ 1 derived column after ETL)  
**Grain:** One row per IPL match

| Column | Raw Type | Cleaned Type | Description | Nulls (Raw) | ETL Action |
|--------|----------|--------------|-------------|-------------|------------|
| `id` | int64 | int64 | Unique match identifier | 0 | No change |
| `season` | object | int64 | IPL season year | 0 | Standardised: `2007/08`→`2008`, `2009/10`→`2010`, `2020/21`→`2021` |
| `city` | object | object | City where match was played | 51 | Imputed from venue name lookup (UAE matches) |
| `date` | object | datetime64 | Match date | 0 | Parsed from string |
| `match_type` | object | object | League / Playoff / Final | 0 | No change |
| `player_of_match` | object | object | Player of the match award | 5 | Kept null — no result matches |
| `venue` | object | object | Full stadium name | 0 | No change |
| `team1` | object | object | First competing team | 0 | Renamed: 4 franchise renames applied |
| `team2` | object | object | Second competing team | 0 | Renamed: 4 franchise renames applied |
| `toss_winner` | object | object | Team winning the coin toss | 0 | Renamed: 4 franchise renames applied |
| `toss_decision` | object | object | `field` or `bat` after toss | 0 | No change |
| `winner` | object | object | Match-winning team | 5 | Kept null — no result / tied matches |
| `result` | object | object | `runs` / `wickets` / `tie` / `no result` | 0 | No change |
| `result_margin` | float64 | float64 | Margin of victory (runs or wickets) | 19 | Kept null — ties/no results have no margin |
| `target_runs` | float64 | float64 | Runs target set for 2nd innings | 3 | Kept null — edge cases |
| `target_overs` | float64 | float64 | Overs for 2nd innings (DLS adjusted) | 3 | Kept null — edge cases |
| `super_over` | object | object | `Y` if match went to super over | 0 | No change |
| `method` | object | object | `Normal` or `D/L` | 1,074 | Filled null → `Normal` (DLS not applied) |
| `umpire1` | object | object | First on-field umpire | 0 | No change |
| `umpire2` | object | object | Second on-field umpire | 0 | No change |
| `toss_win_match` | — | int64 | **DERIVED** — 1 if toss winner won match | — | Created: `toss_winner == winner` |

### Team Name Standardisation Map

| Old Name | Standardised Name | Reason |
|----------|------------------|--------|
| Delhi Daredevils | Delhi Capitals | Franchise rebrand (2019) |
| Kings XI Punjab | Punjab Kings | Franchise rebrand (2021) |
| Rising Pune Supergiant | Rising Pune Supergiants | Spelling inconsistency |
| Royal Challengers Bangalore | Royal Challengers Bengaluru | City name update |

---

## File 2: deliveries.csv

**Shape:** 260,920 rows × 17 columns (+ 2 derived columns after ETL)  
**Grain:** One row per ball delivered in every IPL match

| Column | Raw Type | Cleaned Type | Description | Nulls (Raw) | ETL Action |
|--------|----------|--------------|-------------|-------------|------------|
| `match_id` | int64 | int64 | Foreign key → matches.id | 0 | No change |
| `inning` | int64 | int64 | Innings number (1 or 2; 3+ = super over) | 0 | No change |
| `batting_team` | object | object | Team currently batting | 0 | Renamed: 4 franchise renames applied |
| `bowling_team` | object | object | Team currently bowling | 0 | Renamed: 4 franchise renames applied |
| `over` | int64 | int64 | Over number (0-indexed: 0 = over 1) | 0 | No change |
| `ball` | int64 | int64 | Ball number within the over | 0 | No change |
| `batter` | object | object | Name of the batsman on strike | 0 | No change |
| `bowler` | object | object | Name of the bowler | 0 | No change |
| `non_striker` | object | object | Name of batsman at non-striker end | 0 | No change |
| `batsman_runs` | int64 | int64 | Runs scored off the bat (0–6) | 0 | No change |
| `extra_runs` | int64 | int64 | Extra runs (wide, no-ball, bye, leg-bye) | 0 | No change |
| `total_runs` | int64 | int64 | Total runs on this delivery (bat + extras) | 0 | No change |
| `extras_type` | object | object | Type of extra (`wide`, `noball`, `bye`, `legbyes`, `penalty`) | 246,795 | Filled null → `none` (no extra occurred) |
| `is_wicket` | int64 | int64 | Binary: 1 if dismissal occurred on this ball | 0 | No change |
| `player_dismissed` | object | object | Name of dismissed batsman | 247,970 | Filled null → `not_out` |
| `dismissal_kind` | object | object | Type of dismissal (`caught`, `bowled`, `run out`, `lbw`, `stumped`, etc.) | 247,970 | Filled null → `not_out` |
| `fielder` | object | object | Fielder involved in the dismissal | 251,566 | Filled null → `none` |
| `phase` | — | object | **DERIVED** — `Powerplay` / `Middle` / `Death` | — | Created from `over`: 0–5 = Powerplay, 6–14 = Middle, 15–19 = Death |
| `is_super_over_match` | — | bool | **DERIVED** — True if match went to super over | — | Created from matches.super_over == 'Y' |

### Phase Definition

| Phase | Overs (0-indexed) | Overs (1-indexed) | Description |
|-------|------------------|------------------|-------------|
| Powerplay | 0–5 | 1–6 | Mandatory field restrictions apply |
| Middle | 6–14 | 7–15 | Consolidation/acceleration phase |
| Death | 15–19 | 16–20 | Final assault — maximum hitting |

---

## KPI Output Files (data/processed/)

### kpi_teams.csv
Columns: `team`, `total_matches`, `wins`, `win_rate`

### kpi_batters.csv
Columns: `batter`, `balls`, `batsman_runs`, `strike_rate`  
Filter: minimum 200 balls faced

### kpi_bowlers.csv
Columns: `bowler`, `balls`, `total_runs`, `economy`, `wickets`  
Filter: minimum 200 balls bowled

### kpi_phases.csv
Columns: `phase`, `boundaries`, `balls`, `boundary_pct`

### kpi_seasons.csv
Columns: `season`, `inn1` (avg), `inn2` (avg), `run_diff` (avg)

---

*Data Dictionary maintained by: Data Analytics Capstone Team | April 2026*
