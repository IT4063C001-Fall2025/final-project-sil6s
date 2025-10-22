# 🗂️ Data Types & Sources

This document explains each dataset used in the **NYC Bus Reliability vs Weather** project — including acquisition method, key fields, and intended use.

---

## 1️⃣ MTA Bus Performance (CSV)
**File:** `data/bus_data.csv`  
**Method:** Static CSV download from MTA Bus Performance Dashboard.  
**Purpose:** Quantify **bus reliability** via **Wait Assessment (WA)** — the share of observed trips that meet scheduled headways.

| Column | Description | Type | Example |
|:-------|:-------------|:------|:--------|
| `month` | Month-level date stamp (YYYY-MM-DD) | `datetime` | `2020-01-01` |
| `borough` | NYC borough name | `string` | `Bronx` |
| `day_type` | MTA category (e.g., weekday/weekend) | `string` | `1` (Weekday) |
| `trip_type` | Route service type | `string` | `LCL/LTD` |
| `route_id` | MTA route identifier | `string` | `BX1` |
| `period` | Day period (e.g., Peak, Off-Peak) | `string` | `Off-Peak` |
| `number_of_trips_passing_wait` | Trips meeting headway threshold | `int` | `16464` |
| `number_of_scheduled_trips` | Total scheduled trips | `int` | `21728` |
| `wait_assessment` | Share of trips meeting headway target | `float` (0–1) | `0.7577` |

**Derived fields:**  
- `wa_weighted` = WA weighted by scheduled trips per route/period  
- `pct_passing` = passing ÷ scheduled  

---

## 2️⃣ NOAA Weather Data (CSV)
**File:** `data/weather_data.csv`  
**Method:** Historical daily data from NOAA GHCN-D (Central Park Station `USW00094728`).  
**Purpose:** Measure **weather exposure** — temperature, precipitation, snow, and wind.

| Column | Description | Type | Units | Example |
|:-------|:-------------|:------|:------|:--------|
| `STATION` | NOAA station ID | `string` | — | `USW00094728` |
| `DATE` | Observation date | `datetime` | — | `2020-01-01` |
| `NAME` | Station name | `string` | — | `NY CITY CENTRAL PARK, NY US` |
| `PRCP` | Daily precipitation | `float` | tenths of mm | `191` |
| `SNOW` | Daily snowfall | `float` | tenths of mm | `229` |
| `SNWD` | Snow depth | `float` | tenths of mm | `0` |
| `TMAX` | Max temperature | `float` | tenths °C | `-17` |
| `TMIN` | Min temperature | `float` | tenths °C | `-72` |
| `TAVG` | Avg temperature | `float` | tenths °C | `-45` |
| `AWND` | Average wind speed | `float` | tenths of m/s | `38` |
| `LATITUDE`, `LONGITUDE` | Station coordinates | `float` | degrees | `40.78`, `-73.96` |

**Transformations applied:**  
- Convert tenths → metric units (°C, mm, m/s)  
- Impute `TAVG` = (`TMIN` + `TMAX`) / 2 if missing  
- Aggregate to **monthly means/sums** by date  

---

## 3️⃣ NYC DOT Traffic API (JSON)
**Endpoint:** [https://data.cityofnewyork.us/resource/i4gi-tjb9.json](https://data.cityofnewyork.us/resource/i4gi-tjb9.json)  
**Method:** REST API returning live and historical traffic speed observations from roadway sensors.  
**Purpose:** Capture **congestion** and **average travel speeds** as a mediating factor between weather and bus reliability.

| Field | Description | Type | Example |
|:-------|:-------------|:------|:--------|
| `segmentid` | Unique road segment ID | `string` | `100001` |
| `borough` | NYC borough name | `string` | `Queens` |
| `speed` | Observed vehicle speed | `float` | `25.3` |
| `travel_time` | Average travel time for the segment | `float` | `210.4` |
| `recordedtimestamp` | UTC timestamp of observation | `datetime` | `2024-01-15T08:30:00.000` |

**Aggregation:**  
- Convert to monthly mean `speed` per borough or systemwide average  
- Derived features: `mean_speed_mph`, `%_below_10mph`, `pm_peak_speed_mph`

---

## 4️⃣ MTA Bus Alerts Feed (Protocol Buffers)
**Endpoint:** [https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/camsys%2Fbus-alerts](https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/camsys%2Fbus-alerts)  
**Method:** GTFS-Realtime feed in **Protocol Buffers** format.  
**Purpose:** Measure **bus service disruptions** (detours, cancellations, delays) in real time.

| Field | Description | Type | Example |
|:-------|:-------------|:------|:--------|
| `header.timestamp` | Feed generation timestamp | `int` (epoch) | `1713806122` |
| `entity.id` | Unique alert ID | `string` | `MTA_BUS_ALERT_15243` |
| `alert.cause` | Type of disruption | `enum` | `WEATHER`, `CONSTRUCTION` |
| `alert.effect` | Impact on service | `enum` | `DETOUR`, `STOP_CLOSED`, `REDUCED_SERVICE` |
| `alert.active_period` | Start and end times | `datetime` range | `2024-02-15T10:00Z` → `2024-02-15T15:00Z` |
| `alert.informed_entity.route_id` | Bus route affected | `string` | `M15` |

**Processing plan:**  
- Parse feed via `gtfs-realtime-bindings`  
- Count `len(alerts)` per day → aggregate to `alerts_per_month`  
- Merge with weather and bus reliability data for correlation modeling

---

## 🧮 Summary of Methods

| Source | Type | Acquisition | Update Frequency | Role in Model |
|:-------|:------|:-------------|:-----------------|:--------------|
| MTA Bus Performance | CSV | Static download | Monthly | Dependent variable (`wa_weighted`) |
| NOAA Weather | CSV | Static download | Daily (aggregated monthly) | Independent variable (`precip`, `snow`, `tavg`) |
| NYC DOT Traffic | JSON API | Live API | Hourly (aggregated monthly) | Mediator (`mean_speed_mph`) |
| MTA Bus Alerts | Protocol Buffers | Real-time GTFS feed | Every few minutes (aggregated monthly) | Disruption indicator (`alerts_per_month`) |

---

**Author:** Silas Curry  
**Project:** NYC Bus Reliability vs Weather  
**Last updated:** October 2025
