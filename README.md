# Assignment 3: NASA Events & Weather Data Pipeline (Pack C)

## Overview
A medallion-architecture data pipeline (Bronze → Silver → Gold) that pulls natural disaster events from NASA EONET and historical weather data from Open-Meteo, then joins them into an analysis-ready dataset for statistical hypothesis testing.

## APIs Used
| API | Auth | Data Pulled |
|-----|------|-------------|
| [NASA EONET v3](https://eonet.gsfc.nasa.gov/docs/v3) | None | Wildfire and severe storm events for 2024 |
| [Open-Meteo Historical Weather](https://open-meteo.com/en/docs/historical-weather-api) | None | Daily weather for Los Angeles, 2024 |

**Why Los Angeles?** LA is a wildfire-prone region, making it a natural fit for exploring the relationship between weather conditions and natural disaster events.

## Project Structure
```
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── analysis_preview.md
├── data/
│   ├── bronze/
│   │   ├── eonet/          ← raw NASA EONET JSON snapshots
│   │   └── weather/        ← raw Open-Meteo JSON snapshots
│   ├── silver/
│   │   ├── eonet_daily_counts.csv
│   │   └── la_weather_daily.csv
│   └── gold/
│       └── nasa_weather_daily.csv
├── ingest/
│   ├── ingest_eonet.py
│   └── ingest_weather.py
├── transform/
│   ├── transform_eonet.py
│   ├── transform_weather.py
│   └── build_gold.py
└── notebooks/
```

## How to Run

### 1. Setup
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. Ingestion (Bronze)
```bash
python3 ingest/ingest_eonet.py
python3 ingest/ingest_weather.py
```
Run each script at least twice to create multiple Bronze snapshots.

### 3. Transform (Silver)
```bash
python3 transform/transform_eonet.py
python3 transform/transform_weather.py
```

### 4. Build Gold
```bash
python3 transform/build_gold.py
```

## Pipeline Details

### Bronze Layer
- EONET: Raw JSON responses from NASA EONET API containing wildfire and severe storm event metadata for 2024.
-Weather: Raw JSON responses from Open-Meteo archive API with daily temperature, precipitation, and wind speed for Los Angeles.

### Silver Layer
- **eonet_daily_counts.csv:** Events aggregated to daily counts. Each event's earliest geometry date is used. Events outside 2024 are filtered out. Columns: `date`, `event_count`, `wildfire_count`, `storm_count`.
- **la_weather_daily.csv:** Flat daily weather table. Null precipitation values filled with 0.0. Columns: `date`, `temp_max`, `temp_min`, `precipitation_sum`, `windspeed_max`.

### Gold Layer
- **nasa_weather_daily.csv:** Left join from weather (366 days) to EONET counts on `date`. Days with no events get 0 counts.

**Derived columns:**
| Column | Description |
|--------|-------------|
| `rainy_day` | 1 if precipitation > 0 mm |
| `event_day` | 1 if any EONET event occurred |
| `high_wind_day` | 1 if wind speed > median |
| `season` | Winter / Spring / Summer / Fall |
| `is_weekend` | 1 if Saturday or Sunday |

## Gold Dataset Columns
`date`, `temp_max`, `temp_min`, `precipitation_sum`, `windspeed_max`, `event_count`, `wildfire_count`, `storm_count`, `rainy_day`, `event_day`, `high_wind_day`, `season`, `is_weekend`

## Statistical Tests Supported
| Test | Variables |
|------|-----------|
| One-sample t-test | `event_count` vs benchmark |
| Two-sample t-test | `event_count` grouped by `rainy_day` |
| Proportion z-test | `event_day` grouped by `season` |

See `analysis_preview.md` for full hypothesis details.

## AI Usage
- **Tools used:** Kiro AI assistant (IDE-integrated)
- **Helped with:** Project scaffolding, writing ingestion/transform scripts, DuckDB SQL for Gold layer joins, drafting documentation.
- **Verified/fixed:** The EONET transform initially included events with geometry dates outside 2024 (e.g., 2023-12-31) despite the API query filtering to 2024. This was caught by inspecting the Silver CSV output and fixed by adding a date filter in the transform step. Also had to switch from pandas to pure Python + DuckDB due to GCC version incompatibility with numpy on the build environment.
