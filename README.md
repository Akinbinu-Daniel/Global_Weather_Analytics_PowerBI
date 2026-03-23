# Global_Weather_Analytics_PowerBI
End-to-end Power BI project: raw CSV → star schema → 85 DAX measures across solar, wind, flight risk, insurance, tourism and air quality analytics.
# 🌍 Global Weather Analytics Dashboard
### A Multi-Domain Business Intelligence Project | Power BI

---

## 📌 Project Overview

This project is a fully self-contained business intelligence solution built entirely in **Power BI Desktop**, analysing global weather data across **244+ city locations worldwide**. The analysis transforms a raw CSV dataset into a structured semantic model with **85 DAX measures** across six strategic business domains — all without any external tools, Python, or SQL.

The core question this project answers:

> *How does weather, climate, and environmental conditions affect business operations, investment decisions, and strategic planning across different global markets?*

---

## Dataset

| Property | Detail |
|---|---|
| **Source** | GlobalWeatherRepository (CSV) |
| **Format** | Flat CSV — 38 columns, 244+ location rows |
| **Coverage** | Global cities across Africa, Europe, Asia, Americas, Middle East |
| **Key fields** | Temperature, humidity, wind speed, UV index, cloud cover, air quality indices, sunrise/sunset, precipitation, pressure, moon phase |
| **Tool used** | 100% Power BI Desktop (Power Query + DAX) |

---

## Data Processing Pipeline

All data transformation was performed inside **Power Query (M language)** within Power BI. No external scripts or tools were used.

### Step 1 — Raw Data Ingestion
The source CSV was loaded into Power Query using `Csv.Document()` with explicit column type definitions for all 38 columns. Types were enforced at ingestion (text, number, integer, time, date) to prevent downstream errors.

### Step 2 — Data Cleaning
- Standardised inconsistent condition text values (e.g. `"Partly Cloudy"` → `"Partly cloudy"`) using `Table.ReplaceValue()`
- Trimmed and cleaned the date column using `Text.Trim()` and `Text.Clean()` to remove hidden whitespace characters that caused date parsing failures
- Parsed the date column with explicit locale (`en-GB`) to handle day-first date formats correctly
- Removed redundant columns (`moonrise`, `moonset`, `visibility_miles`) to reduce model size

### Step 3 — Data Normalisation (Star Schema Design)
The flat CSV was normalised into a **star schema** with one fact table and five dimension tables:

```
GlobalWeatherRepository (Fact Table)
    │
    ├── DIM_Location          (Country | City | Timezone)
    ├── DIM_condition_text    (Weather condition labels)
    ├── DIM_Wind_direction    (Wind direction categories)
    ├── DIM_Moon_phase        (Moon phase categories)
    └── Dim_Date              (Date dimension with Year, Quarter, Month, Day)
```

Each dimension table was created by:
1. Selecting relevant columns from the source
2. Removing duplicates (`Table.Distinct()`)
3. Adding a surrogate integer index key (`Table.AddIndexColumn()`)
4. Joining back to the fact table via `Table.NestedJoin()` to replace text columns with integer foreign keys

This eliminated data redundancy and enabled efficient DAX filtering across all dimensions.

### Step 4 — Calculated Columns
Two calculated columns were added directly in the data model:
- **`Temp_Kelvin`** — Converts Celsius to Kelvin (`temperature_celsius + 273.15`) for use in thermodynamic calculations
- **`Air_Quality_Above_Limit`** — Binary flag (0/1) indicating whether a row exceeds WHO 2021 air quality thresholds for PM2.5 (≥5 μg/m³) or PM10 (≥15 μg/m³)
- **`Humidity %`** — Converts the integer humidity column to a decimal percentage for cleaner formatting

### Step 5 — DAX Measure Development
85 measures were built across six analytical domains. See **Analytical Domains** section below.

### Step 6 — Model Organisation
All 85 measures were organised into **8 display folders** for navigability:

| Folder | Purpose |
|---|---|
| ☀️ Solar Analysis | Solar radiation, geometry, UV, daylight, efficiency |
| 💨 Wind Analysis | Wind power density, air density, feasibility scoring |
| ✈️ Flight Risk | Flight delay risk scoring and component hazard rates |
| 🛡️ Insurance Risk | Weather hazard rates and composite insurance risk score |
| 🌍 Tourism | Tourism weather scoring and ranking |
| 📦 Logistics & Operations | Demand signals, productivity risk, logistics disruption |
| 🌫️ Air Quality | EPA index, exceedance rates, city-level air quality risk |
| 📅 Base Metrics | Flags, day counts, correlation coefficients, temperature history |

---

## 📊 Analytical Domains & Key Measures

### ☀️ 1. Solar Energy Market Analysis
Identifies optimal global markets for solar energy investment using physics-based calculations.

**Methodology:**
- `Solar_Declination` — Earth's axial tilt formula: `23.45 × SIN(RADIANS(360 × (284 + Day_of_Year) / 365))`
- `Latitude_Rad` — Converts average latitude to radians for trigonometric use
- `Solar_Radiation_Factor` — Geometry-adjusted solar radiation: `MAX(0, COS(Latitude_Rad - RADIANS(Solar_Declination)))`
- `Cloud_Factor` — Sky clearance factor: `1 - AVG(cloud) / 100`
- `UV_Factor` — Normalised UV index relative to global maximum
- `Temp_Efficiency` — Panel degradation factor: efficiency drops 0.4% per °C above 25°C
- `Daylight_Score` — Normalised daylight hours per location
- `Solar_Power_Index` — Composite: `Solar_Radiation_Factor × Daylight_Score × Cloud_Factor × UV_Factor × Temp_Efficiency`
- `Solar_Index_Normalized` — Indexed to global maximum for cross-location comparison
- `Solar_Strategic_Class` — Market classification: High / Moderate / Low Solar Market

---

### 💨 2. Wind Energy Market Analysis
Assesses wind energy viability using atmospheric physics.

**Methodology:**
- `Air_Density` — Calculated using the **Arden Buck equation** for vapour pressure and the specific gas constant for dry air (287.058 J/kg·K): `(pressure_mb × 100 - 0.378 × P_v) / (287.058 × Temp_K)`
- `Wind_Speed_ms` — Converts kph to m/s: `AVG(wind_kph) / 3.6`
- `Wind_Power_Density` — Wind power formula: `0.5 × Air_Density × Wind_Speed_ms³`
- `Wind_Index` — Normalised relative to global total wind power
- `Wind_Feasibility` — Market tier: High / Moderate / Low Wind Market
- `Renewable_Market_Type` — Combined solar + wind classification: Hybrid Opportunity, Wind-Dominant, Solar-Dominant, or Low Renewable Priority

---

### ✈️ 3. Flight Delay Risk Analysis
Scores locations by their likelihood of causing weather-related flight disruptions.

**Hazard thresholds (aviation research-based):**
| Hazard | Threshold | Weight |
|---|---|---|
| High Wind | ≥ 50 kph | 30% |
| Heavy Rain | ≥ 30 mm | 25% |
| Low Visibility (humidity proxy) | ≥ 90% humidity | 20% |
| Extreme Cold | ≤ 0°C | 15% |
| Extreme Heat | ≥ 40°C | 10% |

- `Flight_Delay_Risk_Score` — Weighted composite of all five hazard rates (decimal format)
- `Flight_Delay_Rank` — Dense rank across all locations (1 = highest risk)

---

### 🛡️ 4. Insurance Risk Analysis
Models weather-driven insurance exposure for each city.

**Hazard weights (based on global insurance loss data):**
| Hazard | Threshold | Weight |
|---|---|---|
| Extreme Rain | ≥ 50 mm | 40% |
| High Wind | ≥ 60 kph | 25% |
| Extreme Heat | ≥ 40°C | 15% |
| Extreme Cold | ≤ -15°C | 10% |
| Poor Air Quality | EPA Index ≥ 4 | 10% |

- `Insurance_Risk_Score` — Composite weighted hazard rate with `COALESCE()` null handling
- `Insurance_Risk_Rank` — Global ranking (1 = highest insurance risk)

---

### 🌍 5. Tourism Weather Scoring
Ranks locations by the quality and consistency of their weather for tourism.

**Ideal tourism conditions:**
- Temperature: 18°C – 28°C
- Precipitation: < 5mm
- Wind: < 25 kph
- Humidity: < 70%

**Weights:** Temperature 40% | Rain 30% | Wind 15% | Humidity 15%

- `Tourism_Weather_Score` — Composite score (displayed as percentage)
- `Tourism_Rank` — Dense rank across all locations

---

### 📦 6. Logistics & Operations Risk
Assesses weather impact on supply chain, outdoor productivity, and product demand.

- `Logistics_Risk_signal` — Sum of active hazard flags (0 = no risk, 3+ = severe)
- `Productivity_Risk` — Combines low daylight, extreme heat, and extreme cold flags
- `Productivity_Risk_reading` — Text classification: No Risk / Prone to Risk / Terrible Disruption
- `Expected_Product_Demand` — Signal-based demand forecast (Strong Hot / Neutral / Strong Cold Product Demand)
- `Pearson Temp vs Spoilage` — Manual Pearson correlation coefficient between temperature and humidity (spoilage proxy), computed entirely in DAX without built-in correlation functions

---

### 🌫️ 7. Air Quality Analysis
Monitors compliance with WHO 2021 air quality standards.

- `Poor_AQ_%` — % of records with EPA index ≥ 4 (Unhealthy)
- `Air_Quality_Risk_Level` — Text classification: Low / Occasional / Moderate / High Risk
- `City_Exceedance_Rate` — Average rate at which a city breaches WHO limits
- `%_Regularly_Poor_Air_Cities` — % of cities where ≥40% of records exceed limits

---

## 🔢 Model Statistics

| Metric | Count |
|---|---|
| Total DAX measures | 85 |
| Dimension tables | 5 |
| Fact table rows | ~244 locations |
| Relationships | 6 |
| Analytical domains | 6 |
| Calculated columns | 3 |

---

## 🛠️ Tools Used

- **Power BI Desktop** — Full project (data ingestion, transformation, modelling, DAX, visualisation)
- **Power Query (M)** — All ETL and normalisation
- **DAX** — All 85 analytical measures

No Python, SQL, R, or external tools were used at any stage.

---

## 👤 Author

Built independently as a portfolio project demonstrating end-to-end business intelligence development — from raw CSV to a fully normalised semantic model with advanced DAX analytics.

**Skills demonstrated:** Data modelling · Star schema design · Power Query ETL · Advanced DAX · Physics-based calculations · Multi-domain business analysis · Composite scoring methodologies

---

## 📄 License

This project is for portfolio and educational purposes.
