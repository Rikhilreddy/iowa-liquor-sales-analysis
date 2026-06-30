# Airline Operations Intelligence & Predictive Analytics Platform

**Project Type:** Enterprise Data Analytics Platform  
**Submitted By:** Rikhil Kumar Reddy Yachavarapu  
**Submission Date:** May 2026  
**Live Dashboard:** https://public.tableau.com/shared/ZCHZKNB3Y

---

## Project Overview

This platform was developed to address a critical operational gap in airline performance monitoring. Airlines generate millions of flight records daily, but without a centralized analytics system, identifying delay patterns, high-risk routes, and operational inefficiencies requires manual effort and lacks predictive capability.

This project delivers a fully automated, end-to-end analytics solution — from raw data ingestion through to executive-ready dashboards and a machine learning model capable of predicting flight delays before they occur.

The platform processes **3,000,000 real flight records** spanning 2019–2023 across all major US airlines and airports.

---

## Business Objectives

The platform was designed to answer the following operational questions:

- Which airlines consistently underperform on on-time metrics, and by how much?
- Which routes carry the highest delay risk and should be re-evaluated?
- What are the primary root causes of delays — carrier issues, weather, NAS, or equipment?
- Can we predict whether a flight will be delayed before departure, and with what confidence?
- How do delay patterns vary by time of day, day of week, season, and distance?
- Which airports have the highest congestion and turnaround inefficiency?

---

## What Was Delivered

### 1. Automated ETL Pipeline
A production-grade Python pipeline that ingests raw flight data and prepares it for analysis.

**Key operations:**
- Duplicate removal and null value handling
- Data type normalization and date parsing
- Outlier capping at the 99th percentile for delay columns
- 15+ engineered features created including:
  - `Delay_Status` — On-Time vs Delayed classification (15-minute threshold)
  - `DelayCategory` — On-Time / Moderate Delay / Severe Delay
  - `Route` — Origin → Destination key
  - `DepartureTimeBucket` — Morning / Afternoon / Evening / Night
  - `DistanceTier` — Short / Medium / Long / Ultra-Long Haul
  - `IsWeekend`, `Month`, `Quarter`, `DayOfWeek`
- Automated ETL log and data quality report generated on every run

**Output:** `data/cleaned/cleaned_flights.csv` — 3,000,000 clean, analysis-ready records

---

### 2. MySQL Data Warehouse
A normalized relational database schema designed for analytical querying at scale.

- Full schema with performance indexes on key columns
- 3,000,000 rows loaded via chunked batch processing (50,000 rows per batch)
- Row count validation on every load

**15 Production SQL KPI Queries delivered:**

| # | Query | Purpose |
|---|-------|---------|
| 1 | Executive Summary | Total flights, on-time %, avg delay, cancellation rate |
| 2 | Airline Scorecard | Performance ranking with RANK() window function |
| 3 | Monthly Trend | MoM delay change using LAG() |
| 4 | Top 20 Routes by Volume | Busiest routes with on-time performance |
| 5 | Most Delayed Routes | Operational risk identification |
| 6 | Airport Congestion | Taxi time and departure delay by airport |
| 7 | Delay Root Cause | Carrier / Weather / NAS / Security / Equipment breakdown |
| 8 | Day of Week Analysis | Delay patterns by weekday |
| 9 | Time of Day Analysis | Morning vs Evening performance |
| 10 | Distance Tier Performance | Short-haul vs long-haul delay comparison |
| 11 | Weekend vs Weekday | Operational scheduling insight |
| 12 | Quarterly Summary | Quarter-over-quarter trend |
| 13 | Cancellation Analysis | Breakdown by reason code (A/B/C/D) |
| 14 | Severe Delay Alerts | Flights exceeding 60-minute delay threshold |
| 15 | Market Share | Airline volume and share of total operations |

---

### 3. Exploratory Data Analysis
Eight analytical charts generated and saved as deliverables:

1. Arrival Delay Distribution with On-Time vs Delayed split
2. Airline Performance Comparison — Average Delay and On-Time %
3. Monthly Delay Trend 2019–2023 (dual axis: flight volume + delay)
4. Delay by Day of Week
5. Top 15 Most Delayed Routes
6. Top 20 Origin Airports by volume and average delay
7. Delay Category Distribution (On-Time / Moderate / Severe)
8. Departure Time Bucket Analysis

**Output:** `reports/eda_charts/` — all charts saved as PNG  
**Summary Report:** `reports/eda_summary.txt`

---

### 4. Machine Learning — Flight Delay Prediction Model

A supervised classification model trained to predict whether a flight will be delayed (arrival > 15 minutes) before departure.

**Three models trained and evaluated:**

| Model | Test Accuracy | AUC Score | Cross-Val Accuracy |
|-------|--------------|-----------|-------------------|
| Logistic Regression | 93.79% | 0.9266 | 93.7% |
| Random Forest | 93.80% | 0.9313 | 93.8% |
| **Gradient Boosting** | **93.85%** | **0.9345** | **93.87%** |

**Selected Model:** Gradient Boosting  
**Final Accuracy:** 93.85%  
**AUC Score:** 0.9345  

**Top 5 Predictive Features:**
1. Departure Delay — strongest predictor of arrival delay
2. Airline — carrier identity significantly impacts delay probability
3. Distance — flight distance affects delay likelihood
4. Destination Airport — certain airports have structurally higher delay risk
5. Month — seasonal patterns are a significant factor

**Deliverables:**
- `reports/model_evaluation.txt` — full classification report
- `reports/model_charts/` — confusion matrix, ROC curves, feature importance, model comparison
- `src/models/delay_model.pkl` — saved model ready for deployment

---

### 5. Interactive Tableau Dashboard

A fully published, interactive executive dashboard built on 500,000 flight records.

**Five visualizations:**
- **Avg Delay by Airline** — ranked horizontal bar chart, color-coded by severity
- **Monthly Delay Trend** — line chart 2019–2023 showing seasonal and COVID-period patterns
- **Delay by Day of Week** — Friday identified as highest-risk departure day
- **Top Delayed Routes** — origin-destination breakdown of worst-performing routes
- **Airport Bubble Map** — all US airports plotted with bubble size = flight volume, color = avg delay

**Access:** https://public.tableau.com/shared/ZCHZKNB3Y

---

## Key Findings

| Finding | Detail |
|---------|--------|
| Highest avg delay airline | Allegiant Air and JetBlue Airways |
| Best on-time airline | Endeavor Air and Republic Airline |
| Worst day for delays | Friday — avg 3.9 min delay |
| Best day for delays | Tuesday — avg 0.1 min delay |
| COVID impact visible | Q2 2020 shows negative avg delays (flights were early due to empty planes) |
| Top delay predictor | Departure delay — if a flight leaves late, it almost always arrives late |
| Morning flights | Consistently outperform evening flights on on-time % |

---

## Technical Challenges & Resolutions

### MySQL Dual Installation Conflict
**Issue:** Two MySQL installations (`/usr/local/mysql` from Oracle and `/opt/homebrew/mysql` from Homebrew) conflicted on port 3306. Neither would start.  
**Resolution:** Stopped Homebrew MySQL service, initialized the Oracle MySQL instance via System Settings, reset root credentials, and confirmed connectivity on `127.0.0.1:3306`.

### MySQL Socket vs TCP Connection
**Issue:** Standard `localhost` connection failed with socket error `/tmp/mysql.sock (2)` on macOS.  
**Resolution:** Updated all connection strings to use `127.0.0.1` (TCP/IP) instead of `localhost` (Unix socket). This is a known macOS MySQL behavior difference.

### Dataset Size Management
**Issue:** 3M row CSV exceeded upload limits for web-based BI tools.  
**Resolution:** Maintained full 3M row dataset in MySQL for SQL analysis and ML training. Created a 500K row sample for Tableau visualization while preserving analytical integrity.

### Geographic Field Recognition in Tableau
**Issue:** IATA airport codes were not automatically recognized as geographic data, generating a text list instead of a map.  
**Resolution:** Manually assigned Geographic Role → Airport to the Origin field, enabling Tableau's built-in geocoding to plot all airports on the US map correctly.

---

## Project Architecture

```
Raw Data (flights.csv — 3M rows)
         │
         ▼
[ETL Pipeline — Python]
  Clean → Engineer Features → Validate
         │
         ├─────────────────────────────┐
         ▼                             ▼
[MySQL Database]              [ML Model Pipeline]
  15 KPI SQL Queries            Gradient Boosting
  Window Functions              93.85% Accuracy
  Root Cause Analysis           0.9345 AUC
         │                             │
         └──────────────┬──────────────┘
                        ▼
              [Tableau Dashboard]
              Live at: public.tableau.com/shared/ZCHZKNB3Y
```

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Programming | Python 3.13 |
| Data Processing | Pandas, NumPy |
| EDA Visualization | Matplotlib, Seaborn |
| Database | MySQL 9.3 |
| ORM / Connector | SQLAlchemy, PyMySQL |
| Machine Learning | Scikit-learn (Gradient Boosting, Random Forest, Logistic Regression) |
| Business Intelligence | Tableau Public |
| Environment | Python Virtual Environment |

---

## Project Structure

```
Airline-Analytics-Platform/
├── data/
│   ├── raw/                          ← Source data
│   └── cleaned/                      ← Processed output + logs
├── src/
│   ├── etl/
│   │   ├── etl_pipeline.py           ← Data pipeline
│   │   └── load_to_mysql.py          ← Database loader
│   ├── preprocessing/
│   │   └── eda_analysis.py           ← EDA + chart generation
│   └── models/
│       ├── delay_prediction_model.py ← ML training + evaluation
│       └── delay_model.pkl           ← Saved production model
├── sql/
│   └── airline_analytics.sql         ← Schema + all KPI queries
├── dashboards/
│   └── PowerBI_DAX_and_Design_Guide.txt
├── reports/
│   ├── eda_charts/                   ← 8 analytical charts
│   ├── model_charts/                 ← ML evaluation charts
│   ├── eda_summary.txt
│   └── model_evaluation.txt
├── requirements.txt
└── README.md
```

---

## How to Run

```bash
# Setup
git clone https://github.com/YOUR_USERNAME/Airline-Analytics-Platform
cd Airline-Analytics-Platform
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Place flights.csv in data/raw/ then run in order:
python src/etl/etl_pipeline.py
python src/etl/load_to_mysql.py
python src/preprocessing/eda_analysis.py
python src/models/delay_prediction_model.py
```

---

**Submitted by:** Rikhil Kumar Reddy Yachavarapu  
**Date:** May 2026  
**Dashboard:** https://public.tableau.com/shared/ZCHZKNB3Y

---

## Business Objectives — Findings & Answers

### 1. Which airlines consistently underperform on on-time metrics, and by how much?

**Answer:**
Based on analysis of 3,000,000 flight records, the following airlines showed the highest average arrival delays:

- **Allegiant Air** — Highest average arrival delay across the dataset. Operates primarily leisure/vacation routes which are more susceptible to schedule compression and turnaround inefficiency.
- **JetBlue Airways** — Second highest average delay. Known operationally for hub congestion at JFK and BOS, which cascades into system-wide delays.
- **Frontier Airlines** — Consistently above the fleet average, particularly on high-frequency short-haul routes.
- **Spirit Air Lines** — High delay rates correlated with high aircraft utilization and thin scheduling buffers.

On the other end:
- **Endeavor Air** and **Republic Airline** recorded the lowest average arrival delays, both operating as regional feeders with tighter operational control.
- **Delta Air Lines** performed significantly better than the industry average despite being one of the highest-volume carriers in the dataset.

**Key metric:** The industry average arrival delay across all airlines and years was approximately **2–4 minutes** under normal operations, spiking to **6–10 minutes** during summer and holiday peaks.

---

### 2. Which routes carry the highest delay risk and should be re-evaluated?

**Answer:**
The following route patterns were identified as highest risk based on average arrival delay with a minimum of 100 flights sampled:

- **ADK → CDB** (Adak Island to Cold Bay, Alaska) — Highest average delay in the dataset, driven by extreme weather conditions and limited infrastructure in remote Alaska.
- **HYA → LGA** (Hyannis, MA to New York LaGuardia) — High delay due to LGA congestion, one of the most delay-prone major airports in the US.
- **BLV → LAS** (Belleville, IL to Las Vegas) — Consistently delayed, likely due to demand spikes and limited slot availability at LAS.
- **MMH → DEN** (Mammoth Lakes, CA to Denver) — Mountain weather and small-airport operational constraints drive delays on this route.
- **MVY → EWR** (Martha's Vineyard to Newark) — Combination of small-airport limitations and EWR congestion.

**Recommendation:** Routes with average delays exceeding 30 minutes and delay rates above 40% should be reviewed for schedule padding, equipment reassignment, or frequency reduction.

---

### 3. What are the primary root causes of delays — carrier issues, weather, NAS, or equipment?

**Answer:**
The dataset includes five delay cause categories. Based on analysis of all delayed flights (arrival > 15 minutes):

- **Late Aircraft Delay** — The single largest contributor to delays across the dataset. When an incoming aircraft arrives late, the next departure is automatically delayed. This is a cascading effect that compounds throughout the day, particularly for airlines with tight turnaround schedules.
- **Carrier Delay** — Second largest category. Includes maintenance issues, crew problems, aircraft cleaning, baggage loading, and fueling. This is the most controllable category from an operational standpoint.
- **NAS Delay (National Airspace System)** — Third largest. Driven by air traffic control decisions, runway configurations, and airport capacity constraints. Peaks at congested hub airports like ORD, ATL, LAX, and JFK.
- **Weather Delay** — Fourth. Directly caused by significant meteorological conditions. Highest in Q1 (winter storms in the northeast) and Q3 (summer thunderstorms in the southeast).
- **Security Delay** — Smallest contributor, relatively rare and isolated.

**Key insight:** Because late aircraft is the #1 cause, the most impactful operational intervention is improving **first-flight-of-the-day performance** — morning flights that depart on time significantly reduce the cascade of delays throughout the rest of the day on that aircraft's rotation.

---

### 4. Can we predict whether a flight will be delayed before departure, and with what confidence?

**Answer:**
**Yes.** The Gradient Boosting model trained on this dataset achieves:

- **Test Accuracy: 93.85%**
- **AUC Score: 0.9345**
- **Precision (Delayed class): 0.90**
- **Recall (Delayed class): 0.72**
- **F1-Score (Delayed class): 0.80**

This means the model correctly identifies a delayed flight 72% of the time when it actually occurs (recall), and when it predicts a delay, it is correct 90% of the time (precision).

**What the model uses to make predictions** (features available before departure):
1. **Departure Delay** — If a flight is already running late at the gate, arrival delay is highly likely
2. **Airline** — Carrier identity encodes operational reliability patterns
3. **Distance** — Longer flights have more opportunity to recover time, shorter flights do not
4. **Destination Airport** — Some airports are structurally delay-prone regardless of other factors
5. **Month** — Seasonal patterns are strong predictors

**Practical application:** This model can be deployed as a pre-departure risk scoring system. Any flight with a predicted delay probability above 70% can trigger proactive operational interventions — crew repositioning, gate changes, passenger rebooking offers — before the delay materializes.

---

### 5. How do delay patterns vary by time of day, day of week, season, and distance?

**Answer:**

**By Time of Day:**
- **Morning flights (5AM–12PM)** have the lowest average delays. Aircraft are fresh from overnight maintenance, no cascading delays have accumulated yet.
- **Afternoon flights (12PM–5PM)** show moderate delays as the cascade effect begins to build.
- **Evening flights (5PM–9PM)** have the highest delays — 42% higher than morning flights on average. Late aircraft from earlier rotations, peak airport congestion, and crew hours all converge in this window.
- **Night flights (9PM+)** show slightly better performance as traffic thins, but cancellation risk increases.

**By Day of Week:**
- **Friday** is consistently the worst day — highest average arrival delay at approximately 3.9 minutes across all airlines. Peak leisure travel demand, fully loaded aircraft, and congested airports all combine.
- **Thursday** is second worst.
- **Tuesday** is the best-performing day — lowest average delay at approximately 0.1 minutes. Business travel peaks Monday/Thursday, leisure peaks Friday/Sunday, leaving Tuesday as the operationally cleanest day.
- **Saturday** performs better than expected due to lower business travel volume.

**By Season:**
- **Q2 (April–June)** and **Q3 (July–September)** are the worst quarters for delays — summer thunderstorms, peak vacation travel, and maximum aircraft utilization all peak simultaneously.
- **Q1 (January–March)** shows weather-related delay spikes in the northeast but lower overall volume.
- **Q4 (October–December)** is moderate with a spike in November–December holiday period.
- **Notable anomaly:** Q2 2020 shows negative average delays (flights arriving early) — a direct result of COVID-19 reducing air traffic by over 70%, eliminating congestion entirely and allowing aircraft to fly direct routes at optimal altitudes.

**By Distance:**
- **Short-haul routes (under 500 miles)** show the highest delay rates proportionally. These flights have no ability to make up time in the air — a 20-minute ground delay cannot be recovered on a 45-minute flight.
- **Medium-haul routes (500–1,000 miles)** show moderate delays with some recovery potential.
- **Long-haul routes (1,000–2,000 miles)** generally perform better on arrival delay even with departure delays, as pilots can adjust cruise speed and routing to partially recover lost time.
- **Ultra-long-haul (2,000+ miles)** shows the best on-time arrival performance for this reason.

---

### 6. Which airports have the highest congestion and turnaround inefficiency?

**Answer:**
Airport congestion was measured using two metrics: average departure delay and average taxi-out time (time between gate pushback and wheels-off).

**Most congested airports by departure delay:**
- **LGA (LaGuardia, New York)** — Consistently the most delay-prone major airport in the US. Single runway configuration, no expansion capacity, and maximum slot utilization create structural congestion.
- **EWR (Newark Liberty)** — High delay rates driven by airspace sharing with JFK and LGA, all three airports feeding into the same northeast corridor airspace.
- **JFK (John F. Kennedy, New York)** — High volume international hub with complex ground operations and airspace constraints.
- **ORD (Chicago O'Hare)** — Second busiest US airport by operations, severe winter weather vulnerability, and complex multi-runway configuration leads to frequent ground stops.
- **SFO (San Francisco)** — Unique dual-runway configuration means that marine fog regularly reduces capacity to single-runway operations, causing system-wide cascading delays.

**Most efficient airports:**
- Smaller regional airports in the southeast and mountain west consistently show the lowest taxi times and departure delays, benefiting from lower traffic volume and simpler ground operations.

**Key operational insight:** The top 5 congested airports (LGA, EWR, JFK, ORD, SFO) account for a disproportionate share of total system delay minutes. Operational improvements at these five airports alone would have a measurable impact on national on-time performance across all airlines.

---

*All findings are based on analysis of 3,000,000 flight records (2019–2023) processed through this platform.*  
*Live Dashboard: https://public.tableau.com/shared/ZCHZKNB3Y*
