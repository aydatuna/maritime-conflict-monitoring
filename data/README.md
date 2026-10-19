# Data Management, Licensing & Schema

This directory contains local data assets, cached benchmarks, and data management documentation for the Maritime Conflict Situation Monitoring project.

---

## 1. Data Sources & Licensing Overview

| Dataset | Source | Access Method | Licensing / Redistribution Status |
|---|---|---|---|
| **Satellite Thermal Anomalies** | NASA FIRMS (VIIRS_SNPP_SP) | REST API (`api/area/csv/`) | Public domain (NASA Open Data Policy). Requires free MAP_KEY registration. |
| **Conflict News Articles** | The Guardian Open Platform | REST API (`/search`) | Free developer tier. Articles fetched live; subject to Guardian Terms of Service. |
| **Global News Tone & Events** | GDELT v2 DOC API | REST API (`api/v2/doc/doc`) | Open access archive; rate-limited public endpoint. |
| **Ground-Truth Conflict Events** | ACLED | OAuth REST API (`api/acled/read`) | **Restricted Redistribution**. Academic/non-commercial use permitted upon registration. Raw CSV redistribution is strictly prohibited by ACLED terms. |
| **Brent Crude Oil Prices** | ICE via Yahoo Finance (`BZ=F`) | `brent_2024.csv` (or `yfinance`) | Public financial market data for research & academic benchmarking. |

---

## 2. Restricted Datasets & Handling Policy

### ACLED Conflict Data (`data/acled_events_raw.csv`)
Under the [ACLED Terms of Use](https://acleddata.com/terms-of-use/), users are permitted to analyze and cite ACLED data for research purposes, but **public re-hosting, republishing, or bulk redistribution of raw ACLED data is prohibited**.

To comply with this licensing constraint:
1. `data/acled_events_raw.csv` is explicitly listed in `.gitignore` and **excluded from version control**.
2. Researchers wishing to reproduce live data collection can register for free API credentials at [acleddata.com/register](https://acleddata.com/register/).
3. Add your credentials to your local `.env` file:
   ```env
   ACLED_USER=your_registered_email
   ACLED_PASS=your_registered_password
   ```
4. When executed, the notebook automatically authenticates via OAuth, queries verified conflict records for Ukraine, Yemen, and Iraq (January 1 – June 30, 2024), and populates `data/acled_events_raw.csv` locally.
5. Alternatively, download the event export directly from the ACLED portal and place the file at `data/acled_events_raw.csv`.

---

## 3. Committed Benchmarks

### Brent Crude Oil Prices (`data/brent_2024.csv`)
* **File:** `data/brent_2024.csv`
* **Coverage:** January 2, 2024 to June 28, 2024 (125 trading days)
* **Fields:** `date` (YYYY-MM-DD), `brent_close` (USD per barrel)
* **Purpose:** Provides a consistent offline market baseline for correlating Red Sea and Basra thermal anomalies against oil price fluctuations without requiring network calls to financial feeds.

---

## 4. PostgreSQL Relational Database Schema

When the pipeline executes, data is ingested, processed, and stored into a local PostgreSQL database (`conflict_monitoring`) across six structured tables:

```
NASA FIRMS API ──► firms_detections ──► DBSCAN Clustering ──► thermal_events
                                                                     │
News APIs      ──► news_articles ──────── Heuristic Matching ────────┤
                                                                     ▼
ACLED API      ──► acled_events ──────── Ground Truth Check ──► event_matches
                                                                     │
Flare Filtering ──────────────────────── 3-Signal Masking ───► thermal_events_masked
```

### Table Definitions

1. **`firms_detections`**
   * Raw satellite observations from the VIIRS instrument on the Suomi NPP satellite.
   * *Key Columns:* `latitude`, `longitude`, `brightness` (Kelvin), `acq_date`, `acq_time`, `satellite`, `confidence`, `frp` (Fire Radiative Power, MW), `daynight`, `region`.

2. **`news_articles`**
   * Collected war and conflict news items from The Guardian and GDELT.
   * *Key Columns:* `title`, `published_date`, `source`, `url`, `region`, `api_source`, `description`.

3. **`thermal_events`**
   * Discrete spatio-temporal clusters produced by DBSCAN ($\epsilon = 5.0$ km, temporal window $= 3$ days).
   * *Key Columns:* `event_id`, `region`, `start_date`, `end_date`, `duration_days`, `n_detections`, `total_frp`, `max_brightness`, `daynight_ratio`, `centroid_lat`, `centroid_lon`.

4. **`event_matches`**
   * Results of matching clustered thermal events against same-region conflict news within $\pm 3$ days.
   * *Key Columns:* `event_id`, `conflict_associated` (0 or 1), `matching_articles`, `top_article_title`, `top_article_url`, `top_article_date`.

5. **`acled_events`**
   * Independent ground-truth conflict incidents verified by ACLED.
   * *Key Columns:* `event_date`, `country`, `admin1`, `location`, `latitude`, `longitude`, `event_type`, `sub_event_type`, `fatalities`, `region`.

6. **`thermal_events_masked`**
   * Thermal events annotated with the 3-signal industrial flare filter flag (`is_industrial`).
   * *Key Columns:* All fields from `thermal_events` plus `is_industrial` (boolean).
