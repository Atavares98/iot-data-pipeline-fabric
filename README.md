# IoT Data Pipeline — Microsoft Fabric

End-to-end IoT data pipeline built on Microsoft Fabric using Medallion architecture (Bronze → Silver → Gold), integrating real sensor data with external weather data, a machine learning forecasting model, and a Power BI dashboard.

Built as part of an MSc dissertation in Computer Engineering at Universidade Portucalense, developed during an internship at DevScope (Microsoft Gold Partner).

---

## Architecture

<img width="1472" height="1246" alt="imagem" src="https://github.com/user-attachments/assets/c1856b73-214e-4767-983b-ee7db5ea1699" />


**Two parallel pipelines:**
- **Sensor pipeline** — IoT data ingested from MongoDB Atlas via CDC (Change Data Capture)
- **Weather pipeline** — Historical and forecast data from Open-Meteo API

Both pipelines converge at the Gold layer for enriched analytics and forecasting.

---

## Pipeline overview

### Bronze layer
Raw data landed as-is, preserving original structure for auditability.
- `bronze_raw` — raw IoT sensor payloads (JSON) from MongoDB Atlas
- `weather_raw` — hourly historical weather data (Open-Meteo)
- `weather_forecast` — 7-day weather forecast (overwrite on each run)

### Silver layer
Cleaned, validated, and deduplicated data ready for analytics.
- `silver_sensors` — parsed sensor readings with SHA-256 deduplication (`event_id = mac + timestamp_ms`), battery validation (>100 → NULL), and quality flags (`is_valid`)
- `silver_weather` — typed and deduplicated weather data
- `silver_weather_forecast` — cleaned forecast data

Incremental loading via watermark pattern (`etl_state` table tracks `last_watermark` per pipeline).

### Gold layer
Aggregated, business-ready tables.
- `gold_hourly_metrics` — avg/min/max temperature and humidity, avg battery, readings count per sensor/day/hour
- `gold_sensor_health` — reading intervals, anomaly detection (10/30/60 min gaps), null battery count
- `gold_daily_readings` — daily reading counts per sensor
- `gold_hourly_weather` — sensor metrics joined with external weather data
- `gold_weather_forecast` — 7-day forecast aligned with sensor schema

---

## Forecasting model

Random Forest model trained on historical sensor + weather data.

| Target | R² score | Features |
|---|---|---|
| Temperature | 0.91 | temp_ext, hour, lag features |
| Humidity | 0.98 | hum_ext, hour, lag features |

- 2 models, 100 trees each
- 7-day horizon (192 hourly records)
- Output stored in `gold_weather_forecast`

---

## Scale

| Layer | Records |
|---|---|
| Bronze (raw) | ~7.4 million |
| Silver (validated) | ~3.84 million |
| Gold hourly metrics | 16,234 |

Data range: **November 2025 – April 2026**

---

## Tech stack

| Category | Tools |
|---|---|
| Platform | Microsoft Fabric |
| Storage | Delta Lake (Lakehouse) |
| Processing | PySpark, Python |
| Ingestion | MongoDB Atlas CDC, REST API |
| ML | Scikit-learn (Random Forest) |
| Visualisation | Power BI |
| Format | Parquet, Delta |

---

## Repository structure

```
├── notebooks/
│   ├── nb_silver_ingest.ipynb        # Bronze → Silver (sensors, incremental)
│   ├── nb_gold.ipynb                 # Silver → Gold (metrics + health)
│   ├── nb_api_bronze.ipynb           # Weather API → Bronze
│   ├── nb_api_silver.ipynb           # Bronze → Silver (weather)
│   ├── nb_api_gold.ipynb             # Silver → Gold (weather)
│   └── gold_forecast_model.ipynb     # Random Forest forecasting model
├── docs/
│   └── architecture.png              # Pipeline architecture diagram
└── README.md
```

---

## Author

**André Tavares**  
MSc Computer Engineering — Universidade Portucalense  
Internship @ [DevScope](https://devscope.net) · Porto, Portugal  
[LinkedIn](https://www.linkedin.com/in/andretavares98) · [GitHub](https://github.com/Atavares98)
