# 📊 Data Sources Documentation

This document describes the datasets used in the **Public Transit Reliability vs. Weather** project for IT4063C – Data Technologies & Analytics.

---

## 🚌 Dataset 1 — MTA Subway Customer Journey-Focused Metrics (2020–2024)

**Filename:** `subway_data.csv`  
**Type:** CSV (File Download)  
**Source:** [MTA Open Data Portal – Subway Customer Journey-Focused Metrics (2020–2024)](https://data.ny.gov/Transportation/MTA-Subway-Customer-Journey-Focused-Metrics-2020-2024/39hk-dx4f)  

### **Description**
This dataset contains operational and performance metrics for New York City’s subway system, focusing on how well trains adhere to scheduled service and rider expectations.  
It measures both **service reliability** (on-time performance, travel time) and **customer experience** (additional waiting or travel time).

Each row represents a specific combination of:
- Date (or Month)
- Subway Division (e.g., A Division)
- Subway Line (numeric identifier)
- Time Period (`peak` or `offpeak`)

These features allow analysis of **temporal and operational patterns** in subway reliability.

### **Columns**
| Column | Description |
|--------|--------------|
| `month` | Observation date (YYYY-MM-DD) |
| `division` | Subway division (A or B Division) |
| `line` | Subway line number |
| `period` | Time of day (`peak` or `offpeak`) |
| `num_passengers` | Estimated number of passengers |
| `additional platform time` | Average additional waiting time (minutes) |
| `additional train time` | Average additional in-train time (minutes) |
| `total_apt` | Total accumulated additional platform time |
| `total_att` | Total accumulated additional travel time |
| `over_five_mins` | Number of riders delayed over five minutes |
| `over_five_mins_perc` | Percentage of riders delayed over five minutes |
| `customer journey time performance` | On-time performance ratio for the period |

### **Purpose**
This file serves as the **primary transit performance dataset**.  
It provides the quantitative basis for understanding how external conditions, such as weather, affect subway reliability and overall passenger experience.

---

## 🌦️ Dataset 2 — NOAA Global Historical Climatology Network (GHCN-Daily)

**Filename:** `weather_data.csv`  
**Type:** CSV (File Download)  
**Source:** [NOAA National Centers for Environmental Information (NCEI) – GHCN-Daily](https://www.ncei.noaa.gov/products/land-based-station/global-historical-climatology-network-daily)  
**Station Used:** `USW00094728 – New York City Central Park, NY US`

### **Description**
The **GHCN-Daily** dataset provides daily meteorological observations from land-based weather stations across the globe.  
For this project, the subset corresponding to the **Central Park (NYC)** station is used to represent local environmental conditions near the subway system.

The dataset includes daily temperature, precipitation, and snowfall data from **1869 to present**, though this project focuses on **2020–2024** to align with subway performance records.

### **Columns**
| Column | Description |
|--------|--------------|
| `STATION` | NOAA weather station identifier (e.g., USW00094728) |
| `DATE` | Observation date (YYYY-MM-DD) |
| `LATITUDE`, `LONGITUDE` | Geographic coordinates of the station |
| `ELEVATION` | Station elevation in meters |
| `NAME` | Station name |
| `PRCP` | Daily total precipitation (tenths of mm) |
| `SNOW` | Daily snowfall (tenths of mm) |
| `SNWD` | Snow depth (tenths of mm) |
| `TMAX` | Maximum temperature (tenths of °C) |
| `TMIN` | Minimum temperature (tenths of °C) |
| `TAVG` | Average daily temperature (when available) |
| `AWND` | Average wind speed (tenths of m/s) |
| `WT##` | Weather event codes (e.g., fog, rain, snow indicators) |

### **Purpose**
This dataset provides the **weather variables** needed to analyze external environmental influences on transit reliability.  
It will be merged with the subway dataset using the `DATE` field to explore how conditions like precipitation, snowfall, and extreme temperatures affect average delays and on-time ratios.

---

## 🔗 Combined Use

Together, these datasets allow for the correlation of **transit performance metrics** (MTA) with **environmental conditions** (NOAA).  
They form the foundation for Exploratory Data Analysis (EDA), visualization, and eventual predictive modeling of how weather impacts public transit reliability in New York City.
s